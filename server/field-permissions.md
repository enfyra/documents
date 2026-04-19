# Field Permissions

Control access to individual columns and relations per role, user, or condition.

## Visibility Baseline: `isPublished`

Every column and relation has an `isPublished` flag (default: `true`).

- `isPublished: true` — accessible by all authenticated users by default
- `isPublished: false` — blocked by default. Only accessible via explicit `allow` rules in `field_permission_definition`

Unpublished fields are omitted entirely from API responses (not returned as `null`). Root admin bypasses all field permission checks.

Sensitive columns like `password`, OAuth `clientSecret`, and storage credentials use `isPublished: false`.

## field_permission_definition

Each rule targets one column OR one relation (not both):

```
POST /api/field_permission_definition
{
  "column": { "id": "<column_definition_id>" },
  "role": { "id": "<role_id>" },
  "action": "read",
  "effect": "allow",
  "isEnabled": true
}
```

| Field | Type | Description |
|---|---|---|
| `column` | FK  column_definition | Target column (null if targeting a relation) |
| `relation` | FK  relation_definition | Target relation (null if targeting a column) |
| `role` | FK  role_definition | Role this rule applies to |
| `allowedUsers` | M2M  user_definition | Specific users (alternative to role — use one or the other) |
| `action` | enum: `read`, `create`, `update` | Which operation this rule controls |
| `effect` | enum: `allow`, `deny` | Whether to allow or deny access |
| `condition` | JSON | Optional filter DSL condition (evaluated per record) |
| `isEnabled` | boolean | Toggle rule on/off |

The table is derived from the column/relation FK — no separate `table` field needed.

### Inverse Relations

`column_definition` and `relation_definition` both have a `fieldPermissions` inverse relation back to `field_permission_definition`. This allows fetching a column/relation with its permission rules embedded:

```
GET /api/column_definition?filter[table][id][_eq]=<tableId>&fields=id,name,fieldPermissions.id,fieldPermissions.effect
```

## Scope

Each rule uses **either** `role` or `allowedUsers`, not both:
- `role` — applies to all users with that role
- `allowedUsers` — applies to specific users regardless of role
- Neither set — global rule (applies to everyone)

## Conditions

Optional JSON filter evaluated per record. Uses the standard filter DSL with `@USER.id` macro:

```json
{
  "ownerId": { "_eq": "@USER.id" }
}
```

This condition allows access only when the record's `ownerId` matches the current user. Conditional rules are evaluated post-SQL (need actual record data).

## Rule Priority

When multiple rules match, they're evaluated by tier (highest priority first):

1. User-specific + conditional
2. Role-based + conditional
3. User-specific + unconditional
4. Role-based + unconditional

Within a tier, `deny` wins over `allow`.

## Enforcement Flow

```
find()
  1. assertQueryAllowed()     — throw 403 if denied field used in filter/sort/aggregate
  2. stripDeniedFields()      — remove denied columns/relations from SELECT/JOIN (pre-SQL)
  3. queryEngine.find()       — SQL only fetches allowed fields
  4. sanitizeFieldPermissions — safety net for conditional rules (post-SQL)
```

- **Read**: denied fields are omitted from the response entirely
- **Filter/sort/aggregate**: denied fields throw `403`
- **Create/update**: denied fields in the request body throw `403`
- **Cascade writes**: field permissions are enforced on child records during cascade create/update

### Pre-SQL Optimization

Unconditional denials are resolved before SQL execution:
- Denied columns are removed from the SELECT list
- Denied relations skip the JOIN entirely
- Wildcard queries (`fields=*`) are resolved to an explicit column list with denied columns excluded

Conditional rules (with `condition` set) cannot be resolved pre-SQL because they depend on record data. These are handled by the post-SQL safety net.

### Primary Keys

Primary key columns (`id` / `_id`) are never stripped, regardless of permission rules.

### Repositories in scripts

- **`$ctx.$repos.main`** and **`$ctx.$repos.secure.<table>`** apply field permission rules (strip/deny as configured).
- **`$ctx.$repos.<table>`** (except `main` / `secure`) does **not** enforce field permissions by default — use **`secure`** when access control must match the metadata rules.

## GraphQL

Unpublished columns and relations (`isPublished: false`) are excluded from the generated GraphQL schema. They won't appear in type definitions, input types, or update input types.

## Examples

### Make a column private (only managers can read)

1. Set `isPublished: false` on the column (via table schema editor)
2. Create an allow rule:

```
POST /api/field_permission_definition
{
  "column": { "id": "<salary_column_id>" },
  "role": { "id": "<manager_role_id>" },
  "action": "read",
  "effect": "allow"
}
```

### Owner-only access with condition

Allow users to see their own salary only:

```
POST /api/field_permission_definition
{
  "column": { "id": "<salary_column_id>" },
  "action": "read",
  "effect": "allow",
  "condition": { "userId": { "_eq": "@USER.id" } }
}
```

### Block a role from updating a field

```
POST /api/field_permission_definition
{
  "column": { "id": "<status_column_id>" },
  "role": { "id": "<viewer_role_id>" },
  "action": "update",
  "effect": "deny"
}
```

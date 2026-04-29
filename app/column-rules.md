# Column Rules

Column Rules let you attach extra validation constraints (min/max, length, pattern, format, etc.) to a column without writing any code. Rules are enforced server-side every time the API receives a `POST` or `PATCH` against the table — invalid payloads are rejected with HTTP 400 and a list of human-readable error messages.

Rules are configured entirely from the admin UI; there is no manual record creation needed.

## When Rules Run

Rules apply to body validation on the dynamic REST endpoint of the table:

- `POST /<table_name>` (create)
- `PATCH /<table_name>/<id>` (update)

They run **only** when the table has `validateBody = true` (the default for new tables). If `validateBody` is off, rules are skipped — the server only enforces basic shape (type, nullability) at the persistence layer.

Rules do **not** run for:
- `GET` requests
- Custom routes that bypass the standard CRUD pipeline
- Records inserted directly via repository calls inside handlers/hooks (those are server-trusted)

## Toggling `validateBody` on a Table

1. Open **Collections > [your table]** (or any system table you own).
2. Open the table form (gear / edit).
3. Toggle **Validate Body** on or off.
4. Save.

Default for newly-created tables is **on**.

## Adding a Rule to a Column

Rules live next to columns in the table editor.

1. Open the table in **Collections** (or **Settings > Tables** for system tables).
2. Locate the column row in the columns table.
3. Click the **ruler icon** in the column row (next to the field-permission shield icon).
4. The **Manage Rules** modal opens with the existing rule list for that column.
5. Click **Add Rule**.
6. Pick a **Rule Type** from the dropdown (only types that make sense for the column's data type are shown; see the matrix below).
7. Fill the **Value** field — the editor adapts to the rule type (number input, regex + flags, format dropdown).
8. Optionally fill **Message** to override the default error text.
9. Toggle **Enabled** (defaults to on).
10. Click **Save**.

## Rule Type Reference

| Rule Type | Applies To | Value Shape | Effect |
|---|---|---|---|
| `min` | int, bigint, float, decimal | `{ v: number }` | Number must be ≥ v |
| `max` | int, bigint, float, decimal | `{ v: number }` | Number must be ≤ v |
| `minLength` | varchar, text, richtext, code, uuid | `{ v: number }` | String length must be ≥ v |
| `maxLength` | varchar, text, richtext, code, uuid | `{ v: number }` | String length must be ≤ v |
| `pattern` | varchar, text, richtext, code | `{ v: string, flags?: string }` | Must match `RegExp(v, flags)` |
| `format` | varchar, text, code | `{ v: 'email' \| 'url' \| 'uuid' \| 'datetime' }` | Must match the named format |
| `minItems` | array-select | `{ v: number }` | Array length must be ≥ v |
| `maxItems` | array-select | `{ v: number }` | Array length must be ≤ v |
| `custom` | any | (free-form) | Reserved — currently passes through; use a pre-hook for fully custom logic |

The Manage Rules modal hides incompatible rule types per column and prevents adding the same rule type twice (except `custom`).

## Important: Rules Are Additive, Not Replacement

Rules **never** replace the column's built-in checks. The server always enforces (in this order):

1. **Type** (from `column.type` — int, varchar, boolean, etc.)
2. **Nullability** (from `column.isNullable` — null is rejected if `false`)
3. **Length cap** for `varchar` (from `options.length`)
4. **Then your rules**, on top of the above

So you cannot use a rule to *make* a field nullable, *change* its type, or remove the length cap — those are properties of the column itself. Edit the column to change them.

There is no `required` rule type. Required-ness comes from `isNullable = false` on the column, plus whether the column has a default value.

## Disabling vs Deleting a Rule

- **Disable** (toggle off) — keeps the rule record but stops enforcement. Useful for temporary debugging.
- **Delete** (trash icon) — removes the rule completely.

Cache invalidation happens automatically; the new validator schema is rebuilt within ~50 ms across all instances.

## Custom Error Messages

When a rule fails, the response body lists each violation as a string. By default the message is generated from the rule type and value (e.g. `"name: String must contain at least 3 character(s)"`).

Set the **Message** field on the rule to override the text. Plain strings — no template variables.

## Error Response Shape

A failed validation returns:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "statusCode": 400,
  "message": [
    "name: String must contain at least 3 character(s)",
    "email: Invalid email"
  ],
  "error": "Bad Request"
}
```

`message` is always an **array of strings**. Front-end forms can iterate and bind each entry to its corresponding field (the prefix before `:` is the field name).

## Cascade Validation (Nested Create)

When the table accepts nested records (e.g. `POST /post` with an inline `comments: [{ body: '...' }]`), child rules from the related table are validated too — provided the related table also has `validateBody = true`. Connect-by-id payloads (`{ author: 5 }` or `{ author: { id: 5 } }`) skip nested validation since no new record is being created.

## System Tables

Some system tables (`table_definition`, `field_permission_definition`, etc.) accept a small set of virtual fields that aren't real columns (`graphqlEnabled`, `config`). The validator allows those by name; everything else outside the schema is rejected as "not allowed". You don't need to do anything special for system tables — rules work the same way.

## Related

- Server-side: validation is implemented in `buildZodFromMetadata` (`src/shared/utils/zod-from-metadata.ts`). Rules are cached in `ColumnRuleCacheService`.
- Field-level read/write access: see [Field Permissions](../server/field-permissions.md).
- Schema confirm/preview when editing a table: see [Schema migration preview](../server/schema-migration-preview.md).

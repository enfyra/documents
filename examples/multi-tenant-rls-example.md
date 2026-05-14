# Row-Level Security Example

Build a shared table where each user only sees rows from their own tenant.

This example uses one relation, one pre-hook, and normal generated CRUD routes.

## What You Build

```text
user_definition
  -> has tenant

project_task
  -> has tenant
  -> generated /project_task route

GET /api/project_task
  -> pre-hook injects tenant filter
  -> user only sees tenant rows
```

Use this pattern for teams, workspaces, organizations, schools, branches, or customer accounts.

## 1. Create Tables

### tenant

| Column | Type | Notes |
|--------|------|-------|
| name | string | Tenant name |

### project_task

| Column | Type | Notes |
|--------|------|-------|
| title | string | Task title |
| status | string | `todo`, `doing`, `done` |
| tenant | many-to-one | Relation to `tenant` |
| owner | many-to-one | Relation to `user_definition` |

### user_definition

Add a relation from `user_definition.tenant` to `tenant`.

Every user should belong to one tenant.

## 2. Seed Example Data

Create two tenants:

```text
Tenant A
Tenant B
```

Assign users:

```text
mai@example.com  -> Tenant A
long@example.com -> Tenant B
```

Create tasks:

```text
Task A1 -> Tenant A
Task B1 -> Tenant B
```

## 3. Add A Pre-Hook To The Task Route

Attach this pre-hook to the generated `/project_task` route for `GET`, `POST`, `PATCH`, and `DELETE`.

```js
const tenantId = @USER?.tenant?.id
if (!tenantId) @THROW403("User has no tenant")

const method = @API.request.method

if (method === "GET") {
  @QUERY.filter = {
    _and: [
      { tenant: { id: { _eq: tenantId } } },
      @QUERY.filter || {}
    ]
  }
}

if (method === "POST") {
  @BODY.tenant = { id: tenantId }
  if (!@BODY.owner && @USER?.id) {
    @BODY.owner = { id: @USER.id }
  }
}

if (method === "PATCH" || method === "DELETE") {
  @QUERY.filter = {
    _and: [
      { tenant: { id: { _eq: tenantId } } },
      @QUERY.filter || {}
    ]
  }
  delete @BODY?.tenant
}
```

What this does:

- `GET`: adds tenant filter to every list request.
- `POST`: forces the new row into the current user's tenant.
- `PATCH` and `DELETE`: only affect rows in the current user's tenant.
- `tenant` cannot be moved by request body.

## 4. Test The Flow

Login as a Tenant A user and call:

```http
GET /api/project_task
```

Expected result:

```text
Task A1 is returned
Task B1 is hidden
```

Try to create a task while sending another tenant:

```json
{
  "title": "Wrong tenant attempt",
  "tenant": { "id": "tenant-b-id" }
}
```

Expected result:

```text
The hook overwrites tenant with the current user's tenant.
```

Try to update a Tenant B row while logged in as Tenant A:

```http
PATCH /api/project_task?filter={"id":{"_eq":"task-b-id"}}
```

Expected result:

```text
No Tenant B row is updated because the hook adds the Tenant A filter.
```

## 5. Add UI Permissions

Use route permissions and field permissions for the user experience:

- Allow tenant users to read/create/update tasks.
- Hide the `tenant` field in forms if users should not choose it.
- Keep admin-only routes for tenant management.

The pre-hook is still the backend enforcement layer. UI permissions improve the interface but should not be the only protection.

## Common Mistakes

### Trusting a tenant value from the client

Do not accept `tenant` from the request body for normal users. Set it from `@USER`.

### Filtering only on the frontend

Frontend filters are useful for UX, but backend pre-hooks enforce the rule for every API client.

### Forgetting update and delete

RLS must cover writes too. A read-only tenant filter is not enough.

## Related Documentation

- [Hooks](../app/hooks-handlers/hooks.md)
- [API Lifecycle](../server/api-lifecycle.md)
- [Field Permissions](../server/field-permissions.md)
- [Query Filtering](../server/query-filtering.md)

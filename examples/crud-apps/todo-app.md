# Todo App

This example builds a small task app where each user only sees their own tasks.

## Tables

Create `todo_task`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `title` | string | Required |
| `description` | text | Optional |
| `status` | select | `open`, `done`, `archived` |
| `dueAt` | datetime | Optional |
| `owner` | many-to-one relation to `enfyra_user` | No inverse relation required |

Add an index on `owner,status,dueAt` for common list filters.

## Create The Current User's Task

Add a `POST /todo_task` pre-hook so the client cannot assign a task to another user.

```javascript
if (!@USER?.id) {
  @THROW401();
}

@BODY.owner = { id: @USER.id };
if (!@BODY.status) {
  @BODY.status = 'open';
}
```

## Scope Reads

Add a `GET /todo_task` pre-hook.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

if (!@USER?.id) {
  @THROW401();
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { owner: { id: { _eq: @USER.id } } }
  ]
};
```

## List Tasks

```bash
curl "$ENFYRA_API_URL/todo_task?fields=id,title,status,dueAt&sort=dueAt,-createdAt&filter={\"status\":{\"_neq\":\"archived\"}}" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

## Create A Task

```bash
curl "$ENFYRA_API_URL/todo_task" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Ship the invite flow","dueAt":"2026-07-01T09:00:00.000Z"}'
```

## Complete A Task

```bash
curl "$ENFYRA_API_URL/todo_task/42" \
  -X PATCH \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status":"done"}'
```

## Delete A Task

```bash
curl "$ENFYRA_API_URL/todo_task/42" \
  -X DELETE \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

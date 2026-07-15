# Team Workspace RLS

This example models shared workspaces where a user can see records only when they are a member of the workspace.

## Tables

Create `workspace`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `name` | string | Required |
| `owner` | many-to-one relation to `enfyra_user` | Required |

Create `workspace_member`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `workspace` | many-to-one relation to `workspace` | Required |
| `member` | many-to-one relation to `enfyra_user` | Required |
| `role` | select | `owner`, `admin`, `member` |

Create `workspace_task`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `workspace` | many-to-one relation to `workspace` | Required |
| `title` | string | Required |
| `status` | select | `open`, `done` |
| `assignee` | many-to-one relation to `enfyra_user` | Optional |

Add a unique constraint on `workspace_member.workspace,member`.

## Scope Workspace Tasks

Add a `GET /workspace_task` pre-hook.

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
    {
      workspace: {
        members: {
          member: {
            id: { _eq: @USER.id }
          }
        }
      }
    }
  ]
};
```

The relation path must match your inverse relation names. If your metadata does not expose `workspace.members`, query `workspace_member` first and filter tasks by the resulting workspace ids.

## Require Admin Role To Create Tasks

Add a pre-hook to `POST /workspace_task`.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

const workspaceId = @BODY.workspace?.id || @BODY.workspace || @QUERY.filter?.workspace?._eq;
if (!workspaceId) {
  @THROW400('workspace is required');
}

const membership = await #workspace_member.find({
  filter: {
    workspace: { id: { _eq: workspaceId } },
    member: { id: { _eq: @USER.id } },
    role: { _in: ['owner', 'admin'] }
  },
  fields: 'id',
  limit: 1
});

if (!membership.data?.[0]) {
  @THROW403();
}
```

For `PATCH /workspace_task/:id` and `DELETE /workspace_task/:id`, load the target task by `@PARAMS.id`, read its `workspace`, and run the same membership check before allowing the write.

## Create A Workspace

A custom `POST /workspaces` handler can create the workspace and owner membership together.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const created = await #workspace.create({
  data: {
    name: @BODY.name,
    owner: { id: @USER.id }
  }
});

const workspace = created.data?.[0];
if (!workspace) {
  @THROW500('workspace was not created');
}

await #workspace_member.create({
  data: {
    workspace: { id: workspace.id },
    member: { id: @USER.id },
    role: 'owner'
  }
});

return workspace;
```

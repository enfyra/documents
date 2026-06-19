# Activity Feed

This example records project activity and broadcasts updates to viewers of the same project.

## Table

Create `project_activity`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `project` | many-to-one relation to `project` | Required |
| `actor` | many-to-one relation to `enfyra_user` | Required |
| `kind` | string | Required |
| `summary` | string | Required |
| `payload` | simple-json | Optional |

## Post-Hook After A Project Change

Add a post-hook to routes that should create activity rows.

```javascript
if (@ERROR || !@DATA || !@USER?.id) {
  return;
}

const projectId = @DATA.project?.id || @DATA.id;
if (!projectId) {
  return;
}

const created = await #project_activity.create({
  data: {
    project: { id: projectId },
    actor: { id: @USER.id },
    kind: 'project.updated',
    summary: 'Project was updated',
    payload: { recordId: @DATA.id }
  }
});

const activity = created.data?.[0] || null;
if (activity) {
  @SOCKET.emitToRoom('/projects', `project:${projectId}`, 'activity:new', activity);
}
```

## Join A Project Room

Create a Socket.IO gateway `/projects` and an event `project:join`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const projectId = @BODY.project;
const access = await #project_member.find({
  filter: {
    project: { id: { _eq: projectId } },
    member: { id: { _eq: @USER.id } }
  },
  fields: 'id',
  limit: 1
});

if (!access.data?.[0] && !@USER.isRootAdmin) {
  @THROW403();
}

@SOCKET.join(`project:${projectId}`);
return { joined: true };
```

## Load The Feed

```bash
curl "$ENFYRA_API_URL/project_activity?filter={\"project\":{\"id\":{\"_eq\":42}}}&fields=id,kind,summary,actor.email,createdAt&sort=-createdAt&limit=25" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

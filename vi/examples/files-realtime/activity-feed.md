---
slug: vi-du/file-va-realtime/dong-hoat-dong
---

# Dòng hoạt động

Ví dụ này ghi lại hoạt động của project và broadcast cập nhật đến những người đang xem cùng project.

## Table

Tạo `project_activity`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `project` | relation many-to-one tới `project` | Bắt buộc |
| `actor` | relation many-to-one tới `enfyra_user` | Bắt buộc |
| `kind` | string | Bắt buộc |
| `summary` | string | Bắt buộc |
| `payload` | simple-json | Không bắt buộc |

## Post-hook sau khi thay đổi project

Thêm post-hook vào những route cần tạo bản ghi hoạt động.

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

## Tham gia project room

Tạo Socket.IO gateway `/projects` và event `project:join`.

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

## Tải dòng hoạt động

```bash
curl "$ENFYRA_API_URL/project_activity?filter={\"project\":{\"id\":{\"_eq\":42}}}&fields=id,kind,summary,actor.email,createdAt&sort=-createdAt&limit=25" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

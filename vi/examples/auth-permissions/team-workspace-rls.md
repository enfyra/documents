---
slug: vi-du/xac-thuc-va-phan-quyen/rls-workspace-nhom
---

# RLS cho workspace nhóm

Ví dụ này mô hình hóa workspace dùng chung, trong đó người dùng chỉ thấy bản ghi khi là thành viên của workspace đó.

## Table

Tạo `workspace`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `name` | string | Bắt buộc |
| `owner` | relation many-to-one với `enfyra_user` | Bắt buộc |

Tạo `workspace_member`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `workspace` | relation many-to-one với `workspace` | Bắt buộc |
| `member` | relation many-to-one với `enfyra_user` | Bắt buộc |
| `role` | select | `owner`, `admin`, `member` |

Tạo `workspace_task`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `workspace` | relation many-to-one với `workspace` | Bắt buộc |
| `title` | string | Bắt buộc |
| `status` | select | `open`, `done` |
| `assignee` | relation many-to-one với `enfyra_user` | Không bắt buộc |

Thêm unique constraint cho `workspace_member.workspace,member`.

## Giới hạn phạm vi task theo workspace

Thêm pre-hook cho `GET /workspace_task`.

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

Đường dẫn relation phải khớp với inverse relation của bạn. Nếu metadata không có `workspace.members`, hãy query `workspace_member` trước rồi lọc task theo các workspace id nhận được.

## Chỉ cho phép admin tạo task

Thêm pre-hook vào `POST /workspace_task`.

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

Với `PATCH /workspace_task/:id` và `DELETE /workspace_task/:id`, hãy tải task đích qua `@PARAMS.id`, đọc `workspace` của nó và chạy cùng kiểm tra membership trước khi cho phép ghi.

## Tạo workspace

Một custom handler `POST /workspaces` có thể tạo workspace cùng membership của chủ sở hữu.

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

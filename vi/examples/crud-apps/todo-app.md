---
slug: vi-du/ung-dung-crud/ung-dung-cong-viec
---

# Ứng dụng công việc

Ví dụ này xây dựng một ứng dụng công việc nhỏ, trong đó mỗi người dùng chỉ nhìn thấy công việc của chính họ.

## Table

Tạo `todo_task`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `title` | string | Bắt buộc |
| `description` | text | Không bắt buộc |
| `status` | select | `open`, `done`, `archived` |
| `dueAt` | datetime | Không bắt buộc |
| `owner` | relation many-to-one tới `enfyra_user` | Không cần inverse relation |

Thêm index cho `owner,status,dueAt` để tối ưu các filter danh sách thường dùng.

## Tạo công việc cho người dùng hiện tại

Thêm pre-hook cho `POST /todo_task` để client không thể gán công việc cho người dùng khác.

```javascript
if (!@USER?.id) {
  @THROW401();
}

@BODY.owner = { id: @USER.id };
if (!@BODY.status) {
  @BODY.status = 'open';
}
```

## Giới hạn dữ liệu đọc

Thêm pre-hook cho `GET /todo_task`.

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

## Liệt kê công việc

```bash
curl "$ENFYRA_API_URL/todo_task?fields=id,title,status,dueAt&sort=dueAt,-createdAt&filter={\"status\":{\"_neq\":\"archived\"}}" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

## Tạo công việc

```bash
curl "$ENFYRA_API_URL/todo_task" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Ship the invite flow","dueAt":"2026-07-01T09:00:00.000Z"}'
```

## Hoàn thành công việc

```bash
curl "$ENFYRA_API_URL/todo_task/42" \
  -X PATCH \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status":"done"}'
```

## Xóa công việc

```bash
curl "$ENFYRA_API_URL/todo_task/42" \
  -X DELETE \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

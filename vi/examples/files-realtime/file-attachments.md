---
slug: vi-du/file-va-realtime/file-dinh-kem
---

# File đính kèm

Ví dụ này gắn file đã tải lên với công việc, ticket hoặc bình luận.

## Table

Tạo `task_attachment`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `task` | relation many-to-one tới `todo_task` | Bắt buộc |
| `file` | relation many-to-one tới `enfyra_file` | Bắt buộc |
| `uploadedBy` | relation many-to-one tới `enfyra_user` | Bắt buộc |

## Tải file lên

Dùng file route có sẵn cho binary upload.

```bash
curl "$ENFYRA_API_URL/enfyra_file" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -F "file=@./contract.pdf" \
  -F "folder=attachments"
```

Response có id của bản ghi `enfyra_file`.

## Gắn file

Thêm pre-hook cho `POST /task_attachment`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

@BODY.uploadedBy = { id: @USER.id };
```

```bash
curl "$ENFYRA_API_URL/task_attachment" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"task":{"id":42},"file":{"id":318}}'
```

## Liệt kê file đính kèm

```bash
curl "$ENFYRA_API_URL/task_attachment?filter={\"task\":{\"id\":{\"_eq\":42}}}&fields=id,file.id,file.filename,file.url,file.size,uploadedBy.email,createdAt" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Giữ kiểm tra quyền sở hữu ở route của công việc cha hoặc trong hook `task_attachment`, để người dùng không thể gắn file vào công việc họ không được phép truy cập.

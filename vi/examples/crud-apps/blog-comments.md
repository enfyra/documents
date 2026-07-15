---
slug: vi-du/ung-dung-crud/blog-va-binh-luan
---

# Blog có bình luận

Ví dụ này xây dựng luồng đọc bài viết công khai, bình luận của người dùng đã xác thực và đường dẫn kiểm duyệt dành cho quản trị viên.

## Table

Tạo `blog_post`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `title` | string | Bắt buộc |
| `slug` | string | Duy nhất |
| `body` | rich text hoặc text | Bắt buộc |
| `status` | select | `draft`, `published`, `archived` |
| `publishedAt` | datetime | Không bắt buộc |
| `author` | relation many-to-one tới `enfyra_user` | Không cần inverse relation |

Tạo `blog_comment`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `post` | relation many-to-one tới `blog_post` | Bắt buộc |
| `author` | relation many-to-one tới `enfyra_user` | Bắt buộc |
| `body` | text | Bắt buộc |
| `status` | select | `pending`, `approved`, `rejected` |

Thêm index cho `blog_post.status,publishedAt` và `blog_comment.post,status,createdAt`.

## Danh sách bài viết công khai

Cho phép công khai `GET /blog_post`, rồi thêm pre-hook để giới hạn người chưa đăng nhập chỉ đọc được bài đã publish.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { status: { _eq: 'published' } }
  ]
};
```

## Đọc bài viết kèm bình luận đã duyệt

```bash
curl "$ENFYRA_API_URL/blog_post?filter={\"slug\":{\"_eq\":\"hello-enfyra\"}}&limit=1&fields=id,title,body,publishedAt,author.email&deep={\"comments\":{\"fields\":\"id,body,createdAt,author.email\",\"filter\":{\"status\":{\"_eq\":\"approved\"}},\"sort\":\"createdAt\"}}"
```

Relation key bên trong `deep` phải khớp với tên inverse relation đã cấu hình trên `blog_comment.post`. Nếu không tạo inverse relation, hãy tải bình luận bằng một lệnh `GET /blog_comment` riêng, lọc theo `post`.

## Tạo bình luận

Thêm pre-hook cho `POST /blog_comment`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

if (!@BODY.post) {
  @THROW400('post is required');
}

@BODY.author = { id: @USER.id };
@BODY.status = 'pending';
```

Sau đó gọi table route.

```bash
curl "$ENFYRA_API_URL/blog_comment" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"post":{"id":12},"body":"This helped me understand hooks."}'
```

## Kiểm duyệt bình luận

Cấp quyền `PATCH /blog_comment` cho moderator và giới hạn người dùng thông thường ở các bình luận của chính họ bằng PATCH pre-hook.

```javascript
if (@USER?.isRootAdmin || @USER?.role?.name === 'moderator') {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { author: { id: { _eq: @USER.id } } }
  ]
};
```

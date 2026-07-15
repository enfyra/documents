---
slug: vi-du/api
---

# Ví dụ API

Các ví dụ REST ngắn cho table route được Enfyra sinh tự động.

Thay `{appUrl}` bằng URL app của bạn, ví dụ `http://localhost:3000`.

## Xác thực

### Đăng nhập

```bash
curl -X POST "{appUrl}/api/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"password"}'
```

### Người dùng hiện tại

```bash
curl "{appUrl}/api/me" \
  -H "Authorization: Bearer <accessToken>"
```

### Đăng xuất

```bash
curl -X POST "{appUrl}/api/logout" \
  -H "Authorization: Bearer <accessToken>"
```

## Bản ghi

### Liệt kê bản ghi

```http
GET {appUrl}/api/post?fields=id,title,createdAt&limit=20
```

### Tìm một bản ghi theo ID

```http
GET {appUrl}/api/post?filter={"id":{"_eq":123}}&limit=1
```

### Tạo bản ghi

```bash
curl -X POST "{appUrl}/api/post" \
  -H "Authorization: Bearer <accessToken>" \
  -H "Content-Type: application/json" \
  -d '{"title":"Hello","status":"draft"}'
```

### Cập nhật bản ghi

```bash
curl -X PATCH "{appUrl}/api/post/123" \
  -H "Authorization: Bearer <accessToken>" \
  -H "Content-Type: application/json" \
  -d '{"status":"published"}'
```

### Xóa bản ghi

```bash
curl -X DELETE "{appUrl}/api/post/123" \
  -H "Authorization: Bearer <accessToken>"
```

## Query

### Filter bằng

```http
GET {appUrl}/api/post?filter={"status":{"_eq":"published"}}
```

### Filter chứa chuỗi

```http
GET {appUrl}/api/post?filter={"title":{"_contains":"release"}}
```

### Filter relation

```http
GET {appUrl}/api/post?filter={"author":{"id":{"_eq":7}}}
```

### Sắp xếp

```http
GET {appUrl}/api/post?sort=-createdAt
```

### Đếm bản ghi

```http
GET {appUrl}/api/post?fields=id&limit=1&meta=totalCount
```

### Đếm bản ghi sau filter

```http
GET {appUrl}/api/post?fields=id&limit=1&meta=filterCount&filter={"status":{"_eq":"published"}}
```

## Field

### Chọn field

```http
GET {appUrl}/api/post?fields=id,title,author.email
```

### Loại trừ field lớn

```http
GET {appUrl}/api/enfyra_route_handler?fields=-compiledCode
```

### Loại trừ field lồng nhau

```http
GET {appUrl}/api/post?fields=-author.avatar
```

## Relation

### Nạp field liên quan

```http
GET {appUrl}/api/post?fields=id,title,author.id,author.email
```

### Deep load bản ghi con

```http
GET {appUrl}/api/post?fields=id,title&deep={"comments":{"fields":"id,body","limit":5}}
```

### Deep load với loại trừ lồng nhau

```http
GET {appUrl}/api/post?fields=id,title&deep={"comments":{"fields":"-compiledCode,-author.avatar","limit":5}}
```

### Sắp xếp bản ghi cha theo số lượng bản ghi con

```http
GET {appUrl}/api/post?fields=id,title&sort=-_count(comments)
```

### Sắp xếp bản ghi cha theo bản ghi con mới nhất

```http
GET {appUrl}/api/post?fields=id,title&sort=-_max(comments.createdAt)
```

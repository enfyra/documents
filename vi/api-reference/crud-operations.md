---
slug: thao-tac-crud
---

# Thao tác CRUD

Khi tạo một bảng trong Enfyra (ví dụ `products`, `orders`, `customers`), bảng đó tự có các endpoint REST. Dùng chúng để liệt kê, tạo, cập nhật và xóa bản ghi từ ứng dụng của bạn.

**URL gốc:** `{appUrl}/api/{routePath}`

`routePath` khớp tên bảng hoặc custom route của bạn (ví dụ `/products`, `/orders`). Kiểm tra Collections trong Enfyra để biết các route hiện có.

## GET /{routePath}

Liệt kê bản ghi; có thể filter, sort và phân trang.

**URL:** `{appUrl}/api/{routePath}`

**Query parameter:** xem [Query Parameters](./query-parameters.md).

```bash
curl "http://localhost:3000/api/products?limit=10&fields=id,name,price" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [
    { "id": 1, "email": "user@example.com", "name": "User 1" },
    { "id": 2, "email": "user2@example.com", "name": "User 2" }
  ],
  "meta": {
    "totalCount": 50,
    "filterCount": 2
  }
}
```

## Lấy một bản ghi

Dynamic REST route không có `GET /{routePath}/:id`. Hãy dùng list endpoint với filter khóa chính và `limit=1`.

**URL:** `{appUrl}/api/{routePath}?filter={"id":{"_eq":1}}&limit=1`

```bash
curl 'http://localhost:3000/api/enfyra_user?filter={"id":{"_eq":1}}&limit=1' \
  -H "Authorization: Bearer YOUR_TOKEN"
```

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [
    {
      "id": 1,
      "email": "user@example.com",
      "name": "User 1",
      "role": { "id": 1, "name": "Admin" }
    }
  ]
}
```

Đọc phần tử đầu tiên trong `data`. Với MongoDB, dùng `_id` thay cho `id` nếu schema của bạn dùng trường này.

## POST /{routePath}

Tạo bản ghi mới. Gửi JSON object có các field của bảng.

```bash
curl -X POST "http://localhost:3000/api/enfyra_user" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email":"new@example.com","name":"New User","password":"secret123"}'
```

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [
    {
      "id": 3,
      "email": "new@example.com",
      "name": "New User",
      "createdAt": "2024-01-15T12:00:00.000Z"
    }
  ]
}
```

Response mặc định trả object kết quả có bản ghi đã tạo trong `data`.

## PATCH /{routePath}/:id

Cập nhật một phần bản ghi hiện có.

```bash
curl -X PATCH "http://localhost:3000/api/enfyra_user/3" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Name"}'
```

Default update handler trả kết quả dạng collection với bản ghi được cập nhật ở `data[0]`.

## DELETE /{routePath}/:id

Xóa bản ghi.

```bash
curl -X DELETE "http://localhost:3000/api/enfyra_user/3" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Default delete handler trả `{ "message": "Success", "statusCode": 200 }`.

## Kiểm tra body (POST / PATCH)

Nếu bảng có `validateBody = true` (mặc định cho bảng mới), server kiểm tra request body theo schema của bảng và **column rules** gắn với các cột trước khi handler chạy.

Thứ tự kiểm tra:

1. **Kiểu cột** (`int`, `varchar`, `boolean`, ...)
2. **Cho phép null** (`isNullable: false` từ chối `null`)
3. **Giới hạn độ dài** của `varchar` (`options.length`)
4. **Column rules** (min/max, minLength/maxLength, pattern, format, minItems/maxItems)
5. **Strictness** — từ chối top-level field không rõ

Response lỗi (HTTP 400):

```json
{
  "statusCode": 400,
  "message": [
    "name: String must contain at least 3 character(s)",
    "email: Invalid email",
    "age: Number must be greater than or equal to 18"
  ],
  "error": "Bad Request"
}
```

`message` luôn là **mảng chuỗi**, mỗi chuỗi ứng với một vi phạm. Phần trước dấu `:` là tên field; client có thể tách theo `:` để ánh xạ lỗi về trường của form.

- Cascade create (ví dụ `POST /post` kèm `comments: [...]`) cũng kiểm tra child record nếu bảng liên quan có `validateBody = true`.
- Dạng connect-by-id (`{ author: 5 }` hoặc `{ author: { id: 5 } }`) bỏ qua nested validation.
- Bảng có `validateBody = false` chỉ áp dụng constraint ở database khi insert/update; rules không được dùng.
- Field permission `deny` cho `create`/`update` trả **403**, không phải 400 — đó không phải lỗi validation.

Để cấu hình rule cho từng cột, xem [Column Rules](../app/column-rules.md).

## Custom route

Bạn có thể thêm custom route (ví dụ `/register`, `/orders/:orderId/items`) trong Enfyra Settings. Mỗi route cung cấp các HTTP method đã cấu hình và dùng cùng base URL, cơ chế xác thực như table route.

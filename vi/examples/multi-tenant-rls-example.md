---
slug: vi-du-rls-da-tenant
---

# Ví dụ row-level security

Xây dựng một bảng dùng chung, trong đó mỗi người dùng chỉ thấy các dòng thuộc tenant của mình.

Ví dụ này dùng một quan hệ, một pre-hook và các CRUD route được sinh bình thường.

## Nội dung sẽ xây dựng

```text
enfyra_user
  -> có tenant

project_task
  -> có tenant
  -> route /project_task được tạo tự động

GET /api/project_task
  -> pre-hook thêm điều kiện lọc tenant
  -> người dùng chỉ thấy dòng thuộc tenant của mình
```

Dùng mẫu này cho team, workspace, tổ chức, trường học, chi nhánh hoặc tài khoản khách hàng.

## 1. Tạo bảng

### tenant

| Cột | Kiểu | Ghi chú |
|--------|------|-------|
| name | string | Tên tenant |

### project_task

| Cột | Kiểu | Ghi chú |
|--------|------|-------|
| title | string | Tiêu đề công việc |
| status | string | `todo`, `doing`, `done` |
| tenant | many-to-one | Quan hệ tới `tenant` |
| owner | many-to-one | Quan hệ tới `enfyra_user` |

### enfyra_user

Thêm quan hệ từ `enfyra_user.tenant` tới `tenant`.

Mọi người dùng nên thuộc đúng một tenant.

## 2. Seed dữ liệu ví dụ

Tạo hai tenant:

```text
Tenant A
Tenant B
```

Gán người dùng:

```text
mai@example.com  -> Tenant A
long@example.com -> Tenant B
```

Tạo công việc:

```text
Task A1 -> Tenant A
Task B1 -> Tenant B
```

## 3. Thêm pre-hook vào task route

Gắn pre-hook này vào route `/project_task` được sinh cho `GET`, `POST`, `PATCH` và `DELETE`.

```js
const tenantId = @USER?.tenant?.id
if (!tenantId) @THROW403("User has no tenant")

const method = @API.request.method

if (method === "GET") {
  @QUERY.filter = {
    _and: [
      { tenant: { id: { _eq: tenantId } } },
      @QUERY.filter || {}
    ]
  }
}

if (method === "POST") {
  @BODY.tenant = { id: tenantId }
  if (!@BODY.owner && @USER?.id) {
    @BODY.owner = { id: @USER.id }
  }
}

if (method === "PATCH" || method === "DELETE") {
  @QUERY.filter = {
    _and: [
      { tenant: { id: { _eq: tenantId } } },
      @QUERY.filter || {}
    ]
  }
  delete @BODY?.tenant
}
```

Ý nghĩa của đoạn này:

- `GET`: thêm tenant filter vào mọi request danh sách.
- `POST`: bắt buộc dòng mới thuộc tenant của người dùng hiện tại.
- `PATCH` và `DELETE`: chỉ tác động các dòng thuộc tenant của người dùng hiện tại.
- Không thể đổi `tenant` qua request body.

## 4. Kiểm thử luồng

Đăng nhập bằng người dùng Tenant A và gọi:

```http
GET /api/project_task
```

Kết quả mong đợi:

```text
Trả về Task A1
Ẩn Task B1
```

Thử tạo một công việc nhưng gửi tenant khác:

```json
{
  "title": "Wrong tenant attempt",
  "tenant": { "id": "tenant-b-id" }
}
```

Kết quả mong đợi:

```text
Hook này ghi đè tenant bằng tenant của người dùng hiện tại.
```

Thử cập nhật dòng của Tenant B khi đang đăng nhập bằng Tenant A:

```http
PATCH /api/project_task?filter={"id":{"_eq":"task-b-id"}}
```

Kết quả mong đợi:

```text
Không có bản ghi Tenant B nào được cập nhật vì hook đã thêm điều kiện lọc Tenant A.
```

## 5. Thêm quyền trên UI

Dùng route permission và field permission để cải thiện trải nghiệm người dùng:

- Cho phép người dùng tenant đọc/tạo/cập nhật công việc.
- Ẩn trường `tenant` trong form nếu người dùng không nên chọn trường này.
- Duy trì route chỉ dành cho admin để quản lý tenant.

Pre-hook vẫn là lớp thực thi ở backend. Quyền UI cải thiện giao diện nhưng không nên là biện pháp bảo vệ duy nhất.

## Lỗi thường gặp

### Tin vào giá trị tenant từ client

Không nhận `tenant` từ request body của người dùng thông thường. Hãy đặt từ `@USER`.

### Chỉ lọc ở frontend

Filter ở frontend hữu ích cho UX, nhưng pre-hook ở backend thực thi quy tắc với mọi API client.

### Quên update và delete

RLS phải bao phủ cả thao tác ghi. Tenant filter chỉ cho đọc là chưa đủ.

## Tài liệu liên quan

- [Hooks](../app/hooks-handlers/hooks.md)
- [API Lifecycle](../server/api-lifecycle.md)
- [Field Permissions](../server/field-permissions.md)
- [Query Filtering](../server/query-filtering.md)

---
slug: ung-dung-dau-tien
---

# Xây dựng ứng dụng đầu tiên

Hướng dẫn này cho bạn một kết quả hoàn chỉnh trước khi tìm hiểu mọi tính năng Enfyra: collection `task`, một bản ghi trong admin app và REST API được tạo tự động để bạn gọi.

**Thời gian:** khoảng 15 phút.  
**Cần có:** một Enfyra app đang chạy và tài khoản admin. Nếu chưa có, xem [Cài đặt](./installation.md).

## Bạn sẽ xây dựng gì

```text
task collection
  ├── title: text
  ├── done: true/false
  └── generated REST API at /api/task
```

Kết thúc hướng dẫn, bạn thấy task trong **Data > task** và nhận được nó từ `GET /api/task`.

## 1. Đăng nhập

1. Mở URL app, thường là `http://localhost:3000`.
2. Đăng nhập bằng thông tin admin đã đặt khi cài đặt.
3. Xác nhận sidebar có **Collections** và **Data**.

Nếu chỉ dùng Docker quick-start mặc định để đánh giá cục bộ, hãy thay thông tin mặc định trước khi triển khai instance ra nơi công khai.

## 2. Tạo collection

1. Vào **Collections** và chọn **Create**.
2. Đặt tên collection là `task`.
3. Thêm cột `title`:
   - Type: `varchar`
   - Nullable: off
4. Thêm cột `done`:
   - Type: `boolean`
   - Default value: `false`
5. Lưu collection.

Enfyra tự thêm primary key. Sau khi lưu, hệ thống tạo physical schema và REST route. REST sẵn sàng ngay; GraphQL là opt-in riêng cho từng bảng.

**Kiểm tra:** `task` xuất hiện dưới **Data** trong sidebar. Nếu chưa thấy, refresh một lần và kiểm tra collection đã lưu không lỗi validation.

Xem [Tạo bảng](./table-creation.md) để biết đầy đủ kiểu cột, relation, index hoặc delete behavior.

## 3. Thêm bản ghi trong admin app

1. Mở **Data > task**.
2. Chọn **Create**.
3. Nhập `Plan the first release` vào `title`, để `done` chưa chọn.
4. Chọn **Save**.

**Kiểm tra:** bản ghi mới xuất hiện trong danh sách task. Bạn đã có collection và dữ liệu được quản lý qua Enfyra.

## 4. Kiểm tra API được tạo

Với browser app, dùng URL app và tiền tố `/api`. App proxy request đến Enfyra server, nên thông thường không cần public hoặc gọi trực tiếp cổng `1105`.

Mở URL này khi vẫn đăng nhập:

```text
http://localhost:3000/api/task?fields=id,title,done&limit=10
```

Bạn sẽ nhận JSON có `data` chứa `Plan the first release`.

Để kiểm tra từ client tự quản lý token, xem luồng login và bearer token tại [API Reference](../api-reference/README.md). Hợp đồng CRUD được tạo là:

```text
GET    /api/task                 list tasks
POST   /api/task                 create a task
PATCH  /api/task/:id             update a task
DELETE /api/task/:id             delete a task
```

Để lấy một bản ghi, dùng `GET /api/task?filter={"id":{"_eq":1}}&limit=1`; dynamic table route không có `GET /api/task/:id`.

## 5. Chọn bước tiếp theo

- Cần relation, validation, encryption, index hoặc GraphQL? Xem [Tạo bảng](./table-creation.md).
- Cần quản lý bản ghi và filter trong admin app? Xem [Quản lý dữ liệu](./data-management.md).
- Cần giới hạn người được gọi route? Xem [Routing Management](../app/routing-management.md) và [Permission Builder](../app/permission-builder.md).
- Cần kết nối frontend, mobile app hoặc server khác? Xem [API Reference](../api-reference/README.md).
- Cần business rule hoặc automation? Xem [Hooks and Handlers](../server/hooks-handlers/README.md) hoặc [Flows](../server/flows.md).

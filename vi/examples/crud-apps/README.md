---
slug: vi-du/ung-dung-crud
---

# Ứng dụng CRUD

Dùng các ví dụ này cho ứng dụng vận hành dựa trên dữ liệu: danh sách công việc, luồng xuất bản, catalog và quản lý đơn hàng.

Mỗi công thức bắt đầu từ table, sau đó mô tả các lệnh REST hoặc GraphQL mà frontend thường cần. Dùng tên property của relation trong request body và filter. Khi đã có relation thật, không tạo thêm scalar foreign key trùng lặp như `userId`.

## Công thức triển khai

- [Ứng dụng Todo](./todo-app.md) — Công việc theo chủ sở hữu với liệt kê, tạo, cập nhật, hoàn thành và xóa.
- [Blog có comment](./blog-comments.md) — Bài viết đã publish, comment lồng nhau, trạng thái moderation và public read.
- [Catalog và đơn hàng](./catalog-orders.md) — Sản phẩm, đơn hàng, item đơn hàng, kiểm tra tồn kho và truy vấn an toàn theo chủ sở hữu.
- [GraphQL Read API](./graphql-read-api.md) — Bật GraphQL cho các table được chọn và query bản ghi liên quan.

## Khi nào dùng nhóm này

Dùng Ứng dụng CRUD khi sản phẩm chủ yếu là bản ghi database kèm truy vấn có nhận biết relation. Chuyển sang [Xác thực và phân quyền](../auth-permissions/README.md) khi phần khó là kiểm soát truy cập, hoặc [Tự động hóa và tích hợp](../automation-integrations/README.md) khi tính năng phụ thuộc hệ thống bên ngoài.

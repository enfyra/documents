---
slug: tham-chieu-context
---

# Tham chiếu Context

Đối tượng `$ctx` (context) có mặt trong mọi hook và handler. Nó cho phép truy cập dữ liệu request, repository cơ sở dữ liệu, hàm hỗ trợ, thao tác cache và nhiều thành phần khác.

## Điều hướng nhanh

- [Dữ liệu request](./request-data.md) — `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- [Repository](./repositories.md) — `$ctx.$repos` cho các thao tác cơ sở dữ liệu
- [Hàm hỗ trợ và cache](./helpers-cache.md) — `$ctx.$helpers` (JWT, bcrypt, crypto, hỗ trợ tệp, giới hạn tần suất) và `$ctx.$cache`
- [Ghi log và xử lý lỗi](./logging-errors.md) — `$ctx.$logs()` và `$ctx.$throw`
- [Tính năng nâng cao](./advanced.md) — tải tệp lên, `$ctx.$env`, thông tin API, context dùng chung, package và các mẫu triển khai

## Nội dung tham khảo

- **[Dữ liệu request](./request-data.md)** — Truy cập thông tin và tham số request HTTP
- **[Repository](./repositories.md)** — Thao tác cơ sở dữ liệu qua repository
- **[Hàm hỗ trợ và cache](./helpers-cache.md)** — Hàm tiện ích (JWT, bcrypt, crypto, giới hạn tần suất) và cache Redis
- **[Ghi log và xử lý lỗi](./logging-errors.md)** — Ghi log và ném lỗi
- **[Tính năng nâng cao](./advanced.md)** — Tải tệp, môi trường đã lọc, thông tin API, context dùng chung và hơn thế nữa

## Tiếp theo

- Xem [Phương thức Repository](../repository-methods/) để thao tác cơ sở dữ liệu
- Xem [Vòng đời API](../api-lifecycle.md) để biết khi nào context khả dụng
- Tìm hiểu [Hook và Handler](../hooks-handlers/) để dùng context trong mã tùy chỉnh

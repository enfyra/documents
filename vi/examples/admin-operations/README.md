---
slug: vi-du/van-hanh-admin
---

# Vận hành Admin

Dùng các ví dụ này khi người vận hành cần một màn hình chuyên biệt thay vì làm việc trực tiếp với bảng dữ liệu thô.

Dynamic admin extension là Vue single-file component được lưu trong `enfyra_extension`. Page extension nên đăng ký page header cấp ứng dụng, dùng quyền cho các thao tác nhạy cảm và liên kết phần kiểm tra bản ghi thô đến `/data/<table_name>`.

## Công thức triển khai

- [Moderation Console](./moderation-console.md) — Rà soát comment đang chờ trong một trang riêng cho người vận hành.
- [Operator Queue](./operator-queue.md) — Tạo hàng đợi công việc có filter, phân trang và thao tác an toàn.
- [Settings Page](./settings-page.md) — Quản lý một table cài đặt một bản ghi bằng `FormEditor`.

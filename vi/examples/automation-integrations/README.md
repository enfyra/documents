---
slug: vi-du/tu-dong-hoa-va-tich-hop
---

# Tự động hóa và tích hợp

Dùng các ví dụ này khi Enfyra cần trao đổi với hệ thống khác, chạy công việc theo lịch hoặc bảo vệ một API tùy biến.

## Công thức triển khai

- [Webhook Ingest](./webhook-ingest.md) — Xác thực và lưu event bên ngoài đã ký theo cách idempotent.
- [Dọn dẹp theo lịch](./scheduled-cleanup.md) — Chạy flow hằng ngày để lưu trữ các bản ghi đã cũ.
- [Public API có rate limit](./rate-limited-public-api.md) — Bảo vệ endpoint cho người dùng ẩn danh hoặc đối tác.
- [Đồng bộ ra ngoài](./outbound-sync.md) — Gửi bản ghi đã thay đổi sang API khác sau khi ghi dữ liệu.

## Lưu ý thiết kế

Giữ credential bên ngoài trong field mã hóa chưa publish, thường nằm trên một table cài đặt một bản ghi. Không đọc secret từ environment variable bên trong dynamic script.

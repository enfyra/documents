---
slug: vi-du/file-va-realtime
---

# File và Realtime

Dùng các ví dụ này cho upload, file đính kèm, thông báo và cập nhật trực tiếp.

File được quản lý qua `enfyra_file` và storage helper. Realtime dùng Socket.IO gateway và event script. Ứng dụng trên trình duyệt nên kết nối qua Socket.IO bridge cùng origin của app thay vì gọi một backend host ẩn.

## Công thức triển khai

- [File đính kèm](./file-attachments.md) — Gắn file đã upload vào bản ghi nghiệp vụ.
- [Tải avatar](./avatar-upload.md) — Lưu avatar của người dùng và mở một profile read an toàn.
- [Thông báo realtime](./realtime-notifications.md) — Lưu thông báo và gửi chúng đến người dùng hiện tại.
- [Dòng hoạt động](./activity-feed.md) — Broadcast hoạt động của project đến một room sau khi ghi dữ liệu.

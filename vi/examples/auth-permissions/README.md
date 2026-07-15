---
slug: vi-du/xac-thuc-va-phan-quyen
---

# Xác thực và phân quyền

Dùng các ví dụ này khi câu hỏi chính là ai có thể đọc hoặc thay đổi một bản ghi.

Enfyra dùng `enfyra_user` làm table người dùng tích hợp sẵn. Hãy relation trực tiếp bản ghi ứng dụng với `enfyra_user`, sau đó áp dụng quyền sở hữu bằng route permission, field permission, pre-hook và handler.

## Công thức triển khai

- [Phạm vi chủ sở hữu profile](./profile-owner-scope.md) — Cho người dùng đọc và cập nhật field profile của chính họ.
- [RLS cho workspace nhóm](./team-workspace-rls.md) — Chia sẻ bản ghi trong workspace nhưng vẫn cô lập các nhóm khác.
- [Form liên hệ công khai](./public-contact-form.md) — Nhận nội dung gửi ẩn danh nhưng chặn việc đọc công khai.

## Nguyên tắc thực hành

Dùng UI permission để giao diện rõ ràng, nhưng luôn thực thi cùng quy tắc trên server. Một nút bị ẩn không phải là ranh giới kiểm soát truy cập.

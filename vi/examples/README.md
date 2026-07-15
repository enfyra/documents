---
slug: vi-du
---

# Ví dụ

Dùng các ví dụ này khi bạn cần snippet ngắn có thể dùng ngay, công thức ứng dụng phổ biến hoặc hướng dẫn hoàn chỉnh cho một tính năng.

Bắt đầu với ví dụ nhỏ khi cần kiểm tra cú pháp nhanh. Dùng công thức theo nhóm khi bạn đang xây một tính năng sản phẩm thông thường. Dùng hướng dẫn đầy đủ khi muốn xem table, route, hook, handler, permission, integration và phần giao diện App được ghép lại như thế nào.

Để thiết lập framework, hãy đọc [Tích hợp](../integrations/README.md) trước.

## Nhóm công thức

Các folder này gom những mẫu sản phẩm phổ biến để menu tài liệu hiển thị rõ từng trường hợp sử dụng.

- [Ứng dụng CRUD](./crud-apps/README.md) — Danh sách việc cần làm, blog có comment, catalog, đơn hàng và GraphQL read tùy chọn.
- [Xác thực và phân quyền](./auth-permissions/README.md) — Profile, dữ liệu theo chủ sở hữu, workspace nhóm và form nhận dữ liệu công khai.
- [File và Realtime](./files-realtime/README.md) — File đính kèm, upload avatar, thông báo người dùng và activity feed realtime.
- [Tự động hóa và tích hợp](./automation-integrations/README.md) — Webhook, dọn dẹp theo lịch, public API có rate limit và đồng bộ ra ngoài.
- [Vận hành Admin](./admin-operations/README.md) — Moderation back-office, hàng đợi cho người vận hành và trang Admin Console động.

## Ví dụ nhỏ

Các trang này được tổ chức theo nhóm và chứa nhiều ví dụ ngắn.

- [Ví dụ API](./api-examples.md) — Đăng nhập, CRUD, filter, field, count, relation, deep query và aggregate sort.
- [Ví dụ Script](./script-examples.md) — Đọc/ghi repository, lỗi, hook, flow, WebSocket script và cache.
- [Ví dụ App](./app-examples.md) — Nạp dữ liệu extension, page shell API, permission, form, SSR auth và realtime.

## Hướng dẫn tính năng hoàn chỉnh

Những ví dụ này dài hơn và cho thấy nhiều phần của Enfyra hoạt động cùng nhau.

- [Ví dụ đăng ký người dùng](./user-registration-example.md) — Tạo endpoint đăng ký tùy biến với validation, hash mật khẩu, kiểm tra trùng lặp và xử lý response.
- [Ví dụ Row-Level Security](./multi-tenant-rls-example.md) — Xây ứng dụng multi-tenant nơi các nhóm dùng chung table nhưng chỉ thấy dữ liệu của mình.
- [Ví dụ ứng dụng chat bên thứ ba](./third-party-chat-app.md) — Xây ứng dụng chat SSR dùng Enfyra auth, REST và Socket.IO qua same-origin proxy.

## Cách dùng ví dụ

1. Chọn ví dụ nhỏ nhất phù hợp với việc bạn cần làm.
2. Sao chép snippet.
3. Thay tên table, field, route và id bằng metadata của bạn.
4. Kiểm tra API, script, extension hoặc luồng App.
5. Chỉ chuyển sang hướng dẫn đầy đủ khi tính năng cần nhiều phần của Enfyra.

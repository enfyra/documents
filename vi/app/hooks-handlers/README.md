---
slug: ung-dung/hook-va-handler
---

# Hook và Handler (giao diện App)

Phần này bao quát mọi cấu hình trong Admin có ảnh hưởng đến **hook và handler phía server**:

- **[Hook](./hooks.md)** — giao diện Hooks System (`preHooks` và `postHooks`)
- **[Custom Handler](./custom-handlers.md)** — giao diện Custom Handlers để thay CRUD mặc định
- **[Quản lý package](./package-management.md)** — cài NPM package để dùng trong hook và handler

Từ góc nhìn của App, bạn dùng công cụ trực quan để cấu hình hành vi; phần **thực thi thực tế** diễn ra trên Enfyra Server.

Để xem hành vi và API đầy đủ ở phía server, xem thêm:

- **[Hook và Handler phía Server](../../server/hooks-handlers/README.md)** — tổng quan hook và handler server
- **[Vòng đời API](../../server/api-lifecycle.md)** — vòng đời request đầy đủ và cách chia sẻ context
- **[Tài liệu Context](../../server/context-reference/README.md)** — tham chiếu đối tượng `$ctx`

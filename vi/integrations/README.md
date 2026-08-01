---
slug: tich-hop
---

# Tích hợp

Dùng các hướng dẫn này khi bạn muốn kết nối một ứng dụng bên ngoài với Enfyra.

## Bắt đầu từ SDK nếu framework đã được hỗ trợ

Enfyra SDK đã hỗ trợ Nuxt, Next.js, Vue (CSR), React (CSR) và Node.js/edge. Nếu app của bạn thuộc nhóm này, hãy bắt đầu từ [tài liệu SDK](../sdk/README.md). Adapter Nuxt và Next.js quản lý same-origin proxy cùng SSR session integration; package Vue và React cung cấp client API nhưng vẫn cần proxy theo hướng dẫn của từng package.

Phần tích hợp bên dưới dành cho:

- Framework **chưa có SDK** (Angular, SvelteKit, Remix) — cần tự cấu hình proxy, cookie bridge và Socket.IO.
- Trường hợp cần hiểu cơ chế proxy/auth mà SDK đang trừu tượng hóa.

Khi kết nối đã xong, hãy dùng [ví dụ](../examples/README.md) để xây dựng tính năng thực tế.

## Hướng dẫn hiện có

- [SSR Frameworks](./ssr-frameworks.md) — Kết nối Angular, SvelteKit hoặc Remix với Enfyra bằng proxy thủ công. Nuxt và Next.js đã có SDK — xem [SDK](../sdk/README.md).
- [MCP Server](./mcp-server.md) — Kết nối Codex, Claude Code, Cursor, VS Code / GitHub Copilot, Google Antigravity hoặc một MCP host khác với instance Enfyra.

## Thứ tự nên đọc

1. Đọc [Tổng quan kiến trúc](../architecture-overview.md) để hiểu app proxy và server runtime.
2. Nếu framework đã có SDK, dùng [SDK](../sdk/README.md). Nếu chưa, cấu hình proxy thủ công theo [SSR Frameworks](./ssr-frameworks.md).
3. Cấu hình coding agent theo [MCP Server](./mcp-server.md) khi cần agent hỗ trợ làm schema, route, script, flow, WebSocket hoặc extension.
4. Xây dựng một tính năng theo [Ví dụ](../examples/README.md).

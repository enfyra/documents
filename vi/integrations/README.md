---
slug: tich-hop
---

# Tích hợp

Dùng các hướng dẫn này khi bạn muốn kết nối một ứng dụng bên ngoài với Enfyra.

Tài liệu tích hợp tập trung vào cách thiết lập framework: quy tắc proxy, auth cookie, luồng OAuth redirect, refresh và realtime transport. Khi kết nối đã xong, hãy dùng các ví dụ để xây dựng tính năng thực tế.

## Hướng dẫn hiện có

- [SSR Frameworks](./ssr-frameworks.md) — Kết nối Nuxt, Next.js, Angular, SvelteKit hoặc Remix với Enfyra bằng REST cùng origin, OAuth cookie và Socket.IO.
- [MCP Server](./mcp-server.md) — Kết nối Codex, Claude Code, Cursor, VS Code / GitHub Copilot, Google Antigravity hoặc một MCP host khác với instance Enfyra.

## Thứ tự nên đọc

1. Đọc [Tổng quan kiến trúc](../architecture-overview.md) để hiểu app proxy và server runtime.
2. Cấu hình framework của bạn theo [SSR Frameworks](./ssr-frameworks.md).
3. Cấu hình coding agent theo [MCP Server](./mcp-server.md) khi cần agent hỗ trợ làm schema, route, script, flow, WebSocket hoặc extension.
4. Xây dựng một tính năng theo [Ví dụ](../examples/README.md).

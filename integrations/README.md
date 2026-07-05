# Integrations

Use these guides when you want to connect an external app to Enfyra.

Integration guides focus on framework setup: proxy rules, auth cookies, OAuth redirect flow, refresh behavior, and realtime transport. After the integration is in place, use the examples to build real features.

## Available Guides

- [SSR Frameworks](./ssr-frameworks.md) - Connect Nuxt, Next.js, Angular, SvelteKit, or Remix to Enfyra with same-origin REST, OAuth cookies, and Socket.IO.
- [MCP Server](./mcp-server.md) - Connect Codex, Claude Code, Cursor, VS Code / GitHub Copilot, Google Antigravity, or another MCP host to an Enfyra instance.

## Recommended Reading Order

1. Read [Architecture Overview](../architecture-overview.md) to understand the app proxy and server runtime.
2. Configure your framework with [SSR Frameworks](./ssr-frameworks.md).
3. Configure an MCP coding agent with [MCP Server](./mcp-server.md) when you want agent-assisted schema, route, script, flow, websocket, or extension work.
4. Build a feature with [Examples](../examples/README.md).

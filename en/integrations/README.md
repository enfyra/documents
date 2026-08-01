# Integrations

Use these guides when you want to connect an external app to Enfyra.

## Start with the SDK if your framework is supported

Enfyra SDK supports Nuxt, Next.js, Vue (CSR), React (CSR), and Node.js/edge. If your app falls into one of these categories, start with the [SDK documentation](../sdk/README.md). The Nuxt and Next.js adapters own their same-origin proxy and SSR session integration; the Vue and React packages provide the client APIs but still require the proxy described in their guides.

The integration guides below are for:

- Frameworks **without an SDK** (Angular, SvelteKit, Remix) — requiring manual proxy, cookie bridge, and Socket.IO setup.
- Understanding the proxy/auth mechanics that the SDK abstracts away.

After the connection is in place, use [examples](../examples/README.md) to build real features.

## Available Guides

- [SSR Frameworks](./ssr-frameworks.md) - Connect Angular, SvelteKit, or Remix to Enfyra with manual proxy setup. Nuxt and Next.js have SDK support — see [SDK](../sdk/README.md).
- [MCP Server](./mcp-server.md) - Connect Codex, Claude Code, Cursor, VS Code / GitHub Copilot, Google Antigravity, or another MCP host to an Enfyra instance.

## Recommended Reading Order

1. Read [Architecture Overview](../architecture-overview.md) to understand the app proxy and server runtime.
2. If your framework has an SDK, use the [SDK](../sdk/README.md). Otherwise, configure a manual proxy with [SSR Frameworks](./ssr-frameworks.md).
3. Configure an MCP coding agent with [MCP Server](./mcp-server.md) when you want agent-assisted schema, route, script, flow, websocket, or extension work.
4. Build a feature with [Examples](../examples/README.md).

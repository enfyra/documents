# Examples

Use these examples when you want to see a complete Enfyra feature built from tables, routes, hooks, handlers, permissions, integrations, and app UI pieces.

Each example focuses on a practical workflow instead of isolated API calls. For framework setup, read [Integrations](../integrations/README.md) first.

## Available Examples

- [User Registration Example](./user-registration-example.md) - Create a custom registration endpoint with validation, password hashing, duplicate checks, and response handling.
- [Row-Level Security Example](./multi-tenant-rls-example.md) - Build a multi-tenant application where teams share tables but only see their own data.
- [Third-Party Chat App Example](./third-party-chat-app.md) - Build an SSR chat app using Enfyra auth, REST, and Socket.IO through a same-origin proxy.

## How To Use Examples

1. Read the goal and table setup.
2. Create the required tables and relations in the admin app.
3. Add the route, hook, handler, permission, integration, or extension records described in the example.
4. Test the flow from the app or API client.
5. Reuse the same pattern in your own project.

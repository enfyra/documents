# Examples

Use these examples when you want short copy-ready snippets, common app recipes, or complete feature walkthroughs.

Start with the small examples when you need syntax quickly. Use category recipes when you are building a normal product feature. Use walkthroughs when you want to see a full feature assembled from tables, routes, hooks, handlers, permissions, integrations, and app UI pieces.

For framework setup, read [Integrations](../integrations/README.md) first.

## Recipe Categories

These folders group common product patterns so the docs navigation can show each use case clearly.

- [CRUD Apps](./crud-apps/README.md) - Todo lists, blogs with comments, catalogs, orders, and optional GraphQL reads.
- [Auth & Permissions](./auth-permissions/README.md) - Profiles, owner-scoped data, team workspaces, and public intake forms.
- [Files & Realtime](./files-realtime/README.md) - Attachments, avatar uploads, user notifications, and realtime activity feeds.
- [Automation & Integrations](./automation-integrations/README.md) - Webhooks, scheduled cleanup, rate-limited public APIs, and outbound sync.
- [Admin Operations](./admin-operations/README.md) - Back-office moderation, operator queues, and dynamic admin console pages.

## Small Examples

These pages are organized by category and contain many small examples.

- [API Examples](./api-examples.md) - Login, CRUD, filters, fields, counts, relations, deep queries, and aggregate sorts.
- [Script Examples](./script-examples.md) - Repository reads/writes, errors, hooks, flows, websocket scripts, and cache.
- [App Examples](./app-examples.md) - Extension data loading, page shell APIs, permissions, forms, SSR auth, and realtime.

## Feature Walkthroughs

These examples are longer and show how several Enfyra pieces fit together.

- [User Registration Example](./user-registration-example.md) - Create a custom registration endpoint with validation, password hashing, duplicate checks, and response handling.
- [Row-Level Security Example](./multi-tenant-rls-example.md) - Build a multi-tenant application where teams share tables but only see their own data.
- [Third-Party Chat App Example](./third-party-chat-app.md) - Build an SSR chat app using Enfyra auth, REST, and Socket.IO through a same-origin proxy.

## How To Use Examples

1. Pick the smallest example that matches what you need.
2. Copy the snippet.
3. Replace table names, field names, route paths, and ids with your own metadata.
4. Test the API, script, extension, or app flow.
5. Move to a walkthrough only when the feature needs multiple Enfyra parts.

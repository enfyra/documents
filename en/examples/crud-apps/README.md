# CRUD Apps

Use these examples for normal data-driven applications: task lists, publishing flows, catalogs, and order management.

Each recipe starts from tables, then shows the REST or GraphQL calls a frontend usually needs. Use relation property names in request bodies and filters. Do not create duplicate scalar foreign-key fields such as `userId` when a real relation is available.

## Recipes

- [Todo App](./todo-app.md) - Owner-scoped tasks with list, create, update, complete, and delete.
- [Blog With Comments](./blog-comments.md) - Published posts, nested comments, moderation status, and public reads.
- [Catalog And Orders](./catalog-orders.md) - Products, orders, order items, inventory checks, and owner-safe reads.
- [GraphQL Read API](./graphql-read-api.md) - Enable GraphQL for selected tables and query related records.

## When To Use This Category

Use CRUD Apps when the product is mostly database records plus relation-aware reads. Move to [Auth & Permissions](../auth-permissions/README.md) when the hard part is access control, or to [Automation & Integrations](../automation-integrations/README.md) when the feature depends on external systems.

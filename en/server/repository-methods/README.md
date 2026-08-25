# Repository Methods

Repositories are the main way to interact with your database tables in Enfyra. Every table you create automatically gets a repository that you can access through `$ctx.$repos.tableName`.

## Quick Reference

**Record methods return data in this format:**
```javascript
{
  data: [...],        // Array of records
  meta: {            // Metadata (when requested)
    totalCount: 100,
    filterCount: 25
  }
}
```

`aggregate()` also returns `{ data: [...] }`, but its rows contain only computed dimensions and measures, never raw records or `meta.aggregate`.

**Available Methods:**
- [Find](./find.md) - Query raw records with filtering, sorting, and pagination
- [Aggregate](./find.md#aggregate-computed-results) - Return scalar summaries or grouped analytics
- [Create](./create-update-delete.md#create) - Create new records
- [Update](./create-update-delete.md#update) - Update existing records by ID
- [Delete](./create-update-delete.md#delete) - Delete records by ID
- [Batch mutations](./create-update-delete.md#batch-create-update-and-delete) - Create, update, or delete many records in one generic-table operation

## Accessing Repositories

Repositories are available through the context object (`$ctx`) in hooks and handlers:

```javascript
// Access a repository by table name
const productsRepo = $ctx.$repos.products;
const usersRepo = $ctx.$repos.enfyra_user;

// Access the main table repository (if configured in route)
const mainRepo = $ctx.$repos.main;
```

**Important:**
- Repositories are resolved from **metadata** by **table `name` or `alias`** (see `RepoRegistryService`). You do not configure a per-route list for basic access.
- **`$ctx.$repos.main`** is the current route’s main table (field permissions enforced).
- **`$ctx.$repos.secure.<name>`** — same enforcement for another table; **`$ctx.$repos.<name>`** — no field-permission enforcement unless you use `main` / `secure`.
- All repository methods are async and require `await`

## Documentation

- **[Find Records](./find.md)** - Complete guide to querying records
- **[Create, Update, and Delete Records](./create-update-delete.md)** - Create, update, and delete operations
- **[Common Patterns](./patterns.md)** - Best practices and common patterns

## Next Steps

- Learn about the [Context Object ($ctx)](../context-reference/) to understand all available properties
- See [Query Filtering](../query-filtering.md) for advanced filtering patterns
- Check [API Lifecycle](../api-lifecycle.md) to understand how repositories fit into the request flow

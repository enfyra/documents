# Repository Methods

Repositories are the main way to interact with your database tables in Enfyra. Every table you create automatically gets a repository that you can access through `$ctx.$repos.tableName`.

## Quick Reference

**All repository methods return data in this format:**
```javascript
{
  data: [...],        // Array of records
  meta: {            // Metadata (when requested)
    totalCount: 100,
    filterCount: 25
  }
}
```

**Available Methods:**
- [`find()`](./find.md) - Query records with filtering, sorting, and pagination
- [`create()`](./create-update-delete.md#create) - Create new records
- [`update()`](./create-update-delete.md#update) - Update existing records by ID
- [`delete()`](./create-update-delete.md#delete) - Delete records by ID

## Accessing Repositories

Repositories are available through the context object (`$ctx`) in hooks and handlers:

```javascript
// Access a repository by table name
const productsRepo = $ctx.$repos.products;
const usersRepo = $ctx.$repos.user_definition;

// Access the main table repository (if configured in route)
const mainRepo = $ctx.$repos.main;
```

**Important:** 
- Tables must be configured in the route's "Target Tables" field to be accessible as repositories
- The main table is always available as `$ctx.$repos.main`
- All repository methods are async and require `await`

## Documentation

- **[find() Method](./find.md)** - Complete guide to querying records
- **[create(), update(), delete() Methods](./create-update-delete.md)** - Create, update, and delete operations
- **[Common Patterns](./patterns.md)** - Best practices and common patterns

## Next Steps

- Learn about the [Context Object ($ctx)](../context-reference/) to understand all available properties
- See [Query Filtering](../query-filtering.md) for advanced filtering patterns
- Check [API Lifecycle](../api-lifecycle.md) to understand how repositories fit into the request flow

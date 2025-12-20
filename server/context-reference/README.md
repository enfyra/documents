# Context Object ($ctx) Reference

The `$ctx` (context) object is available in all hooks and handlers. It provides access to request data, database repositories, helper functions, cache operations, and more.

## Quick Navigation

- [Request Data](./request-data.md) - `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- [Repositories](./repositories.md) - `$ctx.$repos` for database operations
- [Helpers & Cache](./helpers-cache.md) - `$ctx.$helpers` and `$ctx.$cache`
- [Logging & Error Handling](./logging-errors.md) - `$ctx.$logs()` and `$ctx.$throw`
- [Advanced Features](./advanced.md) - File uploads, API info, shared context, packages, and patterns

## Documentation

- **[Request Data](./request-data.md)** - Access HTTP request information and parameters
- **[Repositories](./repositories.md)** - Database operations through repositories
- **[Helpers & Cache](./helpers-cache.md)** - Utility functions and Redis cache operations
- **[Logging & Error Handling](./logging-errors.md)** - Adding logs and throwing errors
- **[Advanced Features](./advanced.md)** - File uploads, API information, shared context, and more

## Next Steps

- See [Repository Methods](../repository-methods/) for database operations
- Check [API Lifecycle](../api-lifecycle.md) to understand when context is available
- Learn about [Hooks and Handlers](../hooks-handlers/) for using context in custom code

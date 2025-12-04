# Enfyra Server Documentation

This documentation covers the Enfyra server architecture, APIs, and development guides. It's organized to help you find information quickly, whether you're learning the basics or looking up specific details.

## Quick Navigation

### Getting Started
- **[Repository Methods](./repository-methods/)** - Complete guide to database operations (find, create, update, delete)
- **[Context Object ($ctx)](./context-reference/)** - All available properties and methods in the context
- **[API Lifecycle](./api-lifecycle.md)** - How requests flow through the system

### Core Concepts
- **[Hooks and Handlers](./hooks-handlers/)** - Creating preHooks, afterHooks, and custom handlers
- **[Query Filtering](./query-filtering.md)** - MongoDB-like filtering operators and examples
- **[Error Handling](./error-handling.md)** - Throwing errors and handling exceptions

### Advanced Topics
- **[Cache Operations](./cache-operations.md)** - Distributed caching and locking
- **[File Handling](./file-handling.md)** - File uploads and management
- **[Cluster Architecture](./cluster-architecture.md)** - Multi-instance coordination

## Finding What You Need

### "I want to query data from a table"
 See [Repository Methods - find()](./repository-methods/find.md)

### "I need to create a new record"
 See [Repository Methods - create()](./repository-methods/create-update-delete.md#create)

### "I want to update a record"
 See [Repository Methods - update()](./repository-methods/create-update-delete.md#update)

### "I need to delete a record"
 See [Repository Methods - delete()](./repository-methods/create-update-delete.md#delete)

### "What properties are available in $ctx?"
 See [Context Reference](./context-reference/)

### "How do I access request body and params?"
 See [Context Reference - Request Data](./context-reference/request-data.md)

### "How do I use repositories in my code?"
 See [Context Reference - Repositories](./context-reference/repositories.md)

### "What helper functions are available?"
 See [Context Reference - Helpers & Cache](./context-reference/helpers-cache.md)

### "How does the request lifecycle work?"
 See [API Lifecycle](./api-lifecycle.md)

### "How do hooks execute in order?"
 See [API Lifecycle - Execution Order](./api-lifecycle.md#execution-order)

### "I want to validate data before saving"
 See [Hooks and Handlers - preHooks](./hooks-handlers/prehooks.md)

### "I need to modify response data"
 See [Hooks and Handlers - afterHooks](./hooks-handlers/afterhooks.md)

### "I want to write custom business logic"
 See [Hooks and Handlers - Custom Handlers](./hooks-handlers/custom-handlers.md)

### "How do I filter data with complex conditions?"
 See [Query Filtering](./query-filtering.md)

### "How do I throw errors properly?"
 See [Error Handling](./error-handling.md)

### "I need to use Redis cache"
 See [Cache Operations](./cache-operations.md)

### "I want to upload files"
 See [File Handling](./file-handling.md)

### "How do repositories work?"
 See [Repository Methods](./repository-methods/)

### "What methods does repository have?"
 See [Repository Methods](./repository-methods/) - Complete list of find, create, update, delete methods

## Documentation Structure

All documentation is organized step-by-step, with clear examples and explanations. Each document focuses on a specific topic and includes:

1. **Overview** - What the topic is about
2. **Step-by-step guides** - How to use it
3. **Examples** - Real code examples
4. **Reference** - Complete API reference
5. **Common patterns** - Best practices and tips

## Repository Methods Overview

The repository is the main way to interact with your database tables. Each table you create automatically gets a repository accessible through `$ctx.$repos.tableName`.

**Available Methods:**
- `find()` - Query records with filtering, sorting, and pagination
- `create()` - Create new records
- `update()` - Update existing records by ID
- `delete()` - Delete records by ID

All methods return data in a consistent format: `{ data: [...], meta: {...} }`

See [Repository Methods Guide](./repository-methods/) for complete details.

## Context Object Overview

The `$ctx` (context) object is available in all hooks and handlers. It provides access to:

- **Request Data**: `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- **Repositories**: `$ctx.$repos.tableName` for database operations
- **Helpers**: `$ctx.$helpers` for JWT, bcrypt, file operations
- **Cache**: `$ctx.$cache` for Redis operations
- **Logging**: `$ctx.$logs()` for adding logs to responses
- **Error Handling**: `$ctx.$throw['400']()` for throwing errors

See [Context Reference](./context-reference/) for complete details.

## API Lifecycle Overview

Every API request follows this flow:

1. **Route Detection** - System matches request to route definition
2. **Context Setup** - Creates `$ctx` with repositories and helpers
3. **preHooks Execution** - Runs all matching preHooks sequentially
4. **Handler Execution** - Custom handler or default CRUD operation
5. **afterHooks Execution** - Runs all matching afterHooks sequentially
6. **Response** - Returns processed data

The same `$ctx` object flows through all phases, so modifications in preHooks are visible to handlers and afterHooks.

See [API Lifecycle](./api-lifecycle.md) for complete details.

## Learning Path

**New to Enfyra Server?** Follow this step-by-step path:

1. **[Repository Methods](./repository-methods/)** - Start here! Learn how to query, create, update, and delete records
2. **[Context Reference](./context-reference/)** - Understand all available properties and methods in `$ctx`
3. **[API Lifecycle](./api-lifecycle.md)** - Learn how requests flow through the system
4. **[Hooks and Handlers](./hooks-handlers/)** - Customize API behavior with hooks and handlers
5. **[Query Filtering](./query-filtering.md)** - Master filtering and querying data
6. **[Error Handling](./error-handling.md)** - Handle errors properly

**Advanced topics:**
- [Cache Operations](./cache-operations.md) - Distributed caching and locking
- [File Handling](./file-handling.md) - File uploads and management
- [Cluster Architecture](./cluster-architecture.md) - Multi-instance coordination

## Next Steps

1. Start with [Repository Methods](./repository-methods/) to learn database operations
2. Read [Context Reference](./context-reference/) to understand available properties
3. Check [API Lifecycle](./api-lifecycle.md) to see how everything fits together
4. Explore [Hooks and Handlers](./hooks-handlers/) for customization

For specific questions, use the Quick Navigation section above to jump directly to relevant documentation.


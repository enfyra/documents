# Hooks and Handlers

Hooks and handlers allow you to customize API behavior at different points in the request lifecycle. Hooks run before and after handlers, while handlers contain the main business logic.

## Quick Navigation

- [Overview](#overview) - What are hooks and handlers
- [preHooks](./prehooks.md) - Execute before handler
- [afterHooks](./afterhooks.md) - Execute after handler
- [Custom Handlers](./custom-handlers.md) - Replace default CRUD
- [Common Patterns](./patterns.md) - Real-world examples and best practices

## Overview

### Hooks

Hooks are code snippets that run at specific points in the request lifecycle:
- **preHooks**: Execute before the handler
- **afterHooks**: Execute after the handler

### Handlers

Handlers contain the main business logic:
- **Custom Handler**: Your custom code
- **Default CRUD**: Automatic CRUD operation based on HTTP method

### Execution Flow

```
preHook #1  preHook #2  Handler  afterHook #1  afterHook #2
```

All hooks and handlers have access to the same `$ctx` object, so changes in one phase are visible to all subsequent phases.

## Documentation

- **[preHooks](./prehooks.md)** - Validation, data transformation, and permission checks
- **[afterHooks](./afterhooks.md)** - Response transformation, audit logging, and side effects
- **[Custom Handlers](./custom-handlers.md)** - Custom business logic and hook types
- **[Common Patterns](./patterns.md)** - Best practices and real-world examples

## Next Steps

- Learn about [Repository Methods](../repository-methods/) for database operations
- See [Context Reference](../context-reference/) for all available properties
- Check [API Lifecycle](./api-lifecycle.md) to understand execution order

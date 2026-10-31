# Hooks and Handlers

Hooks and handlers allow you to customize API behavior at different points in the request lifecycle. Hooks run before and after handlers, while handlers contain the main business logic.

## Quick Navigation

- [Overview](#overview) - What are hooks and handlers
- [preHooks](./prehooks.md) - Execute before handler
- [postHooks](./posthooks.md) - Execute after handler
- [Custom Handlers](./custom-handlers.md) - Replace default CRUD
- [Common Patterns](./patterns.md) - Real-world examples and best practices

## Overview

### Hooks

Hooks are code snippets that run at specific points in the request lifecycle:
- **preHooks**: Execute before the handler
- **postHooks**: Execute after the handler

### Handlers

Handlers contain the main business logic:
- **Custom Handler**: Your custom code
- **Default CRUD**: Automatic CRUD operation based on HTTP method

### Execution Flow

```
Pre-Auth Guards  Auth  Post-Auth Guards  preHook #1  preHook #2  Handler  postHook #1  postHook #2
```

Guards run before hooks and handle IP blocking / rate limiting. Hooks run after guards and handle validation / business logic.

Use `@SHARE` for request-local decisions shared by hooks and the handler. Split responsibilities into named hooks with explicit methods and distinct priorities (lower first): initialize a default decision, let matching later hooks refine it, then consume it in the handler.

A pre-hook may return early to skip only itself; the return value is ignored. Throw to stop the request before the handler. Matching post-hooks still run after ordinary pre-hook or handler errors with `@ERROR` populated; cancellation and deadlines use the separate [terminal audit contract](./posthooks.md).

## Documentation

- **[preHooks](./prehooks.md)** - Validation, data transformation, and permission checks
- **[postHooks](./posthooks.md)** - Response transformation, audit logging, and side effects
- **[Custom Handlers](./custom-handlers.md)** - Custom business logic and hook types
- **[Common Patterns](./patterns.md)** - Best practices and real-world examples

## Next Steps

- Learn about [Repository Methods](../repository-methods/) for database operations
- See [Context Reference](../context-reference/) for all available properties
- Check [API Lifecycle](../api-lifecycle.md) to understand execution order

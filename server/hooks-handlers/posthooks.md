# Hooks and Handlers - postHooks

postHooks execute after the handler (or after a preHook/handler error). Use them for response transformation, audit logging, and side effects.

## Execution Behavior

- postHooks **always run**, even when preHook or handler throws an error
- On error path: `@ERROR` is populated, `@DATA` is `null`, `@STATUS` is the error status code
- On success path: `@ERROR` is `undefined`, `@DATA` is the handler result, `@STATUS` is `200`
- postHooks run as **side effects** — on error path, the original error is always thrown after all postHooks complete
- If one postHook fails, other postHooks still run (individual try/catch per postHook)

## When to Use postHooks

- Transform response data (success path)
- Add computed fields to response
- Log audit trails (both success and error)
- Trigger side effects (emails, notifications)
- Handle/log errors that occurred in preHook or handler
- Add metadata to response

## Basic postHook Example

```javascript
// Transform response (success path only)
if (@DATA && Array.isArray(@DATA.data)) {
  @DATA.data = @DATA.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

## Response Enhancement

```javascript
// Add metadata
if (@DATA) {
  @DATA.meta = {
    processedAt: new Date(),
    processedBy: @USER?.id,
    version: '1.0'
  };
}

// Add computed fields to each item
if (@DATA && Array.isArray(@DATA.data)) {
  @DATA.data = @DATA.data.map(item => ({
    ...item,
    isActive: item.status === 'active',
    displayName: item.nickname || item.name
  }));
}
```

## Audit Logging

```javascript
// Log all operations (success and error)
await #audit_logs.create({
  data: {
    action: `${@API.request.method} ${@API.request.url}`,
    userId: @USER?.id,
    resourceId: @PARAMS.id,
    statusCode: @STATUS,
    error: @ERROR ? @ERROR.message : null,
    timestamp: new Date()
  }
});
```

## Error Handling

postHooks receive error context via `@ERROR` (or `$ctx.$error`). On error path, the error is always re-thrown after postHooks complete — postHooks cannot override the error.

```javascript
// Log errors for monitoring
if (@ERROR) {
  @LOGS(`Error: ${@ERROR.message}`);

  await #error_logs.create({
    data: {
      errorMessage: @ERROR.message,
      statusCode: @ERROR.statusCode,
      userId: @USER?.id,
      url: @API.request.url,
      timestamp: new Date()
    }
  });

  // Send notification on critical errors
  if (@ERROR.statusCode >= 500) {
    await @SOCKET.emitToRoom('admin', 'server-error', {
      message: @ERROR.message,
      url: @API.request.url
    });
  }
}
```

## Available Context in postHooks

| Variable | Success Path | Error Path |
|----------|-------------|------------|
| `@DATA` | Handler result | `null` |
| `@STATUS` | `200` | Error status code (e.g. `400`, `500`) |
| `@ERROR` | `undefined` | `{ message, name, statusCode, details, timestamp }` |
| `@API.error` | `undefined` | Same as `@ERROR` |
| `@API.response` | `undefined` | `{ statusCode, responseTime, timestamp }` |
| `@USER` | Current user | Current user |
| `@SHARE` | Shared data from preHooks | Shared data from preHooks |
| `@LOGS(...)` | Available | Available |

## Next Steps

- See [preHooks](./prehooks.md) for pre-handler operations
- Learn about [Custom Handlers](./custom-handlers.md) for custom business logic
- Check [Common Patterns](./patterns.md) for best practices

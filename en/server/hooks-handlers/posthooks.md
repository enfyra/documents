# Hooks and Handlers - postHooks

postHooks execute after the handler (or after a preHook/handler error). Use them for response transformation, audit logging, and side effects.

## Execution Behavior

- postHooks still run when a preHook or handler throws an ordinary error, even though later preHooks and the handler are skipped after a preHook rejection. Cancellation, execution deadlines and terminal stream failures use the separate opt-in audit path below.
- A preHook that returns early does not reject the request: later preHooks and the handler continue, then postHooks run normally.
- Give dependent postHooks distinct `priority` values (lower first). Read request-local decisions from `@SHARE`; check `@ERROR` before success-only effects.
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
    await @SOCKET.emitToRoom('/admin', 'admin', 'server-error', {
      message: @ERROR.message,
      url: @API.request.url
    });
  }
}
```

## Audit terminated HTTP requests

On a Server release supporting terminal failure audits, an attached post-hook can opt in by starting its source with the exact directive:

```javascript
'enfyra:client-disconnect-audit';
if (@ERROR?.origin) {
  @LOGS('Request terminated', {
    executionId: @API.request.executionId,
    origin: @ERROR.origin,
    code: @ERROR.code,
    statusCode: @STATUS,
  });
}
```

Server runs only these attached audit hooks in a fresh task with a separate five-second admission/execution budget. It does not replay inference or ordinary post-hooks, and the audit task has no live response. This is best-effort finalization, not a durable outbox; verify deployed support before relying on it.

The host-owned first cause is authoritative:

| Origin | Status / code |
|---|---|
| Client disconnect | `499 client_closed_request` |
| Response stream failure | `502 response_stream_error` |
| Stream total/idle deadline | `504 response_stream_timeout` |
| Execution deadline | `504 execution_timeout` |
| Other terminal execution failure | `500 execution_failed` |

Do not reinterpret upstream HTTP 499 or a later abort flag as a client disconnect. `@ERROR` contains `origin`, `statusCode`, `code` and a safe `message`; ordinary error paths retain their normal error shape. The audit context retains authenticated identity and the last host snapshot, not guaranteed in-flight `@SHARE` changes.

`@API.request.executionId` is a Server-generated UUID, independent of caller correlation/session IDs. An application writing failure rows must coordinate with successful settlement under the same transaction/lock and check authoritative persisted state so an already settled execution does not gain a duplicate failure row. Keep authenticated fallback for sparse snapshots and never record prompts, credentials or provider-private diagnostics in public audit data.

Before response headers, Server-owned failures send safe JSON with matching status and paired retry signals. After a stream is committed, its wire status cannot change; Server closes it. No second response is sent after client disconnect. Required durable work still needs an outbox or flow.

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

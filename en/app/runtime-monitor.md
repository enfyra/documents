# Runtime Monitor

Runtime Monitor is available at **Settings > Admin > Runtime Monitor**. It helps admins inspect the running Enfyra server without leaving the admin app.

## Tabs

- **Overview**: process, request, database, queue, websocket, and health metrics.
- **Redis**: current-app Redis usage, key categories, key browser, and editable user-cache values.

The monitor updates over a websocket connection. The Redis tab shows a countdown until the next server snapshot so you know when fresh metrics should arrive.

## Redis Overview

The Redis tab reads only the current app namespace. Enfyra automatically scopes keys by the backend `NODE_NAME`, so one Enfyra app cannot browse another app's Redis namespace from this UI.

Key categories are marked with badges:

- **runtime cache**: Enfyra-managed runtime definition snapshots. Read-only.
- **BullMQ**: queue state and retained jobs. Read-only.
- **Socket.IO**: websocket adapter state. Read-only.
- **runtime monitor**: runtime telemetry. Read-only.
- **system lock**: Enfyra coordination locks. Read-only.
- **user cache**: `$cache` / `@CACHE` values created by handlers, hooks, flows, websocket scripts, or the Redis Key Editor. Editable.

## Allocated Redis Memory

Allocated Redis Memory is the soft allocation for user cache only. It is controlled by `REDIS_USER_CACHE_LIMIT_MB` and defaults to `30` MB.

This limit does not include system keys such as runtime cache, BullMQ, Socket.IO, runtime telemetry, or locks. When user cache exceeds the allocation, Enfyra evicts least-recently-used user-cache keys.

## Key Browser

Search uses logical user-facing keys. For editable user cache, type the same key you would use in code:

```javascript
await @CACHE.set('user:123', user, 300000);
```

In Redis this is stored under the app namespace, but the UI hides the namespace and adds it automatically. You should not type `NODE_NAME` or `user_cache:` in search or editor fields.

The browser loads keys in pages of 10. Use **Load more** to fetch the next page instead of scanning the entire Redis namespace at once.

## Key Editor

Only **user cache** keys are editable. The editor uses the same service as `$cache` / `@CACHE`, so values created in the UI are visible in server scripts and values created by scripts are visible in the UI.

Values are parsed for display when possible and stringified before saving. System keys are read-only to avoid corrupting Enfyra runtime state.

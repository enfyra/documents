# Cluster Architecture

Multiple Enfyra **server** processes can run against the same database and Redis. Coordination uses **Redis Pub/Sub** (cache reload signals), **Redis-backed BullMQ** (background jobs), **Redis for Socket.IO** (`@socket.io/redis-adapter`), and a **Redis lock** only for bootstrap script execution.

This page reflects the current open-source server in the [`server`](https://github.com/enfyra/server) repository.

## What is actually “stateless”

- **HTTP requests** do not rely on server-local session files: auth uses JWT; sessions are stored in the database (`session_definition`).
- Each process still holds **large in-memory caches** (metadata, routes, GraphQL-related data, packages, storage config, OAuth config, websocket definitions, flows, folder tree, etc.). Those structures are **rebuilt from the database** after startup or when a peer signals a reload.
- So instances are **not** “zero RAM state”; they are **interchangeable** as long as they share DB + Redis and receive the same reload signals.

## Instance identity

- On startup, `InstanceService` assigns a **random 32-hex-character instance ID** (used to **ignore Pub/Sub messages published by the same process** so a reload does not immediately re-trigger itself).
- **`NODE_NAME` is separate** from the instance ID. It does **not** auto-generate a UUID for Pub/Sub.

## Cache synchronization (actual behavior)

When a cache service reloads (admin reload, metadata invalidation, etc.), the typical pattern in `BaseCacheService` / `MetadataCacheService` is:

1. Publish a small JSON message on the cache’s Redis channel: `{ instanceId, type: 'RELOAD_SIGNAL', timestamp }`.
2. **Not** included: full cache payloads over Redis (peers always **re-query the database**).
3. Subscribers on the same channel parse the message; if `instanceId` is **not** their own, they run their own **reload from DB** and refresh local in-memory state.

**Important:** There is **no** distributed “only one instance may hit the database” lock for general cache reloads in the current code. Constants define `RELOAD_LOCK_TTL`, but it is **not** wired into cache reload. Under heavy invalidation, several instances can reload metadata/routes **in parallel** (same DB work, acceptable for most deployments).

## `NODE_NAME` and Redis channels

`RedisPubSubService` builds the real channel name as:

- `BASE_CHANNEL` if `NODE_NAME` is **unset**
- `BASE_CHANNEL:NODE_NAME` if `NODE_NAME` is **set**

**All API instances that must share the same live metadata/routes must use the same `NODE_NAME` (or all leave it unset).** If each machine uses a different `NODE_NAME`, they subscribe to **different** channels and **will not** synchronize caches with each other.

`NODE_NAME` is **not** for “unique per instance”; it is an optional **environment / deployment segment** for channel names.

## BullMQ (background jobs)

BullMQ uses the same Redis connection as the app. The queue **key prefix** is `configService.get('NODE_NAME') || 'bull'`.

- For a **single logical cluster**, every Enfyra server instance should use the **same** `NODE_NAME` (or all unset so every process uses the prefix `bull`). Otherwise each instance only processes its **own** isolated queues (e.g. session cleanup, websocket worker jobs may not run as you expect cluster-wide).

## Bootstrap scripts (distributed lock)

`BootstrapScriptService` uses a Redis lock so **only one** instance runs enabled `bootstrap_script_definition` scripts at startup (or on reload):

- Lock key: `bootstrap-script-execution` (see `BOOTSTRAP_SCRIPT_EXECUTION_LOCK_KEY`)
- TTL: **30 seconds** (`REDIS_TTL.BOOTSTRAP_LOCK_TTL`)
- Value: publishing instance’s `instanceId`; released in `finally`

If the lock is not acquired, the instance **skips** running scripts (another instance is responsible).

The constant `enfyra:bootstrap-script-reload` exists in code as `BOOTSTRAP_SCRIPT_RELOAD_EVENT_KEY`; there is **no** separate subscriber wired to it in the current server—bootstrap coordination is the **lock** above plus normal cache invalidation when `bootstrap_script_definition` changes.

## Session cleanup (no Redis lock)

Expired `session_definition` rows are removed by a **BullMQ** repeatable job on queue `sys_session-cleanup` (`SYSTEM_QUEUES.SESSION_CLEANUP`), processor **concurrency 1**, schedule **`0 2 * * *`** (daily). There is **no** one-hour Redis lock for session cleanup in the current implementation.

## WebSockets across nodes

`DynamicWebSocketGateway` configures **`@socket.io/redis-adapter`** so Socket.IO rooms and emits can work across multiple Node processes sharing the same Redis.

Clients still need a load-balancer strategy compatible with WebSockets (e.g. sticky sessions or TCP pass-through). Redis adapter handles **cross-server** propagation, not HTTP stickiness.

## Redis Pub/Sub channel names (base keys)

Defined in server `src/shared/utils/constant.ts` (remember the `NODE_NAME` suffix rule):

| Base channel | Purpose |
| --- | --- |
| `enfyra:metadata-cache-sync` | Table/column/relation metadata cache |
| `enfyra:route-cache-sync` | Route trie, handlers, hooks, permissions |
| `enfyra:package-cache-sync` | Server package list / CDN cache coordination |
| `enfyra:storage-config-cache-sync` | Storage configurations |
| `enfyra:oauth-config-cache-sync` | OAuth provider configs |
| `enfyra:websocket-cache-sync` | Websocket gateway/event definitions |
| `enfyra:flow-cache-sync` | Flow definitions for scheduler/dispatch |
| `enfyra:folder-tree-cache-sync` | Folder tree cache |

GraphQL schema reload is driven by the same metadata/route invalidation pipeline and `GraphqlService.reloadSchema()`—there is **no** separate `enfyra:graphql-*` Pub/Sub channel in constants.

There is **no** `enfyra:ai-config-cache-sync` channel in the open-source server tree at the time of this document; if your deployment adds AI config caching, treat it as product-specific.

## Invalidation  which cache reloads

`CACHE_INVALIDATION_MAP` in `src/shared/utils/cache-events.constants.ts` maps metadata tables to affected caches (metadata, route, GraphQL, storage, websocket, package, bootstrap, OAuth, folder tree, flow). After a qualifying write, the instance emits internal events and affected caches reload and publish `RELOAD_SIGNAL` to peers.

## Fault tolerance (realistic)

- **Redis down at startup**: Pub/Sub client initialization can fail; the process may not start cleanly—check logs (`RedisPubSub`).
- **Redis lost at runtime**: Already-loaded in-memory caches keep serving **stale** data until TTL/operational reload; new reload signals fail to publish/subscribe until Redis returns.
- **Bootstrap lock TTL (30s)**: If a holder crashes, the lock expires so another instance can run scripts on a later attempt.

## Setup checklist for horizontal scaling

1. **Same database** for every API instance (`DB_URI` / Mongo URI, same logical DB).
2. **Same Redis** for Pub/Sub, BullMQ, and Socket.IO adapter (`REDIS_URI` and matching host/port/password if split).
3. **Same `NODE_NAME` on every instance of one cluster** (or **unset** on all) so Pub/Sub channels and BullMQ prefixes match.
4. **Load balancer** in front of HTTP/WebSocket as appropriate for your platform.

Passwords with special characters in URIs must be URL-encoded (same as any JDBC/Redis URL).

**Optional SQL read replicas:** `DB_REPLICA_URIS`, `DB_READ_FROM_MASTER`—see [Installation](../../getting-started/installation.md) for details.

## Benefits

- Scale HTTP by adding processes behind a load balancer.
- Shared Redis adapter + shared DB keep websocket and API behavior consistent across nodes.
- Bootstrap scripts and Bull workers avoid obvious duplicate execution where explicitly locked or single-concurrency.

# Architecture Overview

Enfyra is a programmable backend platform. You define data, routes, permissions, hooks, handlers, flows, and realtime events as metadata, and Enfyra turns that metadata into a running API without restarting the server.

This page explains the main moving parts and the request flow. It is written for builders who want to understand how Enfyra behaves before they build a real app on top of it.

## The Big Picture

```text
Browser or third-party app
        |
        | calls same-origin /api, /enfyra, or /socket.io
        v
Enfyra App / app proxy
        |
        | forwards REST, OAuth, files, GraphQL, Socket.IO transport
        v
Enfyra Server
        |
        | reads metadata, runs guards/hooks/handlers, writes data
        v
Database + Redis
```

The app is the public entry point users normally see. The server is the runtime engine. The database stores both your business data and Enfyra metadata. Redis is used for cache coordination, realtime fanout, queues, and multi-instance behavior.

## What Runs Where

### Enfyra App

The app provides the admin UI and a public app-origin bridge.

Typical responsibilities:

- Show the visual interface for collections, routes, permissions, flows, files, realtime gateways, and extensions.
- Proxy `/api/**` to the hidden Enfyra server.
- Proxy Socket.IO transport such as `/ws/socket.io/**` to the server.
- Let browser code keep same-origin cookies instead of calling the backend host directly.
- Render dynamic admin pages and extension pages.

The app does not connect to your database directly.

### Enfyra Server

The server is the runtime that makes metadata executable.

Typical responsibilities:

- Load table, route, permission, hook, handler, flow, websocket, storage, and package definitions.
- Generate REST endpoints for route-backed tables.
- Run route guards, pre-hooks, handlers, canonical CRUD, post-hooks, and field permissions.
- Run flow steps and websocket event scripts.
- Manage auth sessions, OAuth, refresh tokens, file uploads, GraphQL, and cache reloads.
- Coordinate multiple instances through Redis.

When you change metadata, the server reloads the affected runtime cache. In normal use, you do not restart the server to add a table, route, hook, handler, flow, extension, or websocket event.

### Database

The database stores:

- Your application tables, such as `post`, `project`, `order`, or `chat_message`.
- Enfyra system tables, such as `table_definition`, `route_definition`, `pre_hook_definition`, `flow_definition`, and `websocket_definition`.

Enfyra supports MySQL, PostgreSQL, and MongoDB. The public API uses logical table fields and relation names; Enfyra handles physical SQL foreign keys or Mongo junction collections internally.

### Redis

Redis is used when enabled for runtime coordination:

- Shared runtime cache across instances.
- Cache invalidation signals after metadata changes.
- Socket.IO fanout across multiple server instances.
- Flow queues and background work.
- Managed user cache through `@CACHE` / `$ctx.$cache`.

## How A Table Becomes An API

Example: you create a `post` table with `title`, `content`, and `status`.

1. In the app, create a collection named `post`.
2. Enfyra stores the table metadata in `table_definition`.
3. Enfyra creates or updates the physical database table.
4. Enfyra creates a default REST route for `/post`.
5. The route cache reloads.
6. Your app can immediately call:

```http
GET /api/post?limit=20
POST /api/post
PATCH /api/post/1
DELETE /api/post/1
```

There is no dynamic `GET /api/post/1`. To fetch one record, filter by the primary key:

```http
GET /api/post?filter={"id":{"_eq":1}}&limit=1
```

## Request Flow

When a request hits a dynamic REST route, Enfyra processes it in this order:

```text
HTTP request
  -> route detection
  -> auth/session lookup
  -> guards and route permissions
  -> pre-hooks
  -> custom handler or default CRUD
  -> post-hooks
  -> field permission cleanup
  -> JSON response
```

### Example: Create A Post

Request:

```http
POST /api/post
Content-Type: application/json

{
  "title": "Hello Enfyra",
  "content": "This post was created through a generated API.",
  "status": "draft"
}
```

What Enfyra can do around that request:

- A guard can reject the request before it reaches business logic.
- A pre-hook can set `author` to the current user.
- A custom handler can replace the default create behavior.
- Default CRUD can validate the body and insert the row.
- A post-hook can trigger a flow or emit a websocket event.
- Field permissions can remove fields the caller is not allowed to see.

## Hooks And Handlers

Hooks and handlers are how you add code without editing the server codebase.

### Pre-hook Example

Set the current user as the author before creating a post:

```js
if (@BODY && @USER?.id) {
  @BODY.author = { id: @USER.id }
}
```

### Custom Handler Example

Return a dashboard summary instead of normal CRUD:

```js
const published = await #post.find({
  filter: { status: { _eq: "published" } },
  fields: "id",
  limit: 1,
  meta: "filterCount"
})

const drafts = await #post.find({
  filter: { status: { _eq: "draft" } },
  fields: "id",
  limit: 1,
  meta: "filterCount"
})

return {
  published: published.meta.filterCount,
  drafts: drafts.meta.filterCount
}
```

### Post-hook Example

Trigger a flow after a record is created:

```js
if (!@ERROR && @DATA?.data?.[0]) {
  await @TRIGGER("post-created", {
    postId: @DATA.data[0].id,
    title: @DATA.data[0].title
  })
}
```

## Relations

Relations are first-class metadata, not plain `userId` or JSON fields.

Example: `post.author` points to `user_definition`.

Create and update payloads use the relation property name:

```json
{
  "title": "A related post",
  "author": { "id": 12 }
}
```

Queries also use relation property names:

```http
GET /api/post?filter={"author":{"id":{"_eq":12}}}
```

Deep fetch uses relation names too:

```http
GET /api/post?fields=id,title,author&deep={"author":{"fields":"id,email,displayName"}}
```

You normally do not provide physical foreign key column names. Enfyra derives and manages those internally.

## Auth And OAuth

Enfyra owns auth sessions and refresh tokens. Browser apps should use the app-origin proxy so cookies stay on the app domain.

### Password Login

```http
POST /api/login
```

or in a third-party app that proxies `/enfyra/**`:

```http
POST /enfyra/login
```

Then fetch the current user:

```http
GET /api/me
```

or:

```http
GET /enfyra/me
```

### OAuth Through A Third-Party App

A third-party app should start OAuth through its own proxy prefix:

```js
const redirect = new URL("/chat", window.location.origin)
const url = new URL("/enfyra/auth/google", window.location.origin)
url.searchParams.set("redirect", redirect.toString())
url.searchParams.set("cookieBridgePrefix", "/enfyra")
window.location.href = url.toString()
```

The flow is:

```text
User clicks Google login
  -> third app /enfyra/auth/google
  -> Enfyra redirects to Google
  -> Google redirects to Enfyra callback
  -> Enfyra redirects through {thirdAppOrigin}/enfyra/auth/set-cookies
  -> third app stores cookies through the proxy response
  -> browser returns to /chat
```

The user does not need to copy tokens or manually finish login.

## Realtime

Enfyra realtime is Socket.IO, configured through `websocket_definition` and `websocket_event_definition`.

Example gateway:

```text
path: /chat
requireAuth: true
```

Browser client in a third-party app:

```js
const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true
})
```

The app proxies `/socket.io/**` to the Enfyra app bridge `/ws/socket.io/**`. The backend gateway namespace stays `/chat`.

Example event script:

```js
const { conversationId, text } = @BODY
if (!conversationId || !text) @THROW400("conversationId and text are required")

const membership = await #chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  limit: 1
})

if (!membership.data[0]) @THROW403("Not a conversation member")

const created = await #chat_message.create({
  data: {
    conversation: { id: conversationId },
    sender: { id: @USER.id },
    text
  }
})

@SOCKET.emitToRoom(`conversation:${conversationId}`, "chat:message", {
  message: created.data?.[0] ?? null
})
```

## Flows

Flows are metadata-backed workflows. They are useful when work should be split into ordered steps, retried, scheduled, or tracked in execution history.

Example use cases:

- Send a welcome email after registration.
- Process an order after payment.
- Sync data to another service.
- Run a scheduled cleanup.

Trigger a flow from a hook or handler:

```js
await @TRIGGER("send-welcome-email", {
  userId: @DATA.data[0].id,
  email: @DATA.data[0].email
})
```

## Third-Party Apps

A Nuxt, Next, mobile, or desktop app can use Enfyra as its backend.

Recommended browser app pattern:

```text
Third app public origin
  /enfyra/**     -> Enfyra API
  /socket.io/**  -> Enfyra Socket.IO transport bridge
```

Then the app calls:

```js
fetch("/enfyra/post", { credentials: "include" })
fetch("/enfyra/login", { method: "POST", credentials: "include" })
io("/chat", { path: "/socket.io", withCredentials: true })
```

This keeps the backend hidden, keeps cookies same-origin, and lets Enfyra manage refresh tokens.

## Multi-Instance Behavior

In production, you can run more than one Enfyra server instance. Instances coordinate through Redis.

When one instance changes metadata:

```text
Instance A updates route/table/hook metadata
  -> database stores the metadata
  -> Redis publishes invalidation
  -> other instances reload the affected cache
  -> new API behavior becomes available without restarting
```

For realtime, Socket.IO can fan out room/user events across instances through the Redis adapter.

## A Complete Example Flow

Build a simple support inbox:

1. Create `ticket` table with `subject`, `status`, `priority`, and `description`.
2. Relate `ticket.requester` to `user_definition`.
3. Add a pre-hook on `POST /ticket` to set `requester` from `@USER.id`.
4. Add route permissions so users can create and view their own tickets.
5. Add a pre-hook on `GET /ticket` to filter tickets by requester unless the user is an admin.
6. Add a `ticket-created` flow to notify the support team.
7. Add a `/support` extension page that lists tickets using `/api/ticket`.
8. Add a websocket gateway `/support` if agents need live updates.

The result is a real app feature built from metadata, generated API, small scripts, and optional UI extension code.

## Related Documentation

- [Getting Started](./getting-started/README.md)
- [Installation](./getting-started/installation.md)
- [API Reference](./api-reference/README.md)
- [App Documentation](./app/README.md)
- [Server Documentation](./server/README.md)
- [WebSocket Guide](./server/websocket.md)
- [User Registration Example](./examples/user-registration-example.md)
- [Row-Level Security Example](./examples/multi-tenant-rls-example.md)

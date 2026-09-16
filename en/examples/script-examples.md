# Script Examples

Small examples for handlers, hooks, flows, and websocket event scripts.

## Repository Reads

### Find Rows

```js
const posts = await #post.find({
  fields: "id,title",
  limit: 10
})
```

### Find One Row

```js
const result = await #post.find({
  filter: { id: { _eq: @PARAMS.id } },
  fields: "id,title",
  limit: 1
})

return result.data[0] || null
```

### Count Rows

Use `aggregate()` for a computed count instead of loading records or requesting legacy `meta` aggregate data.

```js
const result = await #post.aggregate({
  filter: { status: { _eq: "published" } },
  measures: {
    posts: { count: "id" }
  }
})

return result.data[0]?.posts ?? 0
```

### Group Rows by Day

```js
const result = await #post.aggregate({
  dimensions: [
    { field: "createdAt", bucket: "day", timezone: "Asia/Ho_Chi_Minh" }
  ],
  measures: {
    posts: { count: "id" }
  },
  sort: [{ field: "createdAt", direction: "asc" }]
})

return result.data
```

## Repository Writes

### Create Row

```js
const created = await #post.create({
  data: {
    title: @BODY.title,
    status: "draft"
  }
})

return created.data[0]
```

### Update Row

```js
const updated = await #post.update({
  id: @PARAMS.id,
  data: {
    title: @BODY.title
  }
})

return updated.data[0]
```

### Delete Row

```js
await #post.delete({
  id: @PARAMS.id
})

return { deleted: true }
```

## Errors

### Required Body Field

```js
if (!@BODY.title) {
  @THROW400("Title is required")
}
```

### Forbidden Action

```js
if (!@USER) {
  @THROW403("Login required")
}
```

### Not Found

```js
const row = await #post.find({
  filter: { id: { _eq: @PARAMS.id } },
  fields: "id",
  limit: 1
})

if (!row.data[0]) {
  @THROW404("post", @PARAMS.id)
}
```

## Hooks

### Pre-Hook Owner Scope

```js
@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { owner: { id: { _eq: @USER.id } } }
  ]
}
```

### Pre-Hook Set Owner On Create

```js
@BODY.owner = { id: @USER.id }
```

### Pre-Hook Strip Server Field

```js
delete @BODY.isAdmin
```

### Post-Hook Shape Response

```js
return {
  id: @DATA.id,
  title: @DATA.title
}
```

## Flows

### Trigger Flow From Handler

```js
await @TRIGGER("send-welcome-email", {
  userId: @USER.id
})

return { queued: true }
```

### Flow Step Reads Payload

```js
const userId = @FLOW_PAYLOAD.userId

return { userId }
```

### Flow Step Uses Previous Step

```js
const user = @FLOW.load_user

return {
  email: user.email
}
```

## WebSocket

### Reply To Current Client

```js
@SOCKET.reply("pong", {
  at: new Date().toISOString()
})
```

### Join Room

```js
@SOCKET.join(`project:${@BODY.projectId}`)
```

### Emit To Current Room

```js
@SOCKET.emitToCurrentRoom(`project:${@BODY.projectId}`, "project:changed", {
  projectId: @BODY.projectId
})
```

### Emit To User

```js
@SOCKET.emitToUser(@USER.id, "notification", {
  title: "Done"
})
```

## Package Streams

See [Package Management](../app/hooks-handlers/package-management.md) for single-consumer, timeout, cancellation, error, and `observer`/`transform` semantics.

### Preflight Then Relay Exact Bytes

```js
const upstream = await @PKGS.undici.request("https://api.example.com/stream", {
  method: "POST",
  body: JSON.stringify(@BODY)
})

const guarded = await $ctx.$streams.preflight(upstream.body, {
  timeoutMs: 15_000
})

await @RES.stream(guarded.stream, {
  statusCode: upstream.statusCode,
  mimetype: "text/event-stream"
})
```

### Consume Before Returning JSON

```js
const upstream = await @PKGS.undici.request("https://api.example.com/result", {
  method: "POST",
  body: JSON.stringify(@BODY)
})

const text = await $ctx.$streams.readText(upstream.body, {
  timeoutMs: 60_000,
  maxBytes: 8 * 1024 * 1024
})

return { result: JSON.parse(text) }
```

## Cache

### Set Cache

```js
await @CACHE.set("report:latest", { count: 12 }, 60000)
```

### Get Cache

```js
const cached = await @CACHE.get("report:latest")
if (cached) return cached
```

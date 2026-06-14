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

```js
const result = await #post.find({
  fields: "id",
  limit: 1,
  meta: "totalCount"
})

return result.meta.totalCount
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

### Emit To Room

```js
@SOCKET.emitToRoom(`project:${@BODY.projectId}`, "project:changed", {
  projectId: @BODY.projectId
})
```

### Emit To User

```js
@SOCKET.emitToUser(@USER.id, "notification", {
  title: "Done"
})
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

---
slug: vi-du-script
---

# Ví dụ script

Các ví dụ nhỏ cho handler, hook, flow và script sự kiện websocket.

## Đọc repository

### Tìm các dòng

```js
const posts = await #post.find({
  fields: "id,title",
  limit: 10
})
```

### Tìm một dòng

```js
const result = await #post.find({
  filter: { id: { _eq: @PARAMS.id } },
  fields: "id,title",
  limit: 1
})

return result.data[0] || null
```

### Đếm dòng

```js
const result = await #post.find({
  fields: "id",
  limit: 1,
  meta: "totalCount"
})

return result.meta.totalCount
```

## Ghi repository

### Tạo dòng

```js
const created = await #post.create({
  data: {
    title: @BODY.title,
    status: "draft"
  }
})

return created.data[0]
```

### Cập nhật dòng

```js
const updated = await #post.update({
  id: @PARAMS.id,
  data: {
    title: @BODY.title
  }
})

return updated.data[0]
```

### Xóa dòng

```js
await #post.delete({
  id: @PARAMS.id
})

return { deleted: true }
```

## Lỗi

### Trường body bắt buộc

```js
if (!@BODY.title) {
  @THROW400("Title is required")
}
```

### Thao tác bị cấm

```js
if (!@USER) {
  @THROW403("Login required")
}
```

### Không tìm thấy

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

## Hook

### Pre-hook giới hạn theo chủ sở hữu

```js
@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { owner: { id: { _eq: @USER.id } } }
  ]
}
```

### Pre-hook đặt chủ sở hữu khi tạo

```js
@BODY.owner = { id: @USER.id }
```

### Pre-hook loại bỏ trường phía server

```js
delete @BODY.isAdmin
```

### Post-hook định hình response

```js
return {
  id: @DATA.id,
  title: @DATA.title
}
```

## Flow

### Kích hoạt flow từ handler

```js
await @TRIGGER("send-welcome-email", {
  userId: @USER.id
})

return { queued: true }
```

### Bước flow đọc payload

```js
const userId = @FLOW_PAYLOAD.userId

return { userId }
```

### Bước flow dùng bước trước đó

```js
const user = @FLOW.load_user

return {
  email: user.email
}
```

## WebSocket

### Trả lời client hiện tại

```js
@SOCKET.reply("pong", {
  at: new Date().toISOString()
})
```

### Tham gia room

```js
@SOCKET.join(`project:${@BODY.projectId}`)
```

### Emit tới room hiện tại

```js
@SOCKET.emitToCurrentRoom(`project:${@BODY.projectId}`, "project:changed", {
  projectId: @BODY.projectId
})
```

### Emit tới người dùng

```js
@SOCKET.emitToUser(@USER.id, "notification", {
  title: "Done"
})
```

## Cache

### Đặt cache

```js
await @CACHE.set("report:latest", { count: 12 }, 60000)
```

### Lấy cache

```js
const cached = await @CACHE.get("report:latest")
if (cached) return cached
```

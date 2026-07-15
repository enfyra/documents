---
slug: tong-quan-kien-truc
---

# Tổng quan kiến trúc

Enfyra là nền tảng backend có thể lập trình. Bạn định nghĩa dữ liệu, route, permission, hook, handler, flow và realtime event bằng metadata; Enfyra biến metadata đó thành API đang chạy mà không cần restart server.

Trang này mô tả các thành phần chính và luồng request, dành cho builder muốn hiểu Enfyra trước khi xây app thực tế.

## Bức tranh tổng thể

```text
Trình duyệt hoặc ứng dụng bên thứ ba
        |
        | gọi /api, /enfyra hoặc /socket.io cùng origin
        v
Ứng dụng Enfyra / proxy ứng dụng
        |
        | chuyển tiếp REST, OAuth, tệp, GraphQL và Socket.IO
        v
Máy chủ Enfyra
        |
        | đọc metadata, chạy guard/hook/handler và ghi dữ liệu
        v
Cơ sở dữ liệu + Redis
```

App là điểm truy cập công khai mà người dùng thường tương tác; server là runtime engine. Database lưu dữ liệu nghiệp vụ lẫn metadata Enfyra. Redis phối hợp cache, realtime fanout, queue và hoạt động đa instance.

## Thành phần chạy ở đâu

### Enfyra App

App cung cấp admin UI và public app-origin bridge:

- Hiển thị collection, route, permission, flow, file, realtime gateway và extension.
- Proxy `/api/**` tới Enfyra server ẩn; proxy Socket.IO như `/ws/socket.io/**` tới server.
- Giữ cookie same-origin cho browser thay vì gọi backend host trực tiếp.
- Render dynamic admin page và extension page.

App không kết nối trực tiếp database.

### Enfyra Server

Server biến metadata thành thực thi:

- Load definition table, route, permission, hook, handler, flow, websocket, storage và package.
- Sinh REST endpoint cho route-backed table.
- Chạy route guard, pre-hook, handler, canonical CRUD, post-hook và field permission.
- Chạy flow step, websocket event script, auth session, OAuth, refresh token, file upload, GraphQL và cache reload.
- Phối hợp nhiều instance qua Redis.

Khi metadata thay đổi, server nạp lại runtime cache bị ảnh hưởng. Trong vận hành thông thường, không cần restart để thêm table, route, hook, handler, flow, extension hoặc WebSocket event.

### Database và Redis

Database lưu bảng ứng dụng như `post`, `project`, `order`, `chat_message` và system table như `enfyra_table`, `enfyra_route`, `enfyra_pre_hook`, `enfyra_flow`, `enfyra_websocket`. Enfyra hỗ trợ MySQL, PostgreSQL, MongoDB; public API dùng logical field và relation name, còn Enfyra tự xử lý SQL foreign key hay Mongo junction collection.

Redis (khi bật) dùng cho shared runtime cache, cache invalidation sau metadata change, Socket.IO fanout giữa server instance, flow queue/background work và user cache qua `@CACHE` / `$ctx.$cache`.

## Một bảng trở thành API thế nào

Ví dụ tạo bảng `post` có `title`, `content`, `status`:

1. Tạo collection `post` trong app.
2. Enfyra lưu metadata vào `enfyra_table`.
3. Enfyra tạo/cập nhật physical database table.
4. Enfyra tạo default REST route `/post`.
5. Route cache reload.
6. App gọi ngay:

```http
GET /api/post?limit=20
POST /api/post
PATCH /api/post/1
DELETE /api/post/1
```

Không có dynamic `GET /api/post/1`. Với SQL, lấy một record bằng filter primary key:

```http
GET /api/post?filter={"id":{"_eq":1}}&limit=1
```

MongoDB thường dùng `_id`.

## Luồng request

```text
HTTP request
  -> nhận diện route
  -> tra cứu auth/session
  -> guard và route permission
  -> pre-hook
  -> handler tùy chỉnh hoặc CRUD mặc định
  -> post-hook
  -> lọc field theo permission
  -> JSON response
```

Ví dụ tạo post:

```http
POST /api/post
Content-Type: application/json

{
  "title": "Hello Enfyra",
  "content": "This post was created through a generated API.",
  "status": "draft"
}
```

Guard có thể từ chối request trước business logic; pre-hook đặt `author` là user hiện tại; custom handler thay default create; CRUD kiểm tra body và insert; post-hook trigger flow hoặc emit websocket event; field permission loại field caller không được xem.

## Hook và handler

Hook/handler cho phép thêm code mà không sửa server codebase.

### Pre-hook

Đặt current user làm author trước khi tạo post:

```js
if (@BODY && @USER?.id) {
  @BODY.author = { id: @USER.id }
}
```

### Custom handler

Trả dashboard summary thay CRUD thông thường:

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

### Post-hook

Trigger flow sau khi tạo record:

```js
if (!@ERROR && @DATA?.data?.[0]) {
  await @TRIGGER("post-created", {
    postId: @DATA.data[0].id,
    title: @DATA.data[0].title
  })
}
```

## Relation

Relation là metadata hạng nhất, không chỉ là field `userId` hoặc JSON. `post.author` trỏ đến `enfyra_user`; create/update dùng property name:

```json
{
  "title": "A related post",
  "author": { "id": 12 }
}
```

Query cũng dùng relation name:

```http
GET /api/post?filter={"author":{"id":{"_eq":12}}}
```

```http
GET /api/post?fields=id,title,author&deep={"author":{"fields":"id,email,displayName"}}
```

Thông thường không cung cấp physical foreign-key column name; Enfyra tự suy ra và quản lý.

## Auth, OAuth và third-party app

Enfyra sở hữu auth session và refresh token. Browser app nên dùng app-origin proxy để cookie ở app domain.

```http
POST /api/login
```

```http
POST /enfyra/login
```

```http
GET /api/me
```

```http
GET /enfyra/me
```

Trong third-party app proxy `/enfyra/**`, dùng `/enfyra/login` và `/enfyra/me`. OAuth bắt đầu qua proxy prefix của app:

```js
const redirect = new URL("/chat", window.location.origin)
const url = new URL("/api/auth/google", "https://demo.enfyra.io")
url.searchParams.set("redirect", redirect.toString())
url.searchParams.set("cookieBridgePrefix", "/enfyra")
window.location.href = url.toString()
```

```text
Người dùng bấm đăng nhập Google
  -> Ứng dụng Enfyra /api/auth/google
  -> Enfyra chuyển hướng đến Google
  -> Google chuyển hướng về callback của Enfyra
  -> Enfyra chuyển hướng qua {thirdAppOrigin}/enfyra/auth/set-cookies
  -> ứng dụng bên thứ ba lưu cookie từ phản hồi proxy
  -> trình duyệt quay lại /chat
```

Không đưa token vào URL/browser storage. Xem [SSR Frameworks](./integrations/ssr-frameworks.md) và [Authentication](./api-reference/authentication.md).

## Realtime và flow

Realtime dùng Socket.IO namespace và transport path, không phải raw websocket path. Browser thường kết nối bằng:

```text
path: /chat
requireAuth: true
```

```js
const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true
})
```

```js
const { conversationId, text } = @BODY
if (!conversationId || !text) @THROW400("conversationId and text are required")

const membership = await @REPOS.chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  limit: 1
})

if (!membership.data[0]) @THROW403("Not a conversation member")

const created = await @REPOS.chat_message.create({
  data: {
    conversation: { id: conversationId },
    sender: { id: @USER.id },
    text
  }
})

await @REPOS.chat_conversation.update({
  id: conversationId,
  data: { lastMessage: { id: created.data[0].id } }
})

@SOCKET.emitToCurrentRoom(`conversation:${conversationId}`, "chat:message", {
  message: created.data?.[0] ?? null
})
```

Websocket metadata định nghĩa gateway, namespace, event và script; Redis fanout đồng bộ event qua nhiều instance.

Flow là automation metadata gồm trigger và step. Trigger có thể đến hook, schedule, manual call, webhook hoặc realtime event. Step có thể thực hiện script, HTTP, condition, delay/queue hoặc thao tác runtime khác; history lưu kết quả thực thi.

```js
await @TRIGGER("send-welcome-email", {
  userId: @DATA.data[0].id,
  email: @DATA.data[0].email
})
```

```text
Origin công khai của ứng dụng bên thứ ba
  /enfyra/**     -> API Enfyra
  /socket.io/**  -> cầu nối transport Socket.IO của Enfyra
```

```js
fetch("/enfyra/post", { credentials: "include" })
fetch("/enfyra/login", { method: "POST", credentials: "include" })
io("/chat", { path: "/socket.io", withCredentials: true })
```

## Multi-instance behavior

Nhiều server instance dùng chung database; khi bật Redis, chúng phối hợp runtime cache, invalidation, Socket.IO fanout và queue. Metadata change phát invalidation để instance khác reload definition bị ảnh hưởng. `NODE_NAME` phải duy nhất cho mỗi node; `SECRET_KEY` phải giống nhau cho mọi node của cùng app.

```text
Instance A cập nhật metadata route/table/hook
  -> cơ sở dữ liệu lưu metadata
  -> Redis phát sự kiện vô hiệu hóa cache
  -> các instance khác nạp lại cache bị ảnh hưởng
  -> hành vi API mới có hiệu lực mà không cần khởi động lại
```

## Ví dụ luồng hoàn chỉnh

1. User gọi `POST /api/post` qua app proxy.
2. Server xác thực cookie/session, kiểm tra permission/guard.
3. Pre-hook gán author.
4. CRUD validate và ghi post vào database.
5. Post-hook trigger flow `post-created`.
6. Flow có thể gửi HTTP notification; websocket event có thể emit update.
7. Response được dọn field theo permission rồi trả JSON qua proxy.

## Tài liệu liên quan

- [Cài đặt](./getting-started/installation.md)
- [API Reference](./api-reference/README.md)
- [Server](./server/README.md)
- [App](./app/README.md)
- [SSR Frameworks](./integrations/ssr-frameworks.md)

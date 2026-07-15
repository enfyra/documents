---
slug: vi-du/ung-dung-chat-ben-thu-ba
---

# Ví dụ ứng dụng chat bên thứ ba

Xây dựng một ứng dụng chat nhỏ trên Enfyra từ ứng dụng SSR bên ngoài.

Hãy đọc [SSR Frameworks](../integrations/ssr-frameworks.md) trước. Ví dụ này giả định ứng dụng của bạn đã proxy:

```text
/enfyra/**     -> Enfyra app /api/**
/socket.io/**  -> Enfyra app /ws/socket.io/**
```

## Nội dung sẽ xây dựng

```text
Conversation list
  -> REST read

Selected conversation
  -> REST message history

Send message
  -> Socket.IO event
  -> Enfyra event script persists message
  -> Enfyra broadcasts to room
```

## 1. Tạo bảng

### chat_conversation

| Trường | Kiểu |
|-------|------|
| title | string |
| kind | string |
| description | text |
| updatedAt | datetime |
| createdBy | many-to-one tới `enfyra_user` |
| lastMessage | many-to-one tới `chat_message`, có thể để trống |

### chat_conversation_member

| Trường | Kiểu |
|-------|------|
| conversation | many-to-one tới `chat_conversation` |
| member | many-to-one tới `enfyra_user` |
| role | string |
| joinedAt | datetime |

### chat_message

| Trường | Kiểu |
|-------|------|
| conversation | many-to-one tới `chat_conversation` |
| sender | many-to-one tới `enfyra_user` |
| text | text |
| persistStatus | string |

### chat_message_read

| Trường | Kiểu |
|-------|------|
| message | many-to-one tới `chat_message` |
| conversation | many-to-one tới `chat_conversation` |
| member | many-to-one tới `enfyra_user` |
| isRead | boolean |
| readAt | datetime |

Dùng cascade delete từ `chat_conversation` tới thành viên/tin nhắn nếu việc xóa cuộc trò chuyện phải xóa dữ liệu chat của nó.

## 2. Tải danh sách cuộc trò chuyện

Chỉ tải danh sách khi trang được render lần đầu.

```ts
const conversations = await fetch(
  "/enfyra/chat_conversation?fields=id,title,kind,lastMessage.id,lastMessage.text,lastMessage.createdAt&limit=0",
  { credentials: "include" },
).then((res) => res.json())
```

Không tải tin nhắn của mọi cuộc trò chuyện khi làm mới trang. Chỉ tải tin nhắn sau khi người dùng chọn một cuộc trò chuyện.
Sắp xếp danh sách ở frontend theo `conversation.lastMessage?.createdAt`. Cuộc trò chuyện chỉ giữ relation tới tin nhắn mới nhất, không nhân đôi các trường text/ngày xem trước.

## 3. Tải tin nhắn sau khi chọn

```ts
const filter = encodeURIComponent(JSON.stringify({
  conversation: { id: { _eq: conversationId } },
}))

const messages = await fetch(
  `/enfyra/chat_message?filter=${filter}&fields=id,text,createdAt,sender&deep=${encodeURIComponent(JSON.stringify({
    sender: {},
  }))}&sort=-createdAt,-id&limit=20`,
  { credentials: "include" },
).then((res) => res.json())
```

Dùng `limit=20` cho trang đang hiển thị. Chỉ tải tin nhắn cũ hơn khi người dùng yêu cầu.

## 4. Kết nối Socket.IO

```ts
import { io } from "socket.io-client"

const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true,
  reconnection: false,
  transports: ["polling"],
  upgrade: false,
})

socket.emit("chat:join")

socket.on("chat:message", (payload) => {
  appendMessage(payload.message)
})
```

`/chat` là websocket namespace của Enfyra. `/socket.io` là transport path được ứng dụng SSR proxy.

## 5. Thêm event tin nhắn

Tạo websocket event tên `chat:message`.

Event script:

```js
const { conversationId, messageId, text } = @BODY

if (!conversationId) @THROW400("conversationId is required")
if (!text) @THROW400("text is required")

const membership = await @REPOS.chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  fields: "id",
  limit: 1
})

if (!membership.data[0]) @THROW403("Not a conversation member")

const created = await @REPOS.chat_message.create({
  data: {
    conversation: { id: conversationId },
    sender: { id: @USER.id },
    text,
    persistStatus: "persisted"
  }
})

await @REPOS.chat_conversation.update({
  filter: { id: { _eq: conversationId } },
  data: {
    lastMessage: { id: created.data[0].id },
    updatedAt: new Date().toISOString()
  }
})

@SOCKET.emitToCurrentRoom(`conversation:${conversationId}`, "chat:message", {
  message: created.data[0]
})

@SOCKET.reply("chat:message:sent", {
  messageId,
  message: created.data[0]
})
```

Client gửi:

```ts
socket.emit("chat:message", {
  conversationId,
  messageId: crypto.randomUUID(),
  text,
})
```

## 6. Thêm event tham gia

Tạo websocket event tên `chat:join`.

```js
const memberships = await @REPOS.chat_conversation_member.find({
  filter: {
    member: { id: { _eq: @USER.id } }
  },
  fields: "id,conversation",
  deep: { conversation: { fields: "id" } },
  limit: 0
})

for (const row of memberships.data || []) {
  const conversationId = row.conversation?.id
  if (conversationId) {
    @SOCKET.join(`conversation:${conversationId}`)
  }
}

@SOCKET.reply("chat:joined", {
  joined: memberships.data?.length || 0
})
```

## 7. Giữ `lastMessage` chính xác khi xóa

Nếu người dùng có thể xóa từng tin nhắn, hãy thêm pre-hook và post-hook cho `DELETE /chat_message`.

Pre-hook tạo bản chụp tin nhắn bị xóa trước khi default delete handler chạy:

```js
const message = await @REPOS.chat_message.find({
  filter: { id: { _eq: @PARAMS.id } },
  fields: "id,createdAt,conversation",
  limit: 1
})

const row = message.data?.[0]
if (!row) return

const conversationId = row.conversation?.id || row.conversation
const conversation = await @REPOS.chat_conversation.find({
  filter: { id: { _eq: conversationId } },
  fields: "id,lastMessage",
  limit: 1
})
const current = conversation.data?.[0]
const currentLastId = current?.lastMessage?.id || current?.lastMessage

const membership = await @REPOS.chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  fields: "id",
  limit: 1
})

if (!membership.data?.length) @THROW403("Not a conversation member")

@SHARE.deletedChatMessage = {
  id: row.id,
  conversationId,
  wasLastMessage: String(currentLastId || "") === String(row.id)
}
```

Post-hook chỉ sửa cuộc trò chuyện khi tin nhắn bị xóa là `lastMessage` hiện tại:

```js
const deleted = @SHARE.deletedChatMessage
if (!deleted?.conversationId) return
if (!deleted.wasLastMessage) return

const nextLast = await @REPOS.chat_message.find({
  filter: { conversation: { id: { _eq: deleted.conversationId } } },
  fields: "id",
  sort: "-createdAt,-id",
  limit: 1
})

await @REPOS.chat_conversation.update({
  id: deleted.conversationId,
  data: {
    lastMessage: nextLast.data?.[0]?.id ? { id: nextLast.data[0].id } : null
  }
})
```

## 8. Thêm trạng thái đã đọc

Khi người dùng mở một cuộc trò chuyện, emit `chat:read`.

```ts
socket.emit("chat:read", {
  conversationId,
  readAt: new Date().toISOString(),
})
```

Server đánh dấu các dòng `chat_message_read` là đã đọc cho `@USER` và emit `chat:read` tới các user room để những tab đang mở khác xóa dấu chưa đọc.

## Lỗi thường gặp

### Tải toàn bộ tin nhắn khi làm mới

Tải cuộc trò chuyện trước. Chỉ tải tin nhắn sau khi chọn.

### Chỉ gửi tin nhắn qua REST

REST có thể tạo bản ghi, nhưng UX chat cần chuyển tin theo realtime. Dùng Socket.IO để gửi và broadcast.

### Tin `conversationId` mà không kiểm tra thành viên

Mọi event tham gia room và gửi tin đều phải xác minh tư cách thành viên trước khi xử lý.

## Tài liệu liên quan

- [SSR Frameworks](../integrations/ssr-frameworks.md)
- [WebSocket Guide](../server/websocket.md)
- [Query Filtering](../server/query-filtering.md)
- [API Lifecycle](../server/api-lifecycle.md)

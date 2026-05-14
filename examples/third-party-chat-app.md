# Third-Party Chat App Example

Build a small chat app on top of Enfyra from an external SSR app.

Read [SSR Frameworks](../integrations/ssr-frameworks.md) first. This example assumes your app already proxies:

```text
/enfyra/**     -> Enfyra app /api/**
/socket.io/**  -> Enfyra app /ws/socket.io/**
```

## What You Build

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

## 1. Create Tables

### chat_conversation

| Field | Type |
|-------|------|
| title | string |
| isGroup | boolean |
| lastMessageAt | datetime |

### chat_conversation_member

| Field | Type |
|-------|------|
| conversation | many-to-one to `chat_conversation` |
| member | many-to-one to `user_definition` |
| lastReadAt | datetime |

### chat_message

| Field | Type |
|-------|------|
| conversation | many-to-one to `chat_conversation` |
| sender | many-to-one to `user_definition` |
| text | text |

Use cascade delete from `chat_conversation` to members/messages if deleting a conversation should remove its chat data.

## 2. Load The Conversation List

Only load the list on initial page render.

```ts
const conversations = await fetch(
  "/enfyra/chat_conversation?fields=id,title,isGroup,lastMessageAt&sort=-lastMessageAt,id&limit=20",
  { credentials: "include" },
).then((res) => res.json())
```

Do not load messages for every conversation during page refresh. Load messages after the user selects one conversation.

## 3. Load Messages After Selection

```ts
const filter = encodeURIComponent(JSON.stringify({
  conversation: { id: { _eq: conversationId } },
}))

const messages = await fetch(
  `/enfyra/chat_message?filter=${filter}&fields=id,text,createdAt,sender&deep=${encodeURIComponent(JSON.stringify({
    sender: { fields: "id,name,email" },
  }))}&sort=createdAt,id&limit=20`,
  { credentials: "include" },
).then((res) => res.json())
```

Use `limit=20` for the visible page. Load older messages only when the user asks.

## 4. Connect Socket.IO

```ts
import { io } from "socket.io-client"

const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true,
})

socket.emit("room:join", { conversationId })

socket.on("chat:message", (payload) => {
  appendMessage(payload.message)
})
```

`/chat` is the Enfyra websocket namespace. `/socket.io` is the transport path proxied by the SSR app.

## 5. Add A Send Message Event

Create a websocket event named `chat:send`.

Event script:

```js
const { conversationId, text } = @BODY

if (!conversationId) @THROW400("conversationId is required")
if (!text) @THROW400("text is required")

const membership = await #chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  fields: "id",
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

await #chat_conversation.update({
  filter: { id: { _eq: conversationId } },
  data: { lastMessageAt: new Date().toISOString() }
})

@SOCKET.emitToRoom(`conversation:${conversationId}`, "chat:message", {
  message: created.data[0]
})
```

Client send:

```ts
socket.emit("chat:send", {
  conversationId,
  text,
})
```

## 6. Add A Join Room Event

Create a websocket event named `room:join`.

```js
const { conversationId } = @BODY

const membership = await #chat_conversation_member.find({
  filter: {
    conversation: { id: { _eq: conversationId } },
    member: { id: { _eq: @USER.id } }
  },
  fields: "id",
  limit: 1
})

if (!membership.data[0]) @THROW403("Not a conversation member")

@SOCKET.join(`conversation:${conversationId}`)
```

## 7. Add Read State

When a user opens a conversation, update their membership row.

```ts
await fetch(`/enfyra/chat_conversation_member?filter=${encodeURIComponent(JSON.stringify({
  conversation: { id: { _eq: conversationId } },
  member: { id: { _eq: currentUserId } },
}))}`, {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  credentials: "include",
  body: JSON.stringify({ lastReadAt: new Date().toISOString() }),
})
```

Unread count can be derived by comparing `chat_message.createdAt` with `chat_conversation_member.lastReadAt`.

## Common Mistakes

### Loading all messages on refresh

Load conversations first. Load messages only after selection.

### Sending messages through REST only

REST can create records, but chat UX needs realtime delivery. Use Socket.IO for send and broadcast.

### Trusting conversationId without membership check

Every room join and send event should verify membership before doing work.

## Related Documentation

- [SSR Frameworks](../integrations/ssr-frameworks.md)
- [WebSocket Guide](../server/websocket.md)
- [Query Filtering](../server/query-filtering.md)
- [API Lifecycle](../server/api-lifecycle.md)

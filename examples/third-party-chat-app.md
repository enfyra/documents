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
| kind | string |
| description | text |
| updatedAt | datetime |
| createdBy | many-to-one to `user_definition` |
| lastMessage | many-to-one to `chat_message`, nullable |

### chat_conversation_member

| Field | Type |
|-------|------|
| conversation | many-to-one to `chat_conversation` |
| member | many-to-one to `user_definition` |
| role | string |
| joinedAt | datetime |

### chat_message

| Field | Type |
|-------|------|
| conversation | many-to-one to `chat_conversation` |
| sender | many-to-one to `user_definition` |
| text | text |
| persistStatus | string |

### chat_message_read

| Field | Type |
|-------|------|
| message | many-to-one to `chat_message` |
| conversation | many-to-one to `chat_conversation` |
| member | many-to-one to `user_definition` |
| isRead | boolean |
| readAt | datetime |

Use cascade delete from `chat_conversation` to members/messages if deleting a conversation should remove its chat data.

## 2. Load The Conversation List

Only load the list on initial page render.

```ts
const conversations = await fetch(
  "/enfyra/chat_conversation?fields=id,title,kind,lastMessage.id,lastMessage.text,lastMessage.createdAt&limit=0",
  { credentials: "include" },
).then((res) => res.json())
```

Do not load messages for every conversation during page refresh. Load messages after the user selects one conversation.
Sort the list in the frontend by `conversation.lastMessage?.createdAt`. The conversation keeps only a relation to the latest message, not duplicated preview text/date fields.

## 3. Load Messages After Selection

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

Use `limit=20` for the visible page. Load older messages only when the user asks.

## 4. Connect Socket.IO

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

`/chat` is the Enfyra websocket namespace. `/socket.io` is the transport path proxied by the SSR app.

## 5. Add A Message Event

Create a websocket event named `chat:message`.

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

@SOCKET.emitToRoom(`conversation:${conversationId}`, "chat:message", {
  message: created.data[0]
})

@SOCKET.reply("chat:message:sent", {
  messageId,
  message: created.data[0]
})
```

Client send:

```ts
socket.emit("chat:message", {
  conversationId,
  messageId: crypto.randomUUID(),
  text,
})
```

## 6. Add A Join Event

Create a websocket event named `chat:join`.

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

## 7. Keep `lastMessage` Correct On Delete

If users can delete individual messages, add a `DELETE /chat_message` pre-hook and post-hook.

The pre-hook snapshots the deleted message before the default delete handler runs:

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

The post-hook only repairs the conversation when the deleted message was the current `lastMessage`:

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

## 8. Add Read State

When a user opens a conversation, emit `chat:read`.

```ts
socket.emit("chat:read", {
  conversationId,
  readAt: new Date().toISOString(),
})
```

The server marks `chat_message_read` rows as read for `@USER` and emits `chat:read` to user rooms so other open tabs clear unread dots.

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

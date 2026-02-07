# WebSocket Guide

WebSocket enables real-time, bidirectional communication between server and clients. This guide explains how to use WebSocket in Enfyra for building real-time features like chat, notifications, and live updates.

## Quick Navigation

- [Overview](#overview) - What WebSocket provides
- [WebSocket Definition Structure](#websocket-definition-structure) - Database tables and configuration
- [Creating WebSocket Gateways](#creating-websocket-gateways) - Setting up WebSocket routes
- [Event Handlers](#event-handlers) - Handling incoming events
- [Connection Handlers](#connection-handlers) - Running scripts on client connection
- [Rejecting Connections](#rejecting-connections) - How to reject connections
- [WebSocket Context ($ctx)](#websocket-context-ctx) - Available properties and methods
- [Authentication](#authentication) - Securing WebSocket connections
- [Broadcasting vs User Messaging](#broadcasting-vs-user-messaging) - Sending patterns
- [Multi-Instance Support](#multi-instance-support) - Redis coordination
- [Client Integration](#client-integration) - How clients connect using SDK
- [Best Practices](#best-practices) - Common patterns and tips

## Overview

Enfyra WebSocket system provides:

- ✅ **Dynamic Gateway Management** - Define WebSocket routes from database
- ✅ **Event Handler System** - Custom event handlers via code scripts
- ✅ **Authentication Support** - JWT token verification, cookies, query params
- ✅ **Multi-Instance Coordination** - Redis pub/sub for cross-instance messaging
- ✅ **User-Specific Messaging** - Send messages to specific users across instances
- ✅ **Broadcast Support** - Send to all connected clients on a path
- ✅ **Target Tables** - Specify which tables/repositories are accessible
- ✅ **Connection Handlers** - Run custom scripts when clients connect
- ✅ **Template Syntax** - Use `@SOCKET` shorthand in handlers

### WebSocket Lifecycle

```
1. Client connects to WebSocket path
   ↓
2. Server verifies JWT token (if requireAuth: true)
   ↓
3. Connection handler executes (via HandlerQueue)
   ↓
4. Client sends event: { event: "message", data: {...} }
   ↓
5. Server matches event to definition
   ↓
6. Event handler executes with $ctx
   ↓
7. Response sent back to client
   ↓
8. Client disconnects
```

## WebSocket Definition Structure

WebSocket configuration is stored in two database tables:

### websocket_definition (Gateways)

| Field | Type | Description |
|-------|------|-------------|
| `path` | string | WebSocket route path (e.g., `/chat`, `/notifications/:userId`) |
| `isEnabled` | boolean | Enable/disable gateway |
| `isSystem` | boolean | System vs user-defined |
| `description` | string | Gateway description |
| `requireAuth` | boolean | Require JWT authentication |
| `connectionHandlerScript` | string | Script to run on connection |
| `connectionHandlerTimeout` | number | Timeout for connection handler (ms) |
| `targetTables` | array | Tables accessible via `$ctx.$repos` |

### websocket_event_definition (Events)

| Field | Type | Description |
|-------|------|-------------|
| `gatewayId` | string | Foreign key to websocket_definition |
| `eventName` | string | Event name to listen for (e.g., `message`, `ping`) |
| `isEnabled` | boolean | Enable/disable event |
| `isSystem` | boolean | System vs user-defined |
| `handlerScript` | string | Event handler code |
| `timeout` | number | Event handler timeout (ms) |

## Creating WebSocket Gateways

### Via Admin UI

1. Navigate to **WebSocket** section in admin panel
2. Click **Add Gateway**
3. Configure:
   - **Path**: Route path (e.g., `/chat`)
   - **Require Auth**: Enable JWT authentication
   - **Target Tables**: Select tables to access in handlers
   - **Connection Handler**: Optional script on connection
4. Click **Save**

### Via Database Directly

```sql
INSERT INTO websocket_definition (path, isEnabled, isSystem, description, requireAuth, targetTables)
VALUES (
  '/chat',
  true,
  false,
  'Real-time chat gateway',
  true,
  JSON_ARRAY(
    JSON_OBJECT('name', 'messages', 'alias', 'messagesRepo'),
    JSON_OBJECT('name', 'users', 'alias', 'usersRepo')
  )
);
```

## Event Handlers

### Creating Event Handlers

1. Create a gateway first
2. Click **Add Event** on the gateway
3. Configure:
   - **Event Name**: Name to listen for (e.g., `message`)
   - **Handler Script**: Code to execute when event is received
4. Click **Save**

### Event Handler Example

```javascript
// Event: "message"
// Handler Script

// Get message data from client
const { text, roomId } = @DATA;

// Validate
if (!text || !roomId) {
  @throw['400']('text and roomId are required');
  return;
}

// Save message to database
const message = await @REPOS.messagesRepo.create({
  data: {
    text: text,
    roomId: roomId,
    userId: @USER.id,
    createdAt: new Date()
  }
});

// Broadcast to all users in room
@SOCKET.broadcast('newMessage', {
  roomId: roomId,
  message: message.data[0]
});

// Return saved message
return message.data[0];
```

### Traditional Syntax

```javascript
// Same handler using traditional $ctx syntax
const { text, roomId } = $ctx.$data;

if (!text || !roomId) {
  $ctx.$throw['400']('text and roomId are required');
  return;
}

const message = await $ctx.$repos.messagesRepo.create({
  data: {
    text: text,
    roomId: roomId,
    userId: $ctx.$user.id,
    createdAt: new Date()
  }
});

$ctx.$socket.broadcast('newMessage', {
  roomId: roomId,
  message: message.data[0]
});

return message.data[0];
```

## Connection Handlers

Connection handlers run when a client connects to the gateway. They're useful for:

- Sending initial data to the client
- Logging connection events
- Subscribing to channels
- Validating connection conditions

### Connection Handler Example

```javascript
// Connection Handler Script

// Log connection
@LOGS('User connected:', @USER.id);

// Send welcome message
@SOCKET.send('connected', {
  userId: @USER.id,
  message: 'Welcome to the chat!',
  timestamp: new Date()
});

// Get recent messages for the user
const recentMessages = await @REPOS.messagesRepo.find({
  limit: 20,
  sort: '-createdAt'
});

// Send recent messages to client
@SOCKET.send('recentMessages', recentMessages.data);

// Store user info in shared context
@SHARE.onlineUsers = await @REPOS.usersRepo.find({
  where: { status: { _eq: 'online' } }
});
```

### Connection Handler vs Event Handler

| Feature | Connection Handler | Event Handler |
|---------|-------------------|---------------|
| **When it runs** | On client connection | When client sends event |
| **Execution** | Via HandlerQueue (async) | Via HandlerExecutor (direct) |
| **@DATA** | Not available | Contains event payload |
| **Use case** | Initialization, welcome messages | Business logic |

### Rejecting Connections

You can reject connections by calling `@SOCKET.close()` in the connection handler:

```javascript
// Connection Handler Script

// Check if user is allowed to connect
if (@USER.status !== 'active') {
  @SOCKET.send('error', { message: 'Account is not active' });
  @SOCKET.close();  // Reject the connection
  @LOGS('Connection rejected for inactive user:', @USER.id);
  return;
}

// Check if user is banned
if (@USER.isBanned) {
  @SOCKET.send('error', { message: 'You are banned from this server' });
  @SOCKET.close();  // Reject the connection
  return;
}

// Check concurrent connections limit
const activeConnections = await @REPOS.activeConnectionsRepo.find({
  where: { userId: { _eq: @USER.id } }
});

if (activeConnections.data.length >= 3) {
  @SOCKET.send('error', { message: 'Too many concurrent connections' });
  @SOCKET.close();  // Reject the connection
  return;
}

// Accept connection - send welcome message
@SOCKET.send('connected', {
  userId: @USER.id,
  message: 'Welcome!'
});
```

**Note:** When you call `@SOCKET.close()`, the connection is terminated immediately. Any code after `close()` will not execute, so always `return` after closing.

## WebSocket Context ($ctx)

WebSocket handlers have access to these context properties:

### Request Data

```javascript
@SOCKET          // WebSocket methods (send, broadcast, etc.)
@DATA            // Event payload data (event handlers only)
@USER            // Authenticated user (if requireAuth: true)
@PARAMS          // URL parameters (e.g., /chat/:roomId)
```

### Database & Utilities

```javascript
@REPOS           // Dynamic repositories from targetTables
@HELPERS         // Helper functions
@CACHE           // Cache operations
@LOGS()          // Logging function
@THROW           // Error throwing
@SHARE           // Shared state between handlers
```

### WebSocket Methods (@SOCKET)

```javascript
// Send to current client only
@SOCKET.send('event', { data: 'value' });

// Send to specific user (any instance)
await @SOCKET.sendToUser('userId123', 'event', { data: 'value' });

// Broadcast to all clients on this gateway path
await @SOCKET.broadcast('event', { data: 'value' });

// Close the connection (reject connection)
@SOCKET.close();

// Check connection state
if (@SOCKET.readyState === 1) {
  // WebSocket is open (WebSocket.OPEN = 1)
}
```

### Traditional Syntax

```javascript
$ctx.$socket.send('event', { data: 'value' });
await $ctx.$socket.sendToUser('userId123', 'event', { data: 'value' });
await $ctx.$socket.broadcast('event', { data: 'value' });
$ctx.$socket.close();

if ($ctx.$socket.readyState === 1) {
  // WebSocket is open
}
```

## Authentication

WebSocket supports three authentication methods:

### 1. Bearer Token (Authorization Header)

```javascript
// Client side
const ws = new WebSocket('ws://localhost:1105/chat', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

### 2. Query Parameter

```javascript
// Client side
const ws = new WebSocket(`ws://localhost:1105/chat?token=${token}`);
```

### 3. Cookie

```javascript
// Cookie is automatically sent if set
const ws = new WebSocket('ws://localhost:1105/chat');
```

### Gateway Configuration

Set `requireAuth: true` on the gateway to enforce authentication:

```javascript
// Gateway definition
{
  path: '/chat',
  requireAuth: true,
  // If client connects without valid token, connection is rejected
}
```

## Broadcasting vs User Messaging

### Broadcast to All Clients

Send to all connected clients on a gateway path:

```javascript
// In event handler
await @SOCKET.broadcast('newMessage', {
  text: 'Hello everyone!',
  from: @USER.name
});
```

**Broadcast across all instances:**
- Uses Redis pub/sub
- All instances receive and forward to their connected clients
- Original sender doesn't receive their own message back

### Send to Specific User

Send to a specific user, whichever instance they're connected to:

```javascript
// In event handler
await @SOCKET.sendToUser(targetUserId, 'privateMessage', {
  text: 'Hello!',
  from: @USER.id
});
```

**User messaging flow:**
1. Check if user is connected to this instance
2. If yes, send directly
3. If no, publish to Redis `ws:user:message` channel
4. Target instance receives and delivers

### Send to Current Client Only

Send only to the client that triggered the event:

```javascript
// In event handler
@SOCKET.send('response', {
  status: 'success',
  data: result
});
```

## Multi-Instance Support

WebSocket supports multiple server instances with Redis coordination:

### WebsocketRedisAdapter

The Redis adapter enables:
- Cross-instance broadcasting
- User-specific messaging across instances
- Cache synchronization
- Distributed state management

### How It Works

```
Instance A                    Redis                     Instance B
    |                           |                           |
    |--[broadcast]------------->|                           |
    |                           |--[publish]--------------->|
    |                           |                           |--[send to clients]
    |                           |                           |
    |<--[ack]-------------------|                           |
```

### Cache Synchronization

When WebSocket definitions change:
1. One instance calls `reloadDefinitions()`
2. Publishes to `enfyra:websocket-cache-sync` channel
3. All instances receive and reload their definitions
4. All instances have consistent gateway/event configurations

### Distributed Lock

Prevents race conditions during reload:
- Lock key: `websocket:reload:lock`
- Ensures only one instance reloads at a time
- Other instances wait for lock release

## Best Practices

### 1. Validate Input Early

```javascript
// Event handler
const { text, roomId } = @DATA;

if (!text) {
  @throw['400']('text is required');
  return;
}

if (!roomId) {
  @throw['400']('roomId is required');
  return;
}

// Continue with validated data
```

### 2. Use Target Tables

Specify which tables are accessible in the gateway:

```javascript
// Gateway targetTables configuration
[
  { name: 'messages', alias: 'messagesRepo' },
  { name: 'users', alias: 'usersRepo' }
]

// In handler, use the alias
const messages = await @REPOS.messagesRepo.find({ limit: 10 });
```

### 3. Handle Connection States

```javascript
// Check before sending
if (@SOCKET.readyState === 1) {
  @SOCKET.send('event', { data: 'value' });
} else {
  @LOGS('WebSocket not connected');
}
```

### 4. Use Connection Handlers Wisely

```javascript
// Connection handlers run async via HandlerQueue
// Use for initialization, not for blocking operations

// ✅ Good: Send initial data
@SOCKET.send('connected', { userId: @USER.id });

// ❌ Bad: Long-running operations
// Don't do heavy processing here, use event handlers instead
```

### 5. Broadcast Responsibly

```javascript
// Only broadcast what's necessary
await @SOCKET.broadcast('userJoined', {
  userId: @USER.id,
  name: @USER.name
  // Don't include sensitive data
});
```

### 6. Error Handling

```javascript
// Always validate and throw meaningful errors
if (!@USER) {
  @throw['401']('Authentication required');
  return;
}

if (@USER.role !== 'admin') {
  @throw['403']('Admin access required');
  return;
}
```

### 7. Use @SHARE for Cross-Handler State

```javascript
// Connection handler: Store state
@SHARE.connectedAt = new Date();
@SHARE.roomInfo = await @REPOS.roomsRepo.find({
  where: { id: { _eq: @PARAMS.roomId } }
});

// Event handler: Use stored state
const connectionTime = @SHARE.connectedAt;
const roomInfo = @SHARE.roomInfo;
```

## Common Patterns

### Pattern 1: Chat System

```javascript
// Connection Handler
@SOCKET.send('connected', { userId: @USER.id });

const rooms = await @REPOS.roomsRepo.find({
  where: { members: { _contains: @USER.id } }
});
@SOCKET.send('userRooms', rooms.data);

// Event Handler: "sendMessage"
const { roomId, text } = @DATA;

const message = await @REPOS.messagesRepo.create({
  data: {
    roomId: roomId,
    userId: @USER.id,
    text: text,
    createdAt: new Date()
  }
});

await @SOCKET.broadcast('newMessage', message.data[0]);
```

### Pattern 2: Notifications

```javascript
// Event Handler: "notifyUser"
const { targetUserId, type, payload } = @DATA;

// Save notification
await @REPOS.notificationsRepo.create({
  data: {
    userId: targetUserId,
    type: type,
    payload: payload,
    isRead: false,
    createdAt: new Date()
  }
});

// Send real-time notification to user
await @SOCKET.sendToUser(targetUserId, 'notification', {
  type: type,
  payload: payload
});
```

### Pattern 3: Live Updates

```javascript
// Event Handler: "updateStatus"
const { status } = @DATA;

// Update user status
await @REPOS.usersRepo.update({
  id: @USER.id,
  data: { status: status }
});

// Broadcast to all connected users
await @SOCKET.broadcast('userStatusChanged', {
  userId: @USER.id,
  status: status
});
```

### Pattern 4: Room-Based Communication

```javascript
// Event Handler: "joinRoom"
const { roomId } = @DATA;

// Validate room exists
const room = await @REPOS.roomsRepo.find({
  where: { id: { _eq: roomId } }
});

if (room.data.length === 0) {
  @throw['404']('Room not found');
  return;
}

// Add user to room
await @REPOS.roomMembersRepo.create({
  data: {
    roomId: roomId,
    userId: @USER.id,
    joinedAt: new Date()
  }
});

// Broadcast to room
await @SOCKET.broadcast('userJoinedRoom', {
  roomId: roomId,
  userId: @USER.id
});
```

## Troubleshooting

### Common Issues

1. **"Connection rejected"**
   - Check if `requireAuth: true` and token is valid
   - Verify token format (Bearer, query param, or cookie)

2. **"Event handler not executing"**
   - Check if event name matches definition
   - Verify `isEnabled: true` on event definition
   - Check handler script for syntax errors

3. **"Messages not reaching other instances"**
   - Verify Redis is running and configured
   - Check WebsocketRedisAdapter is properly initialized

4. **"sendToUser not working"**
   - Verify user ID is correct
   - Check if user is actually connected
   - Ensure Redis pub/sub is working for multi-instance

5. **"Connection handler timeout"**
   - Check `connectionHandlerTimeout` value
   - Optimize connection handler code
   - Move heavy operations to event handlers

## Client Integration

Once you've created WebSocket gateways and event handlers on the server, clients can connect using the Enfyra SDK.

### Nuxt SDK

Use the `useEnfyraWebSocket` composable:

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue';

// Get WebSocket instance with auto-reconnection
const ws = useEnfyraWebSocket('/chat', {
  reconnect: true,
  reconnectInterval: 3000,
  maxReconnectAttempts: 5
});

const handleMessage = (data) => {
  console.log('New message:', data);
};

onMounted(() => {
  ws.connect();
  ws.on('message', handleMessage);
  ws.on('open', () => console.log('Connected!'));
  ws.on('close', () => console.log('Disconnected'));
});

const sendMessage = (text: string) => {
  ws.send('message', { text });
};

onUnmounted(() => {
  ws.off('message', handleMessage);
  ws.close();
});
</script>
```

**See:** [useEnfyraWebSocket Documentation](../../../sdk-nuxt/docs/useEnfyraWebSocket.md)

### Message Format

**Client → Server:**
```json
{
  "event": "message",
  "data": {
    "text": "Hello!",
    "roomId": "123"
  }
}
```

**Server → Client:**
```json
{
  "event": "newMessage",
  "data": {
    "id": "abc",
    "text": "Hello!",
    "userId": "user123"
  }
}
```

### Client Events

Clients can listen for these system events:

| Event | Description |
|-------|-------------|
| `open` | Connection established |
| `close` | Connection closed |
| `error` | Connection error |
| `message` | Any message received |

### Authentication

The SDK automatically includes authentication if using `useEnfyraAuth`:

```typescript
// Token is automatically included
const ws = useEnfyraWebSocket('/chat');
ws.connect();
```

**Manual authentication:**
```typescript
// Via query parameter
const ws = useEnfyraWebSocket(`/chat?token=${token}`);
ws.connect();
```

## Next Steps

- [Repository Methods](./repository-methods/find.md) - Database operations
- [Context Reference](./context-reference/) - Complete $ctx documentation
- [API Lifecycle](./api-lifecycle.md) - How requests flow through the system
- [Hooks and Handlers](./hooks-handlers/) - Custom logic patterns

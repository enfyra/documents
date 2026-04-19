# WebSocket Integration

> **Prerequisite:** The WebSocket gateway must be enabled in Enfyra Admin before connecting. Contact your administrator to enable the gateway for your application.

## Connection

Connect to Enfyra WebSocket using Socket.IO client:

```typescript
import { io } from 'socket.io-client';

const socket = io(`${APP_URL}/ws/${GATEWAY_PATH}`, {
  transports: ['polling', 'websocket'],
  withCredentials: true,
});
```

**URL format:** `{APP_URL}/ws/{gatewayPath}`

- `APP_URL` - Enfyra App URL (e.g., `http://localhost:3000`, `https://your-app.enfyra.io`)
- `gatewayPath` - Configured gateway path (e.g., `chat`, `notifications`)

### Connection Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `transports` | string[] | `['polling', 'websocket']` | Transport methods |
| `withCredentials` | boolean | `true` | Send cookies with requests |
| `reconnect` | string | `'true'` | Auto reconnect when backend disconnects |

### Connection Events

```typescript
socket.on('connect', () => {
  console.log('Connected');
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
});

socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
});
```

### Relay Events

| Event | Description |
|-------|-------------|
| `backend_disconnected` | Backend disconnected, waiting to reconnect |
| `backend_reconnected` | Backend reconnected |

```typescript
socket.on('backend_disconnected', () => {
  console.log('Backend disconnected, waiting...');
});

socket.on('backend_reconnected', () => {
  console.log('Backend reconnected');
});
```

## Sending Messages

```typescript
socket.emit('message', {
  text: 'Hello!',
  roomId: '123',
});
```

### Recommended: ACK + async result events (better UX)

Enfyra runs WebSocket event handlers asynchronously (queued). For the best developer experience:

- Use Socket.IO **ack callback** to confirm the event is queued and receive a `requestId`.
- Listen for `ws:result` / `ws:error` to get the handler outcome (including script logs).

```typescript
socket.emit('message', { text: 'Hello!', roomId: '123' }, (ack) => {
  // ack = { queued: boolean, requestId: string, eventName: string, error?: { code, message } }
  console.log('queued?', ack.queued, 'requestId', ack.requestId);
});

socket.on('ws:result', (payload) => {
  // payload = { requestId, eventName, success: true, result: any, logs: any[] }
  console.log('ws result', payload);
});

socket.on('ws:error', (payload) => {
  // payload = { requestId, eventName, success: false, code, message, logs?: any[], details?: any }
  console.error('ws error', payload);
});
```

### Common Events

| Event | Description | Payload |
|-------|-------------|---------|
| `message` | Send chat message | `{ text: string, roomId: string }` |
| `typing` | User typing indicator | `{ roomId: string, isTyping: boolean }` |
| `joinRoom` | Join a room | `{ roomId: string }` |
| `leaveRoom` | Leave a room | `{ roomId: string }` |
| `updateStatus` | Update user status | `{ status: string }` |

## Receiving Messages

```typescript
socket.on('newMessage', (data) => {
  console.log('New message:', data);
});
```

### Common Server Events

| Event | Description | Payload |
|-------|-------------|---------|
| `connected` | Connection acknowledged | `{ message: string }` |
| `newMessage` | New chat message | `{ id: string, text: string, userId: string }` |
| `userJoined` | User joined room | `{ userId: string, name: string }` |
| `userLeft` | User left room | `{ userId: string }` |
| `notification` | Push notification | `{ type: string, payload: any }` |
| `ws:result` | Handler completed | `{ requestId, eventName, success: true, result, logs }` |
| `ws:error` | Handler error / queue error | `{ requestId, eventName, success: false, code, message, logs?, details? }` |

### Remove Listener

```typescript
// Remove specific listener
socket.off('message', handler);

// Remove all listeners for an event
socket.off('message');
```

## Message Format

### Client  Server

```typescript
socket.emit('message', {
  text: 'Hello World!',
  roomId: 'room-123'
});
```

### Server  Client

```json
{
  "event": "newMessage",
  "data": {
    "id": "abc",
    "text": "Hello World!",
    "userId": "user123"
  }
}
```

## Error Handling

### Connection Errors

```typescript
socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
});
```

### Disconnection

```typescript
socket.on('disconnect', (reason) => {
  // Reasons:
  // - "io server disconnect" - Server disconnected
  // - "io client disconnect" - Client disconnected
  // - "ping timeout"
  // - "transport close"
  // - "transport error"
});
```

### Reconnection

```typescript
const socket = io(url, {
  withCredentials: true,
  reconnection: true,
  reconnectionAttempts: 10,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
});

socket.on('reconnect', (attemptNumber) => {
  console.log('Reconnected after', attemptNumber, 'attempts');
});

socket.on('reconnect_failed', () => {
  console.error('Reconnection failed');
});
```

## Server-Side Handler Scripts (`@SOCKET` API)

Handler scripts run in an isolated sandbox. Use the `@SOCKET` template macro (preferred over `$ctx.$socket`).

### Available Methods

#### WS context (connection handler + event handler)

| Method | Description | Available in |
|--------|-------------|--------------|
| `@SOCKET.reply(event, data)` | Send to the triggering client only | connection + event |
| `@SOCKET.join(room)` | Join a room in the current namespace | connection + event |
| `@SOCKET.leave(room)` | Leave a room | connection + event |
| `@SOCKET.emitToUser(userId, event, data)` | Send to a specific user across all gateways | connection + event |
| `@SOCKET.emitToRoom(room, event, data)` | Send to a named room across all gateways | connection + event |
| `@SOCKET.emitToGateway(path, event, data)` | Broadcast to all connections on a namespace | connection + event |
| `@SOCKET.broadcast(event, data)` | Broadcast to all connections on all gateways | connection + event |
| `@SOCKET.disconnect()` | Force-disconnect the current socket from the gateway | **connection handler only** |

#### HTTP context (handler / hook)

Only `emitToUser`, `emitToRoom`, `emitToGateway`, and `broadcast` are available (no socket to reply/join/leave/disconnect).

### Handler Script Examples

**Join room** (event: `joinRoom`):

```javascript
const { roomId } = @BODY;
if (!roomId) @THROW.badRequest('roomId is required');
@SOCKET.join(`chat_${roomId}`);
@SOCKET.emitToRoom(`chat_${roomId}`, 'userJoined', { userId: @USER.id });
return { joined: roomId };
```

**Send message** (event: `message`):

```javascript
const { text, roomId } = @BODY;
const msg = await #message.create({
  data: { text, roomId, senderId: @USER.id }
});
@SOCKET.emitToRoom(`chat_${roomId}`, 'newMessage', msg);
return { sent: true };
```

**Push notification to a user** (event: `notify`):

```javascript
const { targetUserId, message } = @BODY;
@SOCKET.emitToUser(targetUserId, 'notification', {
  from: @USER.id,
  message,
  timestamp: Date.now(),
});
return { notified: true };
```

**Leave room** (event: `leaveRoom`):

```javascript
const { roomId } = @BODY;
@SOCKET.leave(`chat_${roomId}`);
@SOCKET.emitToRoom(`chat_${roomId}`, 'userLeft', { userId: @USER.id });
return { left: roomId };
```

**Kick user / reject connection** (event: `kickSelf` or connection handler):

```javascript
// Connection handler: reject if user is banned
const bannedResult = await #banned_user.find({
  filter: { userId: { _eq: @USER.id } },
  limit: 1,
});
if (bannedResult.data.length > 0) {
  @SOCKET.reply('kicked', { reason: 'You are banned' });
  @SOCKET.disconnect();
  return;
}
```

```javascript
// Event handler: notify a user that their account is suspended
// Note: @SOCKET.disconnect() is NOT available in event handlers — only in connection handlers.
const userResult = await #user_definition.find({
  filter: { id: { _eq: @USER.id } },
  fields: 'id,isSuspended',
  limit: 1,
});
const user = userResult.data[0];
if (user?.isSuspended) {
  @SOCKET.reply('suspended', { reason: 'Account suspended' });
  return;
}
```

**Connection handler** (`connectionHandlerScript`):

```javascript
@SOCKET.reply('connected', { message: 'Welcome!', userId: @USER.id });
@LOGS('user connected', @USER.id);
```

**From HTTP handler/hook** (e.g., order status update):

```javascript
const order = await #order.update({
  filter: { id: { _eq: @PARAMS.id } },
  body: { status: 'shipped' }
});
@SOCKET.emitToUser(order.userId, 'orderUpdate', { orderId: order.id, status: 'shipped' });
return order;
```

## Examples

### Chat Application

```typescript
import { io, Socket } from 'socket.io-client';

class ChatService {
  private socket: Socket;

  connect() {
    this.socket = io('http://localhost:3000/ws/chat', {
      transports: ['polling', 'websocket'],
      withCredentials: true,
    });

    this.socket.on('newMessage', (message) => {
      this.onNewMessage(message);
    });

    this.socket.on('userJoined', (data) => {
      this.onUserJoined(data);
    });
  }

  sendMessage(text: string, roomId: string) {
    this.socket.emit('message', { text, roomId });
  }

  joinRoom(roomId: string) {
    this.socket.emit('joinRoom', { roomId });
  }

  disconnect() {
    this.socket?.disconnect();
  }

  private onNewMessage(message: any) {
    console.log('New message:', message);
  }

  private onUserJoined(data: any) {
    console.log('User joined:', data);
  }
}
```

## Testing WebSocket handlers (recommended)

When developing WebSocket handler scripts, you can test them without building a client app.

### `POST /admin/test/run` (kind: `websocket_event`)

Send a test payload and a handler script; the server runs the script with a simulated `@SOCKET` and returns:
- `result`: the handler return value
- `logs`: script logs (`@LOGS(...)`)
- `emitted`: array of `{ method, args }` — each `@SOCKET` call captured (e.g. `{ method: 'reply', args: ['reply', { ok: true }] }`)

```bash
curl -X POST "$API_URL/admin/test/run" \
  -H "content-type: application/json" \
  -d '{
    "kind": "websocket_event",
    "gatewayPath": "/chat",
    "eventName": "message",
    "timeoutMs": 5000,
    "payload": { "text": "hello" },
    "script": " @LOGS(\"received\", @BODY); @SOCKET.reply(\"reply\", { ok: true }); return { ok: true }; "
  }'
```

### Vue 3 Composition API

```typescript
import { ref, onUnmounted } from 'vue';
import { io, Socket } from 'socket.io-client';

export function useWebSocket(gatewayPath: string) {
  const socket = ref<Socket | null>(null);
  const isConnected = ref(false);

  const connect = () => {
    socket.value = io(`${import.meta.env.VITE_APP_URL}/ws/${gatewayPath}`, {
      transports: ['polling', 'websocket'],
      withCredentials: true,
    });

    socket.value.on('connect', () => {
      isConnected.value = true;
    });

    socket.value.on('disconnect', () => {
      isConnected.value = false;
    });
  };

  const disconnect = () => {
    socket.value?.disconnect();
    socket.value = null;
    isConnected.value = false;
  };

  const emit = (event: string, data: any) => {
    socket.value?.emit(event, data);
  };

  const on = (event: string, callback: (data: any) => void) => {
    socket.value?.on(event, callback);
  };

  const off = (event: string, callback?: (data: any) => void) => {
    socket.value?.off(event, callback as any);
  };

  onUnmounted(() => {
    disconnect();
  });

  return { socket, isConnected, connect, disconnect, emit, on, off };
}

// Usage
const { isConnected, connect, emit, on } = useWebSocket('chat');

connect();

on('newMessage', (message) => {
  messages.value.push(message);
});

emit('message', { text: 'Hello!', roomId: '123' });
```

### React Hook

```typescript
import { useEffect, useRef, useState } from 'react';
import { io, Socket } from 'socket.io-client';

export function useWebSocket(gatewayPath: string) {
  const socketRef = useRef<Socket | null>(null);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    socketRef.current = io(`${process.env.REACT_APP_URL}/ws/${gatewayPath}`, {
      transports: ['polling', 'websocket'],
      withCredentials: true,
    });

    socketRef.current.on('connect', () => setIsConnected(true));
    socketRef.current.on('disconnect', () => setIsConnected(false));

    return () => {
      socketRef.current?.disconnect();
    };
  }, [gatewayPath]);

  const emit = (event: string, data: any) => {
    socketRef.current?.emit(event, data);
  };

  const on = (event: string, callback: (data: any) => void) => {
    socketRef.current?.on(event, callback);
  };

  const off = (event: string, callback?: (data: any) => void) => {
    socketRef.current?.off(event, callback as any);
  };

  return { isConnected, emit, on, off };
}

// Usage
function ChatComponent() {
  const { isConnected, emit, on, off } = useWebSocket('chat');

  useEffect(() => {
    const handleNewMessage = (message: any) => {
      setMessages(prev => [...prev, message]);
    };

    on('newMessage', handleNewMessage);

    return () => {
      off('newMessage', handleNewMessage);
    };
  }, [on, off]);

  return <div>{isConnected ? 'Connected' : 'Disconnected'}</div>;
}
```

## Best Practices

### Clean Up Listeners

```typescript
onUnmounted(() => {
  socket.off('newMessage');
  socket.disconnect();
});
```

### Validate Messages

```typescript
socket.on('newMessage', (data) => {
  if (!data.id || !data.text) {
    console.warn('Invalid message format');
    return;
  }

  addMessage(data);
});
```

### Debounce Rapid Events

```typescript
import { debounce } from 'lodash';

const emitTyping = debounce((roomId: string) => {
  socket.emit('typing', { roomId, isTyping: true });
}, 300);
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection fails | Check URL format, CORS settings |
| Messages not received | Verify event name matches server, listener registered before message arrives |
| Frequent disconnections | Check network stability, server load, increase ping timeout |

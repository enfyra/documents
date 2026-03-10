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
  roomId: '123'
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
| `error` | Server error | `{ message: string, code: string }` |

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

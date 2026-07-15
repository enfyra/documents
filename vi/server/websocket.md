---
slug: websocket
---

# Tích hợp WebSocket

> **Điều kiện trước:** WebSocket gateway phải được bật trong Enfyra Admin trước khi kết nối. Hãy liên hệ quản trị viên để bật gateway cho ứng dụng của bạn.

## Kết nối

Kết nối tới Enfyra WebSocket bằng Socket.IO client:

```typescript
import { io } from 'socket.io-client';

const socket = io(`/${GATEWAY_NAMESPACE}`, {
  path: '/socket.io',
  withCredentials: true,
});
```

**Định dạng URL cho app thứ ba:** kết nối tới Socket.IO namespace trên origin của app thứ ba và dùng đường dẫn Socket.IO transport cục bộ.

- `GATEWAY_NAMESPACE` - đường dẫn backend gateway đã cấu hình, bỏ dấu gạch chéo đầu trong string template (với gateway metadata path `/chat`, dùng `chat`).
- `path` - đường dẫn Socket.IO transport cục bộ. App thứ ba proxy `/socket.io/**` tới Enfyra app bridge `/ws/socket.io/**`.

Client của Enfyra app có thể dùng bridge tích hợp `io('/ws/chat', { path: '/ws/socket.io' })`. App thứ ba nên dùng `io('/chat', { path: '/socket.io' })` để namespace vẫn là `/chat` trong khi transport được proxy qua app thứ ba.

### Tuỳ chọn kết nối

| Tuỳ chọn | Kiểu | Mặc định | Mô tả |
|--------|------|---------|-------------|
| `transports` | string[] | `['polling', 'websocket']` | Các phương thức transport |
| `path` | string | `/socket.io` | Socket.IO transport endpoint trên app origin |
| `withCredentials` | boolean | `true` | Gửi cookie cùng request |
| `reconnect` | string | `'true'` | Tự kết nối lại khi backend ngắt kết nối |

### Sự kiện kết nối

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

### Sự kiện relay

| Sự kiện | Mô tả |
|-------|-------------|
| `backend_disconnected` | Backend đã ngắt, đang chờ kết nối lại |
| `backend_reconnected` | Backend đã kết nối lại |

```typescript
socket.on('backend_disconnected', () => {
  console.log('Backend disconnected, waiting...');
});

socket.on('backend_reconnected', () => {
  console.log('Backend reconnected');
});
```

## Gửi tin nhắn

```typescript
socket.emit('message', {
  text: 'Hello!',
  roomId: '123',
});
```

### Khuyến nghị: ACK + sự kiện kết quả bất đồng bộ (UX tốt hơn)

Enfyra chạy WebSocket event handler bất đồng bộ (qua hàng đợi). Để có trải nghiệm phát triển tốt nhất:

- Dùng Socket.IO **ack callback** để xác nhận sự kiện đã vào hàng đợi và nhận `requestId`.
- Lắng nghe `ws:result` / `ws:error` để nhận kết quả handler (bao gồm script log).

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

### Sự kiện thường dùng

| Sự kiện | Mô tả | Payload |
|-------|-------------|---------|
| `message` | Gửi tin nhắn chat | `{ text: string, roomId: string }` |
| `typing` | Chỉ báo người dùng đang nhập | `{ roomId: string, isTyping: boolean }` |
| `joinRoom` | Vào phòng | `{ roomId: string }` |
| `leaveRoom` | Rời phòng | `{ roomId: string }` |
| `updateStatus` | Cập nhật trạng thái người dùng | `{ status: string }` |

## Nhận tin nhắn

```typescript
socket.on('newMessage', (data) => {
  console.log('New message:', data);
});
```

### Sự kiện server thường dùng

| Sự kiện | Mô tả | Payload |
|-------|-------------|---------|
| `connected` | Kết nối đã được xác nhận | `{ message: string }` |
| `newMessage` | Tin nhắn chat mới | `{ id: string, text: string, userId: string }` |
| `userJoined` | Người dùng đã vào phòng | `{ userId: string, name: string }` |
| `userLeft` | Người dùng đã rời phòng | `{ userId: string }` |
| `notification` | Push notification | `{ type: string, payload: any }` |
| `ws:result` | Handler đã hoàn tất | `{ requestId, eventName, success: true, result, logs }` |
| `ws:error` | Lỗi handler / lỗi hàng đợi | `{ requestId, eventName, success: false, code, message, logs?, details? }` |

### Gỡ listener

```typescript
// Gỡ listener cụ thể
socket.off('message', handler);

// Gỡ mọi listener của một event
socket.off('message');
```

## Định dạng tin nhắn

### Client → Server

```typescript
socket.emit('message', {
  text: 'Hello World!',
  roomId: 'room-123'
});
```

### Server → Client

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

## Xử lý lỗi

### Lỗi kết nối

```typescript
socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
});
```

### Ngắt kết nối

```typescript
socket.on('disconnect', (reason) => {
  // Lý do:
  // - "io server disconnect" - Server đã ngắt kết nối
  // - "io client disconnect" - Client đã ngắt kết nối
  // - "ping timeout"
  // - "transport close"
  // - "transport error"
});
```

### Kết nối lại

```typescript
const socket = io(url, {
  path: '/socket.io',
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

## Script handler phía server (`@SOCKET` API)

Script handler chạy trong sandbox cô lập. Dùng macro template `@SOCKET` (được ưu tiên hơn `$ctx.$socket`).

### Phương thức có sẵn

#### WS context (connection handler + event handler)

| Phương thức | Mô tả | Có trong |
|--------|-------------|--------------|
| `@SOCKET.reply(event, data)` | Chỉ gửi tới client đã kích hoạt sự kiện | connection + event |
| `@SOCKET.join(room)` | Vào một phòng trong namespace hiện tại | connection + event |
| `@SOCKET.leave(room)` | Rời phòng | connection + event |
| `@SOCKET.emitToUser(userId, event, data)` | Gửi tới một người dùng cụ thể trên mọi gateway | connection + event |
| `@SOCKET.emitToRoom(path, room, event, data)` | Gửi tới phòng có tên trong gateway namespace cụ thể | HTTP, flow, connection + event |
| `@SOCKET.emitToCurrentRoom(room, event, data)` | Gửi tới phòng có tên trong gateway namespace hiện tại | connection + event |
| `@SOCKET.broadcastToRoom(room, event, data)` | Gửi tới phòng có tên trong gateway namespace hiện tại, trừ socket đã kích hoạt | connection + event |
| `@SOCKET.emitToGateway(path, event, data)` | Broadcast tới mọi kết nối của một namespace | connection + event |
| `@SOCKET.broadcast(event, data)` | Broadcast tới mọi kết nối trên mọi gateway | connection + event |
| `@SOCKET.disconnect()` | Buộc socket hiện tại ngắt khỏi gateway | **chỉ connection handler** |

#### HTTP context (handler / hook)

Chỉ có `emitToUser`, `emitToRoom`, `emitToGateway` và `broadcast` (không có socket để reply/join/leave/disconnect). HTTP handler, hook và flow step phải truyền gateway path vào `emitToRoom`.

### Ví dụ script handler

**Vào phòng** (event: `joinRoom`):

```javascript
const { roomId } = @BODY;
if (!roomId) @THROW400('roomId is required');
@SOCKET.join(`chat_${roomId}`);
@SOCKET.emitToCurrentRoom(`chat_${roomId}`, 'userJoined', { userId: @USER.id });
return { joined: roomId };
```

**Gửi tin nhắn** (event: `message`):

```javascript
const { text, roomId } = @BODY;
const msg = await #message.create({
  data: { text, roomId, senderId: @USER.id }
});
@SOCKET.emitToCurrentRoom(`chat_${roomId}`, 'newMessage', msg);
return { sent: true };
```

**Gửi push notification tới người dùng** (event: `notify`):

```javascript
const { targetUserId, message } = @BODY;
@SOCKET.emitToUser(targetUserId, 'notification', {
  from: @USER.id,
  message,
  timestamp: Date.now(),
});
return { notified: true };
```

**Rời phòng** (event: `leaveRoom`):

```javascript
const { roomId } = @BODY;
@SOCKET.leave(`chat_${roomId}`);
@SOCKET.emitToCurrentRoom(`chat_${roomId}`, 'userLeft', { userId: @USER.id });
return { left: roomId };
```

**Đẩy người dùng ra / từ chối kết nối** (event: `kickSelf` hoặc connection handler):

```javascript
// Connection handler: từ chối nếu người dùng bị cấm
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
// Event handler: báo cho người dùng biết tài khoản đã bị tạm ngưng
// Lưu ý: @SOCKET.disconnect() KHÔNG có trong event handler — chỉ có trong connection handler.
const userResult = await #enfyra_user.find({
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

**Từ HTTP handler/hook** (ví dụ cập nhật trạng thái đơn hàng):

```javascript
const order = await #order.update({
  filter: { id: { _eq: @PARAMS.id } },
  body: { status: 'shipped' }
});
@SOCKET.emitToUser(order.userId, 'orderUpdate', { orderId: order.id, status: 'shipped' });
return order;
```

## Ví dụ

### Ứng dụng chat

```typescript
import { io, Socket } from 'socket.io-client';

class ChatService {
  private socket: Socket;

  connect() {
    this.socket = io('/chat', {
      path: '/socket.io',
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

## Kiểm thử WebSocket handler (khuyến nghị)

Khi phát triển WebSocket handler script, bạn có thể kiểm thử mà không cần xây app client.

### `POST /admin/test/run` (kind: `websocket_event`)

Gửi payload kiểm thử và handler script; server chạy script với `@SOCKET` mô phỏng rồi trả về:
- `result`: giá trị handler trả về
- `logs`: script log (`@LOGS(...)`)
- `emitted`: mảng `{ method, args }` — mỗi lệnh gọi `@SOCKET` được ghi nhận (ví dụ `{ method: 'reply', args: ['reply', { ok: true }] }`)

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
    socket.value = io(`/${gatewayPath}`, {
      path: '/socket.io',
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

// Cách dùng
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
    socketRef.current = io(`/${gatewayPath}`, {
      path: '/socket.io',
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

// Cách dùng
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

## Thực hành tốt

### Dọn dẹp listener

```typescript
onUnmounted(() => {
  socket.off('newMessage');
  socket.disconnect();
});
```

### Xác thực tin nhắn

```typescript
socket.on('newMessage', (data) => {
  if (!data.id || !data.text) {
    console.warn('Invalid message format');
    return;
  }

  addMessage(data);
});
```

### Debounce sự kiện dồn dập

```typescript
import { debounce } from 'lodash';

const emitTyping = debounce((roomId: string) => {
  socket.emit('typing', { roomId, isTyping: true });
}, 300);
```

## Khắc phục sự cố

| Vấn đề | Cách xử lý |
|-------|----------|
| Không kết nối được | Kiểm tra định dạng URL và cấu hình CORS |
| Không nhận được tin nhắn | Kiểm tra event name khớp với server, listener đã đăng ký trước khi tin nhắn đến |
| Thường xuyên ngắt kết nối | Kiểm tra độ ổn định mạng, tải server và tăng ping timeout |

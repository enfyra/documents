---
slug: sdk/realtime
---

# Realtime

Dùng `WebSocketClient` hoặc framework helper `useWebSocket()` để kết nối third-party app với Socket.IO gateway của Enfyra.

## Điều kiện cần

Trước khi kết nối:

1. Tạo và bật gateway cùng event trong Enfyra.
2. Cấp quyền cho user mục tiêu.
3. Expose Socket.IO transport qua deployment nếu browser dùng same-origin bridge.
4. Xác định gateway name và event name ứng dụng cần dùng.

## Core client setup

```ts
import { WebSocketClient } from '@enfyra/sdk-core'

const realtime = new WebSocketClient({
  baseUrl: 'https://admin.example.com',
  gateway: 'notifications',
  getAuthToken: () => enfyra.auth.getToken(),
})
```

Mặc định:

- Socket.IO path: `/socket.io`
- Gửi credential: có
- Reconnect: bật
- Số lần reconnect tối đa: 5
- Khoảng reconnect: 1 giây

## Đăng ký listener trước khi kết nối

Listener đăng ký trước `connect()` được giữ lại và gắn khi socket được tạo:

```ts
const stopMessage = realtime.on('notification:created', (payload) => {
  console.log(payload)
})

realtime.on('connect', () => {
  console.log('Realtime connected')
})

realtime.on('connect_error', (error) => {
  console.error('Realtime failed', error)
})

await realtime.connect()
```

Unsubscribe function xóa một listener:

```ts
stopMessage()
```

## Emit event

```ts
realtime.emit('notification:read', {
  notificationId,
})
```

`emit()` chỉ gửi khi socket đang connected. Nếu sản phẩm cần đảm bảo delivery, dùng REST route làm durable command và dùng realtime để thông báo state mới cho client.

## Ngắt kết nối

```ts
realtime.disconnect()
```

Disconnect khi user logout, rời workspace tồn tại lâu hoặc application scope sở hữu connection bị hủy.

## Framework composable

Nuxt, Vue và React cung cấp state theo lifecycle:

```ts
const {
  connected,
  connecting,
  error,
  connect,
  disconnect,
  emit,
  on,
} = useWebSocket('notifications', { immediate: true })
```

Subscribe sau khi connection tồn tại:

```ts
await connect()

const stop = on('notification:created', (payload) => {
  notifications.value.unshift(payload)
})

onBeforeUnmount(stop)
```

Framework helper cũng disconnect khi component unmount.

## Xử lý sự cố

- `Not connected. Call connect() first.`: chờ `connect()` trước khi gọi `on()` trong framework helper.
- `connect_error` lặp lại: kiểm tra gateway access, transport proxy, origin, cookie và token expiry.
- Connect được nhưng không có event: kiểm tra event name và server có emit đến gateway/room/user mà client tham gia hay không.
- Event đến hai lần: đảm bảo component gỡ listener và không tạo connection trùng.
- Message mất khi disconnect: realtime không phải durable job queue; persist business command qua REST hoặc flow.

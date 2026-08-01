# Realtime

Use `WebSocketClient` or a framework `useWebSocket()` helper to connect a third-party application to an Enfyra Socket.IO gateway.

## Prerequisites

Before connecting:

1. Create and enable the gateway and its events in Enfyra.
2. Grant the intended user access.
3. Expose the Socket.IO transport through your deployment when the browser uses a same-origin bridge.
4. Know the gateway name and event names your application consumes.

## Core Client Setup

```ts
import { WebSocketClient } from '@enfyra/sdk-core'

const realtime = new WebSocketClient({
  baseUrl: 'https://admin.example.com',
  gateway: 'notifications',
  getAuthToken: () => enfyra.auth.getToken(),
})
```

Default behavior:

- Socket.IO path: `/socket.io`
- Credentials: included
- Reconnection: enabled
- Maximum reconnect attempts: 5
- Reconnect interval: 1 second

## Listen Before Connecting

Listeners registered before `connect()` are retained and attached when the socket is created:

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

The unsubscribe function removes one listener:

```ts
stopMessage()
```

## Emit an Event

```ts
realtime.emit('notification:read', {
  notificationId,
})
```

`emit()` sends only while the socket is connected. If your product must guarantee delivery, use a REST route as the durable command and use realtime only to notify clients of the resulting state.

## Disconnect

```ts
realtime.disconnect()
```

Disconnect when the user signs out, leaves a long-lived isolated workspace, or the owning application scope is destroyed.

## Framework Composables

Nuxt, Vue, and React provide lifecycle-aware state:

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

Subscribe after a connection exists:

```ts
await connect()

const stop = on('notification:created', (payload) => {
  notifications.value.unshift(payload)
})

onBeforeUnmount(stop)
```

The framework helper also disconnects on component unmount.

## Troubleshooting

- `Not connected. Call connect() first.`: wait for `connect()` before calling `on()` in the framework helper.
- Repeated `connect_error`: verify gateway access, transport proxy, origin, cookies, and token expiry.
- Connects but no events arrive: confirm the event name and that the server emits to the gateway/room/user your client joined.
- Events arrive twice: ensure each component removes its listener and does not create duplicate connections.
- Messages disappear during a disconnect: realtime emission is not a durable job queue; persist business commands through REST or a flow.

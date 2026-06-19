# Realtime Notifications

This example persists user notifications and emits a live event to the recipient.

## Table

Create `user_notification`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `recipient` | many-to-one relation to `enfyra_user` | Required |
| `title` | string | Required |
| `body` | text | Optional |
| `targetPath` | string | Optional |
| `readAt` | datetime | Nullable |

## Notify From A Handler

Inside any handler, create the notification and emit to the user.

```javascript
const created = await #user_notification.create({
  data: {
    recipient: { id: recipientId },
    title: 'Comment approved',
    body: 'Your comment is now visible.',
    targetPath: `/posts/${postId}`
  }
});

const notification = created.data?.[0] || null;
if (notification) {
  @SOCKET.emitToUser(recipientId, 'notification:new', notification);
}

return notification;
```

## Scope Reads

Add a `GET /user_notification` pre-hook.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { recipient: { id: { _eq: @USER.id } } }
  ]
};
```

## Browser Listener

```javascript
import { io } from 'socket.io-client';

const socket = io('/notifications', {
  path: '/socket.io',
  withCredentials: true
});

socket.on('notification:new', (notification) => {
  notifications.value.unshift(notification);
});
```

Create a Socket.IO gateway for `/notifications`. If the gateway requires auth, Enfyra loads the current user for event scripts and joins the user's server-side socket identity.

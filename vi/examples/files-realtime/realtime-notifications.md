---
slug: vi-du/file-va-realtime/thong-bao-realtime
---

# Thông báo realtime

Ví dụ này lưu thông báo của người dùng và phát một event trực tiếp đến người nhận.

## Table

Tạo `user_notification`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `recipient` | relation many-to-one tới `enfyra_user` | Bắt buộc |
| `title` | string | Bắt buộc |
| `body` | text | Không bắt buộc |
| `targetPath` | string | Không bắt buộc |
| `readAt` | datetime | Có thể để `null` |

## Gửi thông báo từ handler

Trong bất kỳ handler nào, hãy tạo thông báo rồi phát tới người dùng.

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

## Giới hạn dữ liệu đọc

Thêm pre-hook cho `GET /user_notification`.

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

## Listener trên trình duyệt

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

Tạo Socket.IO gateway cho `/notifications`. Nếu gateway yêu cầu xác thực, Enfyra tải người dùng hiện tại cho event script và tham gia socket identity phía server của người dùng đó.

---
slug: ung-dung/giam-sat-runtime
---

# Giám sát runtime

Giám sát runtime có tại **Cài đặt > Quản trị > Giám sát runtime**. Công cụ này giúp quản trị viên kiểm tra Enfyra Server đang chạy mà không cần rời ứng dụng quản trị.

## Các tab

- **Tổng quan**: chỉ số về tiến trình, request, cơ sở dữ liệu, hàng đợi, websocket và tình trạng hoạt động.
- **Redis**: mức sử dụng Redis của ứng dụng hiện tại, nhóm khóa, trình duyệt khóa và giá trị user cache có thể chỉnh sửa.

Màn hình giám sát cập nhật qua kết nối websocket. Tab Redis hiển thị thời gian đếm ngược đến lần chụp trạng thái máy chủ tiếp theo để bạn biết khi nào có chỉ số mới.

## Tổng quan Redis

Tab Redis chỉ đọc namespace của ứng dụng hiện tại. Enfyra tự giới hạn khóa theo `NODE_NAME` của backend, vì vậy một ứng dụng Enfyra không thể duyệt namespace Redis của ứng dụng khác qua giao diện này.

Các nhóm khóa có nhãn:

- **runtime cache**: bản chụp định nghĩa runtime do Enfyra quản lý. Chỉ đọc.
- **BullMQ**: trạng thái hàng đợi và job được giữ lại. Chỉ đọc.
- **Socket.IO**: trạng thái adapter websocket. Chỉ đọc.
- **runtime monitor**: dữ liệu theo dõi runtime. Chỉ đọc.
- **system lock**: khóa điều phối của Enfyra. Chỉ đọc.
- **user cache**: giá trị `$cache` / `@CACHE` do handler, hook, flow, script websocket hoặc Trình sửa khóa Redis tạo. Có thể chỉnh sửa.

## Bộ nhớ Redis được cấp

Bộ nhớ Redis được cấp là mức phân bổ mềm chỉ dành cho user cache. Giá trị này do `REDIS_USER_CACHE_LIMIT_MB` kiểm soát và mặc định là `30` MB.

Giới hạn không bao gồm các khóa hệ thống như runtime cache, BullMQ, Socket.IO, dữ liệu theo dõi runtime hoặc lock. Khi user cache vượt mức phân bổ, Enfyra loại bỏ các khóa user cache ít được dùng gần đây nhất.

## Trình duyệt khóa

Tìm kiếm dùng khóa logic mà người dùng nhìn thấy. Với user cache có thể chỉnh sửa, nhập cùng khóa bạn dùng trong mã:

```javascript
await @CACHE.set('user:123', user, 300000);
```

Trong Redis, khóa này được lưu dưới namespace của ứng dụng, nhưng giao diện ẩn namespace và tự thêm nó. Không nhập `NODE_NAME` hoặc `user_cache:` vào trường tìm kiếm hay chỉnh sửa.

Trình duyệt tải khóa theo từng trang 10 mục. Dùng **Tải thêm** để lấy trang tiếp theo thay vì quét toàn bộ namespace Redis cùng lúc.

## Trình sửa khóa

Chỉ khóa **user cache** mới có thể chỉnh sửa. Trình sửa dùng cùng service với `$cache` / `@CACHE`, vì vậy giá trị tạo trong giao diện có thể dùng trong script máy chủ và giá trị do script tạo cũng hiện trong giao diện.

Giá trị được phân tích để hiển thị khi có thể và chuyển thành chuỗi trước khi lưu. Khóa hệ thống chỉ đọc để tránh làm hỏng trạng thái runtime của Enfyra.

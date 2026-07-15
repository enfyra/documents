---
slug: may-chu-enfyra
---

# Tài liệu Enfyra Server

Tài liệu này bao quát kiến trúc, API và hướng dẫn phát triển của Enfyra server. Nội dung được tổ chức để giúp bạn tìm thông tin nhanh, dù bạn đang học những điều cơ bản hay tra cứu chi tiết cụ thể.

> **Mới dùng Enfyra?** Hãy bắt đầu với [Hướng dẫn cài đặt](../getting-started/installation.md) để thiết lập workspace app + server.

## Điều hướng nhanh

### Bắt đầu
- **[Phương thức Repository](repository-methods/find.md)** - Hướng dẫn đầy đủ về các thao tác cơ sở dữ liệu (find, create, update, delete)
- **[Tham chiếu Context](context-reference/request-data.md)** - Toàn bộ thuộc tính và phương thức có trong `$ctx`
- **[Vòng đời API](./api-lifecycle.md)** - Cách request đi qua hệ thống

### Khái niệm cốt lõi
- **[Hooks và Handlers](hooks-handlers/prehooks.md)** - Tạo preHooks, postHooks và handler tùy chỉnh
- **[Guards](./guards.md)** - Bảo vệ route theo khai báo (chặn IP, giới hạn tốc độ)
- **[Lọc truy vấn](./query-filtering.md)** - Toán tử lọc giống MongoDB và ví dụ
- **[Xử lý lỗi](./error-handling.md)** - Ném lỗi và xử lý ngoại lệ

### Bảo mật & kiểm soát truy cập
- **[Quyền trường](./field-permissions.md)** - Kiểm soát truy cập theo cột và quan hệ (mốc `isPublished` + quy tắc)
- **[Guards](./guards.md)** - Bảo vệ route theo khai báo (chặn IP, giới hạn tốc độ)

### Chủ đề nâng cao
- **[Hướng dẫn WebSocket](./websocket.md)** - Giao tiếp WebSocket thời gian thực
- **[Thao tác Cache](./cache-operations.md)** - Cache phân tán và khóa
- **[Xử lý tệp](./file-handling.md)** - Tải lên và quản lý tệp
- **[Kiến trúc Cluster](./cluster-architecture.md)** - Phối hợp nhiều instance
- **Quản trị Redis Runtime** - Có trong tab Redis của Runtime Monitor thuộc app quản trị; hiển thị các nhóm Redis của app hiện tại, phân bổ user-cache và các khóa `$cache` có thể chỉnh sửa
- **[Xem trước migration schema](./schema-migration-preview.md)** - `enfyra_table` PATCH, hash và cảnh báo xóa quan hệ đã được xác nhận

## Tìm nội dung bạn cần

### "Tôi muốn truy vấn dữ liệu từ một bảng"
 Xem [Tìm bản ghi](./repository-methods/find.md)

### "Tôi cần tạo một bản ghi mới"
 Xem [Tạo bản ghi](repository-methods/create-update-delete.md#create)

### "Tôi muốn cập nhật một bản ghi"
 Xem [Cập nhật bản ghi](repository-methods/create-update-delete.md#update)

### "Tôi cần xóa một bản ghi"
 Xem [Xóa bản ghi](repository-methods/create-update-delete.md#delete)

### "Những thuộc tính nào có trong $ctx?"
 Xem [Tham chiếu Context](context-reference/request-data.md)

### "Làm cách nào để truy cập request body và params?"
 Xem [Tham chiếu Context - Dữ liệu Request](./context-reference/request-data.md)

### "Làm cách nào để dùng repository trong mã của tôi?"
 Xem [Tham chiếu Context - Repositories](./context-reference/repositories.md)

### "Có những hàm helper nào?"
 Xem [Tham chiếu Context - Helpers & Cache](./context-reference/helpers-cache.md)

### "Vòng đời request hoạt động thế nào?"
 Xem [Vòng đời API](./api-lifecycle.md)

### "Các hook thực thi theo thứ tự nào?"
 Xem [Vòng đời API - Thứ tự thực thi](./api-lifecycle.md#execution-order)

### "Tôi muốn kiểm tra dữ liệu trước khi lưu"
 Xem [Hooks và Handlers - preHooks](./hooks-handlers/prehooks.md)

### "Tôi cần chỉnh sửa dữ liệu response"
 Xem [Hooks và Handlers - postHooks](./hooks-handlers/posthooks.md)

### "Tôi muốn viết logic nghiệp vụ tùy chỉnh"
 Xem [Hooks và Handlers - Custom Handlers](./hooks-handlers/custom-handlers.md)

### "Làm cách nào để lọc dữ liệu với điều kiện phức tạp?"
 Xem [Lọc truy vấn](./query-filtering.md)

### "Làm cách nào để ném lỗi đúng cách?"
 Xem [Xử lý lỗi](./error-handling.md)

### "Tôi muốn kiểm soát ai có thể xem các cột cụ thể"
 Xem [Quyền trường](./field-permissions.md)

### "Tôi muốn bảo vệ API của mình khỏi bị lạm dụng"
 Xem [Guards](./guards.md) để bảo vệ route theo khai báo, hoặc [Tham chiếu Context - Helpers & Cache](./context-reference/helpers-cache.md#rate-limiting) để giới hạn tốc độ bằng mã

### "Tôi cần dùng Redis cache"
 Xem [Thao tác Cache](./cache-operations.md)

### "Tôi muốn tải tệp lên"
 Xem [Xử lý tệp](./file-handling.md)

### "Tôi cần giao tiếp WebSocket thời gian thực"
 Xem [Hướng dẫn WebSocket](./websocket.md)

### "Repository hoạt động thế nào?"
 Xem [Phương thức Repository](repository-methods/find.md)

### "Repository có những phương thức nào?"
 Xem [Phương thức Repository](repository-methods/find.md) - Danh sách đầy đủ các phương thức find, create, update, delete

## Cấu trúc tài liệu

Toàn bộ tài liệu được tổ chức theo từng bước, với ví dụ và giải thích rõ ràng. Mỗi tài liệu tập trung vào một chủ đề cụ thể và gồm:

1. **Tổng quan** - Chủ đề nói về điều gì
2. **Hướng dẫn từng bước** - Cách sử dụng
3. **Ví dụ** - Ví dụ mã thực tế
4. **Tham chiếu** - Tham chiếu API đầy đủ
5. **Mẫu phổ biến** - Thực hành tốt và mẹo

## Tổng quan về phương thức Repository

Repository là cách chính để tương tác với các bảng cơ sở dữ liệu. Mỗi bảng bạn tạo tự động có một repository, có thể truy cập qua `$ctx.$repos.tableName`.

**Các phương thức có sẵn:**
- Find - Truy vấn bản ghi có lọc, sắp xếp và phân trang
- Create - Tạo bản ghi mới
- Update - Cập nhật bản ghi hiện có theo ID
- Delete - Xóa bản ghi theo ID

Mọi phương thức đều trả về dữ liệu theo định dạng nhất quán: `{ data: [...], meta: {...} }`

Xem [Hướng dẫn phương thức Repository](repository-methods/find.md) để biết đầy đủ chi tiết.

## Tổng quan về đối tượng Context

Đối tượng `$ctx` (context) có sẵn trong mọi hook và handler. Đối tượng này cung cấp quyền truy cập đến:

- **Dữ liệu Request**: `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- **Repositories**: `$ctx.$repos.tableName` để thao tác cơ sở dữ liệu
- **Helpers**: `$ctx.$helpers` cho JWT, bcrypt, thao tác tệp
- **Cache**: `$ctx.$cache` cho các thao tác Redis
- **Ghi log**: `$ctx.$logs()` để thêm log vào response
- **Xử lý lỗi**: `$ctx.$throw['400']()` để ném lỗi

Xem [Tham chiếu Context](context-reference/request-data.md) để biết đầy đủ chi tiết.

## Tổng quan về vòng đời API

Mỗi API request đi theo luồng sau:

1. **Phát hiện Route** - Hệ thống khớp request với định nghĩa route
2. **Guards trước xác thực** - Chặn IP, giới hạn tốc độ toàn cục
3. **Thiết lập Context** - Tạo `$ctx` với repository và helper
4. **Xác thực** - Xác minh JWT và kiểm tra role
5. **Guards sau xác thực** - Giới hạn tốc độ theo người dùng
6. **Thực thi preHooks** - Chạy tuần tự mọi preHook khớp
7. **Thực thi Handler** - Handler tùy chỉnh hoặc thao tác CRUD mặc định
8. **Thực thi postHooks** - Chạy tuần tự mọi postHook khớp
9. **Response** - Trả về dữ liệu đã xử lý

Cùng một đối tượng `$ctx` đi qua mọi giai đoạn, nên các thay đổi trong preHooks sẽ hiển thị cho handler và postHooks.

Xem [Vòng đời API](./api-lifecycle.md) để biết đầy đủ chi tiết.

## Lộ trình học

**Mới dùng Enfyra Server?** Hãy theo lộ trình từng bước này:

1. **[Phương thức Repository](repository-methods/find.md)** - Bắt đầu tại đây! Học cách truy vấn, tạo, cập nhật và xóa bản ghi
2. **[Tham chiếu Context](context-reference/request-data.md)** - Hiểu mọi thuộc tính và phương thức có trong `$ctx`
3. **[Vòng đời API](./api-lifecycle.md)** - Tìm hiểu cách request đi qua hệ thống
4. **[Hooks và Handlers](hooks-handlers/prehooks.md)** - Tùy chỉnh hành vi API bằng hook và handler
5. **[Guards](./guards.md)** - Bảo vệ route bằng chặn IP và giới hạn tốc độ
6. **[Lọc truy vấn](./query-filtering.md)** - Thành thạo việc lọc và truy vấn dữ liệu
7. **[Xử lý lỗi](./error-handling.md)** - Xử lý lỗi đúng cách

**Chủ đề nâng cao:**
- [Thao tác Cache](./cache-operations.md) - Cache phân tán và khóa
- [Xử lý tệp](./file-handling.md) - Tải lên và quản lý tệp
- [Kiến trúc Cluster](./cluster-architecture.md) - Phối hợp nhiều instance

## Bước tiếp theo

1. Bắt đầu với [Phương thức Repository](repository-methods/find.md) để tìm hiểu các thao tác cơ sở dữ liệu
2. Đọc [Tham chiếu Context](context-reference/request-data.md) để hiểu các thuộc tính có sẵn
3. Xem [Vòng đời API](./api-lifecycle.md) để biết mọi thành phần kết hợp với nhau thế nào
4. Khám phá [Hooks và Handlers](hooks-handlers/prehooks.md) để tùy chỉnh

Với câu hỏi cụ thể, hãy dùng phần Điều hướng nhanh ở trên để chuyển thẳng đến tài liệu liên quan.

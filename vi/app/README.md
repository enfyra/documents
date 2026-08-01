---
slug: ung-dung
---

# Enfyra App và Extension

Phần này hướng dẫn UI và extension chạy bên trong **Enfyra Admin**: page, widget, form, permission, menu, header action và operator workflow.

Nếu bạn đang xây web app, backend cho mobile app, CLI hoặc service được deploy riêng, hãy dùng [tài liệu SDK](../sdk/README.md). SDK app và Enfyra Admin extension có runtime API khác nhau.

Đây là một **API client**: ứng dụng không bao giờ kết nối trực tiếp với cơ sở dữ liệu. Mọi dữ liệu đều đến từ **Máy chủ Enfyra (cổng 1105)** qua HTTP API.

> **Mới sử dụng Enfyra?** Bắt đầu với [Hướng dẫn cài đặt](../getting-started/installation.md) để thiết lập không gian làm việc cho ứng dụng và máy chủ của bạn.

## Điều hướng nhanh

- **Tích hợp API trong extension**
  - **[Tích hợp API](./api-integration.md)** – Gọi backend API từ page, widget và extension chạy bên trong Enfyra Admin

- **Tiện ích mở rộng & Widget**  
  - **[Hệ thống tiện ích mở rộng](./extension-system.md)** – Tạo trang, widget tùy chỉnh và tiện ích mở rộng shell toàn cục bằng component Vue  
  - **[Hành động tiêu đề](./header-actions.md)** – Đưa các nút tùy chỉnh vào tiêu đề ứng dụng và tiêu đề phụ

- **Biểu mẫu & nhập dữ liệu**  
  - **[Hệ thống biểu mẫu](./form-system.md)** – Biểu mẫu tạo tự động, validation, relation và theo dõi thay đổi  
  - **[Bộ chọn relation](./relation-picker.md)** – Chọn bản ghi liên quan trong biểu mẫu  
  - **[Hệ thống lọc](./filter-system.md)** – Bộ lọc nâng cao cho bảng và bộ chọn

- **Quyền và khả năng hiển thị**  
  - **[Trình dựng quyền](./permission-builder.md)** – Tạo quy tắc phân quyền trực quan  
  - **[Thành phần quyền](./permission-components.md)** – Component `PermissionGate` và composable `usePermissions`

- **Xác thực**
  - **[Quy tắc cột](./column-rules.md)** – Đính kèm các ràng buộc tối thiểu/tối đa, độ dài, mẫu, định dạng vào các cột; các quy tắc được thực thi phía máy chủ trên POST/PATCH

- **Điều hướng & Bố cục**  
  - **[Quản lý menu](./menu-management.md)** – Cấu hình thanh bên và menu  
  - **[Quản lý phương thức](./method-management.md)** – Bản ghi HTTP method và màu badge dùng trong biểu mẫu route
  - **[Tiêu đề trang](./page-header.md)** – Tiêu đề trang có số liệu và nền chuyển sắc

- **Móc, Trình xử lý & Gói (phía giao diện người dùng)**  
  - **[Hooks](hooks-handlers/hooks.md)** – Quản lý hook từ giao diện người dùng ứng dụng  
  - **[Trình xử lý tùy chỉnh](hooks-handlers/custom-handlers.md)** – Quản lý trình xử lý tùy chỉnh từ giao diện người dùng ứng dụng  
  - **[Quản lý package](hooks-handlers/package-management.md)** – Cài package NPM cho hook và handler

- **Bộ nhớ & Tệp**  
  - **[Quản lý lưu trữ](./storage-management.md)** – Tải lên tệp, thư mục và cấu hình lưu trữ

- **Hoạt động**
  - **[Nhật ký máy chủ](./log-viewing.md)** – Đọc và theo dõi log backend từ ứng dụng quản trị
  - **[Theo dõi runtime](./runtime-monitor.md)** – Theo dõi tiến trình, database, queue, websocket và Redis

## Lộ trình học cho Enfyra App

Nếu bạn đang **xây dựng trên giao diện người dùng ứng dụng Enfyra**, thì đây là thứ tự được đề xuất:

1. **Hiểu kiến trúc**  
   - **[Tổng quan về kiến trúc](../architecture-overview.md)** – Cách giao diện người dùng và phụ trợ phối hợp với nhau  
   - **[Tổng quan về máy chủ](../server/README.md)** – Các khái niệm và API phía máy chủ

2. **Làm việc với dữ liệu từ ứng dụng**  
   - **[Hệ thống biểu mẫu](./form-system.md)** – Cách tạo biểu mẫu từ bảng  
   - **[Hệ thống lọc](./filter-system.md)** – Lọc dữ liệu trong bảng và bộ chọn relation  
   - **[Tích hợp API](./api-integration.md)** – Gọi API từ trang, tiện ích mở rộng và widget

3. **Xây dựng giao diện người dùng và quy trình làm việc tùy chỉnh**  
   - **[Hệ thống tiện ích mở rộng](./extension-system.md)** – Tạo trang, widget và tiện ích mở rộng shell dùng chung  
   - **[Hành động tiêu đề](./header-actions.md)** – Thêm hành động tùy chỉnh vào tiêu đề  
   - **[Quản lý menu](./menu-management.md)** – Kết nối tiện ích mở rộng với menu

4. **Bảo mật và kiểm soát giao diện người dùng**  
   - **[Trình dựng quyền](./permission-builder.md)** – Xác định quy tắc phân quyền  
   - **[Thành phần quyền](./permission-components.md)** – Ẩn hoặc hiện giao diện theo quyền  
   - **[Trình dựng quyền](./permission-builder.md)** – Đánh giá quyền backend (xem tài liệu này để biết chi tiết)

5. **Đi sâu hơn vào tùy chỉnh phụ trợ (từ giao diện người dùng ứng dụng)**  
   - **[Hooks](hooks-handlers/hooks.md)** – Định cấu hình các hook chạy trên máy chủ  
   - **[Trình xử lý tùy chỉnh](hooks-handlers/custom-handlers.md)** – Ghi đè hành vi CRUD mặc định  
   - **[Quản lý package](hooks-handlers/package-management.md)** – Cài package NPM cho hook/handler  
   - **[Hook và handler phía máy chủ](../server/hooks-handlers/README.md)** – Hành vi đầy đủ ở backend

## Điều này liên quan như thế nào đến Tài liệu máy chủ

- **Tài liệu máy chủ** (`../server/README.md`) mô tả **những API tồn tại và cách chúng hoạt động**.  
- **Tài liệu ứng dụng** (thư mục này) mô tả **cách ứng dụng quản trị viên sử dụng các API đó** thông qua biểu mẫu, bảng, tiện ích mở rộng và quyền.
- **Tài liệu SDK** (`../sdk/README.md`) mô tả **cách third-party app được deploy riêng dùng Enfyra làm backend**.

Khi nghi ngờ:

- Nếu bạn hỏi **“Tôi có thể sử dụng điểm cuối/trường/bộ lọc nào?”** – hãy truy cập **tài liệu máy chủ**.  
- Nếu bạn hỏi **“Làm cách nào để hiển thị thông tin này trong giao diện người dùng / tiện ích mở rộng?”** – hãy ở trong **tài liệu ứng dụng**.
- Nếu bạn hỏi **“Nuxt, Vue, React, Node.js hoặc service riêng của tôi gọi Enfyra thế nào?”** – hãy đọc **tài liệu SDK**.

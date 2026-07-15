---
slug: ung-dung/hook-va-handler/hook
---

# Hệ thống móc

Móc là một tính năng mạnh mẽ cho phép bạn chèn mã tùy chỉnh vào các điểm cụ thể trong vòng đời yêu cầu API. Thay vì tạo [Trình xử lý tùy chỉnh](./custom-handlers.md) đầy đủ, bạn có thể sử dụng móc để sửa đổi yêu cầu và phản hồi với mã tối thiểu.

**Để hiểu đầy đủ về vòng đời, hãy xem [Vòng đời API](../../server/api-lifecycle.md)**

## Hook là gì?

Hook thực thi mã JavaScript tại hai thời điểm quan trọng:
- **PreHook**: Chạy **trước** thao tác chính (trước các thao tác cơ sở dữ liệu)
- **postHook**: Chạy **sau** thao tác chính (sau thao tác cơ sở dữ liệu)

**Ưu điểm chính**: Các móc chia sẻ cùng một ngữ cảnh trong toàn bộ vòng đời yêu cầu, khiến chúng trở nên hoàn hảo cho các sửa đổi nhẹ mà không cần thay thế toàn bộ trình xử lý.

## Khi nào nên sử dụng Hook và Handler

### Sử dụng Hook Khi:
- **Sửa đổi dữ liệu yêu cầu**: Chuyển đổi đầu vào trước khi vận hành cơ sở dữ liệu
- **Thêm xác thực**: Kiểm tra dữ liệu trước khi xử lý
- **Sửa đổi bộ lọc truy vấn**: Thay đổi bộ lọc theo chương trình  
- **Chuyển đổi phản hồi**: Định dạng dữ liệu đầu ra sau các thao tác
- **Thêm ghi nhật ký**: Theo dõi hoạt động mà không thay đổi logic cốt lõi
- **Tự động tạo trường**: Tạo sên, mật khẩu băm, đặt dấu thời gian

### Sử dụng Trình xử lý tùy chỉnh khi:
- **Logic nghiệp vụ phức tạp**: Thao tác nhiều bước trên nhiều bảng
- **Thay thế toàn bộ thao tác**: Hoạt động hoàn toàn khác với CRUD
- **Cuộc gọi API bên ngoài**: Tích hợp của bên thứ ba yêu cầu quy trình làm việc phức tạp

## Đối tượng bối cảnh $ctx

Hook có quyền truy cập vào một đối tượng ngữ cảnh được chia sẻ (`$ctx`) tồn tại trong toàn bộ vòng đời yêu cầu API. Ngữ cảnh này chứa dữ liệu yêu cầu, kho lưu trữ cơ sở dữ liệu, chức năng trợ giúp và thông tin người dùng.

**Để biết thông tin chi tiết về vòng đời và tài liệu $ctx đầy đủ, hãy xem [Vòng đời API](../../server/api-lifecycle.md#chia-se-context)**

### Thuộc tính ngữ cảnh chính có sẵn trong Hook
```javascript
$ctx = {
  // Request data
  $body: {},           // Request body (can be modified)
  $params: {},         // URL parameters (/users/:id)
  $query: {},          // Query parameters (?limit=10)
  $user: {},           // Current authenticated user

  // Database access
  $repos: {            // Auto-generated repositories
    main: repository,  // Main table repository
    users: repository, // Named repositories for target tables
  },

  // NPM packages (installed via Package Management)
  $pkgs: {             // Access to installed NPM packages
    axios: axios,      // HTTP client library
    lodash: lodash,    // Utility functions
    moment: moment,    // Date manipulation
  },
  
  // Utilities  
  $helpers: {          // Helper functions
    $jwt: function,    // Create JWT tokens
    $bcrypt: object,   // Hash/compare passwords
    autoSlug: function // Generate URL slugs
  },
  
  // Logging & sharing
  $logs: function,     // Add logs to response
  $share: {            // Share data between hooks
    $logs: []
  }
};
```
**Quan trọng:** Đối tượng `$ctx` là **cùng một tham chiếu** trên tất cả các hook và trình xử lý trong một yêu cầu. Những thay đổi được thực hiện trong preHooks sẽ ảnh hưởng đến postHooks và các trình xử lý.

## Tạo móc

### Bước 1: Truy cập quản lý Hooks
1. Điều hướng đến **Cài đặt > Tuyến đường**
2. Mở Route cần chạy hook
3. Trong **Quy trình thực thi**, nhấp vào **Thêm Pre-Hook** hoặc **Thêm Post-Hook**

### Bước 2: Cấu hình Hook
Bạn sẽ thấy biểu mẫu tạo hook với các trường sau:

- **Name**: Tên mô tả cho hook
- **PreHook**: Mã JavaScript chạy trước thao tác
- **postHook**: Mã JavaScript chạy sau thao tác  
- **Ưu tiên**: Thứ tự thực hiện (0 = mức độ ưu tiên cao nhất)
- **IsEnabled**: Bật tắt hook
- **Mô tả**: Tài liệu cho mục đích của hook
- **Route**: Tuyến đường hiện tại được chọn tự động khi bạn tạo hook từ trang chi tiết tuyến đường
- **Phương thức**: Chọn bản ghi phương thức HTTP sẽ kích hoạt hook này. Các phương thức tích hợp sẵn là `GET`, `POST`, `PATCH` và `DELETE`; trước tiên phải tạo các phương thức tùy chỉnh như `PUT` trong **Cài đặt > Phương thức**.

### Bước 3: Chọn Phương thức
- **Lựa chọn phương thức**: Chọn phương thức HTTP nào kích hoạt hook này
- **Nhiều phương thức**: Một hook có thể áp dụng cho nhiều phương thức HTTP trên cùng một tuyến

## Viết mã hook

### Sử dụng gói NPM trong Hooks

Hook có toàn quyền truy cập vào các gói NPM được cài đặt thông qua [Package Management](./package-management.md):
```javascript
// PreHook example - Using lodash to validate data
const _ = $ctx.$pkgs.lodash;

if (!_.isObject($ctx.$body) || _.isEmpty($ctx.$body)) {
  throw new Error('Request body must be a non-empty object');
}

// postHook example - Using moment for timestamps
const moment = $ctx.$pkgs.moment;

($ctx.$data?.data || []).forEach(record => {
  record.formatted_date = moment(record.created_at).format('YYYY-MM-DD HH:mm:ss');
});
```
**Hướng dẫn gói hoàn chỉnh**: Xem [Quản lý gói](./package-management.md) để cài đặt và sử dụng các gói NPM.

### Phát triển móc nâng cao

Để biết thông tin chi tiết về cách viết mã JavaScript trong hook, bao gồm đối tượng ngữ cảnh (`$ctx`), các hàm có sẵn và ví dụ toàn diện, hãy xem:

**[Hướng dẫn về Hooks và Handlers](../../server/hooks-handlers/)** - Hướng dẫn đầy đủ về preHooks và postHooks

Điều này bao gồm:
- Bối cảnh móc và các biến có sẵn
- Ví dụ về mã PreHook và postHook
- Chức năng truy cập cơ sở dữ liệu và tiện ích
- Luồng thực thi và hệ thống ưu tiên
- Các phương pháp hay nhất và kỹ thuật gỡ lỗi

## Ví dụ thực tế

**[Ví dụ đăng ký người dùng](../../examples/user-registration-example.md)** - Xem các hook hoạt động với email chào mừng postHook bằng gói gật đầu.

Móc cung cấp sự cân bằng hoàn hảo giữa tính đơn giản và sức mạnh, cho phép bạn tùy chỉnh hành vi API mà không gặp sự phức tạp của trình xử lý tùy chỉnh đầy đủ.

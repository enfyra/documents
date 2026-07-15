---
slug: vong-doi-api
---

# Vòng đời API

Hiểu cách request API đi qua Enfyra sẽ giúp bạn xây dựng hook và handler hiệu quả. Hướng dẫn này trình bày trọn vẹn vòng đời request và cách đối tượng context đi qua từng giai đoạn.

## Điều hướng nhanh

- [Tổng quan luồng request](#tong-quan-luong-request) - Toàn cảnh vòng đời
- [Các giai đoạn](#cac-giai-doan) - Giải thích chi tiết từng giai đoạn
- [Chia sẻ context](#chia-se-context) - Cách $ctx đi qua các giai đoạn
- [Thứ tự thực thi](#thu-tu-thuc-thi) - Trình tự chạy hook và handler
- [Mẫu thường dùng](#mau-thuong-dung) - Ví dụ thực tế

## Tổng quan luồng request

Mọi request API trong Enfyra đều theo vòng đời này:

```
HTTP request  Nhận diện route  Guard trước auth  Thiết lập context  JWT auth  Kiểm tra role  Guard sau auth  preHook  Handler  postHook  Response
```

### Luồng trực quan

```
1. Client gửi HTTP request
   ↓
2. Hệ thống nhận diện route và khớp với định nghĩa route
   ↓
3. Guard trước auth chạy (chặn IP, rate limit toàn cục)
   ↓                          ↓ (bị từ chối)
4. Tạo context ($ctx)           Trả về 403/429
   ↓
5. Xác thực JWT
   ↓                          ↓ (không có token)
6. Kiểm tra role/permission     Trả về 401
   ↓                          ↓ (không có quyền)
7. Guard sau auth chạy          Trả về 403
   ↓                          ↓ (bị từ chối)
8. Chạy mọi preHook phù hợp     Trả về 403/429
   ↓                          ↓ (có lỗi)
9. Handler chạy                 Bỏ qua handler
   ↓                          ↓
10. postHook chạy               postHook vẫn chạy (`@ERROR` có dữ liệu)
   ↓                          ↓
11. Gửi response                Throw lại lỗi gốc
```

## Các giai đoạn

### Giai đoạn 1: Nhận diện route

Hệ thống tự đối chiếu request đến với một định nghĩa route.

**Diễn ra như sau:**
- Phân tích URL và method của request
- Kiểm tra route cache để tìm route khớp
- Nạp định nghĩa route từ database
- Xác định các bảng đích
- Tìm các hook và handler liên quan

**Tự động vận hành:**
- Không cần cấu hình thủ công
- Route được định nghĩa trong database
- Hệ thống đảm nhiệm toàn bộ logic đối chiếu

### Giai đoạn 2: Guard trước xác thực

Guard có `position: pre_auth` chạy trước xác thực. Lúc này chỉ có địa chỉ IP của client.

**Diễn ra như sau:**
- Kiểm tra quy tắc whitelist/blacklist IP
- Áp dụng giới hạn tần suất theo IP và route
- Request bị từ chối trả về 403 (quy tắc IP) hoặc 429 (giới hạn tần suất)

**Xem [Guards](./guards.md) để biết cách cấu hình.**

### Giai đoạn 3: Khởi tạo context

Đối tượng `$ctx` (context) được tạo và khởi tạo.

**Bao gồm:**
- Dữ liệu request: `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- Repository: `$ctx.$repos` cho mọi bảng đích
- Helper: `$ctx.$helpers` cho tiện ích như JWT, bcrypt
- Cache: `$ctx.$cache` cho thao tác Redis
- Ghi log: hàm `$ctx.$logs()`
- Xử lý lỗi: các method `$ctx.$throw`

**Lưu ý:** Cùng một tham chiếu đối tượng `$ctx` được dùng xuyên suốt vòng đời request.

### Giai đoạn 4: Xác thực và phân quyền

Xác thực JWT và kiểm soát truy cập theo vai trò.

**Diễn ra như sau:**
- Xác minh JWT token
- Nạp user cùng vai trò của họ
- Kiểm tra quyền của route (method public bỏ qua xác thực)
- Root admin vượt qua mọi kiểm tra quyền

### Giai đoạn 5: Guard sau xác thực

Guard có `position: post_auth` chạy sau xác thực. Cả IP client và ID user đều đã có sẵn.

**Diễn ra như sau:**
- Áp dụng giới hạn tần suất riêng cho user
- Đánh giá quy tắc kết hợp IP + user
- Request bị từ chối trả về 403 hoặc 429

**Xem [Guards](./guards.md) để biết cách cấu hình.**

### Giai đoạn 6: Thực thi preHook

Mọi preHook khớp đều chạy tuần tự trước handler.

**Thứ tự thực thi:**
1. preHook global (mọi route, mọi method)
2. preHook global (mọi route, một method cụ thể)
3. preHook của route (một route, mọi method)
4. preHook của route (một route, một method cụ thể)

**preHook có thể:**
- Kiểm tra dữ liệu request
- Biến đổi dữ liệu đầu vào
- Kiểm tra quyền
- Sửa `$ctx.$body` hoặc `$ctx.$query`
- Lưu dữ liệu vào `$ctx.$share` để dùng về sau
- Ném lỗi để dừng thực thi

**Ví dụ:**
```javascript
// preHook: Validate and transform
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$share.validationPassed = true;
```

### Giai đoạn 7: Thực thi handler

Handler thực thi logic nghiệp vụ chính.

**Các loại handler:**
- **Custom Handler**: Mã tùy biến trong định nghĩa route
- **Default CRUD**: Thao tác CRUD tự động dựa trên HTTP method

**Handler có thể:**
- Dùng repository để query/tạo/cập nhật/xóa dữ liệu
- Truy cập toàn bộ thuộc tính `$ctx` do preHook thay đổi
- Trả về dữ liệu để afterHook dùng qua `$ctx.$data`
- Ném lỗi

**Ví dụ:**
```javascript
// Custom handler
const result = await $ctx.$repos.products.create({
  data: $ctx.$body
});

return result;
```

### Giai đoạn 8: Thực thi postHook

Mọi postHook khớp đều chạy tuần tự. **postHook luôn chạy**, kể cả khi preHook hoặc handler ném lỗi.

**Thứ tự thực thi:**
1. postHook global (mọi route, mọi method)
2. postHook global (mọi route, một method cụ thể)
3. postHook của route (một route, mọi method)
4. postHook của route (một route, một method cụ thể)

**Khi thành công:**
- `@DATA` chứa kết quả của handler
- `@STATUS` là `200`
- `@ERROR` là `undefined`

**Khi có lỗi:**
- `@DATA` là `null`
- `@STATUS` là mã trạng thái lỗi, ví dụ `400`, `500`
- `@ERROR` chứa `{ message, name, statusCode, details, timestamp }`
- Lỗi gốc **luôn được ném lại** sau khi mọi postHook hoàn tất
- Nếu một postHook thất bại, các postHook khác vẫn chạy

**postHook có thể:**
- Biến đổi dữ liệu response khi thành công
- Ghi audit log khi thành công lẫn khi lỗi
- Kích hoạt tác vụ phụ như email hoặc notification
- Ghi log/theo dõi lỗi qua `@ERROR`

**Ví dụ:**
```javascript
// postHook: Audit logging on both success and error
await #audit_logs.create({
  data: {
    action: `${@API.request.method} ${@API.request.url}`,
    userId: @USER?.id,
    statusCode: @STATUS,
    error: @ERROR ? @ERROR.message : null,
    timestamp: new Date()
  }
});
```

### Giai đoạn 9: Trả response

Response đã được xử lý được gửi về client.

**Response bao gồm:**
- Dữ liệu từ handler, có thể đã được postHook thay đổi
- Log thu thập từ mọi giai đoạn
- Mã trạng thái HTTP
- Header

## Chia sẻ context

Đối tượng `$ctx` là **cùng một tham chiếu** trong toàn bộ vòng đời request. Điều này có nghĩa:

### Tham chiếu xuyên suốt

Thay đổi ở một giai đoạn sẽ hiển thị trong mọi giai đoạn tiếp theo:

```javascript
// preHook #1 modifies context
$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$share.validationPassed = true;

// preHook #2 sees the changes
$ctx.$logs(`Email normalized: ${$ctx.$body.email}`);
$ctx.$logs(`Validation: ${$ctx.$share.validationPassed}`);

// Handler also sees all changes
const email = $ctx.$body.email;  // Already normalized
if ($ctx.$share.validationPassed) {
  // Validation already passed
}

// postHook gets final state
$ctx.$data.email = $ctx.$body.email;  // Use normalized email
```

### Các thuộc tính context có sẵn

Trong mọi giai đoạn, bạn có thể truy cập:

```javascript
// Request data (immutable after setup)
$ctx.$req          // Express request object
$ctx.$user         // Authenticated user

// Request data (mutable)
$ctx.$body         // Request body - can be modified in preHooks
$ctx.$params       // URL parameters
$ctx.$query        // Query string parameters

// Repositories (available after context setup)
$ctx.$repos        // Table repositories

// Utilities
$ctx.$helpers      // Helper functions
$ctx.$cache        // Cache operations
$ctx.$logs()       // Logging function
$ctx.$throw        // Error throwing

// Response data (available in postHook)
$ctx.$data         // Response data from handler (null on error)
$ctx.$statusCode   // HTTP status code (200 on success, error code on failure)
$ctx.$error        // Error context (undefined on success, {message, name, statusCode, details, timestamp} on error)

// Shared context (persists across all phases)
$ctx.$share        // Shared data container

// API information (available in postHook)
$ctx.$api          // Request/response/error details
```

## Thứ tự thực thi

Hook chạy theo thứ tự có thể dự đoán dựa trên phạm vi và bộ lọc method.

### Logic lọc hook

Một hook sẽ chạy nếu khớp một trong các điều kiện sau:
- Hook global không lọc method (chạy trên mọi route, mọi method)
- Hook global có lọc method (chạy trên mọi route, một method cụ thể)
- Hook của route không lọc method (chạy trên một route, mọi method)
- Hook của route có lọc method (chạy trên một route, một method cụ thể)

### Ví dụ thực thi

Với request `POST /users` có các hook sau:

1. preHook global (mọi method)
2. preHook global (chỉ POST)
3. preHook của route `/users` (mọi method)
4. preHook của route `/users` (chỉ POST)

**Trình tự thực thi:**
```
[Global preHook - all]  [Global preHook - POST]  [Route preHook - all]  [Route preHook - POST]  [Handler]  [postHooks in same order]
```

### Thực thi tuần tự

Mọi hook khớp chạy **tuần tự** từng cái một, không song song. Điều này đảm bảo:
- Thứ tự thực thi có thể dự đoán
- Thay đổi từ hook trước hiển thị cho hook sau
- Dễ debug và ghi log

## Mẫu thường dùng

### Mẫu 1: Kiểm tra trong preHook

```javascript
// preHook: Validate request
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

if (!$ctx.$body.password || $ctx.$body.password.length < 6) {
  $ctx.$throw['422']('Password must be at least 6 characters');
  return;
}

// Store validation result for later use
$ctx.$share.validationPassed = true;
```

### Mẫu 2: Biến đổi dữ liệu trong preHook

```javascript
// preHook: Normalize and enrich data
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
$ctx.$body.slug = $ctx.$helpers.autoSlug($ctx.$body.title);

// Add computed fields
$ctx.$body.createdBy = $ctx.$user.id;
$ctx.$body.createdAt = new Date();
```

### Mẫu 3: Bổ sung response trong postHook

```javascript
// postHook: Add metadata to response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`,
    processedAt: new Date()
  }));
}

$ctx.$data.meta = {
  processedBy: $ctx.$user.id,
  timestamp: new Date()
};
```

### Mẫu 4: Chia sẻ context giữa các hook

```javascript
// preHook: Store data
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// postHook: Use shared data
if ($ctx.$share.processStartTime) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Mẫu 5: Xử lý lỗi trong postHook

```javascript
// postHook: runs on both success and error
if (@ERROR) {
  @LOGS(`Error: ${@ERROR.message}`);
  
  await #error_logs.create({
    data: {
      action: 'error_occurred',
      errorMessage: @ERROR.message,
      statusCode: @ERROR.statusCode,
      userId: @USER?.id
    }
  });
} else {
  @LOGS('Operation completed successfully');
}
```

### Mẫu 6: Kiểm tra quyền trong preHook

```javascript
// preHook: Check permissions
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

if ($ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Admin access required');
  return;
}

// Check resource ownership
const resource = await $ctx.$repos.resources.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (resource.data.length === 0) {
  $ctx.$throw['404']('Resource not found');
  return;
}

if (resource.data[0].userId !== $ctx.$user.id && $ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Access denied');
  return;
}
```

## Thực hành tốt

### Quản lý context

1. **Đặt tên rõ nghĩa** cho thuộc tính tùy biến trong `$ctx.$share`
2. **Kiểm tra sự tồn tại** trước khi truy cập thuộc tính lồng nhau
3. **Dọn dẹp** thuộc tính tạm trong postHook nếu cần
4. **Ghi log thay đổi quan trọng** để debug

### Tổ chức hook

1. Dùng **hook global** cho mối quan tâm xuyên suốt như xác thực, log, audit
2. Dùng **hook theo route** cho logic nghiệp vụ
3. Dùng **hook theo method** cho logic của từng thao tác
4. **Giữ hook tập trung** — mỗi hook chỉ nên có một trách nhiệm

### Hiệu năng

1. **Giảm số lần gọi database** trong hook — gộp batch khi có thể
2. **Lưu cache các thao tác tốn kém** trong `$ctx.$share`
3. **Trả về sớm** để tránh xử lý không cần thiết
4. **Cân nhắc thứ tự thực thi** để tối ưu hiệu năng

### Xử lý lỗi

1. **Kiểm tra sớm** trong preHook để fail fast
2. **Cung cấp thông báo lỗi có ý nghĩa**
3. **Dùng `$ctx.$logs()`** cho thông tin debug
4. **Xử lý lỗi cẩn trọng** trong postHook khi phù hợp

## Bước tiếp theo

- Tìm hiểu [Repository Methods](./repository-methods/) cho các thao tác database
- Xem [Context Reference](./context-reference/) để biết mọi thuộc tính có sẵn
- Xem [Hooks and Handlers](./hooks-handlers/) để tạo logic tùy biến

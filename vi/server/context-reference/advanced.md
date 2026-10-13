---
slug: context-nang-cao
---

# Tham chiếu Context - Tính năng nâng cao

Các tính năng context nâng cao gồm tải tệp lên, thông tin API, context dùng chung và truy cập package.

## Transaction

Dùng `await @TRANSACTION.run(async () => { ... })` khi các mutation qua repository phải cùng commit hoặc cùng rollback. Nó tương đương với `$ctx.$transaction.run(...)` và hoạt động trên database backend đang cấu hình.

```javascript
await @TRANSACTION.run(async () => {
  await @REPOS.orders.create({ data: @BODY.order });
  await @REPOS.inventory.update({
    id: @BODY.inventoryId,
    data: { reserved: true }
  });
});
```

Chỉ các thao tác repository mới thuộc transaction. Không được giả định `@FETCH`, `@STORAGE`, `@CACHE`, `@SOCKET` hoặc `@TRIGGER` có thể rollback; khi cần, chỉ thực hiện các side effect này sau khi transaction thành công.

## Tải tệp lên

Truy cập thông tin về tệp đã tải lên.

Method multipart có named field cung cấp `$ctx.$uploadFile` theo tên đã cấu hình; field count-one giữ một file, multi-file giữ array. Field tùy chọn không có file sẽ vắng mặt, đọc key thiếu sẽ throw. Kiểm tra `Object.keys` trước khi đọc tùy chọn. Xem [Xử lý tệp](../file-handling.md) để biết cấu hình, vòng đời file tạm và tác động phụ storage.

### $ctx.$uploadedFile

Thông tin về tệp đã tải lên (nếu có).

```javascript
if ($ctx.$uploadedFile) {
  const filename = $ctx.$uploadedFile.originalname;
  const mimetype = $ctx.$uploadedFile.mimetype;
  const size = $ctx.$uploadedFile.size;
  const fieldname = $ctx.$uploadedFile.fieldname;
}
```

`$ctx.$uploadedFile` mô tả file multipart, không chứa buffer body. Truyền trực tiếp vào `$ctx.$storage.$upload({ file: $ctx.$uploadedFile })` hoặc `$ctx.$storage.$update(id, { file: $ctx.$uploadedFile })` để Enfyra stream từ đường dẫn file tạm.

### Xử lý tệp đã tải lên

```javascript
if ($ctx.$uploadedFile) {
  $ctx.$logs(`File uploaded: ${$ctx.$uploadedFile.originalname}`);
  $ctx.$logs(`File size: ${$ctx.$uploadedFile.size} bytes`);
  $ctx.$logs(`MIME type: ${$ctx.$uploadedFile.mimetype}`);
  
  // Save request file using the streaming helper.
  const fileResult = await $ctx.$storage.$upload({
    file: $ctx.$uploadedFile,
    description: $ctx.$body.description
  });
}
```

### Streaming response HTTP

Dùng Server package đã cài có readable body một consumer. Lấy upstream URL thuộc allow-list và credential private từ cấu hình server; không dùng URL do caller cung cấp cho proxy có đặc quyền.

```javascript
const upstream = await @PKGS.undici.request(configuredUpstreamUrl, {
  method: 'GET',
  headers: configuredPrivateHeaders,
});
const ready = await $ctx.$streams.preflight(upstream.body, {
  timeoutMs: 5000,
  maxFirstChunkBytes: 65536,
});
return await @RES.stream(ready.stream, {
  mimetype: 'text/event-stream',
});
```

`preflight` quan sát raw chunk không rỗng đầu tiên **trước** khi commit response, trả `{ stream, firstChunk, cancel }`. Relay `stream`, không dùng body gốc, để chunk đầu xuất hiện đúng một lần. Gọi trực tiếp `@RES.stream(upstream.body)` sẽ commit trước khi đọc; chỉ dùng khi đó là contract mong muốn. SSE chỉ có reasoning hoặc raw byte khác cũng được tính là hoạt động, không chứng minh generation thành công.

Các helper `$ctx.$streams` dùng chung giới hạn task:

- `readBytes` / `readText` đọc mà không mở response, nhận `timeoutMs`, `idleTimeoutMs` và `maxBytes` (mặc định 8 MiB). `readText` decode UTF-8 incremental qua byte bị chia.
- `guard` trả readable có giới hạn tổng/idle/byte. `tap` quan sát raw chunk với callback backpressure tuần tự. `cancel` đóng stream đang sở hữu; preflight cũng trả hàm cancel.
- Mỗi readable chỉ có một consumer. Không preflight/đọc hết rồi dùng lại body gốc hoặc pull đồng thời. Deadline first-byte của preflight không trở thành deadline replay; relay còn lại vẫn phải nằm trong giới hạn method.
- `ERR_PACKAGE_STREAM_TIMEOUT` đóng stream đích; `ERR_PACKAGE_STREAM_EMPTY` là EOF trước byte đầu. Task cancellation vẫn đóng resource của task. Kiểm tra Server/Kernel đã deploy hỗ trợ các API này.

`@RES.stream` là ranh giới terminal: await và không trả payload thứ hai. `transform(text, kind)` nhận fragment chunk/end đã decode UTF-8; trả string/null/undefined để thay/bỏ/giữ output. Buffer SSE record hoàn chỉnh qua nhiều callback, không giả định ranh giới network chunk. Transform lỗi làm stream thất bại; `observer` chỉ quan sát best-effort sau commit, không dùng làm readiness hay settlement bắt buộc. `@HELPERS.$fetch` buffer body, không phải streaming transport. Giữ provider header và secret private.

Xem [postHooks](../hooks-handlers/posthooks.md) về audit terminal best-effort riêng; upstream error, client disconnect và deadline stream do Server quản lý là các nguyên nhân khác nhau.

## Thông tin API

Truy cập thông tin chi tiết về request và response API hiện tại.

### Thông tin request

```javascript
const method = $ctx.$api.request.method;           // HTTP method
const url = $ctx.$api.request.url;                 // Request URL
const timestamp = $ctx.$api.request.timestamp;     // Request timestamp
const correlationId = $ctx.$api.request.correlationId;  // Unique request ID
const executionId = $ctx.$api.request.executionId;
const userAgent = $ctx.$api.request.userAgent;     // User agent
const ip = $ctx.$api.request.ip;                   // Client IP
```

### Thông tin response

```javascript
const statusCode = $ctx.$api.response.statusCode;  // HTTP status code
const responseTime = $ctx.$api.response.responseTime;  // Response time in ms
const timestamp = $ctx.$api.response.timestamp;    // Response timestamp
```

### Thông tin lỗi (chỉ trong postHook)

```javascript
if ($ctx.$api.error) {
  const message = $ctx.$api.error.message;         // Error message
  const stack = $ctx.$api.error.stack;             // Stack trace
  const name = $ctx.$api.error.name;               // Error class name
  const statusCode = $ctx.$api.error.statusCode;   // HTTP error status
  const timestamp = $ctx.$api.error.timestamp;     // Error timestamp
  const details = $ctx.$api.error.details;         // Additional details
}
```

## Context dùng chung

Lưu dữ liệu tồn tại xuyên suốt các hook và handler.

### $ctx.$share

Vùng chứa dữ liệu dùng chung để truyền dữ liệu giữa các hook.

```javascript
// In preHook - store data
$ctx.$share.validationPassed = true;
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// In postHook - access shared data
if ($ctx.$share.validationPassed) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Truy cập log

```javascript
// All logs are stored in $ctx.$share.$logs
const allLogs = $ctx.$share.$logs;  // Array of all logged messages
```

## Truy cập môi trường

Đọc các giá trị môi trường không nhạy cảm từ bản chụp môi trường đã được lọc.

### $ctx.$env

`$ctx.$env` exposes a sanitized copy of `process.env` for dynamic scripts. It is useful for non-secret runtime switches such as node names, deployment labels, public URLs, or feature flags.

```javascript
const nodeName = $ctx.$env.NODE_NAME || 'default';
const appMode = $ctx.$env.ENFYRA_MODE || 'all';
```

Các key hạ tầng nhạy cảm không được expose. Danh sách chặn hiện tại gồm:

- `DB_URI`
- `DB_REPLICA_URIS`
- `REDIS_URI`
- `SECRET_KEY`
- `ADMIN_PASSWORD`

Không dùng `$ctx.$env` để lưu secret của ứng dụng. Hãy lưu secret vào field bảng chưa publish có `isEncrypted=true`, sau đó đọc/ghi plaintext qua repository hoặc REST như bình thường.

Field bảng được mã hóa gắn với `SECRET_KEY` của server. Khi self-host, hãy giữ `SECRET_KEY` ổn định, có backup và giống nhau trên mọi server instance của cùng một ứng dụng Enfyra. Thay đổi hoặc làm mất key này sẽ khiến dữ liệu đã mã hóa không thể giải mã.

## Truy cập package

Truy cập các package NPM đã cài trong hệ thống.

### $ctx.$pkgs

Truy cập package đã cài theo tên.

```javascript
// Use installed packages
const axios = $ctx.$pkgs.axios;
const lodash = $ctx.$pkgs.lodash;
const moment = $ctx.$pkgs.moment;

// Example: Using axios
if ($ctx.$pkgs.axios) {
  const response = await $ctx.$pkgs.axios.get('https://api.example.com/data');
}
```

**Lưu ý:** Package phải được cài qua Package Management trên UI thì mới truy cập được.

## Mẫu phổ biến

### Mẫu 1: Xác thực và biến đổi dữ liệu request

```javascript
// Validate required fields
if (!$ctx.$body.email) {
  @THROW400('Email is required');
  return;
}

// Transform data
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
```

### Mẫu 2: Kiểm tra quyền người dùng

```javascript
if (!$ctx.$user) {
  @THROW401('Authentication required');
  return;
}

if ($ctx.$user.role !== 'admin') {
  @THROW403('Admin access required');
  return;
}
```

### Mẫu 3: Dùng context chung giữa các hook

```javascript
// In preHook
$ctx.$share.userId = $ctx.$user.id;
$ctx.$share.validationData = { passed: true };

// In postHook
const userId = $ctx.$share.userId;
if ($ctx.$share.validationData.passed) {
  // Use validation result
}
```

### Mẫu 4: Cache dữ liệu được truy cập thường xuyên

```javascript
// Check cache first
const cacheKey = `user:${$ctx.$params.id}`;
let user = await $ctx.$cache.get(cacheKey);

if (!user) {
  // Cache miss - fetch from database
  const result = await $ctx.$repos.enfyra_user.find({
    filter: { id: { _eq: $ctx.$params.id } }
  });
  
  if (result.data.length > 0) {
    user = result.data[0];
    // Cache for 5 minutes
    await $ctx.$cache.set(cacheKey, user, 300000);
  }
}
```

### Mẫu 5: Xử lý lỗi trong postHook

```javascript
// In postHook
if ($ctx.$api.error) {
  // Log error
  $ctx.$logs(`Error: ${$ctx.$api.error.message}`);
  
  // Log to audit system
  await $ctx.$repos.audit_logs.create({
    data: {
      action: 'error_occurred',
      userId: $ctx.$user?.id,
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      timestamp: new Date()
    }
  });
} else {
  // Success - add metadata
  $ctx.$data.processedAt = new Date();
  $ctx.$data.processedBy = $ctx.$user?.id;
}
```

## Tiếp theo

- Xem [Xử lý tệp](../file-handling.md) để biết hướng dẫn tải tệp đầy đủ
- Xem [Thao tác cache](../cache-operations.md) để biết các mẫu cache chi tiết
- Tìm hiểu [Xử lý lỗi](../error-handling.md) để biết các mẫu xử lý lỗi

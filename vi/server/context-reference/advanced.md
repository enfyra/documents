---
slug: context-nang-cao
---

# Tham chiếu Context - Tính năng nâng cao

Các tính năng context nâng cao gồm tải tệp lên, thông tin API, context dùng chung và truy cập package.

## Tải tệp lên

Truy cập thông tin về tệp đã tải lên.

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

`$ctx.$uploadedFile` describes a multipart request file. It does not expose the file body as a buffer. Pass it directly to `$ctx.$storage.$upload({ file: $ctx.$uploadedFile })` or `$ctx.$storage.$update(id, { file: $ctx.$uploadedFile })` so Enfyra streams from the temp file path.

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

Với endpoint proxy, dùng một server package đã cài đặt để lấy readable stream rồi truyền thẳng vào `@RES.stream`:

```javascript
const upstream = await @PKGS.undici.request(@QUERY.url, { method: 'GET' });

await @RES.stream(upstream.body, {
  mimetype: 'text/event-stream',
  transform: (text, kind) => {
    if (kind === 'end') return 'data: [DONE]\n\n';
    return text.replace(/^data:/gm, 'event: converted\ndata:');
  }
});
```

`transform(text, kind)` chạy trước khi relay mỗi `chunk` và lần `end` cuối. Return string để thay output, `null` để bỏ output, hoặc `undefined` để giữ nguyên. Nó dùng để điều chỉnh stream mà client nhận; lỗi bị ném sẽ làm stream thất bại. Enfyra dùng một UTF-8 decoder incremental cho toàn bộ stream trước khi gọi `transform`, nên ký tự bị cắt giữa các byte network vẫn còn nguyên trong `text`. Tuy vậy, `text` vẫn có thể kết thúc giữa một dòng SSE, JSON object hoặc event. Hãy giữ SSE buffer theo từng request và chỉ parse record kết thúc bằng dòng trống (`\n\n` hoặc dạng CRLF); không dùng kích thước chunk cố định hay dấu cách làm ranh giới. `@HELPERS.$fetch` buffer toàn bộ response nên không phù hợp với SSE hoặc chat streaming. Giữ authorization header của upstream ở server, chỉ chuyển tiếp response header an toàn, và không return payload khác sau khi `@RES.stream` bắt đầu.

## Thông tin API

Truy cập thông tin chi tiết về request và response API hiện tại.

### Thông tin request

```javascript
const method = $ctx.$api.request.method;           // HTTP method
const url = $ctx.$api.request.url;                 // Request URL
const timestamp = $ctx.$api.request.timestamp;     // Request timestamp
const correlationId = $ctx.$api.request.correlationId;  // Unique request ID
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
  $ctx.$throw['400']('Email is required');
  return;
}

// Transform data
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
```

### Mẫu 2: Kiểm tra quyền người dùng

```javascript
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

if ($ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Admin access required');
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

---
slug: xu-ly-loi
---

# Xử lý lỗi

Enfyra có sẵn cơ chế xử lý lỗi để ném lỗi HTTP và xử lý ngoại lệ trong hook, handler.

## Điều hướng nhanh

- [Ném lỗi](#nem-loi) - Cách ném lỗi HTTP
- [Mã trạng thái HTTP](#ma-trang-thai-http) - Các mã trạng thái hỗ trợ
- [Xử lý lỗi trong postHook](#xu-ly-loi-trong-posthook) - Xử lý lỗi đã xảy ra
- [Mẫu thường dùng](#mau-thuong-dung) - Ví dụ xử lý lỗi thực tế

## Ném lỗi

Dùng `$ctx.$throw` để ném lỗi HTTP với mã trạng thái phù hợp.

### Cú pháp cơ bản

```javascript
$ctx.$throw['STATUS_CODE']('Error message');
```

### Các mã trạng thái HTTP thường dùng

```javascript
// Bad Request (400)
$ctx.$throw['400']('Invalid input data');

// Unauthorized (401)
$ctx.$throw['401']('Authentication required');

// Forbidden (403)
$ctx.$throw['403']('Insufficient permissions');

// Not Found (404)
$ctx.$throw['404']('Resource not found');

// Conflict (409)
$ctx.$throw['409']('Email already exists');

// Unprocessable Entity (422)
$ctx.$throw['422']('Validation failed');

// Too Many Requests (429) - Rate Limiting
$ctx.$throw['429']('Rate limit exceeded. Try again later');

// Internal Server Error (500)
$ctx.$throw['500']('Internal server error');

// Service Unavailable (503)
$ctx.$throw['503']('Service temporarily unavailable');
```

## Mã trạng thái HTTP

### 400 Yêu cầu không hợp lệ

Dùng cho dữ liệu đầu vào không hợp lệ hoặc request sai định dạng.

```javascript
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

if (!isValidEmail($ctx.$body.email)) {
  $ctx.$throw['400']('Invalid email format');
  return;
}
```

### 401 Chưa xác thực

Dùng khi cần xác thực nhưng request không cung cấp thông tin xác thực.

```javascript
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

if (!isValidToken($ctx.$req.headers.authorization)) {
  $ctx.$throw['401']('Invalid or expired token');
  return;
}
```

### 403 Bị từ chối

Dùng khi user đã xác thực nhưng không có quyền.

```javascript
if ($ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Admin access required');
  return;
}

// Check resource ownership
const resource = await $ctx.$repos.resources.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (resource.data[0].userId !== $ctx.$user.id) {
  $ctx.$throw['403']('Access denied');
  return;
}
```

### 404 Không tìm thấy

Dùng khi tài nguyên được yêu cầu không tồn tại.

```javascript
const product = await $ctx.$repos.products.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (product.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}
```

### 409 Xung đột

Dùng khi trạng thái hiện tại bị xung đột, ví dụ bản ghi trùng lặp.

```javascript
// Check if email already exists
const existing = await $ctx.$repos.enfyra_user.find({
  filter: { email: { _eq: $ctx.$body.email } }
});

if (existing.data.length > 0) {
  $ctx.$throw['409']('Email already exists');
  return;
}
```

### 422 Không thể xử lý thực thể

Dùng cho lỗi kiểm tra khi định dạng dữ liệu đúng nhưng vi phạm quy tắc nghiệp vụ.

```javascript
if ($ctx.$body.password && $ctx.$body.password.length < 6) {
  $ctx.$throw['422']('Password must be at least 6 characters');
  return;
}

if ($ctx.$body.age && ($ctx.$body.age < 0 || $ctx.$body.age > 120)) {
  $ctx.$throw['422']('Age must be between 0 and 120');
  return;
}
```

### 429 Quá nhiều request

Dùng cho giới hạn tần suất khi client vượt số request cho phép.

```javascript
// Using rate limit helper
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}

// Skip rate limit for admins
if (!$ctx.$user?.isRootAdmin) {
  const rateResult = await $ctx.$helpers.$rateLimit.byUser({
    maxRequests: 1000,
    perSeconds: 3600
  });

  if (!rateResult.allowed) {
    $ctx.$throw['429']('API rate limit exceeded');
    return;
  }
}
```

### 500 Lỗi máy chủ nội bộ

Dùng cho lỗi server không mong đợi.

```javascript
try {
  // Some operation
} catch (error) {
  $ctx.$logs(`Unexpected error: ${error.message}`);
  $ctx.$throw['500']('Internal server error');
  return;
}
```

## Xử lý lỗi trong postHook

Trong postHook, bạn có thể kiểm tra lỗi xảy ra trong request và xử lý phù hợp. Cùng thông tin lỗi có tại **`@ERROR` / `$ctx.$error`** (macro template) và **`$ctx.$api.error`** (lồng trong metadata API).

### Kiểm tra lỗi

```javascript
// In postHook
if ($ctx.$api.error) {
  // Error occurred
  // $ctx.$api.error contains error details
  // $ctx.$data will be null
  // $ctx.$statusCode will be the error status code
} else {
  // Success
  // $ctx.$data contains response data
  // $ctx.$statusCode contains success status (200, 201, etc.)
}
```

**Lưu ý:** `$ctx.$api.error` chỉ có trong postHook, không có trong preHook.

### Thuộc tính đối tượng lỗi

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

### Ghi log lỗi

```javascript
// In postHook
if ($ctx.$api.error) {
  $ctx.$logs(`Error occurred: ${$ctx.$api.error.message}`);
  $ctx.$logs(`Error status: ${$ctx.$api.error.statusCode}`);
  $ctx.$logs(`Error stack: ${$ctx.$api.error.stack}`);
}
```

### Tạo log lỗi

```javascript
// In postHook
if ($ctx.$api.error) {
  // Log to audit system
  await $ctx.$repos.error_logs.create({
    data: {
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      userId: $ctx.$user?.id,
      url: $ctx.$req.url,
      method: $ctx.$req.method,
      timestamp: new Date()
    }
  });
}
```

## Mẫu thường dùng

### Mẫu 1: Kiểm tra trường bắt buộc

```javascript
// In preHook
const requiredFields = ['email', 'password', 'name'];

for (const field of requiredFields) {
  if (!$ctx.$body[field]) {
    $ctx.$throw['400'](`${field} is required`);
    return;
  }
}
```

### Mẫu 2: Kiểm tra tài nguyên tồn tại

```javascript
// In preHook or handler
const resource = await $ctx.$repos.products.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (resource.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Use resource.data[0] safely
const product = resource.data[0];
```

### Mẫu 3: Kiểm tra quy tắc nghiệp vụ

```javascript
// In preHook
if ($ctx.$body.email) {
  // Check email format
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test($ctx.$body.email)) {
    $ctx.$throw['422']('Invalid email format');
    return;
  }
  
  // Check if email already exists
  const existing = await $ctx.$repos.enfyra_user.find({
    filter: { email: { _eq: $ctx.$body.email } }
  });
  
  if (existing.data.length > 0) {
    $ctx.$throw['409']('Email already exists');
    return;
  }
}
```

### Mẫu 4: Kiểm tra quyền

```javascript
// In preHook
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

### Mẫu 5: Phục hồi lỗi trong postHook

```javascript
// In postHook
if ($ctx.$api.error) {
  // Log error
  $ctx.$logs(`Error: ${$ctx.$api.error.message}`);
  
  // Log to audit system
  await $ctx.$repos.error_logs.create({
    data: {
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      userId: $ctx.$user?.id,
      url: $ctx.$req.url,
      timestamp: new Date()
    }
  });
  
  // Optionally send notification
  // await sendErrorNotification($ctx.$api.error);
} else {
  // Success - log audit trail
  await $ctx.$repos.audit_logs.create({
    data: {
      action: `${$ctx.$req.method} ${$ctx.$req.url}`,
      userId: $ctx.$user?.id,
      statusCode: $ctx.$statusCode,
      timestamp: new Date()
    }
  });
}
```

### Mẫu 6: Try-catch cho thao tác bên ngoài

```javascript
// In handler
try {
  // External API call or complex operation
  const result = await externalApiCall($ctx.$body);
  return result;
} catch (error) {
  $ctx.$logs(`External API error: ${error.message}`);
  $ctx.$throw['500']('Failed to process request');
  return;
}
```

### Mẫu 7: Kiểm tra kiểu dữ liệu

```javascript
// In preHook
if ($ctx.$body.price !== undefined) {
  if (typeof $ctx.$body.price !== 'number') {
    $ctx.$throw['400']('Price must be a number');
    return;
  }

  if ($ctx.$body.price < 0) {
    $ctx.$throw['422']('Price cannot be negative');
    return;
  }
}
```

### Mẫu 8: Giới hạn tần suất

```javascript
// In preHook - Rate limit by IP
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}

// In preHook - Rate limit login attempts
const loginResult = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 60
});

if (!loginResult.allowed) {
  $ctx.$throw['429'](`Too many login attempts. Try again in ${loginResult.retryAfter}s`);
  return;
}

// In preHook - Skip rate limit for admins
if (!$ctx.$user?.isRootAdmin) {
  const apiResult = await $ctx.$helpers.$rateLimit.byUser({
    maxRequests: 1000,
    perSeconds: 3600
  });

  if (!apiResult.allowed) {
    $ctx.$throw['429']('API rate limit exceeded');
    return;
  }
}
```

## Thực hành tốt

1. **Ném lỗi sớm** — kiểm tra và ném lỗi ngay khi có thể, trong preHook
2. **Dùng mã trạng thái phù hợp** — chọn đúng mã HTTP cho từng loại lỗi
3. **Đưa thông báo lỗi rõ ràng** — giúp người dùng hiểu vấn đề
4. **Ghi log lỗi** — dùng `$ctx.$logs()` để phục vụ debug
5. **Xử lý lỗi cẩn trọng** — dùng postHook xử lý lỗi đã xảy ra trong handler
6. **Không để lộ thông tin nhạy cảm** — không đưa chi tiết nội bộ vào thông báo lỗi
7. **Trả về sớm** — dùng `return` sau khi ném lỗi để dừng thực thi

## Bước tiếp theo

- Tìm hiểu [Context Reference](./context-reference/) về các thuộc tính liên quan đến lỗi
- Xem [API Lifecycle](./api-lifecycle.md) để hiểu lỗi phát sinh ở thời điểm nào
- Xem [Hooks and Handlers](./hooks-handlers/) để xử lý lỗi ở các giai đoạn khác nhau

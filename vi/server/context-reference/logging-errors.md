---
slug: ghi-log-va-xu-ly-loi-context
---

# Tham chiếu Context - Ghi log và xử lý lỗi

Ghi log vào response API và ném lỗi HTTP với mã trạng thái phù hợp.

## Ghi log

Thêm log vào response API. Các log này tự động xuất hiện trong response.

### Ghi log cơ bản

```javascript
$ctx.$logs('Operation started');
$ctx.$logs('User created successfully');
$ctx.$logs(`Processing order: ${orderId}`);
```

### Ghi log biến

```javascript
const userId = $ctx.$user.id;
const orderId = 123;

$ctx.$logs(`User ID: ${userId}`);
$ctx.$logs(`Order ID: ${orderId}`);
$ctx.$logs(`Processing data:`, dataObject);
```

### Ghi log ở các giai đoạn khác nhau

```javascript
// In preHook
$ctx.$logs('Validation started');

// In handler
$ctx.$logs('Creating user...');

// In postHook
$ctx.$logs('User created successfully');
```

**Lưu ý:** Log tự xuất hiện trong response API; bạn không cần tự thêm chúng vào giá trị trả về.

## Xử lý lỗi

Ném lỗi HTTP với mã trạng thái phù hợp.

### Ném lỗi theo mã trạng thái HTTP

```javascript
// Bad Request
$ctx.$throw['400']('Invalid input data');

// Unauthorized
$ctx.$throw['401']('Authentication required');

// Forbidden
$ctx.$throw['403']('Insufficient permissions');

// Not Found
$ctx.$throw['404']('Resource not found');

// Conflict
$ctx.$throw['409']('Email already exists');

// Unprocessable Entity
$ctx.$throw['422']('Validation failed');

// Too Many Requests (Rate Limiting)
$ctx.$throw['429']('Rate limit exceeded. Try again later');

// Internal Server Error
$ctx.$throw['500']('Internal server error');

// Service Unavailable
$ctx.$throw['503']('Service temporarily unavailable');
```

### Xử lý lỗi trong postHook

Trong postHook, bạn có thể kiểm tra lỗi xảy ra khi xử lý request:

```javascript
// Check if error occurred
if ($ctx.$api.error) {
  // Error case
  $ctx.$logs(`Error occurred: ${$ctx.$api.error.message}`);
  $ctx.$logs(`Error status: ${$ctx.$api.error.statusCode}`);
  
  // $ctx.$data will be null when error occurs
  // $ctx.$statusCode will be the error status code
} else {
  // Success case
  $ctx.$logs('Operation completed successfully');
  // $ctx.$data contains the response data
  // $ctx.$statusCode contains success status (200, 201, etc.)
}
```

**Lưu ý:** `$ctx.$api.error` chỉ có trong postHook, không có trong preHook.

## Tiếp theo

- Xem [Xử lý lỗi](../error-handling.md) để biết hướng dẫn đầy đủ
- Xem [Tính năng nâng cao](./advanced.md) để biết thông tin API và chi tiết lỗi
- Tìm hiểu [Hook và Handler](../hooks-handlers/) để dùng ghi log trong hook

---
slug: pre-hook
---

# Hooks và Handler - preHooks

preHook chạy trước handler. Dùng chúng để xác thực, biến đổi dữ liệu và kiểm tra quyền.

## Khi nào dùng preHook

- Xác thực dữ liệu request
- Biến đổi hoặc chuẩn hóa dữ liệu đầu vào
- Kiểm tra quyền người dùng
- Thay đổi body request hoặc tham số query
- Lưu dữ liệu vào context dùng chung để sử dụng về sau

## Ví dụ preHook cơ bản

```javascript
// Validate required fields
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

if (!$ctx.$body.password) {
  $ctx.$throw['400']('Password is required');
  return;
}

// Normalize email
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();

// Store validation result
$ctx.$share.validationPassed = true;
```

## Biến đổi dữ liệu

```javascript
// Normalize data
$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$body.name = $ctx.$body.name.trim();

// Generate slug
if ($ctx.$body.title) {
  $ctx.$body.slug = $ctx.$helpers.autoSlug($ctx.$body.title);
}

// Add computed fields
$ctx.$body.createdBy = $ctx.$user.id;
$ctx.$body.createdAt = new Date();
```

## Kiểm tra quyền

```javascript
// Check authentication
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

// Check role
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

if (resource.data[0].userId !== $ctx.$user.id) {
  $ctx.$throw['403']('Access denied');
  return;
}
```

## Thay đổi tham số query

```javascript
// Add user-specific filter
if ($ctx.$user.role !== 'admin') {
  // Non-admins only see their own records
  $ctx.$query.filter = {
    ...($ctx.$query.filter || {}),
    userId: { _eq: $ctx.$user.id }
  };
}
```

## Giới hạn tần suất

Bảo vệ route API khỏi bị lạm dụng bằng hàm hỗ trợ giới hạn tần suất.

### Giới hạn tần suất cơ bản theo IP

```javascript
// Limit requests per IP address
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}
```

### Giới hạn số lần thử đăng nhập

```javascript
// Protect login endpoint from brute force
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Too many login attempts. Try again in ${result.retryAfter}s`);
  return;
}
```

### Giới hạn tần suất theo người dùng

```javascript
// Limit API requests per authenticated user
const result = await $ctx.$helpers.$rateLimit.byUser({
  maxRequests: 1000,
  perSeconds: 3600  // 1 hour
});

if (!result.allowed) {
  $ctx.$throw['429']('API rate limit exceeded');
  return;
}
```

### Bỏ qua giới hạn cho quản trị viên

```javascript
// Only apply rate limit to non-admin users
if (!$ctx.$user?.isRootAdmin) {
  const result = await $ctx.$helpers.$rateLimit.byIp({
    maxRequests: 100,
    perSeconds: 60
  });

  if (!result.allowed) {
    $ctx.$throw['429']('Rate limit exceeded');
    return;
  }
}
```

### Khóa giới hạn tần suất tùy chỉnh

```javascript
// Rate limit per specific resource
const resourceId = $ctx.$params.id;
const result = await $ctx.$helpers.$rateLimit.check(
  `resource:${resourceId}:${$ctx.$user?.id || $ctx.$req.ip}`,
  { maxRequests: 10, perSeconds: 60 }
);

if (!result.allowed) {
  $ctx.$throw['429']('Too many requests to this resource');
  return;
}
```

## Tiếp theo

- Xem [postHooks](./posthooks.md) để biết thao tác sau handler
- Tìm hiểu [Handler tùy chỉnh](./custom-handlers.md) cho logic nghiệp vụ tùy chỉnh
- Xem [Mẫu phổ biến](./patterns.md) để biết thực hành tốt

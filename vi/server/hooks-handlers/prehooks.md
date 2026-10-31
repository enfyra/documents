---
slug: pre-hook
---

# Hooks và Handler - preHooks

preHook chạy trước handler. Dùng chúng để xác thực, biến đổi dữ liệu và kiểm tra quyền.

## Return sớm hoặc từ chối request

Pre-hook có thể dùng `return` để bỏ qua phần mã còn lại của chính hook đó. Giá trị trả về bị bỏ qua: các pre-hook sau và handler vẫn tiếp tục chạy. Trả về `{ error: ... }`, `false` hay `null` không từ chối request và không trở thành response.

Muốn dừng request trước handler, hãy throw bằng `@THROW400('Invalid input')`, `@THROW.http(statusCode, message)` hoặc `@THROW.json(body, { statusCode, headers })`. Throw bỏ qua các pre-hook còn lại và handler, nhưng các post-hook phù hợp vẫn chạy với `@ERROR` có giá trị. Post-hook không thể biến lần từ chối đó thành response thành công. Cancellation, execution deadline và lỗi stream terminal dùng [nhánh audit opt-in](./posthooks.md) thay vì chạy lại hook thông thường.

## Chia hook theo thứ tự để truyền decision

Tách các trách nhiệm độc lập thành những hook có tên rõ ràng. Chọn HTTP method cần áp dụng và đặt `priority` khác nhau: số nhỏ chạy trước. Lưu decision dùng chung trong `@SHARE`, không truyền bằng giá trị return của hook. Hook không liên quan nên return ngay trước khi thực hiện I/O.

Ví dụ, gắn hai hook sau vào cùng operation `POST`:

| Priority | Hook | Trách nhiệm |
|---|---|---|
| `10` | Default delivery | Khởi tạo decision do server quản lý |
| `20` | Express delivery | Chỉ điều chỉnh decision khi phù hợp |

Default delivery:

```javascript
@SHARE.deliveryDecision = { mode: 'standard', requiresSignature: false };
```

Express delivery:

```javascript
if (@BODY.delivery !== 'express') return;
@SHARE.deliveryDecision = { mode: 'express', requiresSignature: true };
```

Handler sử dụng decision cuối cùng:

```javascript
const decision = @SHARE.deliveryDecision;
return { delivery: decision.mode, requiresSignature: decision.requiresSignature };
```

Nếu không hook chuyên biệt nào phù hợp, handler dùng decision mặc định. Endpoint URL, credential và quyết định phân quyền phải do server quản lý; không nhận override từ client cho routing riêng tư. Không đưa toàn bộ shared context vào response công khai.

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
  @THROW400('Email is required');
  return;
}

if (!$ctx.$body.password) {
  @THROW400('Password is required');
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
  @THROW401('Authentication required');
  return;
}

// Check role
if ($ctx.$user.role !== 'admin') {
  @THROW403('Admin access required');
  return;
}

// Check resource ownership
const resource = await $ctx.$repos.resources.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (resource.data.length === 0) {
  @THROW404('Resource not found');
  return;
}

if (resource.data[0].userId !== $ctx.$user.id) {
  @THROW403('Access denied');
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
  @THROW429(`Rate limit exceeded. Try again in ${result.retryAfter}s`);
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
  @THROW429(`Too many login attempts. Try again in ${result.retryAfter}s`);
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
  @THROW429('API rate limit exceeded');
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
    @THROW429('Rate limit exceeded');
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
  @THROW429('Too many requests to this resource');
  return;
}
```

## Tiếp theo

- Xem [postHooks](./posthooks.md) để biết thao tác sau handler
- Tìm hiểu [Handler tùy chỉnh](./custom-handlers.md) cho logic nghiệp vụ tùy chỉnh
- Xem [Mẫu phổ biến](./patterns.md) để biết thực hành tốt

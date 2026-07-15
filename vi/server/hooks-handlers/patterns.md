---
slug: mau-hook-va-handler
---

# Hooks và Handler - Các mẫu phổ biến

Các mẫu phổ biến và thực hành tốt khi làm việc với hook và handler.

## Mẫu phổ biến

### Mẫu 1: Xác thực và biến đổi

```javascript
// preHook
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
$ctx.$share.validationPassed = true;

// Handler uses normalized data automatically
```

### Mẫu 2: Lọc theo quyền

```javascript
// preHook
if ($ctx.$user.role !== 'admin') {
  // Non-admins only see their own records
  $ctx.$query.filter = {
    ...($ctx.$query.filter || {}),
    userId: { _eq: $ctx.$user.id }
  };
}

// Handler/Default CRUD uses filtered query
```

### Mẫu 3: Bổ sung response

```javascript
// postHook
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    displayName: `${item.firstName} ${item.lastName}`,
    formattedPrice: `$${item.price.toFixed(2)}`
  }));
}
```

### Mẫu 4: Audit trail

```javascript
// postHook — runs on both success and error
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

### Mẫu 5: Context dùng chung

```javascript
// preHook
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// postHook
if ($ctx.$share.processStartTime) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Mẫu 6: Ghi log lỗi

```javascript
// postHook — runs even when preHook/handler throws
if (@ERROR) {
  @LOGS(`Error occurred: ${@ERROR.message}`);

  await #error_logs.create({
    data: {
      errorMessage: @ERROR.message,
      statusCode: @ERROR.statusCode,
      userId: @USER?.id,
      url: @API.request.url
    }
  });
}
```

### Mẫu 7: Giới hạn tần suất

```javascript
// preHook - Protect endpoints from abuse
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}
```

### Mẫu 8: Giới hạn tần suất và bỏ qua quản trị viên

```javascript
// preHook - Skip rate limit for admin users
if (!$ctx.$user?.isRootAdmin) {
  const result = await $ctx.$helpers.$rateLimit.byUser({
    maxRequests: 1000,
    perSeconds: 3600
  });

  if (!result.allowed) {
    $ctx.$throw['429']('API rate limit exceeded');
    return;
  }
}
```

### Mẫu 9: Giới hạn số lần thử đăng nhập

```javascript
// preHook - Protect login from brute force
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Too many login attempts. Try again in ${result.retryAfter}s`);
  return;
}
```

## Thực hành tốt

### Tổ chức hook

1. Dùng **global hook** cho các mối quan tâm xuyên suốt như auth và logging.
2. Dùng **route hook** cho logic riêng của từng route.
3. Giữ hook tập trung vào một trách nhiệm.
4. Đặt tên rõ nghĩa cho định nghĩa hook.

### Chất lượng mã

1. Validation sớm trong preHook.
2. Biến đổi dữ liệu trong preHook trước handler.
3. Bổ sung dữ liệu response trong postHook sau handler.
4. Dùng context dùng chung để truyền dữ liệu giữa các hook.
5. Ghi log các bước quan trọng để debug.

### Xử lý lỗi

1. Throw error sớm trong preHook để fail fast.
2. **Xử lý lỗi phù hợp** trong postHook
3. Cung cấp thông báo lỗi có ý nghĩa.
4. Dùng HTTP status code phù hợp.

### Hiệu năng

1. Giảm số lần gọi database; batch operation khi có thể.
2. Cache các thao tác tốn kém trong context dùng chung.
3. Dùng early return để tránh xử lý không cần thiết.
4. Cân nhắc thứ tự thực thi để có hiệu năng tốt.

## Tiếp theo

- Xem [preHooks](./prehooks.md) để biết thao tác trước handler
- Tìm hiểu [postHooks](./posthooks.md) để biết thao tác sau handler
- Xem [Handler tùy chỉnh](./custom-handlers.md) để biết logic nghiệp vụ tùy chỉnh
- Tìm hiểu [Phương thức Repository](../repository-methods/) để thao tác cơ sở dữ liệu
- Xem [Tham chiếu Context](../context-reference/) để biết mọi thuộc tính có sẵn
- Xem [Vòng đời API](../api-lifecycle.md) để hiểu thứ tự thực thi

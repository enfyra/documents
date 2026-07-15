---
slug: bo-nho-dem-va-khoa
---

# Thao tác bộ nhớ đệm

Enfyra cung cấp thao tác user cache được quản lý và cơ chế khóa phân tán qua Redis. Hãy dùng cache để tối ưu hiệu năng, giới hạn tần suất và điều phối giữa các instance.

## Điều hướng nhanh

- [Bắt đầu](#bat-dau) - Thao tác cache cơ bản
- [Lưu trữ cache](#luu-tru-cache) - Đọc và ghi giá trị cache
- [Khóa phân tán](#khoa-phan-tan) - Điều phối thao tác giữa các instance
- [Mẫu thường dùng](#mau-thuong-dung) - Ví dụ sử dụng thực tế

## Bắt đầu

Mọi thao tác cache đều có qua `$ctx.$cache` và cần `await`.

```javascript
// All cache functions require await
const value = await $ctx.$cache.get('key');
await $ctx.$cache.set('key', value, 60000);
```

`$ctx.$cache` và macro `@CACHE` dùng chung một vùng user cache. Hãy dùng khóa logic như `user:123`, `session:abc` hoặc `report:daily`; Enfyra tự thêm phạm vi namespace của app hiện tại. Không đưa `NODE_NAME`, `user_cache:` hay tiền tố Redis vào mã handler.

Trên deployment dùng Redis, user cache được lưu dưới `NODE_NAME:user_cache:*`. Redis Admin Key Editor có thể chỉnh sửa cũng dùng đúng contract này, nên giá trị ghi từ giao diện admin sẽ đọc được qua `$ctx.$cache`.

Ngưỡng phân bổ mềm do `REDIS_USER_CACHE_LIMIT_MB` điều khiển, mặc định là `30` MB. Khi user cache vượt ngưỡng, Enfyra chỉ xóa các khóa user cache ít được dùng gần đây nhất. Snapshot runtime cache, BullMQ queue, trạng thái Socket.IO, telemetry runtime và lock là khóa hệ thống; chúng không tính vào user cache và không bị cơ chế dọn user cache loại bỏ.

## Lưu trữ cache

### Lấy giá trị từ cache

Lấy một giá trị từ cache.

```javascript
const cachedValue = await $ctx.$cache.get(key);

// Example
const user = await $ctx.$cache.get('user:123');
if (user) {
  // Use cached value
} else {
  // Cache miss - fetch from database
}
```

### Ghi giá trị cache kèm TTL

Lưu một giá trị vào cache kèm thời hạn hết hạn.

```javascript
await $ctx.$cache.set(key, value, ttlMs);

// Example - cache for 5 minutes (300000 milliseconds)
await $ctx.$cache.set('user:123', userData, 300000);

// Example - cache for 1 hour (3600000 milliseconds)
await $ctx.$cache.set('product:456', productData, 3600000);
```

**TTL (thời gian sống) tính bằng mili giây:**
- 1000 = 1 giây
- 60000 = 1 phút
- 300000 = 5 phút
- 3600000 = 1 giờ
- 86400000 = 1 ngày

### Ghi giá trị cache không hết hạn

Lưu một giá trị vào cache không tự hết hạn.

```javascript
await $ctx.$cache.setNoExpire(key, value);

// Example
await $ctx.$cache.setNoExpire('config:app', configData);
```

Mục user cache lưu lâu dài vẫn có thể bị loại do vượt ngưỡng phân bổ mềm. Hãy ưu tiên TTL cho dữ liệu có thể dựng lại từ database hoặc nguồn bên ngoài.

### Xóa khóa cache

Xóa một giá trị khỏi cache.

```javascript
await $ctx.$cache.deleteKey(key);

// Example
await $ctx.$cache.deleteKey('user:123');
```

### Kiểm tra khóa tồn tại

Kiểm tra khóa cache có tồn tại với một giá trị cụ thể hay không.

```javascript
const exists = await $ctx.$cache.exists(key, value);

// Example
const lockExists = await $ctx.$cache.exists('user-lock:123', 'user-456');
```

## Khóa phân tán

Khóa phân tán ngăn thao tác đồng thời trên nhiều instance.

### Nhận khóa

Thử nhận khóa. Trả về `true` khi thành công, `false` khi khóa đã được giữ.

```javascript
const lockAcquired = await $ctx.$cache.acquire(key, value, ttlMs);

// Example
const lockKey = `user-lock:${userId}`;
const lockValue = $ctx.$user.id;
const acquired = await $ctx.$cache.acquire(lockKey, lockValue, 10000); // 10 seconds

if (acquired) {
  // Lock acquired - proceed with critical operation
} else {
  // Lock already held by another instance
}
```

### Nhả khóa

Nhả một khóa do bạn đã nhận.

```javascript
const released = await $ctx.$cache.release(key, value);

// Example
const lockKey = `user-lock:${userId}`;
const lockValue = $ctx.$user.id;
const released = await $ctx.$cache.release(lockKey, lockValue);
```

**Lưu ý:** Chỉ instance đã nhận khóa mới có thể nhả nó; điều này được xác minh bằng value.

### Mẫu khóa với try-finally

Luôn nhả khóa trong khối `finally` để bảo đảm dọn dẹp ngay cả khi có lỗi.

```javascript
const lockKey = `record-lock:${recordId}`;
const lockValue = $ctx.$user.id;
const lockAcquired = await $ctx.$cache.acquire(lockKey, lockValue, 10000);

if (!lockAcquired) {
  $ctx.$throw['409']('Record is currently being modified');
  return;
}

try {
  // Critical operation here
  $ctx.$logs(`Acquired lock for record: ${recordId}`);
  
  // Perform the operation
  await $ctx.$repos.records.update({
    id: recordId,
    data: updateData
  });
  
} finally {
  // Always release the lock
  await $ctx.$cache.release(lockKey, lockValue);
  $ctx.$logs(`Released lock for record: ${recordId}`);
}
```

## Mẫu thường dùng

### Mẫu 1: Ưu tiên đọc cache

Kiểm tra cache trước, rồi fallback về database khi cache miss.

```javascript
const cacheKey = `user-profile:${$ctx.$params.id}`;
let userProfile = await $ctx.$cache.get(cacheKey);

if (!userProfile) {
  // Cache miss - fetch from database
  const result = await $ctx.$repos.enfyra_user.find({
    filter: { id: { _eq: $ctx.$params.id } }
  });
  
  if (result.data.length > 0) {
    userProfile = result.data[0];
    // Cache for 5 minutes
    await $ctx.$cache.set(cacheKey, userProfile, 300000);
    $ctx.$logs(`User profile cached: ${$ctx.$params.id}`);
  }
} else {
  $ctx.$logs(`User profile served from cache: ${$ctx.$params.id}`);
}

// Use userProfile...
```

### Mẫu 2: Invalidate cache khi cập nhật

Xóa cache khi dữ liệu được cập nhật.

```javascript
// Update record
const result = await $ctx.$repos.products.update({
  id: productId,
  data: updateData
});

// Invalidate cache
await $ctx.$cache.deleteKey(`product:${productId}`);
$ctx.$logs(`Cache invalidated for product: ${productId}`);
```

### Mẫu 3: Giới hạn tần suất

Dùng helper giới hạn tần suất có sẵn để bảo vệ API ổn định.

```javascript
// Recommended: Use $helpers.$rateLimit for rate limiting
const result = await $ctx.$helpers.$rateLimit.byUser({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}

$ctx.$logs(`Rate limit check passed. Remaining: ${result.remaining}`);
```

**Lựa chọn khác: tự giới hạn tần suất bằng cache** (cho tình huống tùy biến):

```javascript
// Manual rate limit: max 10 requests per minute per user
const rateLimitKey = `rate-limit:${$ctx.$user.id}:${$ctx.$req.url}`;
const currentCount = await $ctx.$cache.get(rateLimitKey) || 0;

if (currentCount >= 10) {
  $ctx.$throw['429']('Rate limit exceeded. Please try again later.');
  return;
}

// Increment counter with 60 second TTL
await $ctx.$cache.set(rateLimitKey, currentCount + 1, 60000);
$ctx.$logs(`Rate limit check passed for user: ${$ctx.$user.id}`);
```

> **Lưu ý:** Khuyến nghị dùng helper `$helpers.$rateLimit` vì nó sử dụng thuật toán cửa sổ trượt Redis để giới hạn chính xác hơn. Xem [Helpers & Cache - Rate Limiting](./context-reference/helpers-cache.md#rate-limiting) để biết đầy đủ.

### Mẫu 4: Ngăn chỉnh sửa đồng thời

Dùng khóa phân tán để ngăn chỉnh sửa đồng thời.

```javascript
const lockKey = `record-lock:${$ctx.$params.id}`;
const lockValue = $ctx.$user.id;

const lockAcquired = await $ctx.$cache.acquire(lockKey, lockValue, 10000);
if (!lockAcquired) {
  $ctx.$throw['409']('Record is currently being modified by another user');
  return;
}

try {
  // Get current record
  const current = await $ctx.$repos.products.find({
    filter: { id: { _eq: $ctx.$params.id } }
  });
  
  if (current.data.length === 0) {
    $ctx.$throw['404']('Product not found');
    return;
  }
  
  // Perform update
  const result = await $ctx.$repos.products.update({
    id: $ctx.$params.id,
    data: updateData
  });
  
  // Invalidate cache
  await $ctx.$cache.deleteKey(`product:${$ctx.$params.id}`);
  
} finally {
  await $ctx.$cache.release(lockKey, lockValue);
}
```

### Mẫu 5: Cache dữ liệu cấu hình

Cache dữ liệu cấu hình ít thay đổi.

```javascript
const configKey = 'app:configuration';
let config = await $ctx.$cache.get(configKey);

if (!config) {
  // Load from database
  const result = await $ctx.$repos.configurations.find({
    filter: { isActive: { _eq: true } }
  });
  
  if (result.data.length > 0) {
    config = result.data[0];
    // Cache for 1 hour
    await $ctx.$cache.set(configKey, config, 3600000);
  }
}

// Use config...
```

### Mẫu 6: Quản lý session

Lưu dữ liệu session vào cache kèm thời hạn hết hạn.

```javascript
// Create session
const sessionId = generateSessionId();
const sessionData = {
  userId: user.id,
  email: user.email,
  createdAt: new Date()
};

// Cache session for 7 days
await $ctx.$cache.set(`session:${sessionId}`, sessionData, 7 * 24 * 60 * 60 * 1000);

// Later: Retrieve session
const session = await $ctx.$cache.get(`session:${sessionId}`);
if (!session) {
  $ctx.$throw['401']('Session expired');
  return;
}
```

### Mẫu 7: Làm nóng cache

Nạp trước vào cache các dữ liệu được truy cập thường xuyên.

```javascript
// In a background process or bootstrap script
const popularProducts = await $ctx.$repos.products.find({
  filter: { isPopular: { _eq: true } },
  limit: 100
});

for (const product of popularProducts.data) {
  await $ctx.$cache.set(
    `product:${product.id}`,
    product,
    3600000 // 1 hour
  );
}
```

## Thực hành tốt

1. **Luôn dùng await** — mọi hàm cache đều async và cần await
2. **Đặt TTL phù hợp** — chọn TTL theo tần suất dữ liệu thay đổi
3. **Dùng mẫu khóa logic** — như `user:123`, `session:456`, `lock:789`; Enfyra tự thêm namespace app
4. **Luôn nhả khóa** — dùng try-finally để chắc chắn khóa được nhả
5. **Invalidate cache khi cập nhật** — xóa khóa cache lúc dữ liệu thay đổi
6. **Xử lý cache miss** — luôn có phương án fallback về database
7. **Dùng khóa cho thao tác trọng yếu** — ngăn chỉnh sửa đồng thời bằng khóa phân tán
8. **Tôn trọng giới hạn user cache** — giữ dữ liệu `$cache` có thể dựng lại vì LRU có thể xóa khóa cũ khi vượt ngưỡng

## Quy ước đặt tên khóa

Dùng quy ước nhất quán khi đặt tên khóa cache:

```javascript
// Resource by ID
`user:${userId}`
`product:${productId}`
`order:${orderId}`

// Locks
`lock:user:${userId}`
`lock:record:${recordId}`

// Rate limiting
`rate-limit:${userId}:${endpoint}`

// Sessions
`session:${sessionId}`

// Configuration
`config:${configName}`

// Aggregated data
`stats:daily:${date}`
```

## Bước tiếp theo

- Tìm hiểu [Context Reference](./context-reference/) để truy cập cache
- Xem [Cluster Architecture](./cluster-architecture.md) để hiểu điều phối phân tán
- Xem [Error Handling](./error-handling.md) để xử lý lỗi cache

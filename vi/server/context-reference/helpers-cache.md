---
slug: helpers-va-cache-context
---

# Tham chiếu Context - Helpers và bộ nhớ đệm

Các hàm tiện ích cho tác vụ phổ biến và thao tác cache phân tán.

## Hàm hỗ trợ

Các hàm tiện ích cho tác vụ phổ biến. Mọi hàm hỗ trợ đều cần `await`.

### Tạo token JWT

```javascript
const token = await $ctx.$helpers.$jwt(payload, expiration);

// Example
const token = await $ctx.$helpers.$jwt(
  { userId: 123, email: 'user@example.com' },
  '7d'  // Expires in 7 days
);
```

**Định dạng thời hạn:**
- - 15 phút.
- `'1h'`- 1 giờ
- `'1d'`- 1 ngày
- 7 ngày
- 30 Ngày

### Băm mật khẩu

```javascript
// Hash password
const hash = await $ctx.$helpers.$bcrypt.hash(plainPassword);

// Verify password
const isValid = await $ctx.$helpers.$bcrypt.compare(plainPassword, hashedPassword);

// Example
const hashedPassword = await $ctx.$helpers.$bcrypt.hash('myPassword123');
const isValid = await $ctx.$helpers.$bcrypt.compare('myPassword123', hashedPassword);
```

### Tự tạo slug

Tạo slug thân thiện với URL từ văn bản.

```javascript
const slug = $ctx.$helpers.autoSlug(text);

// Example
const slug = $ctx.$helpers.autoSlug('My Product Name');
// Result: 'my-product-name'
```

### Hàm hỗ trợ crypto

Sử dụng`$ctx.$helpers.$crypto`cho các trình trợ giúp mật mã bị chặn bên trong móc, trình xử lý, luồng và tập lệnh websocket.

```javascript
const id = $ctx.$helpers.$crypto.randomUUID();
const token = $ctx.$helpers.$crypto.randomBytes(32, 'base64url');
const digest = $ctx.$helpers.$crypto.sha256('payload');
const signature = $ctx.$helpers.$crypto.hmacSha256('payload', 'shared-secret');
```

Các mã hóa được hỗ trợ cho`randomBytes`⟭, ⟬1 và`hmacSha256`là`hex`⟭, ⟬4 và`base64url`.`randomBytes`được giới hạn ở 4096 byte.

Tạo khóa SSH với cùng một trình trợ giúp tiền điện tử:

```javascript
const keyPair = await $ctx.$helpers.$crypto.generateSshKeyPair('deploy@example.com');

// keyPair.publicKey is OpenSSH format.
// keyPair.privateKey is RSA PKCS#1 PEM.
```

Không sử dụng legacy`$ctx.$helpers.$ssh`hoặc trình trợ giúp mã hóa thủ công trong các tập lệnh mới. Các giá trị cơ sở dữ liệu cần mã hóa ở phần còn lại nên được lưu trữ trong các cột được đánh dấu`isEncrypted=true`; các tập lệnh đọc và ghi các giá trị văn bản thuần túy.

### Hàm sleep

Tạm dừng một tập lệnh động trong một khoảng thời gian giới hạn. Thời gian chạy kẹp độ trễ từ 0 đến 30000 mili giây.

```javascript
await $ctx.$helpers.$sleep(1000);
```

### Hàm hỗ trợ storage

Tải tệp lên storage.

```javascript
const fileResult = await $ctx.$storage.$upload({
  originalname: 'image.jpg',
  filename: 'custom-filename.jpg',
  mimetype: 'image/jpeg',
  buffer: fileBuffer,
  size: 1024000,
  folder: 123,  // Optional: folder ID
  storageConfig: 1,  // Optional: storage config ID
  title: 'My Image',  // Optional
  description: 'Image description'  // Optional
});
```

### Hàm cập nhật tệp

Cập nhật tệp đã có.

```javascript
await $ctx.$storage.$update(fileId, {
  buffer: newFileBuffer,
  originalname: 'new-name.jpg',
  mimetype: 'image/jpeg',
  folder: 456,
  title: 'Updated Title'
});
```

### Hàm xóa tệp

Xóa tệp.

```javascript
await $ctx.$storage.$delete(fileId);
```

Đăng ký một đối tượng đã tồn tại trong phần phụ trợ lưu trữ được định cấu hình mà không cần tải lên byte:

```javascript
await $ctx.$storage.$registerFile({
  filename: 'backup.sql.gz',
  mimetype: 'application/gzip',
  location: 'backups/project-1/backup.sql.gz',
  size: 1024,
  storageConfig: 1,
});
```

### Giới hạn tần suất

Bảo vệ các tuyến API của bạn với giới hạn tốc độ linh hoạt bằng cách sử dụng thuật toán cửa sổ trượt Redis.

```javascript
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
}
```

#### Mẫu giới hạn tần suất

| Phương thức | Định dạng khóa | Mô tả |
|--------|------------|-------------|
|`byIp(options)`|`ip:{ip}:{route}`| Giới hạn tốc độ theo IP máy khách trên mỗi tuyến đường |
| ⟬0 |`user:{userId}:{route}`⟭ | Giới hạn mức giá theo người dùng được xác thực trên mỗi tuyến đường |
| ⟬0 |`route:{route}`⟭ | Giới hạn tỷ lệ trên toàn cầu cho mỗi tuyến đường (tất cả người dùng/giới hạn chia sẻ IP) |
|`byIpGlobal(options)`|`ip:{ip}`| Giới hạn tốc độ theo IP trên tất cả các tuyến đường |
|`byUserGlobal(options)`|`user:{userId}`| Giới hạn mức giá của người dùng trên tất cả các tuyến đường |
|`check(key, options)`| Khóa tùy chỉnh | Giới hạn tốc độ với khóa tùy chỉnh |
|`reset(key)`| - | Đặt lại giới hạn tốc độ cho một khóa |
|`status(key, options)`| - | Kiểm tra trạng thái giới hạn tốc độ mà không tăng |

#### Tùy chọn

```javascript
{
  maxRequests: 100,  // Maximum requests allowed in the window
  perSeconds: 60     // Time window in seconds
}
```

#### Đối tượng kết quả

```javascript
{
  allowed: boolean,     // Whether the request is allowed
  remaining: number,    // Remaining requests in window
  resetAt: number,      // Unix timestamp when window resets
  retryAfter: number,   // Seconds to wait before retry (0 if allowed)
  limit: number,        // The max requests limit
  window: number        // The window in seconds
}
```

#### Ví dụ

**Số lần đăng nhập giới hạn tốc độ theo IP:**
```javascript
// In preHook for POST /auth/login
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Too many login attempts. Try again in ${result.retryAfter}s`);
}
```

** API giới hạn tỷ lệ theo người dùng:**
```javascript
// In preHook for API routes
const result = await $ctx.$helpers.$rateLimit.byUser({
  maxRequests: 1000,
  perSeconds: 3600  // 1 hour
});

if (!result.allowed) {
  $ctx.$throw['429'](`API rate limit exceeded. Try again in ${result.retryAfter}s`);
}
```

**Bỏ qua giới hạn tỷ lệ cho quản trị viên:**
```javascript
if (!$ctx.$user?.isRootAdmin) {
  const result = await $ctx.$helpers.$rateLimit.byIp({
    maxRequests: 100,
    perSeconds: 60
  });

  if (!result.allowed) {
    $ctx.$throw['429']('Rate limit exceeded');
  }
}
```

** Khóa tùy chỉnh cho tài nguyên cụ thể:**
```javascript
const resourceId = $ctx.$params.id;
const result = await $ctx.$helpers.$rateLimit.check(
  `resource:${resourceId}:${$ctx.$user?.id || $ctx.$req.ip}`,
  { maxRequests: 10, perSeconds: 60 }
);

if (!result.allowed) {
  $ctx.$throw['429']('Too many requests to this resource');
}
```

**Kiểm tra trạng thái không tăng:**
```javascript
// Check if rate limited without counting the request
const status = await $ctx.$helpers.$rateLimit.status(
  `ip:${$ctx.$req.ip}:/api/expensive`,
  { maxRequests: 5, perSeconds: 3600 }
);

if (!status.allowed) {
  $ctx.$throw['429']('Daily limit reached');
}
```

**Đặt lại giới hạn tỷ lệ sau khi hành động thành công:**
```javascript
// In postHook after successful action
await $ctx.$helpers.$rateLimit.reset(`ip:${$ctx.$req.ip}:/auth/forgot-password`);
```

## Cache

Các thao tác user cache và khóa phân tán. Mọi hàm cache đều cần `await`.

`$ctx.$cache`lưu trữ dữ liệu ứng dụng được tạo bởi trình xử lý, móc, luồng, tập lệnh websocket và`@CACHE`macro. Sử dụng các khóa logic như`user:123`; không bao gồm`NODE_NAME`⟭, ⟬4 hoặc bất kỳ tiền tố không gian tên Redis nào. Khi bộ nhớ cache người dùng Redis được bật, Enfyra lưu trữ các giá trị đó trong không gian tên ứng dụng hiện tại là`NODE_NAME:user_cache:*`.

Bộ nhớ cache của người dùng có phân bổ mềm được kiểm soát bởi`REDIS_USER_CACHE_LIMIT_MB`(mặc định`30`). Nếu phân bổ vượt quá, Enfyra chỉ loại bỏ các khóa bộ nhớ cache người dùng ít được sử dụng gần đây nhất. Các phím System Redis như ảnh chụp nhanh bộ nhớ cache thời gian chạy, hàng đợi BullMQ, Socket.IO, đo từ xa thời gian chạy và khóa không được tính hoặc loại bỏ bởi hạn ngạch này.

Với các kiểm tra proxy có lưu lượng lớn, chỉ cache catalog hoặc dữ liệu quyền đã suy ra bằng TTL ngắn, rõ ràng. Credential, thu hồi token và quyết định bảo mật khác phải được kiểm tra trực tiếp; cache hit không bao giờ tự chứng minh credential còn hợp lệ.

### Lấy giá trị cache

```javascript
const cachedValue = await $ctx.$cache.get(key);

// Example
const user = await $ctx.$cache.get('user:123');
```

### Đặt giá trị cache

```javascript
// Set with TTL (time-to-live in milliseconds)
await $ctx.$cache.set(key, value, ttlMs);

// Example - cache for 5 minutes
await $ctx.$cache.set('user:123', userData, 300000);

// Set without expiration
await $ctx.$cache.setNoExpire(key, value);
```

Ưu tiên`set(key, value, ttlMs)`với TTL cho dữ liệu hoạt động.`setNoExpire`giữ một giá trị liên tục từ góc độ TTL, nhưng nó vẫn có thể bị trục xuất nếu vượt quá phân bổ bộ đệm người dùng.

### Khóa phân tán

Nhận và nhả khóa cho các thao tác quan trọng.

```javascript
// Acquire lock
const lockAcquired = await $ctx.$cache.acquire(key, value, ttlMs);

// Release lock
const released = await $ctx.$cache.release(key, value);

// Example
const lockKey = `user-lock:${userId}`;
const lockValue = $ctx.$user.id;
const acquired = await $ctx.$cache.acquire(lockKey, lockValue, 10000); // 10 seconds

if (acquired) {
  try {
    // Critical operation here
  } finally {
    await $ctx.$cache.release(lockKey, lockValue);
  }
}
```

### Kiểm tra cache tồn tại

```javascript
const exists = await $ctx.$cache.exists(key, value);

// Example
const lockExists = await $ctx.$cache.exists('user-lock:123', 'user-456');
```

### Xóa khóa cache

```javascript
await $ctx.$cache.deleteKey(key);

// Example
await $ctx.$cache.deleteKey('user:123');
```

## Tiếp theo

- Xem[Logging & Error Handling](./logging-errors.md)để ghi nhật ký và ném lỗi
- Chọn[Advanced Features](./advanced.md)để tải lên tệp và hơn thế nữa
- Tìm hiểu về[Cache Operations](../cache-operations.md)cho các mẫu bộ nhớ cache chi tiết

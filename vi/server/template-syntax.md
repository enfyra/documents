---
slug: cu-phap-mau
---

# Hướng dẫn cú pháp mẫu

Enfyra có **ba cách tương đương** để truy cập thuộc tính context. Bạn có thể dùng cú pháp đầy đủ `$ctx.$property`, cú pháp template hoặc cú pháp bảng trực tiếp; cả ba hoạt động như nhau.

## Tổng quan

**Cả ba cú pháp đều được hỗ trợ đầy đủ và tương đương:**

```javascript
//  Cú pháp đầy đủ (luôn hoạt động)
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.enfyra_user.find({...});
const slug = $ctx.$helpers.autoSlug('Hello World');
$ctx.$logs('Operation completed');

//  Cú pháp template (lối tắt tiện dụng)
const data = await @CACHE.get('key');
const users = await @REPOS.enfyra_user.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;

//  Cú pháp bảng trực tiếp (ngắn nhất khi truy cập database)
const data = await @CACHE.get('key');
const users = await #enfyra_user.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;
```

**Cú pháp template chỉ là syntactic sugar** - Enfyra Server thay nó bằng cú pháp đầy đủ `$ctx.$property` trước khi code biên dịch được chuyển cho kernel executor. **Bạn có thể dùng phong cách mình thích, kể cả trộn cả ba trong cùng một file.**

Trong tuỳ chọn **`find()`**, truyền điều kiện qua **`filter`**. Endpoint danh sách REST cũng dùng tên query parameter là **`filter`**.

## Template có sẵn

| Template | Thay thế thành | Mô tả |
|----------|-------------|-------------|
| `@CACHE` | `$ctx.$cache` | Thao tác cache và distributed lock |
| `@REPOS` | `$ctx.$repos` | Truy cập database repository |
| `@HELPERS` | `$ctx.$helpers` | Hàm tiện ích và helper |
| `@FETCH` | `$ctx.$helpers.$fetch` | HTTP client đã tăng cường chống SSRF (viết tắt của `@HELPERS.$fetch`) |
| `@LOGS` | `$ctx.$logs` | Hàm ghi log |
| `@BODY` | `$ctx.$body` | Dữ liệu request body |
| `@ENV` | `$ctx.$env` | Biến môi trường đã được làm sạch do host runtime cung cấp |
| `@DATA` | `$ctx.$data` | Đối tượng dữ liệu response |
| `@STATUS` | `$ctx.$statusCode` | HTTP status code (200 khi thành công, error code khi thất bại — có trong postHook) |
| `@ERROR` | `$ctx.$error` | Error context trong postHook (`{ message, name, statusCode, details, timestamp }` — `undefined` khi thành công) |
| `@PARAMS` | `$ctx.$params` | Tham số route |
| `@QUERY` | `$ctx.$query` | Tham số query |
| `@USER` | `$ctx.$user` | Thông tin người dùng hiện tại |
| `@REQ` | `$ctx.$req` | Đối tượng request của Express |
| `@RES` | `$ctx.$res` | Đối tượng response của Express (chỉ handler) |
| `@SHARE` | `$ctx.$share` | Dữ liệu dùng chung giữa các hook |
| `@API` | `$ctx.$api` | Thông tin request/response API |
| `@SOCKET` | `$ctx.$socket` | Thao tác WebSocket (join, leave, reply, emitToUser, emitToRoom, emitToCurrentRoom, broadcastToRoom, emitToGateway, broadcast; `disconnect` chỉ trong connection handler) |
| `@TRIGGER` | `$ctx.$trigger` | Kích hoạt flow theo id hoặc tên (`@TRIGGER(flowIdOrName, payload?)`) |
| `@FLOW` | `$ctx.$flow` | Flow context hiện tại trong flow step (payload, output step trước, meta) |
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` | Payload gốc truyền vào flow |
| `@FLOW_LAST` | `$ctx.$flow.$last` | Output của flow step trước |
| `@FLOW_META` | `$ctx.$flow.$meta` | Metadata thực thi flow (id, name, runId, v.v.) |
| `@UPLOADED_FILE` | `$ctx.$uploadedFile` | Thông tin file đã tải lên |
| `@PKGS` | `$ctx.$pkgs` | npm package đã cài để dùng trong handler |
| `@THROW` | `$ctx.$throw` | Hàm ném lỗi |
| `@THROW400` | `$ctx.$throw['400']` | HTTP 400 Bad Request (lối tắt) |
| `@THROW401` | `$ctx.$throw['401']` | HTTP 401 Unauthorized (lối tắt) |
| `@THROW403` | `$ctx.$throw['403']` | HTTP 403 Forbidden (lối tắt) |
| `@THROW404` | `$ctx.$throw['404']` | HTTP 404 Not Found (lối tắt) |
| `@THROW409` | `$ctx.$throw['409']` | HTTP 409 Conflict (lối tắt) |
| `@THROW422` | `$ctx.$throw['422']` | HTTP 422 Validation Error (lối tắt) |
| `@THROW429` | `$ctx.$throw['429']` | HTTP 429 Rate Limit Exceeded (lối tắt) |
| `@THROW500` | `$ctx.$throw['500']` | HTTP 500 Internal Error (lối tắt) |
| `@THROW503` | `$ctx.$throw['503']` | HTTP 503 Service Unavailable (lối tắt) |
| `#table_name` | `$ctx.$repos.table_name` | Truy cập bảng trực tiếp (ví dụ `#enfyra_user`, `#product`) |
| `%pkg_name` | `$ctx.$pkgs.pkg_name` | Truy cập package viết tắt (ví dụ `%axios`, `%lodash`, `%moment`) |

## Ví dụ sử dụng

### Thao tác cache

`@CACHE` trỏ tới cùng managed user cache với `$ctx.$cache`. Chỉ dùng logical key; Enfyra tự áp namespace của app hiện tại. Không đưa `NODE_NAME`, `user_cache:` hay tiền tố Redis vào code template. Dữ liệu user cache bị giới hạn bởi `REDIS_USER_CACHE_LIMIT_MB` (mặc định `30` MB); Enfyra sẽ loại các key ít dùng nhất khi vượt giới hạn.

```javascript
// Lấy dữ liệu từ cache
const cachedData = await @CACHE.get('user:123');

// Lưu dữ liệu vào cache kèm TTL
await @CACHE.set('user:123', userData, 3600000); // 1 hour

// Kiểm tra key có tồn tại
const exists = await @CACHE.exists('user:123');

// Xoá khỏi cache
await @CACHE.deleteKey('user:123');

// Distributed lock
const lockAcquired = await @CACHE.acquire('critical-operation', 'instance-1', 30000);
if (lockAcquired) {
  try {
// Thao tác quan trọng ở đây
    await performCriticalOperation();
  } finally {
    await @CACHE.release('critical-operation', 'instance-1');
  }
}
```

### Thao tác database

#### Dùng cú pháp @REPOS:
```javascript
// Tìm bản ghi có filter và phân trang
const users = await @REPOS.enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name',   // Only fetch required fields
  limit: 10,                  // Max 10 records (default: 10)
  sort: '-createdAt'          // Sort by createdAt DESC
});

// Lấy TOÀN BỘ bản ghi (không giới hạn)
const allUsers = await @REPOS.enfyra_user.find({
  filter: { isActive: true },
  limit: 0  // 0 = fetch all
});

// Sắp xếp nhiều trường
const sorted = await @REPOS.enfyra_user.find({
  sort: 'name,-createdAt'  // Sort by name ASC, then createdAt DESC
});

// Quan hệ lồng nhau (lấy dữ liệu liên quan trong MỘT query)
const usersWithPosts = await @REPOS.enfyra_user.find({
  fields: 'id,email,posts.title,posts.createdAt',  // Nested field: posts.title
  filter: { isActive: true }
});

// Lọc theo quan hệ lồng nhau
const usersInRole = await @REPOS.enfyra_user.find({
  filter: {
    role: {
      name: { _eq: 'Admin' }  // Filter by related role name
    }
  },
  fields: 'id,email,role.name'
});

// Tạo bản ghi mới
const newUser = await @REPOS.enfyra_user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    isActive: true
  }
});

// Cập nhật bản ghi theo ID
const updatedUser = await @REPOS.enfyra_user.update({
  id: userId,
  data: {
    name: 'Jane Doe',
    lastLogin: new Date()
  }
});

// Xoá bản ghi theo ID
await @REPOS.enfyra_user.delete({ id: userId });
```

#### Dùng cú pháp #table_name (ngắn hơn):
```javascript
// Tìm bản ghi có filter và phân trang
const users = await #enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name',   // Only fetch required fields
  limit: 10,                  // Max 10 records (default: 10)
  sort: '-createdAt'          // Sort by createdAt DESC
});

// Lấy TOÀN BỘ bản ghi (không giới hạn)
const allUsers = await #enfyra_user.find({
  filter: { isActive: true },
  limit: 0  // 0 = fetch all
});

// Sắp xếp nhiều trường
const sorted = await #enfyra_user.find({
  sort: 'name,-createdAt'  // Sort by name ASC, then createdAt DESC
});

// Quan hệ lồng nhau (lấy dữ liệu liên quan trong MỘT query)
const usersWithPosts = await #enfyra_user.find({
  fields: 'id,email,posts.title,posts.createdAt',  // Nested field: posts.title
  filter: { isActive: true }
});

// Lọc theo quan hệ lồng nhau
const usersInRole = await #enfyra_user.find({
  filter: {
    role: {
      name: { _eq: 'Admin' }  // Filter by related role name
    }
  },
  fields: 'id,email,role.name'
});

// Tạo bản ghi mới
const newUser = await #enfyra_user.create({ data: {
  email: 'user@example.com',
  name: 'John Doe',
  isActive: true
}});

// Cập nhật bản ghi theo ID
const updatedUser = await #enfyra_user.update({ id: userId, data: {
  name: 'Jane Doe',
  lastLogin: new Date()
}});

// Xoá bản ghi theo ID
await #enfyra_user.delete({ id: userId });
```

### Hàm helper

```javascript
// Tạo JWT token (gọi $jwt như một hàm: payload, expiresIn)
const token = await @HELPERS.$jwt({ userId: 123, role: 'admin' }, '1h');

// Hash mật khẩu (một đối số; salt round do hệ thống tự quản lý)
const hashedPassword = await @HELPERS.$bcrypt.hash('password123');

// Xác thực mật khẩu
const isValid = await @HELPERS.$bcrypt.compare('password123', hashedPassword);

// Tạo slug thân thiện với URL
const slug = @HELPERS.autoSlug('Hello World!'); // "hello-world"
```

### Dùng package

**Cú pháp truyền thống:**
```javascript
// Truy cập npm package đã cài
const axios = $ctx.$pkgs.axios;
const lodash = $ctx.$pkgs.lodash;
const moment = $ctx.$pkgs.moment;

// Dùng package như bình thường
const response = await axios.get('https://api.example.com/data');

// Chuyển đổi dữ liệu bằng lodash
const grouped = lodash.groupBy(response.data, 'category');

// Định dạng ngày bằng moment
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Cú pháp template (rút gọn):**
```javascript
// Access installed npm packages
const axios = @PKGS.axios;
const lodash = @PKGS.lodash;
const moment = @PKGS.moment;

// Dùng package như bình thường
const response = await axios.get('https://api.example.com/data');

// Transform data with lodash
const grouped = lodash.groupBy(response.data, 'category');

// Format dates with moment
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Cú pháp viết tắt (`%`):**
```javascript
// Truy cập package trực tiếp - ngắn nhất
const axios = %axios;
const lodash = %lodash;
const moment = %moment;

const response = await axios.get('https://api.example.com/data');
const summary = lodash.groupBy(response.data, 'category');
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Cú pháp kết hợp:**
```javascript
// Bạn có thể kết hợp cả ba cách!
const axios = %axios;                         // Shorthand syntax
const lodash = @PKGS.lodash;                 // Template syntax  
const moment = $ctx.$pkgs.moment;             // Traditional syntax

const response = await axios.get('https://api.example.com/data');
const summary = lodash.groupBy(response.data, 'category');
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

### Ghi log

```javascript
// Ghi log cơ bản
@LOGS('User operation started');

// Ghi log kèm dữ liệu
@LOGS('User created:', { id: 123, email: 'user@example.com' });

// Nhiều tham số
@LOGS('Cache operation', 'key:', 'user:123', 'result:', cachedData);

// Ghi log lỗi
@LOGS('Error occurred:', error.message, error.stack);
```

### Tải file lên và streaming

```javascript
// Truy cập file đã tải lên
const file = @UPLOADED_FILE;
@LOGS('File uploaded:', file.originalname, file.mimetype, file.size);

// Lưu file request đã tải lên vào storage và enfyra_file.
// Thao tác stream từ file tạm của server, không buffer toàn bộ file.
const savedFile = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  description: @BODY.description
});

// Stream response (cho file lớn hoặc xử lý ảnh)
const { Readable } = require('stream');
const sharp = @PKGS.sharp;

// Tải xuống và thay đổi kích thước ảnh
const response = await fetch(@QUERY.imageUrl);
const stream = Readable.fromWeb(response.body);

const transformer = sharp()
  .resize(800, 600, { fit: 'inside' })
  .jpeg({ quality: 85 });

// Stream tới client (tiết kiệm bộ nhớ)
@RES.stream(stream.pipe(transformer), {
  mimetype: 'image/jpeg',
  filename: 'resized-image.jpg'
});
```

** Xem [File Handling](./file-handling.md)** để biết đầy đủ về tải file lên, streaming và xử lý ảnh.

### Xử lý lỗi

**Lỗi HTTP status code (`@THROW`):**

**Cách 1: Dùng ngoặc vuông và dấu nháy**
```javascript
// Ném lỗi HTTP 400 Bad Request
@THROW['400']('Email is required');

// Ném lỗi HTTP 401 Unauthorized
@THROW['401']('Invalid credentials');

// Ném lỗi HTTP 403 Forbidden
@THROW['403']('Insufficient permissions');

// Ném lỗi HTTP 404 Not Found
@THROW['404']('User not found', 'user_id_123');

// Ném lỗi HTTP 409 Conflict (khi trùng lặp)
@THROW['409']('Email already exists', 'email', 'user@example.com');

// Ném lỗi HTTP 422 Validation Error
@THROW['422']('Invalid data format');

// Ném lỗi HTTP 500 Internal Server Error
@THROW['500']('Database connection failed');
```

**Cách 2: Lối tắt trực tiếp (không cần dấu nháy)**
```javascript
// Ném lỗi HTTP 400 Bad Request
@THROW400('Email is required');

// Ném lỗi HTTP 401 Unauthorized  
@THROW401('Invalid credentials');

// Ném lỗi HTTP 403 Forbidden
@THROW403('Insufficient permissions');

// Ném lỗi HTTP 404 Not Found
@THROW404('User not found', 'user_id_123');

// Ném lỗi HTTP 409 Conflict (khi trùng lặp)
@THROW409('Email already exists');

// Ném lỗi HTTP 422 Validation Error
@THROW422('Invalid data format');

// Ném lỗi HTTP 429 Rate Limit Exceeded
@THROW429(100, 'per minute');

// Ném lỗi HTTP 500 Internal Server Error
@THROW500('Database connection failed');

// Ném lỗi HTTP 503 Service Unavailable
@THROW503('Service unavailable');
```

## Cách dùng nâng cao

### Chuỗi thao tác

```javascript
// Cache kèm database fallback
let data = await @CACHE.get('products:featured');
if (!data) {
  data = await #products.find({
    filter: { featured: true },
    fields: 'id,name,price,image'
  });
  await @CACHE.set('products:featured', data, 3600000);
}
@LOGS('Featured products loaded:', data.length);
```

### Xử lý lỗi kèm logging

```javascript
try {
  const user = await #enfyra_user.find({ filter: { id: userId } });
  if (!user.data.length) {
    @THROW404('User not found');
  }
  
  const updatedUser = await #enfyra_user.update({
    id: userId,
    data: { lastLogin: new Date() }
  });
  
  @LOGS('User login updated:', updatedUser.id);
  return updatedUser;
  
} catch (error) {
  @LOGS('User update failed:', error.message);
  throw error;
}
```

### Logic nghiệp vụ phức tạp

```javascript
// User registration with validation and caching
async function registerUser(userData) {
  // Kiểm tra người dùng đã tồn tại
  const existingUser = await #enfyra_user.find({
    filter: { email: userData.email },
    fields: 'id'
  });
  
  if (existingUser.data.length > 0) {
    @THROW409('Email already exists');
  }
  
  // Hash mật khẩu
  const hashedPassword = await @HELPERS.$bcrypt.hash(userData.password);
  
  // Tạo người dùng
  const newUser = await #enfyra_user.create({ data: {
    ...userData,
    password: hashedPassword,
    createdAt: new Date()
  });
  
  // Cache dữ liệu người dùng
  await @CACHE.set(`user:${newUser.id}`, newUser, 1800000); // 30 minutes
  
  // Tạo JWT token
  const token = await @HELPERS.$jwt({ userId: newUser.id }, '24h');
  
  @LOGS('User registered successfully:', newUser.id);
  
  return { user: newUser, token };
}
```

## Cách thay thế template hoạt động

Cú pháp template **chỉ là tiện ích về cú pháp**. Bên trong, Enfyra xử lý như sau:

1. **Gửi code**: Code có `@CACHE`, `@REPOS`, v.v. được gửi lên.
2. **Xử lý template**: Template tự động được thay bằng cú pháp đầy đủ `$ctx.$property`.
3. **Thực thi code**: Code đã xử lý chạy bình thường trong handler executor.
4. **Trả kết quả**: Việc thực thi tiếp tục bình thường với cú pháp đã thay thế.

### Ví dụ chuyển đổi

```javascript
// Code của bạn (cú pháp template):
const data = await @CACHE.get('key');
const users = await #enfyra_user.find({...});

// Nội dung thực sự chạy (sau khi thay thế):
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.enfyra_user.find({...});
```

** Việc thay thế diễn ra trong suốt** - bạn có thể trộn cả ba cú pháp trong cùng một file nếu muốn.

## Thực hành tốt

### 1. Chọn phong cách của bạn (cách nào cũng dùng được)

```javascript
//  Cách 1 - Dùng cú pháp đầy đủ xuyên suốt
const user = await $ctx.$repos.users.find({ filter: { id: userId } });
await $ctx.$cache.set(`user:${userId}`, user);
$ctx.$logs('User cached:', userId);

//  Cách 2 - Dùng cú pháp template xuyên suốt  
const user = await @REPOS.users.find({ filter: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

//  Cách 3 - Cú pháp bảng trực tiếp (ngắn nhất khi truy cập database)
const user = await #enfyra_user.find({ filter: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

//  Cách 4 - Trộn cả ba (hoàn toàn ổn)
const user = await #enfyra_user.find({ filter: { id: userId } });        // Direct table
await $ctx.$cache.set(`user:${userId}`, user);                    // Full syntax
@LOGS('User cached:', userId);                                     // Template syntax

//  Cách 5 - Tổ hợp nào cũng hoạt động!
const user = await $ctx.$repos.enfyra_user.find({ filter: { id: userId } }); // Full
await @CACHE.set(`user:${userId}`, user);                             // Template
const products = await #product.find({ filter: { userId } }); // Direct
$ctx.$logs('All operations completed');                               // Full
```

### 2. Tận dụng chọn trường

```javascript
//  Tốt - chỉ lấy trường cần thiết
const users = await #enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name' // Performance optimization
});

//  Tốt - lấy source script có thể sửa mà không lấy compiled output sinh sẵn
const handlers = await #enfyra_route_handler.find({
  fields: '-compiledCode'
});

//  Tránh lấy mọi trường khi không cần thiết
const users = await #enfyra_user.find({
  filter: { isActive: true }
  // Mặc định lấy mọi trường
});
```

Khi bất kỳ tên trường nào bắt đầu bằng `-`, phạm vi `fields` đó dùng chế độ loại trừ. Ví dụ, `fields: 'id,-compiledCode'` vẫn trả về mọi trường có thể đọc trừ `compiledCode`.

### 3. Xử lý lỗi đúng cách

```javascript
//  Tốt - xử lý lỗi đầy đủ
try {
  const result = await @REPOS.users.create({ data: userData });
  @LOGS('User created successfully');
  return result;
} catch (error) {
  @LOGS('User creation failed:', error.message);
  @THROW500('Failed to create user');
}
```

### 4. Chiến lược cache

```javascript
//  Tốt - cache với TTL phù hợp
const cacheKey = `user:${userId}`;
let user = await @CACHE.get(cacheKey);

if (!user) {
  user = await @REPOS.users.find({ filter: { id: userId } });
  await @CACHE.set(cacheKey, user, 1800000); // 30 minutes
}
```

## Tương thích và linh hoạt

- ** Hỗ trợ cả ba cú pháp**: Dùng `$ctx.$property`, `@TEMPLATE` hoặc `#table_name` theo lựa chọn của bạn.
- ** Có thể kết hợp**: Bạn có thể dùng bất kỳ tổ hợp nào trong cùng một file.
- ** Dùng ở mọi context**: Hoạt động trong Bootstrap Script, Hook và Custom Handler.
- ** Hỗ trợ IDE**: `$ctx.$property` có autocomplete tốt hơn.
- ** Debugging**: Stack trace hiển thị cú pháp `$ctx.$property` sau xử lý.
- ** Không ảnh hưởng hiệu năng**: Thay template chỉ là xử lý chuỗi.
- ** Linh hoạt tối đa**: Chọn cú pháp phù hợp nhất với phong cách của bạn.

## Tài liệu liên quan

- [Context Reference](./context-reference/README.md) - Tài liệu đầy đủ về đối tượng `$ctx`.
- [File Handling](./file-handling.md) - Hướng dẫn tải file và response streaming.
- [Custom Handlers](./hooks-handlers/custom-handlers.md) - Dùng template trong handler.
- [PreHooks](./hooks-handlers/prehooks.md) - Dùng template trong prehook.
- [postHooks](./hooks-handlers/posthooks.md) - Dùng template trong postHook.

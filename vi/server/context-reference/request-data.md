---
slug: du-lieu-request-context
---

# Tham chiếu Context - Dữ liệu request

Truy cập thông tin và tham số của request HTTP.

## $ctx.$body

Dữ liệu body của request (`POST` và `PATCH`).

```javascript
// Access request body
const email = $ctx.$body.email;
const name = $ctx.$body.name;

// Modify request body (in preHook)
$ctx.$body.email = $ctx.$body.email.toLowerCase();
```

## $ctx.$params

Tham số trên đường dẫn URL được khai báo trong route.

```javascript
// Route: /users/:id
const userId = $ctx.$params.id;

// Route: /orders/:orderId/products/:productId
const orderId = $ctx.$params.orderId;
const productId = $ctx.$params.productId;
```

## $ctx.$query

Tham số query string trên URL.

```javascript
// URL: /products?page=1&limit=20&sort=-price
const page = $ctx.$query.page;      // 1
const limit = $ctx.$query.limit;    // 20
const sort = $ctx.$query.sort;      // "-price"

// Access filter from query
const filter = $ctx.$query.filter;  // Filter object from ?filter={...}
```

## $ctx.$user

Thông tin người dùng đang được xác thực.

```javascript
const userId = $ctx.$user.id;
const userEmail = $ctx.$user.email;
const userRole = $ctx.$user.role;

// Check if user is authenticated
if (!$ctx.$user) {
  $ctx.$throw['401']('Unauthorized');
  return;
}
```

## $ctx.$req

Đối tượng request Express cùng các thông tin bổ sung.

```javascript
const method = $ctx.$req.method;        // 'GET', 'POST', 'PATCH', 'DELETE'
const url = $ctx.$req.url;              // Full request URL
const ip = $ctx.$req.ip;                // Client IP address
const headers = $ctx.$req.headers;      // Request headers
const userAgent = $ctx.$req.headers['user-agent'];
```

## $ctx.$data

Dữ liệu response (có trong postHook và handler).

```javascript
// In postHook - modify response data
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

## $ctx.$statusCode

Mã trạng thái HTTP. Có thể thay đổi trong hook.

```javascript
// Change status code
$ctx.$statusCode = 201;  // Created

// Check current status
if ($ctx.$statusCode === 200) {
  // Success response
}
```

## Tiếp theo

- Xem [Repository](./repositories.md) để thao tác cơ sở dữ liệu
- Xem [Hàm hỗ trợ và cache](./helpers-cache.md) để dùng các hàm tiện ích
- Tìm hiểu [Ghi log và xử lý lỗi](./logging-errors.md)

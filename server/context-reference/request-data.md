# Context Reference - Request Data

Access HTTP request information and parameters.

## $ctx.$body

Request body data (POST, PATCH, PUT requests).

```javascript
// Access request body
const email = $ctx.$body.email;
const name = $ctx.$body.name;

// Modify request body (in preHook)
$ctx.$body.email = $ctx.$body.email.toLowerCase();
```

## $ctx.$params

URL path parameters from route definitions.

```javascript
// Route: /users/:id
const userId = $ctx.$params.id;

// Route: /orders/:orderId/products/:productId
const orderId = $ctx.$params.orderId;
const productId = $ctx.$params.productId;
```

## $ctx.$query

Query string parameters from URL.

```javascript
// URL: /products?page=1&limit=20&sort=-price
const page = $ctx.$query.page;      // 1
const limit = $ctx.$query.limit;    // 20
const sort = $ctx.$query.sort;      // "-price"

// Access filter from query
const filter = $ctx.$query.filter;  // Filter object from ?filter={...}
```

## $ctx.$user

Current authenticated user information.

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

Express request object with additional details.

```javascript
const method = $ctx.$req.method;        // 'GET', 'POST', 'PATCH', 'DELETE'
const url = $ctx.$req.url;              // Full request URL
const ip = $ctx.$req.ip;                // Client IP address
const headers = $ctx.$req.headers;      // Request headers
const userAgent = $ctx.$req.headers['user-agent'];
```

## $ctx.$data

Response data (available in afterHook and handlers).

```javascript
// In afterHook - modify response data
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

## $ctx.$statusCode

HTTP status code. Can be modified in hooks.

```javascript
// Change status code
$ctx.$statusCode = 201;  // Created

// Check current status
if ($ctx.$statusCode === 200) {
  // Success response
}
```

## Next Steps

- See [Repositories](./repositories.md) for database operations
- Check [Helpers & Cache](./helpers-cache.md) for utility functions
- Learn about [Logging & Error Handling](./logging-errors.md)


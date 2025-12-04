# Error Handling

Enfyra provides built-in error handling mechanisms for throwing HTTP errors and handling exceptions in hooks and handlers.

## Quick Navigation

- [Throwing Errors](#throwing-errors) - How to throw HTTP errors
- [HTTP Status Codes](#http-status-codes) - Available status codes
- [Error Handling in afterHook](#error-handling-in-afterhook) - Handling errors that occurred
- [Common Patterns](#common-patterns) - Real-world error handling examples

## Throwing Errors

Use `$ctx.$throw` to throw HTTP errors with appropriate status codes.

### Basic Syntax

```javascript
$ctx.$throw['STATUS_CODE']('Error message');
```

### Common HTTP Status Codes

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

// Internal Server Error (500)
$ctx.$throw['500']('Internal server error');
```

## HTTP Status Codes

### 400 Bad Request

Use for invalid input data or malformed requests.

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

### 401 Unauthorized

Use when authentication is required but not provided.

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

### 403 Forbidden

Use when user is authenticated but doesn't have permission.

```javascript
if ($ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Admin access required');
  return;
}

// Check resource ownership
const resource = await $ctx.$repos.resources.find({
  where: { id: { _eq: $ctx.$params.id } }
});

if (resource.data[0].userId !== $ctx.$user.id) {
  $ctx.$throw['403']('Access denied');
  return;
}
```

### 404 Not Found

Use when requested resource doesn't exist.

```javascript
const product = await $ctx.$repos.products.find({
  where: { id: { _eq: $ctx.$params.id } }
});

if (product.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}
```

### 409 Conflict

Use when there's a conflict with current state (e.g., duplicate entry).

```javascript
// Check if email already exists
const existing = await $ctx.$repos.user_definition.find({
  where: { email: { _eq: $ctx.$body.email } }
});

if (existing.data.length > 0) {
  $ctx.$throw['409']('Email already exists');
  return;
}
```

### 422 Unprocessable Entity

Use for validation errors when data format is correct but business rules fail.

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

### 500 Internal Server Error

Use for unexpected server errors.

```javascript
try {
  // Some operation
} catch (error) {
  $ctx.$logs(`Unexpected error: ${error.message}`);
  $ctx.$throw['500']('Internal server error');
  return;
}
```

## Error Handling in afterHook

In afterHook, you can check if an error occurred during the request and handle it appropriately.

### Checking for Errors

```javascript
// In afterHook
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

**Important:** `$ctx.$api.error` is only available in afterHook, not in preHook.

### Error Object Properties

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

### Logging Errors

```javascript
// In afterHook
if ($ctx.$api.error) {
  $ctx.$logs(`Error occurred: ${$ctx.$api.error.message}`);
  $ctx.$logs(`Error status: ${$ctx.$api.error.statusCode}`);
  $ctx.$logs(`Error stack: ${$ctx.$api.error.stack}`);
}
```

### Creating Error Logs

```javascript
// In afterHook
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

## Common Patterns

### Pattern 1: Validate Required Fields

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

### Pattern 2: Check Resource Existence

```javascript
// In preHook or handler
const resource = await $ctx.$repos.products.find({
  where: { id: { _eq: $ctx.$params.id } }
});

if (resource.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Use resource.data[0] safely
const product = resource.data[0];
```

### Pattern 3: Validate Business Rules

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
  const existing = await $ctx.$repos.user_definition.find({
    where: { email: { _eq: $ctx.$body.email } }
  });
  
  if (existing.data.length > 0) {
    $ctx.$throw['409']('Email already exists');
    return;
  }
}
```

### Pattern 4: Permission Checking

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
  where: { id: { _eq: $ctx.$params.id } }
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

### Pattern 5: Error Recovery in afterHook

```javascript
// In afterHook
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

### Pattern 6: Try-Catch for External Operations

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

### Pattern 7: Validate Data Types

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

## Best Practices

1. **Throw errors early** - Validate and throw errors as soon as possible (in preHooks)
2. **Use appropriate status codes** - Choose the right HTTP status code for each error type
3. **Provide clear error messages** - Help users understand what went wrong
4. **Log errors** - Use `$ctx.$logs()` to log errors for debugging
5. **Handle errors gracefully** - Use afterHook to handle errors that occurred in handlers
6. **Don't expose sensitive information** - Don't include internal details in error messages
7. **Return early** - Use `return` after throwing errors to stop execution

## Next Steps

- Learn about [Context Reference](./context-reference/) for error-related properties
- See [API Lifecycle](./api-lifecycle.md) to understand when errors occur
- Check [Hooks and Handlers](./hooks-handlers/) for error handling in different phases


# Context Reference - Logging & Error Handling

Add logs to API responses and throw HTTP errors with proper status codes.

## Logging

Add logs to API responses. Logs are automatically included in the response.

### Basic Logging

```javascript
$ctx.$logs('Operation started');
$ctx.$logs('User created successfully');
$ctx.$logs(`Processing order: ${orderId}`);
```

### Logging Variables

```javascript
const userId = $ctx.$user.id;
const orderId = 123;

$ctx.$logs(`User ID: ${userId}`);
$ctx.$logs(`Order ID: ${orderId}`);
$ctx.$logs(`Processing data:`, dataObject);
```

### Logging in Different Phases

```javascript
// In preHook
$ctx.$logs('Validation started');

// In handler
$ctx.$logs('Creating user...');

// In afterHook
$ctx.$logs('User created successfully');
```

**Note:** Logs appear automatically in API responses. You don't need to manually include them in the return value.

## Error Handling

Throw HTTP errors with proper status codes.

### Throw HTTP Status Errors

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

// Internal Server Error
$ctx.$throw['500']('Internal server error');
```

### Error Handling in AfterHook

In afterHook, you can check for errors that occurred during the request:

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

**Note:** `$ctx.$api.error` is only available in afterHook, not in preHook.

## Next Steps

- See [Error Handling](./error-handling.md) for complete error handling guide
- Check [Advanced Features](./advanced.md) for API information and error details
- Learn about [Hooks and Handlers](../hooks-handlers/) for using logging in hooks


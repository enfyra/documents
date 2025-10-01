# Context Reference ($ctx)

The `$ctx` object is the central context that provides access to all system functions, data, and utilities in Enfyra. It's available in both [Hooks](./hook-development.md) and [Custom Handlers](../frontend/custom-handlers.md).

## Overview

The context object provides a unified interface for:
- **Request Data**: Access to HTTP request information
- **Database Operations**: Repository access for CRUD operations
- **Utility Functions**: Helper functions for common tasks
- **Cache Operations**: Distributed caching and locking
- **Logging System**: Built-in logging for debugging and audit trails
- **Package Access**: NPM packages installed in the system

## Request Data

### HTTP Request Information
```javascript
$ctx.$body        // Request body (POST/PATCH data)
$ctx.$params      // URL parameters (e.g., :id from /products/:id)
$ctx.$query       // Query string parameters (e.g., ?page=1&limit=10)
$ctx.$user        // Current authenticated user information
```

### Request Details
```javascript
$ctx.$req.method  // HTTP method (GET, POST, PATCH, DELETE)
$ctx.$req.url     // Full request URL
$ctx.$req.ip      // Client IP address
$ctx.$req.headers // Request headers (authorization, user-agent)
```

## Database Access

### Repository Access
```javascript
$ctx.$repos       // Access to table repositories
// Use the table names configured in route's targetTables
$ctx.$repos.products    // If "products" is in targetTables
$ctx.$repos.categories  // If "categories" is in targetTables
$ctx.$repos.users       // If "users" is in targetTables
```

**Important**: Configure the tables you need in the route's **Target Tables** field. Each target table becomes available as a repository in `$ctx.$repos`.

### Repository Methods
Each repository provides full CRUD operations:

```javascript
// Find records (returns {data: [], meta: {totalCount, filterCount}})
const result = await $ctx.$repos.products.find({
  where: { category: { _eq: 'electronics' } }
});

// Create new record (returns query result with created record)
const result = await $ctx.$repos.products.create({
  name: $ctx.$body.name,
  price: $ctx.$body.price
});

// Update record by ID (returns query result with updated record)
const result = await $ctx.$repos.products.update($ctx.$params.id, {
  price: $ctx.$body.price
});

// Delete record by ID (returns success message)
const result = await $ctx.$repos.products.delete($ctx.$params.id);
```

### Query Filtering
The `where` field uses **Enfyra's API Filtering system** with MongoDB-like operators:

```javascript
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      { price: { _between: [100, 500] } },        // price >= 100 AND <= 500
      { category: { _in: ['electronics', 'gadgets'] } }, // category IN (...)
      { name: { _contains: 'phone' } },           // name CONTAINS 'phone'
      { isActive: { _eq: true } }                 // isActive = true
    ]
  }
});
```

**Available Operators**:
- **Comparison**: `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_between`
- **Arrays**: `_in`, `_not_in` 
- **Text Search**: `_contains`, `_starts_with`, `_ends_with`
- **Null Checks**: `_is_null`
- **Logical**: `_and`, `_or`, `_not`
- **Relations**: Filter by related table data
- **Aggregations**: `_count`, `_sum`, `_avg`, `_min`, `_max`

## Utility Functions

**⚠️ Important: All helper functions require `await`** - They use IPC bridge for execution:

### JWT Token Generation
```javascript
const token = await $ctx.$helpers.$jwt(payload, expiration)
// Example: const token = await $ctx.$helpers.$jwt({userId: 123}, '7d')
```

### Password Hashing and Verification
```javascript
const hash = await $ctx.$helpers.$bcrypt.hash(plainPassword)
const isValid = await $ctx.$helpers.$bcrypt.compare(plainPassword, hashedPassword)
```

### URL-Friendly Slug Generation
```javascript
const slug = await $ctx.$helpers.autoSlug(text)
// Example: const slug = await $ctx.$helpers.autoSlug('My Product Name') → 'my-product-name'
```

## Cache Operations

**⚠️ Important: All cache functions require `await`** - They use IPC bridge for execution:

### Distributed Locking
```javascript
// Acquire distributed lock
const lockAcquired = await $ctx.$cache.acquire(key, value, ttlMs)
// Example: const acquired = await $ctx.$cache.acquire('user-lock', 'user-123', 5000)

// Release distributed lock
const released = await $ctx.$cache.release(key, value)
// Example: const released = await $ctx.$cache.release('user-lock', 'user-123')
```

### Cache Storage
```javascript
// Get cached value
const cachedValue = await $ctx.$cache.get(key)
// Example: const user = await $ctx.$cache.get('user:123')

// Set cached value with TTL
await $ctx.$cache.set(key, value, ttlMs)
// Example: await $ctx.$cache.set('user:123', userData, 60000) // 60 seconds

// Set cached value without expiration
await $ctx.$cache.setNoExpire(key, value)
// Example: await $ctx.$cache.setNoExpire('config:app', configData)
```

### Cache Management
```javascript
// Check if key exists with specific value
const exists = await $ctx.$cache.exists(key, value)
// Example: const exists = await $ctx.$cache.exists('user-lock', 'user-123')

// Delete cached key
await $ctx.$cache.deleteKey(key)
// Example: await $ctx.$cache.deleteKey('user:123')
```

### Cache Use Cases

#### Distributed Locking for Critical Operations
```javascript
// Prevent concurrent modifications to the same record
const lockKey = `record-lock:${$ctx.$params.id}`;
const lockValue = $ctx.$user.id;

const lockAcquired = await $ctx.$cache.acquire(lockKey, lockValue, 10000); // 10 seconds
if (!lockAcquired) {
  throw new Error('Record is currently being modified by another user');
}

try {
  // Perform the critical operation
  $ctx.$logs('Acquired lock for record:', $ctx.$params.id);
  
  // Your business logic here...
  
} finally {
  // Always release the lock
  await $ctx.$cache.release(lockKey, lockValue);
  $ctx.$logs('Released lock for record:', $ctx.$params.id);
}
```

#### Cache-Based Rate Limiting
```javascript
// Implement rate limiting using cache
const rateLimitKey = `rate-limit:${$ctx.$user.id}:${$ctx.$req.url}`;
const currentCount = await $ctx.$cache.get(rateLimitKey) || 0;

if (currentCount >= 10) { // Max 10 requests per minute
  throw new Error('Rate limit exceeded. Please try again later.');
}

// Increment counter with 60 second TTL
await $ctx.$cache.set(rateLimitKey, currentCount + 1, 60000);
$ctx.$logs('Rate limit check passed for user:', $ctx.$user.id);
```

#### Cache-Enhanced Data Retrieval
```javascript
// Check cache first, then fallback to database
const cacheKey = `user-profile:${$ctx.$params.id}`;
let userProfile = await $ctx.$cache.get(cacheKey);

if (!userProfile) {
  // Cache miss - fetch from database
  const result = await $ctx.$repos.users.find({
    where: { id: { _eq: $ctx.$params.id } }
  });
  
  if (result.data.length > 0) {
    userProfile = result.data[0];
    // Cache for 5 minutes
    await $ctx.$cache.set(cacheKey, userProfile, 300000);
    $ctx.$logs('User profile cached:', $ctx.$params.id);
  }
} else {
  $ctx.$logs('User profile served from cache:', $ctx.$params.id);
}

// Use userProfile in your logic...
```

## Logging System

### Basic Logging
```javascript
$ctx.$logs('Debug message', data)
$ctx.$logs('Hook executed:', { hookName: 'validation-hook' })
$ctx.$logs('User created:', user)
$ctx.$logs('Processing order:', { orderId: 123, total: 99.99 })
```

### Log Types and Usage
```javascript
// Information logging
$ctx.$logs('Handler started', { method: $ctx.$req.method, url: $ctx.$req.url })

// Data processing logs
$ctx.$logs('Validating input:', $ctx.$body)
$ctx.$logs('Database query result:', { count: result.data.length })

// Business logic logs
$ctx.$logs('User permission check:', { userId: $ctx.$user.id, hasAccess: true })
$ctx.$logs('External API call:', { endpoint: '/payments', status: 'success' })

// Error context (before throwing)
$ctx.$logs('Validation failed:', { errors: ['Email required', 'Password too short'] })
```

### Automatic Log Response
**Enfyra automatically includes logs in API responses when logs exist** - you don't need to manually return them:

```javascript
// Your handler code
$ctx.$logs('Processing user registration')
$ctx.$logs('Validating email:', $ctx.$body.email)

const user = await $ctx.$repos.users.create({
  email: $ctx.$body.email,
  name: $ctx.$body.name
})

$ctx.$logs('User created successfully:', { id: user.id })

// Just return your data - logs are added automatically
return {
  success: true,
  user: user.data[0]
}
```

**Actual API Response (automatic):**
```json
{
  "success": true,
  "user": {
    "id": 123,
    "email": "user@example.com", 
    "name": "John Doe"
  },
  "logs": [
    "Processing user registration",
    "Validating email:",
    "user@example.com",
    "User created successfully:",
    { "id": 123 }
  ]
}
```

### Shared Context
```javascript
$ctx.$share.$logs  // Array of all logged messages from current request
// Useful for debugging or sharing data between preHook and afterHook

// Store data in shared context
$ctx.$share.validationPassed = true;
$ctx.$share.processStartTime = Date.now();

// Access shared data later
if ($ctx.$share.validationPassed) {
  $ctx.$data.processingTime = Date.now() - $ctx.$share.processStartTime;
}
```

## Package Access

### NPM Packages
```javascript
$ctx.$pkgs        // Access to installed NPM packages
// Use packages installed via Package Management
$ctx.$pkgs.axios     // If axios package is installed
$ctx.$pkgs.lodash    // If lodash package is installed
$ctx.$pkgs.moment    // If moment package is installed
```

**🔗 Learn More**: See [Package Management](../frontend/package-management.md) for installing and using NPM packages in your handlers.

## File Upload Support

### Uploaded File Information
```javascript
$ctx.$uploadedFile  // Information about uploaded file (if any)
// Available properties:
$ctx.$uploadedFile.originalname  // Original filename
$ctx.$uploadedFile.mimetype      // MIME type
$ctx.$uploadedFile.buffer        // File buffer
$ctx.$uploadedFile.size          // File size in bytes
$ctx.$uploadedFile.fieldname     // Form field name
```

## Best Practices

### Code Patterns
- **Always Await**: All `$ctx.$helpers` and `$ctx.$cache` functions require `await` due to IPC bridge
- **Extract Results**: Store helper results in variables before using them
- **Check Data Arrays**: Repository methods return `{data: []}`, always check `data.length`
- **Shared Context**: Use `$ctx.$share` to pass data between preHook and afterHook
- **Cache Key Patterns**: Use consistent naming patterns like `user:123`, `session:456`, `lock:789`
- **Lock Management**: Always release locks in `finally` blocks to prevent deadlocks

### Performance
- **Cache-First Strategy**: Check cache before expensive database operations
- **Appropriate TTL**: Set reasonable cache expiration times based on data volatility
- **Batch Operations**: Group related cache operations when possible
- **Minimal Queries**: Only query what you absolutely need

### Security
- **User Context**: Always validate `$ctx.$user` permissions when needed
- **Input Sanitization**: Clean and validate `$ctx.$body` data
- **Sensitive Data**: Remove sensitive fields for non-privileged users
- **Cache Security**: Don't store sensitive data in cache without encryption
- **Lock Security**: Use user-specific values for distributed locks to prevent conflicts
- **Rate Limiting**: Implement rate limiting using cache to prevent abuse

### Error Handling
- **Throw Errors**: Use `throw new Error('message')` for validation failures
- **Descriptive Messages**: Provide clear error messages for debugging
- **Status Codes**: Errors automatically return appropriate HTTP status codes
- **Graceful Fallbacks**: Handle cases where expected data might be missing
- **Cache Error Handling**: Handle cache failures gracefully with database fallbacks
- **Lock Timeout Handling**: Implement proper timeout handling for distributed locks
- **Retry Logic**: Implement retry logic for transient cache failures

### Debugging
- **Use Logging**: Add `$ctx.$logs()` calls to track execution
- **Check Request Method**: Log `$ctx.$req.method` to see which operations trigger code
- **Inspect Context**: Log `$ctx.$body` and `$ctx.$data` to see transformations
- **Client-Side Debugging**: Logs appear automatically in API responses for easy debugging
- **Cache Debugging**: Log cache operations to track cache hits/misses and TTL
- **Lock Debugging**: Log lock acquisition/release to debug concurrency issues

## Context Sharing Between Hooks

```javascript
// PreHook: Store validation result in shared context
$ctx.$share.validationPassed = true;
$ctx.$share.processStartTime = Date.now();
$ctx.$logs('PreHook: Validation completed');

// AfterHook: Use validation result from shared context
if ($ctx.$share.validationPassed) {
  $ctx.$data.verified = true;
  $ctx.$data.processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$logs('AfterHook: Added verification flag');
}
```

## Related Documentation

- **[Hook Development](./hook-development.md)** - Using context in hooks
- **[Custom Handlers](../frontend/custom-handlers.md)** - Using context in handlers
- **[API Lifecycle](./api-lifecycle.md)** - Complete request processing pipeline
- **[Routing Management](../frontend/routing-management.md)** - Configuring target tables
- **[Package Management](../frontend/package-management.md)** - Installing NPM packages

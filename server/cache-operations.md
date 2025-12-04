# Cache Operations

Enfyra provides distributed caching and locking operations through Redis. Use cache operations for performance optimization, rate limiting, and coordinating between instances.

## Quick Navigation

- [Getting Started](#getting-started) - Basic cache operations
- [Cache Storage](#cache-storage) - Getting and setting cached values
- [Distributed Locking](#distributed-locking) - Coordinating operations across instances
- [Common Patterns](#common-patterns) - Real-world usage examples

## Getting Started

All cache operations are available through `$ctx.$cache` and require `await`.

```javascript
// All cache functions require await
const value = await $ctx.$cache.get('key');
await $ctx.$cache.set('key', value, 60000);
```

## Cache Storage

### Get Cached Value

Retrieve a value from cache.

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

### Set Cached Value with TTL

Store a value in cache with expiration time.

```javascript
await $ctx.$cache.set(key, value, ttlMs);

// Example - cache for 5 minutes (300000 milliseconds)
await $ctx.$cache.set('user:123', userData, 300000);

// Example - cache for 1 hour (3600000 milliseconds)
await $ctx.$cache.set('product:456', productData, 3600000);
```

**TTL (Time-To-Live) is in milliseconds:**
- 1000 = 1 second
- 60000 = 1 minute
- 300000 = 5 minutes
- 3600000 = 1 hour
- 86400000 = 1 day

### Set Cached Value Without Expiration

Store a value in cache that never expires.

```javascript
await $ctx.$cache.setNoExpire(key, value);

// Example
await $ctx.$cache.setNoExpire('config:app', configData);
```

### Delete Cache Key

Remove a value from cache.

```javascript
await $ctx.$cache.deleteKey(key);

// Example
await $ctx.$cache.deleteKey('user:123');
```

### Check if Key Exists

Check if a cache key exists with a specific value.

```javascript
const exists = await $ctx.$cache.exists(key, value);

// Example
const lockExists = await $ctx.$cache.exists('user-lock:123', 'user-456');
```

## Distributed Locking

Distributed locking prevents concurrent operations across multiple instances.

### Acquire Lock

Try to acquire a lock. Returns `true` if successful, `false` if lock is already held.

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

### Release Lock

Release a lock that you acquired.

```javascript
const released = await $ctx.$cache.release(key, value);

// Example
const lockKey = `user-lock:${userId}`;
const lockValue = $ctx.$user.id;
const released = await $ctx.$cache.release(lockKey, lockValue);
```

**Important:** Only the instance that acquired the lock can release it (verified by value).

### Lock Pattern with Try-Finally

Always release locks in a `finally` block to ensure cleanup even if errors occur.

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

## Common Patterns

### Pattern 1: Cache-First Data Retrieval

Check cache first, fallback to database if cache miss.

```javascript
const cacheKey = `user-profile:${$ctx.$params.id}`;
let userProfile = await $ctx.$cache.get(cacheKey);

if (!userProfile) {
  // Cache miss - fetch from database
  const result = await $ctx.$repos.user_definition.find({
    where: { id: { _eq: $ctx.$params.id } }
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

### Pattern 2: Invalidate Cache on Update

Clear cache when data is updated.

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

### Pattern 3: Rate Limiting

Use cache to implement rate limiting.

```javascript
// Rate limit: max 10 requests per minute per user
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

### Pattern 4: Prevent Concurrent Modifications

Use distributed locking to prevent concurrent modifications.

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
    where: { id: { _eq: $ctx.$params.id } }
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

### Pattern 5: Cache Configuration Data

Cache configuration data that doesn't change frequently.

```javascript
const configKey = 'app:configuration';
let config = await $ctx.$cache.get(configKey);

if (!config) {
  // Load from database
  const result = await $ctx.$repos.configurations.find({
    where: { isActive: { _eq: true } }
  });
  
  if (result.data.length > 0) {
    config = result.data[0];
    // Cache for 1 hour
    await $ctx.$cache.set(configKey, config, 3600000);
  }
}

// Use config...
```

### Pattern 6: Session Management

Store session data in cache with expiration.

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

### Pattern 7: Cache Warming

Pre-populate cache with frequently accessed data.

```javascript
// In a background process or bootstrap script
const popularProducts = await $ctx.$repos.products.find({
  where: { isPopular: { _eq: true } },
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

## Best Practices

1. **Always use await** - All cache functions are async and require await
2. **Set appropriate TTL** - Choose TTL based on how often data changes
3. **Use consistent key patterns** - Use patterns like `user:123`, `session:456`, `lock:789`
4. **Always release locks** - Use try-finally blocks to ensure locks are released
5. **Invalidate cache on updates** - Delete cache keys when data is updated
6. **Handle cache misses** - Always have a fallback to database when cache misses
7. **Use locking for critical operations** - Prevent concurrent modifications with distributed locks

## Key Naming Conventions

Use consistent naming patterns for cache keys:

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

## Next Steps

- Learn about [Context Reference](./context-reference/) for cache access
- See [Cluster Architecture](./cluster-architecture.md) to understand distributed coordination
- Check [Error Handling](./error-handling.md) for handling cache failures


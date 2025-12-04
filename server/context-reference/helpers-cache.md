# Context Reference - Helpers & Cache

Utility functions for common tasks and distributed caching operations.

## Helpers

Utility functions for common tasks. All helper functions require `await`.

### JWT Token Generation

```javascript
const token = await $ctx.$helpers.$jwt(payload, expiration);

// Example
const token = await $ctx.$helpers.$jwt(
  { userId: 123, email: 'user@example.com' },
  '7d'  // Expires in 7 days
);
```

**Expiration formats:**
- `'15m'` - 15 minutes
- `'1h'` - 1 hour
- `'1d'` - 1 day
- `'7d'` - 7 days
- `'30d'` - 30 days

### Password Hashing

```javascript
// Hash password
const hash = await $ctx.$helpers.$bcrypt.hash(plainPassword);

// Verify password
const isValid = await $ctx.$helpers.$bcrypt.compare(plainPassword, hashedPassword);

// Example
const hashedPassword = await $ctx.$helpers.$bcrypt.hash('myPassword123');
const isValid = await $ctx.$helpers.$bcrypt.compare('myPassword123', hashedPassword);
```

### Auto Slug Generation

Generate URL-friendly slugs from text.

```javascript
const slug = await $ctx.$helpers.autoSlug(text);

// Example
const slug = await $ctx.$helpers.autoSlug('My Product Name');
// Result: 'my-product-name'
```

### File Upload Helper

Upload files to storage.

```javascript
const fileResult = await $ctx.$helpers.$uploadFile({
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

### File Update Helper

Update existing files.

```javascript
await $ctx.$helpers.$updateFile(fileId, {
  buffer: newFileBuffer,
  originalname: 'new-name.jpg',
  mimetype: 'image/jpeg',
  folder: 456,
  title: 'Updated Title'
});
```

### File Delete Helper

Delete files.

```javascript
await $ctx.$helpers.$deleteFile(fileId);
```

## Cache

Distributed caching and locking operations. All cache functions require `await`.

### Get Cached Value

```javascript
const cachedValue = await $ctx.$cache.get(key);

// Example
const user = await $ctx.$cache.get('user:123');
```

### Set Cached Value

```javascript
// Set with TTL (time-to-live in milliseconds)
await $ctx.$cache.set(key, value, ttlMs);

// Example - cache for 5 minutes
await $ctx.$cache.set('user:123', userData, 300000);

// Set without expiration
await $ctx.$cache.setNoExpire(key, value);
```

### Distributed Locking

Acquire and release locks for critical operations.

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

### Check Cache Exists

```javascript
const exists = await $ctx.$cache.exists(key, value);

// Example
const lockExists = await $ctx.$cache.exists('user-lock:123', 'user-456');
```

### Delete Cache Key

```javascript
await $ctx.$cache.deleteKey(key);

// Example
await $ctx.$cache.deleteKey('user:123');
```

## Next Steps

- See [Logging & Error Handling](./logging-errors.md) for logging and error throwing
- Check [Advanced Features](./advanced.md) for file uploads and more
- Learn about [Cache Operations](./cache-operations.md) for detailed cache patterns


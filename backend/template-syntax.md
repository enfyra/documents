# Template Syntax Guide

Enfyra provides **three equivalent ways** to access context properties. You can use the full `$ctx.$property` syntax, template syntax, or direct table syntax - all work exactly the same way.

## Overview

**All three syntaxes are fully supported and equivalent:**

```javascript
// ✅ Full syntax (always works)
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.user_definition.find({...});
const slug = $ctx.$helpers.autoSlug('Hello World');
$ctx.$logs('Operation completed');

// ✅ Template syntax (convenience shortcut)
const data = await @CACHE.get('key');
const users = await @REPOS.user_definition.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;

// ✅ Direct table syntax (shortest for database)
const data = await @CACHE.get('key');
const users = await #user_definition.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;
```

**💡 Template syntax is just syntactic sugar** - it gets automatically replaced with the full `$ctx.$property` syntax during execution. **Use whichever style you prefer - you can even mix all three in the same file!**

## Available Templates

| Template | Replacement | Description |
|----------|-------------|-------------|
| `@CACHE` | `$ctx.$cache` | Cache operations and distributed locking |
| `@REPOS` | `$ctx.$repos` | Database repository access |
| `@HELPERS` | `$ctx.$helpers` | Utility functions and helpers |
| `@LOGS` | `$ctx.$logs` | Logging functions |
| `@BODY` | `$ctx.$body` | Request body data |
| `@DATA` | `$ctx.$data` | Response data object |
| `@STATUS` | `$ctx.$statusCode` | HTTP status code |
| `@PARAMS` | `$ctx.$params` | Route parameters |
| `@QUERY` | `$ctx.$query` | Query parameters |
| `@USER` | `$ctx.$user` | Current user information |
| `@REQ` | `$ctx.$req` | Express request object |
| `@SHARE` | `$ctx.$share` | Shared data between hooks |
| `@API` | `$ctx.$api` | API request/response information |
| `@UPLOADED` | `$ctx.$uploadedFile` | Uploaded file information |
| `@THROW` | `$ctx.$throw` | Error throwing functions |
| `@THROW400` | `$ctx.$throw['400']` | HTTP 400 Bad Request (shortcut) |
| `@THROW401` | `$ctx.$throw['401']` | HTTP 401 Unauthorized (shortcut) |
| `@THROW403` | `$ctx.$throw['403']` | HTTP 403 Forbidden (shortcut) |
| `@THROW404` | `$ctx.$throw['404']` | HTTP 404 Not Found (shortcut) |
| `@THROW409` | `$ctx.$throw['409']` | HTTP 409 Conflict (shortcut) |
| `@THROW422` | `$ctx.$throw['422']` | HTTP 422 Validation Error (shortcut) |
| `@THROW500` | `$ctx.$throw['500']` | HTTP 500 Internal Error (shortcut) |
| `@THROW503` | `$ctx.$throw['503']` | HTTP 503 Service Unavailable (shortcut) |
| `#table_name` | `$ctx.$repos.table_name` | Direct table access (e.g., `#user_definition`, `#product_definition`) |

## Usage Examples

### Cache Operations

```javascript
// Get data from cache
const cachedData = await @CACHE.get('user:123');

// Set data in cache with TTL
await @CACHE.set('user:123', userData, 3600000); // 1 hour

// Check if key exists
const exists = await @CACHE.exists('user:123');

// Delete from cache
await @CACHE.delete('user:123');

// Distributed locking
const lockAcquired = await @CACHE.acquire('critical-operation', 'instance-1', 30000);
if (lockAcquired) {
  try {
    // Critical operation here
    await performCriticalOperation();
  } finally {
    await @CACHE.release('critical-operation', 'instance-1');
  }
}
```

### Database Operations

#### Using @REPOS syntax:
```javascript
// Find records with filtering
const users = await @REPOS.user_definition.find({
  where: { isActive: true },
  fields: 'id,email,name', // Only fetch required fields
  limit: 10,
  page: 1
});

// Create new record
const newUser = await @REPOS.users.create({
  email: 'user@example.com',
  name: 'John Doe',
  isActive: true
});

// Update record
const updatedUser = await @REPOS.users.update(
  { id: 123 },
  { name: 'Jane Doe', lastLogin: new Date() }
);

// Delete record
await @REPOS.users.delete({ id: 123 });

// Count records
const totalUsers = await @REPOS.users.count({ isActive: true });
```

#### Using #table_name syntax (shorter):
```javascript
// Find records with filtering
const users = await #user_definition.find({
  where: { isActive: true },
  fields: 'id,email,name', // Only fetch required fields
  limit: 10,
  page: 1
});

// Create new record
const newUser = await #user_definition.create({
  email: 'user@example.com',
  name: 'John Doe',
  isActive: true
});

// Update record
const updatedUser = await #user_definition.update(
  { id: 123 },
  { name: 'Jane Doe', lastLogin: new Date() }
);

// Delete record
await #user_definition.delete({ id: 123 });

// Count records
const totalUsers = await #user_definition.count({ isActive: true });
```

### Helper Functions

```javascript
// Generate JWT token
const token = @HELPERS.$jwt.sign({ userId: 123, role: 'admin' }, '1h');

// Hash password
const hashedPassword = await @HELPERS.$bcrypt.hash('password123', 10);

// Verify password
const isValid = await @HELPERS.$bcrypt.compare('password123', hashedPassword);

// Generate URL-friendly slug
const slug = @HELPERS.autoSlug('Hello World!'); // "hello-world"
```

### Logging

```javascript
// Basic logging
@LOGS('User operation started');

// Log with data
@LOGS('User created:', { id: 123, email: 'user@example.com' });

// Multiple parameters
@LOGS('Cache operation', 'key:', 'user:123', 'result:', cachedData);

// Error logging
@LOGS('Error occurred:', error.message, error.stack);
```

### Error Handling

**HTTP Status Code Errors (`@THROW`):**

**Option 1: With brackets and quotes**
```javascript
// Throw HTTP 400 Bad Request
@THROW['400']('Email is required');

// Throw HTTP 401 Unauthorized
@THROW['401']('Invalid credentials');

// Throw HTTP 403 Forbidden
@THROW['403']('Insufficient permissions');

// Throw HTTP 404 Not Found
@THROW['404']('User not found', 'user_id_123');

// Throw HTTP 409 Conflict (for duplicates)
@THROW['409']('Email already exists', 'email', 'user@example.com');

// Throw HTTP 422 Validation Error
@THROW['422']('Invalid data format');

// Throw HTTP 500 Internal Server Error
@THROW['500']('Database connection failed');
```

**Option 2: Direct shortcuts (no quotes needed)**
```javascript
// Throw HTTP 400 Bad Request
@THROW400('Email is required');

// Throw HTTP 401 Unauthorized  
@THROW401('Invalid credentials');

// Throw HTTP 403 Forbidden
@THROW403('Insufficient permissions');

// Throw HTTP 404 Not Found
@THROW404('User not found', 'user_id_123');

// Throw HTTP 409 Conflict (for duplicates)
@THROW409('Email already exists');

// Throw HTTP 422 Validation Error
@THROW422('Invalid data format');

// Throw HTTP 500 Internal Server Error
@THROW500('Database connection failed');

// Throw HTTP 503 Service Unavailable
@THROW503('Service unavailable');
```


## Advanced Usage

### Chaining Operations

```javascript
// Cache with database fallback
let data = await @CACHE.get('products:featured');
if (!data) {
  data = await #products.find({
    where: { featured: true },
    fields: 'id,name,price,image'
  });
  await @CACHE.set('products:featured', data, 3600000);
}
@LOGS('Featured products loaded:', data.length);
```

### Error Handling with Logging

```javascript
try {
  const user = await #user_definition.find({ where: { id: userId } });
  if (!user.data.length) {
    @THROW404('User not found');
  }
  
  const updatedUser = await #user_definition.update(
    { id: userId },
    { lastLogin: new Date() }
  );
  
  @LOGS('User login updated:', updatedUser.id);
  return updatedUser;
  
} catch (error) {
  @LOGS('User update failed:', error.message);
  throw error;
}
```

### Complex Business Logic

```javascript
// User registration with validation and caching
async function registerUser(userData) {
  // Check if user exists
  const existingUser = await #user_definition.find({
    where: { email: userData.email },
    fields: 'id'
  });
  
  if (existingUser.data.length > 0) {
    @THROW409('Email already exists');
  }
  
  // Hash password
  const hashedPassword = await @HELPERS.$bcrypt.hash(userData.password, 10);
  
  // Create user
  const newUser = await #user_definition.create({
    ...userData,
    password: hashedPassword,
    createdAt: new Date()
  });
  
  // Cache user data
  await @CACHE.set(`user:${newUser.id}`, newUser, 1800000); // 30 minutes
  
  // Generate JWT token
  const token = @HELPERS.$jwt.sign({ userId: newUser.id }, '24h');
  
  @LOGS('User registered successfully:', newUser.id);
  
  return { user: newUser, token };
}
```

## How Template Replacement Works

Template syntax is **purely a convenience feature**. Here's what happens behind the scenes:

1. **Code Submission**: Your code with `@CACHE`, `@REPOS`, etc. is submitted
2. **Template Processing**: Templates are automatically replaced with full `$ctx.$property` syntax
3. **Code Execution**: The processed code runs normally in the handler executor
4. **Result Return**: Normal execution continues with the replaced syntax

### Example Transformation

```javascript
// Your code (template syntax):
const data = await @CACHE.get('key');
const users = await #user_definition.find({...});

// What actually runs (after replacement):
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.users.find({...});
```

**🔧 The replacement is transparent** - you can mix all three syntaxes in the same file if you want!

## Best Practices

### 1. Choose Your Style (All Work!)

```javascript
// ✅ Option 1 - Full syntax throughout
const user = await $ctx.$repos.users.find({ where: { id: userId } });
await $ctx.$cache.set(`user:${userId}`, user);
$ctx.$logs('User cached:', userId);

// ✅ Option 2 - Template syntax throughout  
const user = await @REPOS.users.find({ where: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

// ✅ Option 3 - Direct table syntax (shortest for database)
const user = await #user_definition.find({ where: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

// ✅ Option 4 - Mix all three (totally fine!)
const user = await #user_definition.find({ where: { id: userId } });        // Direct table
await $ctx.$cache.set(`user:${userId}`, user);                    // Full syntax
@LOGS('User cached:', userId);                                     // Template syntax

// ✅ Option 5 - Any combination works!
const user = await $ctx.$repos.user_definition.find({ where: { id: userId } }); // Full
await @CACHE.set(`user:${userId}`, user);                             // Template
const products = await #product_definition.find({ where: { userId } }); // Direct
$ctx.$logs('All operations completed');                               // Full
```

### 2. Leverage Field Selection

```javascript
// ✅ Good - only fetch needed fields
const users = await #user_definition.find({
  where: { isActive: true },
  fields: 'id,email,name' // Performance optimization
});

// ❌ Avoid fetching all fields unnecessarily
const users = await #user_definition.find({
  where: { isActive: true }
  // Fetches all fields by default
});
```

### 3. Proper Error Handling

```javascript
// ✅ Good - comprehensive error handling
try {
  const result = await @REPOS.users.create(userData);
  @LOGS('User created successfully');
  return result;
} catch (error) {
  @LOGS('User creation failed:', error.message);
  @THROW500('Failed to create user');
}
```

### 4. Cache Strategy

```javascript
// ✅ Good - cache with appropriate TTL
const cacheKey = `user:${userId}`;
let user = await @CACHE.get(cacheKey);

if (!user) {
  user = await @REPOS.users.find({ where: { id: userId } });
  await @CACHE.set(cacheKey, user, 1800000); // 30 minutes
}
```

## Compatibility & Flexibility

- **✅ All Three Syntaxes Supported**: Use `$ctx.$property`, `@TEMPLATE`, OR `#table_name` - your choice!
- **✅ Mix and Match**: You can use any combination in the same file
- **✅ All Contexts**: Works in Bootstrap Scripts, Hooks, and Custom Handlers
- **✅ IDE Support**: `$ctx.$property` has better autocomplete support
- **✅ Debugging**: Stack traces show the processed `$ctx.$property` syntax
- **✅ No Performance Impact**: Template replacement is just string processing
- **✅ Maximum Flexibility**: Choose the syntax that fits your style best

## Related Documentation

- [Context Reference](./context-reference.md) - Complete `$ctx` object documentation
- [Bootstrap Scripts](./bootstrap-scripts.md) - Using templates in bootstrap scripts
- [Hook Development](./hook-development.md) - Using templates in hooks
- [Custom Handlers](../frontend/custom-handlers.md) - Using templates in handlers

# Template Syntax Guide

Enfyra provides **three equivalent ways** to access context properties. You can use the full `$ctx.$property` syntax, template syntax, or direct table syntax - all work exactly the same way.

## Overview

**All three syntaxes are fully supported and equivalent:**

```javascript
//  Full syntax (always works)
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.enfyra_user.find({...});
const slug = $ctx.$helpers.autoSlug('Hello World');
$ctx.$logs('Operation completed');

//  Template syntax (convenience shortcut)
const data = await @CACHE.get('key');
const users = await @REPOS.enfyra_user.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;

//  Direct table syntax (shortest for database)
const data = await @CACHE.get('key');
const users = await #enfyra_user.find({...});
const slug = @HELPERS.autoSlug('Hello World');
@LOGS('Operation completed');
const userId = @USER.id;
const bodyData = @BODY.name;
```

**Template syntax is just syntactic sugar** - Enfyra Server replaces it with the full `$ctx.$property` syntax before compiled code is passed to the kernel executor. **Use whichever style you prefer - you can even mix all three in the same file!**

In **`find()`** options, pass predicates as **`filter`**. REST list endpoints also use the query parameter name **`filter`**.

## Available Templates

| Template | Replacement | Description |
|----------|-------------|-------------|
| `@CACHE` | `$ctx.$cache` | Cache operations and distributed locking |
| `@REPOS` | `$ctx.$repos` | Database repository access |
| `@HELPERS` | `$ctx.$helpers` | Utility functions and helpers |
| `@FETCH` | `$ctx.$helpers.$fetch` | SSRF-hardened HTTP client (shorthand for `@HELPERS.$fetch`) |
| `@LOGS` | `$ctx.$logs` | Logging functions |
| `@BODY` | `$ctx.$body` | Request body data |
| `@ENV` | `$ctx.$env` | Sanitized environment variables exposed by the host runtime |
| `@DATA` | `$ctx.$data` | Response data object |
| `@STATUS` | `$ctx.$statusCode` | HTTP status code (200 on success, error code on failure — available in postHooks) |
| `@ERROR` | `$ctx.$error` | Error context in postHooks (`{ message, name, statusCode, details, timestamp }` — `undefined` on success) |
| `@PARAMS` | `$ctx.$params` | Route parameters |
| `@QUERY` | `$ctx.$query` | Query parameters |
| `@USER` | `$ctx.$user` | Current user information |
| `@REQ` | `$ctx.$req` | Express request object |
| `@RES` | `$ctx.$res` | Express response object (handlers only) |
| `@SHARE` | `$ctx.$share` | Shared data between hooks |
| `@API` | `$ctx.$api` | API request/response information |
| `@SOCKET` | `$ctx.$socket` | WebSocket operations (join, leave, reply, emitToUser, emitToRoom, emitToCurrentRoom, broadcastToRoom, emitToGateway, broadcast; `disconnect` only in connection handlers) |
| `@TRIGGER` | `$ctx.$trigger` | Trigger a flow by id or name (`@TRIGGER(flowIdOrName, payload?)`) |
| `@TRANSACTION` | `$ctx.$transaction` | Atomic repository mutation scope; use `await @TRANSACTION.run(async () => { ... })` |
| `@FLOW` | `$ctx.$flow` | Current flow context inside flow steps (payload, last step output, meta) |
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` | Original payload passed into the flow |
| `@FLOW_LAST` | `$ctx.$flow.$last` | Output of the previous flow step |
| `@FLOW_META` | `$ctx.$flow.$meta` | Flow execution metadata (id, name, runId, etc.) |
| `@UPLOADED_FILE` | `$ctx.$uploadedFile` | Uploaded file information |
| `@PKGS` | `$ctx.$pkgs` | Installed npm packages for use in handlers |
| `@THROW.http(statusCode, message?)` | `$ctx.$throw.http(statusCode, message?)` | Quick generic HTTP error with a dynamic status |
| `@THROW400(message)` | `$ctx.$throw.http(400, message)` | Quick HTTP 400 Bad Request; message is required |
| `@THROW401(message)` | `$ctx.$throw.http(401, message)` | Quick HTTP 401 Unauthorized; message is required |
| `@THROW403(message)` | `$ctx.$throw.http(403, message)` | Quick HTTP 403 Forbidden; message is required |
| `@THROW404(message)` | `$ctx.$throw.http(404, message)` | Quick HTTP 404 Not Found; message is required |
| `@THROW409(message)` | `$ctx.$throw.http(409, message)` | Quick HTTP 409 Conflict; message is required |
| `@THROW422(message)` | `$ctx.$throw.http(422, message)` | Quick HTTP 422 Validation Error; message is required |
| `@THROW429(message)` | `$ctx.$throw.http(429, message)` | Quick HTTP 429 Rate Limit Exceeded; message is required |
| `@THROW500(message)` | `$ctx.$throw.http(500, message)` | Quick HTTP 500 Internal Error; message is required |
| `@THROW503(message)` | `$ctx.$throw.http(503, message)` | Quick HTTP 503 Service Unavailable; message is required |
| `#table_name` | `$ctx.$repos.table_name` | Direct table access (e.g., `#enfyra_user`, `#product`) |
| `%pkg_name` | `$ctx.$pkgs.pkg_name` | Shorthand package access (e.g., `%axios`, `%lodash`, `%moment`) |

## Usage Examples

### Cache Operations

`@CACHE` maps to the same managed user cache as `$ctx.$cache`. Use logical keys only; Enfyra applies the current app namespace internally. Do not include `NODE_NAME`, `user_cache:`, or Redis prefixes in template code. User-cache data is limited by `REDIS_USER_CACHE_LIMIT_MB` (default `30` MB), and Enfyra evicts least-recently-used user-cache keys when the allocation is exceeded.

```javascript
// Get data from cache
const cachedData = await @CACHE.get('user:123');

// Set data in cache with TTL
await @CACHE.set('user:123', userData, 3600000); // 1 hour

// Check if key exists
const exists = await @CACHE.exists('user:123');

// Delete from cache
await @CACHE.deleteKey('user:123');

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
// Find records with filtering and pagination
const users = await @REPOS.enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name',   // Only fetch required fields
  limit: 10,                  // Max 10 records (default: 10)
  sort: '-createdAt'          // Sort by createdAt DESC
});

// Fetch ALL records (no limit)
const allUsers = await @REPOS.enfyra_user.find({
  filter: { isActive: true },
  limit: 0  // 0 = fetch all
});

// Multi-field sorting
const sorted = await @REPOS.enfyra_user.find({
  sort: 'name,-createdAt'  // Sort by name ASC, then createdAt DESC
});

// Nested relations (get related data in ONE query)
const usersWithPosts = await @REPOS.enfyra_user.find({
  fields: 'id,email,posts.title,posts.createdAt',  // Nested field: posts.title
  filter: { isActive: true }
});

// Filter by nested relation
const usersInRole = await @REPOS.enfyra_user.find({
  filter: {
    role: {
      name: { _eq: 'Admin' }  // Filter by related role name
    }
  },
  fields: 'id,email,role.name'
});

// Create new record
const newUser = await @REPOS.enfyra_user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    isActive: true
  }
});

// Update record by ID
const updatedUser = await @REPOS.enfyra_user.update({
  id: userId,
  data: {
    name: 'Jane Doe',
    lastLogin: new Date()
  }
});

// Delete record by ID
await @REPOS.enfyra_user.delete({ id: userId });
```

#### Batch mutations

Use `createMany`, `updateMany`, or `deleteMany` only inside a dynamic handler, hook, or flow when the same operation must affect multiple records. These are repository methods, not public REST batch endpoints.

Batch mutations are available only for plain generic tables. They reject metadata/schema routes and tables with custom normalization or lifecycle behavior. Enfyra validates and authorizes every input record before the write, then performs one bulk write followed by one runtime reload and one mutation event. Use a secure repository (`@REPOS.main`, `@REPOS.secure.<table>`, or `#secure.<table>`) for user-facing code; ownership, tenant, and membership checks remain your responsibility.

`createMany` accepts an array of record bodies and returns `data` plus `count`:

```javascript
const records = @BODY.records;
if (!Array.isArray(records) || records.length === 0) {
  @THROW400('records must be a non-empty array');
}

const result = await #secure.orders.createMany({
  data: records,
  fields: ['id', 'status']
});

return { data: result.data, count: result.count };
```

`updateMany` applies one `data` object to every supplied ID. Relation payloads are intentionally rejected, so use individual updates when the change must connect, disconnect, or cascade relations.

```javascript
const result = await #secure.orders.updateMany({
  ids: @BODY.ids,
  data: { status: 'archived' },
  fields: ['id', 'status']
});

return { data: result.data, count: result.count };
```

`deleteMany` accepts IDs and returns the affected `count`:

```javascript
const result = await #secure.orders.deleteMany({
  ids: @BODY.ids
});

return { deleted: result.count };
```

#### Using #table_name syntax (shorter):
```javascript
// Find records with filtering and pagination
const users = await #enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name',   // Only fetch required fields
  limit: 10,                  // Max 10 records (default: 10)
  sort: '-createdAt'          // Sort by createdAt DESC
});

// Fetch ALL records (no limit)
const allUsers = await #enfyra_user.find({
  filter: { isActive: true },
  limit: 0  // 0 = fetch all
});

// Multi-field sorting
const sorted = await #enfyra_user.find({
  sort: 'name,-createdAt'  // Sort by name ASC, then createdAt DESC
});

// Nested relations (get related data in ONE query)
const usersWithPosts = await #enfyra_user.find({
  fields: 'id,email,posts.title,posts.createdAt',  // Nested field: posts.title
  filter: { isActive: true }
});

// Filter by nested relation
const usersInRole = await #enfyra_user.find({
  filter: {
    role: {
      name: { _eq: 'Admin' }  // Filter by related role name
    }
  },
  fields: 'id,email,role.name'
});

// Create new record
const newUser = await #enfyra_user.create({ data: {
  email: 'user@example.com',
  name: 'John Doe',
  isActive: true
}});

// Update record by ID
const updatedUser = await #enfyra_user.update({ id: userId, data: {
  name: 'Jane Doe',
  lastLogin: new Date()
}});

// Delete record by ID
await #enfyra_user.delete({ id: userId });
```

### Helper Functions

```javascript
// Generate JWT token (call $jwt as a function: payload, expiresIn)
const token = await @HELPERS.$jwt({ userId: 123, role: 'admin' }, '1h');

// Hash password (single argument; salt rounds are managed internally)
const hashedPassword = await @HELPERS.$bcrypt.hash('password123');

// Verify password
const isValid = await @HELPERS.$bcrypt.compare('password123', hashedPassword);

// Generate URL-friendly slug
const slug = @HELPERS.autoSlug('Hello World!'); // "hello-world"
```

### Package Usage

**Traditional Syntax:**
```javascript
// Access installed npm packages
const axios = $ctx.$pkgs.axios;
const lodash = $ctx.$pkgs.lodash;
const moment = $ctx.$pkgs.moment;

// Use package normally
const response = await axios.get('https://api.example.com/data');

// Transform data with lodash
const grouped = lodash.groupBy(response.data, 'category');

// Format dates with moment
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Template Syntax (Shortened):**
```javascript
// Access installed npm packages
const axios = @PKGS.axios;
const lodash = @PKGS.lodash;
const moment = @PKGS.moment;

// Use package normally
const response = await axios.get('https://api.example.com/data');

// Transform data with lodash
const grouped = lodash.groupBy(response.data, 'category');

// Format dates with moment
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Shorthand Syntax (`%`):**
```javascript
// Direct package access - shortest possible
const axios = %axios;
const lodash = %lodash;
const moment = %moment;

const response = await axios.get('https://api.example.com/data');
const summary = lodash.groupBy(response.data, 'category');
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
```

**Mixed Syntax:**
```javascript
// You can mix all three ways!
const axios = %axios;                         // Shorthand syntax
const lodash = @PKGS.lodash;                 // Template syntax  
const moment = $ctx.$pkgs.moment;             // Traditional syntax

const response = await axios.get('https://api.example.com/data');
const summary = lodash.groupBy(response.data, 'category');
const timestamp = moment().format('YYYY-MM-DD HH:mm:ss');
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

### File Upload & Streaming

```javascript
// Access uploaded file
const file = @UPLOADED_FILE;
@LOGS('File uploaded:', file.originalname, file.mimetype, file.size);

// Save uploaded request file to storage and enfyra_file.
// This streams from the server temp file and does not buffer the full file.
const savedFile = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  description: @BODY.description
});

// Stream an upstream response (for large files or SSE)
const upstream = await @PKGS.undici.request(@QUERY.imageUrl, { method: 'GET' });

await @RES.stream(upstream.body, {
  mimetype: upstream.headers['content-type'] || 'application/octet-stream'
});
```

`@RES.stream` expects a readable from an installed server package. Native `fetch`, `Readable`, and `AbortController` are not portable template APIs. `@HELPERS.$fetch` buffers the response and should not be used for SSE or chat streaming.

** See [File Handling](./file-handling.md)** for complete guide on file uploads, streaming, and image processing.

### Error Handling

Use `@THROW.http(statusCode, message?)` for a quick generic Enfyra HTTP error with a dynamic status. The fixed-status helpers remain available as readable source syntax; each requires exactly one message and compiles to the same `.http` method. The generic envelope always includes trace fields under `error` and omits `error.details` when no details exist.

```javascript
@THROW.http(502, 'The upstream service failed');
@THROW400('Email is required');
@THROW401('Invalid credentials');
@THROW403('Insufficient permissions');
@THROW404('User not found');
@THROW409('Email already exists');
@THROW422('Invalid data format');
@THROW429('Too many requests');
@THROW500('Database connection failed');
@THROW503('Service unavailable');
```

Do not call numeric properties such as `$ctx.$throw['400']`, and do not pass details or semantic arguments to a fixed-status helper. For custom JSON error fields, status, and headers, use `@THROW.json`. The body and optional `body.error` must be objects. ESV preserves non-reserved custom fields, writes root `success: false` and root `statusCode` equal to the HTTP response status, removes `error.statusCode`, and merges server-owned `timestamp`, pathname-only `path`, `method`, and `correlationId` into `error`. Do not declare `body.success`, `body.statusCode`, or `body.error.statusCode`; select the status only through `options.statusCode`. `X-Correlation-ID` carries the same identifier as `error.correlationId`. The call terminates the handler, defaults to status `500`, and accepts only `400`–`599`:

```javascript
@THROW.json(
  {
    error: {
      type: 'api_error',
      code: 'upstream_error',
      message: 'Please retry shortly.',
      should_retry: true,
      retry_after_seconds: 5
    }
  },
  {
    statusCode: 502,
    headers: {
      'x-should-retry': 'true',
      'Retry-After': '5'
    }
  }
);
```

The client receives the custom fields plus the server trace:

```json
{
  "success": false,
  "statusCode": 502,
  "error": {
    "type": "api_error",
    "code": "upstream_error",
    "message": "Please retry shortly.",
    "should_retry": true,
    "retry_after_seconds": 5,
    "timestamp": "<server ISO timestamp>",
    "path": "<request path>",
    "method": "POST",
    "correlationId": "<server correlation ID>"
  }
}
```

`@RES.json` is the separate success-only boundary. It accepts statuses `200`–`399` and should be returned as the terminal handler statement:

```javascript
return await @RES.json(
  { data: { id: 'project-1' }, success: true },
  { statusCode: 201, headers: { 'x-resource-created': 'true' } }
);
```

## Advanced Usage

### Chaining Operations

```javascript
// Cache with database fallback
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

### Error Handling with Logging

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

### Complex Business Logic

```javascript
// User registration with validation and caching
async function registerUser(userData) {
  // Check if user exists
  const existingUser = await #enfyra_user.find({
    filter: { email: userData.email },
    fields: 'id'
  });
  
  if (existingUser.data.length > 0) {
    @THROW409('Email already exists');
  }
  
  // Hash password
  const hashedPassword = await @HELPERS.$bcrypt.hash(userData.password);
  
  // Create user
  const newUser = await #enfyra_user.create({ data: {
    ...userData,
    password: hashedPassword,
    createdAt: new Date()
  });
  
  // Cache user data
  await @CACHE.set(`user:${newUser.id}`, newUser, 1800000); // 30 minutes
  
  // Generate JWT token
  const token = await @HELPERS.$jwt({ userId: newUser.id }, '24h');
  
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
const users = await #enfyra_user.find({...});

// What actually runs (after replacement):
const data = await $ctx.$cache.get('key');
const users = await $ctx.$repos.enfyra_user.find({...});
```

** The replacement is transparent** - you can mix all three syntaxes in the same file if you want!

## Best Practices

### 1. Choose Your Style (All Work!)

```javascript
//  Option 1 - Full syntax throughout
const user = await $ctx.$repos.users.find({ filter: { id: userId } });
await $ctx.$cache.set(`user:${userId}`, user);
$ctx.$logs('User cached:', userId);

//  Option 2 - Template syntax throughout  
const user = await @REPOS.users.find({ filter: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

//  Option 3 - Direct table syntax (shortest for database)
const user = await #enfyra_user.find({ filter: { id: userId } });
await @CACHE.set(`user:${userId}`, user);
@LOGS('User cached:', userId);

//  Option 4 - Mix all three (totally fine!)
const user = await #enfyra_user.find({ filter: { id: userId } });        // Direct table
await $ctx.$cache.set(`user:${userId}`, user);                    // Full syntax
@LOGS('User cached:', userId);                                     // Template syntax

//  Option 5 - Any combination works!
const user = await $ctx.$repos.enfyra_user.find({ filter: { id: userId } }); // Full
await @CACHE.set(`user:${userId}`, user);                             // Template
const products = await #product.find({ filter: { userId } }); // Direct
$ctx.$logs('All operations completed');                               // Full
```

### 2. Leverage Field Selection

```javascript
//  Good - only fetch needed fields
const users = await #enfyra_user.find({
  filter: { isActive: true },
  fields: 'id,email,name' // Performance optimization
});

//  Good - fetch editable script source without generated compiled output
const handlers = await #enfyra_route_handler.find({
  fields: '-compiledCode'
});

//  Avoid fetching all fields unnecessarily
const users = await #enfyra_user.find({
  filter: { isActive: true }
  // Fetches all fields by default
});
```

When any field token starts with `-`, that `fields` scope uses exclude mode. For example, `fields: 'id,-compiledCode'` still returns all readable fields except `compiledCode`.

### 3. Proper Error Handling

```javascript
//  Good - comprehensive error handling
try {
  const result = await @REPOS.users.create({ data: userData });
  @LOGS('User created successfully');
  return result;
} catch (error) {
  @LOGS('User creation failed:', error.message);
  @THROW500('Failed to create user');
}
```

### 4. Cache Strategy

```javascript
//  Good - cache with appropriate TTL
const cacheKey = `user:${userId}`;
let user = await @CACHE.get(cacheKey);

if (!user) {
  user = await @REPOS.users.find({ filter: { id: userId } });
  await @CACHE.set(cacheKey, user, 1800000); // 30 minutes
}
```

## Compatibility & Flexibility

- ** All Three Syntaxes Supported**: Use `$ctx.$property`, `@TEMPLATE`, OR `#table_name` - your choice!
- ** Mix and Match**: You can use any combination in the same file
- ** All Contexts**: Works in Bootstrap Scripts, Hooks, and Custom Handlers
- ** IDE Support**: `$ctx.$property` has better autocomplete support
- ** Debugging**: Stack traces show the processed `$ctx.$property` syntax
- ** No Performance Impact**: Template replacement is just string processing
- ** Maximum Flexibility**: Choose the syntax that fits your style best

## Related Documentation

- [Context Reference](./context-reference/README.md) - Complete `$ctx` object documentation
- [File Handling](./file-handling.md) - File upload and response streaming guide
- [Custom Handlers](./hooks-handlers/custom-handlers.md) - Using templates in handlers
- [PreHooks](./hooks-handlers/prehooks.md) - Using templates in prehooks
- [postHooks](./hooks-handlers/posthooks.md) - Using templates in postHooks

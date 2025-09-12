# Hook Development Guide

## What are Hooks?

Hooks are a powerful middleware system that allows you to inject custom JavaScript code at specific points in the API request lifecycle. Unlike [Custom Handlers](../frontend/custom-handlers.md) which replace the entire operation, hooks **extend** the default behavior with minimal code.

## Hook Types

Enfyra provides two types of hooks:

### PreHook
- **Executes**: Before the main database operation
- **Purpose**: Modify incoming requests, validate data, transform input
- **Can modify**: `$ctx.$body`, `$ctx.$query`, `$ctx.$params`
- **Common uses**: Input validation, data transformation, automatic field generation, query filtering

### AfterHook  
- **Executes**: After the main database operation
- **Purpose**: Transform responses, add metadata, perform cleanup
- **Can modify**: `$ctx.$data` (response), `$ctx.$statusCode`
- **Common uses**: Response formatting, audit logging, computed fields, data masking

## What Can Hooks Do?

Hooks can perform a wide variety of operations:

### 🔧 Data Transformation
- **Auto-generate fields**: Create slugs, timestamps, UUIDs
- **Hash sensitive data**: Encrypt passwords, API keys
- **Format inputs**: Clean phone numbers, normalize emails
- **Convert data types**: String to numbers, date parsing

### ✅ Validation & Security
- **Input validation**: Email format, required fields, data ranges
- **Business rules**: Check inventory, validate relationships
- **Security checks**: Rate limiting, IP restrictions, role validation
- **Data sanitization**: Remove HTML tags, escape SQL

### 🔍 Query Modification
- **Auto-filtering**: Hide inactive records, apply tenant isolation
- **Dynamic conditions**: Add user-specific filters
- **Permission-based filtering**: Show only accessible data
- **Search enhancement**: Add full-text search, fuzzy matching

### 📊 Response Enhancement
- **Computed fields**: Calculate totals, format names, generate URLs
- **Metadata addition**: Add request info, processing time, cache status
- **Data masking**: Hide sensitive info based on user role
- **Format conversion**: Transform data for different clients

### 📝 Auditing & Logging
- **Audit trails**: Log all data changes with user context
- **Performance monitoring**: Track request timing, database queries
- **Debug information**: Add detailed execution logs
- **Business analytics**: Track user actions, feature usage

### 🔗 Integration
- **External APIs**: Send notifications, sync with third-party services
- **Cache management**: Invalidate cache entries, update search indexes
- **Event triggering**: Notify other systems of data changes
- **Workflow automation**: Trigger business processes

## Why Use Hooks?

### ✅ Advantages over Custom Handlers
- **Lightweight**: No need to reimplement CRUD operations
- **Composable**: Multiple hooks can work together on same route
- **Maintainable**: Small, focused pieces of code
- **Performance**: Less overhead than full handler replacement
- **Priority system**: Control execution order across hooks

### ✅ Perfect for Cross-Cutting Concerns
- **Consistent validation**: Apply same rules across multiple tables
- **Automatic auditing**: Log changes without modifying business logic
- **Global transformations**: Apply company-wide data policies
- **Security enforcement**: Implement consistent access controls

---

This guide covers writing JavaScript code for hooks - both PreHooks (before operations) and AfterHooks (after operations). For UI instructions on creating hooks, see [Hooks System](../frontend/hooks.md).

## Hook Context ($ctx)

Hooks use the same context object as handlers, providing full access to request data and system functions:

### Request Data
```javascript
$ctx.$body        // Request body (POST/PATCH data) - modifiable in preHook
$ctx.$params      // URL parameters (e.g., :id from /products/:id)
$ctx.$query       // Query string parameters - modifiable in preHook
$ctx.$user        // Current authenticated user information
```

### Request Information
```javascript
$ctx.$req.method  // HTTP method (GET, POST, PATCH, DELETE)
$ctx.$req.url     // Full request URL
$ctx.$req.ip      // Client IP address
$ctx.$req.headers // Request headers (authorization, user-agent)
```

### Response Data (AfterHook only)
```javascript
$ctx.$data        // Response data from the main operation - modifiable
$ctx.$statusCode  // HTTP status code - modifiable
```

### Database Access
```javascript
$ctx.$repos       // Access to table repositories (same as in Custom Handlers)
// Use the table names configured in route's targetTables
$ctx.$repos.products    // If "products" is in targetTables
$ctx.$repos.categories  // If "categories" is in targetTables
```

**Important**: Configure the tables you need in the route's **Target Tables** field. Each target table becomes available as a repository in `$ctx.$repos`.

### Utility Functions
**⚠️ Important: All helper functions require `await`** - They use IPC bridge for execution:

```javascript
// JWT token generation (async)
const token = await $ctx.$helpers.$jwt(payload, expiration)
// Example: const token = await $ctx.$helpers.$jwt({userId: 123}, '7d')

// Password hashing and verification (async)  
const hash = await $ctx.$helpers.$bcrypt.hash(plainPassword)
const isValid = await $ctx.$helpers.$bcrypt.compare(plainPassword, hashedPassword)

// URL-friendly slug generation (async)
const slug = await $ctx.$helpers.autoSlug(text)
// Example: const slug = await $ctx.$helpers.autoSlug('My Product Name') → 'my-product-name'
```

### Logging System
```javascript
$ctx.$logs('Debug message', data)
$ctx.$logs('Hook executed:', { hookName: 'validation-hook' })
// Logs are automatically included in API response for client-side debugging

$ctx.$share.$logs  // Array of all logged messages from current request
// Useful for debugging or sharing data between preHook and afterHook
```

## PreHook Examples

PreHooks execute **before** the main database operation, allowing you to modify the request or add validation.

### Modify Query Filter
```javascript
// Add automatic filtering to hide inactive records
if (!$ctx.$query.filter) {
  $ctx.$query.filter = {};
}

$ctx.$query.filter = {
  _and: [
    $ctx.$query.filter,
    { isActive: { _eq: true } }  // Always filter active records
  ]
};

$ctx.$logs('Added active filter to query');
```

### Auto-Generate Slug
```javascript
// Automatically create slug from name field
if ($ctx.$body.name && !$ctx.$body.slug) {
  $ctx.$body.slug = await $ctx.$helpers.autoSlug($ctx.$body.name);
  $ctx.$logs('Generated slug:', $ctx.$body.slug);
}
```

### Hash Password (System Hook Example)
```javascript
// Hash password before saving to database
if ($ctx.$body.password) {
  $ctx.$body.password = await $ctx.$helpers.$bcrypt.hash($ctx.$body.password);
  $ctx.$logs('Password hashed for user');
}
```

### Request Validation
```javascript
// Validate email format
if ($ctx.$body.email && !$ctx.$body.email.includes('@')) {
  throw new Error('Invalid email format');
}

// Add creation timestamp
if ($ctx.$req.method === 'POST') {
  $ctx.$body.createdAt = new Date().toISOString();
  $ctx.$body.createdBy = $ctx.$user.id;
}

$ctx.$logs('Validation passed for:', $ctx.$body.email);
```

### Dynamic Field Population
```javascript
// Add user context to data
if ($ctx.$req.method === 'POST' || $ctx.$req.method === 'PATCH') {
  $ctx.$body.modifiedBy = $ctx.$user.id;
  $ctx.$body.modifiedAt = new Date().toISOString();
  
  if ($ctx.$req.method === 'POST') {
    $ctx.$body.organizationId = $ctx.$user.organizationId; // Auto-assign organization
  }
}

$ctx.$logs('Added audit fields for user:', $ctx.$user.id);
```

### Complex Validation with Database Access
```javascript
// Validate unique email before creating user
if ($ctx.$req.method === 'POST' && $ctx.$body.email) {
  const existingUser = await $ctx.$repos.users.find({
    where: { email: { _eq: $ctx.$body.email } }
  });
  
  if (existingUser.data.length > 0) {
    throw new Error('Email already exists');
  }
  
  $ctx.$logs('Email validation passed:', $ctx.$body.email);
}
```

## AfterHook Examples

AfterHooks execute **after** the main database operation, allowing you to transform the response data.

### Transform Response Data
```javascript
// Add computed fields to response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`,
    isOwner: item.createdBy === $ctx.$user.id
  }));
}

$ctx.$logs('Added computed fields to response');
```

### Add Response Metadata
```javascript
// Add custom metadata to response
$ctx.$data = {
  ...$ctx.$data,
  meta: {
    ...$ctx.$data.meta,
    requestedBy: $ctx.$user.email,
    requestedAt: new Date().toISOString(),
    apiVersion: 'v2.1'
  }
};

$ctx.$logs('Added custom metadata to response');
```

### Success Message Hook (System Hook Example)
```javascript
// Add standardized success message
$ctx.$data = { 
  statusCode: $ctx.$statusCode, 
  ...$ctx.$data, 
  message: 'Success' 
};
```

### Conditional Response Modification
```javascript
// Hide sensitive data for non-admin users
if ($ctx.$user.role !== 'admin' && $ctx.$data.data) {
  if (Array.isArray($ctx.$data.data)) {
    $ctx.$data.data = $ctx.$data.data.map(item => {
      const { password, secretKey, internalNotes, ...publicData } = item;
      return publicData;
    });
  } else {
    const { password, secretKey, internalNotes, ...publicData } = $ctx.$data.data;
    $ctx.$data.data = publicData;
  }
  
  $ctx.$logs('Removed sensitive fields for non-admin user');
}
```

### Audit Log Creation
```javascript
// Log all data modifications to audit table
if ($ctx.$req.method !== 'GET' && $ctx.$data.data) {
  const auditEntry = {
    userId: $ctx.$user.id,
    action: $ctx.$req.method,
    tableName: 'users', // or get from route context
    recordId: $ctx.$data.data.id,
    changes: JSON.stringify($ctx.$body),
    timestamp: new Date().toISOString()
  };
  
  await $ctx.$repos.audit_logs.create(auditEntry);
  $ctx.$logs('Created audit log entry');
}
```

## Hook Execution Flow

```
1. Request arrives → Context ($ctx) created
2. PreHook executes → Can modify $ctx.$body, $ctx.$query, etc.
3. Main operation (CRUD/Handler) → Uses modified context
4. AfterHook executes → Can modify $ctx.$data (response)
5. Response sent → Includes logs if any exist
```

### Context Sharing Between Hooks
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

## Hook Priority and Execution Order

- **Priority**: Lower numbers execute first (0 = highest priority)
- **Multiple Hooks**: Hooks with the same priority execute in database order
- **PreHooks**: All preHooks execute before the main operation
- **AfterHooks**: All afterHooks execute after the main operation

### Priority Example
```javascript
// Priority 0 - executes first
{ 
  priority: 0, 
  preHook: "$ctx.$logs('First preHook')" 
}

// Priority 10 - executes second  
{ 
  priority: 10, 
  preHook: "$ctx.$logs('Second preHook')" 
}

// Main CRUD operation happens here

// Priority 0 - executes first in afterHooks
{ 
  priority: 0, 
  afterHook: "$ctx.$logs('First afterHook')" 
}
```

## Database Repository Methods

Each repository in `$ctx.$repos` uses the same methods as in [Custom Handlers](../frontend/custom-handlers.md):

### Basic Operations
```javascript
// Find records (returns {data: [], meta: {totalCount, filterCount}})
const result = await $ctx.$repos.products.find({
  where: { category: { _eq: 'electronics' } }
});
const products = result.data;

// Create new record (returns query result with created record)
const result = await $ctx.$repos.products.create({
  name: $ctx.$body.name,
  price: $ctx.$body.price
});
const newProduct = result.data[0];

// Update record by ID (returns query result with updated record)
const result = await $ctx.$repos.products.update($ctx.$params.id, {
  price: $ctx.$body.price
});
const updatedProduct = result.data[0];

// Delete record by ID (returns success message)
const result = await $ctx.$repos.products.delete($ctx.$params.id);
// Returns: { message: 'Delete successfully!', statusCode: 200 }
```

### Query Filtering
The `where` field uses **Enfyra's API Filtering system** - the same MongoDB-like operators available in the API (see [API Filtering](./api-filtering.md)):

```javascript
// Complex filtering using API Filtering operators
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

## System Hooks

Enfyra includes several built-in system hooks that demonstrate best practices:

### Default Response Hook
```javascript
// afterHook - adds standard success message to all responses
$ctx.$data = { 
  statusCode: $ctx.$statusCode, 
  ...$ctx.$data, 
  message: 'Success'
}
```

### User Password Hash Hook  
```javascript
// preHook - automatically hashes passwords for user_definition table
if ($ctx.$body.password) { 
  $ctx.$body.password = await $ctx.$helpers.$bcrypt.hash($ctx.$body.password);
}
```

### Auto-Slug Generation Hook
```javascript
// preHook - creates URL-friendly slugs from name field
if ($ctx.$body.name) { 
  $ctx.$body.slug = await $ctx.$helpers.autoSlug($ctx.$body.name); 
}
```

## Best Practices

### Hook Design
- **Single Responsibility**: Each hook should do one specific thing
- **Conditional Logic**: Use `$ctx.$req.method` to run different logic for different HTTP methods
- **Error Handling**: Use `throw new Error('message')` for validation failures
- **Idempotent Operations**: Ensure hooks can run multiple times safely

### Code Patterns
- **Always Await Helpers**: All `$ctx.$helpers` functions require `await` due to IPC bridge
- **Extract Helper Results**: Store helper results in variables before using them
- **Check Data Arrays**: Repository methods return `{data: []}`, always check `data.length`
- **Shared Context**: Use `$ctx.$share` to pass data between preHook and afterHook

### Performance
- **Lightweight Code**: Keep hook code simple and fast
- **Avoid Heavy Operations**: Use custom handlers for complex business logic  
- **Minimal Database Queries**: Only query what you absolutely need
- **Cache Results**: Store expensive computations in `$ctx.$share`

### Security
- **User Context**: Always validate `$ctx.$user` permissions when needed
- **Input Sanitization**: Clean and validate `$ctx.$body` data in preHooks
- **Sensitive Data**: Remove sensitive fields in afterHooks for non-privileged users
- **SQL Injection Prevention**: Use repository methods instead of raw SQL

### Debugging
- **Use Logging**: Add `$ctx.$logs()` calls to track hook execution
- **Check Request Method**: Log `$ctx.$req.method` to see which operations trigger hooks
- **Inspect Context**: Log `$ctx.$body` and `$ctx.$data` to see transformations
- **Priority Testing**: Use different priorities to ensure correct execution order
- **Client-Side Debugging**: Logs appear automatically in API responses for easy debugging

### Error Handling
- **Throw Errors**: Use `throw new Error('message')` for validation failures
- **Descriptive Messages**: Provide clear error messages for debugging
- **Status Codes**: Errors automatically return appropriate HTTP status codes
- **Graceful Fallbacks**: Handle cases where expected data might be missing

### Integration with Other Systems
- **Filter Modification**: Perfect for modifying [API Filtering](./api-filtering.md) queries
- **Handler Coordination**: Use hooks to set up context for [Custom Handlers](../frontend/custom-handlers.md)
- **Route Integration**: Work seamlessly with [Routing Management](../frontend/routing-management.md)

Hooks provide powerful customization capabilities while maintaining the simplicity and performance of the core Enfyra system.
# API Lifecycle

Understanding how API requests flow through Enfyra helps you build effective hooks and handlers. This guide explains the complete request lifecycle and how the context object flows through each phase.

## Quick Navigation

- [Request Flow Overview](#request-flow-overview) - High-level lifecycle
- [Phase Breakdown](#phase-breakdown) - Detailed explanation of each phase
- [Context Sharing](#context-sharing) - How $ctx flows through phases
- [Execution Order](#execution-order) - Hook and handler execution sequence
- [Common Patterns](#common-patterns) - Real-world examples

## Request Flow Overview

Every API request in Enfyra follows this lifecycle:

```
HTTP Request → Route Detection → Context Setup → preHooks → Handler → afterHooks → Response
```

### Visual Flow

```
1. Client sends HTTP request
   ↓
2. System detects route and matches to route definition
   ↓
3. Context ($ctx) is created with repositories and helpers
   ↓
4. All matching preHooks execute sequentially
   ↓
5. Handler executes (custom handler or default CRUD)
   ↓
6. All matching afterHooks execute sequentially
   ↓
7. Response is sent back to client
```

## Phase Breakdown

### Phase 1: Route Detection

The system automatically matches the incoming request to a route definition.

**What happens:**
- Request URL and method are analyzed
- Route cache is checked for matching route
- Route definition is loaded from database
- Target tables are identified
- Hooks and handlers are discovered

**Automatic operation:**
- No manual configuration needed
- Routes are defined in the database
- System handles all matching logic

### Phase 2: Context Setup

The `$ctx` (context) object is created and initialized.

**What's included:**
- Request data: `$ctx.$body`, `$ctx.$params`, `$ctx.$query`, `$ctx.$user`
- Repositories: `$ctx.$repos` for all target tables
- Helpers: `$ctx.$helpers` for utilities (JWT, bcrypt, etc.)
- Cache: `$ctx.$cache` for Redis operations
- Logging: `$ctx.$logs()` function
- Error handling: `$ctx.$throw` methods

**Important:** The same `$ctx` object reference is used throughout the entire request lifecycle.

### Phase 3: preHooks Execution

All matching preHooks execute sequentially before the handler.

**Execution order:**
1. Global preHooks (all routes, all methods)
2. Global preHooks (all routes, specific method)
3. Route-specific preHooks (specific route, all methods)
4. Route-specific preHooks (specific route, specific method)

**What preHooks can do:**
- Validate request data
- Transform input data
- Check permissions
- Modify `$ctx.$body` or `$ctx.$query`
- Store data in `$ctx.$share` for later use
- Throw errors to stop execution

**Example:**
```javascript
// preHook: Validate and transform
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$share.validationPassed = true;
```

### Phase 4: Handler Execution

The handler executes the main business logic.

**Handler types:**
- **Custom Handler**: Your custom code from route definition
- **Default CRUD**: Automatic CRUD operation based on HTTP method

**What handlers can do:**
- Use repositories to query/create/update/delete data
- Access all `$ctx` properties modified by preHooks
- Return data that will be available in `$ctx.$data` for afterHooks
- Throw errors

**Example:**
```javascript
// Custom handler
const result = await $ctx.$repos.products.create({
  data: $ctx.$body
});

return result;
```

### Phase 5: afterHooks Execution

All matching afterHooks execute sequentially after the handler.

**Execution order:**
1. Global afterHooks (all routes, all methods)
2. Global afterHooks (all routes, specific method)
3. Route-specific afterHooks (specific route, all methods)
4. Route-specific afterHooks (specific route, specific method)

**What afterHooks can do:**
- Transform response data
- Add computed fields
- Log audit trails
- Trigger side effects (emails, notifications)
- Handle errors that occurred in handler
- Access `$ctx.$data` with handler response

**Example:**
```javascript
// afterHook: Transform response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

### Phase 6: Response Delivery

The processed response is sent back to the client.

**Response includes:**
- Data from handler (possibly modified by afterHooks)
- Logs collected from all phases
- HTTP status code
- Headers

## Context Sharing

The `$ctx` object is the **same reference** throughout the entire request lifecycle. This means:

### Persistent Reference

Changes made in one phase are visible in all subsequent phases:

```javascript
// preHook #1 modifies context
$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$share.validationPassed = true;

// preHook #2 sees the changes
$ctx.$logs(`Email normalized: ${$ctx.$body.email}`);
$ctx.$logs(`Validation: ${$ctx.$share.validationPassed}`);

// Handler also sees all changes
const email = $ctx.$body.email;  // Already normalized
if ($ctx.$share.validationPassed) {
  // Validation already passed
}

// afterHook gets final state
$ctx.$data.email = $ctx.$body.email;  // Use normalized email
```

### Available Context Properties

Throughout all phases, you have access to:

```javascript
// Request data (immutable after setup)
$ctx.$req          // Express request object
$ctx.$user         // Authenticated user

// Request data (mutable)
$ctx.$body         // Request body - can be modified in preHooks
$ctx.$params       // URL parameters
$ctx.$query        // Query string parameters

// Repositories (available after context setup)
$ctx.$repos        // Table repositories

// Utilities
$ctx.$helpers      // Helper functions
$ctx.$cache        // Cache operations
$ctx.$logs()       // Logging function
$ctx.$throw        // Error throwing

// Response data (available in handler and afterHook)
$ctx.$data         // Response data from handler
$ctx.$statusCode   // HTTP status code

// Shared context (persists across all phases)
$ctx.$share        // Shared data container

// API information (available in afterHook)
$ctx.$api          // Request/response/error details
```

## Execution Order

Hooks execute in a predictable order based on their scope and method filters.

### Hook Filtering Logic

A hook runs if it matches any of these conditions:
- Global hook with no method filter (runs on all routes, all methods)
- Global hook with method filter (runs on all routes, specific method)
- Route-specific hook with no method filter (runs on specific route, all methods)
- Route-specific hook with method filter (runs on specific route, specific method)

### Example Execution

For a `POST /users` request with these hooks:

1. Global preHook (all methods)
2. Global preHook (POST only)
3. Route preHook for `/users` (all methods)
4. Route preHook for `/users` (POST only)

**Execution sequence:**
```
[Global preHook - all] → [Global preHook - POST] → [Route preHook - all] → [Route preHook - POST] → [Handler] → [afterHooks in same order]
```

### Sequential Execution

All matching hooks run **sequentially** (one after another), not in parallel. This ensures:
- Predictable execution order
- Changes from one hook are visible to the next
- Easy debugging and logging

## Common Patterns

### Pattern 1: Validation in preHook

```javascript
// preHook: Validate request
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

if (!$ctx.$body.password || $ctx.$body.password.length < 6) {
  $ctx.$throw['422']('Password must be at least 6 characters');
  return;
}

// Store validation result for later use
$ctx.$share.validationPassed = true;
```

### Pattern 2: Data Transformation in preHook

```javascript
// preHook: Normalize and enrich data
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
$ctx.$body.slug = await $ctx.$helpers.autoSlug($ctx.$body.title);

// Add computed fields
$ctx.$body.createdBy = $ctx.$user.id;
$ctx.$body.createdAt = new Date();
```

### Pattern 3: Response Enhancement in afterHook

```javascript
// afterHook: Add metadata to response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`,
    processedAt: new Date()
  }));
}

$ctx.$data.meta = {
  processedBy: $ctx.$user.id,
  timestamp: new Date()
};
```

### Pattern 4: Shared Context Between Hooks

```javascript
// preHook: Store data
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// afterHook: Use shared data
if ($ctx.$share.processStartTime) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Pattern 5: Error Handling in afterHook

```javascript
// afterHook: Handle errors
if ($ctx.$api.error) {
  // Error occurred
  $ctx.$logs(`Error: ${$ctx.$api.error.message}`);
  
  // Log to audit system
  await $ctx.$repos.audit_logs.create({
    data: {
      action: 'error_occurred',
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      userId: $ctx.$user?.id
    }
  });
} else {
  // Success
  $ctx.$logs('Operation completed successfully');
}
```

### Pattern 6: Permission Checking in preHook

```javascript
// preHook: Check permissions
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

## Best Practices

### Context Management

1. **Use descriptive names** for custom properties in `$ctx.$share`
2. **Check existence** before accessing nested properties
3. **Clean up** temporary properties in afterHooks if needed
4. **Log important changes** for debugging

### Hook Organization

1. **Global hooks** for cross-cutting concerns (auth, logging, audit)
2. **Route-specific hooks** for business logic
3. **Method-specific hooks** for operation-specific logic
4. **Keep hooks focused** - one responsibility per hook

### Performance

1. **Minimize database calls** in hooks - batch operations when possible
2. **Cache expensive operations** in `$ctx.$share`
3. **Use early returns** to avoid unnecessary processing
4. **Consider execution order** for optimal performance

### Error Handling

1. **Validate early** in preHooks to fail fast
2. **Provide meaningful error messages**
3. **Use `$ctx.$logs()`** for debugging information
4. **Handle errors gracefully** in afterHooks when possible

## Next Steps

- Learn about [Repository Methods](./repository-methods/) for database operations
- See [Context Reference](./context-reference/) for all available properties
- Check [Hooks and Handlers](./hooks-handlers/) for creating custom logic


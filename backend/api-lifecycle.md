# API Lifecycle

Understanding Enfyra's API request lifecycle is crucial for building effective hooks and handlers. This guide explains the complete flow and how context sharing works throughout the request.


## Quick Navigation

**Core Concepts:**
- [Request Flow](#request-flow) - Complete lifecycle overview
- [Context Sharing ($ctx)](#context-sharing-ctx) - How data flows between phases
- [Hook System](#hook-system) - Global vs route-specific hooks
- [Execution Order](#execution-order) - Predictable hook sequencing

**Practical Usage:**
- [Context Modifications](#context-modifications) - Changing data between hooks
- [Real Examples](#real-examples) - Complete lifecycle scenarios
- [Best Practices](#best-practices) - Effective lifecycle management

---

## Request Flow

Every API request in Enfyra follows this lifecycle:

```
HTTP Request → Route Detection (High-Performance) → Context Setup → preHook(s) → Handler → afterHook(s) → Response
```

**Performance Note**: Route detection bypasses Express's middleware stack and uses a custom engine that's significantly faster than Express's built-in route tree matching.

### Phase Breakdown

**1. Route Detection (Automatic)**
- **High-Performance Engine**: Bypasses Express middleware stack entirely
- **Custom Route Matching**: Uses optimized algorithm that's many times faster than Express's route tree
- **Smart Parameter Extraction**: Automatically extracts path parameters (e.g., `/users/123` → `{ id: "123" }`)
- **Hook & Handler Discovery**: Determines all applicable hooks and handlers based on route and method
- **Zero Configuration**: This entire process is 100% automatic - you never need to configure routes manually

**2. Context Setup ($ctx)**
- Creates shared context object
- Initializes repositories for all target tables
- Sets up helper functions and user data
- **CRITICAL: Same reference throughout entire request**

**3. preHook(s) Execution**
- All matching preHooks run sequentially
- Can modify request data, add validation, transform input
- Changes to $ctx affect subsequent hooks and handlers

**4. Handler Execution**  
- Custom handler OR default CRUD operation
- Has access to all $ctx modifications from preHooks
- Can use modified data and add its own changes

**5. afterHook(s) Execution**
- All matching afterHooks run sequentially
- Can transform response, add computed fields, trigger side effects
- Final $ctx state affects the response

**6. Response Delivery**
- Returns processed data with optional logs
- Includes any modifications made throughout lifecycle

## Context Sharing ($ctx)

The `$ctx` object is the **same reference** throughout the entire request lifecycle. This means:

### **Persistent Reference**
```javascript
// preHook #1 modifies context
$ctx.$share.customField = "hello";
$ctx.$body.processed = true;

// preHook #2 sees the changes
console.log($ctx.$share.customField); // "hello"
console.log($ctx.$body.processed); // true

// Handler also sees all changes
if ($ctx.$body.processed) {
  // This will execute because preHook #1 set it
}

// afterHook gets final state
$ctx.$share.customField += " world"; // Now "hello world"
```

### **Available Context Properties**

```javascript
$ctx = {
  // Request data
  $body: {},           // Request body (mutable)
  $params: {},         // Path parameters (e.g., /users/:id)
  $query: {},          // Query parameters (e.g., ?limit=10)
  $user: {},           // Authenticated user info
  
  // Database access
  $repos: {            // Dynamic repositories for all tables
    main: repository,  // Main table repository
    users: repository, // Named table repositories
    // ... other target tables
  },
  
  // Utilities
  $helpers: {          // Utility functions
    $jwt: function,    // JWT token creation
    $bcrypt: object,   // Password hashing
    autoSlug: function // Slug generation
  },
  
  // Logging
  $logs: function,     // Add logs to response
  $share: {            // Shared data container
    $logs: []          // Collected logs
  },
  
  // Response control
  $statusCode: 200,    // HTTP status (can be changed)
  $data: {},          // Response data (available in afterHook)
  
  // Error handling
  $errors: {}          // Error handlers
};
```

## Hook System

### Global vs Route-Specific Hooks

**Global Hooks** run on ALL API endpoints:
```javascript
{
  "route": null,          // No specific route
  "methods": [],          // Empty = all HTTP methods
  "preHook": "// code",
  "afterHook": "// code"
}

// This runs on: GET /users, POST /products, DELETE /orders, etc.
```

**Method-Specific Global Hooks:**
```javascript
{
  "route": null,          // No specific route
  "methods": ["GET"],     // Only GET requests
  "preHook": "// code"
}

// This runs on: GET /users, GET /products, but NOT POST /users
```

**Route-Specific Hooks:**
```javascript
{
  "route": { "id": "route123", "path": "/users" },
  "methods": ["POST"],
  "preHook": "// code"
}

// This runs ONLY on: POST /users
```

### Hook Filtering Logic

The system uses this exact logic to determine which hooks run:

```javascript
const isGlobalAll = !hook.route && methodList.length === 0;        // Global + all methods
const isGlobalMethod = !hook.route && methodList.includes(method); // Global + specific method
const isLocalAll = hook.route?.id === route.id && methodList.length === 0;     // Route + all methods
const isLocalMethod = hook.route?.id === route.id && methodList.includes(method); // Route + specific method

// Hook runs if ANY condition is true
```

## Execution Order

Hooks execute **sequentially** in database order (no priority system):

```javascript
// Request: POST /users
// Available hooks:
// 1. Global preHook (all methods)
// 2. Global preHook (POST only) 
// 3. /users preHook (all methods)
// 4. /users preHook (POST only)

Execution Flow:
[preHook #1] → [preHook #2] → [preHook #3] → [preHook #4] → [Handler] → [afterHook #1] → [afterHook #2]
```

### Predictable Execution

```javascript
// preHook #1: Sets base validation
$ctx.$share.validated = false;
if ($ctx.$body.email) {
  $ctx.$share.validated = true;
}

// preHook #2: Adds additional checks (sees validated = true)
if ($ctx.$share.validated && $ctx.$body.name) {
  $ctx.$share.fullyValidated = true;
}

// Handler: Uses validation results
if (!$ctx.$share.fullyValidated) {
  throw new Error("Validation failed");
}

// afterHook: Adds final metadata (sees all previous changes)
$ctx.$data.validationPassed = $ctx.$share.fullyValidated;
```

## Context Modifications

### Modifying Request Data

```javascript
// preHook: Transform incoming data
if ($ctx.$body.email) {
  $ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
}

// Add computed fields
$ctx.$body.slug = $ctx.$helpers.autoSlug($ctx.$body.title);

// Store data in $share for other hooks/handlers
$ctx.$share.requestTimestamp = new Date();
$ctx.$share.validationErrors = [];
```

### Modifying Query Parameters

```javascript
// preHook: Add filters based on user
if ($ctx.$user.role !== 'admin') {
  // Modify the filter to only show user's own records
  $ctx.$query.filter = {
    ...($ctx.$query.filter || {}),
    userId: { _eq: $ctx.$user.id }
  };
}
```

### Modifying Response Data

```javascript
// afterHook: Transform response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  // Add computed field to each item
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}

// Add metadata
$ctx.$data.processedAt = new Date();
$ctx.$data.processedBy = $ctx.$user.id;
```

## Real Examples

### Example 1: User Registration with Validation

```javascript
// preHook #1: Validate input
if (!$ctx.$body.email || !$ctx.$body.password) {
  throw new Error("Email and password required");
}

// Store validation result in $share
$ctx.$share.validationPassed = true;
$ctx.$logs("Validation passed for", $ctx.$body.email);

// preHook #2: Hash password (sees validationPassed in $share)
if ($ctx.$share.validationPassed) {
  $ctx.$body.password = await $ctx.$helpers.$bcrypt.hash($ctx.$body.password);
  $ctx.$logs("Password hashed");
}

// Handler: Create user (sees hashed password and $share data)
// Uses default CRUD with modified $ctx.$body

// afterHook: Generate welcome token (sees created user data)
$ctx.$data.welcomeToken = $ctx.$helpers.$jwt(
  { userId: $ctx.$data.id, type: 'welcome' },
  '24h'
);
$ctx.$logs("Welcome token generated");
```

### Example 2: Global Audit Logging

```javascript
// Global preHook: Log all requests
$ctx.$share.auditData = {
  userId: $ctx.$user?.id,
  action: `${$ctx.$req.method} ${$ctx.$req.path}`,
  timestamp: new Date(),
  ip: $ctx.$req.ip
};

// Any subsequent preHook sees auditData in $share
if ($ctx.$share.auditData.userId) {
  $ctx.$logs("Authenticated request by user", $ctx.$share.auditData.userId);
}

// Global afterHook: Save audit log
await $ctx.$repos.auditLogs.create($ctx.$share.auditData);
```

### Example 3: Multi-Step Data Processing

```javascript
// preHook #1: Parse and validate JSON fields
if ($ctx.$body.metadata) {
  try {
    $ctx.$body.metadata = JSON.parse($ctx.$body.metadata);
    $ctx.$share.metadataParsed = true;
  } catch (e) {
    $ctx.$share.metadataParsed = false;
    $ctx.$logs("Invalid JSON in metadata field");
  }
}

// preHook #2: Enrich metadata (sees parsed data status in $share)
if ($ctx.$share.metadataParsed && $ctx.$body.metadata) {
  $ctx.$body.metadata.processedAt = new Date();
  $ctx.$body.metadata.processedBy = $ctx.$user.id;
  $ctx.$logs("Metadata enriched");
}

// Handler: Save enriched data

// afterHook: Generate processing report
$ctx.$data.processingReport = {
  metadataParsed: $ctx.$share.metadataParsed,
  fieldsProcessed: Object.keys($ctx.$body.metadata || {}).length,
  totalProcessingSteps: 2
};
```

## Best Practices

### Context Management
1. **Use descriptive names** for custom context properties
2. **Check existence** before accessing nested properties
3. **Clean up** temporary properties in afterHooks if needed
4. **Log important changes** for debugging

### Hook Organization
1. **Global hooks** for cross-cutting concerns (auth, logging, audit)
2. **Route-specific hooks** for business logic
3. **Method-specific hooks** for operation-specific logic
4. **Keep hooks focused** - one responsibility per hook

### Performance Considerations
1. **Minimize database calls** in hooks
2. **Cache expensive operations** in context
3. **Use early returns** to avoid unnecessary processing
4. **Consider hook execution order** for optimal performance

### Error Handling
1. **Validate context state** before using it
2. **Provide meaningful error messages**
3. **Use $ctx.$logs** for debugging information
4. **Clean up resources** in case of errors

## Technical Deep Dive: Route Detection

### High-Performance Route Engine

Enfyra uses a custom route detection system that provides significant performance advantages:

**Traditional Express Approach:**
```javascript
// Express processes middleware stack sequentially
app.use(middleware1);
app.use(middleware2);
app.get('/users/:id', handler); // Goes through entire middleware chain
```

**Enfyra's Approach:**
```javascript
// Bypasses Express middleware stack entirely
// Direct route matching with custom algorithm
// 1. Load all routes from database with SWR (Stale-While-Revalidate) caching
// 2. Use optimized matching algorithm on cached routes
// 3. Extract parameters and context in single pass
// 4. Directly execute matched hooks/handlers
```

**Performance Benefits:**
- **Many times faster** than Express route tree matching
- **No middleware overhead** for API routes
- **Direct execution path** from request to handler
- **SWR Caching**: Routes loaded from database with Stale-While-Revalidate pattern for extreme performance
- **Optimized for dynamic routes** generated from database

**Automatic Operation:**
- Route detection is **100% automatic**
- No manual route configuration required
- **SWR Pattern**: Routes cached for instant access, revalidated in background
- Dynamic routes generated from your route definitions
- Supports complex patterns: `/users/:id`, `/api/v1/products`, etc.
- Parameter extraction handled automatically


## Best Practices

### Context Management
1. **Use `$ctx.$share`** for custom properties that need to persist across hooks
2. **Check existence** before accessing nested properties
3. **Clean up** temporary properties in afterHooks if needed
4. **Log important changes** using `$ctx.$logs()` for debugging

### Hook Organization
1. **Global hooks** for cross-cutting concerns (auth, logging, audit)
2. **Route-specific hooks** for business logic
3. **Method-specific hooks** for operation-specific logic
4. **Keep hooks focused** - one responsibility per hook

### Performance Considerations
1. **Leverage the fast route engine** - it's optimized for high throughput
2. **Minimize database calls** in hooks - batch operations when possible
3. **Cache expensive operations** in `$ctx.$share`
4. **Use early returns** to avoid unnecessary processing
5. **Consider hook execution order** for optimal performance

### Error Handling
1. **Validate context state** before using it
2. **Provide meaningful error messages**
3. **Use `$ctx.$logs()`** for debugging information
4. **Clean up resources** in case of errors

This lifecycle system gives you complete control over every API request while maintaining predictable, debuggable behavior through the shared context approach.
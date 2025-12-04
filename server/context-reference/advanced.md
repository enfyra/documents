# Context Reference - Advanced Features

Advanced context features including file uploads, API information, shared context, and package access.

## File Uploads

Access information about uploaded files.

### $ctx.$uploadedFile

Information about the uploaded file (if any).

```javascript
if ($ctx.$uploadedFile) {
  const filename = $ctx.$uploadedFile.originalname;
  const mimetype = $ctx.$uploadedFile.mimetype;
  const size = $ctx.$uploadedFile.size;
  const buffer = $ctx.$uploadedFile.buffer;
  const fieldname = $ctx.$uploadedFile.fieldname;
}
```

### Processing Uploaded Files

```javascript
if ($ctx.$uploadedFile) {
  $ctx.$logs(`File uploaded: ${$ctx.$uploadedFile.originalname}`);
  $ctx.$logs(`File size: ${$ctx.$uploadedFile.size} bytes`);
  $ctx.$logs(`MIME type: ${$ctx.$uploadedFile.mimetype}`);
  
  // Save file using helper
  const fileResult = await $ctx.$helpers.$uploadFile({
    originalname: $ctx.$uploadedFile.originalname,
    mimetype: $ctx.$uploadedFile.mimetype,
    buffer: $ctx.$uploadedFile.buffer,
    size: $ctx.$uploadedFile.size
  });
}
```

## API Information

Access detailed information about the current API request and response.

### Request Information

```javascript
const method = $ctx.$api.request.method;           // HTTP method
const url = $ctx.$api.request.url;                 // Request URL
const timestamp = $ctx.$api.request.timestamp;     // Request timestamp
const correlationId = $ctx.$api.request.correlationId;  // Unique request ID
const userAgent = $ctx.$api.request.userAgent;     // User agent
const ip = $ctx.$api.request.ip;                   // Client IP
```

### Response Information

```javascript
const statusCode = $ctx.$api.response.statusCode;  // HTTP status code
const responseTime = $ctx.$api.response.responseTime;  // Response time in ms
const timestamp = $ctx.$api.response.timestamp;    // Response timestamp
```

### Error Information (AfterHook Only)

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

## Shared Context

Store data that persists across hooks and handlers.

### $ctx.$share

Shared data container for passing data between hooks.

```javascript
// In preHook - store data
$ctx.$share.validationPassed = true;
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// In afterHook - access shared data
if ($ctx.$share.validationPassed) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Accessing Logs

```javascript
// All logs are stored in $ctx.$share.$logs
const allLogs = $ctx.$share.$logs;  // Array of all logged messages
```

## Package Access

Access NPM packages installed in the system.

### $ctx.$pkgs

Access installed packages by name.

```javascript
// Use installed packages
const axios = $ctx.$pkgs.axios;
const lodash = $ctx.$pkgs.lodash;
const moment = $ctx.$pkgs.moment;

// Example: Using axios
if ($ctx.$pkgs.axios) {
  const response = await $ctx.$pkgs.axios.get('https://api.example.com/data');
}
```

**Note:** Packages must be installed through Package Management in the UI to be accessible.

## Common Patterns

### Pattern 1: Validate and Transform Request Data

```javascript
// Validate required fields
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

// Transform data
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
```

### Pattern 2: Check User Permissions

```javascript
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

if ($ctx.$user.role !== 'admin') {
  $ctx.$throw['403']('Admin access required');
  return;
}
```

### Pattern 3: Use Shared Context Between Hooks

```javascript
// In preHook
$ctx.$share.userId = $ctx.$user.id;
$ctx.$share.validationData = { passed: true };

// In afterHook
const userId = $ctx.$share.userId;
if ($ctx.$share.validationData.passed) {
  // Use validation result
}
```

### Pattern 4: Cache Frequently Accessed Data

```javascript
// Check cache first
const cacheKey = `user:${$ctx.$params.id}`;
let user = await $ctx.$cache.get(cacheKey);

if (!user) {
  // Cache miss - fetch from database
  const result = await $ctx.$repos.user_definition.find({
    where: { id: { _eq: $ctx.$params.id } }
  });
  
  if (result.data.length > 0) {
    user = result.data[0];
    // Cache for 5 minutes
    await $ctx.$cache.set(cacheKey, user, 300000);
  }
}
```

### Pattern 5: Handle Errors in AfterHook

```javascript
// In afterHook
if ($ctx.$api.error) {
  // Log error
  $ctx.$logs(`Error: ${$ctx.$api.error.message}`);
  
  // Log to audit system
  await $ctx.$repos.audit_logs.create({
    data: {
      action: 'error_occurred',
      userId: $ctx.$user?.id,
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      timestamp: new Date()
    }
  });
} else {
  // Success - add metadata
  $ctx.$data.processedAt = new Date();
  $ctx.$data.processedBy = $ctx.$user?.id;
}
```

## Next Steps

- See [File Handling](./file-handling.md) for complete file upload guide
- Check [Cache Operations](./cache-operations.md) for detailed cache patterns
- Learn about [Error Handling](./error-handling.md) for error handling patterns


# Hooks and Handlers - preHooks

preHooks execute before the handler. Use them for validation, data transformation, and permission checks.

## When to Use preHooks

- Validate request data
- Transform or normalize input data
- Check user permissions
- Modify request body or query parameters
- Store data in shared context for later use

## Basic preHook Example

```javascript
// Validate required fields
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

if (!$ctx.$body.password) {
  $ctx.$throw['400']('Password is required');
  return;
}

// Normalize email
$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();

// Store validation result
$ctx.$share.validationPassed = true;
```

## Data Transformation

```javascript
// Normalize data
$ctx.$body.email = $ctx.$body.email.toLowerCase();
$ctx.$body.name = $ctx.$body.name.trim();

// Generate slug
if ($ctx.$body.title) {
  $ctx.$body.slug = await $ctx.$helpers.autoSlug($ctx.$body.title);
}

// Add computed fields
$ctx.$body.createdBy = $ctx.$user.id;
$ctx.$body.createdAt = new Date();
```

## Permission Checking

```javascript
// Check authentication
if (!$ctx.$user) {
  $ctx.$throw['401']('Authentication required');
  return;
}

// Check role
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

if (resource.data[0].userId !== $ctx.$user.id) {
  $ctx.$throw['403']('Access denied');
  return;
}
```

## Modifying Query Parameters

```javascript
// Add user-specific filter
if ($ctx.$user.role !== 'admin') {
  // Non-admins only see their own records
  $ctx.$query.filter = {
    ...($ctx.$query.filter || {}),
    userId: { _eq: $ctx.$user.id }
  };
}
```

## Rate Limiting

Protect your API routes from abuse using the rate limiting helper.

### Basic Rate Limiting by IP

```javascript
// Limit requests per IP address
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 100,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Rate limit exceeded. Try again in ${result.retryAfter}s`);
  return;
}
```

### Rate Limit Login Attempts

```javascript
// Protect login endpoint from brute force
const result = await $ctx.$helpers.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 60
});

if (!result.allowed) {
  $ctx.$throw['429'](`Too many login attempts. Try again in ${result.retryAfter}s`);
  return;
}
```

### Rate Limit by User

```javascript
// Limit API requests per authenticated user
const result = await $ctx.$helpers.$rateLimit.byUser({
  maxRequests: 1000,
  perSeconds: 3600  // 1 hour
});

if (!result.allowed) {
  $ctx.$throw['429']('API rate limit exceeded');
  return;
}
```

### Skip Rate Limit for Admins

```javascript
// Only apply rate limit to non-admin users
if (!$ctx.$user?.isRootAdmin) {
  const result = await $ctx.$helpers.$rateLimit.byIp({
    maxRequests: 100,
    perSeconds: 60
  });

  if (!result.allowed) {
    $ctx.$throw['429']('Rate limit exceeded');
    return;
  }
}
```

### Custom Rate Limit Key

```javascript
// Rate limit per specific resource
const resourceId = $ctx.$params.id;
const result = await $ctx.$helpers.$rateLimit.check(
  `resource:${resourceId}:${$ctx.$user?.id || $ctx.$req.ip}`,
  { maxRequests: 10, perSeconds: 60 }
);

if (!result.allowed) {
  $ctx.$throw['429']('Too many requests to this resource');
  return;
}
```

## Next Steps

- See [postHooks](./posthooks.md) for post-handler operations
- Learn about [Custom Handlers](./custom-handlers.md) for custom business logic
- Check [Common Patterns](./patterns.md) for best practices


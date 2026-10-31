# Hooks and Handlers - preHooks

preHooks execute before the handler. Use them for validation, data transformation, and permission checks.

## Return early or reject the request

A pre-hook may use `return` to skip only its own remaining code. Its return value is ignored: later pre-hooks and the handler continue. Returning `{ error: ... }`, `false`, or `null` does not reject the request or become the response.

To stop the request before the handler, throw with `@THROW400('Invalid input')`, `@THROW.http(statusCode, message)`, or `@THROW.json(body, { statusCode, headers })`. A throw skips later pre-hooks and the handler, but matching post-hooks still run with `@ERROR` populated. Post-hooks cannot turn that rejection into a successful response. Cancellation, execution deadlines and terminal stream failures use the [opt-in audit path](./posthooks.md) instead of replaying ordinary hooks.

## Compose ordered decisions

Split independent responsibilities into named hooks. Select the intended HTTP methods and assign distinct `priority` values: lower numbers run first. Store shared decisions in `@SHARE`, not in a hook's return value. An unrelated hook should return immediately before doing I/O.

For example, attach these two hooks to the same `POST` operation:

| Priority | Hook | Responsibility |
|---|---|---|
| `10` | Default delivery | Initialize a server-owned decision |
| `20` | Express delivery | Refine the decision only when applicable |

Default delivery:

```javascript
@SHARE.deliveryDecision = { mode: 'standard', requiresSignature: false };
```

Express delivery:

```javascript
if (@BODY.delivery !== 'express') return;
@SHARE.deliveryDecision = { mode: 'express', requiresSignature: true };
```

The handler consumes the final decision:

```javascript
const decision = @SHARE.deliveryDecision;
return { delivery: decision.mode, requiresSignature: decision.requiresSignature };
```

If no specialized hook matches, the handler uses the default. Keep endpoint URLs, credentials and authorization decisions server-owned; never accept client overrides for private routing. Do not expose the complete shared context in a public response.

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
  @THROW400('Email is required');
  return;
}

if (!$ctx.$body.password) {
  @THROW400('Password is required');
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
  $ctx.$body.slug = $ctx.$helpers.autoSlug($ctx.$body.title);
}

// Add computed fields
$ctx.$body.createdBy = $ctx.$user.id;
$ctx.$body.createdAt = new Date();
```

## Permission Checking

```javascript
// Check authentication
if (!$ctx.$user) {
  @THROW401('Authentication required');
  return;
}

// Check role
if ($ctx.$user.role !== 'admin') {
  @THROW403('Admin access required');
  return;
}

// Check resource ownership
const resource = await $ctx.$repos.resources.find({
  filter: { id: { _eq: $ctx.$params.id } }
});

if (resource.data.length === 0) {
  @THROW404('Resource not found');
  return;
}

if (resource.data[0].userId !== $ctx.$user.id) {
  @THROW403('Access denied');
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
  @THROW429(`Rate limit exceeded. Try again in ${result.retryAfter}s`);
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
  @THROW429(`Too many login attempts. Try again in ${result.retryAfter}s`);
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
  @THROW429('API rate limit exceeded');
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
    @THROW429('Rate limit exceeded');
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
  @THROW429('Too many requests to this resource');
  return;
}
```

## Next Steps

- See [postHooks](./posthooks.md) for post-handler operations
- Learn about [Custom Handlers](./custom-handlers.md) for custom business logic
- Check [Common Patterns](./patterns.md) for best practices


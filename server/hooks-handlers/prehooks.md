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

## Next Steps

- See [afterHooks](./afterhooks.md) for post-handler operations
- Learn about [Custom Handlers](./custom-handlers.md) for custom business logic
- Check [Common Patterns](./patterns.md) for best practices


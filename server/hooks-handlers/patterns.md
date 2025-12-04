# Hooks and Handlers - Common Patterns

Common patterns and best practices for working with hooks and handlers.

## Common Patterns

### Pattern 1: Validation and Transformation

```javascript
// preHook
if (!$ctx.$body.email) {
  $ctx.$throw['400']('Email is required');
  return;
}

$ctx.$body.email = $ctx.$body.email.toLowerCase().trim();
$ctx.$share.validationPassed = true;

// Handler uses normalized data automatically
```

### Pattern 2: Permission-Based Filtering

```javascript
// preHook
if ($ctx.$user.role !== 'admin') {
  // Non-admins only see their own records
  $ctx.$query.filter = {
    ...($ctx.$query.filter || {}),
    userId: { _eq: $ctx.$user.id }
  };
}

// Handler/Default CRUD uses filtered query
```

### Pattern 3: Response Enhancement

```javascript
// afterHook
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    displayName: `${item.firstName} ${item.lastName}`,
    formattedPrice: `$${item.price.toFixed(2)}`
  }));
}
```

### Pattern 4: Audit Trail

```javascript
// afterHook
if (!$ctx.$api.error) {
  await $ctx.$repos.audit_logs.create({
    data: {
      action: `${$ctx.$req.method} ${$ctx.$req.url}`,
      userId: $ctx.$user?.id,
      statusCode: $ctx.$statusCode,
      timestamp: new Date()
    }
  });
}
```

### Pattern 5: Shared Context

```javascript
// preHook
$ctx.$share.processStartTime = Date.now();
$ctx.$share.userId = $ctx.$user.id;

// afterHook
if ($ctx.$share.processStartTime) {
  const processingTime = Date.now() - $ctx.$share.processStartTime;
  $ctx.$data.processingTime = processingTime;
}
```

### Pattern 6: Error Recovery

```javascript
// afterHook
if ($ctx.$api.error) {
  // Log error
  $ctx.$logs(`Error occurred: ${$ctx.$api.error.message}`);
  
  // Create error log
  await $ctx.$repos.error_logs.create({
    data: {
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      userId: $ctx.$user?.id,
      url: $ctx.$req.url
    }
  });
  
  // Optionally send notification
  // await sendErrorNotification($ctx.$api.error);
}
```

## Best Practices

### Hook Organization

1. **Global hooks** for cross-cutting concerns (auth, logging)
2. **Route hooks** for route-specific logic
3. **Keep hooks focused** - one responsibility per hook
4. **Use descriptive names** in hook definitions

### Code Quality

1. **Validate early** in preHooks
2. **Transform data** in preHooks before handler
3. **Enhance responses** in afterHooks after handler
4. **Use shared context** to pass data between hooks
5. **Log important steps** for debugging

### Error Handling

1. **Throw errors early** in preHooks for fast failure
2. **Handle errors gracefully** in afterHooks
3. **Provide meaningful error messages**
4. **Use appropriate HTTP status codes**

### Performance

1. **Minimize database calls** - batch operations when possible
2. **Cache expensive operations** in shared context
3. **Use early returns** to avoid unnecessary processing
4. **Consider execution order** for optimal performance

## Next Steps

- See [preHooks](./prehooks.md) for pre-handler operations
- Learn about [afterHooks](./afterhooks.md) for post-handler operations
- Check [Custom Handlers](./custom-handlers.md) for custom business logic
- Learn about [Repository Methods](../repository-methods/) for database operations
- See [Context Reference](../context-reference/) for all available properties
- Check [API Lifecycle](./api-lifecycle.md) to understand execution order


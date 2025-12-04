# Hooks and Handlers - afterHooks

afterHooks execute after the handler. Use them for response transformation, audit logging, and side effects.

## When to Use afterHooks

- Transform response data
- Add computed fields to response
- Log audit trails
- Trigger side effects (emails, notifications)
- Handle errors that occurred in handler
- Add metadata to response

## Basic afterHook Example

```javascript
// Transform response
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

## Response Enhancement

```javascript
// Add metadata
if ($ctx.$data) {
  $ctx.$data.meta = {
    processedAt: new Date(),
    processedBy: $ctx.$user?.id,
    version: '1.0'
  };
}

// Add computed fields to each item
if ($ctx.$data && Array.isArray($ctx.$data.data)) {
  $ctx.$data.data = $ctx.$data.data.map(item => ({
    ...item,
    isActive: item.status === 'active',
    displayName: item.nickname || item.name
  }));
}
```

## Audit Logging

```javascript
// Log successful operations
if (!$ctx.$api.error && $ctx.$data) {
  await $ctx.$repos.audit_logs.create({
    data: {
      action: `${$ctx.$req.method} ${$ctx.$req.url}`,
      userId: $ctx.$user?.id,
      resourceId: $ctx.$params.id,
      statusCode: $ctx.$statusCode,
      timestamp: new Date()
    }
  });
}
```

## Error Handling

```javascript
// Handle errors in afterHook
if ($ctx.$api.error) {
  // Error occurred
  $ctx.$logs(`Error: ${$ctx.$api.error.message}`);
  
  // Log to audit system
  await $ctx.$repos.error_logs.create({
    data: {
      errorMessage: $ctx.$api.error.message,
      statusCode: $ctx.$api.error.statusCode,
      userId: $ctx.$user?.id,
      url: $ctx.$req.url,
      timestamp: new Date()
    }
  });
  
  // Optionally modify error response
  // (though usually you'd want to handle this in error handlers)
} else {
  // Success case
  $ctx.$logs('Operation completed successfully');
}
```

## Next Steps

- See [preHooks](./prehooks.md) for pre-handler operations
- Learn about [Custom Handlers](./custom-handlers.md) for custom business logic
- Check [Common Patterns](./patterns.md) for best practices


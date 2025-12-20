# Repository Methods - Common Patterns

Common patterns and best practices for working with repository methods.

## Common Patterns

### Pattern 1: Create and Return Full Record

```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'New Product',
    price: 99.99
  }
});

// result.data[0] contains the full record with ID, timestamps, etc.
const newProduct = result.data[0];
```

### Pattern 2: Update Based on Current Data

```javascript
// Get current record
const current = await $ctx.$repos.products.find({
  where: { id: { _eq: 123 } }
});

if (current.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

const product = current.data[0];

// Update based on current data
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    stock: (product.stock || 0) + 10  // Increment stock
  }
});
```

### Pattern 3: Conditional Update or Create

```javascript
const existing = await $ctx.$repos.products.find({
  where: { name: { _eq: 'Product Name' } }
});

if (existing.data.length > 0) {
  // Update existing
  const result = await $ctx.$repos.products.update({
    id: existing.data[0].id,
    data: {
      price: 89.99
    }
  });
} else {
  // Create new
  const result = await $ctx.$repos.products.create({
    data: {
      name: 'Product Name',
      price: 89.99
    }
  });
}
```

### Pattern 4: Delete with Validation

```javascript
// Check if record can be deleted
const product = await $ctx.$repos.products.find({
  where: { id: { _eq: 123 } }
});

if (product.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Check business rules
if (product.data[0].status === 'active') {
  $ctx.$throw['400']('Cannot delete active product');
  return;
}

// Delete
const result = await $ctx.$repos.products.delete({
  id: 123
});
```

---

## Return Value Format

All repository methods return data in a consistent format:

### Success Response

```javascript
{
  data: [
    { id: 1, name: 'Product 1', price: 99.99 },
    { id: 2, name: 'Product 2', price: 149.99 }
  ],
  meta: {                    // Only when meta parameter is provided
    totalCount: 100,
    filterCount: 2
  }
}
```

### Error Response

When an error occurs, the system throws an exception that should be caught:

```javascript
try {
  const result = await $ctx.$repos.products.find({ ... });
} catch (error) {
  // error.message contains the error description
  // Use $ctx.$throw to throw proper HTTP errors
}
```

---

## Best Practices

1. **Always check data arrays**: Repository methods return `{data: []}`, always check `data.length` before accessing `data[0]`
2. **Use fields parameter**: When you only need specific fields, use the `fields` parameter to reduce data transfer
3. **Validate before operations**: Check if records exist before updating or deleting
4. **Handle errors**: Always wrap repository calls in try-catch blocks for proper error handling
5. **Use limit for queries**: Don't fetch all records unless necessary - use appropriate limits
6. **Leverage metadata**: Use `meta` parameter to get counts without fetching all data
7. **Batch operations**: When possible, use filters to get multiple records in one query instead of multiple individual queries

## Next Steps

- Learn about the [find() method](./find.md) for querying records
- See [create(), update(), delete() methods](./create-update-delete.md)
- Check [Context Reference](../context-reference/) to understand all available properties
- See [Query Filtering](../query-filtering.md) for advanced filtering patterns
- Check [API Lifecycle](../api-lifecycle.md) to understand how repositories fit into the request flow


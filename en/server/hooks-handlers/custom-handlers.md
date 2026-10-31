# Hooks and Handlers - Custom Handlers & Hook Types

Custom handlers replace the default CRUD operation with your own business logic.

## Custom Handlers

### When to Use Custom Handlers

- Complex business logic that doesn't fit CRUD
- Multi-step operations
- External API integrations
- Custom validation beyond preHooks
- Special response formats

### Basic Custom Handler

```javascript
// Create with additional logic
const result = await $ctx.$repos.products.create({
  data: {
    name: $ctx.$body.name,
    price: $ctx.$body.price,
    createdBy: $ctx.$user.id
  }
});

// Perform additional operations
const product = result.data[0];

// Create related records
await $ctx.$repos.product_images.create({
  data: {
    productId: product.id,
    imageUrl: $ctx.$body.imageUrl
  }
});

// Return custom response
return {
  success: true,
  product: product,
  message: 'Product created successfully'
};
```

### Multi-Step Operations

```javascript
// Create order with items
const orderResult = await $ctx.$repos.orders.create({
  data: {
    customerId: $ctx.$body.customerId,
    total: 0,
    status: 'pending'
  }
});

const order = orderResult.data[0];
let total = 0;

// Create order items
for (const item of $ctx.$body.items) {
  const productResult = await $ctx.$repos.products.find({
    filter: { id: { _eq: item.productId } }
  });
  
  if (productResult.data.length === 0) {
    @THROW404(`Product ${item.productId} not found`);
    return;
  }
  
  const product = productResult.data[0];
  const itemTotal = product.price * item.quantity;
  total += itemTotal;
  
  await $ctx.$repos.order_items.create({
    data: {
      orderId: order.id,
      productId: item.productId,
      quantity: item.quantity,
      price: product.price,
      total: itemTotal
    }
  });
}

// Update order total
await $ctx.$repos.orders.update({
  id: order.id,
  data: { total: total }
});

// Return complete order
const finalOrder = await $ctx.$repos.orders.find({
  filter: { id: { _eq: order.id } }
});

return finalOrder;
```

### Default CRUD Behavior

If you don't provide a custom handler, the system automatically performs CRUD operations based on HTTP method:

- **GET**: Query records using repository `find()`
- **POST**: Create record using repository `create()`
- **PATCH**: Update record using repository `update()`
- **DELETE**: Delete record using repository `delete()`

The default CRUD uses data from:
- `$ctx.$query` for GET requests (filter, fields, limit, sort)
- `$ctx.$body` for POST/PATCH requests (data to create/update)
- `$ctx.$params.id` for PATCH/DELETE requests (record ID)

---

## Hook Types

### Global Hooks

Global hooks run on all routes.

**Configuration:**
- Route: `null` (no specific route)
- Methods: Explicit method names such as `['GET', 'POST', 'PATCH', 'DELETE']` or `['POST', 'PATCH']`; do not use an empty list to mean all methods.

**Example:**
```javascript
// Global preHook - runs on all routes for the selected methods
// Configuration: route = null, methods = ['GET', 'POST', 'PATCH', 'DELETE']

// Global preHook - runs on all routes, POST only
// Configuration: route = null, methods = ['POST']
```

### Route-Specific Hooks

Route-specific hooks run only on a specific route.

**Configuration:**
- Route: Specific route (e.g., route with path `/users`)
- Methods: Select explicit methods and attach the hook to the matching route method configurations.

**Example:**
```javascript
// Route preHook - runs on /users for the selected methods
// Configuration: route = /users route, methods = ['GET', 'POST', 'PATCH', 'DELETE']

// Route preHook - runs on /users route, POST only
// Configuration: route = /users route, methods = ['POST']
```

### Execution Order

Matching pre-hooks run in ascending `priority`, then the handler, then matching post-hooks in ascending `priority`. Global scope does not override priority. Use distinct priorities for dependent hooks and select the intended HTTP methods explicitly.

For shared decisions, initialize a default in `@SHARE` in an early hook, refine it in matching later hooks and consume it in the handler. An unrelated pre-hook may return immediately; its return value is ignored. Throw to stop the request before the handler. Post-hooks still run after ordinary pre-hook/handler errors and should check `@ERROR` before success-only effects. See [preHooks](./prehooks.md) for a complete ordered-decision example.

## Next Steps

- See [preHooks](./prehooks.md) for pre-handler operations
- Learn about [postHooks](./posthooks.md) for post-handler operations
- Check [Common Patterns](./patterns.md) for best practices

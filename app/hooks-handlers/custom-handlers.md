# Custom Handlers

Custom Handlers let you replace default CRUD operations with your own JavaScript code. Instead of basic create/read/update/delete behavior, you can write custom functions that handle complex business logic, external integrations, or specialized data processing.

**For complete request lifecycle understanding, see [API Lifecycle](../../server/api-lifecycle.md)**

> **Template Syntax Note**: All examples use the traditional `$ctx.$property` syntax, but you can also use the shorter template syntax (`@BODY`, `@REPOS`, `#table_name`). See [Template Syntax Guide](../../server/template-syntax.md) for details. Both work identically and can be mixed freely.

## When to Use Custom Handlers

- **Complex Business Logic**: Implement multi-step operations that span multiple tables
- **Data Validation**: Add custom validation rules beyond basic schema constraints
- **External Integrations**: Call third-party APIs, send emails, or trigger webhooks
- **Data Transformation**: Process or transform data before saving to database
- **Custom Responses**: Return specialized data formats or computed values

## Creating Custom Handlers

### Step 1: Access Handler Management
1. Navigate to **Settings  Handlers** in the sidebar
2. Click **"Create New Handler"** button

### Step 2: Configure Handler
You'll see the handler creation form with these fields:

- **Logic**: Large code editor where you write your custom JavaScript
- **Description**: Text area to document what this handler does  
- **Route**: Click the relation picker (pencil icon) to select which route this handler applies to
- **Method**: Click the relation picker (pencil icon) to select the HTTP method (GET, POST, PATCH, DELETE)

### Step 3: Link to Route and Method
- **Route Selection**: Use the relation picker to search and select the specific route
- **Method Selection**: Use the relation picker to choose from available HTTP methods
- **Unique Constraint**: Each route+method combination can only have one handler

### Step 4: Handler Execution
When a request matches the route and method, your custom handler code executes instead of the default CRUD operation.

**Important: Return Values**  
Your handler **MUST return a value** - this becomes the API response. If you don't return anything, the API will return nothing to the client.

## Handler Context ($ctx)

Your handler code receives a rich context object `$ctx` with everything you need.

**For complete context reference, see [Context Reference](../../server/context-reference/README.md)**

## Database Repository Methods

Each repository in `$ctx.$repos` provides full database access through the QueryEngine.

**For complete database operations and examples, see [Context Reference](../../server/context-reference/README.md#database-access)**

## Example Handlers

### Custom Product Creation
```javascript
// POST /products handler
// Target Tables: products, categories, audit_logs

// Validate category exists
const categoryResult = await $ctx.$repos.categories.find({
  where: { id: { _eq: $ctx.$body.categoryId } },
  fields: 'id,name' // Only fetch required fields
});

if (!categoryResult.data.length) {
  throw new Error('Category not found');
}

// Create product with auto-generated slug
const slug = await $ctx.$helpers.autoSlug($ctx.$body.name);
const productResult = await $ctx.$repos.products.create({
  data: {
  name: $ctx.$body.name,
  slug: slug,
  price: $ctx.$body.price,
  categoryId: $ctx.$body.categoryId,
  createdBy: $ctx.$user.id
});

const newProduct = productResult.data[0];

// Log the creation
await $ctx.$repos.audit_logs.create({ data: {
  action: 'product_created',
  userId: $ctx.$user.id,
  entityId: newProduct.id,
  details: JSON.stringify({ productName: newProduct.name })
}});

$ctx.$logs(`Product created: ${newProduct.name}`);

return {
  success: true,
  product: newProduct,
  logs: $ctx.$share.$logs
};
```

### User Authentication
```javascript
// POST /auth/login handler  
// Target Tables: user_definition

const { email, password } = $ctx.$body;

// Find user by email
const userResult = await $ctx.$repos.user_definition.find({
  where: { email: { _eq: email } },
  fields: 'id,email,password' // Only fetch authentication fields
});

if (!userResult.data.length) {
  throw new Error('User not found');
}

const user = userResult.data[0];

// Verify password
const validPassword = await $ctx.$helpers.$bcrypt.compare(
  password, 
  user.password
);

if (!validPassword) {
  throw new Error('Invalid password');
}

// Generate JWT token
const token = await $ctx.$helpers.$jwt(
  { userId: user.id, email: user.email },
  '7d'
);

// Update last login
await $ctx.$repos.user_definition.update({ id: user.id, data: {
  lastLoginAt: new Date()
} });

return {
  success: true,
  token: token,
  user: {
    id: user.id,
    email: user.email,
    name: user.name
  }
};
```

### Advanced Filtering with API Filtering System
```javascript
// GET /products/advanced-search handler  
// Target Tables: products, categories, users
// Demonstrates the full power of API Filtering in handlers

const result = await $ctx.$repos.products.find({
  where: {
    // Logical operators
    _or: [
      {
        _and: [
          { price: { _between: [100, 500] } },    // Price range
          { category: { name: { _contains: 'electronics' } } }, // Relation filtering
          { stock: { _gt: 0 } }                   // In stock
        ]
      },
      {
        _and: [
          { featured: { _eq: true } },            // Featured products
          { rating: { _gte: 4.5 } },             // High rating
          { reviews: { _count: { _gte: 10 } } }   // Aggregation: min 10 reviews
        ]
      }
    ],
    // Text search across multiple fields
    _or: [
      { name: { _contains: $ctx.$query.search || '' } },
      { description: { _contains: $ctx.$query.search || '' } },
      { tags: { _contains: $ctx.$query.search || '' } }
    ],
    // Exclude discontinued products
    status: { _not_in: ['discontinued', 'out-of-stock'] },
    // Only published products
    publishedAt: { _is_null: false }
  }
});

// Use aggregation filtering to get categories with products count
const categoriesWithCount = await $ctx.$repos.categories.find({
  where: {
    products: { _count: { _gt: 0 } }  // Categories that have products
  },
  fields: 'id,name,description' // Only fetch essential category info
});

return {
  products: result.data,
  categories: categoriesWithCount.data,
  meta: {
    ...result.meta,
    searchTerm: $ctx.$query.search,
    appliedFilters: 'advanced-filtering-demo'
  }
};
```

### Complex Data Processing
```javascript
// GET /reports/sales handler
// Target Tables: orders, products, users

const { startDate, endDate, category } = $ctx.$query;

// Advanced filtering using API Filtering operators
const ordersResult = await $ctx.$repos.orders.find({
  where: {
    _and: [
      // Date range
      { createdAt: { _between: [startDate, endDate] } },
      // Only completed orders
      { status: { _in: ['completed', 'shipped', 'delivered'] } },
      // Filter by category if provided
      ...(category ? [{ 
        product: { 
          category: { name: { _eq: category } } 
        } 
      }] : []),
      // Minimum order value
      { total: { _gte: 10 } }
    ]
  }
});

// Get high-value customers using aggregation
const vipCustomers = await $ctx.$repos.users.find({
  where: {
    orders: {
      _sum: { total: { _gte: 1000 } }  // Customers with $1000+ total orders
    }
  }
});

return {
  summary: {
    totalRevenue: ordersResult.data.reduce((sum, order) => sum + order.total, 0),
    totalOrders: ordersResult.data.length,
    vipCustomerCount: vipCustomers.data.length
  },
  orders: ordersResult.data,
  meta: ordersResult.meta
};
```

## Best Practices

### Handler Creation
- **Access via Settings  Handlers**: Use the dedicated handler management interface
- **Relation Pickers**: Use the pencil icons to properly link routes and methods
- **Unique Combinations**: Remember each route+method can only have one handler
- **Return Values**: Always return something - this becomes the API response

### Route Configuration  
- **Target Tables**: Configure tables in the route's Target Tables field to access them via `$ctx.$repos`
- **Descriptive Names**: Use clear handler descriptions for maintenance
- **Method Specificity**: Create separate handlers for different HTTP methods

### Code Patterns
- **Always Await**: All `$ctx.$helpers` and `$ctx.$cache` functions require `await` due to IPC bridge
- **Extract Results**: Store helper results in variables before using them
- **Check Data Arrays**: Repository methods return `{data: []}`, always check `data.length`

### Error Handling
- **Use $throw Methods**: Use `$ctx.$throw['400']()` instead of `throw new Error()` for consistent error handling
- **HTTP Status Codes**: Use numeric methods like `$ctx.$throw['404']()` for standard HTTP errors
- **Semantic Errors**: Use descriptive methods like `$ctx.$throw.businessLogic()` for business logic errors
- **Error Recovery**: Check `$ctx.$api.error` in afterHook to handle errors gracefully (only available in afterHook)
- **Descriptive Messages**: Provide clear error messages and details for debugging
- **Status Codes**: Errors automatically return appropriate HTTP status codes

### Performance
- **Minimal Queries**: Only fetch data you need
- **Batch Operations**: Use bulk operations for multiple records
- **Avoid N+1**: Be mindful of queries in loops

### Security
- **User Context**: Always check `$ctx.$user` for authentication
- **Input Validation**: Validate all `$ctx.$body` and `$ctx.$query` data
- **Sanitization**: Clean user input before database operations

### Logging
- **Debug Information**: Use `$ctx.$logs()` for troubleshooting
- **Audit Trails**: Log important business operations
- **Performance Tracking**: Log execution times for optimization

**For complete best practices including cache operations and API filtering, see [Context Reference](../../server/context-reference/README.md#best-practices)**

Custom Handlers provide unlimited flexibility while maintaining security through isolated execution and rich context access.

## Related Documentation

- **[Context Reference](../../server/context-reference/README.md)** - Complete $ctx object reference
- **[File Handling](../../server/file-handling.md)** - File upload and response streaming guide
- **[Hooks](./hooks.md)** - Lightweight request/response hooks for simple customizations
- **[Hooks and Handlers](../../server/hooks-handlers/)** - Advanced hook programming with examples
- **[API Lifecycle](../../server/api-lifecycle.md)** - Complete request processing pipeline
- **[Routing Management](../routing-management.md)** - UI guide for creating custom endpoints

## Practical Examples

- **[User Registration Example](../../examples/user-registration-example.md)** - Complete walkthrough of creating `/register` endpoint with validation, password hashing, and welcome emails


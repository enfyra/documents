# Custom Handlers

Custom Handlers let you replace default CRUD operations with your own JavaScript code. Instead of basic create/read/update/delete behavior, you can write custom functions that handle complex business logic, external integrations, or specialized data processing.

## When to Use Custom Handlers

- **Complex Business Logic**: Implement multi-step operations that span multiple tables
- **Data Validation**: Add custom validation rules beyond basic schema constraints
- **External Integrations**: Call third-party APIs, send emails, or trigger webhooks
- **Data Transformation**: Process or transform data before saving to database
- **Custom Responses**: Return specialized data formats or computed values

## Creating Custom Handlers

### Step 1: Access Handler Management
1. Navigate to **Settings → Handlers** in the sidebar
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

**⚠️ Important: Return Values**
Your handler **MUST return a value** - this becomes the API response. If you don't return anything, the API will return nothing to the client.

## Handler Context ($ctx)

Your handler code receives a rich context object `$ctx` with everything you need:

### Request Data
```javascript
$ctx.$body        // Request body (POST/PATCH data)
$ctx.$params      // URL parameters (e.g., :id from /products/:id)
$ctx.$query       // Query string parameters (e.g., ?page=1&limit=10)
$ctx.$user        // Current authenticated user information
```

### Request Information
```javascript
$ctx.$req.method  // HTTP method (GET, POST, PATCH, DELETE)
$ctx.$req.url     // Full request URL
$ctx.$req.ip      // Client IP address
$ctx.$req.headers // Request headers (authorization, user-agent)
```

### Database Access
```javascript
$ctx.$repos       // Access to table repositories
// Use the table names you configured in targetTables
$ctx.$repos.products    // If "products" is in targetTables
$ctx.$repos.categories  // If "categories" is in targetTables  
$ctx.$repos.users       // If "users" is in targetTables
```

**Important**: Configure the tables you need in the route's **Target Tables** field (see [Routing Management](./routing-management.md) for route configuration). Each target table becomes available as a repository in `$ctx.$repos`.

### Utility Functions
**⚠️ Important: All helper functions require `await`** - They use IPC bridge for execution:

```javascript
// JWT token generation (async)
const token = await $ctx.$helpers.$jwt(payload, expiration)
// Example: const token = await $ctx.$helpers.$jwt({userId: 123}, '7d')

// Password hashing and verification (async)  
const hash = await $ctx.$helpers.$bcrypt.hash(plainPassword)
const isValid = await $ctx.$helpers.$bcrypt.compare(plainPassword, hashedPassword)

// URL-friendly slug generation (async)
const slug = await $ctx.$helpers.autoSlug(text)
// Example: const slug = await $ctx.$helpers.autoSlug('My Product Name') → 'my-product-name'
```

### Logging System
Enfyra provides a built-in logging system for handlers that helps with debugging and audit trails:

```javascript
// Basic logging
$ctx.$logs('Debug message', data)
$ctx.$logs('User created:', user)
$ctx.$logs('Processing order:', { orderId: 123, total: 99.99 })

// Logs are automatically collected and timestamped
// Can be viewed in the admin interface for debugging
```

**Log Types and Usage:**
```javascript
// Information logging
$ctx.$logs('Handler started', { method: $ctx.$req.method, url: $ctx.$req.url })

// Data processing logs
$ctx.$logs('Validating input:', $ctx.$body)
$ctx.$logs('Database query result:', { count: result.data.length })

// Business logic logs
$ctx.$logs('User permission check:', { userId: $ctx.$user.id, hasAccess: true })
$ctx.$logs('External API call:', { endpoint: '/payments', status: 'success' })

// Error context (before throwing)
$ctx.$logs('Validation failed:', { errors: ['Email required', 'Password too short'] })
```

### Automatic Log Response
**Enfyra automatically includes logs in API responses when logs exist** - you don't need to manually return them:

```javascript
// Your handler code
$ctx.$logs('Processing user registration')
$ctx.$logs('Validating email:', $ctx.$body.email)

const user = await $ctx.$repos.users.create({
  email: $ctx.$body.email,
  name: $ctx.$body.name
})

$ctx.$logs('User created successfully:', { id: user.id })

// Just return your data - logs are added automatically
return {
  success: true,
  user: user.data[0]
}
```

**Actual API Response (automatic):**
```json
{
  "success": true,
  "user": {
    "id": 123,
    "email": "user@example.com", 
    "name": "John Doe"
  },
  "logs": [
    "Processing user registration",
    "Validating email:",
    "user@example.com",
    "User created successfully:",
    { "id": 123 }
  ]
}
```

### Client-Side Debugging Benefits
The automatic logs inclusion allows for **client-side debugging without server access**:

```javascript
// Frontend code - no server log access needed
const response = await fetch('/api/users', {
  method: 'POST',
  body: JSON.stringify({ email: 'user@example.com', name: 'John' })
})

const data = await response.json()

// Debug directly from API response
if (data.logs) {
  console.log('Handler execution logs:', data.logs)
  // See exactly what happened in the handler without checking server logs!
}
```

**When logs appear in response:**
- ✅ **With logs**: `{ result: {...}, logs: [...] }` - when `$ctx.$logs()` was called
- ✅ **Without logs**: `{ result: {...} }` - when no logging occurred

### Log Structure
Based on the backend implementation, logs are stored as raw arguments passed to `$ctx.$logs()`:

```javascript
// When you call:
$ctx.$logs('User created:', { id: 123, email: 'user@example.com' })
$ctx.$logs('Processing complete')
$ctx.$logs('Debug data:', someObject, 'additional info')

// $ctx.$share.$logs contains:
[
  'User created:',
  { id: 123, email: 'user@example.com' },
  'Processing complete', 
  'Debug data:',
  someObject,
  'additional info'
]

// All arguments are flattened into a single array
```

### Best Practices for Logging
```javascript
// ✅ Good: Log important operations
$ctx.$logs('Starting user registration', { email: $ctx.$body.email })
$ctx.$logs('Email validation passed')
$ctx.$logs('User created successfully', { userId: newUser.id })

// ✅ Good: Log before database operations
$ctx.$logs('Creating user record:', { email, name, role })
const user = await $ctx.$repos.users.create(userData)
$ctx.$logs('User record created:', { id: user.id })

// ✅ Good: Log external API calls
$ctx.$logs('Calling payment API:', { amount, currency })
const payment = await callPaymentAPI(paymentData)
$ctx.$logs('Payment API response:', { transactionId: payment.id, status: payment.status })

// ❌ Avoid: Logging sensitive data
$ctx.$logs('User login attempt:', { password: $ctx.$body.password }) // Don't do this!

// ✅ Better: Log without sensitive info
$ctx.$logs('User login attempt:', { email: $ctx.$body.email, hasPassword: !!$ctx.$body.password })
```

## Database Repository Methods

Each repository in `$ctx.$repos` provides full database access through the QueryEngine:

### Basic Operations
```javascript
// Find records (returns {data: [], meta: {totalCount, filterCount}})
const result = await $ctx.$repos.products.find({
  where: { category: { _eq: 'electronics' } }
});
const products = result.data;

// Create new record (returns query result with created record)
const result = await $ctx.$repos.products.create({
  name: $ctx.$body.name,
  price: $ctx.$body.price,
  category: $ctx.$body.category
});
const newProduct = result.data[0];

// Update record by ID (returns query result with updated record)
const result = await $ctx.$repos.products.update($ctx.$params.id, {
  price: $ctx.$body.price
});
const updatedProduct = result.data[0];

// Delete record by ID (returns success message)
const result = await $ctx.$repos.products.delete($ctx.$params.id);
// Returns: { message: 'Delete successfully!', statusCode: 200 }
```

### Query Filtering
The `where` field in the `find` method uses **Enfyra's API Filtering system** - the same MongoDB-like operators available in the API:

```javascript
// Find with complex filters using API Filtering operators
const result = await $ctx.$repos.products.find({
  where: {
    price: { _gte: 100, _lte: 500 },        // price >= 100 AND <= 500
    category: { _in: ['electronics', 'gadgets'] }, // category IN (...)
    name: { _contains: 'phone' },           // name CONTAINS 'phone' (case-insensitive)
    isActive: { _eq: true },                // isActive = true
    description: { _is_null: false }        // description IS NOT NULL
  }
});
```

**🔥 Full API Filtering Power**: The `where` parameter supports **all** operators from [API Filtering](../api/api-filtering.md):
- **Comparison**: `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_between`
- **Arrays**: `_in`, `_not_in` 
- **Text Search**: `_contains`, `_starts_with`, `_ends_with`
- **Null Checks**: `_is_null`
- **Logical**: `_and`, `_or`, `_not`
- **Relations**: Filter by related table data
- **Aggregations**: `_count`, `_sum`, `_avg`, `_min`, `_max`

### Query Parameters Integration
The repository automatically uses query parameters from the request:
```javascript
// These query params are automatically applied to find():
// ?fields=name,price&limit=20&page=2&sort=-createdAt&meta=*

const result = await $ctx.$repos.products.find({
  where: { category: { _eq: 'electronics' } }
});

// Equivalent to:
// - SELECT name,price FROM products 
// - WHERE category = 'electronics'
// - ORDER BY createdAt DESC
// - LIMIT 20 OFFSET 20
// - Return meta information (totalCount, etc.)
```

### Default Filter Behavior
If you **don't provide a `where` parameter**, the repository automatically uses `$ctx.$query.filter`:

```javascript
// Without custom where - uses query filter automatically
const result = await $ctx.$repos.products.find({});
// This automatically applies ?filter={"price":{"_gte":100}} from URL

// Same as explicitly using query filter
const result = await $ctx.$repos.products.find({
  where: $ctx.$query.filter || {}
});
```

### Modifying Query Filters
You can **modify the filter before using it** by changing `$ctx.$query.filter`:

```javascript
// Original query: ?filter={"category":"electronics"}
// Add additional conditions to user's filter

// Get existing filter or empty object
const baseFilter = $ctx.$query.filter || {};

// Add your custom conditions
$ctx.$query.filter = {
  _and: [
    baseFilter,                           // Keep user's original filter
    { isActive: { _eq: true } },         // Add: only active products
    { stock: { _gt: 0 } }                // Add: only in-stock products
  ]
};

// Now find() will use the modified filter
const result = await $ctx.$repos.products.find({});
```

### Override vs Extend Filters
```javascript
// Option 1: Override completely (ignore user filter)
const result = await $ctx.$repos.products.find({
  where: { price: { _gte: 100 } }  // Only this condition
});

// Option 2: Extend user filter (recommended)
$ctx.$query.filter = {
  _and: [
    $ctx.$query.filter || {},      // User's filter
    { price: { _gte: 100 } }       // Your additional filter
  ]
};
const result = await $ctx.$repos.products.find({});
```

## Example Handlers

### Custom Product Creation
```javascript
// POST /api/products handler
// Target Tables: products, categories, audit_logs

// Validate category exists
const categoryResult = await $ctx.$repos.categories.find({
  where: { id: { _eq: $ctx.$body.categoryId } }
});

if (!categoryResult.data.length) {
  throw new Error('Category not found');
}

// Create product with auto-generated slug
const slug = await $ctx.$helpers.autoSlug($ctx.$body.name);
const productResult = await $ctx.$repos.products.create({
  name: $ctx.$body.name,
  slug: slug,
  price: $ctx.$body.price,
  categoryId: $ctx.$body.categoryId,
  createdBy: $ctx.$user.id
});

const newProduct = productResult.data[0];

// Log the creation
await $ctx.$repos.audit_logs.create({
  action: 'product_created',
  userId: $ctx.$user.id,
  entityId: newProduct.id,
  details: JSON.stringify({ productName: newProduct.name })
});

$ctx.$logs('Product created:', newProduct.name);

return {
  success: true,
  product: newProduct,
  logs: $ctx.$share.$logs
};
```

### User Authentication
```javascript
// POST /api/auth/login handler  
// Target Tables: users

const { email, password } = $ctx.$body;

// Find user by email
const userResult = await $ctx.$repos.users.find({
  where: { email: { _eq: email } }
});

if (!userResult.data.length) {
  throw new Error('User not found');
}

const user = userResult.data[0];

// Verify password
const validPassword = await $ctx.$helpers.$bcrypt.compare(
  password, 
  user.hashedPassword
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
await $ctx.$repos.users.update(user.id, {
  lastLoginAt: new Date()
});

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
// GET /api/products/advanced-search handler  
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
  }
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
// GET /api/reports/sales handler
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
- **Access via Settings → Handlers**: Use the dedicated handler management interface
- **Relation Pickers**: Use the pencil icons to properly link routes and methods
- **Unique Combinations**: Remember each route+method can only have one handler
- **Return Values**: Always return something - this becomes the API response

### Route Configuration  
- **Target Tables**: Configure tables in the route's Target Tables field to access them via `$ctx.$repos`
- **Descriptive Names**: Use clear handler descriptions for maintenance
- **Method Specificity**: Create separate handlers for different HTTP methods

### Code Patterns
- **Always Await Helpers**: All `$ctx.$helpers` functions require `await` due to IPC bridge
- **Extract Helper Results**: Store helper results in variables before using them
- **Check Data Arrays**: Repository methods return `{data: []}`, always check `data.length`

### API Filtering Integration
- **Use Full Filter Power**: The `where` parameter in `$ctx.$repos.find()` supports the complete [API Filtering](../backend/api-filtering.md) system
- **Complex Queries**: Leverage `_and`, `_or`, `_not` for complex logic instead of multiple separate queries
- **Relation Filtering**: Filter by related table data directly in the `where` clause
- **Aggregation Queries**: Use `_count`, `_sum`, `_avg`, `_min`, `_max` for statistical filtering
- **Consistent API**: Same operators work in handlers, direct API calls, and frontend filters

### Error Handling
- **Throw Errors**: Use `throw new Error('message')` for validation failures
- **Descriptive Messages**: Provide clear error messages for debugging
- **Status Codes**: Errors automatically return appropriate HTTP status codes

### Performance
- **Minimal Queries**: Only fetch data you need
- **Batch Operations**: Use bulk operations for multiple records
- **Avoid N+1**: Be mindful of queries in loops
- **Cache Helper Results**: Don't call the same helper multiple times

### Security
- **User Context**: Always check `$ctx.$user` for authentication
- **Input Validation**: Validate all `$ctx.$body` and `$ctx.$query` data
- **Sanitization**: Clean user input before database operations

### Logging
- **Debug Information**: Use `$ctx.$logs()` for troubleshooting
- **Audit Trails**: Log important business operations
- **Performance Tracking**: Log execution times for optimization

Custom Handlers provide unlimited flexibility while maintaining security through isolated execution and rich context access.
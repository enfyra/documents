# API Querying

Enfyra provides a powerful MongoDB-like querying system for your API endpoints. This system allows developers to filter, sort, paginate, and load related data using comparison operators, text search, logical operators, relation filtering, and aggregation functions across any table in your database.

> **📝 Template Syntax Note**: All code examples showing `$ctx.$property` can also use template syntax (`@BODY`, `@REPOS`, `#table_name`) - see [Template Syntax Guide](./template-syntax.md) for details. Both syntaxes work identically and can be mixed together.

## Quick Navigation

**Essential Sections:**
- [API Lifecycle](#api-lifecycle) - Quick overview (see [full docs](./api-lifecycle.md))
- [Query Parameters](#query-parameters) - Basic params like `fields`, `limit`, `sort`, `filter`
- [Filter Operators](#filter-operators) - `_eq`, `_gt`, `_contains`, etc.
- [Logical Operators](#logical-operators) - `_and`, `_or`, `_not` combinations

**Advanced Features:**
- [Auto-Join Feature](#auto-join-feature) - Automatic JOIN creation for related data
- [Relation Filtering](#relation-filtering) - Filter by related table data
- [Aggregation Filtering](#aggregation-filtering) - `_count`, `_sum`, `_avg` operations
- [Field Selection](#field-selection) - Choose specific fields to return
- [Sorting & Pagination](#sorting) - Control data ordering and paging

**💼 Practical Usage:**
- [URL Query Examples](#url-query-examples) - REST API examples
- [Complex Examples](#complex-examples) - Real-world use cases
- [Custom Handler Integration](#api-usage-in-custom-handlers) - Use in your code

**🔗 Related APIs:**
- **[GraphQL API](./graphql-api.md)** - Complete GraphQL queries and mutations guide
- **[Swagger API](./swagger-api.md)** - Interactive API documentation and testing

**Developer Resources:**
- [Performance Tips](#performance-tips) - Optimization guidelines
- [Error Handling](#error-handling) - Validation and debugging
- [TypeScript Support](#typescript-support) - Type definitions
- [Best Practices](#best-practices) - Production-ready patterns

---

## API Lifecycle

Enfyra follows a structured request lifecycle with hooks and handlers. For complete details on the lifecycle system, context sharing, and hook execution, see **[API Lifecycle](./api-lifecycle.md)**.

### Quick Overview
```
HTTP Request → preHook(s) → Handler → afterHook(s) → Response
```

**Key Points:**
- **Context Sharing**: `$ctx` object is the same reference throughout the entire request
- **Global Hooks**: Run on ALL endpoints (`route: null`)
- **Sequential Execution**: All matching hooks run in database order
- **Persistent Changes**: Modifications in preHooks affect handlers and afterHooks

**For detailed information see:** **[→ API Lifecycle Documentation](./api-lifecycle.md)**

---

## Auto-Join Feature

The Query Engine automatically creates JOIN queries when you reference related tables. No manual JOIN configuration needed!

### Automatic JOIN on Field Selection

```http
# Select user with role name - AUTO JOIN with roles table
GET /users?fields=id,name,email,role.name,role.permissions

# Select post with author details - AUTO JOIN with users table
GET /posts?fields=title,content,author.name,author.email
```

### Automatic JOIN on Filtering

```http
# Find posts by author name - AUTO JOIN with users table
GET /posts?filter={"author":{"name":{"_contains":"john"}}}

# Find users with admin role - AUTO JOIN with roles table  
GET /users?filter={"role":{"name":{"_eq":"admin"}}}

# Complex multi-level JOIN - AUTO JOIN users -> posts -> comments
GET /users?filter={"posts":{"comments":{"approved":{"_eq":true}}}}
```

### Automatic JOIN on Sorting

```http
# Sort posts by author name - AUTO JOIN with users table
GET /posts?sort=author.name,-createdAt

# Sort orders by customer email - AUTO JOIN with customers table
GET /orders?sort=customer.email,orderDate
```

The engine optimizes these JOINs automatically:
- Uses LEFT JOIN for optional relations
- Uses INNER JOIN when filtering requires the relation to exist
- Avoids duplicate JOINs when same relation is used multiple times
- Creates proper join conditions based on foreign keys

## Query Parameters

All API endpoints support these query parameters for data filtering and manipulation:

### Basic Parameters
```
GET /api/products?fields=name,price&limit=20&page=2&sort=-createdAt&meta=*
```

- **`fields`**: Select specific fields to return (e.g., `name,price` or `name,category.name`)
- **`limit`**: Maximum number of records to return (default: 10)
- **`page`**: Page number for pagination (starts at 1)
- **`sort`**: Sort fields (prefix with `-` for descending, e.g., `-price,name`)
- **`meta`**: Include metadata (`totalCount`, `filterCount`, or `*` for all)
- **`filter`**: Complex filtering conditions (see below)

## Filter Operators

### Comparison Operators

```javascript
// Equal
{ "price": { "_eq": 100 } }

// Not equal  
{ "price": { "_neq": 100 } }

// Greater than
{ "price": { "_gt": 100 } }

// Greater than or equal
{ "price": { "_gte": 100 } }

// Less than
{ "price": { "_lt": 100 } }

// Less than or equal
{ "price": { "_lte": 100 } }

// Between (inclusive range)
{ "price": { "_between": [50, 200] } }
// or as string: { "price": { "_between": "50,200" } }
```

### Array Operators

```javascript
// In array
{ "categoryId": { "_in": [1, 2, 3] } }
// or as string: { "categoryId": { "_in": "1,2,3" } }
// or JSON string: { "categoryId": { "_in": "[1,2,3]" } }

// Not in array
{ "categoryId": { "_not_in": [1, 2, 3] } }
// Same string formats as _in
```

### Text Search Operators

```javascript
// Contains substring (case-insensitive with accent support)
{ "name": { "_contains": "phone" } }

// Starts with
{ "name": { "_starts_with": "iPhone" } }

// Ends with  
{ "name": { "_ends_with": "Pro" } }
```

### Null Operators

```javascript
// Is null
{ "description": { "_is_null": true } }

// Is not null
{ "description": { "_is_null": false } }
```

## Logical Operators

### AND Conditions (Default)
```javascript
// Multiple conditions are AND by default
{
  "price": { "_gte": 100 },
  "categoryId": { "_eq": 1 }
}
```

### Explicit AND
```javascript
{
  "_and": [
    { "price": { "_gte": 100 } },
    { "categoryId": { "_eq": 1 } }
  ]
}
```

### OR Conditions
```javascript
{
  "_or": [
    { "price": { "_lt": 50 } },
    { "price": { "_gt": 500 } }
  ]
}
```

### NOT Conditions
```javascript
{
  "_not": {
    "category": { "_eq": "electronics" }
  }
}
```

### Complex Combinations
```javascript
{
  "_or": [
    {
      "_and": [
        { "price": { "_gte": 100 } },
        { "category": { "_eq": "electronics" } }
      ]
    },
    {
      "_and": [
        { "price": { "_lt": 50 } },
        { "name": { "_contains": "sale" } }
      ]
    }
  ]
}
```

## Relation Filtering

Filter records based on their related data:

### Basic Relation Filtering
```javascript
// Find users who have posts with more than 1000 views
{
  "posts": {
    "views": { "_gt": 1000 }
  }
}
```

### Relation IN/NOT IN
```javascript
// Find users who have posts with specific IDs
{
  "posts": { "_in": [1, 5, 10] }
}

// Find users who don't have posts with specific IDs
{
  "posts": { "_not_in": [1, 5, 10] }
}
```

### Nested Relation Filtering
```javascript
// Find users whose posts have comments containing "great"
{
  "posts": {
    "comments": {
      "content": { "_contains": "great" }
    }
  }
}
```

## Aggregation Filtering

Filter based on counts and calculations of related data:

### Count Aggregation
```javascript
// Users with exactly 5 posts
{ "posts": { "_count": { "_eq": 5 } } }

// Users with more than 3 posts
{ "posts": { "_count": { "_gt": 3 } } }

// Users with between 2 and 10 posts
{ "posts": { "_count": { "_between": [2, 10] } } }
```

### Mathematical Aggregations
```javascript
// Users whose posts have average views > 1000
{
  "posts": {
    "_avg": {
      "views": { "_gt": 1000 }
    }
  }
}

// Users whose posts have total views > 10000
{
  "posts": {
    "_sum": {
      "views": { "_gt": 10000 }
    }
  }
}

// Users whose most viewed post has > 5000 views
{
  "posts": {
    "_max": {
      "views": { "_gt": 5000 }
    }
  }
}

// Users whose least viewed post has > 100 views
{
  "posts": {
    "_min": {
      "views": { "_gt": 100 }
    }
  }
}
```

## Field Selection

Control which fields are returned in the response:

### Basic Field Selection
```
?fields=name,price,category
```

### Relation Fields
```
?fields=name,price,category.name,category.description
```

### Wildcard Selection
```
?fields=*                    // All fields from main table
?fields=category.*           // All fields from category relation
?fields=name,category.*      // name + all category fields
```

## Sorting

Sort results by one or multiple fields:

### Single Field Sort
```
?sort=name          // Ascending
?sort=-price        // Descending
```

### Multiple Field Sort
```
?sort=category,-price,name   // category ASC, price DESC, name ASC
```

### Relation Field Sort
```
?sort=category.name,-price   // Sort by category name, then price descending
```

## Pagination

Control result pagination:

```
?page=1&limit=20     // First 20 results
?page=3&limit=10     // Results 21-30
```

## Metadata

Get additional information about the query results:

```
?meta=totalCount              // Total records in table
?meta=filterCount             // Records matching current filter
?meta=totalCount,filterCount  // Both counts
?meta=*                       // All available metadata
```

Example response with metadata:
```json
{
  "data": [...],
  "meta": {
    "totalCount": 1000,
    "filterCount": 25
  }
}
```

## URL Query Examples

### Simple Filtering
```http
GET /api/products?filter={"price":{"_gte":100,"_lte":500}}
```

**JavaScript Implementation:**
```javascript
// URL encoding is handled automatically by URLSearchParams
const filter = { price: { _gte: 100, _lte: 500 } };
const params = new URLSearchParams({
  filter: JSON.stringify(filter)
});
const url = `/api/products?${params.toString()}`;

// Using fetch
const response = await fetch(url);
const data = await response.json();
```

### Working with relations & cascade when calling APIs

The cascade rules you configure with `onDelete` are enforced at the database and runtime levels when you call the default REST APIs:

- **One‑to‑many / Many‑to‑one (1‑N / N‑1)**  
  - **Delete parent:**  
    ```http
    DELETE /posts/1
    ```
    - If `onDelete = CASCADE` on `post.comments`, deleting the post will delete all related comments.  
    - If `onDelete = SET NULL`, comments are kept and their `postId` is set to `NULL`.  
    - If `onDelete = RESTRICT`, the API returns an error if comments still exist.
  - **Delete child:**  
    ```http
    DELETE /comments/10
    ```
    - The parent post is never deleted; only the comment is removed.
  - **Update children through parent (detach vs keep)**  
    ```http
    PATCH /posts/1
    Content-Type: application/json

    {
      "title": "Updated post",
      "comments": [
        { "id": 10, "content": "keep this comment" },
        { "content": "new comment" }
      ]
    }
    ```
    - Existing children not present in the `comments` array are **detached**: their FK is set to `NULL` (not deleted).  
    - New objects without `id` are created and linked to the post.

- **Many‑to‑many (N‑N)**  
  - Updating relations only changes junction rows, not main table data:
    ```http
    PATCH /posts/1
    Content-Type: application/json

    {
      "tags": [1, 2, { "name": "new-tag" }]
    }
    ```
    - The system will:
      - Ensure tag `1` and `2` are linked via the junction table.  
      - Create a new `tag` if it doesn’t have an `id`, then link it.  
      - Remove any previous tag links **not included** in the new array.  
    - Tag records themselves are never deleted when you remove them from the `tags` array.

- **One‑to‑one (1‑1) with `onDelete = CASCADE`**  
  - When a 1‑1 relation is configured with `onDelete = CASCADE`, deleting either side via API will delete the paired record as well:
    ```http
    # User ↔ Profile (1‑1, CASCADE)
    DELETE /users/1      # deletes user + profile
    DELETE /profiles/1   # deletes profile + user
    ```
  - This behavior is enforced by Enfyra’s runtime hooks, even if only one side has the physical foreign key.


### Complex Filtering with Pagination
```http
GET /api/users?filter={"_or":[{"age":{"_lt":25}},{"posts":{"_count":{"_gte":5}}}]}&page=2&limit=10&sort=-createdAt
```

**JavaScript Implementation:**
```javascript
const buildQueryUrl = (baseUrl, options = {}) => {
  const params = new URLSearchParams();
  
  if (options.filter) {
    params.append('filter', JSON.stringify(options.filter));
  }
  if (options.fields) {
    params.append('fields', Array.isArray(options.fields) ? options.fields.join(',') : options.fields);
  }
  if (options.sort) {
    params.append('sort', Array.isArray(options.sort) ? options.sort.join(',') : options.sort);
  }
  if (options.page) params.append('page', options.page);
  if (options.limit) params.append('limit', options.limit);
  if (options.meta) params.append('meta', options.meta);
  
  return `${baseUrl}?${params.toString()}`;
};

// Usage
const url = buildQueryUrl('/api/users', {
  filter: {
    _or: [
      { age: { _lt: 25 } },
      { posts: { _count: { _gte: 5 } } }
    ]
  },
  page: 2,
  limit: 10,
  sort: '-createdAt'
});
```

### Field Selection with Relations
```http
GET /api/products?fields=name,price,category.name&filter={"category":{"name":{"_contains":"electronics"}}}
```

**Frontend Integration Example:**
```javascript
// React hook for API querying
const useProducts = (filters = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchProducts = async () => {
      setLoading(true);
      try {
        const url = buildQueryUrl('/api/products', {
          fields: 'name,price,category.name',
          filter: {
            category: { name: { _contains: 'electronics' } },
            ...filters
          }
        });
        
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch');
        
        const result = await response.json();
        setData(result.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchProducts();
  }, [filters]);

  return { data, loading, error };
};
```

## API Usage in Custom Handlers

When using the filter system in [Custom Handlers](../frontend/custom-handlers.md), you can access and utilize these same filtering capabilities:

```javascript
// In a custom handler
const result = await $ctx.$repos.products.find({
  where: {
    price: { _between: [100, 500] },
    category: { 
      name: { _contains: 'electronics' } 
    }
  }
});

// The repository automatically applies query parameters:
// ?fields=name,price&limit=20&sort=-createdAt
// These are automatically used by the find() method
```

The API filtering system integrates seamlessly with:
- **Default CRUD operations** on [routes](../frontend/routing-management.md)
- **Custom handlers** with full filter access
- **Frontend applications** consuming the API
- **Third-party integrations** and webhooks

## Complex Examples

### Example 1: E-commerce Product Search

Find active products in specific categories with good ratings:

```javascript
// Find active products in specific categories with good ratings
const products = await queryEngine.find({
  tableName: 'products',
  filter: {
    _and: [
      { status: { _eq: 'active' } },
      { category: { _in: ['electronics', 'computers'] } },
      { price: { _between: [100, 1000] } },
      {
        reviews: {
          _avg: {
            rating: { _gte: 4.0 }
          }
        }
      },
      {
        reviews: {
          _count: { _gte: 10 }  // At least 10 reviews
        }
      }
    ]
  },
  fields: 'id,name,price,category',
  sort: ['-reviews.rating', 'price'],
  page: 1,
  limit: 20,
  meta: 'totalCount,filterCount'
});
```

### Example 2: Blog Post Query with Author

Find recent published posts with their authors:

```javascript
// Find recent published posts with their authors
const posts = await queryEngine.find({
  tableName: 'posts',
  filter: {
    _and: [
      { status: { _eq: 'published' } },
      { publishedAt: { _lte: new Date() } },
      {
        _or: [
          { tags: { _contains: 'javascript' } },
          { tags: { _contains: 'typescript' } }
        ]
      }
    ]
  },
  fields: 'id,title,excerpt,publishedAt,author.name,author.avatar',
  sort: ['-publishedAt'],
  limit: 10
});
```

### Example 3: User Activity Report

Find active users with recent activity:

```javascript
// Find active users with recent activity
const activeUsers = await queryEngine.find({
  tableName: 'users',
  filter: {
    _and: [
      { status: { _eq: 'active' } },
      { lastLoginAt: { _gte: '2024-01-01' } },
      {
        _or: [
          {
            posts: {
              _count: { _gte: 5 }
            }
          },
          {
            comments: {
              _count: { _gte: 20 }
            }
          }
        ]
      }
    ]
  },
  fields: 'id,name,email,lastLoginAt',
  sort: ['-lastLoginAt'],
  limit: 50,
  meta: '*'  // Include all meta information
});
```

## Performance Tips

### Database Optimization
1. **Add Indexes**: Create indexes for frequently filtered fields
```sql
-- Add indexes for common filters
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_posts_published_at ON posts(published_at);

-- Composite indexes for complex filters  
CREATE INDEX idx_users_status_created ON users(status, created_at);
```

2. **Use Field Selection**: Only select fields you need
```javascript
// ❌ Over-fetching
GET /api/users?fields=*

// ✅ Specific fields only
GET /api/users?fields=id,name,email
```

### Query Optimization
3. **Use Auto-Join for Related Data**: Use auto-join to fetch related data efficiently
```javascript
// ✅ Efficient: Single JOIN query
GET /api/posts?fields=title,author.name&filter={"status":"published"}
```

4. **Limit Results**: Always use pagination for large datasets
```javascript
// ✅ Always set limits
GET /api/users?limit=20&page=1

// ❌ Dangerous: Could return millions of records
GET /api/users?limit=0
```

5. **Filter Early**: Apply most selective filters first
```javascript
// ✅ Most selective filter first
{
  "_and": [
    { "id": { "_in": [1,2,3] } },          // Very selective
    { "status": { "_eq": "active" } },      // Less selective
    { "name": { "_contains": "john" } }      // Least selective
  ]
}
```

### Caching Strategy
6. **Implement Response Caching**:
```javascript
// Cache static/slow-changing data
const CACHE_DURATION = 5 * 60 * 1000; // 5 minutes
const cache = new Map();

const cachedQuery = async (url) => {
  const cached = cache.get(url);
  if (cached && Date.now() - cached.timestamp < CACHE_DURATION) {
    return cached.data;
  }
  
  const data = await apiQuery(url);
  cache.set(url, { data, timestamp: Date.now() });
  return data;
};
```

7. **Monitor Query Performance**:
```javascript  
// Track slow queries
const performanceQuery = async (url, options) => {
  const startTime = Date.now();
  try {
    const result = await apiQuery(url, options);
    const duration = Date.now() - startTime;
    
    if (duration > 1000) {
      console.warn(`Slow query detected: ${url} took ${duration}ms`);
    }
    
    return result;
  } catch (error) {
    console.error(`Query failed: ${url}`, error);
    throw error;
  }
};
```

## Error Handling

The Query Engine validates input and provides structured error responses:

### Common Error Types

```javascript
// ❌ Invalid operator
GET /api/users?filter={"age":{"_invalid":18}}
// Response: 400 Bad Request
{
  "error": "Invalid operator '_invalid' for field 'age'",
  "code": "INVALID_OPERATOR", 
  "field": "age",
  "operator": "_invalid"
}

// ❌ Invalid field
GET /api/users?filter={"nonexistent":{"_eq":"value"}}
// Response: 400 Bad Request  
{
  "error": "Field 'nonexistent' does not exist in table 'users'",
  "code": "INVALID_FIELD",
  "field": "nonexistent",
  "table": "users"
}

// ❌ Invalid relation
GET /api/users?filter={"invalidRelation":{"name":"john"}}
// Response: 400 Bad Request
{
  "error": "Relation 'invalidRelation' not found in table 'users'", 
  "code": "INVALID_RELATION",
  "relation": "invalidRelation",
  "table": "users"
}
```

### Production Error Handling

```javascript
const apiQuery = async (endpoint, options = {}) => {
  try {
    const url = buildQueryUrl(endpoint, options);
    const response = await fetch(url);
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new APIError(errorData.error, errorData.code, errorData);
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof APIError) {
      // Handle specific API errors
      switch (error.code) {
        case 'INVALID_FIELD':
          console.warn(`Field ${error.details.field} not available in ${error.details.table}`);
          break;
        case 'INVALID_OPERATOR':
          console.warn(`Operator ${error.details.operator} not supported for field ${error.details.field}`);
          break;
        case 'INVALID_RELATION':
          console.warn(`Relation ${error.details.relation} not found`);
          break;
        default:
          console.error('API Error:', error.message);
      }
      throw error;
    } else {
      // Handle network/parsing errors
      console.error('Network Error:', error.message);
      throw new Error('Failed to fetch data');
    }
  }
};

class APIError extends Error {
  constructor(message, code, details) {
    super(message);
    this.name = 'APIError';
    this.code = code;
    this.details = details;
  }
}

// Usage with error handling
const fetchUsers = async (filters) => {
  try {
    const result = await apiQuery('/api/users', {
      filter: filters,
      fields: 'id,name,email',
      limit: 20
    });
    return result.data;
  } catch (error) {
    if (error.code === 'INVALID_FIELD') {
      // Fallback to basic fields
      return await apiQuery('/api/users', {
        filter: filters,
        fields: 'id,name', // Remove invalid field
        limit: 20
      });
    }
    throw error;
  }
};
```

## TypeScript Support

The Query Engine is fully typed. Example interfaces:

```typescript
interface QueryOptions {
  tableName: string;
  filter?: FilterCondition;
  fields?: string | string[];
  sort?: string[];
  page?: number;
  limit?: number;
  meta?: string;
}

interface FilterCondition {
  [field: string]:
    | any // Direct value (implicit _eq)
    | { [operator: string]: any }
    | FilterCondition; // Nested conditions
}
```

## Best Practices

### Performance Optimization
- **Use field selection**: Only request fields you need
- **Apply filters early**: Use specific filters to reduce data transfer
- **Use auto-join**: Leverage auto-join feature for related data
- **Use pagination**: Don't load all records at once

### Filter Design
- **Combine operators**: Use `_and`/`_or` for complex conditions
- **Index key fields**: Ensure filtered fields are indexed in your database
- **Text search optimization**: Use `_starts_with` instead of `_contains` when possible
- **Relation filtering**: Filter at the source rather than post-processing

### Query Structure
```javascript
// Good: Specific and efficient
{
  "filter": {
    "_and": [
      { "status": { "_eq": "active" } },
      { "price": { "_gte": 100 } }
    ]
  },
  "fields": "name,price,status",
  "limit": 20,
  "sort": ["-updatedAt"]
}

// Avoid: Too broad and inefficient
{
  "fields": "*"
}
```

The API filtering system provides MongoDB-like querying power while maintaining SQL database performance and type safety through Enfyra's query engine.
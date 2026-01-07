# Query Filtering

Enfyra provides MongoDB-like filtering operators for querying data. Use these operators in the `where` parameter of repository `find()` calls or in URL query strings.

## Quick Navigation

- [Comparison Operators](#comparison-operators) - `_eq`, `_gt`, `_lt`, etc.
- [Array Operators](#array-operators) - `_in`, `_not_in`
- [Text Search](#text-search) - `_contains`, `_starts_with`, `_ends_with`
- [Null Checks](#null-checks) - `_is_null`, `_is_not_null`
- [Logical Operators](#logical-operators) - `_and`, `_or`, `_not`
- [Range Operators](#range-operators) - `_between`
- [Complex Examples](#complex-examples) - Real-world patterns

## Comparison Operators

### _eq (Equal)

Match exact value.

```javascript
// Find products with price = 100
const result = await $ctx.$repos.products.find({
  where: { price: { _eq: 100 } }
});

// URL: /products?filter={"price":{"_eq":100}}
```

### _neq (Not Equal)

Exclude matching value.

```javascript
// Find products where price != 0
const result = await $ctx.$repos.products.find({
  where: { price: { _neq: 0 } }
});
```

### _gt (Greater Than)

Match values greater than.

```javascript
// Find products with price > 100
const result = await $ctx.$repos.products.find({
  where: { price: { _gt: 100 } }
});
```

### _gte (Greater Than or Equal)

Match values greater than or equal to.

```javascript
// Find products with price >= 100
const result = await $ctx.$repos.products.find({
  where: { price: { _gte: 100 } }
});
```

### _lt (Less Than)

Match values less than.

```javascript
// Find products with price < 500
const result = await $ctx.$repos.products.find({
  where: { price: { _lt: 500 } }
});
```

### _lte (Less Than or Equal)

Match values less than or equal to.

```javascript
// Find products with price <= 500
const result = await $ctx.$repos.products.find({
  where: { price: { _lte: 500 } }
});
```

## Array Operators

### _in (In Array)

Match any value in the array.

```javascript
// Find products in specific categories
const result = await $ctx.$repos.products.find({
  where: {
    category: { _in: ['electronics', 'gadgets', 'phones'] }
  }
});

// URL: /products?filter={"category":{"_in":["electronics","gadgets"]}}
```

### _not_in (Not In Array)

Exclude values in the array.

```javascript
// Find products not in these categories
const result = await $ctx.$repos.products.find({
  where: {
    category: { _not_in: ['discontinued', 'old'] }
  }
});
```

## Text Search

Text search operators are case-insensitive.

### _contains (Contains Text)

Match fields containing the text.

```javascript
// Find products with name containing "phone"
const result = await $ctx.$repos.products.find({
  where: {
    name: { _contains: 'phone' }
  }
});

// Matches: "iPhone", "Phone Case", "Smartphone"
```

### _starts_with (Starts With Text)

Match fields starting with the text.

```javascript
// Find products starting with "Apple"
const result = await $ctx.$repos.products.find({
  where: {
    name: { _starts_with: 'Apple' }
  }
});

// Matches: "Apple iPhone", "Apple Watch"
// Doesn't match: "iPhone", "My Apple Product"
```

### _ends_with (Ends With Text)

Match fields ending with the text.

```javascript
// Find products ending with "Pro"
const result = await $ctx.$repos.products.find({
  where: {
    name: { _ends_with: 'Pro' }
  }
});

// Matches: "iPhone Pro", "MacBook Pro"
// Doesn't match: "Pro iPhone", "Professional"
```

## Null Checks

### _is_null (Field Is Null)

Match fields that are null.

```javascript
// Find products without description
const result = await $ctx.$repos.products.find({
  where: {
    description: { _is_null: true }
  }
});
```

### _is_not_null (Field Is Not Null)

Match fields that are not null.

```javascript
// Find products with description
const result = await $ctx.$repos.products.find({
  where: {
    description: { _is_not_null: true }
  }
});
```

## Range Operators

### _between (Between Values)

Match values between two numbers (inclusive).

```javascript
// Find products with price between 100 and 500
const result = await $ctx.$repos.products.find({
  where: {
    price: { _between: [100, 500] }
  }
});

// Equivalent to: price >= 100 AND price <= 500
```

## Logical Operators

### _and (All Conditions Must Match)

Combine multiple conditions with AND logic.

```javascript
// Find active products in electronics category with price >= 100
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      { category: { _eq: 'electronics' } },
      { price: { _gte: 100 } },
      { isActive: { _eq: true } }
    ]
  }
});
```

### _or (At Least One Condition Must Match)

Combine multiple conditions with OR logic.

```javascript
// Find products that are active OR on sale
const result = await $ctx.$repos.products.find({
  where: {
    _or: [
      { isActive: { _eq: true } },
      { isOnSale: { _eq: true } }
    ]
  }
});
```

### _not (Negate Condition)

Negate a condition.

```javascript
// Find products that are not discontinued
const result = await $ctx.$repos.products.find({
  where: {
    _not: {
      status: { _eq: 'discontinued' }
    }
  }
});
```

### Complex Logical Combinations

Combine logical operators for complex queries.

```javascript
// Find active products in electronics OR gadgets, with price between 100-500
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      {
        _or: [
          { category: { _eq: 'electronics' } },
          { category: { _eq: 'gadgets' } }
        ]
      },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } }
    ]
  }
});
```

## Complex Examples

### Example 1: Multiple Conditions

```javascript
// Find active products in electronics or gadgets category
// with price between 100-500 and name containing "phone"
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      {
        _or: [
          { category: { _eq: 'electronics' } },
          { category: { _eq: 'gadgets' } }
        ]
      },
      { price: { _between: [100, 500] } },
      { name: { _contains: 'phone' } },
      { isActive: { _eq: true } }
    ]
  }
});
```

### Example 2: Exclude Specific Values

```javascript
// Find products not in discontinued or old categories
// with description and price >= 50
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      {
        category: { _not_in: ['discontinued', 'old'] }
      },
      { description: { _is_not_null: true } },
      { price: { _gte: 50 } }
    ]
  }
});
```

### Example 3: Text Search with Other Conditions

```javascript
// Find products with name starting with "Apple"
// price >= 200, and not discontinued
const result = await $ctx.$repos.products.find({
  where: {
    _and: [
      { name: { _starts_with: 'Apple' } },
      { price: { _gte: 200 } },
      {
        _not: {
          status: { _eq: 'discontinued' }
        }
      }
    ]
  }
});
```

### Example 4: Date Range Queries

```javascript
// Find orders created between two dates
const startDate = new Date('2024-01-01');
const endDate = new Date('2024-12-31');

const result = await $ctx.$repos.orders.find({
  where: {
    createdAt: { _between: [startDate, endDate] }
  }
});
```

## Filtering by Relations

Filter records based on related table data. You can filter relations in two ways:

### Method 1: Filter by Relation ID Directly

Filter directly on the relation using ID comparison operators. This is the simplest way to filter by relation ID.

```javascript
// Find menu items without a parent (parent is null)
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _is_null: true }
  }
});

// Find menu items with a parent (parent is not null)
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _is_not_null: true }
  }
});

// Find menu items where parent ID equals 3
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _eq: 3 }
  }
});

// Find menu items where parent ID is in [3, 4, 5]
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _in: [3, 4, 5] }
  }
});

// Find menu items where parent ID is not 3
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _neq: 3 }
  }
});

// Find menu items where parent ID is not in [1, 2]
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: { _not_in: [1, 2] }
  }
});
```

### Method 2: Filter by Relation ID via id/_id Field

You can also filter by explicitly specifying the `id` or `_id` field of the relation.

```javascript
// Find menu items where parent ID equals 3
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: {
      id: { _eq: 3 }
    }
  }
});

// Find menu items without a parent
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: {
      id: { _is_null: true }
    }
  }
});

// Find menu items where parent ID is in [3, 4, 5]
const result = await $ctx.$repos.menu_definition.find({
  where: {
    parent: {
      id: { _in: [3, 4, 5] }
    }
  }
});
```

### Method 3: Filter by Relation Fields

Filter by fields within the related table.

```javascript
// Find products where category name is "Electronics"
const result = await $ctx.$repos.products.find({
  where: {
    category: {
      name: { _eq: 'Electronics' }
    }
  }
});

// Find orders where customer email contains "@example.com"
const result = await $ctx.$repos.orders.find({
  where: {
    customer: {
      email: { _contains: '@example.com' }
    }
  }
});
```

**Note:** Both Method 1 and Method 2 achieve the same result when filtering by relation ID. Use Method 1 for simpler syntax, or Method 2 if you prefer explicit field specification.

## Using Filters in URL Queries

Filters can be used in URL query strings for REST API calls.

```http
# Simple filter
GET /products?filter={"category":{"_eq":"electronics"}}

# Multiple conditions
GET /products?filter={"price":{"_gte":100},"isActive":{"_eq":true}}

# Complex filter with logical operators
GET /products?filter={"_and":[{"category":{"_eq":"electronics"}},{"price":{"_between":[100,500]}}]}

# Text search
GET /products?filter={"name":{"_contains":"phone"}}
```

## Combining with Sorting and Pagination

Combine filters with sorting and pagination.

```javascript
const result = await $ctx.$repos.products.find({
  where: {
    category: { _eq: 'electronics' },
    price: { _between: [100, 500] },
    isActive: { _eq: true }
  },
  fields: 'id,name,price',
  sort: '-price',
  limit: 20,
  meta: 'totalCount'
});
```

## Best Practices

1. **Use appropriate operators** - Choose the right operator for your use case
2. **Combine conditions logically** - Use `_and` and `_or` to build complex queries
3. **Use indexes** - Create database indexes on frequently filtered fields
4. **Limit results** - Always use `limit` for queries that might return many records
5. **Use text search carefully** - Text search can be slower, use it with other filters

## Deep Queries (Nested Relations)

Query multiple levels of related data in a single request using the `deep` parameter. This is particularly useful for fetching complex object graphs.

### Basic Deep Query Syntax

```javascript
// Fetch users with their posts
const result = await $ctx.$repos.users.find({
  fields: 'id,name,email',
  deep: {
    posts: {
      fields: 'id,title,content'
    }
  }
});
```

### Multi-Level Nested Queries

Fetch deeply nested relations across multiple tables:

```javascript
// Fetch users with posts, comments, and authors
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      deep: {
        comments: {
          fields: 'id,content',
          deep: {
            author: {
              fields: 'id,name,email'
            }
          }
        }
      }
    }
  }
});
```

### Deep Query with Filter

Filter nested relations at any level:

```javascript
// Fetch users with active posts only
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      filter: {
        isActive: { _eq: true }
      }
    }
  }
});

// Nested filter at multiple levels
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      filter: {
        status: { _eq: 'published' }
      },
      deep: {
        comments: {
          fields: 'id,content',
          filter: {
            isApproved: { _eq: true }
          }
        }
      }
    }
  }
});
```

### Deep Query with Sort

Sort nested relations at any level:

```javascript
// Fetch users with posts sorted by date
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      sort: '-createdAt'  // Latest posts first
    }
  }
});

// Multi-level sorting
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      sort: '-createdAt',
      deep: {
        comments: {
          fields: 'id,content',
          sort: 'createdAt'  // Ascending for comments
        }
      }
    }
  }
});
```

### Deep Query with Limit and Offset

Limit the number of nested records returned:

```javascript
// Fetch users with their 5 most recent posts
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      sort: '-createdAt',
      limit: 5
    }
  }
});

// Pagination for nested relations
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      page: 2,      // Page 2
      limit: 10     // 10 items per page
    }
  }
});
```

### Combining All Deep Options

Combine filter, sort, and pagination in deep queries:

```javascript
// Complex deep query example
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      filter: {
        status: { _eq: 'published' }
      },
      sort: '-createdAt',
      limit: 10,
      deep: {
        comments: {
          fields: 'id,content,createdAt',
          filter: {
            isApproved: { _eq: true }
          },
          sort: 'createdAt',
          limit: 5
        }
      }
    }
  }
});
```

### URL Examples

Deep queries work in REST API URLs:

```http
# Single level deep
GET /users?fields=id,name&deep={"posts":{"fields":"id,title"}}

# Multi-level deep
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","deep":{"comments":{"fields":"id,content"}}}}

# Deep with filter
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","filter":{"status":{"_eq":"published"}}}}

# Deep with sort and limit
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt","limit":5}}

# Complete deep query
GET /users?fields=id,name&deep={"posts":{"fields":"id,title,createdAt","filter":{"isActive":{"_eq":true}},"sort":"-createdAt","limit":10}}
```

### Performance Considerations

1. **Limit nested results** - Always use `limit` on one-to-many and many-to-many relations
2. **Filter at the right level** - Apply filters as early as possible to reduce data transfer
3. **Select only needed fields** - Specify exact fields in `fields` parameter instead of using `*`
4. **Avoid excessive nesting** - Deep queries with many levels can impact performance
5. **Use pagination** - For large nested datasets, use `page` and `limit` instead of fetching all

### SQL vs MongoDB

**SQL Databases (PostgreSQL, MySQL):**
- Uses CTE (Common Table Expressions) for optimization
- Generates efficient subqueries for each nested level
- Automatically adds WHERE clauses to join related tables correctly

**MongoDB:**
- Uses aggregation pipeline with $lookup operations
- Automatically determines localField and foreignField from metadata
- Preserves type handling (ObjectId conversion)

Both database types handle the same deep query syntax transparently.

## Next Steps

- See [Repository Methods](./repository-methods/) for complete find() documentation
- Learn about [Context Reference](./context-reference/) for accessing query parameters
- Check [API Lifecycle](./api-lifecycle.md) to understand query processing


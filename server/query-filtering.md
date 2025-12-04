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

Filter records based on related table data.

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

## Next Steps

- See [Repository Methods](./repository-methods/) for complete find() documentation
- Learn about [Context Reference](./context-reference/) for accessing query parameters
- Check [API Lifecycle](./api-lifecycle.md) to understand query processing


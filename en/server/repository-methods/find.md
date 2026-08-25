# Find Records

Query records from a table with filtering, sorting, pagination, and field selection.

## Basic Usage

```javascript
// Find all records (up to default limit of 10)
const result = await $ctx.$repos.products.find({});

// Access the data
const products = result.data; // Array of product records
```

## Parameters

```javascript
await $ctx.$repos.tableName.find({
  filter: { ... },       // Filter conditions (optional)
  fields: '...',         // Fields to return (optional)
  limit: 10,             // Max records to return (optional, default: 10)
  sort: '...',           // Sort order (optional, default: primary key field)
  meta: 'totalCount'     // Request metadata (optional)
})
```

## Finding Records

### Get all records (no limit)
```javascript
const result = await $ctx.$repos.products.find({
  limit: 0  // 0 = no limit, fetch all records
});
const allProducts = result.data;
```

### Get limited records
```javascript
const result = await $ctx.$repos.products.find({
  limit: 20  // Return max 20 records
});
```

### Filter records
```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    category: { _eq: 'electronics' },
    price: { _gte: 100 }
  }
});
```

### Filter with multiple conditions
```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { category: { _eq: 'electronics' } },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } }
    ]
  }
});
```

## Selecting Fields

### Return specific fields
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,price'  // Comma-separated field names
});
```

### Return all fields (default)
```javascript
const result = await $ctx.$repos.products.find({
  // No fields parameter = return all fields
});
```

### Include related table fields
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,category.name,category.description'
});
// Returns: [{id: 1, name: "Phone", category: {name: "Electronics", description: "..."}}]
```

### Include all fields from a relation
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,category.*'  // category.* = all category fields
});
```

### Exclude fields
```javascript
const result = await $ctx.$repos.enfyra_route_handler.find({
  fields: '-compiledCode'
});
```

Any field prefixed with `-` switches that `fields` scope to exclude mode. In exclude mode, positive field names are ignored, so `fields: 'id,-compiledCode'` returns all readable fields except `compiledCode`.

Nested exclusions use dot notation:

```javascript
const result = await $ctx.$repos.posts.find({
  fields: '-author.avatar'
});
```

### Deep Queries (Nested Relations)

For complex nested queries across multiple levels, use the `deep` parameter:

```javascript
// Fetch products with category, and category's parent category
const result = await $ctx.$repos.products.find({
  fields: 'id,name',
  deep: {
    category: {
      fields: 'id,name',
      deep: {
        parent: {
          fields: 'id,name'
        }
      }
    }
  }
});

// With filter, sort, and pagination
const result = await $ctx.$repos.products.find({
  fields: 'id,name',
  deep: {
    category: {
      fields: 'id,name',
      filter: { isActive: { _eq: true } },
      sort: 'name',
      limit: 10
    }
  }
});
```

Deep fields also support exclude mode at each nested scope:

```javascript
const result = await $ctx.$repos.posts.find({
  fields: 'id,title',
  deep: {
    comments: {
      fields: '-compiledCode,-author.avatar',
      limit: 10,
      deep: {
        author: {
          fields: '-avatar'
        }
      }
    }
  }
});
```

**For comprehensive deep query documentation**, see [Deep Queries Guide](../query-filtering.md#deep-queries-nested-relations) with:
- Multi-level nesting examples
- Filtering nested relations
- Sorting and pagination per level
- Performance best practices
- URL query examples

## Aggregate Computed Results

Use `aggregate()` when you need calculated values instead of raw records. It is separate from `find()` and returns computed rows under `data`; it never returns raw records or `meta.aggregate`.

### Scalar summary

Omit `dimensions` to return one summary row for the filtered set:

```javascript
const result = await $ctx.$repos.orders.aggregate({
  filter: {
    status: { _eq: 'paid' }
  },
  measures: {
    orders: { count: 'id' },
    revenue: { sum: 'amount' },
    averageOrderValue: { avg: 'amount' }
  }
});

// result.data[0] = { orders: 42, revenue: 18400, averageOrderValue: 438.1 }
```

Supported measure operations are `count`, `countDistinct`, `sum`, `avg`, `min`, and `max`. Numeric operations require numeric fields. Use `countDistinct` for values such as unique customers.

### Grouped analytics

Add `dimensions` when you need one row per value or time bucket:

```javascript
const result = await $ctx.$repos.ai_usage.aggregate({
  filter: {
    createdAt: {
      _gte: '2026-08-01T00:00:00+07:00',
      _lte: '2026-08-31T23:59:59.999+07:00'
    }
  },
  dimensions: [
    {
      field: 'createdAt',
      bucket: 'day',
      timezone: 'Asia/Ho_Chi_Minh'
    }
  ],
  measures: {
    requests: { count: 'id' },
    totalTokens: { sum: 'totalTokens' },
    creditChargedMilli: { sum: 'creditChargedMilli' }
  },
  sort: [{ field: 'createdAt', direction: 'asc' }],
  limit: 100
});
```

A dimension can be a scalar field or a date bucket using `hour`, `day`, `week`, `month`, or `year`. `sort` fields reference dimension output keys or measure names. `page` and `limit` paginate grouped rows.

Aggregate fields are subject to the same authorization and encrypted-field restrictions as other repository queries. Use a field-permission-enforced repository such as `$ctx.$repos.main` or `$ctx.$repos.secure.<tableName>` when the query crosses a user-facing security boundary.

Do not put `aggregate` inside `find()`; that legacy shape is no longer supported.

## Sorting

### Sort ascending
```javascript
const result = await $ctx.$repos.products.find({
  sort: 'name'  // Sort by name ascending
});
```

### Sort descending
```javascript
const result = await $ctx.$repos.products.find({
  sort: '-price'  // Prefix with "-" for descending
});
```

### Multi-field sorting
```javascript
const result = await $ctx.$repos.products.find({
  sort: 'category,-price'  // Sort by category ASC, then price DESC
});
```

## Filter Operators

The `filter` parameter supports MongoDB-like operators:

### Comparison Operators
```javascript
filter: {
  price: { _eq: 100 },        // Equal to
  price: { _neq: 100 },       // Not equal to
  price: { _gt: 100 },        // Greater than
  price: { _gte: 100 },       // Greater than or equal
  price: { _lt: 500 },        // Less than
  price: { _lte: 500 },       // Less than or equal
  price: { _between: [100, 500] }  // Between (inclusive)
}
```

### Array Operators
```javascript
filter: {
  category: { _in: ['electronics', 'gadgets'] },      // In array
  category: { _not_in: ['discontinued', 'old'] }     // Not in array
}
```

### Text Search Operators
```javascript
filter: {
  name: { _contains: 'phone' },        // Contains text (case-insensitive)
  name: { _starts_with: 'Apple' },     // Starts with
  name: { _ends_with: 'Pro' }          // Ends with
}
```

### Null Checks
```javascript
filter: {
  description: { _is_null: true },        // Field is null
  description: { _is_not_null: true }     // Field is not null
}
```

### Logical Operators
```javascript
filter: {
  _and: [                                    // All conditions must match
    { category: { _eq: 'electronics' } },
    { price: { _gte: 100 } }
  ],
  _or: [                                     // At least one condition must match
    { status: { _eq: 'active' } },
    { status: { _eq: 'pending' } }
  ],
  _not: {                                    // Negate condition
    category: { _eq: 'discontinued' }
  }
}
```

### Complex Combinations
```javascript
filter: {
  _and: [
    { category: { _in: ['electronics', 'gadgets'] } },
    {
      _or: [
        { price: { _lt: 100 } },
        { isOnSale: { _eq: true } }
      ]
    },
    { description: { _is_not_null: true } }
  ]
}
```

## Filtering by Relations

Filter records based on related table data:

```javascript
// Find products where category name is 'Electronics'
const result = await $ctx.$repos.products.find({
  filter: {
    category: {
      name: { _eq: 'Electronics' }
    }
  }
});
```

## Metadata

Request metadata about the query:

```javascript
const result = await $ctx.$repos.products.find({
  filter: { category: { _eq: 'electronics' } },
  meta: 'totalCount'  // Get total count of all records (before filter)
});

console.log(result.meta.totalCount);  // Total records in table
console.log(result.data.length);      // Records matching filter
```

Available metadata options:
- `'totalCount'` - Total number of records in table (ignores filter)
- `'filterCount'` - Number of records matching the filter
- `['totalCount', 'filterCount']` - Both counts

## Complete Example

```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { category: { _in: ['electronics', 'gadgets'] } },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } },
      { description: { _is_not_null: true } }
    ]
  },
  fields: 'id,name,price,category.name',
  sort: '-price',
  limit: 20,
  meta: ['totalCount', 'filterCount']
});

const products = result.data;
const totalCount = result.meta.totalCount;
const filteredCount = result.meta.filterCount;
```

## Next Steps

- See [Create, Update, and Delete Records](./create-update-delete.md)
- Learn about [Common Patterns](./patterns.md)
- Check [Query Filtering](../query-filtering.md) for more filtering examples

# Repository Methods - find()

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
  where: { ... },        // Filter conditions (optional)
  fields: '...',         // Fields to return (optional)
  limit: 10,             // Max records to return (optional, default: 10)
  sort: '...',           // Sort order (optional, default: 'id')
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
  where: {
    category: { _eq: 'electronics' },
    price: { _gte: 100 }
  }
});
```

### Filter with multiple conditions
```javascript
const result = await $ctx.$repos.products.find({
  where: {
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

The `where` parameter supports MongoDB-like operators:

### Comparison Operators
```javascript
where: {
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
where: {
  category: { _in: ['electronics', 'gadgets'] },      // In array
  category: { _not_in: ['discontinued', 'old'] }     // Not in array
}
```

### Text Search Operators
```javascript
where: {
  name: { _contains: 'phone' },        // Contains text (case-insensitive)
  name: { _starts_with: 'Apple' },     // Starts with
  name: { _ends_with: 'Pro' }          // Ends with
}
```

### Null Checks
```javascript
where: {
  description: { _is_null: true },        // Field is null
  description: { _is_not_null: true }     // Field is not null
}
```

### Logical Operators
```javascript
where: {
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
where: {
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
  where: {
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
  where: { category: { _eq: 'electronics' } },
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
  where: {
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

- See [create(), update(), delete() methods](./create-update-delete.md)
- Learn about [Common Patterns](./patterns.md)
- Check [Query Filtering](./query-filtering.md) for more filtering examples


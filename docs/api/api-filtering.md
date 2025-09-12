# API Filtering

Enfyra provides a powerful MongoDB-like filter system for querying your API endpoints programmatically. This system allows developers to filter data using comparison operators, text search, logical operators, relation filtering, and aggregation functions across any table in your database.

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
- **`deep`**: Load nested relations with their own filtering (see [Deep Relations](#deep-relations))

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

## Deep Relations

Load nested relations with their own filtering, pagination, and sorting:

### Basic Deep Relations
```javascript
{
  "deep": {
    "posts": {
      "fields": ["title", "views"],
      "limit": 5,
      "sort": ["-views"]
    }
  }
}
```

### Filtered Deep Relations
```javascript
{
  "deep": {
    "posts": {
      "fields": ["title", "views"],
      "filter": { "views": { "_gt": 1000 } },
      "limit": 3,
      "sort": ["-views"]
    }
  }
}
```

### Nested Deep Relations
```javascript
{
  "deep": {
    "posts": {
      "fields": ["title", "views"],
      "limit": 3,
      "deep": {
        "comments": {
          "fields": ["content", "createdAt"],
          "filter": { "content": { "_contains": "awesome" } },
          "limit": 2,
          "sort": ["-createdAt"]
        }
      }
    }
  }
}
```

## URL Query Examples

### Simple Filtering
```
GET /api/products?filter={"price":{"_gte":100,"_lte":500}}
```

### Complex Filtering with Pagination
```
GET /api/users?filter={"_or":[{"age":{"_lt":25}},{"posts":{"_count":{"_gte":5}}}]}&page=2&limit=10&sort=-createdAt
```

### Field Selection with Relations
```
GET /api/products?fields=name,price,category.name&filter={"category":{"name":{"_contains":"electronics"}}}
```

### Deep Relations Example
```
GET /api/users?deep={"posts":{"fields":["title","views"],"filter":{"views":{"_gt":1000}},"limit":3}}
```

## API Usage in Custom Handlers

When using the filter system in [Custom Handlers](../backend/custom-handlers.md), you can access and utilize these same filtering capabilities:

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
- **Default CRUD operations** on [routes](../backend/routing-management.md)
- **Custom handlers** with full filter access
- **Frontend applications** consuming the API
- **Third-party integrations** and webhooks

## Best Practices

### Performance Optimization
- **Use field selection**: Only request fields you need
- **Apply filters early**: Use specific filters to reduce data transfer
- **Limit deep relations**: Avoid loading excessive nested data
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
  "fields": "*",
  "deep": {
    "everything": {
      "fields": ["*"],
      "deep": { "more": {} }
    }
  }
}
```

The API filtering system provides MongoDB-like querying power while maintaining SQL database performance and type safety through Enfyra's query engine.
# Context Reference - Repositories

Access database tables through repositories. See [Repository Methods](../repository-methods/) for complete details.

## Accessing Repositories

```javascript
// Access repository by table name
const productsRepo = $ctx.$repos.products;
const usersRepo = $ctx.$repos.user_definition;

// Access main table repository
const mainRepo = $ctx.$repos.main;

// Check if repository exists
if ($ctx.$repos.products) {
  const result = await $ctx.$repos.products.find({});
}
```

## Repository Methods

Each repository provides these methods:

```javascript
// Find records
const result = await $ctx.$repos.products.find({
  where: { category: { _eq: 'electronics' } },
  fields: 'id,name,price',
  limit: 10,
  sort: '-createdAt'
});

// Create record
const createResult = await $ctx.$repos.products.create({
  data: {
    name: 'New Product',
    price: 99.99
  }
});

// Update record
const updateResult = await $ctx.$repos.products.update({
  id: 123,
  data: { price: 89.99 }
});

// Delete record
const deleteResult = await $ctx.$repos.products.delete({
  id: 123
});
```

**Important:** All repository methods return `{ data: [...], meta: {...} }` format.

See [Repository Methods Guide](../repository-methods/) for complete documentation.

## Next Steps

- See [Request Data](./request-data.md) for accessing request information
- Check [Helpers & Cache](./helpers-cache.md) for utility functions
- Learn about [Repository Methods](../repository-methods/) for complete method reference


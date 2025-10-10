# GraphQL API

Enfyra provides a powerful GraphQL API that automatically generates queries and mutations for all tables in your database. The GraphQL API uses the same query engine as REST endpoints, supporting filtering, sorting, pagination, and relation loading.

## GraphQL Endpoint

Access the GraphQL API at:

```
http://localhost:1105/graphql
```

Replace `localhost:1105` with your backend server URL in production.

## Quick Navigation

**Essential Sections:**
- [GraphQL Queries](#graphql-queries) - Read data with filtering and sorting
- [GraphQL Mutations](#graphql-mutations) - Create, update, and delete operations
- [Permission Control](#permission-control) - Configure access control
- [Error Handling](#error-handling) - Handle errors gracefully

**Advanced Topics:**
- [Relation Loading](#relation-loading) - Load related data
- [Best Practices](#best-practices) - Production-ready patterns

---

## GraphQL Queries

GraphQL queries allow you to read data from your tables with powerful filtering, sorting, and pagination capabilities.

### Basic Query

Query a table with filtering and field selection:

```graphql
query {
  users(
    filter: {status: {_eq: "active"}}
    sort: ["name", "-createdAt"]
    page: 1
    limit: 10
  ) {
    data {
      id
      name
      email
      role {
        name
      }
    }
    meta {
      totalCount
      filterCount
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "users": {
      "data": [
        {
          "id": 1,
          "name": "John Doe",
          "email": "john@example.com",
          "role": {
            "name": "Admin"
          }
        }
      ],
      "meta": {
        "totalCount": 100,
        "filterCount": 25
      }
    }
  }
}
```

### Query Parameters

GraphQL queries support filtering, sorting, and pagination through query arguments:

- **`filter`**: JSON object for filtering conditions (see [Filter Operators](./api-querying.md#filter-operators))
- **`sort`**: Array of sort fields (prefix with `-` for descending, e.g., `["name", "-createdAt"]`)
- **`page`**: Page number for pagination (starts at 1)
- **`limit`**: Maximum records to return (default: 10)

**Note:** Unlike REST endpoints, GraphQL does NOT use `fields` or `meta` parameters. Instead:
- **Field Selection**: Specify fields in the GraphQL selection set (e.g., `data { id name email }`)
- **Metadata**: Include meta fields in the selection set (e.g., `meta { totalCount filterCount }`)


### Complex Filtering Example

```graphql
query {
  posts(
    filter: {
      _or: [
        {views: {_gt: 1000}},
        {featured: {_eq: true}}
      ],
      status: {_eq: "published"}
    }
    sort: ["-views"]
    limit: 20
  ) {
    data {
      id
      title
      views
      author {
        name
      }
    }
    meta {
      totalCount
    }
  }
}
```

### Relation Loading

Load related data using field selection or deep relations:

```graphql
# Load related data through GraphQL selection set
query {
  posts(limit: 10) {
    data {
      id
      title
      author {
        name
        email
      }
      category {
        name
      }
    }
  }
}
```

**Note:** GraphQL automatically handles relation loading through the selection set. Simply include the relation fields you need in your query.

---

## GraphQL Mutations

Enfyra automatically generates GraphQL mutations for Create, Update, and Delete operations for all tables in your database. Mutations follow a consistent naming pattern based on table names.

### Mutation Naming Convention

For a table named `users`, the following mutations are automatically available:
- `create_users`: Create a new record
- `update_users`: Update an existing record
- `delete_users`: Delete a record

### Create Mutation

Create a new record in a table:

```graphql
mutation {
  create_users(input: {
    name: "John Doe"
    email: "john@example.com"
    status: "active"
  }) {
    id
    name
    email
    status
    createdAt
    updatedAt
  }
}
```

**With Variables:**
```graphql
mutation CreateUser($input: UsersInput!) {
  create_users(input: $input) {
    id
    name
    email
    createdAt
  }
}
```

**Variables:**
```json
{
  "input": {
    "name": "John Doe",
    "email": "john@example.com",
    "status": "active"
  }
}
```

**Notes:**
- Input fields are based on table columns (excluding `id`, `createdAt`, `updatedAt`)
- Required fields depend on your table schema
- Returns the created record with all requested fields

### Update Mutation

Update an existing record by ID:

```graphql
mutation {
  update_users(id: "123", input: {
    name: "Jane Doe"
    status: "inactive"
  }) {
    id
    name
    email
    status
    updatedAt
  }
}
```

**With Variables:**
```graphql
mutation UpdateUser($id: ID!, $input: UsersInput!) {
  update_users(id: $id, input: $input) {
    id
    name
    email
    status
    updatedAt
  }
}
```

**Variables:**
```json
{
  "id": "123",
  "input": {
    "name": "Jane Doe",
    "status": "inactive"
  }
}
```

**Notes:**
- Only provide fields you want to update in the `input`
- The `id` parameter is required to identify the record
- Returns the updated record with all requested fields

### Delete Mutation

Delete a record by ID:

```graphql
mutation {
  delete_users(id: "123")
}
```

**With Variables:**
```graphql
mutation DeleteUser($id: ID!) {
  delete_users(id: $id)
}
```

**Variables:**
```json
{
  "id": "123"
}
```

**Response:**
```json
{
  "data": {
    "delete_users": "Delete id 123 successfully"
  }
}
```

**Notes:**
- Returns a success message string
- No need to specify return fields (it's a String type)
- Throws an error if the record doesn't exist

---

## Permission Control

GraphQL queries and mutations require appropriate permissions configured in the Routing Management system.

### Permission Methods

- **`GQL_QUERY`**: Controls read access via GraphQL queries
- **`GQL_MUTATION`**: Controls write access (create, update, delete) via GraphQL mutations

### Configuring Permissions

GraphQL permissions are configured the same way as REST API permissions through Route Permissions. For complete step-by-step instructions, examples, and best practices, see:

**→ [Routing Management - GraphQL Permission Control](../frontend/routing-management.md#graphql-permission-control)**

**Quick Summary:**
- GraphQL permissions are configured in **Settings → Routings → Route Permissions**
- `GQL_QUERY` and `GQL_MUTATION` are separate from REST methods (`GET`, `POST`, etc.)
- You can grant GraphQL access without REST access and vice versa
- Permissions take effect immediately after saving

---

## Error Handling

GraphQL errors follow a standardized format with detailed error messages and codes.

### Query Errors

```graphql
query {
  users(filter: "{\"invalidField\":{\"_eq\":\"value\"}}") {
    data
  }
}
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Field 'invalidField' does not exist in table 'users'",
      "path": ["users"],
      "extensions": {
        "code": "SCRIPT_ERROR"
      }
    }
  ],
  "data": null
}
```

### Mutation Errors

```graphql
mutation {
  delete_users(id: "999")
}
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "id 999 is not exists!",
      "path": ["delete_users"],
      "extensions": {
        "code": "MUTATION_ERROR"
      }
    }
  ],
  "data": null
}
```

### Frontend Error Handling

```javascript
// React example with error handling
const createUser = async (userData) => {
  try {
    const result = await graphqlClient.mutation({
      mutation: CREATE_USER_MUTATION,
      variables: { input: userData }
    });
    return result.data.create_users;
  } catch (error) {
    if (error.graphQLErrors) {
      error.graphQLErrors.forEach(({ message, extensions }) => {
        if (extensions?.code === 'MUTATION_ERROR') {
          console.error('Mutation failed:', message);
          // Show user-friendly error message
        }
      });
    }
    throw error;
  }
};
```

---

## Complete Examples

### User Management Example

```graphql
# 1. Query users with filtering
query GetActiveUsers {
  users(
    filter: "{\"status\":{\"_eq\":\"active\"}}"
    fields: "id,name,email,role.name"
    sort: "name"
    limit: 20
  ) {
    data
    meta
  }
}

# 2. Create a new user
mutation CreateUser {
  create_users(input: {
    name: "Alice Smith"
    email: "alice@example.com"
    roleId: "2"
    status: "active"
  }) {
    id
    name
    email
    role {
      id
      name
    }
    createdAt
  }
}

# 3. Update the user
mutation UpdateUser {
  update_users(id: "456", input: {
    status: "inactive"
  }) {
    id
    name
    status
    updatedAt
  }
}

# 4. Delete the user
mutation DeleteUser {
  delete_users(id: "456")
}
```

### Blog Post Management

```graphql
# Query published posts with author details
query GetPublishedPosts {
  posts(
    filter: "{\"status\":{\"_eq\":\"published\"},\"publishedAt\":{\"_lte\":\"2024-01-20\"}}"
    fields: "id,title,excerpt,author.name,author.avatar,category.name"
    sort: "-publishedAt"
    limit: 10
  ) {
    data
    meta
  }
}

# Create a new post
mutation CreatePost($input: PostsInput!) {
  create_posts(input: $input) {
    id
    title
    content
    status
    createdAt
  }
}

# Update post status
mutation PublishPost($id: ID!) {
  update_posts(id: $id, input: {
    status: "published"
    publishedAt: "2024-01-20T10:00:00Z"
  }) {
    id
    status
    publishedAt
  }
}
```

---

## Best Practices

### 1. Request Only Required Fields

```graphql
# ✅ Good: Only request fields you need
mutation {
  create_users(input: { name: "John", email: "john@example.com" }) {
    id
    name
  }
}

# ❌ Avoid: Requesting all fields when not needed
mutation {
  create_users(input: { name: "John", email: "john@example.com" }) {
    id
    name
    email
    status
    createdAt
    updatedAt
    role { id name permissions }
  }
}
```

### 2. Use Variables for Dynamic Data

```graphql
# ✅ Good: Use variables
mutation CreateUser($input: UsersInput!) {
  create_users(input: $input) {
    id
    name
  }
}

# ❌ Avoid: Hardcoded values
mutation {
  create_users(input: { name: "John", email: "john@example.com" }) {
    id
    name
  }
}
```

### 3. Use Auto-Join Instead of Deep Relations

```graphql
# ✅ Good: Use auto-join for related fields
query {
  posts(
    fields: "id,title,author.name,category.name"
    limit: 20
  ) {
    data
  }
}

# ❌ Avoid: Deep relations for simple fields (N+1 queries)
query {
  posts(
    deep: "{\"author\":{\"fields\":[\"name\"]},\"category\":{\"fields\":[\"name\"]}}"
    limit: 20
  ) {
    data
  }
}
```

### 4. Validate Input Before Mutation

```javascript
// Validate data before sending to GraphQL
const validateUserInput = (input) => {
  if (!input.email || !input.email.includes('@')) {
    throw new Error('Invalid email format');
  }
  if (!input.name || input.name.length < 2) {
    throw new Error('Name must be at least 2 characters');
  }
  return true;
};

// Use in mutation
const createUser = async (userData) => {
  validateUserInput(userData);
  return await graphqlClient.mutation({
    mutation: CREATE_USER_MUTATION,
    variables: { input: userData }
  });
};
```

### 5. Handle Errors Gracefully

```javascript
const safeGraphQLQuery = async (query, variables) => {
  try {
    const result = await graphqlClient.query({ query, variables });
    return { data: result.data, error: null };
  } catch (error) {
    console.error('GraphQL Error:', error);
    
    // Extract user-friendly error message
    const message = error.graphQLErrors?.[0]?.message || 'An error occurred';
    const code = error.graphQLErrors?.[0]?.extensions?.code;
    
    return { data: null, error: { message, code } };
  }
};
```

### 6. Use Pagination for Large Datasets

```graphql
# ✅ Good: Always paginate
query {
  users(
    limit: 20
    page: 1
  ) {
    data
    meta
  }
}

# ❌ Dangerous: No pagination
query {
  users(limit: 0) {
    data
  }
}
```

### 7. Implement Optimistic Updates

```javascript
// Update UI immediately, rollback on error
const updateUserOptimistic = async (userId, updates) => {
  // Update UI immediately
  setUser({ ...user, ...updates });
  
  try {
    // Send mutation to server
    const result = await graphqlClient.mutation({
      mutation: UPDATE_USER_MUTATION,
      variables: { id: userId, input: updates }
    });
    
    // Confirm with server response
    setUser(result.data.update_users);
  } catch (error) {
    // Rollback on error
    setUser(originalUser);
    showError('Update failed');
  }
};
```

---

## Related Documentation

- **[API Querying](./api-querying.md)**: Detailed filter operators and query syntax
- **[API Lifecycle](./api-lifecycle.md)**: Request lifecycle, hooks, and context
- **[Permission System](./permission-system.md)**: Advanced permission configuration
- **[Context Reference](./context-reference.md)**: Available context variables

---

## GraphQL Playground

Enfyra includes a built-in GraphQL playground for testing queries and mutations:

1. Start your Enfyra server
2. Navigate to `http://localhost:3000/graphql` (or your configured endpoint)
3. Use the playground to:
   - Explore the auto-generated schema
   - Test queries and mutations
   - View documentation for all available types
   - Debug with real-time error messages

The GraphQL playground provides:
- **Auto-completion** for queries and mutations
- **Schema exploration** with all available types and fields
- **Real-time validation** of your queries
- **Query history** for quick iteration

---

## Related Documentation

- **[API Querying](./api-querying.md)** - Advanced filtering, sorting, and aggregation options
- **[Swagger API](./swagger-api.md)** - Interactive REST API documentation and testing
- **[Routing Management](../frontend/routing-management.md)** - Configure routes and permissions


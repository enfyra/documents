# Swagger API Documentation

Enfyra provides built-in Swagger/OpenAPI documentation for all your REST API endpoints. The documentation is automatically generated from your table and route definitions, and updates in real-time when your schema changes.

## Quick Navigation

**Essential Sections:**
- [Accessing Swagger UI](#accessing-swagger-ui) - View and test your APIs
- [Auto-Generated Documentation](#auto-generated-documentation) - What gets documented automatically
- [Testing APIs](#testing-apis) - Use Swagger UI to test endpoints
- [Permission Control](#permission-control) - Configure who can access documentation

**Advanced Topics:**
- [OpenAPI Specification](#openapi-specification) - Export and use the spec
- [Hot Reload](#hot-reload) - Schema changes and automatic updates

---

## Accessing Swagger UI

Enfyra's Swagger documentation is available at:

```
http://localhost:1105/api-docs
```

Replace `localhost:1105` with your backend server URL in production.

### What You'll See

The Swagger UI provides:
- **Interactive API documentation** for all your routes
- **Try it out** feature to test endpoints directly
- **Request/Response schemas** with data types and validation
- **Authentication support** with JWT bearer tokens
- **Example requests and responses**

---

## Auto-Generated Documentation

Swagger automatically documents all routes from your Route Definitions.

### For Each Route

Every route you create gets documented with:

**Available Operations:**
- `GET {path}` - List/find records with filtering and pagination
- `POST {path}` - Create new records
- `PATCH {path}/{id}` - Update existing records
- `DELETE {path}/{id}` - Delete records

**Parameters Documented:**
- **Query Parameters:** filter, fields, sort, page, limit, meta
- **Path Parameters:** id for update/delete operations
- **Request Body:** Auto-generated schemas from table columns
- **Response Schemas:** Complete data models with relations

### Request Body Schemas

For each table, Swagger generates input schemas automatically:

**Create Operation (POST):**
- Includes all columns except: `id`, `createdAt`, `updatedAt`
- Required fields based on `isNullable` column property
- Data types from column definitions

**Update Operation (PATCH):**
- Same fields as create
- All fields are optional (partial update)

**Example for `users` table:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "status": "active"
}
```

### Response Schemas

**List Response (GET):**
```json
{
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "createdAt": "2024-01-20T10:00:00Z",
      "updatedAt": "2024-01-20T10:00:00Z"
    }
  ],
  "meta": {
    "totalCount": 100,
    "filterCount": 25
  }
}
```

**Single Item Response (POST, PATCH):**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T10:00:00Z"
}
```

---

## Testing APIs

Swagger UI allows you to test your APIs directly from the browser.

### Testing GET Requests

1. Find the endpoint you want to test (e.g., `GET /users`)
2. Click **"Try it out"** button
3. Enter query parameters:
   - `filter`: `{"status":{"_eq":"active"}}`
   - `limit`: `10`
   - `page`: `1`
4. Click **"Execute"**
5. View the response below

### Testing POST Requests (Create)

1. Find the create endpoint (e.g., `POST /users`)
2. Click **"Try it out"**
3. Edit the request body JSON
4. Add authentication if required
5. Click **"Execute"**
6. View the created record in the response

### Testing PATCH Requests (Update)

1. Find the update endpoint (e.g., `PATCH /users/{id}`)
2. Click **"Try it out"**
3. Enter the record `id`
4. Edit the request body with fields to update
5. Click **"Execute"**

### Testing DELETE Requests

1. Find the delete endpoint (e.g., `DELETE /users/{id}`)
2. Click **"Try it out"**
3. Enter the record `id`
4. Click **"Execute"**

### Using Authentication

For protected endpoints:

1. Click **"Authorize"** button at the top
2. Enter your JWT token in the format: `Bearer YOUR_TOKEN`
3. Click **"Authorize"**
4. Close the dialog
5. All subsequent requests will include the token

**Getting a JWT Token:**
- Login via `/auth/login` endpoint
- Copy the access token from response
- Use it in Swagger UI

---

## Permission Control

Access to Swagger documentation is controlled through the Route Permissions system.

### Default Access

By default, Swagger documentation is **public** (anyone can view) with the `publishedMethods: ["GET"]` configuration.

### Restricting Access

To restrict Swagger access to specific roles:

1. Navigate to **Settings → Routings**
2. Find the `/api-docs` route
3. Edit **Published Methods** - remove `GET` to require authentication
4. Add **Route Permissions** for specific roles (e.g., "Developer", "Admin")
5. Save changes

**Example Configuration:**
```
Route: /api-docs
Published Methods: [] (empty - require auth)
Route Permissions:
  - Role: Developer
    Methods: [GET]
  - Role: Admin
    Methods: [GET]
```

Result: Only developers and admins can access Swagger documentation.

For detailed permission configuration, see **[Routing Management Guide](../frontend/routing-management.md)**.

---

## OpenAPI Specification

### Accessing the Spec

The raw OpenAPI specification is available at:

```
http://localhost:1105/api-docs/json
```

This returns the complete OpenAPI 3.0 specification in JSON format.

### Using the Spec

**Import into Postman:**
1. Open Postman
2. Click **Import**
3. Enter the URL: `http://localhost:1105/api-docs/json`
4. Postman will import all endpoints

**Generate Client Code:**
```bash
# Using OpenAPI Generator
npx @openapitools/openapi-generator-cli generate \
  -i http://localhost:1105/api-docs/json \
  -g typescript-axios \
  -o ./generated-client
```

**Download for Documentation:**
```bash
curl http://localhost:1105/api-docs/json > openapi.json
```

### Spec Contents

The OpenAPI specification includes:

- **All routes** from your route definitions
- **All schemas** from your table definitions
- **Authentication schemes** (JWT Bearer)
- **Request/Response examples**
- **Error responses** (400, 401, 403, 404)
- **Query parameters** documentation
- **Filter system** documentation

---

## Hot Reload

Swagger documentation updates automatically when your schema changes.

### Automatic Updates

When you:
- **Create a new table** → New schemas appear
- **Add/modify columns** → Schemas update
- **Create routes** → New endpoints appear
- **Update routes** → Documentation updates
- **Delete tables/routes** → Documentation removes them

**No server restart required!**

### Cluster Support

In multi-instance deployments:
- Schema changes sync across all instances via Redis
- All instances reload Swagger documentation automatically
- Any instance serves up-to-date documentation

---

## Filter System Documentation

Swagger documents the powerful filter system available on all GET endpoints.

### Filter Parameter

All list endpoints (GET) support the `filter` query parameter with MongoDB-like operators:

**Comparison Operators:**
- `_eq`: Equal
- `_neq`: Not equal
- `_gt`: Greater than
- `_gte`: Greater than or equal
- `_lt`: Less than
- `_lte`: Less than or equal
- `_in`: In array
- `_not_in`: Not in array

**Text Operators:**
- `_contains`: Contains substring
- `_starts_with`: Starts with
- `_ends_with`: Ends with

**Logical Operators:**
- `_and`: AND conditions
- `_or`: OR conditions
- `_not`: NOT condition

**Example Filter:**
```json
{
  "status": {"_eq": "active"},
  "age": {"_gte": 18},
  "name": {"_contains": "john"}
}
```

For complete filter documentation, see **[API Querying Guide](./api-querying.md)**.

---

## Best Practices

### Documentation Maintenance

**Keep Descriptions Updated:**
- Add descriptions to your routes
- Document custom handlers
- Add meaningful field descriptions in table columns

**Use Examples:**
- Test your APIs via Swagger
- Verify request/response formats
- Share documentation with team members

### Security Considerations

**Production Deployments:**
```
# Option 1: Restrict to internal users
Published Methods: []
Route Permissions: Developer role only

# Option 2: Completely disable
Route: /api-docs
Is Enabled: false

# Option 3: Keep public (for public APIs)
Published Methods: [GET]
```

**Recommendation:**
- **Development:** Keep public for easy testing
- **Staging:** Restrict to team members
- **Production:** Decide based on API visibility needs

### Performance Tips

**Large Schemas:**
- Swagger spec regenerates on schema changes only
- Spec is cached in memory
- No performance impact on API endpoints

**Multiple Instances:**
- All instances share the same spec
- Redis coordination ensures consistency
- No duplicate generation overhead

---

## Related Documentation

- **[API Querying](./api-querying.md)**: Complete filter system and query syntax
- **[GraphQL API](./graphql-api.md)**: GraphQL queries and mutations
- **[Routing Management](../frontend/routing-management.md)**: Create and configure routes
- **[Permission System](./permission-system.md)**: Advanced access control

---

## Swagger UI Features

### Interactive Testing

- **Execute requests** directly from browser
- **See real responses** with actual data
- **Test error cases** with invalid inputs
- **Copy as cURL** for command-line testing

### Schema Explorer

- **Browse all data models** and their fields
- **See field types** and validation rules
- **Explore relationships** between tables
- **View required fields** and constraints

### Authentication Testing

- **Add JWT tokens** once, use for all requests
- **Test protected endpoints** with proper credentials
- **Switch between users** by changing tokens
- **Persistent auth** across page reloads

---

For advanced API features including filtering, sorting, pagination, and relation loading, see the **[API Querying Documentation](./api-querying.md)**.


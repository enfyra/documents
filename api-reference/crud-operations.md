# CRUD Operations

When you create a table in Enfyra (e.g. products, orders, customers), it automatically gets REST endpoints. Use them from your app to list, create, update, and delete records.

**Base:** `{appUrl}/api/{routePath}`

The route path matches your table name or custom route (e.g. `/products`, `/orders`). Check your Enfyra Collections to see available routes.

## GET /{routePath}

List records with optional filtering, sorting, and pagination.

**URL:** `{appUrl}/api/{routePath}`

**Query Parameters:** See [Query Parameters](./query-parameters.md).

**Example:**
```bash
curl "http://localhost:3000/api/products?limit=10&fields=id,name,price" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response (200):**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [
    { "id": 1, "email": "user@example.com", "name": "User 1" },
    { "id": 2, "email": "user2@example.com", "name": "User 2" }
  ],
  "meta": {
    "totalCount": 50,
    "filterCount": 2
  }
}
```

---

## GET /{routePath}/:id

Get a single record by ID.

**URL:** `{appUrl}/api/{routePath}/{id}`

**Example:**
```bash
curl "http://localhost:3000/api/user_definition/1" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response (200):**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "id": 1,
    "email": "user@example.com",
    "name": "User 1",
    "role": { "id": 1, "name": "Admin" }
  }
}
```

**Note:** For MongoDB, use `_id` instead of `id` if your schema uses it.

---

## POST /{routePath}

Create a new record.

**URL:** `{appUrl}/api/{routePath}`

**Request Body:** JSON object with the table's fields.

**Example:**
```bash
curl -X POST "http://localhost:3000/api/user_definition" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email":"new@example.com","name":"New User","password":"secret123"}'
```

**Response (201):**
```json
{
  "statusCode": 201,
  "message": "Success",
  "data": {
    "id": 3,
    "email": "new@example.com",
    "name": "New User",
    "createdAt": "2024-01-15T12:00:00.000Z"
  }
}
```

---

## PATCH /{routePath}/:id

Update an existing record.

**URL:** `{appUrl}/api/{routePath}/{id}`

**Request Body:** JSON object with fields to update (partial update).

**Example:**
```bash
curl -X PATCH "http://localhost:3000/api/user_definition/3" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Name"}'
```

**Response (200):** Updated record.

---

## DELETE /{routePath}/:id

Delete a record.

**URL:** `{appUrl}/api/{routePath}/{id}`

**Example:**
```bash
curl -X DELETE "http://localhost:3000/api/user_definition/3" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response (200):** Success or deleted record.

---

## Body Validation (POST / PATCH)

If the table has `validateBody = true` (default for new tables), the server validates the request body against the table's schema and any **column rules** attached to its columns. Validation runs before the handler executes.

**What is checked, in order:**

1. Column **type** (int, varchar, boolean, etc.)
2. **Nullability** (`isNullable: false` columns reject `null`)
3. **Length cap** for `varchar` (`options.length`)
4. **Column rules** (min/max, minLength/maxLength, pattern, format, minItems/maxItems)
5. **Strictness** — unknown top-level fields are rejected

**Failure response (HTTP 400):**

```json
{
  "statusCode": 400,
  "message": [
    "name: String must contain at least 3 character(s)",
    "email: Invalid email",
    "age: Number must be greater than or equal to 18"
  ],
  "error": "Bad Request"
}
```

`message` is always an **array of strings** — one per violation. The prefix before `:` is the field name; clients can split on `:` to map errors back to form fields.

**Notes:**

- Cascade create payloads (e.g. `POST /post` with inline `comments: [...]`) validate child records too if the related table also has `validateBody = true`.
- Connect-by-id shapes (`{ author: 5 }` or `{ author: { id: 5 } }`) skip nested validation.
- Tables with `validateBody = false` only enforce database-level constraints at insert/update time; rules are not consulted.
- Field-permission `deny` on `create`/`update` returns **403**, not 400 — that's not validation.

To configure rules per column, see the admin app guide: [Column Rules](../app/column-rules.md).

---

## Custom Routes

You can add custom routes (e.g. `/register`, `/orders/:orderId/items`) in Enfyra Settings. Each route exposes the HTTP methods configured for it. Use the same base URL and authentication as for table routes.

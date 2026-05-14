# User Registration Example

Build a public `POST /register` endpoint that creates a user safely.

This example uses one custom route, one pre-hook, and one custom handler. It is intentionally small so you can copy the pattern into your own app.

## What You Build

```text
POST /api/register
  -> pre-hook validates input
  -> handler hashes password
  -> handler creates user_definition row
  -> response returns safe user fields only
```

Use this when the default `POST /user_definition` route is too generic for public signup.

## 1. Create The Route

In the admin app, create a route:

| Field | Value |
|-------|-------|
| Path | `/register` |
| Method | `POST` |
| Handler | Custom |
| Target table | `user_definition` |

Keep the route public only if this endpoint is meant for signup. Otherwise attach normal route permissions.

## 2. Add A Pre-Hook

Attach this pre-hook to `POST /register`.

It checks the body before the handler runs.

```js
const { email, password } = @BODY

if (!email) @THROW400("Email is required")
if (!password) @THROW400("Password is required")
if (password.length < 8) @THROW400("Password must be at least 8 characters")

const existing = await #user_definition.find({
  filter: { email: { _eq: email } },
  fields: "id",
  limit: 1
})

if (existing.data[0]) @THROW409("Email already exists")
```

Why this belongs in a pre-hook:

- It rejects invalid requests before the handler does work.
- It keeps the handler focused on creation.
- The same pattern works for tenant checks, quotas, or invite validation.

## 3. Add The Custom Handler

Attach this handler to `POST /register`.

```js
const { email, password, name } = @BODY

const hashedPassword = await @HELPERS.$bcrypt.hash(password)

const result = await #user_definition.create({
  data: {
    email,
    password: hashedPassword,
    name: name || null,
    isActive: true
  }
})

const user = result.data[0]

return {
  id: user.id,
  email: user.email,
  name: user.name,
  isActive: user.isActive
}
```

The handler does not return `password`.

## 4. Test The Endpoint

```bash
curl -X POST "http://localhost:3000/api/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "mai@example.com",
    "password": "password123",
    "name": "Mai Tran"
  }'
```

Expected response shape:

```json
{
  "id": 12,
  "email": "mai@example.com",
  "name": "Mai Tran",
  "isActive": true
}
```

## 5. Optional: Trigger A Welcome Flow

If you want email or onboarding work, trigger a flow in a post-hook.

```js
if (!@ERROR && @DATA?.id) {
  await @TRIGGER("welcome-user", {
    userId: @DATA.id,
    email: @DATA.email
  })
}
```

Use a flow when the work can happen after the response or may need retry/history.

## Common Mistakes

### Creating users from the browser with `/user_definition`

Use a dedicated `/register` route for public signup. It gives you one place to validate, hash, and return safe fields.

### Returning the full user row

Never return `password`, reset tokens, OAuth secrets, or internal fields from a public registration endpoint.

### Putting all logic in one handler

Use pre-hook for validation, handler for creation, and post-hook/flow for side effects.

## Related Documentation

- [Routing Management](../app/routing-management.md)
- [Custom Handlers](../app/hooks-handlers/custom-handlers.md)
- [Hooks](../app/hooks-handlers/hooks.md)
- [Flows](../server/flows.md)

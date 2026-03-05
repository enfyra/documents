# API Reference

Guide for **building your app** with the Enfyra API. Use these endpoints to authenticate users, fetch and modify data, and work with files from your custom frontend, mobile app, or external service.

All API requests use the **app URL** with the `/api` prefix.

## Base URL

```
{appUrl}/api/{endpoint}
```

**Examples:**
- Development: `http://localhost:3000/api/me`
- Production: `https://your-app.enfyra.com/api/products`

The Enfyra app proxies requests to the backend, so you call `{appUrl}/api/...` from your app—no need to talk to the backend directly.

## Quick Navigation

| Topic | Documentation |
|-------|---------------|
| **Overview** | Base URL, headers, authentication, response format | [overview.md](./overview.md) |
| **Authentication** | Login, logout, refresh token, OAuth, /me | [authentication.md](./authentication.md) |
| **CRUD Operations** | List, create, update, delete records | [crud-operations.md](./crud-operations.md) |
| **Query Parameters** | filter, fields, sort, limit, page | [query-parameters.md](./query-parameters.md) |
| **File & Storage** | Upload files, list folders, download assets | [file-storage.md](./file-storage.md) |

## Using the API

### cURL Example

```bash
# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"your_password"}'

# Get current user
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"

# List your data
curl "http://localhost:3000/api/products?limit=10" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### JavaScript / fetch

```javascript
const appUrl = 'http://localhost:3000';

// Login
const { data } = await fetch(`${appUrl}/api/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' }),
}).then(r => r.json());

const token = data.accessToken;

// Fetch your data
const products = await fetch(`${appUrl}/api/products?limit=20`, {
  headers: { 'Authorization': `Bearer ${token}` },
}).then(r => r.json());
```

### Enfyra SDK (Nuxt / Next.js)

When using `@enfyra/sdk-nuxt` or `@enfyra/sdk-next`, the composables handle the base URL and authentication:

```vue
<script setup>
const { data, execute } = useApi('/products', { query: { limit: 20 } });
onMounted(() => execute());
</script>
```

## What You Get

- **Auth** – Login, logout, refresh token, OAuth (Google, Facebook, GitHub)
- **Your tables** – Each table you create gets CRUD endpoints (e.g. `/products`, `/orders`)
- **Query** – Filter, sort, paginate with MongoDB-like operators
- **Files** – Upload, organize in folders, serve assets

For Enfyra admin and configuration (routes, hooks, handlers, etc.), see [Server Documentation](../server/README.md).

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
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"your_password"}'

# Get current user
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"

# List your data
curl "http://localhost:3000/api/products?limit=10" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### JavaScript / fetch (Token-Based)

```javascript
const appUrl = 'http://localhost:3000';

// Login
const response = await fetch(`${appUrl}/api/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' }),
}).then(r => r.json());

// Fetch your data. The browser sends httpOnly auth cookies automatically.
const products = await fetch(`${appUrl}/api/products?limit=20`).then(r => r.json());
```

### JavaScript / fetch (Cookie-Based)

Cookie-based authentication automatically handles tokens via HTTP-only cookies for enhanced security:

```javascript
const appUrl = 'http://localhost:3000';

// Login - cookies are automatically set by server
const response = await fetch(`${appUrl}/api/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' })
});

// Fetch your data - browser automatically sends cookies
const products = await fetch(`${appUrl}/api/products?limit=20`).then(r => r.json());
```

For Nuxt, Next, or another SSR app, proxy all Enfyra calls through your app origin. The Enfyra app commonly uses `/api`; third apps can use a prefix such as `/enfyra` and forward it to the Enfyra app `/api` base. Use `{prefix}/login` for password login. For OAuth, start at `{prefix}/auth/{provider}?redirect=<absoluteReturnUrl>&cookieBridgePrefix=<prefix>` and enable Enfyra OAuth set-cookie mode. Enfyra redirects through `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies`, returns `Set-Cookie` for that app origin, then redirects to `redirect`.

**Cookie-Based Benefits:**
- **Enhanced Security**: HTTP-only cookies cannot be accessed by JavaScript
- **CSRF Protection**: Built-in protection with SameSite attribute
- **Automatic Refresh**: Server handles token refresh automatically
- **Simple**: No manual cookie handling needed - browser does it automatically

## What You Get

- **Auth** – Login, logout, refresh token, OAuth (Google, Facebook, GitHub)
- **Your tables** – Each table you create gets CRUD endpoints (e.g. `/products`, `/orders`)
- **Query** – Filter, sort, paginate with MongoDB-like operators
- **Files** – Upload, organize in folders, serve assets

For Enfyra admin and configuration (routes, hooks, handlers, etc.), see [Server Documentation](../server/README.md).

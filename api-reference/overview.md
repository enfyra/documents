# API Overview

Essential details for calling the Enfyra API from your app.

## Base URL Format

All Enfyra REST API endpoints use:

```
{appUrl}/api/{path}
```

| Environment | appUrl | Example |
|-------------|--------|---------|
| Local development | `http://localhost:3000` | `http://localhost:3000/api/me` |
| Production | Your deployed app URL | `https://app.yourdomain.com/api/me` |

**Important:** Always use an app-origin proxy, not the raw backend host. The Enfyra app commonly exposes `/api/**`. Third apps may expose their own prefix, such as `/enfyra/**`, and proxy it to the Enfyra app `/api/**` base. This avoids CORS and lets browser cookies stay on the app origin.

## Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Content-Type` | For POST/PATCH | `application/json` for JSON bodies |
| `Authorization` | Most endpoints | `Bearer {accessToken}` for authenticated requests |
| `Cookie` | Alternative auth | Session cookies if using cookie-based auth |

## Authentication

Most endpoints require a valid JWT access token. Obtain it via:

1. **POST** `{appUrl}/api/login` – SSR/cookie login through the app proxy
2. **GET** `{appUrl}/api/auth/{provider}?redirect=...` – OAuth login (Google, Facebook, GitHub)

Include the token in requests:

```
Authorization: Bearer eyJhbGc...
```

For Nuxt, Next, or another SSR app, prefer cookie-based sessions. Proxy Enfyra through a same-origin prefix, call `{prefix}/login`, fetch the user with `{prefix}/me`, and let the browser send cookies with same-origin requests.

For third apps, OAuth needs one extra query parameter:

```text
GET /enfyra/auth/google?redirect=https%3A%2F%2Fchat.example.com%2Fchat&cookieBridgePrefix=/enfyra
```

`redirect` must be an absolute `http(s)` URL. `cookieBridgePrefix` is the third app proxy prefix that forwards to Enfyra API routes. Enfyra uses it to redirect through `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies`, so cookies are written on the third app origin before returning to `redirect`.

Nuxt third app proxy example:

```ts
export default defineNuxtConfig({
  routeRules: {
    '/enfyra/**': {
      proxy: {
        to: 'https://demo.enfyra.io/api/**',
        fetchOptions: { redirect: 'manual' },
      },
    },
    '/socket.io/**': {
      proxy: { to: 'https://demo.enfyra.io/ws/socket.io/**' },
    },
  },
})
```

**Public endpoints** (no auth required):

- `POST /api/login`
- `POST /api/auth/login` only for manual token clients
- `POST /api/auth/logout`
- `POST /api/auth/refresh-token`
- `GET /api/auth/:provider` (OAuth redirect)
- `GET /api/auth/:provider/callback` (OAuth callback)

## Response Format

### Success (2xx)

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [ ... ],
  "meta": {
    "totalCount": 100,
    "filterCount": 25
  }
}
```

- `data`: Array of records (for list endpoints) or single object (for create/update/me)
- `meta`: Optional; present when `meta=totalCount` or `meta=filterCount` is requested

### Error (4xx, 5xx)

```json
{
  "statusCode": 400,
  "message": "Bad Request",
  "details": "Email is required"
}
```

## HTTP Methods

| Method | Typical use |
|--------|-------------|
| `GET` | List records, get single record by ID, /me |
| `POST` | Create record |
| `PATCH` | Update record by ID |
| `DELETE` | Delete record by ID |

## Next Steps

- [Authentication Endpoints](./authentication.md) – Login, logout, refresh, OAuth, /me
- [CRUD Operations](./crud-operations.md) – Table routes and request/response patterns

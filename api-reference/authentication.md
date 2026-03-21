# Authentication Endpoints

All URLs use the base: `{appUrl}/api`

## POST /auth/login

Authenticate with email and password. Returns access and refresh tokens.

**URL:** `{appUrl}/api/auth/login`

**Request Body:**
```json
{
  "email": "admin@example.com",
  "password": "your_password",
  "remember": false
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | User email |
| password | string | Yes | User password |
| remember | boolean | No | If true, refresh token lasts longer (default: false) |

**Response (200):**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expTime": 900,
    "user": {
      "id": 1,
      "email": "admin@example.com",
      "name": "Admin",
      "role": { "id": 1, "name": "Admin" }
    }
  }
}
```

**Example:**
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"1234"}'
```

---

## POST /auth/logout

Invalidate the current session (refresh token).

**URL:** `{appUrl}/api/auth/logout`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| refreshToken | string | Yes | The refresh token to invalidate |

**Response (200):** Success message.

---

## POST /auth/refresh-token

Get a new access token using a refresh token.

**URL:** `{appUrl}/api/auth/refresh-token`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response (200):**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "expTime": 900
  }
}
```

---

## GET /auth/:provider

Initiate OAuth login. Redirects the user to the provider's authorization page.

**URL:** `{appUrl}/api/auth/{provider}?redirect={redirectUrl}`

| Provider | Path |
|----------|------|
| Google | `{appUrl}/api/auth/google?redirect=...` |
| Facebook | `{appUrl}/api/auth/facebook?redirect=...` |
| GitHub | `{appUrl}/api/auth/github?redirect=...` |

**Query Parameters:**
| Parameter | Required | Description |
|-----------|----------|-------------|
| redirect | Yes | URL to redirect after OAuth success (must be URL-encoded) |

**Example:**
```
http://localhost:3000/api/auth/google?redirect=http%3A%2F%2Flocalhost%3A3000%2Fdashboard
```

**Response:** HTTP 302 redirect to the OAuth provider.

---

## GET /auth/:provider/callback

OAuth callback. The provider redirects here after the user authorizes. The backend exchanges the code for tokens and redirects to the app callback URL with tokens in query params.

**URL:** `{appUrl}/api/auth/{provider}/callback?code=...&state=...`

This endpoint is typically called by the OAuth provider, not directly by your app.

---

## GET /me

Get the current authenticated user's profile.

**URL:** `{appUrl}/api/me`

**Headers:** `Authorization: Bearer {accessToken}`

**Response (200):**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "id": 1,
    "email": "admin@example.com",
    "name": "Admin",
    "isActive": true,
    "role": { "id": 1, "name": "Admin" },
    "createdAt": "2024-01-15T10:00:00.000Z"
  }
}
```

**Example:**
```bash
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

---

## PATCH /me

Update the current user's profile (e.g. name, password).

**URL:** `{appUrl}/api/me`

**Headers:** `Authorization: Bearer {accessToken}`

**Request Body:**
```json
{
  "name": "New Name",
  "password": "new_password"
}
```

| Field | Type | Description |
|-------|------|-------------|
| name | string | Display name |
| password | string | New password (will be hashed) |

**Response (200):** Updated user object.

---


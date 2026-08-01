# Custom API & Transforms

Use the client's HTTP methods when your third-party application calls a custom Enfyra route, needs a non-table response, downloads another response type, or wants request-specific transformation.

## Typed HTTP Methods

```ts
interface PublishResult {
  articleId: number
  publishedAt: string
}

const { data } = await enfyra.post<PublishResult>('/articles/publish', {
  articleId: 42,
})
```

Available methods:

```ts
enfyra.get<T>(endpoint, options?)
enfyra.post<T>(endpoint, body?, options?)
enfyra.put<T>(endpoint, body?, options?)
enfyra.patch<T>(endpoint, body?, options?)
enfyra.delete<T>(endpoint, options?)
```

## Query Parameters and Headers

```ts
const response = await enfyra.get<Report>('/reports/monthly', {
  query: {
    month: '2026-07',
    locale: 'en',
  },
  headers: {
    'x-request-purpose': 'dashboard',
  },
})
```

Public routes use the same client and need no special authentication configuration:

```ts
const { data } = await enfyra.get<PublicStatus>('/public/status')
```

## Response Types

JSON is the default. Select another type when needed:

```ts
const { data: text } = await enfyra.get<string>('/exports/preview', {
  responseType: 'text',
})

const { data: blob } = await enfyra.get<Blob>('/exports/latest', {
  responseType: 'blob',
})
```

Supported values are `json`, `text`, `blob`, and `arrayBuffer`.

## Per-Request Transform

Use `transform` when one call needs a custom result shape. The transformer receives and must return the full `HttpResponse`:

```ts
interface ApiArticle {
  id: number
  title: string
}

interface ArticleOption {
  value: number
  label: string
}

const response = await enfyra.get<ArticleOption[]>('/articles', {
  transform: (current) => ({
    ...current,
    data: (current.data as unknown as ApiArticle[]).map((article) => ({
      value: article.id,
      label: article.title,
    })),
  }),
})
```

Preserve `status`, `headers`, and `meta` unless you intentionally change them.

For table query data only, `.from(table).transform()` is shorter because it receives `data` directly.

## Global Response Transform

Register a transformer in client configuration:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
  onResponse: (response) => ({
    ...response,
    data: normalizeDates(response.data),
  }),
})
```

You may provide one function or an array. You can also register another transformer later:

```ts
enfyra.onResponse(async (response) => {
  await metrics.record(response.status)
  return response
})
```

Global transformers run in registration order. A per-request transformer runs after the global transformers.

## Global Error Transform

```ts
import { EnfyraClient, EnfyraError } from '@enfyra/sdk-core'

const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
  onError: (error) => {
    if (error.code === 'RATE_LIMITED') {
      return new EnfyraError({
        code: error.code,
        statusCode: error.statusCode,
        details: error.details,
        message: 'Please wait before trying again.',
      })
    }
    return error
  },
})
```

An error transformer must return an `EnfyraError`. Preserve the original code, status, and structured details when changing the displayed message.

## Handle Errors at the Call Site

```ts
import { isEnfyraError } from '@enfyra/sdk-core'

try {
  await enfyra.post('/articles/publish', { articleId })
} catch (error) {
  if (!isEnfyraError(error)) throw error

  switch (error.code) {
    case 'UNAUTHORIZED':
      return openLogin()
    case 'FORBIDDEN':
      return showMessage('You do not have access to publish this article.')
    case 'VALIDATION_ERROR':
      return showValidation(error.details)
    case 'RATE_LIMITED':
      return showMessage('Please wait and try again.')
    default:
      return showMessage(error.message)
  }
}
```

Known codes include `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `VALIDATION_ERROR`, `RATE_LIMITED`, `SERVER_ERROR`, `NETWORK_ERROR`, and `TIMEOUT_ERROR`. Enfyra may also return a more specific string code.

## Free-Form Fetch

Use `enfyra.fetch()` when a library expects the standard Fetch API:

```ts
const response = await enfyra.fetch('/custom-report', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ range: 'month' }),
})
```

Relative URLs resolve from the client base. Absolute URLs remain absolute. Enfyra credentials are never attached to an absolute URL outside the configured Enfyra base scope.

## Use an Existing Axios Instance

```ts
import axios from 'axios'

const api = axios.create({ baseURL: 'https://admin.example.com' })
enfyra.attachAxios(api)

const response = await api.get('/api/me')
```

The attachment is idempotent, attaches credentials only to Enfyra-scoped requests, and retries one eligible `401` after the SDK refreshes the session. Release interceptors when the integration is torn down:

```ts
enfyra.dispose()
```

## Cancellation and Unsafe Retries

```ts
const controller = new AbortController()

const request = enfyra.post('/reports/build', input, {
  signal: controller.signal,
  timeout: 60_000,
  retryUnsafe: false,
})

controller.abort()
```

Enable `retryUnsafe: true` only when the endpoint is idempotent or provides its own idempotency key.

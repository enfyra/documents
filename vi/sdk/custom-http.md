---
slug: sdk/custom-route-http-transform
---

# Custom API và Transform

Dùng HTTP method của client khi third-party app gọi custom route, cần response không theo table, tải response type khác hoặc muốn transform riêng cho request.

## HTTP method có type

```ts
interface PublishResult {
  articleId: number
  publishedAt: string
}

const { data } = await enfyra.post<PublishResult>('/articles/publish', {
  articleId: 42,
})
```

Các method hiện có:

```ts
enfyra.get<T>(endpoint, options?)
enfyra.post<T>(endpoint, body?, options?)
enfyra.put<T>(endpoint, body?, options?)
enfyra.patch<T>(endpoint, body?, options?)
enfyra.delete<T>(endpoint, options?)
```

## Query parameter và header

```ts
const response = await enfyra.get<Report>('/reports/monthly', {
  query: {
    month: '2026-07',
    locale: 'vi',
  },
  headers: {
    'x-request-purpose': 'dashboard',
  },
})
```

Public route dùng cùng client và không cần cấu hình xác thực riêng:

```ts
const { data } = await enfyra.get<PublicStatus>('/public/status')
```

## Response type

JSON là mặc định. Chọn type khác khi cần:

```ts
const { data: text } = await enfyra.get<string>('/exports/preview', {
  responseType: 'text',
})

const { data: blob } = await enfyra.get<Blob>('/exports/latest', {
  responseType: 'blob',
})
```

Giá trị hỗ trợ gồm `json`, `text`, `blob` và `arrayBuffer`.

## Transform theo request

`transform` nhận và phải trả về đầy đủ `HttpResponse`:

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

Giữ `status`, `headers` và `meta` trừ khi bạn chủ động thay đổi chúng.

Với riêng table query data, `.from(table).transform()` ngắn hơn vì nhận `data` trực tiếp.

## Global response transform

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
  onResponse: (response) => ({
    ...response,
    data: normalizeDates(response.data),
  }),
})
```

Có thể truyền một function hoặc array. Bạn cũng có thể đăng ký thêm sau:

```ts
enfyra.onResponse(async (response) => {
  await metrics.record(response.status)
  return response
})
```

Global transformer chạy theo thứ tự đăng ký. Per-request transformer chạy sau global transformer.

## Global error transform

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
        message: 'Vui lòng chờ trước khi thử lại.',
      })
    }
    return error
  },
})
```

Error transformer phải trả `EnfyraError`. Giữ code, status và structured details khi thay message hiển thị.

## Xử lý lỗi tại call site

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
      return showMessage('Bạn không có quyền publish bài viết này.')
    case 'VALIDATION_ERROR':
      return showValidation(error.details)
    case 'RATE_LIMITED':
      return showMessage('Vui lòng chờ rồi thử lại.')
    default:
      return showMessage(error.message)
  }
}
```

Known code gồm `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `VALIDATION_ERROR`, `RATE_LIMITED`, `SERVER_ERROR`, `NETWORK_ERROR` và `TIMEOUT_ERROR`. Enfyra cũng có thể trả string code cụ thể hơn.

## Free-form fetch

```ts
const response = await enfyra.fetch('/custom-report', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ range: 'month' }),
})
```

Relative URL resolve từ client base. Absolute URL được giữ nguyên. SDK không gắn Enfyra credential vào absolute URL nằm ngoài Enfyra base scope đã cấu hình.

## Gắn Axios instance có sẵn

```ts
import axios from 'axios'

const api = axios.create({ baseURL: 'https://admin.example.com' })
enfyra.attachAxios(api)

const response = await api.get('/api/me')
```

Attachment là idempotent, chỉ gắn credential vào Enfyra-scoped request và retry một `401` hợp lệ sau khi SDK refresh session. Gỡ interceptor khi integration bị hủy:

```ts
enfyra.dispose()
```

## Hủy request và unsafe retry

```ts
const controller = new AbortController()

const request = enfyra.post('/reports/build', input, {
  signal: controller.signal,
  timeout: 60_000,
  retryUnsafe: false,
})

controller.abort()
```

Chỉ bật `retryUnsafe: true` khi endpoint idempotent hoặc có idempotency key riêng.

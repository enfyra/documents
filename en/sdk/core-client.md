# Core Client

Use `@enfyra/sdk-core` directly in Node.js, edge runtimes, framework-free browser applications, workers, scripts, and integrations that need full control over client creation.

## Install

```bash
yarn add @enfyra/sdk-core
```

## Create a Client

Each Enfyra instance exposes one URL. Use that exact URL as `baseUrl`.

For a local Enfyra instance:

```ts
import { EnfyraClient } from '@enfyra/sdk-core'

const enfyra = new EnfyraClient({
  baseUrl: 'http://localhost:3000',
})
```

For a deployed Enfyra instance:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
})
```

Do not add `/api` or another path. The SDK resolves Enfyra API endpoints from this single URL.

For Nuxt or Next.js, configure this origin through the framework adapter instead. `@enfyra/sdk-nuxt` creates the Nuxt client automatically; `@enfyra/sdk-next` reads `ENFYRA_APP_URL` and provides request-scoped server clients.

## Type Your Data

Define the shape your application reads rather than using `any`:

```ts
interface Article {
  id: number
  title: string
  status: 'draft' | 'published'
  createdAt: string
}

const result = await enfyra
  .from<Article>('articles')
  .select(['id', 'title', 'status', 'createdAt'])
  .execute()
```

The SDK type helps your application at compile time. Enfyra route and field permissions still decide what the server returns at runtime.

## Understand the Response

Raw HTTP methods return an `HttpResponse<T>`:

```ts
const response = await enfyra.get<Article[]>('/articles', {
  query: { fields: 'id,title', limit: 20 },
})

console.log(response.data)
console.log(response.status)
console.log(response.headers)
console.log(response.meta?.totalCount)
```

Query Builder methods return a smaller result containing `data` and optional `meta`:

```ts
const { data, meta } = await enfyra
  .from<Article>('articles')
  .limit(20)
  .meta(['totalCount'])
  .execute()
```

Do not assume `data` is always an array. An ID-scoped or custom endpoint response may be one object, while server behavior and route configuration determine the exact shape.

## Access the Main Capabilities

```ts
enfyra.auth
enfyra.storage
enfyra.from('articles')
enfyra.get('/custom-route')
enfyra.post('/custom-route', { action: 'publish' })
enfyra.fetch('/custom-route')
```

Continue with [Authentication](./authentication.md), [Data Queries and CRUD](./data-queries.md), and [Custom Routes, HTTP, and Transforms](./custom-http.md).

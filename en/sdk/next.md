# Next

Use `@enfyra/sdk-next` for a third-party Next.js application built with the App Router. It adds the same-origin Enfyra rewrite, provides providerless browser hooks, and creates request-scoped server clients that forward auth cookies safely.

This package is for your own Next.js application. It is not the API used by React extensions running inside Enfyra Admin.

## Install

```bash
yarn add @enfyra/sdk-next
```

Set the private Enfyra App origin:

```dotenv
ENFYRA_APP_URL=http://localhost:3000
```

For a new app or an app without custom Next.js configuration, replace `next.config.mjs` with:

```js
export { default } from '@enfyra/sdk-next'
```

That is the complete default setup. The preset adds a rewrite from `/api/enfyra/**` to `${ENFYRA_APP_URL}/api/**`. Do not create a catch-all route, middleware, or Provider for the SDK.

`ENFYRA_APP_URL` must be an HTTP or HTTPS origin without a path suffix. Do not add `/api`.

## Existing Next.js Configuration

Wrap an existing object, synchronous function, or asynchronous function config with `withEnfyra()`:

```ts
import { withEnfyra } from '@enfyra/sdk-next'

const nextConfig = {
  reactStrictMode: true,
}

export default withEnfyra(nextConfig)
```

Existing rewrites, redirects, headers, and `basePath` are preserved. You can pass `{ appUrl, routePrefix, realtime }` as the second argument when the defaults do not fit.

## Authentication

```tsx
'use client'

import { useAuth, useEnfyra } from '@enfyra/sdk-next/client'

export function AccountPanel() {
  const { user, isAuthenticated, pending, error, login, logout } = useAuth()

  if (isAuthenticated) {
    return (
      <div>
        <p>Signed in as {user?.email}</p>
        <button onClick={() => logout()}>Sign out</button>
      </div>
    )
  }

  return (
    <button
      disabled={pending}
      onClick={() => login({ email: 'me@example.com', password: 'secret' })}
    >
      {pending ? 'Signing in…' : 'Sign in'}
    </button>
  )
}

export function ProtectedFeed() {
  const client = useEnfyra()
  return <Feed client={client} />
}
```

No Provider is required. `useEnfyra()` returns the original `EnfyraClient` from `@enfyra/sdk-core`, and `useAuth()` refreshes the cookie session after hydration. Browser traffic stays on the same origin through the configured rewrite.

## Server Components

```tsx
import { createServerEnfyra } from '@enfyra/sdk-next/server'

interface Article {
  id: number
  title: string
  status: string
}

export default async function Page() {
  const enfyra = await createServerEnfyra()

  const result = await enfyra
    .from<Article>('articles')
    .select(['id', 'title', 'status'])
    .filter({ status: { _eq: 'published' } })
    .sort('-createdAt')
    .limit(20)
    .execute()

  return <pre>{JSON.stringify(result.data, null, 2)}</pre>
}
```

`createServerEnfyra()` reads the current request headers and creates a fresh client. Do not cache this client across requests. Only `cookie` and `authorization` are forwarded from the incoming request.

## Server Actions

```tsx
'use server'

import { createServerActionEnfyra } from '@enfyra/sdk-next/server'

export async function createPost(formData: FormData): Promise<void> {
  const title = String(formData.get('title') ?? '').trim()
  if (!title) throw new Error('title is required')

  const { client } = await createServerActionEnfyra()
  await client.from('posts').insert({ title, status: 'draft' })
}
```

`createServerActionEnfyra()` returns a request-scoped `client` and `applySetCookies()`. Use `applySetCookies(response.headers['set-cookie'] ?? [])` after an auth request that returns rotated or deleted cookies.

## Configuration Options

| Option | Default | Purpose |
|---|---|---|
| `appUrl` | `ENFYRA_APP_URL` | Enfyra App origin without `/api` |
| `routePrefix` | `/api/enfyra` | Browser-facing same-origin prefix |
| `realtime` | `false` | Also add the Socket.IO rewrite under the SDK prefix |

Use the configured preset when you prefer explicit values:

```ts
import enfyra from '@enfyra/sdk-next'

export default enfyra({
  appUrl: 'https://admin.example.com',
  realtime: true,
})
```

## Storage

```tsx
'use client'

import { useStorage } from '@enfyra/sdk-next/client'

export function AssetUploader() {
  const { upload, pending } = useStorage()

  async function onChange(event: React.ChangeEvent<HTMLInputElement>) {
    const file = event.target.files?.[0]
    if (!file) return
    await upload({ file, title: file.name })
  }

  return (
    <input type="file" onChange={onChange} disabled={pending} />
  )
}
```

`useStorage()` shares the providerless browser client, so storage uploads use the same-origin rewrite.

## Errors

Errors produced by `@enfyra/sdk-core` (`EnfyraError`) flow through unchanged. Server Components and Server Actions should treat them as regular thrown errors and render a fallback or call `redirect()`:

```tsx
import { isEnfyraError } from '@enfyra/sdk-core'

export default async function Page() {
  try {
    const enfyra = await createServerEnfyra()
    const result = await enfyra.from('articles').limit(1).execute()
    return <pre>{JSON.stringify(result.data, null, 2)}</pre>
  } catch (err) {
    if (isEnfyraError(err)) {
      return <p>Cannot load articles: {err.message}</p>
    }
    throw err
  }
}
```

## Deployment Checklist

- Set `ENFYRA_APP_URL` in the server runtime environment.
- Do not expose the private App origin as a public runtime variable.
- Keep the Enfyra preset or `withEnfyra()` wrapper enabled in every deployed environment.
- Do not create an App Router route or middleware at the configured `routePrefix`; the SDK owns that prefix through rewrites.
- Do not use `output: 'export'`; cookie auth requires a Next.js server runtime.
- Verify login, `/me`, logout, OAuth callback, SSR refresh, and one protected data request after deployment.

## Requirements

- Next.js 14, 15, or 16 with the App Router
- React 18 or 19
- Node.js 18.18 or newer

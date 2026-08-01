---
slug: sdk/next
---

# Next

Dùng `@enfyra/sdk-next` cho third-party app viết bằng Next.js App Router. Package thêm same-origin rewrite tới Enfyra, cung cấp browser hook không cần Provider và tạo server client riêng cho từng request để forward auth cookie an toàn.

Package này dành cho Next.js app riêng của bạn, không phải API dùng trong React extension chạy bên trong Enfyra Admin.

## Cài đặt

```bash
yarn add @enfyra/sdk-next
```

Đặt private origin của Enfyra App:

```dotenv
ENFYRA_APP_URL=http://localhost:3000
```

Với app mới hoặc app chưa có cấu hình Next.js riêng, thay nội dung `next.config.mjs` bằng:

```js
export { default } from '@enfyra/sdk-next'
```

Đây là setup mặc định đầy đủ. Preset thêm rewrite từ `/api/enfyra/**` tới `${ENFYRA_APP_URL}/api/**`. Không tạo catch-all route, middleware hay Provider cho SDK.

`ENFYRA_APP_URL` phải là HTTP hoặc HTTPS origin không có path suffix. Không thêm `/api`.

## Cấu hình Next.js hiện có

Bọc object config, sync function hoặc async function hiện có bằng `withEnfyra()`:

```ts
import { withEnfyra } from '@enfyra/sdk-next'

const nextConfig = {
  reactStrictMode: true,
}

export default withEnfyra(nextConfig)
```

Các rewrite, redirect, header và `basePath` hiện có vẫn được giữ nguyên. Có thể truyền `{ appUrl, routePrefix, realtime }` làm argument thứ hai khi cần thay đổi mặc định.

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

Không cần Provider. `useEnfyra()` trả về `EnfyraClient` gốc từ `@enfyra/sdk-core`, còn `useAuth()` refresh cookie session sau khi hydrate. Browser traffic giữ nguyên same origin qua rewrite đã cấu hình.

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

`createServerEnfyra()` đọc header của request hiện tại và tạo client mới. Không cache client này giữa các request. Chỉ `cookie` và `authorization` được forward từ incoming request.

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

`createServerActionEnfyra()` trả về `client` theo request cùng `applySetCookies()`. Sau auth request có cookie rotate hoặc delete, gọi `applySetCookies(response.headers['set-cookie'] ?? [])`.

## Tùy chọn cấu hình

| Tùy chọn | Mặc định | Mục đích |
|---|---|---|
| `appUrl` | `ENFYRA_APP_URL` | Enfyra App origin không có `/api` |
| `routePrefix` | `/api/enfyra` | Same-origin prefix phía browser |
| `realtime` | `false` | Thêm Socket.IO rewrite bên dưới SDK prefix |

Dùng preset có cấu hình khi muốn truyền giá trị trực tiếp:

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

`useStorage()` dùng chung providerless browser client, nên storage upload đi qua same-origin rewrite.

## Errors

Error do `@enfyra/sdk-core` (`EnfyraError`) trả về không bị biến đổi. Server Component và Server Action nên treat chúng như thrown error thường và render fallback hoặc gọi `redirect()`:

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

## Checklist deploy

- Đặt `ENFYRA_APP_URL` trong server runtime environment.
- Không expose private App origin thành public runtime variable.
- Giữ Enfyra preset hoặc wrapper `withEnfyra()` trong mọi môi trường deploy.
- Không tạo App Router route hay middleware tại `routePrefix` đã cấu hình; SDK sở hữu prefix này qua rewrites.
- Không dùng `output: 'export'`; cookie auth cần Next.js server runtime.
- Kiểm tra login, `/me`, logout, OAuth callback, SSR refresh và một protected data request sau deploy.

## Yêu cầu

- Next.js 14, 15 hoặc 16 với App Router
- React 18 hoặc 19
- Node.js 18.18 trở lên

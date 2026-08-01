---
slug: sdk/react
---

# React

Dùng `@enfyra/sdk-react` cho client-rendered React app sử dụng Enfyra làm backend. Package cung cấp client provider, auth state, query, mutation, storage và realtime hook.

Package này chỉ chạy trong browser và dành cho React app deploy riêng, không phải extension chạy bên trong Enfyra Admin.

## Cài đặt

```bash
yarn add @enfyra/sdk-react @enfyra/sdk-core zustand
```

## Thêm Provider

```tsx
import { EnfyraProvider } from '@enfyra/sdk-react'

export function App() {
  return (
    <EnfyraProvider
      config={{
        baseUrl: 'https://admin.example.com',
      }}
    >
      <ApplicationRoutes />
    </EnfyraProvider>
  )
}
```

Chỉ cấu hình URL của Enfyra App một lần. Provider tạo client và giữ signed-in session cho các SDK hook.

## Truy cập original client

```tsx
import { useEnfyra } from '@enfyra/sdk-react'

function PublishButton({ articleId }: { articleId: number }) {
  const client = useEnfyra()

  return (
    <button onClick={() => client.post('/articles/publish', { articleId })}>
      Publish
    </button>
  )
}
```

## Xác thực

```tsx
import { useAuth } from '@enfyra/sdk-react'

function Session() {
  const { user, isAuthenticated, pending, error, login, logout, refresh } = useAuth()

  if (pending) return <p>Checking session…</p>
  if (error) return <button onClick={() => refresh()}>Retry session check</button>

  if (isAuthenticated) {
    return (
      <p>
        Signed in as {user?.email}
        <button onClick={() => logout()}>Sign out</button>
      </p>
    )
  }

  return (
    <button onClick={() => login({ email, password, remember: true })}>
      Sign in
    </button>
  )
}
```

Auth state dùng chung trong provider. Login, logout hoặc refresh cập nhật mọi consumer.

Với cookie OAuth:

```ts
const client = useEnfyra()
window.location.href = client.auth.getOAuthRedirectUrl(
  'google',
  `${window.location.origin}/dashboard`,
)
```

## Query dữ liệu

```tsx
interface Article {
  id: number
  title: string
  status: string
}

function ArticleList() {
  const { data, pending, error, meta, refresh } = useQuery<Article[]>('articles', {
    select: ['id', 'title', 'status'],
    filter: { status: { _eq: 'published' } },
    sort: '-createdAt',
    limit: 20,
    meta: ['totalCount'],
  })

  if (pending) return <p>Loading…</p>
  if (error) return <button onClick={refresh}>Try again</button>

  return (
    <ul>
      {data?.map((article) => <li key={article.id}>{article.title}</li>)}
    </ul>
  )
}
```

Query chạy khi component mount và khi serialized query input thay đổi. Dùng `immediate: false` cho manual query.

## Mutation

```tsx
function CreateArticle() {
  const createArticle = useMutation<Article>('articles', {
    operation: 'insert',
    onSuccess: () => console.log('Created'),
  })

  async function submit() {
    await createArticle.execute({
      data: { title: 'New article', status: 'draft' },
    })
  }

  return (
    <button disabled={createArticle.pending} onClick={submit}>
      Create
    </button>
  )
}
```

Cập nhật và xóa:

```ts
const updateArticle = useMutation<Article>('articles', { operation: 'update' })
await updateArticle.execute({ id: articleId, data: { status: 'published' } })

const deleteArticle = useMutation('articles', { operation: 'delete' })
await deleteArticle.execute({ id: articleId })
```

Hook trả `null` sau handled failure và expose lỗi trong state. Dùng original client khi code cần throw vào error boundary hoặc đọc đầy đủ error details.

## Storage

```tsx
function UploadField() {
  const { upload, uploading, getDownloadUrl } = useStorage()

  async function handleFile(file: File) {
    const record = await upload(file, { folder: 'customer-documents' })
    if (record) console.log(getDownloadUrl(record.id))
  }

  return (
    <input
      type="file"
      disabled={uploading}
      onChange={(event) => {
        const file = event.target.files?.[0]
        if (file) void handleFile(file)
      }}
    />
  )
}
```

## Realtime

```tsx
function Notifications() {
  const realtime = useWebSocket('notifications', { immediate: true })

  useEffect(() => {
    if (!realtime.connected) return
    return realtime.on('notification:created', (payload) => {
      console.log(payload)
    })
  }, [realtime.connected, realtime.on])

  return realtime.error ? <p>{realtime.error.message}</p> : null
}
```

## Boundary CSR

`EnfyraProvider` throw khi evaluate trên server. Không dùng `@enfyra/sdk-react` trong React Server Component hoặc SSR render process. Với Next.js App Router, dùng [`@enfyra/sdk-next`](./next.md), package cung cấp browser hook không cần Provider và server client theo từng request.

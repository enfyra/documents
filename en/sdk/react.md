# React

Use `@enfyra/sdk-react` for a client-rendered React application that uses Enfyra as its backend. The package provides a client provider, auth state, queries, mutations, storage, and realtime hooks.

This package is CSR-only. It is for a separately deployed React application, not an extension running inside Enfyra Admin.

## Install

```bash
yarn add @enfyra/sdk-react @enfyra/sdk-core zustand
```

## Add the Provider

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

Configure the Enfyra App URL once. The provider creates the client and keeps the signed-in session available to the SDK hooks.

## Access the Original Client

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

## Authentication

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

Auth state is shared by the provider. A login, logout, or refresh updates all consumers.

For cookie OAuth:

```ts
const client = useEnfyra()
window.location.href = client.auth.getOAuthRedirectUrl(
  'google',
  `${window.location.origin}/dashboard`,
)
```

## Query Data

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

Queries execute when the component mounts and whenever their serialized query inputs change. Use `immediate: false` for a manual query.

## Mutations

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

Update and delete:

```ts
const updateArticle = useMutation<Article>('articles', { operation: 'update' })
await updateArticle.execute({ id: articleId, data: { status: 'published' } })

const deleteArticle = useMutation('articles', { operation: 'delete' })
await deleteArticle.execute({ id: articleId })
```

The hook returns `null` after a handled failure and exposes the error in hook state. Use the original client when code must throw to an error boundary or inspect the complete error details.

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

## CSR Boundary

`EnfyraProvider` throws when evaluated on the server. Do not use `@enfyra/sdk-react` in React Server Components or an SSR render process. For Next.js App Router, use [`@enfyra/sdk-next`](./next.md), which provides providerless browser hooks and request-scoped server clients.

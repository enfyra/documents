# Vue

Use `@enfyra/sdk-vue` for a client-rendered Vue 3 application that uses Enfyra as its backend. This package is CSR-only and must not run in an SSR process.

This guide is for a separately deployed Vue application, not a Vue extension running inside Enfyra Admin.

## Install

```bash
yarn add @enfyra/sdk-vue @enfyra/sdk-core
```

## Add the Plugin

```ts
import { createApp } from 'vue'
import { createEnfyra } from '@enfyra/sdk-vue'
import App from './App.vue'

createApp(App)
  .use(createEnfyra({
    baseUrl: 'https://admin.example.com',
  }))
  .mount('#app')
```

Configure the Enfyra App URL once. The plugin creates the client and keeps the signed-in session available to the SDK composables.

## Access the Original Client

```vue
<script setup lang="ts">
const client = useEnfyra()

const result = await client
  .from('articles')
  .select(['id', 'title'])
  .limit(20)
  .execute()
</script>
```

`useEnfyra()` must run below the app that installed `createEnfyra()`.

## Authentication

```vue
<script setup lang="ts">
const { user, isAuthenticated, pending, error, login, logout, refresh } = useAuth()

async function signIn() {
  const result = await login({
    email: email.value,
    password: password.value,
    remember: true,
  })

  if (result) router.push('/dashboard')
}
</script>
```

`useAuth()` refreshes the current user when it is first used. Only a 401 clears the user; other failures remain in `error`.

For cookie OAuth, use the original client:

```ts
const client = useEnfyra()
window.location.href = client.auth.getOAuthRedirectUrl(
  'google',
  `${window.location.origin}/dashboard`,
)
```

## Query Data

```vue
<script setup lang="ts">
interface Article {
  id: number
  title: string
  status: string
}

const { data, pending, error, meta, refresh } = useQuery<Article[]>('articles', {
  select: ['id', 'title', 'status'],
  filter: { status: { _eq: 'published' } },
  sort: '-createdAt',
  limit: 20,
  meta: ['totalCount'],
})
</script>
```

The query runs on component mount. Use `immediate: false` and call `refresh()` for user-triggered searches.

## Create, Update, and Delete

```ts
const createArticle = useMutation<Article>('articles', {
  operation: 'insert',
  onSuccess: () => list.refresh(),
})

await createArticle.execute({
  data: { title: 'New article', status: 'draft' },
})

const updateArticle = useMutation<Article>('articles', {
  operation: 'update',
})
await updateArticle.execute({ id: articleId, data: { status: 'published' } })

const deleteArticle = useMutation('articles', {
  operation: 'delete',
})
await deleteArticle.execute({ id: articleId })
```

`execute()` returns `null` after a handled failure and exposes the normalized error through the composable state. Use `useEnfyra()` directly when the caller must catch the full error.

## Storage

```ts
const { upload, uploading, download, getDownloadUrl, getFolderTree } = useStorage()

const record = await upload(file, {
  folder: folderId,
  title: file.name,
})
```

## Realtime

```ts
const realtime = useWebSocket('notifications')

await realtime.connect()
const stop = realtime.on('notification:created', (payload) => {
  console.log(payload)
})

onBeforeUnmount(stop)
```

The composable disconnects automatically when its owning component unmounts.

## CSR Boundary

Do not import this package into server rendering, static generation execution, or a shared module evaluated by Node.js. Use `@enfyra/sdk-nuxt` for Nuxt SSR. For framework-neutral server code, create `EnfyraClient` from `@enfyra/sdk-core` per request or job.

# Nuxt

Use `@enfyra/sdk-nuxt` for a third-party Nuxt 3 or Nuxt 4 application. It configures the same-origin Enfyra bridge, creates one SDK client per SSR request, forwards auth cookies safely, and auto-imports SDK composables.

This package is for your own Nuxt application. It is not the API used by Vue extensions running inside Enfyra Admin.

## Install

```bash
yarn add @enfyra/sdk-nuxt @enfyra/sdk-core
```

Set the private Enfyra App origin:

```dotenv
ENFYRA_APP_URL=https://admin.example.com
```

Enable the module:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
})
```

That is the complete default setup. Do not repeat the same URL in `nuxt.config.ts`.

If you deliberately prefer configuration over an environment variable, set `enfyra.appUrl` instead:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
  enfyra: {
    appUrl: 'https://admin.example.com',
  },
})
```

The option overrides `ENFYRA_APP_URL` when both exist. Normally, choose one source.

## Optional Proxy Prefix

The browser prefix defaults to `/enfyra`. Change it only when that path conflicts with your app:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
  enfyra: {
    routePrefix: '/backend',
  },
})
```

The module keeps the private App origin server-only and exposes only the local prefix to the browser.

## Call Enfyra Directly

All SDK composables are auto-imported:

```vue
<script setup lang="ts">
interface Article {
  id: number
  title: string
  status: string
}

const client = useEnfyra()
const { data: articles, error, refresh } = await useAsyncData(
  'published-articles',
  async () => {
    const result = await client
      .from<Article>('articles')
      .select(['id', 'title', 'status'])
      .filter({ status: { _eq: 'published' } })
      .sort('-createdAt')
      .limit(20)
      .execute()

    return result.data
  },
)
</script>
```

`useEnfyra()` returns the original `EnfyraClient`. Use it directly for custom routes, query transforms, storage, or any core capability.

## Authentication

```vue
<script setup lang="ts">
const {
  user,
  isAuthenticated,
  pending,
  error,
  login,
  logout,
  refresh,
  oauthLogin,
} = useAuth()

await refresh()

async function signIn(email: string, password: string) {
  const result = await login({ email, password, remember: true })
  if (result) await navigateTo('/dashboard')
}
</script>
```

Only a 401 clears the current user. A network or server failure remains available through `error`.

Start OAuth from a browser action:

```ts
oauthLogin('google')
```

The helper uses the current browser URL as the post-login redirect.

## Client-Reactive Queries

Use `useQuery()` for a query that should execute after the component mounts in the browser:

```ts
const {
  data,
  error,
  pending,
  status,
  meta,
  refresh,
} = useQuery<Article[]>('articles', {
  select: ['id', 'title', 'status'],
  filter: { status: { _eq: 'published' } },
  sort: '-createdAt',
  limit: 20,
  meta: ['totalCount'],
})
```

`useQuery()` runs from `onMounted`; it is not an SSR data-fetching replacement. For server-rendered page data, use `useEnfyra()` inside Nuxt `useAsyncData()` as shown above.

Set `immediate: false` when the user must choose filters first:

```ts
const query = useQuery<Article[]>('articles', {
  immediate: false,
  limit: 20,
})

await query.refresh()
```

## Mutations

```ts
const createArticle = useMutation<Article>('articles', {
  operation: 'insert',
  onSuccess: () => refreshNuxtData('published-articles'),
})

await createArticle.execute({
  data: { title: 'New article', status: 'draft' },
})
```

Update and delete:

```ts
const updateArticle = useMutation<Article>('articles', {
  operation: 'update',
})
await updateArticle.execute({ id: articleId, data: { status: 'published' } })

const deleteArticle = useMutation('articles', {
  operation: 'delete',
})
await deleteArticle.execute({ id: articleId })
```

For custom actions, use the original client:

```ts
const client = useEnfyra()
await client.post('/articles/archive', { articleId })
```

## Response Transformation

```ts
const client = useEnfyra()

const { data } = await client
  .from<Article>('articles')
  .transform((value) => {
    const rows = Array.isArray(value) ? value : [value]
    return rows.map((row) => ({ ...row, title: row.title.trim() }))
  })
  .execute()
```

Global transformers can be registered once in a Nuxt plugin using the injected client:

```ts
export default defineNuxtPlugin(() => {
  const client = useEnfyra()
  client.onError((error) => {
    reportError(error)
    return error
  })
})
```

## Storage

```ts
const { upload, uploading, download, getDownloadUrl, getFolderTree } = useStorage()

const record = await upload(file, {
  folder: folderId,
  title: file.name,
})
```

Use `useEnfyra().storage.getStorageConfigs()` when the user is allowed to select a storage backend.

## Realtime

```ts
const realtime = useWebSocket('notifications', { immediate: true })

watch(realtime.connected, (connected) => {
  if (!connected) return
  realtime.on('notification:created', (payload) => {
    console.log(payload)
  })
})
```

Your deployment still needs the Socket.IO transport proxy when realtime is used. The REST module route rule does not replace the WebSocket transport path.

## Deployment Checklist

- Set `ENFYRA_APP_URL` in the server runtime environment.
- Do not expose the private App origin as a public runtime variable.
- Keep the chosen route prefix stable because OAuth cookies depend on it.
- Do not add a second route rule or cookie handler for the same SDK prefix.
- Verify login, `/me`, logout, OAuth callback, SSR refresh, and one protected data request after deployment.

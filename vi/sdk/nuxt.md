---
slug: sdk/nuxt
---

# Nuxt

Dùng `@enfyra/sdk-nuxt` cho third-party app viết bằng Nuxt 3 hoặc Nuxt 4. Package cấu hình same-origin bridge, tạo SDK client riêng cho từng SSR request, forward auth cookie an toàn và auto-import composable.

Package này dành cho Nuxt app riêng của bạn, không phải API dùng trong Vue extension chạy bên trong Enfyra Admin.

## Cài đặt

```bash
yarn add @enfyra/sdk-nuxt @enfyra/sdk-core
```

Đặt private origin của Enfyra App:

```dotenv
ENFYRA_APP_URL=https://admin.example.com
```

Bật module:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
})
```

Đây là setup mặc định đầy đủ. Không lặp lại cùng URL trong `nuxt.config.ts`.

Nếu chủ động muốn dùng config thay env, đặt `enfyra.appUrl`:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
  enfyra: {
    appUrl: 'https://admin.example.com',
  },
})
```

Option override `ENFYRA_APP_URL` nếu cả hai tồn tại. Thông thường chỉ chọn một nguồn.

## Proxy prefix tùy chọn

Browser prefix mặc định là `/enfyra`. Chỉ đổi khi path này xung đột với app:

```ts
export default defineNuxtConfig({
  modules: ['@enfyra/sdk-nuxt'],
  enfyra: {
    routePrefix: '/backend',
  },
})
```

Module giữ App origin ở private server config và chỉ expose local prefix cho browser.

## Gọi Enfyra trực tiếp

Tất cả SDK composable được auto-import:

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

`useEnfyra()` trả original `EnfyraClient`. Dùng trực tiếp cho custom route, query transform, storage hoặc mọi core capability.

## Xác thực

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

Chỉ 401 mới clear current user. Network hoặc server failure vẫn nằm trong `error`.

Bắt đầu OAuth từ browser action:

```ts
oauthLogin('google')
```

Helper dùng browser URL hiện tại làm post-login redirect.

## Client-reactive query

Dùng `useQuery()` cho query chạy sau khi component mount trong browser:

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

`useQuery()` chạy từ `onMounted`; đây không phải giải pháp thay thế SSR data fetching. Với page data cần server-render, dùng `useEnfyra()` trong Nuxt `useAsyncData()` như ví dụ phía trên.

Đặt `immediate: false` khi user cần chọn filter trước:

```ts
const query = useQuery<Article[]>('articles', {
  immediate: false,
  limit: 20,
})

await query.refresh()
```

## Mutation

```ts
const createArticle = useMutation<Article>('articles', {
  operation: 'insert',
  onSuccess: () => refreshNuxtData('published-articles'),
})

await createArticle.execute({
  data: { title: 'New article', status: 'draft' },
})
```

Cập nhật và xóa:

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

Với custom action, dùng original client:

```ts
const client = useEnfyra()
await client.post('/articles/archive', { articleId })
```

## Transform response

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

Có thể đăng ký global transformer một lần trong Nuxt plugin bằng injected client:

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

Dùng `useEnfyra().storage.getStorageConfigs()` khi user được phép chọn storage backend.

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

Deployment vẫn cần Socket.IO transport proxy khi dùng realtime. REST route rule của module không thay thế WebSocket transport path.

## Checklist deploy

- Đặt `ENFYRA_APP_URL` trong server runtime environment.
- Không expose private App origin thành public runtime variable.
- Giữ route prefix ổn định vì OAuth cookie phụ thuộc nó.
- Không thêm route rule hoặc cookie handler thứ hai cho cùng SDK prefix.
- Kiểm tra login, `/me`, logout, OAuth callback, SSR refresh và một protected data request sau deploy.

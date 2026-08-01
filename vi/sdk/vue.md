---
slug: sdk/vue
---

# Vue

Dùng `@enfyra/sdk-vue` cho client-rendered Vue 3 app sử dụng Enfyra làm backend. Package này chỉ chạy trong browser và không được dùng trong SSR process.

Hướng dẫn này dành cho Vue app deploy riêng, không phải Vue extension chạy bên trong Enfyra Admin.

## Cài đặt

```bash
yarn add @enfyra/sdk-vue @enfyra/sdk-core
```

## Thêm plugin

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

Chỉ cấu hình URL của Enfyra App một lần. Plugin tạo client và giữ signed-in session cho các SDK composable.

## Truy cập original client

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

`useEnfyra()` phải chạy bên dưới app đã install `createEnfyra()`.

## Xác thực

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

`useAuth()` refresh current user khi được dùng lần đầu. Chỉ 401 clear user; failure khác nằm trong `error`.

Với cookie OAuth, dùng original client:

```ts
const client = useEnfyra()
window.location.href = client.auth.getOAuthRedirectUrl(
  'google',
  `${window.location.origin}/dashboard`,
)
```

## Query dữ liệu

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

Query chạy khi component mount. Dùng `immediate: false` và gọi `refresh()` cho search do user kích hoạt.

## Tạo, cập nhật và xóa

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

`execute()` trả `null` sau handled failure và expose normalized error qua composable state. Dùng `useEnfyra()` trực tiếp khi caller cần catch đầy đủ error.

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

Composable tự disconnect khi component sở hữu nó unmount.

## Boundary CSR

Không import package này vào server rendering, static generation execution hoặc shared module được Node.js evaluate. Dùng `@enfyra/sdk-nuxt` cho Nuxt SSR. Với framework-neutral server code, tạo `EnfyraClient` từ `@enfyra/sdk-core` cho từng request hoặc job.

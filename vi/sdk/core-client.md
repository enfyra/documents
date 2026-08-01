---
slug: sdk/core-client
---

# Core Client

Dùng `@enfyra/sdk-core` trực tiếp trong Node.js, edge runtime, browser app không framework, worker, script và integration cần toàn quyền kiểm soát cách tạo client.

## Cài đặt

```bash
yarn add @enfyra/sdk-core
```

## Tạo client

Mỗi Enfyra instance expose một URL duy nhất. Dùng chính URL đó làm `baseUrl`.

Với Enfyra chạy local:

```ts
import { EnfyraClient } from '@enfyra/sdk-core'

const enfyra = new EnfyraClient({
  baseUrl: 'http://localhost:3000',
})
```

Với Enfyra đã deploy:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
})
```

Không thêm `/api` hoặc path khác. SDK tự resolve các Enfyra API endpoint từ URL duy nhất này.

Với Nuxt hoặc Next.js, cấu hình origin này qua framework adapter. `@enfyra/sdk-nuxt` tự tạo Nuxt client; `@enfyra/sdk-next` đọc `ENFYRA_APP_URL` và cung cấp server client theo từng request.

## Khai báo type dữ liệu

Mô tả đúng shape ứng dụng đọc thay vì dùng `any`:

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

Type của SDK hỗ trợ compile-time. Route và field permission của Enfyra vẫn quyết định server trả gì ở runtime.

## Hiểu response

Raw HTTP method trả về `HttpResponse<T>`:

```ts
const response = await enfyra.get<Article[]>('/articles', {
  query: { fields: 'id,title', limit: 20 },
})

console.log(response.data)
console.log(response.status)
console.log(response.headers)
console.log(response.meta?.totalCount)
```

Query Builder trả về result gọn hơn gồm `data` và `meta` nếu có:

```ts
const { data, meta } = await enfyra
  .from<Article>('articles')
  .limit(20)
  .meta(['totalCount'])
  .execute()
```

Không mặc định `data` luôn là array. ID-scoped hoặc custom endpoint có thể trả một object; route configuration và server behavior quyết định shape thực tế.

## Truy cập các khả năng chính

```ts
enfyra.auth
enfyra.storage
enfyra.from('articles')
enfyra.get('/custom-route')
enfyra.post('/custom-route', { action: 'publish' })
enfyra.fetch('/custom-route')
```

Đọc tiếp [Xác thực](./authentication.md), [Query dữ liệu và CRUD](./data-queries.md), và [Custom route, HTTP và transform](./custom-http.md).

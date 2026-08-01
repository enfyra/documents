---
slug: sdk
---

# SDK

Enfyra SDK kết nối third-party application với Enfyra qua URL duy nhất do Enfyra instance expose. Chọn package đúng framework, cấu hình URL đó một lần và dùng SDK trực tiếp.

Các hướng dẫn này dành cho application do bạn tự build và deploy. Để viết extension chạy bên trong Enfyra Admin, dùng [tài liệu App](../app/README.md).

## Packages

| Package | Dùng cho | Hướng dẫn | npm |
|---|---|---|---|
| `@enfyra/sdk-core` | Node.js, worker, script, edge runtime hoặc custom client | [Core Client](./core-client.md) | [Xem trên npm](https://www.npmjs.com/package/@enfyra/sdk-core) |
| `@enfyra/sdk-nuxt` | Nuxt 3 và Nuxt 4 application | [Nuxt](./nuxt.md) | [Xem trên npm](https://www.npmjs.com/package/@enfyra/sdk-nuxt) |
| `@enfyra/sdk-next` | Next.js 14, 15 và 16 App Router application | [Next.js](./next.md) | [Xem trên npm](https://www.npmjs.com/package/@enfyra/sdk-next) |
| `@enfyra/sdk-vue` | Client-rendered Vue 3 application | [Vue](./vue.md) | [Xem trên npm](https://www.npmjs.com/package/@enfyra/sdk-vue) |
| `@enfyra/sdk-react` | Client-rendered React application | [React](./react.md) | [Xem trên npm](https://www.npmjs.com/package/@enfyra/sdk-react) |

## Kết nối Enfyra

Mỗi Enfyra instance expose một URL duy nhất. Dùng URL đó làm SDK `baseUrl`:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'http://localhost:3000',
})
```

Ở production, thay localhost bằng URL Enfyra đã deploy. Không thêm `/api`; SDK tự resolve API endpoint.

Sau đó bạn có thể dùng SDK cho authentication, query và CRUD dữ liệu, custom API và transform, file và storage, cùng realtime connection.

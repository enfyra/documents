# SDK

Enfyra SDKs connect a third-party application to Enfyra through the single URL exposed by your Enfyra instance. Choose the package for your framework, configure that URL once, and use the SDK directly.

These guides are for applications that you build and deploy separately. To build an extension inside Enfyra Admin, use the [App documentation](../app/README.md).

## Packages

| Package | Use it for | Guide | npm |
|---|---|---|---|
| `@enfyra/sdk-core` | Node.js, workers, scripts, edge runtimes, or custom clients | [Core Client](./core-client.md) | [View on npm](https://www.npmjs.com/package/@enfyra/sdk-core) |
| `@enfyra/sdk-nuxt` | Nuxt 3 and Nuxt 4 applications | [Nuxt](./nuxt.md) | [View on npm](https://www.npmjs.com/package/@enfyra/sdk-nuxt) |
| `@enfyra/sdk-next` | Next.js 14, 15, and 16 App Router applications | [Next.js](./next.md) | [View on npm](https://www.npmjs.com/package/@enfyra/sdk-next) |
| `@enfyra/sdk-vue` | Client-rendered Vue 3 applications | [Vue](./vue.md) | [View on npm](https://www.npmjs.com/package/@enfyra/sdk-vue) |
| `@enfyra/sdk-react` | Client-rendered React applications | [React](./react.md) | [View on npm](https://www.npmjs.com/package/@enfyra/sdk-react) |

## Connect to Enfyra

Each Enfyra instance exposes one URL. Use it as the SDK `baseUrl`:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'http://localhost:3000',
})
```

In production, replace the localhost URL with your deployed Enfyra URL. Do not add `/api`; the SDK resolves API endpoints automatically.

From there, the SDK provides authentication, data queries and CRUD, custom API calls and transforms, files and storage, and realtime connections.

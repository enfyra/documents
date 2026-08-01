# Files & Storage

Use the SDK storage API to upload files, download protected assets, build download URLs, browse the folder tree, and list enabled storage configurations.

## Upload a File

```ts
const fileRecord = await enfyra.storage.upload({
  file: selectedFile,
  folder: folderId,
  title: 'Customer contract',
  description: 'Signed contract uploaded from the portal',
  storageConfig: storageConfigId,
})
```

`file` accepts a browser `File` or any `Blob`. The SDK sends multipart form data to the Enfyra file route.

Optional upload fields:

| Field | Purpose |
|---|---|
| `folder` | Place the file in an Enfyra folder |
| `title` | User-facing title |
| `description` | User-facing description |
| `storageConfig` | Select an enabled storage backend |
| `uploadId` | Send an application-generated tracking ID through `x-enfyra-upload-id` |

## Upload from a Browser Input

```ts
async function handleFile(input: HTMLInputElement) {
  const file = input.files?.[0]
  if (!file) return

  const record = await enfyra.storage.upload({
    file,
    folder: 'customer-documents',
  })

  console.log(record.id, record.name)
}
```

Validate file size and accepted MIME types in the UI for a fast user experience. Server-side route and storage rules remain authoritative.

## Get a Download URL

```ts
const url = enfyra.storage.getDownloadUrl(fileId)
```

The result points to `/assets/<fileId>` under the configured SDK base. Use it for an image source, anchor, or browser navigation when the current auth model can authorize that request.

```html
<img :src="client.storage.getDownloadUrl(file.id)" :alt="file.name" />
```

## Download as a Blob

```ts
const blob = await enfyra.storage.download(fileId)
const objectUrl = URL.createObjectURL(blob)

const anchor = document.createElement('a')
anchor.href = objectUrl
anchor.download = 'contract.pdf'
anchor.click()

URL.revokeObjectURL(objectUrl)
```

Use blob download when the asset is protected or when your app needs to control the saved filename.

## Browse Folders

```ts
const folders = await enfyra.storage.getFolderTree()

function walk(nodes: typeof folders, depth = 0) {
  for (const node of nodes) {
    console.log(`${'  '.repeat(depth)}${node.name}`)
    walk(node.children ?? [], depth + 1)
  }
}
```

## List Enabled Storage Configurations

```ts
const configurations = await enfyra.storage.getStorageConfigs()
```

The result contains `id`, `name`, `type`, and `isEnabled`. Show this selector only to users allowed to choose a backend. Most applications should use the server's default storage configuration.

## Framework Helpers

Nuxt, Vue, and React expose `useStorage()` with the full reactive storage helper:

```ts
const { upload, uploading, download, getDownloadUrl, getFolderTree } = useStorage()

const record = await upload(file, {
  folder: folderId,
  uploadId: crypto.randomUUID(),
})
```

The helpers return `null` for handled upload/download failures. Use the core client directly when you need the full `EnfyraError` and structured details.

Next.js exposes an upload-focused providerless helper from its client entry:

```tsx
'use client'

import { useStorage } from '@enfyra/sdk-next/client'

const { upload, pending, error } = useStorage()
const record = await upload({ file, folder: folderId, title: file.name })
```

## Practical Safety

- Do not expose private storage backend credentials to the client.
- Treat download URLs as protected resources unless the file route is intentionally public.
- Keep file permission checks on the server.
- Revoke browser object URLs after use.
- Use a stable `uploadId` if your UI correlates upload progress events with one selected file.

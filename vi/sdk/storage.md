---
slug: sdk/file-storage
---

# File và Storage

Dùng SDK storage API để upload file, download protected asset, tạo download URL, duyệt folder tree và lấy storage configuration đang bật.

## Upload file

```ts
const fileRecord = await enfyra.storage.upload({
  file: selectedFile,
  folder: folderId,
  title: 'Customer contract',
  description: 'Signed contract uploaded from the portal',
  storageConfig: storageConfigId,
})
```

`file` nhận browser `File` hoặc `Blob`. SDK gửi multipart form data đến file route của Enfyra.

| Field tùy chọn | Mục đích |
|---|---|
| `folder` | Đặt file trong folder Enfyra |
| `title` | Tiêu đề hiển thị cho user |
| `description` | Mô tả hiển thị cho user |
| `storageConfig` | Chọn storage backend đang bật |
| `uploadId` | Gửi tracking ID của application qua `x-enfyra-upload-id` |

## Upload từ browser input

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

Validate file size và MIME type trong UI để phản hồi nhanh cho user. Route và storage rule phía server vẫn là nguồn quyết định cuối cùng.

## Lấy download URL

```ts
const url = enfyra.storage.getDownloadUrl(fileId)
```

Kết quả trỏ tới `/assets/<fileId>` dưới SDK base đã cấu hình. Dùng cho image source, anchor hoặc browser navigation khi auth model hiện tại có thể authorize request đó.

```html
<img :src="client.storage.getDownloadUrl(file.id)" :alt="file.name" />
```

## Download thành Blob

```ts
const blob = await enfyra.storage.download(fileId)
const objectUrl = URL.createObjectURL(blob)

const anchor = document.createElement('a')
anchor.href = objectUrl
anchor.download = 'contract.pdf'
anchor.click()

URL.revokeObjectURL(objectUrl)
```

Dùng blob download cho protected asset hoặc khi app cần kiểm soát filename được lưu.

## Duyệt folder

```ts
const folders = await enfyra.storage.getFolderTree()

function walk(nodes: typeof folders, depth = 0) {
  for (const node of nodes) {
    console.log(`${'  '.repeat(depth)}${node.name}`)
    walk(node.children ?? [], depth + 1)
  }
}
```

## Lấy storage configuration đang bật

```ts
const configurations = await enfyra.storage.getStorageConfigs()
```

Kết quả gồm `id`, `name`, `type` và `isEnabled`. Chỉ hiển thị selector này cho user được phép chọn backend. Phần lớn app nên dùng storage configuration mặc định của server.

## Framework helper

Nuxt, Vue và React expose `useStorage()` với full reactive storage helper:

```ts
const { upload, uploading, download, getDownloadUrl, getFolderTree } = useStorage()

const record = await upload(file, {
  folder: folderId,
  uploadId: crypto.randomUUID(),
})
```

Helper trả `null` cho upload/download failure đã xử lý. Dùng core client trực tiếp khi cần đầy đủ `EnfyraError` và structured details.

Next.js expose providerless helper tập trung vào upload từ client entry:

```tsx
'use client'

import { useStorage } from '@enfyra/sdk-next/client'

const { upload, pending, error } = useStorage()
const record = await upload({ file, folder: folderId, title: file.name })
```

## An toàn thực tế

- Không expose private storage backend credential ra client.
- Xem download URL là protected resource trừ khi file route được public có chủ đích.
- Giữ file permission check ở server.
- Revoke browser object URL sau khi dùng.
- Dùng `uploadId` ổn định nếu UI ghép upload progress event với một file đã chọn.

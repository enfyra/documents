---
slug: sdk/xac-thuc
---

# Xác thực

Chỉ cấu hình URL của Enfyra App một lần, sau đó dùng authentication API do package SDK cung cấp. Ứng dụng không cần chọn hoặc tự quản lý session transport.

## Core Client

Tạo client và gọi `auth.login()`:

```ts
import { EnfyraClient } from '@enfyra/sdk-core'

const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
})

await enfyra.auth.login({
  email: 'user@example.com',
  password: 'your-password',
})

const user = await enfyra.auth.getMe()
```

Client giữ session được tạo bởi `login()` và refresh khi cần. Đăng xuất bằng chính client instance đó:

```ts
await enfyra.auth.logout()
```

Authentication method của Core throw `EnfyraError` khi request thất bại. Xử lý lỗi tại form hoặc service đã bắt đầu action:

```ts
import { isEnfyraError } from '@enfyra/sdk-core'

try {
  await enfyra.auth.login({ email, password })
} catch (error) {
  if (isEnfyraError(error) && error.statusCode === 401) {
    showMessage('Email hoặc mật khẩu không đúng')
  } else {
    showMessage('Không thể đăng nhập')
  }
}
```

## Nuxt

Sau khi cài `@enfyra/sdk-nuxt`, dùng composable `useAuth()` đã được auto-import:

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

async function submit() {
  await login({ email: email.value, password: password.value })
}
</script>

<template>
  <button :disabled="pending" @click="submit">
    Đăng nhập
  </button>
  <button v-if="isAuthenticated" @click="logout">
    Đăng xuất {{ user?.email }}
  </button>
  <p v-if="error">{{ error.message }}</p>
</template>
```

Nuxt module tự quản lý SSR session. Code ứng dụng chỉ dùng `useAuth()` hoặc original client do `useEnfyra()` trả về.

Bắt đầu OAuth provider đã được bật từ browser action:

```vue
<button @click="oauthLogin('google')">
  Tiếp tục với Google
</button>
```

## Next.js

Sau khi cấu hình `@enfyra/sdk-next`, dùng providerless client hook trong Client Component:

```tsx
'use client'

import { useAuth } from '@enfyra/sdk-next/client'

export function AccountButton() {
  const { user, isAuthenticated, pending, login, logout } = useAuth()

  if (isAuthenticated) {
    return <button onClick={() => logout()}>Đăng xuất {user?.email}</button>
  }

  return (
    <button
      disabled={pending}
      onClick={() => login({ email: 'me@example.com', password: 'secret' })}
    >
      Đăng nhập
    </button>
  )
}
```

Không cần Provider. Hook khôi phục cookie session sau khi hydrate thông qua `/api/enfyra/me`. Server Component và Server Action dùng API theo từng request trong [hướng dẫn Next.js](./next.md).

## Vue

Install plugin một lần:

```ts
createApp(App)
  .use(createEnfyra({
    baseUrl: 'https://admin.example.com',
  }))
  .mount('#app')
```

Sau đó gọi `useAuth()` bên trong component:

```vue
<script setup lang="ts">
import { useAuth } from '@enfyra/sdk-vue'

const { user, isAuthenticated, pending, error, login, logout } = useAuth()

async function submit() {
  await login({ email: email.value, password: password.value })
}
</script>
```

`login()` trả `null` khi thất bại và đặt `error`. Composable cập nhật `user` và `isAuthenticated` sau khi đăng nhập thành công.

## React

Thêm provider một lần:

```tsx
<EnfyraProvider config={{ baseUrl: 'https://admin.example.com' }}>
  <App />
</EnfyraProvider>
```

Sau đó dùng hook trong component:

```tsx
import { useAuth } from '@enfyra/sdk-react'

function LoginForm() {
  const { user, isAuthenticated, pending, error, login, logout } = useAuth()

  async function submit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault()
    await login({ email, password })
  }

  // Render form hoặc signed-in user tại đây.
}
```

Hook load current user khi mount. `login()` trả `null` khi thất bại và expose lỗi qua `error`.

## API token phía server

Chỉ dùng API token trong trusted server code. Không đưa PAT vào browser bundle, public environment variable, source file hoặc client-rendered component.

```ts
const enfyra = new EnfyraClient({
  baseUrl: process.env.ENFYRA_APP_URL!,
})

await enfyra.auth.exchangeApiToken(process.env.ENFYRA_API_TOKEN!)

const projects = await enfyra
  .from('projects')
  .select(['id', 'name'])
  .execute()
```

Tiếp tục dùng cùng client instance sau khi exchange để các request tiếp theo dùng authenticated session.

## Public route

Public route không cần login và không cần client config riêng:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
})

const { data } = await enfyra.get('/public/articles')
```

Route phải được cho phép public access trong Enfyra. SDK config không thể biến protected route thành public.

## Application flow nên dùng

1. Cấu hình package một lần khi application khởi động.
2. Load current user khi application hoặc protected area bắt đầu.
3. Redirect hoặc hiện login form nếu không có user.
4. Dùng cùng SDK client cho authenticated query và mutation.
5. Gọi `logout()` và xóa application-specific user state khi user đăng xuất.

Không log password, API token, access token, refresh token hoặc toàn bộ authentication response.

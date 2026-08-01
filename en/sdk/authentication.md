# Authentication

Configure the Enfyra App URL once, then use the authentication API exposed by your SDK package. You do not need to select or manage a session transport.

## Core Client

Create the client and call `auth.login()`:

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

The client keeps the session created by `login()` and refreshes it when required. Sign out with the same client instance:

```ts
await enfyra.auth.logout()
```

Core authentication methods throw an `EnfyraError` when the request fails. Handle the error at the form or service boundary that started the action:

```ts
import { isEnfyraError } from '@enfyra/sdk-core'

try {
  await enfyra.auth.login({ email, password })
} catch (error) {
  if (isEnfyraError(error) && error.statusCode === 401) {
    showMessage('Email or password is incorrect')
  } else {
    showMessage('Unable to sign in')
  }
}
```

## Nuxt

After installing `@enfyra/sdk-nuxt`, use the auto-imported `useAuth()` composable:

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
    Sign in
  </button>
  <button v-if="isAuthenticated" @click="logout">
    Sign out {{ user?.email }}
  </button>
  <p v-if="error">{{ error.message }}</p>
</template>
```

The Nuxt module manages the SSR session internally. Application code only uses `useAuth()` or the original client returned by `useEnfyra()`.

Start an enabled OAuth provider from a browser action:

```vue
<button @click="oauthLogin('google')">
  Continue with Google
</button>
```

## Next.js

After configuring `@enfyra/sdk-next`, use its providerless client hook from a Client Component:

```tsx
'use client'

import { useAuth } from '@enfyra/sdk-next/client'

export function AccountButton() {
  const { user, isAuthenticated, pending, login, logout } = useAuth()

  if (isAuthenticated) {
    return <button onClick={() => logout()}>Sign out {user?.email}</button>
  }

  return (
    <button
      disabled={pending}
      onClick={() => login({ email: 'me@example.com', password: 'secret' })}
    >
      Sign in
    </button>
  )
}
```

No Provider is required. The hook restores the cookie session after hydration through `/api/enfyra/me`. Server Components and Server Actions use the request-scoped APIs described in the [Next.js guide](./next.md).

## Vue

Install the plugin once:

```ts
createApp(App)
  .use(createEnfyra({
    baseUrl: 'https://admin.example.com',
  }))
  .mount('#app')
```

Then call `useAuth()` inside a component:

```vue
<script setup lang="ts">
import { useAuth } from '@enfyra/sdk-vue'

const { user, isAuthenticated, pending, error, login, logout } = useAuth()

async function submit() {
  await login({ email: email.value, password: password.value })
}
</script>
```

`login()` returns `null` on failure and sets `error`. The composable updates `user` and `isAuthenticated` after a successful login.

## React

Add the provider once:

```tsx
<EnfyraProvider config={{ baseUrl: 'https://admin.example.com' }}>
  <App />
</EnfyraProvider>
```

Then use the hook in a component:

```tsx
import { useAuth } from '@enfyra/sdk-react'

function LoginForm() {
  const { user, isAuthenticated, pending, error, login, logout } = useAuth()

  async function submit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault()
    await login({ email, password })
  }

  // Render the form or the signed-in user here.
}
```

The hook loads the current user when it mounts. `login()` returns `null` on failure and exposes the failure through `error`.

## Server-Side API Tokens

Use an API token only in trusted server code. Never include a PAT in a browser bundle, public environment variable, source file, or client-rendered component.

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

Keep using the same client instance after the exchange so subsequent requests use the authenticated session.

## Public Routes

Public routes require no login and no special client configuration:

```ts
const enfyra = new EnfyraClient({
  baseUrl: 'https://admin.example.com',
})

const { data } = await enfyra.get('/public/articles')
```

The route must already allow public access in Enfyra. SDK configuration cannot make a protected route public.

## Recommended Application Flow

1. Configure the package once at application startup.
2. Load the current user when the application or protected area starts.
3. Redirect or show a login form when no user is available.
4. Use the same SDK client for authenticated queries and mutations.
5. Call `logout()` and clear application-specific user state when the user signs out.

Never log passwords, API tokens, access tokens, refresh tokens, or complete authentication responses.

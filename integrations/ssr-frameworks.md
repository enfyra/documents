# SSR Frameworks

Connect a Nuxt, Next.js, SvelteKit, or Remix app to Enfyra with same-origin REST, OAuth cookies, refresh-token support, and Socket.IO.

## Integration Model

```text
Browser
  -> your SSR app origin
  -> local Enfyra proxy prefix
  -> Enfyra app /api and /ws bridge
  -> Enfyra server
```

Do not call the raw Enfyra server from browser code. Browser code should call your app origin only.

## Required Proxies

Expose two paths from your app:

| Local path | Forward to | Purpose |
|------------|------------|---------|
| `/enfyra/**` | `https://demo.enfyra.io/api/**` | REST, auth, OAuth, files, GraphQL |
| `/socket.io/**` | `https://demo.enfyra.io/ws/socket.io/**` | Socket.IO transport |

You may choose another REST prefix, but keep it stable. If your prefix is `/enfyra`, OAuth must pass `cookieBridgePrefix=/enfyra`.

## Shared Browser Code

After the proxy exists, browser calls are the same in every SSR framework.

Password login:

```ts
await fetch("/enfyra/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  credentials: "include",
  body: JSON.stringify({ email, password, remember: true }),
})
```

Current user:

```ts
const me = await fetch("/enfyra/me", {
  credentials: "include",
})
```

OAuth:

```ts
const redirect = new URL("/chat", window.location.origin)
const url = new URL("/enfyra/auth/google", window.location.origin)

url.searchParams.set("redirect", redirect.toString())
url.searchParams.set("cookieBridgePrefix", "/enfyra")

window.location.href = url.toString()
```

Socket.IO:

```ts
import { io } from "socket.io-client"

const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true,
})
```

`/chat` is the Enfyra websocket namespace. `/socket.io` is the transport path your SSR app proxies to the Enfyra app websocket bridge.

## OAuth Flow

```text
User clicks Google
  -> /enfyra/auth/google?redirect=...&cookieBridgePrefix=/enfyra
  -> Enfyra redirects to Google
  -> Google returns to Enfyra callback
  -> Enfyra redirects through /enfyra/auth/set-cookies
  -> the proxy response sets cookies on your app origin
  -> browser returns to redirect
```

The user does not handle tokens manually.

## Nuxt

Use `routeRules`.

```ts
export default defineNuxtConfig({
  routeRules: {
    "/enfyra/**": {
      proxy: {
        to: "https://demo.enfyra.io/api/**",
        fetchOptions: { redirect: "manual" },
      },
    },
    "/socket.io/**": {
      proxy: {
        to: "https://demo.enfyra.io/ws/socket.io/**",
      },
    },
  },
})
```

`redirect: "manual"` keeps OAuth and set-cookie redirects under your app origin.

## Next.js

Use `rewrites()`.

```js
const nextConfig = {
  async rewrites() {
    return [
      {
        source: "/enfyra/:path*",
        destination: "https://demo.enfyra.io/api/:path*",
      },
      {
        source: "/socket.io/:path*",
        destination: "https://demo.enfyra.io/ws/socket.io/:path*",
      },
    ]
  },
}

export default nextConfig
```

For server components, forward the incoming cookie header when fetching through your own app URL:

```ts
import { cookies } from "next/headers"

const res = await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/enfyra/me`, {
  headers: { cookie: (await cookies()).toString() },
  cache: "no-store",
})
```

## SvelteKit

Use a server hook for the REST/auth prefix.

```ts
import type { Handle } from "@sveltejs/kit"

const ENFYRA_API = "https://demo.enfyra.io/api"

export const handle: Handle = async ({ event, resolve }) => {
  if (!event.url.pathname.startsWith("/enfyra/")) {
    return resolve(event)
  }

  const upstream = new URL(
    event.url.pathname.replace(/^\/enfyra/, ""),
    ENFYRA_API,
  )
  upstream.search = event.url.search

  return fetch(upstream, {
    method: event.request.method,
    headers: event.request.headers,
    body:
      event.request.method === "GET" || event.request.method === "HEAD"
        ? undefined
        : event.request.body,
    redirect: "manual",
  })
}
```

Configure your deployment proxy to forward `/socket.io/**` to `https://demo.enfyra.io/ws/socket.io/**`.

## Remix

Create a catch-all resource route for `/enfyra/*`.

```ts
import type { ActionFunctionArgs, LoaderFunctionArgs } from "@remix-run/node"

const ENFYRA_API = "https://demo.enfyra.io/api"

async function proxy(request: Request, params: { "*": string }) {
  const url = new URL(request.url)
  const upstream = new URL(`/${params["*"] ?? ""}`, ENFYRA_API)
  upstream.search = url.search

  return fetch(upstream, {
    method: request.method,
    headers: request.headers,
    body:
      request.method === "GET" || request.method === "HEAD"
        ? undefined
        : request.body,
    redirect: "manual",
  })
}

export const loader = ({ request, params }: LoaderFunctionArgs) =>
  proxy(request, params)

export const action = ({ request, params }: ActionFunctionArgs) =>
  proxy(request, params)
```

Configure your hosting proxy to forward `/socket.io/**` to `https://demo.enfyra.io/ws/socket.io/**`.

## Common Mistakes

### Missing cookieBridgePrefix

If the app proxy prefix is `/enfyra`, OAuth must include:

```text
cookieBridgePrefix=/enfyra
```

Without this, Enfyra uses the default `/api` cookie bridge, which is correct for the Enfyra app itself but wrong for a third-party app using `/enfyra`.

### Calling the backend host from browser code

Avoid:

```ts
fetch("https://raw-server.example.com/project_task")
```

Use:

```ts
fetch("/enfyra/project_task", { credentials: "include" })
```

### Connecting to `/ws/chat`

Do not treat Enfyra realtime as a raw WebSocket path. Use Socket.IO namespace plus transport path:

```ts
io("/chat", { path: "/socket.io", withCredentials: true })
```

## Next Step

After your framework is integrated, build a real feature with [Third-Party Chat App Example](../examples/third-party-chat-app.md).

# SSR Frameworks

Connect a Nuxt, Next.js, Angular, SvelteKit, or Remix app to Enfyra with same-origin REST, OAuth cookies, refresh-token support, and Socket.IO.

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

OAuth starts on the Enfyra app URL, then returns through your app proxy:

```ts
const redirect = new URL("/chat", window.location.origin)
const url = new URL("/api/auth/google", "https://demo.enfyra.io")

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
  reconnection: false,
  transports: ["polling"],
  upgrade: false,
})
```

`/chat` is the Enfyra websocket namespace. `/socket.io` is the transport path your SSR app proxies to the Enfyra app websocket bridge.

The demo chat apps use polling-only Socket.IO because it works consistently through simple SSR framework rewrites and reverse proxies. If your deployment proxy supports websocket upgrade cleanly, you can enable websocket transport later.

## OAuth Flow

```text
User clicks Google
  -> https://demo.enfyra.io/api/auth/google?redirect=...&cookieBridgePrefix=/enfyra
  -> Enfyra redirects to Google
  -> Google returns to Enfyra callback
  -> Enfyra redirects through {yourAppOrigin}/enfyra/auth/set-cookies
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
        source: "/socket.io/",
        destination: "https://demo.enfyra.io/ws/socket.io/",
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

## Angular

Use the Angular dev-server proxy for local development and configure the same routes in your production reverse proxy.

```json
{
  "/enfyra/**": {
    "target": "https://demo.enfyra.io/api",
    "secure": true,
    "changeOrigin": true,
    "pathRewrite": {
      "^/enfyra": ""
    }
  },
  "/socket.io/**": {
    "target": "https://demo.enfyra.io/ws",
    "secure": true,
    "changeOrigin": true,
    "ws": true
  }
}
```

Add the proxy file to the Angular serve target:

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "serve": {
          "options": {
            "proxyConfig": "src/proxy.conf.json"
          }
        }
      }
    }
  }
}
```

Use an HTTP interceptor so every Enfyra request sends cookies:

```ts
import { ApplicationConfig, inject } from "@angular/core"
import { HttpInterceptorFn, provideHttpClient, withInterceptors } from "@angular/common/http"
import { CanActivateFn, provideRouter, Router } from "@angular/router"
import { catchError, map, of } from "rxjs"

import { routes } from "./app.routes"
import { EnfyraAuthService } from "./enfyra-auth.service"

export const enfyraCredentialsInterceptor: HttpInterceptorFn = (req, next) => {
  if (!req.url.startsWith("/enfyra/")) return next(req)
  return next(req.clone({ withCredentials: true }))
}

export const requireUserGuard: CanActivateFn = () => {
  const auth = inject(EnfyraAuthService)
  const router = inject(Router)

  return auth.loadMe().pipe(
    map(user => user ? true : router.createUrlTree(["/login"])),
    catchError(() => of(router.createUrlTree(["/login"]))),
  )
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([enfyraCredentialsInterceptor])),
    provideRouter(routes),
  ],
}
```

Create a small auth service around `/enfyra/login`, `/enfyra/me`, and `/enfyra/logout`:

```ts
import { Injectable, signal } from "@angular/core"
import { HttpClient } from "@angular/common/http"
import { Observable, switchMap, tap } from "rxjs"

type EnfyraUser = {
  id: string | number
  email?: string
}

@Injectable({ providedIn: "root" })
export class EnfyraAuthService {
  readonly user = signal<EnfyraUser | null>(null)

  constructor(private readonly http: HttpClient) {}

  login(email: string, password: string): Observable<EnfyraUser | null> {
    return this.http.post("/enfyra/login", { email, password, remember: true }).pipe(
      switchMap(() => this.loadMe()),
    )
  }

  loadMe(): Observable<EnfyraUser | null> {
    return this.http.get<EnfyraUser | null>("/enfyra/me").pipe(
      tap(user => this.user.set(user)),
    )
  }

  logout(): Observable<unknown> {
    return this.http.post("/enfyra/logout", {}).pipe(
      tap(() => this.user.set(null)),
    )
  }

  startGoogleOAuth(returnPath = "/") {
    const redirect = new URL(returnPath, window.location.origin)
    const url = new URL("/api/auth/google", "https://demo.enfyra.io")
    url.searchParams.set("redirect", redirect.toString())
    url.searchParams.set("cookieBridgePrefix", "/enfyra")
    window.location.href = url.toString()
  }
}
```

Create one Socket.IO connection in an app-level service after auth is known:

```ts
import { computed, effect, Injectable, signal } from "@angular/core"
import { io, type Socket } from "socket.io-client"

import { EnfyraAuthService } from "./enfyra-auth.service"

@Injectable({ providedIn: "root" })
export class EnfyraRealtimeService {
  private socket: Socket | null = null
  private readonly connected = signal(false)
  readonly isConnected = computed(() => this.connected())

  constructor(private readonly auth: EnfyraAuthService) {
    effect(() => {
      const user = this.auth.user()
      if (user) this.connect()
      else this.disconnect()
    })
  }

  connect() {
    if (this.socket) return this.socket

    this.socket = io("/chat", {
      path: "/socket.io",
      withCredentials: true,
      reconnection: true,
      reconnectionAttempts: Infinity,
      reconnectionDelay: 2000,
      reconnectionDelayMax: 30000,
      transports: ["polling"],
      upgrade: false,
    })

    this.socket.on("connect", () => this.connected.set(true))
    this.socket.on("disconnect", () => this.connected.set(false))
    return this.socket
  }

  disconnect() {
    this.socket?.disconnect()
    this.socket = null
    this.connected.set(false)
  }

  onMessage(handler: (event: unknown) => void) {
    const activeSocket = this.connect()
    activeSocket.on("chat:message", handler)
    return () => activeSocket.off("chat:message", handler)
  }
}
```

Route guards are for browser navigation only. Keep Enfyra route permissions, guards, and owner checks on the server side.

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

### Starting OAuth from the wrong origin

The production demo chat apps start OAuth on the Enfyra app URL:

```ts
new URL("/api/auth/google", "https://demo.enfyra.io")
```

They do not start Google OAuth at `/enfyra/auth/google`. The local `/enfyra/**` proxy is still required for the later `/enfyra/auth/set-cookies` bridge.

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

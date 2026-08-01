---
slug: framework-ssr
---

# Framework SSR

Kết nối app SSR hoặc CSR với Enfyra bằng REST cùng origin, OAuth cookie, refresh-token support và Socket.IO.

## Framework đã có SDK — dùng SDK thay vì tự cấu hình proxy

Nếu app của bạn dùng một trong các framework dưới đây, hãy bắt đầu bằng SDK package tương ứng. Nuxt và Next.js tự cấu hình framework proxy. Vue và React chỉ chạy CSR nên vẫn cần same-origin proxy; hãy làm theo hướng dẫn SDK của package trước khi dùng phần tham khảo thủ công bên dưới. Khả năng realtime và cách cấu hình phụ thuộc vào từng package.

| Framework | Package | Hướng dẫn |
|---|---|---|
| Nuxt 3/4 | `@enfyra/sdk-nuxt` | [SDK Nuxt](../sdk/nuxt.md) |
| Next.js App Router | `@enfyra/sdk-next` | [SDK Next.js](../sdk/next.md) |
| Vue 3 (CSR) | `@enfyra/sdk-vue` | [SDK Vue](../sdk/vue.md) |
| React (CSR) | `@enfyra/sdk-react` | [SDK React](../sdk/react.md) |
| Node.js / worker / edge | `@enfyra/sdk-core` | [SDK Core Client](../sdk/core-client.md) |

Chỉ tiếp tục đọc phần bên dưới khi framework của bạn **chưa có SDK** (Angular, SvelteKit, Remix) hoặc bạn cần hiểu cơ chế proxy/cookie mà SDK đang trừu tượng hóa.

---

## Mô hình tích hợp

```text
Trình duyệt
  -> origin của ứng dụng SSR
  -> tiền tố proxy Enfyra cục bộ
  -> cầu nối /api và /ws của ứng dụng Enfyra
  -> máy chủ Enfyra
```

Không gọi raw Enfyra server trong browser code. Browser chỉ gọi origin của chính app bạn.

## Proxy cần có

Expose hai path từ app:

| Local path | Chuyển tiếp đến | Mục đích |
|------------|-----------------|----------|
| `/enfyra/**` | `https://demo.enfyra.io/api/**` | REST, auth, OAuth, file, GraphQL |
| `/socket.io/**` | `https://demo.enfyra.io/ws/socket.io/**` | Socket.IO transport |

Bạn có thể chọn REST prefix khác nhưng phải giữ ổn định. Nếu là `/enfyra`, OAuth bắt buộc truyền `cookieBridgePrefix=/enfyra`.

## Browser code dùng chung

Khi proxy đã có, browser call giống nhau trong mọi SSR framework.

Đăng nhập password:

```ts
await fetch("/enfyra/login", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  credentials: "include",
  body: JSON.stringify({ email, password, remember: true }),
})
```

Lấy current user:

```ts
const me = await fetch("/enfyra/me", {
  credentials: "include",
})
```

Bắt đầu OAuth ở Enfyra app URL rồi quay về qua app proxy:

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

`/chat` là Enfyra websocket namespace; `/socket.io` là transport path SSR app proxy đến Enfyra websocket bridge. Demo chat app dùng polling-only vì hoạt động ổn định qua rewrite/reverse proxy đơn giản; khi deployment proxy hỗ trợ WebSocket upgrade tốt, bạn có thể bật websocket transport.

## Luồng OAuth

```text
Người dùng bấm đăng nhập Google
  -> https://demo.enfyra.io/api/auth/google?redirect=...&cookieBridgePrefix=/enfyra
  -> Enfyra chuyển hướng đến Google
  -> Google quay về callback của Enfyra
  -> Enfyra chuyển hướng qua {yourAppOrigin}/enfyra/auth/set-cookies
  -> phản hồi proxy đặt cookie trên origin của ứng dụng
  -> trình duyệt quay lại URL redirect
```

Người dùng không tự xử lý token.

## Nuxt

> **Đã có SDK:** Nếu bạn đang bắt đầu project Nuxt mới với Enfyra, dùng [`@enfyra/sdk-nuxt`](../sdk/nuxt.md) thay vì tự cấu hình proxy. Phần bên dưới chỉ dành cho trường hợp cần hiểu hoặc tùy chỉnh proxy thủ công.

Dùng `routeRules`:

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

`redirect: "manual"` giữ OAuth và set-cookie redirect dưới app origin.

## Next.js

> **Đã có SDK:** Nếu bạn đang bắt đầu project Next.js mới với Enfyra, dùng [`@enfyra/sdk-next`](../sdk/next.md) thay vì tự cấu hình rewrites. Phần bên dưới chỉ dành cho trường hợp cần hiểu hoặc tùy chỉnh proxy thủ công.

Dùng `rewrites()`:

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

Với server component, forward incoming cookie khi fetch qua URL app:

```ts
import { cookies } from "next/headers"

const res = await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/enfyra/me`, {
  headers: { cookie: (await cookies()).toString() },
  cache: "no-store",
})
```

## Angular

> **Chưa có SDK.** Angular chưa có Enfyra SDK package. Dùng cấu hình proxy thủ công bên dưới.

Dùng Angular dev-server proxy ở local và cấu hình route tương tự trong production reverse proxy:

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

Gắn proxy file vào Angular serve target:

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

Dùng HTTP interceptor để mọi Enfyra request gửi cookie:

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

Tạo auth service nhỏ xoay quanh `/enfyra/login`, `/enfyra/me`, `/enfyra/logout`; sau khi auth đã biết, tạo một Socket.IO connection trong app-level service. Route guard chỉ bảo vệ browser navigation; Enfyra route permission, guard và owner check vẫn phải nằm ở server.

Ví dụ khởi tạo OAuth trong Angular service:

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

## SvelteKit

> **Chưa có SDK.** SvelteKit chưa có Enfyra SDK package. Dùng server hook proxy thủ công bên dưới.

Dùng server hook cho REST/auth prefix:

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

Cấu hình deployment proxy để chuyển `/socket.io/**` tới `https://demo.enfyra.io/ws/socket.io/**`.

## Remix

> **Chưa có SDK.** Remix chưa có Enfyra SDK package. Dùng catch-all resource route thủ công bên dưới.

Tạo catch-all resource route cho `/enfyra/*`:

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

Cấu hình hosting proxy để chuyển `/socket.io/**` tới `https://demo.enfyra.io/ws/socket.io/**`.

## Lỗi thường gặp

### Thiếu cookieBridgePrefix

Proxy prefix `/enfyra` thì OAuth phải có:

```text
cookieBridgePrefix=/enfyra
```

Thiếu nó, Enfyra dùng cookie bridge `/api` mặc định, đúng cho Enfyra app nhưng sai với third-party app dùng `/enfyra`.

### Bắt đầu OAuth sai origin

Demo chat app production bắt đầu OAuth trên Enfyra app URL:

```ts
new URL("/api/auth/google", "https://demo.enfyra.io")
```

Không bắt đầu Google OAuth tại `/enfyra/auth/google`. Local `/enfyra/**` proxy vẫn cần cho bridge `/enfyra/auth/set-cookies` về sau.

### Browser code gọi backend host

Tránh:

```ts
fetch("https://raw-server.example.com/project_task")
```

Dùng:

```ts
fetch("/enfyra/project_task", { credentials: "include" })
```

### Kết nối `/ws/chat`

Đừng coi Enfyra realtime là raw WebSocket path. Dùng Socket.IO namespace và transport path:

```ts
io("/chat", { path: "/socket.io", withCredentials: true })
```

## Bước tiếp theo

Sau khi tích hợp framework, xây feature thực với [Ví dụ third-party chat app](../examples/third-party-chat-app.md).

# App Examples

Small examples for Enfyra App extensions and external SSR apps.

## Extension Data

### Load Rows With `useApi`

```vue
<script setup>
const { data, execute } = useApi("/post", {
  query: { fields: "id,title", limit: 10 }
})

onMounted(() => execute())
</script>
```

### Create Row

```vue
<script setup>
const title = ref("")
const toast = useToast()

async function save() {
  await $fetch("/api/post", {
    method: "POST",
    body: { title: title.value }
  })
  toast.add({ title: "Saved" })
}
</script>
```

### Show Current User

```vue
<script setup>
const { me, fetchUser } = useAuth()

onMounted(() => fetchUser())
</script>

<template>
  <p>{{ me?.email }}</p>
</template>
```

## Extension Shell

### Register Page Header

```vue
<script setup>
const { registerPageHeader } = usePageHeaderRegistry()

registerPageHeader({
  title: "Posts",
  description: "Manage content",
  leadingIcon: "",
  variant: "minimal"
})
</script>
```

### Register Header Action

```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry()

registerHeaderActions({
  id: "refresh-posts",
  label: "Refresh",
  icon: "",
  color: "neutral",
  variant: "soft",
  onClick: () => refresh()
})
</script>
```

### Register Menu And Account Notifications

```vue
<script setup>
const unread = ref(0)
const count = computed(() => unread.value > 0 ? String(unread.value) : null)

const { register: registerAccountPanel } = useAccountPanelRegistry()
registerAccountPanel({
  id: "notifications",
  label: "Notifications",
  icon: computed(() => unread.value > 0 ? "" : ""),
  count,
  badgeColor: "error",
  order: 20
})

const { register: registerMenuNotification } = useMenuNotificationRegistry()
watchEffect(() => {
  registerMenuNotification({
    id: "notifications-menu",
    target: { path: "/notifications" },
    value: count.value,
    color: unread.value > 0 ? "error" : "neutral",
    title: "Unread notifications"
  })
})
</script>
```

### Gate A Button By Permission

```vue
<template>
  <PermissionGate :condition="{ and: [{ route: '/post', methods: ['POST'] }] }">
    <UButton type="button">Create</UButton>
  </PermissionGate>
</template>
```

## Extension Forms

### Use `FormEditor`

```vue
<template>
  <FormEditor
    table-name="post"
    :record="record"
    :includes="['title', 'status']"
  />
</template>
```

### Open A Modal

```vue
<template>
  <UButton type="button" @click="open = true">Open</UButton>
  <CommonModal
    v-model:open="open"
    :cancel-action="{ label: 'Close', tone: 'neutral', onClick: () => (open = false) }"
  >
    <template #body>Modal content</template>
  </CommonModal>
</template>

<script setup>
const open = ref(false)
</script>
```

## SSR App Proxy

### Nuxt Proxy

```ts
export default defineNuxtConfig({
  routeRules: {
    "/enfyra/**": {
      proxy: `${process.env.API_URL}/**`
    }
  }
})
```

### Password Login

```ts
await $fetch("/enfyra/login", {
  method: "POST",
  body: { email, password }
})
```

### Fetch Current User

```ts
const user = await $fetch("/enfyra/me")
```

### Logout

```ts
await $fetch("/enfyra/logout", {
  method: "POST"
})
```

## SSR Realtime

### Socket.IO Client

```ts
import { io } from "socket.io-client"

const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true,
  transports: ["polling"],
  upgrade: false
})
```

### Listen For Event

```ts
socket.on("chat:message", (payload) => {
  messages.value.push(payload.message)
})
```

### Send Event

```ts
socket.emit("chat:message", {
  conversationId,
  text
})
```

---
slug: vi-du-ung-dung
---

# Ví dụ ứng dụng

Các ví dụ nhỏ cho Enfyra App extension và ứng dụng SSR bên ngoài.

## Dữ liệu extension

### Tải các dòng bằng `useApi`

```vue
<script setup>
const { data, execute } = useApi("/post", {
  query: { fields: "id,title", limit: 10 }
})

onMounted(() => execute())
</script>
```

### Tạo dòng

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

### Hiển thị người dùng hiện tại

```vue
<script setup>
const { me, fetchUser } = useAuth()

onMounted(() => fetchUser())
</script>

<template>
  <p>{{ me?.email }}</p>
</template>
```

## Khung extension

### Đăng ký tiêu đề trang

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

### Đăng ký thao tác ở header

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

### Đăng ký thông báo menu và tài khoản

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

### Phân quyền cho một nút

```vue
<template>
  <PermissionGate :condition="{ and: [{ route: '/post', methods: ['POST'] }] }">
    <UButton type="button">Create</UButton>
  </PermissionGate>
</template>
```

## Form extension

### Dùng `FormEditor`

```vue
<template>
  <FormEditor
    table-name="post"
    :record="record"
    :includes="['title', 'status']"
  />
</template>
```

### Mở modal

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

## Proxy cho ứng dụng SSR

### Proxy Nuxt

```ts
export default defineNuxtConfig({
  routeRules: {
    "/enfyra/**": {
      proxy: `${process.env.API_URL}/**`
    }
  }
})
```

### Đăng nhập bằng mật khẩu

```ts
await $fetch("/enfyra/login", {
  method: "POST",
  body: { email, password }
})
```

### Lấy người dùng hiện tại

```ts
const user = await $fetch("/enfyra/me")
```

### Đăng xuất

```ts
await $fetch("/enfyra/logout", {
  method: "POST"
})
```

## Realtime cho SSR

### Socket.IO client

```ts
import { io } from "socket.io-client"

const socket = io("/chat", {
  path: "/socket.io",
  withCredentials: true,
  transports: ["polling"],
  upgrade: false
})
```

### Lắng nghe sự kiện

```ts
socket.on("chat:message", (payload) => {
  messages.value.push(payload.message)
})
```

### Gửi sự kiện

```ts
socket.emit("chat:message", {
  conversationId,
  text
})
```

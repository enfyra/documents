---
slug: ung-dung/dau-trang
---

# Hệ thống đầu trang

Hệ thống đầu trang cho phép bạn đăng ký nội dung đầu trang tùy chỉnh để hiển thị ở phía trên mỗi trang, giúp người dùng nắm được ngữ cảnh và thông tin điều hướng.

## Mục lục

- [Tổng quan](#tong-quan)
- [Cách dùng cơ bản](#cach-dung-co-ban)
- [Các biến thể đầu trang](#cac-bien-the-dau-trang)
- [Tùy chọn gradient](#tuy-chon-gradient)
- [Biểu tượng đầu dòng](#bieu-tuong-dau-dong)
- [Hiển thị số liệu](#hien-thi-so-lieu)
- [Tính năng nâng cao](#tinh-nang-nang-cao)
- [Ví dụ thực tế](#vi-du-thuc-te)
- [Thực hành tốt](#thuc-hanh-tot)
- [Tham chiếu API](#tham-chieu-api)

## Tổng quan

Đầu trang cung cấp:

- **Tiêu đề và mô tả**: Giúp người dùng hiểu rõ ngữ cảnh của trang.
- **Biểu tượng đầu dòng**: Biểu tượng không bắt buộc bên cạnh tiêu đề; cũng có thể được lấy từ menu trên thanh bên theo tuyến hiện tại.
- **Hiển thị số liệu**: Cho biết nhanh các chỉ số quan trọng.
- **Header native**: Tiêu đề không có frame riêng, nằm trong body cuộn của dashboard và thẳng hàng với container nội dung 75rem (1200px) được căn giữa. Không có border hay padding riêng; thanh điều hướng chính vẫn đứng yên.
- **Giao diện theo theme**: Nền main/header theo theme của app; biểu tượng đầu dòng dùng màu primary đang chọn.
- **Tự động dọn dẹp**: Nội dung đầu trang tự động được xóa khi chuyển tuyến.

## Cách dùng cơ bản

### Đầu trang đơn giản

```vue
<script setup>
const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: "Dashboard",
  description: "Welcome to your dashboard",
  variant: "minimal",
  gradient: "cyan"
});
</script>
```

### Đầu trang có mô tả

```vue
<script setup>
const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: "User Management",
  description: "Manage user accounts, roles, and permissions",
  variant: "default",
  gradient: "blue"
});
</script>
```

## Các biến thể đầu trang

### Biến thể mặc định

Bố cục tiêu chuẩn gồm tiêu đề, mô tả và số liệu không bắt buộc.

```vue
registerPageHeader({
  title: "Collections",
  description: "Manage your data collections",
  variant: "default",
  gradient: "purple"
});
```

### Biến thể tối giản

Dùng cùng bố cục header native như `default`. Bỏ `description` và `stats` nếu chỉ cần tiêu đề.

```vue
registerPageHeader({
  title: "Dashboard",
  variant: "minimal",
  gradient: "cyan"
});
```

### Biến thể tập trung vào số liệu

Nhấn mạnh phần hiển thị số liệu.

```vue
registerPageHeader({
  title: "Analytics",
  description: "View your analytics data",
  variant: "stats-focus",
  gradient: "blue",
  stats: [
    { label: "Total Users", value: 1250 },
    { label: "Active Sessions", value: 42 }
  ]
});
```

## Tùy chọn gradient

`gradient` nhận `purple`, `blue`, `cyan` hoặc `none`, mặc định là `none`. Header native luôn render nền main theo theme với mọi giá trị; tùy chọn này không tạo dải màu hoặc ô biểu tượng riêng. Chọn màu primary qua phần Appearance của app.

```vue
// Native theme surface
registerPageHeader({
  title: "Settings",
  gradient: "purple"
});

// Default appearance
registerPageHeader({
  title: "Simple Page",
  gradient: "none"
});
```

## Biểu tượng đầu dòng

- **`leadingIcon`**: Tên biểu tượng Iconify / Nuxt UI, ví dụ `i-lucide-layout-dashboard`. Biểu tượng dùng màu primary bên cạnh tiêu đề, không có ô bao riêng.
- **`hideLeadingIcon: true`**: Luôn ẩn biểu tượng đầu dòng, kể cả khi menu có biểu tượng cho đường dẫn hiện tại.
- **Mặc định**: Nếu bỏ qua `leadingIcon` và không ẩn biểu tượng, ứng dụng sẽ thử dùng **biểu tượng menu** đã đăng ký cho tuyến hiện tại qua `useMenuRegistry` / `findMenuIconForPath`.

## Hiển thị số liệu

### Số liệu cơ bản

```vue
const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: "User Manager",
  stats: [
    { label: "Total Users", value: 1250 },
    { label: "Active", value: 892 },
    { label: "Pending", value: 45 }
  ]
});
```

### Số liệu phản ứng

Cập nhật đầu trang mỗi khi các giá trị nguồn thay đổi, tương tự cách xử lý tiêu đề động:

```vue
<script setup>
const totalCount = ref(0);
const activeCount = ref(0);

const { registerPageHeader } = usePageHeaderRegistry();

watch([totalCount, activeCount], () => {
  registerPageHeader({
    title: "Dashboard",
    stats: [
      { label: "Total", value: totalCount.value },
      { label: "Active", value: activeCount.value },
    ],
  });
}, { immediate: true });

onMounted(async () => {
  const data = await fetchStats();
  totalCount.value = data.total;
  activeCount.value = data.active;
});
</script>
```

## Tính năng nâng cao

### Đầu trang có điều kiện

```vue
<script setup>
const route = useRoute();
const { registerPageHeader } = usePageHeaderRegistry();

// Only show header on specific route
if (route.name === 'dashboard') {
  registerPageHeader({
    title: "Dashboard",
    variant: "minimal"
  });
}
</script>
```

### Tiêu đề và số liệu động

`registerPageHeader` lưu một **ảnh chụp tĩnh** của cấu hình, không giữ các ref phản ứng bên trong đối tượng. Khi tiêu đề hoặc số liệu phụ thuộc vào dữ liệu bất đồng bộ, hãy dùng `watch` hoặc `watchEffect` rồi gọi lại `registerPageHeader` mỗi khi giá trị thay đổi.

```vue
<script setup>
const route = useRoute();
const tableName = computed(() => String(route.params.table ?? ""));
const recordCount = ref(0);

const { registerPageHeader } = usePageHeaderRegistry();

watch([tableName, recordCount], ([name, count]) => {
  if (!name) return;
  registerPageHeader({
    title: `${name} data`,
    description: "Browse and manage records",
    gradient: "purple",
    stats: [
      { label: "Total", value: count },
    ],
  });
}, { immediate: true });
</script>
```

### Kiểm tra trạng thái đầu trang

```vue
<script setup>
const { hasPageHeader, pageHeader } = usePageHeaderRegistry();

// Check if header is registered
if (hasPageHeader.value) {
  console.log('Current header:', pageHeader.value);
}
</script>
```

## Ví dụ thực tế

### 1. Trang bảng dữ liệu

```vue
<template>
  <div class="p-6">
    <!-- Data table content -->
  </div>
</template>

<script setup>
const route = useRoute();
const tableName = computed(() => String(route.params.table ?? ""));
const { data: records } = useApi(() => `/${tableName.value}`);

const { registerPageHeader } = usePageHeaderRegistry();

watch(
  [tableName, records],
  () => {
    const name = tableName.value;
    if (!name) return;
    registerPageHeader({
      title: `${name} data`,
      description: `Browse and manage ${name} records`,
      variant: "default",
      gradient: "blue",
      stats: [
        { label: "Total Records", value: records.value?.meta?.totalCount ?? 0 },
        { label: "Showing", value: records.value?.data?.length ?? 0 },
      ],
    });
  },
  { immediate: true, deep: true },
);
</script>
```

### 2. Trang cài đặt

```vue
<template>
  <div class="settings-page">
    <h2>General Settings</h2>
    <!-- Settings form -->
  </div>
</template>

<script setup>
const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: "Settings",
  description: "Configure application settings and preferences",
  variant: "minimal",
  gradient: "purple"
});
</script>
```

### 3. Bảng điều khiển phân tích

```vue
<template>
  <div class="analytics-dashboard">
    <!-- Charts and analytics -->
  </div>
</template>

<script setup>
const totalUsers = ref(0);
const activeUsers = ref(0);
const revenue = ref(0);

const { registerPageHeader } = usePageHeaderRegistry();

watch([totalUsers, activeUsers, revenue], () => {
  registerPageHeader({
    title: "Analytics",
    description: "Real-time analytics and insights",
    variant: "stats-focus",
    gradient: "cyan",
    stats: [
      { label: "Total Users", value: totalUsers.value.toLocaleString() },
      { label: "Active Now", value: activeUsers.value },
      { label: "Revenue", value: `$${revenue.value.toLocaleString()}` },
    ],
  });
}, { immediate: true });

// Fetch analytics data
onMounted(async () => {
  const { data } = await useApi('/analytics');
  totalUsers.value = data.value.totalUsers;
  activeUsers.value = data.value.activeUsers;
  revenue.value = data.value.revenue;
});
</script>
```

## Thực hành tốt

### 1. Dùng biến thể phù hợp

```javascript
// Dashboard/landing pages
{ variant: "minimal" }

// Data listing pages
{ variant: "default", stats: [...] }

// Analytics pages
{ variant: "stats-focus", stats: [...] }
```

### 2. Để theme quản lý giao diện header

```javascript
{ gradient: "none" }
```

### 3. Viết mô tả ngắn gọn

```javascript
// Good
{ description: "Manage user accounts and permissions" }

// Too long
{ description: "This page allows you to manage all user accounts in the system including creating new users, editing existing users, and configuring their permissions..." }
```

### 4. Dùng số liệu phản ứng

```javascript
// Good - reactive
const stats = computed(() => [
  { label: "Total", value: count.value }
]);

// Avoid - static
const stats = [
  { label: "Total", value: 100 }
];
```

### 5. Xóa đầu trang khi không cần

```javascript
const { clearPageHeader } = usePageHeaderRegistry();

// Clear when navigating away
onUnmounted(() => {
  clearPageHeader();
});
```

## Tham chiếu API

### Interface `PageHeaderConfig`

```typescript
interface PageHeaderConfig {
  title: string;                           // Page title (required)
  description?: string;                    // Page description
  stats?: PageHeaderStat[];                // Statistics to display
  variant?: "default" | "minimal" | "stats-focus";  // Layout variant
  gradient?: "purple" | "blue" | "cyan" | "none";   // Accepted config; appearance remains theme-owned
  leadingIcon?: string;                    // Icon name; if omitted, menu icon for route may be used
  hideLeadingIcon?: boolean;               // If true, no leading icon
}
```

### Interface `PageHeaderStat`

```typescript
interface PageHeaderStat {
  label: string;      // Stat label
  value: string | number;  // Stat value
}
```

### Registry đầu trang

```typescript
const {
  // Read-only current header config
  pageHeader: Readonly<Ref<PageHeaderConfig | null>>,

  // Check if header is registered
  hasPageHeader: ComputedRef<boolean>,

  // Register page header
  registerPageHeader: (config: PageHeaderConfig) => void,

  // Clear page header
  clearPageHeader: () => void
} = usePageHeaderRegistry();
```

## Tóm tắt

Hệ thống đầu trang cung cấp:

**Nội dung đầu trang nhất quán** trên toàn ứng dụng.

**Tiêu đề native cuộn cùng nội dung**, thẳng hàng với container chung.

**Biểu tượng đầu dòng màu primary không bắt buộc**, được lấy từ menu hoặc chỉ định trực tiếp.

**Hiển thị số liệu** cho các chỉ số quan trọng.

**Tự động dọn dẹp** khi chuyển tuyến.

**Cập nhật qua `watch`** khi tiêu đề hoặc số liệu thay đổi.

**Tài liệu liên quan:**

- **[Thao tác đầu trang](./header-actions.md)** - Thêm thao tác vào đầu trang.
- **[Hệ thống biểu mẫu](./form-system.md)** - Tạo biểu mẫu động.

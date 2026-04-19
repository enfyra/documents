# Page Header System

The Page Header System allows you to register custom page headers that appear at the top of each page, providing context and navigation information to users.

## Table of Contents

- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Header Variants](#header-variants)
- [Gradient Options](#gradient-options)
- [Leading icon](#leading-icon)
- [Stats Display](#stats-display)
- [Advanced Features](#advanced-features)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)

## Overview

Page headers provide:
- **Title and Description**: Clear page context for users
- **Leading icon**: Optional icon beside the title (or resolved from the sidebar menu for the current route)
- **Stats Display**: Show key metrics at a glance
- **Visual Variants**: Different layouts for different use cases
- **Accent strip**: When `gradient` is not `none`, a subtle **horizontal tint** runs behind the header row (not a full-page background)
- **Auto-Cleanup**: Headers automatically cleared on route change

## Basic Usage

### Simple Page Header

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

### Page Header with Description

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

## Header Variants

### Default Variant
Standard layout with title, description, and optional stats.

```vue
registerPageHeader({
  title: "Collections",
  description: "Manage your data collections",
  variant: "default",
  gradient: "purple"
});
```

### Minimal Variant
Compact layout focusing on title only.

```vue
registerPageHeader({
  title: "Dashboard",
  variant: "minimal",
  gradient: "cyan"
});
```

### Stats Focus Variant
Emphasizes statistics display.

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

## Gradient Options

`gradient` controls two things: (1) a **light horizontal strip** behind the header row, and (2) **colors for the leading icon tile** when an icon is shown.

- **purple** / **blue** / **cyan**: Tinted strip + matching icon shell
- **none**: No strip; icon shell uses neutral surface styling

```vue
// Accent strip + icon shell (purple family)
registerPageHeader({
  title: "Settings",
  gradient: "purple"
});

// No accent strip (neutral icon tile if icon is shown)
registerPageHeader({
  title: "Simple Page",
  gradient: "none"
});
```

## Leading icon

- **`leadingIcon`**: Iconify / Nuxt UI icon name (e.g. `i-lucide-layout-dashboard`). Shown in a rounded tile next to the title.
- **`hideLeadingIcon: true`**: Never show the icon tile (even if the menu has an icon for this path).
- **Default**: If you omit `leadingIcon` and do not hide it, the app tries to use the **menu icon** registered for the current route (`useMenuRegistry` / `findMenuIconForPath`).

## Stats Display

### Basic Stats

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

### Reactive stats

Update the header whenever the underlying numbers change (same pattern as dynamic titles):

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

## Advanced Features

### Conditional Page Headers

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

### Dynamic titles and stats

`registerPageHeader` stores a **plain snapshot** of the config (not reactive refs inside the object). When the title or stats depend on async data, use `watch` (or `watchEffect`) and call `registerPageHeader` again when values change.

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

### Checking Header Status

```vue
<script setup>
const { hasPageHeader, pageHeader } = usePageHeaderRegistry();

// Check if header is registered
if (hasPageHeader.value) {
  console.log('Current header:', pageHeader.value);
}
</script>
```

## Real-World Examples

### 1. Data Table Page

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

### 2. Settings Page

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

### 3. Analytics Dashboard

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

## Best Practices

### 1. Use Appropriate Variants

```javascript
// Dashboard/landing pages
{ variant: "minimal" }

// Data listing pages
{ variant: "default", stats: [...] }

// Analytics pages
{ variant: "stats-focus", stats: [...] }
```

### 2. Match Gradient to Content

```javascript
// Data/collections
{ gradient: "blue" }

// Settings/configuration
{ gradient: "purple" }

// Dashboard/overview
{ gradient: "cyan" }
```

### 3. Keep Descriptions Concise

```javascript
// Good
{ description: "Manage user accounts and permissions" }

// Too long
{ description: "This page allows you to manage all user accounts in the system including creating new users, editing existing users, and configuring their permissions..." }
```

### 4. Use Reactive Stats

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

### 5. Clear Headers When Not Needed

```javascript
const { clearPageHeader } = usePageHeaderRegistry();

// Clear when navigating away
onUnmounted(() => {
  clearPageHeader();
});
```

## API Reference

### PageHeaderConfig Interface

```typescript
interface PageHeaderConfig {
  title: string;                           // Page title (required)
  description?: string;                    // Page description
  stats?: PageHeaderStat[];                // Statistics to display
  variant?: "default" | "minimal" | "stats-focus";  // Layout variant
  gradient?: "purple" | "blue" | "cyan" | "none";   // Header strip + leading icon tint
  leadingIcon?: string;                    // Icon name; if omitted, menu icon for route may be used
  hideLeadingIcon?: boolean;               // If true, no leading icon tile
}
```

### PageHeaderStat Interface

```typescript
interface PageHeaderStat {
  label: string;      // Stat label
  value: string | number;  // Stat value
}
```

### usePageHeaderRegistry()

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

## Summary

The Page Header System provides:

 **Consistent page headers** across the application
 **Multiple layout variants** for different page types
 **Accent strip + optional leading icon** (menu-driven or explicit)
 **Stats display** for key metrics
 **Auto-cleanup** on route changes
 **Update via `watch`** when title or stats change

**Related Documentation:**
- **[Header Actions](./header-actions.md)** - Adding actions to headers
- **[Form System](./form-system.md)** - Dynamic form generation

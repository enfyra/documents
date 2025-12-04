# Page Header System

The Page Header System allows you to register custom page headers that appear at the top of each page, providing context and navigation information to users.

## Table of Contents

- [Overview](#overview)
- [Basic Usage](#basic-usage)
- [Header Variants](#header-variants)
- [Gradient Options](#gradient-options)
- [Stats Display](#stats-display)
- [Advanced Features](#advanced-features)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)

## Overview

Page headers provide:
- **Title and Description**: Clear page context for users
- **Stats Display**: Show key metrics at a glance
- **Visual Variants**: Different layouts for different use cases
- **Gradient Themes**: Purple, blue, cyan backgrounds
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

Available gradients:
- **purple**: Purple gradient background
- **blue**: Blue gradient background
- **cyan**: Cyan gradient background
- **none**: No gradient (solid background)

```vue
// Purple gradient
registerPageHeader({
  title: "Settings",
  gradient: "purple"
});

// No gradient
registerPageHeader({
  title: "Simple Page",
  gradient: "none"
});
```

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

### Reactive Stats

```vue
<script setup>
const totalCount = ref(0);
const activeCount = ref(0);

const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: "Dashboard",
  stats: computed(() => [
    { label: "Total", value: totalCount.value },
    { label: "Active", value: activeCount.value }
  ])
});

// Update stats
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

### Dynamic Titles

```vue
<script setup>
const tableName = route.params.table;
const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: computed(() => `${tableName} Collection`),
  description: `Manage ${tableName} records`,
  gradient: "purple"
});
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
const tableName = route.params.table;
const { data: records } = useApi(() => `/${tableName}`);

const { registerPageHeader } = usePageHeaderRegistry();

registerPageHeader({
  title: computed(() => `${tableName} Data`),
  description: `Browse and manage ${tableName} records`,
  variant: "default",
  gradient: "blue",
  stats: computed(() => [
    { label: "Total Records", value: records.value?.meta?.totalCount || 0 },
    { label: "Showing", value: records.value?.data?.length || 0 }
  ])
});
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

registerPageHeader({
  title: "Analytics",
  description: "Real-time analytics and insights",
  variant: "stats-focus",
  gradient: "cyan",
  stats: computed(() => [
    { label: "Total Users", value: totalUsers.value.toLocaleString() },
    { label: "Active Now", value: activeUsers.value },
    { label: "Revenue", value: `$${revenue.value.toLocaleString()}` }
  ])
});

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
  gradient?: "purple" | "blue" | "cyan" | "none";   // Background gradient
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
 **Gradient themes** for visual variety
 **Stats display** for key metrics
 **Auto-cleanup** on route changes
 **Reactive support** for dynamic content

**Related Documentation:**
- **[Header Actions](./header-actions.md)** - Adding actions to headers
- **[Form System](./form-system.md)** - Dynamic form generation

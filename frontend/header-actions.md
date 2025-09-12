# Header Actions System

Extensions can dynamically inject custom actions into the app's header and sub-header areas, providing seamless integration with the core interface. **This demonstrates the incredible power of Enfyra's extension system - extensions can intervene and customize ANY part of the application interface.**

🔗 **Live Demo**: [https://demo.enfyra.io/](https://demo.enfyra.io/) - See header actions in action!

## Table of Contents

- [Overview](#overview)
- [Header Actions vs Sub-Header Actions](#header-actions-vs-sub-header-actions)
- [Action Types](#action-types)
- [Permission Integration](#permission-integration)
- [Basic Usage](#basic-usage)
- [Advanced Features](#advanced-features)
- [Positioning and Layout](#positioning-and-layout)
- [Route-Based Actions](#route-based-actions)
- [Custom Components](#custom-components)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)

## Overview

**🚀 Extension Power**: Extensions can completely control the application interface through header actions. This is just one example of how Enfyra extensions can **intervene in ANY part of the app**, making them incredibly powerful for customization.

The Header Actions system provides two main areas for custom actions:

- **Header Actions**: Located in the top-right corner of the main header
- **Sub-Header Actions**: Located in the page sub-header, can be positioned left or right

Both systems are **permission-aware** and **route-sensitive**, allowing for sophisticated customization based on user roles and current page context.

## Header Actions vs Sub-Header Actions

### Header Actions (Top-Right Corner)
```
┌─────────────────────────────────────────────────────┐
│ Enfyra App                           [Header Actions] │
└─────────────────────────────────────────────────────┘
```

### Sub-Header Actions (Page Level)
```
┌─────────────────────────────────────────────────────┐
│ [Left Actions]                    [Right Actions] │
└─────────────────────────────────────────────────────┘
│                    Page Content                      │
```

## Action Types

### 1. Button Actions
Standard buttons with icons, labels, and click handlers:

```javascript
const buttonAction = {
  id: 'export-data',
  label: 'Export',
  icon: 'lucide:download',
  variant: 'solid',
  color: 'primary',
  onClick: () => exportData()
};
```

### 2. Custom Component Actions
Inject completely custom components:

```javascript
const componentAction = {
  id: 'custom-widget',
  component: 'CustomWidget',
  props: { data: dynamicData }
};
```

## Permission Integration

**🔐 Every action is automatically wrapped with PermissionGate** - the same powerful permission system used throughout Enfyra.

**→ [Learn about Permission System](./permission-components.md)**

```javascript
const permissionAction = {
  id: 'admin-action',
  label: 'Admin Panel',
  icon: 'lucide:settings',
  onClick: () => openAdminPanel(),
  permission: {
    route: '/admin',
    actions: ['read']
  }
};
```

**Permission Features:**
- Actions automatically hidden if user lacks permission
- Supports complex AND/OR permission conditions
- Works with role-based access control
- Dynamic permission checking based on data context

**→ [Complete Permission Guide](./permission-builder.md)**

## Basic Usage

### Header Actions in Extensions

```vue
<script setup>
// Available globally in extensions
const { register } = useHeaderActionRegistry();

onMounted(() => {
  // Register single action
  register({
    id: 'save-report',
    label: 'Save Report',
    icon: 'lucide:save',
    color: 'success',
    onClick: () => {
      toast.add({
        title: 'Report saved!',
        color: 'success'
      });
    },
    permission: {
      route: '/reports',
      actions: ['create']
    }
  });
});
</script>
```

### Sub-Header Actions in Extensions

```vue
<script setup>
// Available globally in extensions
const { register } = useSubHeaderActionRegistry();

onMounted(() => {
  register({
    id: 'filter-toggle',
    label: 'Filters',
    icon: 'lucide:filter',
    side: 'left', // Position on left side
    variant: 'soft',
    onClick: () => toggleFilters()
  });
});
</script>
```

## Advanced Features

### Reactive Properties
All properties can be reactive using refs or computed:

```vue
<script setup>
const { ref, computed } = Vue;
const isLoading = ref(false);
const itemCount = ref(0);

const dynamicLabel = computed(() => 
  `Export (${itemCount.value} items)`
);

register({
  id: 'dynamic-export',
  label: dynamicLabel,
  loading: isLoading,
  disabled: computed(() => itemCount.value === 0),
  onClick: async () => {
    isLoading.value = true;
    await exportItems();
    isLoading.value = false;
  }
});
</script>
```

### Multiple Actions Registration

```vue
<script setup>
onMounted(() => {
  const actions = [
    {
      id: 'refresh',
      icon: 'lucide:refresh-cw',
      onClick: () => refresh()
    },
    {
      id: 'export',
      label: 'Export',
      icon: 'lucide:download',
      onClick: () => exportData()
    },
    {
      id: 'settings',
      icon: 'lucide:settings',
      onClick: () => openSettings()
    }
  ];

  // Register all at once
  actions.forEach(action => register(action));
});
</script>
```

## Positioning and Layout

### Sub-Header Positioning

```vue
<script setup>
onMounted(() => {
  // Left side actions (typically filters, views)
  register({
    id: 'view-toggle',
    label: 'Grid View',
    icon: 'lucide:grid',
    side: 'left',
    onClick: () => toggleView()
  });

  // Right side actions (typically actions, exports)
  register({
    id: 'export',
    label: 'Export',
    icon: 'lucide:download',
    side: 'right', // Default
    onClick: () => exportData()
  });
});
</script>
```

### Responsive Behavior
Actions automatically adapt to screen size:
- Mobile: Only icons shown
- Desktop: Icons + labels shown
- Tablet: Smaller buttons

## Route-Based Actions

### Show/Hide on Specific Routes

```vue
<script setup>
onMounted(() => {
  register({
    id: 'route-specific',
    label: 'Data Tools',
    icon: 'lucide:database',
    showOn: ['/data'], // Only show on data routes
    hideOn: ['/settings'], // Hide on settings routes
    onClick: () => openDataTools()
  });
});
</script>
```

### Global vs Route-Specific Actions

```vue
<script setup>
onMounted(() => {
  // Persists across all routes
  register({
    id: 'global-help',
    icon: 'lucide:help-circle',
    global: true,
    onClick: () => openHelp()
  });

  // Cleared when navigating away
  register({
    id: 'page-specific',
    label: 'Page Action',
    onClick: () => doPageSpecificAction()
  });
});
</script>
```

## Custom Components

### Injecting Custom Components

```vue
<template>
  <div id="my-extension">
    <!-- Extension content -->
  </div>
</template>

<script setup>
// Define a custom component
const CustomStatusWidget = {
  template: `
    <div class="flex items-center gap-2">
      <UBadge :color="status.color">{{ status.text }}</UBadge>
      <UButton @click="refresh" icon="lucide:refresh-cw" variant="ghost" size="sm" />
    </div>
  `,
  setup() {
    const status = ref({ text: 'Online', color: 'green' });
    const refresh = () => {
      // Refresh logic
    };
    return { status, refresh };
  }
};

onMounted(() => {
  // Register as header action
  useHeaderActionRegistry().register({
    id: 'status-widget',
    component: CustomStatusWidget
  });
});
</script>
```

### Component with Props

```vue
<script setup>
const DataCounter = {
  template: `
    <UBadge variant="soft" color="primary">
      {{ count }} items
    </UBadge>
  `,
  props: ['count']
};

const itemCount = ref(150);

register({
  id: 'data-counter',
  component: DataCounter,
  props: {
    count: itemCount.value
  }
});
</script>
```

## Real-World Examples

### 1. Data Export Extension

```vue
<template>
  <div class="p-6">
    <h2>Data Management</h2>
    <!-- Extension content -->
  </div>
</template>

<script setup>
const isExporting = ref(false);
const selectedFormat = ref('json');

const exportData = async () => {
  isExporting.value = true;
  try {
    const { data } = await useApi('/export', {
      method: 'POST',
      body: { format: selectedFormat.value }
    });
    
    // Download file
    downloadFile(data.url);
    
    toast.add({
      title: 'Export completed',
      description: `Data exported as ${selectedFormat.value.toUpperCase()}`,
      color: 'success'
    });
  } catch (error) {
    toast.add({
      title: 'Export failed',
      description: error.message,
      color: 'error'
    });
  } finally {
    isExporting.value = false;
  }
};

onMounted(() => {
  // Export dropdown component
  const ExportDropdown = {
    template: `
      <UDropdown :items="exportItems">
        <UButton 
          label="Export"
          icon="lucide:download"
          :loading="loading"
          variant="solid"
          color="primary"
        />
      </UDropdown>
    `,
    setup() {
      const exportItems = [
        [
          {
            label: 'JSON',
            icon: 'lucide:file-json',
            click: () => {
              selectedFormat.value = 'json';
              exportData();
            }
          },
          {
            label: 'CSV',
            icon: 'lucide:file-spreadsheet',
            click: () => {
              selectedFormat.value = 'csv';
              exportData();
            }
          },
          {
            label: 'PDF',
            icon: 'lucide:file-text',
            click: () => {
              selectedFormat.value = 'pdf';
              exportData();
            }
          }
        ]
      ];
      
      return {
        exportItems,
        loading: isExporting
      };
    }
  };

  useHeaderActionRegistry().register({
    id: 'export-dropdown',
    component: ExportDropdown,
    permission: {
      route: '/data',
      actions: ['read']
    }
  });
});
</script>
```

### 2. Real-time Status Dashboard

```vue
<template>
  <div class="dashboard p-6">
    <h1>System Dashboard</h1>
    <!-- Dashboard content -->
  </div>
</template>

<script setup>
const connectionStatus = ref('connecting');
const lastUpdate = ref(null);
const autoRefresh = ref(true);

// Real-time connection monitoring
const StatusIndicator = {
  template: `
    <div class="flex items-center gap-2">
      <UBadge 
        :color="badgeColor" 
        variant="soft"
        class="animate-pulse"
      >
        {{ statusText }}
      </UBadge>
      <span class="text-xs text-gray-500">
        {{ lastUpdateText }}
      </span>
    </div>
  `,
  setup() {
    const badgeColor = computed(() => {
      switch (connectionStatus.value) {
        case 'connected': return 'green';
        case 'connecting': return 'yellow';
        case 'error': return 'red';
        default: return 'gray';
      }
    });
    
    const statusText = computed(() => {
      return connectionStatus.value.charAt(0).toUpperCase() + 
             connectionStatus.value.slice(1);
    });
    
    const lastUpdateText = computed(() => {
      if (!lastUpdate.value) return '';
      return `Updated ${formatDistanceToNow(lastUpdate.value)} ago`;
    });
    
    return {
      badgeColor,
      statusText,
      lastUpdateText
    };
  }
};

onMounted(() => {
  // Register status in sub-header left
  useSubHeaderActionRegistry().register({
    id: 'connection-status',
    component: StatusIndicator,
    side: 'left'
  });

  // Register controls in sub-header right
  useSubHeaderActionRegistry().register({
    id: 'auto-refresh-toggle',
    label: computed(() => autoRefresh.value ? 'Auto-refresh ON' : 'Auto-refresh OFF'),
    icon: 'lucide:refresh-cw',
    variant: computed(() => autoRefresh.value ? 'solid' : 'outline'),
    color: computed(() => autoRefresh.value ? 'primary' : 'neutral'),
    side: 'right',
    onClick: () => {
      autoRefresh.value = !autoRefresh.value;
      toast.add({
        title: `Auto-refresh ${autoRefresh.value ? 'enabled' : 'disabled'}`,
        color: autoRefresh.value ? 'success' : 'neutral'
      });
    }
  });

  // Monitor connection
  watchConnection();
});

const watchConnection = () => {
  const interval = setInterval(async () => {
    try {
      connectionStatus.value = 'connecting';
      const { data } = await useApi('/health');
      connectionStatus.value = 'connected';
      lastUpdate.value = new Date();
    } catch (error) {
      connectionStatus.value = 'error';
    }
  }, autoRefresh.value ? 5000 : 30000);

  onUnmounted(() => clearInterval(interval));
};
</script>
```

### 3. Advanced Permission-Based Actions

```vue
<template>
  <div class="admin-panel p-6">
    <h2>Administrative Tools</h2>
    <!-- Admin content -->
  </div>
</template>

<script setup>
const { me } = useEnfyraAuth();
const userRole = computed(() => me.value?.role?.name);

onMounted(() => {
  // Different actions based on user role
  const adminActions = [
    {
      id: 'system-settings',
      label: 'System',
      icon: 'lucide:server',
      permission: {
        route: '/admin',
        actions: ['update']
      },
      onClick: () => navigateTo('/admin/system')
    },
    {
      id: 'user-management',
      label: 'Users',
      icon: 'lucide:users',
      permission: {
        route: '/user_definition',
        actions: ['create', 'update', 'delete']
      },
      onClick: () => navigateTo('/admin/users')
    },
    {
      id: 'audit-logs',
      label: 'Audit',
      icon: 'lucide:file-search',
      permission: {
        route: '/audit_log',
        actions: ['read']
      },
      onClick: () => navigateTo('/admin/audit')
    }
  ];

  // Super admin only actions
  const superAdminActions = [
    {
      id: 'danger-zone',
      label: 'Danger Zone',
      icon: 'lucide:alert-triangle',
      color: 'error',
      permission: {
        and: [
          { route: '/admin', actions: ['delete'] },
          { allowedUsers: [me.value?.id] } // Only specific user
        ]
      },
      onClick: () => openDangerZone()
    }
  ];

  // Register actions based on permissions
  [...adminActions, ...superAdminActions].forEach(action => {
    useHeaderActionRegistry().register(action);
  });

  // Dynamic action based on user data
  if (userRole.value === 'manager') {
    useSubHeaderActionRegistry().register({
      id: 'team-overview',
      label: `Team Overview (${me.value.team?.memberCount || 0})`,
      icon: 'lucide:users',
      side: 'left',
      onClick: () => showTeamOverview()
    });
  }
});
</script>
```

## Best Practices

### 1. Use Meaningful IDs
```javascript
// Good
{ id: 'export-user-data', label: 'Export Users' }

// Avoid
{ id: 'btn1', label: 'Export Users' }
```

### 2. Implement Permission Checks
```javascript
// Always include permission conditions
{
  id: 'delete-action',
  label: 'Delete',
  icon: 'lucide:trash-2',
  color: 'error',
  permission: {
    route: '/data',
    actions: ['delete']
  }
}
```

### 3. Provide Loading States
```javascript
const isProcessing = ref(false);

{
  id: 'process-data',
  label: 'Process',
  loading: isProcessing,
  onClick: async () => {
    isProcessing.value = true;
    await processData();
    isProcessing.value = false;
  }
}
```

### 4. Use Appropriate Positioning
```javascript
// Left side: Views, filters, status
{ side: 'left', id: 'filter-toggle' }

// Right side: Actions, exports, settings
{ side: 'right', id: 'export-data' }
```

### 5. Clean Up on Unmount
```javascript
onUnmounted(() => {
  // Actions are automatically cleaned up on route change
  // Only manual cleanup needed for global actions
});
```

### 6. Cross-Reference Related Documentation
When working with permissions:
- **→ [Permission Builder](./permission-builder.md)** - Create complex permission rules
- **→ [Permission Components](./permission-components.md)** - PermissionGate usage
- **→ [Permission System](../backend/permission-system.md)** - Backend permission architecture

## API Reference

### HeaderAction Interface

```typescript
interface HeaderAction {
  // Core Properties
  id: string;                                    // Unique identifier
  label?: string | ComputedRef<string>;          // Button text
  icon?: string | ComputedRef<string>;           // Icon name
  
  // Styling
  variant?: 'solid' | 'outline' | 'ghost' | 'soft';
  color?: 'primary' | 'secondary' | 'warning' | 'success' | 'info' | 'error' | 'neutral';
  size?: 'sm' | 'md' | 'lg' | 'xl';
  class?: string;                                // Custom CSS classes
  
  // State
  loading?: boolean | Ref<boolean> | ComputedRef<boolean>;
  disabled?: boolean | Ref<boolean> | ComputedRef<boolean>;
  show?: boolean | Ref<boolean> | ComputedRef<boolean>;
  
  // Actions
  onClick?: () => void;                          // Click handler
  to?: string | Ref<string> | ComputedRef<string>; // Navigation target
  submit?: () => void;                           // Submit handler
  
  // Positioning & Visibility
  side?: 'left' | 'right';                       // Sub-header position (default: 'right')
  showOn?: string[];                             // Show on specific routes
  hideOn?: string[];                             // Hide on specific routes
  global?: boolean;                              // Persist across routes
  
  // Permissions
  permission?: PermissionCondition;              // Permission requirements
  
  // Custom Components
  component?: string | any;                      // Custom component
  props?: Record<string, any>;                   // Component props
  key?: string;                                  // Force re-render key
}
```

### useHeaderActionRegistry()

```typescript
const { register } = useHeaderActionRegistry();

// Register single action
register(action: HeaderAction): void

// Auto-registration on composable creation
useHeaderActionRegistry(actions: HeaderAction | HeaderAction[])
```

### useSubHeaderActionRegistry()

```typescript
const { register } = useSubHeaderActionRegistry();

// Register single action
register(action: HeaderAction): void

// Auto-registration on composable creation
useSubHeaderActionRegistry(actions: HeaderAction | HeaderAction[])
```

## Summary

The Header Actions system showcases the **incredible power of Enfyra extensions** - they can seamlessly integrate with and control ANY part of the application interface. This is just one example of how extensions can:

✅ **Intervene in core UI areas** - Headers, sidebars, forms, tables
✅ **Inject custom functionality** - Buttons, widgets, complete components  
✅ **Respect permission systems** - Automatic PermissionGate integration
✅ **Respond to route changes** - Dynamic behavior based on current page
✅ **Provide real-time updates** - Reactive properties and live data

**🚀 Extensions aren't limited to custom pages - they can enhance and customize every aspect of the Enfyra experience, making them incredibly powerful for building exactly what you need.**

**Related Documentation:**
- **[Extension System](./extension-system.md)** - Complete extension development guide
- **[Permission Components](./permission-components.md)** - PermissionGate and permission integration
- **[API Integration](./api-integration.md)** - Fetching data for dynamic actions
# Permission Components

Enfyra provides two main tools for controlling UI visibility based on user permissions: the `PermissionGate` component for declarative rendering and the `usePermissions` composable for programmatic checks.

## PermissionGate Component

The `PermissionGate` component wraps UI elements and automatically shows/hides them based on user permissions.

### Basic Usage

```vue
<PermissionGate :condition="{ route: '/user_definition', actions: ['read'] }">
  <div>This content only shows if user can read users</div>
</PermissionGate>
```

### Multiple Actions

Check if user has ANY of the listed actions:

```vue
<PermissionGate :condition="{ route: '/user_definition', actions: ['create', 'update'] }">
  <UButton>Edit User</UButton>
</PermissionGate>
```

### Complex Conditions

#### AND Logic
User must have ALL permissions:

```vue
<PermissionGate :condition="{
  and: [
    { route: '/user_definition', actions: ['read'] },
    { route: '/roles', actions: ['read'] }
  ]
}">
  <div>User can read both users AND roles</div>
</PermissionGate>
```

#### OR Logic
User needs ANY of these permissions:

```vue
<PermissionGate :condition="{
  or: [
    { route: '/user_definition', actions: ['create'] },
    { route: '/user_definition', actions: ['update'] }
  ]
}">
  <UButton>Modify User</UButton>
</PermissionGate>
```

#### Nested Conditions
Combine AND/OR for complex logic:

```vue
<PermissionGate :condition="{
  or: [
    { route: '/admin', actions: ['read'] },
    {
      and: [
        { route: '/user_definition', actions: ['read'] },
        { route: '/user_definition', actions: ['update'] }
      ]
    }
  ]
}">
  <div>Admin OR (can read AND update users)</div>
</PermissionGate>
```

## usePermissions Composable

The `usePermissions` composable provides programmatic permission checking in your Vue components.

### Setup

```vue
<script setup lang="ts">
const { hasPermission, checkPermissionCondition } = usePermissions();
</script>
```

### Check Specific Permission

```vue
<script setup lang="ts">
const { hasPermission } = usePermissions();

// Check single permission
const canCreateUsers = computed(() => {
  return hasPermission('/user_definition', 'POST'); // POST = create
});

// Use in functions
async function deleteUser(id: string) {
  if (!hasPermission('/user_definition', 'DELETE')) {
    toast.add({
      title: 'Access Denied',
      description: 'You do not have permission to delete users',
      color: 'error'
    });
    return;
  }
  
  await api.delete(`/user_definition/${id}`);
}
</script>
```

### Check Complex Conditions

```vue
<script setup lang="ts">
const { checkPermissionCondition } = usePermissions();

// Complex permission check
const canManageUsers = computed(() => {
  return checkPermissionCondition({
    and: [
      { route: '/user_definition', actions: ['read'] },
      { 
        or: [
          { route: '/user_definition', actions: ['create'] },
          { route: '/user_definition', actions: ['update'] }
        ]
      }
    ]
  });
});
</script>
```

### HTTP Method Mapping

The system maps actions to HTTP methods:

| Action | HTTP Method | Usage |
|--------|-------------|-------|
| `read` | GET | View/list data |
| `create` | POST | Create new records |
| `update` | PATCH | Modify existing records |
| `delete` | DELETE | Remove records |

## Integration with Menu System

The menu system uses both `PermissionGate` and `usePermissions` internally to control menu visibility.

### How Menus Use Permissions

When you set permissions on a menu item:

```javascript
// Menu configuration
{
  label: 'User Management',
  route: '/settings/users',
  permission: {
    or: [
      { route: '/users', actions: ['read'] },
      { route: '/users', actions: ['create'] }
    ]
  }
}
```

The menu system:
1. Uses `checkPermissionCondition` to evaluate the permission
2. Only renders the menu item if permission check passes
3. Automatically hides parent menus if no child items are accessible

### Menu Components

**Menu System** uses permissions:
```vue
// Internally filters menu items based on permissions
const visibleItems = menuGroups.filter(item => {
  if (!item.permission) return true;
  return checkPermissionCondition(item.permission);
});
```

**Menu Items** wrapped in PermissionGate:
```vue
<PermissionGate :condition="menuItem.permission">
  <MenuItem :item="menuItem" />
</PermissionGate>
```

**Result**: Menus automatically adapt to user permissions without manual configuration.

## Common Patterns

### Conditional Buttons

```vue
<template>
  <div class="flex gap-2">
    <PermissionGate :condition="{ route: '/user_definition', actions: ['create'] }">
      <UButton color="primary" @click="createUser">
        Create User
      </UButton>
    </PermissionGate>
    
    <PermissionGate :condition="{ route: '/user_definition', actions: ['delete'] }">
      <UButton color="red" @click="deleteSelected">
        Delete Selected
      </UButton>
    </PermissionGate>
  </div>
</template>
```

### Table Actions

```vue
<template>
  <UTable :rows="users">
    <template #actions="{ row }">
      <PermissionGate :condition="{ route: '/user_definition', actions: ['update'] }">
        <UButton size="sm" @click="editUser(row.id)">Edit</UButton>
      </PermissionGate>
      
      <PermissionGate :condition="{ route: '/user_definition', actions: ['delete'] }">
        <UButton size="sm" color="red" @click="deleteUser(row.id)">Delete</UButton>
      </PermissionGate>
    </template>
  </UTable>
</template>
```

### Form Submission

```vue
<script setup lang="ts">
const { hasPermission } = usePermissions();

async function handleSubmit() {
  // Check permission before processing
  if (!hasPermission('/user_definition', 'POST')) {
    toast.add({
      title: 'Access Denied', 
      description: 'You cannot create users',
      color: 'error'
    });
    return;
  }
  
  // Validate and submit
  const { isValid, errors } = validate(formData.value);
  if (!isValid) {
    formErrors.value = errors;
    return;
  }
  
  await api.post('/user_definition', formData.value);
}
</script>
```

### Header Actions

```vue
<script setup lang="ts">
// Register header action with permission
useHeaderActionRegistry({
  id: 'create-user',
  label: 'Create User',
  permission: {
    route: '/user_definition',
    actions: ['create']
  },
  onClick: () => navigateTo('/users/create')
});
</script>
```

## Special Cases

### Root Admin

Root admins bypass all permission checks:

```vue
<script setup lang="ts">
const { me } = useEnfyraAuth();

// Root admin has all permissions automatically
if (me.value?.isRootAdmin) {
  // All permission checks return true
}
</script>
```

### Allow All

Grant unrestricted access (use sparingly):

```vue
<PermissionGate :condition="{ allowAll: true }">
  <div>Always visible content</div>
</PermissionGate>
```

### Direct User Permissions

Users can have permissions that bypass their role:

```javascript
// User's direct permissions override role permissions
// Checked automatically by usePermissions
```

## Best Practices

### Use PermissionGate for UI
- Wrap buttons, menu items, sections
- Keeps templates clean and declarative
- Automatically handles permission changes

### Use usePermissions for Logic
- Business logic and validation
- Computed properties for complex checks
- API calls and data processing

### Cache Permission Checks
```vue
<script setup lang="ts">
// Good - computed property caches result
const canEdit = computed(() => hasPermission('/users', 'PATCH'));

// Avoid - checking in template repeatedly
// <div v-if="hasPermission('/users', 'PATCH')">
</script>
```

### Match API Routes
Always use actual API endpoint paths:
```javascript
// Good - matches API endpoint
{ route: '/user_definition', actions: ['read'] }

// Bad - doesn't match actual route
{ route: '/users', actions: ['read'] }
```

## Debugging

### Check Current Permissions

```vue
<script setup lang="ts">
const { me } = useEnfyraAuth();
const { hasPermission } = usePermissions();

// Debug user permissions
console.log('User:', me.value);
console.log('Role:', me.value?.role);
console.log('Is Root Admin:', me.value?.isRootAdmin);

// Test specific permissions
console.log('Can read users:', hasPermission('/users', 'GET'));
console.log('Can create users:', hasPermission('/users', 'POST'));
</script>
```

### Test Permission Conditions

```vue
<script setup lang="ts">
const condition = {
  and: [
    { route: '/users', actions: ['read'] },
    { route: '/roles', actions: ['read'] }
  ]
};

const hasAccess = checkPermissionCondition(condition);
console.log('Condition result:', hasAccess);
</script>
```

## Related Documentation

- **[Permission System](../server/permission-system.md)** - Backend permission architecture
- **[Permission Builder](./permission-builder.md)** - Visual permission configuration
- **[Menu Management](./menu-management.md)** - How menus use permissions
- **[Form System](./form-system.md)** - Permission integration in forms
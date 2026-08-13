# Permission Components

Enfyra provides two main tools for controlling UI visibility based on user permissions: the `PermissionGate` component for declarative rendering and the `usePermissions` composable for programmatic checks.

## PermissionGate Component

The `PermissionGate` component wraps UI elements and automatically shows/hides them based on user permissions.

### Basic Usage

```vue
<PermissionGate :condition="{ route: '/enfyra_user', methods: ['GET'] }">
  <div>This content only shows if user can read users</div>
</PermissionGate>
```

### Multiple Actions

Check if user has ANY of the listed actions:

```vue
<PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST', 'PATCH'] }">
  <UButton>Edit User</UButton>
</PermissionGate>
```

### Complex Conditions

#### AND Logic
User must have ALL permissions:

```vue
<PermissionGate :condition="{
  and: [
    { route: '/enfyra_user', methods: ['GET'] },
    { route: '/roles', methods: ['GET'] }
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
    { route: '/enfyra_user', methods: ['POST'] },
    { route: '/enfyra_user', methods: ['PATCH'] }
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
    { route: '/admin', methods: ['GET'] },
    {
      and: [
        { route: '/enfyra_user', methods: ['GET'] },
        { route: '/enfyra_user', methods: ['PATCH'] }
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
  return hasPermission('/enfyra_user', 'POST'); // POST = create
});

// Use in functions
async function deleteUser(id: string) {
  if (!hasPermission('/enfyra_user', 'DELETE')) {
    toast.add({
      title: 'Access Denied',
      description: 'You do not have permission to delete users',
      color: 'error'
    });
    return;
  }
  
  await api.delete(`/enfyra_user/${id}`);
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
      { route: '/enfyra_user', methods: ['GET'] },
      { 
        or: [
          { route: '/enfyra_user', methods: ['POST'] },
          { route: '/enfyra_user', methods: ['PATCH'] }
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

Menu visibility is a separate role-to-menu contract. It does not reuse route/method conditions and it does not grant API access.

### Menu visibility contract

```javascript
// enfyra_menu
{
  label: 'User Management',
  path: '/settings/users',
  isPublic: false,
  menuPermissions: [
    { isEnabled: true, role: { id: 7, name: 'operator' } }
  ]
}
```

- `isPublic: true` shows an enabled menu to every role.
- New menus default to `isPublic: false`; the built-in Dashboard (`/dashboard`) is the initial public exception. Other menus are hidden from non-root roles on a fresh install until a rule is added.
- `isPublic: false` requires an enabled `enfyra_menu_permission` row for the user's role.
- A private menu with no enabled role rows is hidden from non-root roles.
- A parent remains visible when any child is visible.

The app sidebar evaluates this contract through `usePermissions().hasMenuPermission()`. The old `enfyra_menu.permission` JSON field is not used for navigation visibility.

### Menu visibility and action gates

Keep route/method `PermissionGate` conditions inside the page for buttons, forms, tabs, and actions:

```vue
<PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST'] }">
  <UButton @click="createUser">Create user</UButton>
</PermissionGate>
```

The backend route permission remains authoritative. A menu may be visible while an API call returns `403`; configure `PermissionGate` and route access independently.

## Common Patterns

### Conditional Buttons

```vue
<template>
  <div class="flex gap-2">
    <PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST'] }">
      <UButton color="primary" @click="createUser">
        Create User
      </UButton>
    </PermissionGate>
    
    <PermissionGate :condition="{ route: '/enfyra_user', methods: ['DELETE'] }">
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
      <PermissionGate :condition="{ route: '/enfyra_user', methods: ['PATCH'] }">
        <UButton size="sm" @click="editUser(row.id)">Edit</UButton>
      </PermissionGate>
      
      <PermissionGate :condition="{ route: '/enfyra_user', methods: ['DELETE'] }">
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
  if (!hasPermission('/enfyra_user', 'POST')) {
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
  
  await api.post('/enfyra_user', formData.value);
}
</script>
```

### Header Actions

```vue
<script setup lang="ts">
const { register: registerHeaderActions } = useHeaderActionRegistry();
// Register header action with permission
registerHeaderActions({
  id: 'create-user',
  label: 'Create User',
  permission: {
    route: '/enfyra_user',
    methods: ['POST']
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
const { me } = useAuth();

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
{ route: '/enfyra_user', methods: ['GET'] }

// Bad - doesn't match actual route
{ route: '/users', methods: ['GET'] }
```

## Debugging

### Check Current Permissions

```vue
<script setup lang="ts">
const { me } = useAuth();
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
    { route: '/users', methods: ['GET'] },
    { route: '/roles', methods: ['GET'] }
  ]
};

const hasAccess = checkPermissionCondition(condition);
console.log('Condition result:', hasAccess);
</script>
```

## Related Documentation

- **[Permission Builder](./permission-builder.md)** - Backend permission architecture
- **[Permission Builder](./permission-builder.md)** - Visual permission configuration
- **[Menu Management](./menu-management.md)** - How menus use permissions
- **[Form System](./form-system.md)** - Permission integration in forms

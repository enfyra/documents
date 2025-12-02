# API Integration

Enfyra uses the official **@enfyra/sdk-nuxt** package for all API interactions, with a custom `useApi` wrapper that provides enhanced error handling and additional features. The SDK provides optimized composables for fetching data, authentication, and real-time updates with built-in caching and error handling.

**📖 Complete SDK Documentation**: [https://github.com/dothinh115/enfyra-sdk-nuxt](https://github.com/dothinh115/enfyra-sdk-nuxt)

## Backend Dependency

**CRITICAL**: All API calls in the frontend connect to your **backend server URL** (typically `http://localhost:1105`). The `useApi()` composable internally makes HTTP requests to:

```
${BACKEND_URL}/api/${endpoint}
```

**No API exists on the frontend** - it's purely a client consuming backend APIs. When you create tables, APIs are generated on the backend server and consumed by frontend via HTTP requests.

## SDK Overview

The Enfyra integration provides these main composables:
- `useApi()` - Custom API wrapper with enhanced error handling (recommended)
- `useEnfyraApi()` - Direct SDK API data fetching with caching
- `useEnfyraAuth()` - Authentication management
- Built-in TypeScript support and error handling
- Automatic retry mechanisms and loading states

## useApi Composable (Recommended)

### Basic Usage

```vue
<script setup>
// Fetch data with custom error handling and caching
// This makes HTTP request to: ${BACKEND_URL}/api/user_definition
const { data, pending, error, refresh } = await useApi('/user_definition', {
  query: {
    limit: 10,
    fields: 'id,email,name,role.name'
  }
});

// Direct SDK access also available for advanced usage
// Also makes HTTP request to backend server
const { data: directData } = await useEnfyraApi('/user_definition', {
  query: { limit: 10 }
});
</script>

<template>
  <div>
    <!-- Loading state -->
    <div v-if="pending">Loading users...</div>
    
    <!-- Error handling -->
    <UAlert v-else-if="error" color="red">
      Error: {{ error.message }}
    </UAlert>
    
    <!-- Data display -->
    <div v-else>
      <div v-for="user in data?.data" :key="user.id">
        {{ user.name }} - {{ user.role?.name }}
      </div>
      
      <!-- Refresh button -->
      <UButton @click="refresh">Refresh Data</UButton>
    </div>
  </div>
</template>
```

### Advanced Query Options

```vue
<script setup>
// Complex query with filtering, sorting, and pagination using custom wrapper
const { data, pending, error } = await useApi('/product_definition', {
  query: {
    // Pagination
    page: 1,
    limit: 20,
    
    // Field selection
    fields: 'id,name,price,category.name,createdAt',
    
    // Filtering
    filter: {
      price: { _gte: 100 }, // Price >= 100
      category: {
        name: { _contains: 'electronics' }
      },
      isActive: { _eq: true }
    },
    
    // Sorting
    sort: '-createdAt,name', // Sort by createdAt desc, then name asc
    
    // Relations
    include: 'category,reviews'
  },
  
  // Caching options
  key: 'products-list',
  server: false, // Client-side only
  default: () => ({ data: [], total: 0 })
});
</script>
```

### POST/PUT/DELETE Operations

```vue
<script setup>
const toast = useToast();

// Create new record using custom API wrapper
// POST request to: ${BACKEND_URL}/api/user_definition
const createUser = async (userData) => {
  try {
    const { data } = await useApi('/user_definition', {
      method: 'POST',
      body: userData
    });
    
    toast.add({
      title: 'Success',
      description: `User ${data.name} created successfully`,
      color: 'success'
    });
    
    return data;
  } catch (error) {
    toast.add({
      title: 'Error',
      description: error.message,
      color: 'error'
    });
    throw error;
  }
};

// Update existing record
// PATCH request to: ${BACKEND_URL}/api/user_definition/${userId}
const updateUser = async (userId, updates) => {
  const { data } = await useApi(`/user_definition/${userId}`, {
    method: 'PATCH',
    body: updates
  });
  
  return data;
};

// Delete record
// DELETE request to: ${BACKEND_URL}/api/user_definition/${userId}
const deleteUser = async (userId) => {
  await useApi(`/user_definition/${userId}`, {
    method: 'DELETE'
  });
};
</script>
```

## Authentication Integration

```vue
<script setup>
// Authentication composable
const { me, isLoggedIn, login, logout } = useEnfyraAuth();

// Login function
const handleLogin = async (credentials) => {
  try {
    await login(credentials);
    
    toast.add({
      title: 'Welcome back!',
      description: `Logged in as ${me.value.email}`,
      color: 'success'
    });
  } catch (error) {
    toast.add({
      title: 'Login failed',
      description: error.message,
      color: 'error'
    });
  }
};

// Logout function
const handleLogout = async () => {
  await logout();
  await navigateTo('/login');
};
</script>

<template>
  <div>
    <!-- Authenticated user info -->
    <div v-if="isLoggedIn">
      <p>Welcome, {{ me.email }}!</p>
      <p>Role: {{ me.role?.name }}</p>
      <UButton @click="handleLogout">Logout</UButton>
    </div>
    
    <!-- Login form -->
    <div v-else>
      <UButton @click="handleLogin({ email, password })">
        Login
      </UButton>
    </div>
  </div>
</template>
```

## Permission Integration

API calls often need permission checks. Enfyra provides the powerful `PermissionGate` component and `usePermissions` composable for controlling access to API functionality.

**→ [Complete Permission Guide](./permission-components.md)** - Learn about PermissionGate and usePermissions
**→ [Permission Builder](./permission-builder.md)** - Visual interface for creating permission rules

```vue
<template>
  <!-- Only show button if user has permission -->
  <PermissionGate :condition="{ route: '/user_definition', actions: ['create'] }">
    <UButton @click="createUser">Create User</UButton>
  </PermissionGate>
</template>

<script setup>
// Check permissions programmatically
const { hasPermission } = usePermissions();

const deleteUser = async (userId) => {
  // Check permission before API call
  if (!hasPermission('/user_definition', 'DELETE')) {
    toast.add({
      title: 'Permission denied',
      color: 'error'
    });
    return;
  }
  
  await useApi(`/user_definition/${userId}`, {
    method: 'DELETE'
  });
};
</script>
```

## API Integration in Extensions

### Dashboard Extension Example

```vue
<template>
  <div class="p-6 space-y-6">
    <h1 class="text-2xl font-bold">Dashboard</h1>
    
    <!-- Stats Cards -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold">{{ userStats?.total || 0 }}</div>
          <div class="text-sm text-gray-500">Total Users</div>
        </div>
      </UCard>
      
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold">{{ productStats?.total || 0 }}</div>
          <div class="text-sm text-gray-500">Products</div>
        </div>
      </UCard>
      
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold">{{ recentOrders.length }}</div>
          <div class="text-sm text-gray-500">Recent Orders</div>
        </div>
      </UCard>
    </div>
    
    <!-- Recent Activity -->
    <UCard>
      <template #header>
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-semibold">Recent Activity</h3>
          <UButton @click="refreshData" :loading="refreshing" size="sm">
            <UIcon name="lucide:refresh-cw" />
            Refresh
          </UButton>
        </div>
      </template>
      
      <UTable 
        :rows="recentActivity" 
        :columns="activityColumns"
        :loading="activityPending"
      />
    </UCard>
  </div>
</template>

<script setup>
const { ref, onMounted } = Vue;
const toast = useToast();

// Reactive state
const refreshing = ref(false);

// Fetch user statistics
const { data: userStats, pending: usersPending } = await useApi('/user_definition', {
  query: {
    limit: 0, // Get count only
    fields: 'id'
  },
  key: 'user-stats'
});

// Fetch product statistics
const { data: productStats, pending: productsPending } = await useApi('/product_definition', {
  query: {
    limit: 0,
    fields: 'id'
  },
  key: 'product-stats'
});

// Fetch recent orders
const { data: ordersData, pending: ordersPending } = await useApi('/order_definition', {
  query: {
    limit: 10,
    fields: 'id,total,status,user.name,createdAt',
    sort: '-createdAt',
    include: 'user'
  },
  key: 'recent-orders'
});

// Fetch recent activity
const { data: activityData, pending: activityPending, refresh: refreshActivity } = await useApi('/activity_log', {
  query: {
    limit: 20,
    fields: 'id,action,details,user.name,createdAt',
    sort: '-createdAt',
    include: 'user'
  },
  key: 'recent-activity'
});

// Computed properties
const recentOrders = computed(() => ordersData.value?.data || []);
const recentActivity = computed(() => activityData.value?.data || []);

const activityColumns = [
  { key: 'action', label: 'Action' },
  { key: 'user.name', label: 'User' },
  { key: 'details', label: 'Details' },
  { key: 'createdAt', label: 'Time' }
];

// Methods
const refreshData = async () => {
  refreshing.value = true;
  
  try {
    // Refresh all data
    await Promise.all([
      refreshActivity()
    ]);
    
    toast.add({
      title: 'Data refreshed',
      description: 'Dashboard data has been updated',
      color: 'success'
    });
  } catch (error) {
    toast.add({
      title: 'Refresh failed',
      description: error.message,
      color: 'error'
    });
  } finally {
    refreshing.value = false;
  }
};

// Real-time updates (optional)
onMounted(() => {
  // Set up auto-refresh every 30 seconds
  const interval = setInterval(() => {
    refreshData();
  }, 30000);
  
  // Cleanup
  onUnmounted(() => {
    clearInterval(interval);
  });
});
</script>
```

### Data Management Extension Example

```vue
<template>
  <div class="p-6 space-y-6">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-bold">User Management</h1>
        <p class="text-gray-500">Manage system users and roles</p>
      </div>
      
      <!-- Permission-controlled button - see Permission Components guide -->
      <PermissionGate :condition="{ route: '/user_definition', actions: ['create'] }">
        <UButton @click="showCreateModal = true" color="primary">
          <UIcon name="lucide:plus" />
          Add User
        </UButton>
      </PermissionGate>
    </div>
    
    <!-- Filters -->
    <UCard>
      <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
        <UInput 
          v-model="searchTerm" 
          placeholder="Search users..." 
          icon="lucide:search"
        />
        
        <USelect 
          v-model="selectedRole" 
          placeholder="Filter by role"
          :options="roleOptions"
        />
        
        <USelect 
          v-model="statusFilter" 
          placeholder="Filter by status"
          :options="statusOptions"
        />
        
        <UButton @click="applyFilters" :loading="pending">
          Apply Filters
        </UButton>
      </div>
    </UCard>
    
    <!-- Users Table -->
    <UCard>
      <UTable 
        :rows="users" 
        :columns="columns" 
        :loading="pending"
        @select="handleSelection"
      >
        <template #actions-data="{ row }">
          <div class="flex gap-2">
            <PermissionGate :condition="{ route: '/user_definition', actions: ['update'] }">
              <UButton 
                @click="editUser(row)" 
                size="sm" 
                variant="outline"
              >
                Edit
              </UButton>
            </PermissionGate>
            
            <PermissionGate :condition="{ route: '/user_definition', actions: ['delete'] }">
              <UButton 
                @click="deleteUser(row.id)" 
                size="sm" 
                color="red"
                variant="outline"
              >
                Delete
              </UButton>
            </PermissionGate>
          </div>
        </template>
      </UTable>
      
      <!-- Pagination -->
      <div class="flex justify-between items-center mt-4">
        <div class="text-sm text-gray-500">
          Showing {{ users.length }} of {{ totalUsers }} users
        </div>
        
        <UPagination 
          v-model="currentPage" 
          :page-count="pageCount"
          @update:model-value="loadUsers"
        />
      </div>
    </UCard>
    
    <!-- Create/Edit Modal -->
    <UModal v-model="showCreateModal">
      <UCard>
        <template #header>
          <h3>{{ editingUser ? 'Edit User' : 'Create User' }}</h3>
        </template>
        
        <div class="space-y-4">
          <UInput v-model="userForm.name" label="Name" />
          <UInput v-model="userForm.email" label="Email" type="email" />
          <USelect v-model="userForm.roleId" label="Role" :options="roleOptions" />
          <UToggle v-model="userForm.isActive" label="Active" />
        </div>
        
        <template #footer>
          <div class="flex gap-2">
            <UButton @click="showCreateModal = false" variant="outline">
              Cancel
            </UButton>
            <UButton @click="saveUser" :loading="saving">
              {{ editingUser ? 'Update' : 'Create' }}
            </UButton>
          </div>
        </template>
      </UCard>
    </UModal>
  </div>
</template>

<script setup>
const { ref, reactive, computed, watch } = Vue;
const toast = useToast();
const { confirm } = useConfirm();

// State
const searchTerm = ref('');
const selectedRole = ref(null);
const statusFilter = ref(null);
const currentPage = ref(1);
const showCreateModal = ref(false);
const editingUser = ref(null);
const saving = ref(false);

const userForm = reactive({
  name: '',
  email: '',
  roleId: null,
  isActive: true
});

// Build query dynamically
const apiQuery = computed(() => ({
  page: currentPage.value,
  limit: 20,
  fields: 'id,name,email,isActive,role.name,createdAt',
  include: 'role',
  sort: '-createdAt',
  filter: {
    ...(searchTerm.value && {
      _or: [
        { name: { _contains: searchTerm.value } },
        { email: { _contains: searchTerm.value } }
      ]
    }),
    ...(selectedRole.value && { roleId: { _eq: selectedRole.value } }),
    ...(statusFilter.value !== null && { isActive: { _eq: statusFilter.value } })
  }
}));

// Fetch users with reactive query
const { data: usersData, pending, refresh: refreshUsers } = await useApi('/user_definition', {
  query: apiQuery,
  key: 'users-list'
});

// Fetch roles for dropdown
const { data: rolesData } = await useApi('/role_definition', {
  query: {
    fields: 'id,name',
    sort: 'name'
  },
  key: 'roles-list'
});

// Computed properties
const users = computed(() => usersData.value?.data || []);
const totalUsers = computed(() => usersData.value?.total || 0);
const pageCount = computed(() => Math.ceil(totalUsers.value / 20));

const roleOptions = computed(() => 
  rolesData.value?.data?.map(role => ({
    label: role.name,
    value: role.id
  })) || []
);

const statusOptions = [
  { label: 'All', value: null },
  { label: 'Active', value: true },
  { label: 'Inactive', value: false }
];

const columns = [
  { key: 'name', label: 'Name' },
  { key: 'email', label: 'Email' },
  { key: 'role.name', label: 'Role' },
  { key: 'isActive', label: 'Status' },
  { key: 'createdAt', label: 'Created' },
  { key: 'actions', label: 'Actions' }
];

// Methods
const applyFilters = () => {
  currentPage.value = 1; // Reset to first page
  // Query will automatically update due to computed property
};

const loadUsers = () => {
  refreshUsers();
};

const editUser = (user) => {
  editingUser.value = user;
  Object.assign(userForm, {
    name: user.name,
    email: user.email,
    roleId: user.role?.id,
    isActive: user.isActive
  });
  showCreateModal.value = true;
};

const saveUser = async () => {
  saving.value = true;
  
  try {
    if (editingUser.value) {
      // Update existing user
      await $enfyraApi(`/user_definition/${editingUser.value.id}`, {
        method: 'PATCH',
        body: userForm
      });
      
      toast.add({
        title: 'User updated',
        description: `${userForm.name} has been updated successfully`,
        color: 'success'
      });
    } else {
      // Create new user
      await $enfyraApi('/user_definition', {
        method: 'POST',
        body: userForm
      });
      
      toast.add({
        title: 'User created',
        description: `${userForm.name} has been created successfully`,
        color: 'success'
      });
    }
    
    showCreateModal.value = false;
    resetForm();
    refreshUsers();
    
  } catch (error) {
    toast.add({
      title: 'Error',
      description: error.message,
      color: 'error'
    });
  } finally {
    saving.value = false;
  }
};

const deleteUser = async (userId) => {
  const isConfirmed = await confirm({
    title: 'Delete User',
    content: 'Are you sure you want to delete this user? This action cannot be undone.',
    confirmText: 'Delete'
  });
  
  if (!isConfirmed) return;
  
  try {
    await $enfyraApi(`/user_definition/${userId}`, {
      method: 'DELETE'
    });
    
    toast.add({
      title: 'User deleted',
      description: 'User has been deleted successfully',
      color: 'success'
    });
    
    refreshUsers();
  } catch (error) {
    toast.add({
      title: 'Delete failed',
      description: error.message,
      color: 'error'
    });
  }
};

const resetForm = () => {
  editingUser.value = null;
  Object.assign(userForm, {
    name: '',
    email: '',
    roleId: null,
    isActive: true
  });
};

// Watch for modal close to reset form
watch(showCreateModal, (newValue) => {
  if (!newValue) {
    resetForm();
  }
});
</script>
```

## Error Handling Patterns

```vue
<script setup>
// Global error handling
const handleApiError = (error) => {
  console.error('API Error:', error);
  
  // Different error types
  if (error.statusCode === 401) {
    toast.add({
      title: 'Authentication required',
      description: 'Please log in to continue',
      color: 'error'
    });
    navigateTo('/login');
  } else if (error.statusCode === 403) {
    toast.add({
      title: 'Access denied',
      description: 'You don\'t have permission to perform this action',
      color: 'error'
    });
  } else if (error.statusCode === 404) {
    toast.add({
      title: 'Not found',
      description: 'The requested resource was not found',
      color: 'error'
    });
  } else {
    toast.add({
      title: 'Error',
      description: error.message || 'An unexpected error occurred',
      color: 'error'
    });
  }
};

// API call with error handling
const fetchData = async () => {
  try {
    const { data } = await $enfyraApi('/endpoint');
    return data;
  } catch (error) {
    handleApiError(error);
    throw error;
  }
};
</script>
```

## Caching and Performance

```vue
<script setup>
// Cache key strategies
const userListKey = computed(() => 
  `users-${currentPage.value}-${searchTerm.value}-${selectedRole.value}`
);

// Fetch with custom cache key
const { data, pending, refresh } = await useApi('/user_definition', {
  query: apiQuery,
  key: userListKey, // Dynamic cache key
  server: false, // Client-side only
  default: () => ({ data: [], total: 0 })
});

// Refresh specific cached data
const invalidateCache = () => {
  // Refresh current query
  refresh();
  
  // Or clear specific cache
  clearNuxtData('users-list');
};

// Optimistic updates
const updateUserOptimistically = async (userId, updates) => {
  // Update local data immediately
  const currentData = data.value;
  if (currentData) {
    const userIndex = currentData.data.findIndex(u => u.id === userId);
    if (userIndex !== -1) {
      currentData.data[userIndex] = { ...currentData.data[userIndex], ...updates };
    }
  }
  
  try {
    // Send API request
    await $enfyraApi(`/user_definition/${userId}`, {
      method: 'PATCH',
      body: updates
    });
  } catch (error) {
    // Revert on error
    refresh();
    throw error;
  }
};
</script>
```

## Best Practices

### 1. Use Computed Queries
```vue
<script setup>
// Good - reactive query
const apiQuery = computed(() => ({
  filter: { name: { _contains: searchTerm.value } },
  page: currentPage.value
}));

const { data } = await useApi('/users', {
  query: apiQuery,
  key: 'users-list'
});
</script>
```

### 2. Handle Loading States
```vue
<template>
  <div>
    <USkeleton v-if="pending" class="h-20" />
    <UserCard v-else :user="data" />
  </div>
</template>
```

### 3. Implement Proper Error Boundaries
```vue
<script setup>
// Component-level error handling
onErrorCaptured((error) => {
  console.error('Component error:', error);
  toast.add({
    title: 'Something went wrong',
    description: 'Please try refreshing the page',
    color: 'error'
  });
  return false; // Prevent error from bubbling up
});
</script>
```

### 4. Use TypeScript (when available)
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  role?: {
    id: number;
    name: string;
  };
}

const { data } = await useApi<{ data: User[], total: number }>('/user_definition');
```

## Related Documentation

- **[Extension System](./extension-system.md)** - Using API in extensions
- **[Form System](./form-system.md)** - API integration in forms  
- **[Permission System](../server/permission-system.md)** - API permission handling
- **[SDK Documentation](https://github.com/dothinh115/enfyra-sdk-nuxt)** - Complete SDK reference

The Enfyra SDK provides everything you need for robust API integration with built-in caching, error handling, and TypeScript support.
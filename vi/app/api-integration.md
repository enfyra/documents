---
slug: ung-dung/tich-hop-api
---

# Tích hợp API

Enfyra sử dụng **thành phần kết hợp `useApi` cục bộ** cung cấp các tương tác API với khả năng xử lý lỗi nâng cao và các tính năng bổ sung. Thành phần kết hợp được xây dựng bằng cách sử dụng `$fetch` và xử lý tất cả các yêu cầu HTTP đến Máy chủ Enfyra.

Từ góc độ của ứng dụng, **tất cả các lệnh gọi API đều chuyển đến Máy chủ Enfyra (cổng 1105)**. Giao diện người dùng không bao giờ giao tiếp trực tiếp với cơ sở dữ liệu – nó chỉ gọi các điểm cuối HTTP do máy chủ hiển thị.

## Phụ thuộc phụ trợ

**TIÊU CHUẨN**: Tất cả lệnh gọi API ở giao diện người dùng đều sử dụng **`API_URL`** từ môi trường ứng dụng (xem `app/env_example`, thường là `http://localhost:1105/`). Thành phần kết hợp `useApi()` bên trong thực hiện các yêu cầu HTTP tới:
```
${API_URL}/api/${endpoint}
```
**Không có API nào tồn tại trên giao diện người dùng** - đó hoàn toàn là một khách hàng sử dụng API phụ trợ. Khi bạn tạo bảng, API sẽ được tạo trên máy chủ phụ trợ và được giao diện người dùng sử dụng thông qua các yêu cầu HTTP.

## Tổng quan về thành phần API

Ứng dụng Enfyra cung cấp:
- `useApi()` - API cục bộ có thể kết hợp với khả năng xử lý lỗi nâng cao (được khuyến nghị)
- Tích hợp hỗ trợ TypeScript và xử lý lỗi
- Kiểm soát thực thi thủ công (phải gọi `execute()` để chạy yêu cầu)
- Hỗ trợ tải lên tập tin và hoạt động hàng loạt

## useApi có thể kết hợp (Được khuyến nghị)

### Cách sử dụng cơ bản
```vue
<script setup>
// Fetch data with custom error handling
// This makes HTTP request to: ${API_URL}/enfyra/api/enfyra_user
const { data, pending, error, refresh, execute } = useApi('/enfyra_user', {
  query: {
    limit: 10,
    fields: 'id,email,name,role.name'
  }
});

// IMPORTANT: useApi does NOT auto-execute. You must call execute() to run the request
onMounted(() => {
  execute();
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
### Tùy chọn truy vấn nâng cao
```vue
<script setup>
// Complex query with filtering, sorting, and pagination
const { data, pending, error, execute } = useApi('/product', {
  query: computed(() => ({
    // Pagination
    page: currentPage.value,
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
  }))
});

// Execute the query
onMounted(() => {
  execute();
});

// Re-execute when page changes
watch(currentPage, () => {
  execute();
});
</script>
```
### Các thao tác POST/PATCH/DELETE
```vue
<script setup>
const toast = useToast();

// Create new record
// POST request to: ${API_URL}/enfyra/api/enfyra_user
const createUser = async (userData) => {
  const { data, error, execute } = useApi('/enfyra_user', {
    method: 'post',
    body: userData
  });
  
  await execute();
  
  if (error.value) {
    throw new Error(error.value.message);
  }
  
  toast.add({
    title: 'Success',
    description: `User created successfully`,
    color: 'success'
  });
  
  return data.value;
};

// Update existing record
// PATCH request to: ${API_URL}/enfyra/api/enfyra_user/${userId}
const updateUser = async (userId, updates) => {
  const { data, error, execute } = useApi('/enfyra_user', {
    method: 'patch',
    body: updates
  });
  
  await execute({ id: userId });
  
  if (error.value) {
    throw new Error(error.value.message);
  }
  
  return data.value;
};

// Delete record
// DELETE request to: ${API_URL}/enfyra/api/enfyra_user/${userId}
const deleteUser = async (userId) => {
  const { error, execute } = useApi('/enfyra_user', {
    method: 'delete'
  });
  
  await execute({ id: userId });
  
  if (error.value) {
    throw new Error(error.value.message);
  }
};

// File Upload (Batch)
// POST request with multiple files
const uploadFiles = async (files: File[]) => {
  const formDataArray = files.map((file) => {
    const formData = new FormData();
    formData.append('file', file);
    // Add additional fields if needed
    formData.append('folder', folderId);
    return formData;
  });
  
  const { data, error, execute } = useApi('/enfyra_file', {
    method: 'post'
  });
  
  await execute({ files: formDataArray });
  
  if (error.value) {
    throw new Error(error.value.message);
  }
  
  return data.value;
};
</script>
```
## Hỗ trợ MongoDB

Khi làm việc với cơ sở dữ liệu MongoDB, ứng dụng sẽ tự động xử lý các trường `_id` thay vì `id`. Tất cả các truy vấn và bộ lọc API đều sử dụng trình trợ giúp `getIdFieldName()` để hỗ trợ cả PostgreSQL (`id`) và MongoDB (`_id`).
```vue
<script setup>
const { getId, getIdFieldName } = useDatabase();

// Filter queries automatically use correct ID field
const { data, execute } = useApi('/enfyra_folder', {
  query: computed(() => {
    const idField = getIdFieldName();
    return {
      filter: {
        parent: {
          [idField]: {
            _is_null: true
          }
        }
      }
    };
  })
});

// Getting IDs from records
const folderId = getId(folder); // Returns id or _id based on database

// Always use getId() instead of direct .id access
const navigateToFolder = (folder) => {
  navigateTo(`/storage/management/folder/${getId(folder)}`);
};
</script>
```
## Tích hợp xác thực
```vue
<script setup>
// Authentication composable
const { me, isLoggedIn, login, logout } = useAuth();

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
## Tích hợp quyền

Các cuộc gọi API thường cần kiểm tra quyền. Enfyra cung cấp thành phần `PermissionGate` mạnh mẽ và thành phần kết hợp `usePermissions` để kiểm soát quyền truy cập vào chức năng API.

**[Hướng dẫn phân quyền đầy đủ](./permission-components.md)** – Tìm hiểu về `PermissionGate` và `usePermissions`
**[Trình dựng quyền](./permission-builder.md)** – Giao diện trực quan để tạo quy tắc phân quyền
```vue
<template>
  <!-- Only show button if user has permission -->
  <PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST'] }">
    <UButton @click="createUser">Create User</UButton>
  </PermissionGate>
</template>

<script setup>
// Check permissions programmatically
const { hasPermission } = usePermissions();

const deleteUser = async (userId) => {
  // Check permission before API call
  if (!hasPermission('/enfyra_user', 'DELETE')) {
    toast.add({
      title: 'Permission denied',
      color: 'error'
    });
    return;
  }
  
  await useApi(`/enfyra_user/${userId}`, {
    method: 'DELETE'
  });
};
</script>
```
## Tích hợp API trong Tiện ích mở rộng

### Ví dụ về tiện ích mở rộng trang tổng quan
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
const { data: userStats, pending: usersPending } = await useApi('/enfyra_user', {
  query: {
    limit: 0, // Get count only
    fields: 'id'
  },
  key: 'user-stats'
});

// Fetch product statistics
const { data: productStats, pending: productsPending } = await useApi('/product', {
  query: {
    limit: 0,
    fields: 'id'
  },
  key: 'product-stats'
});

// Fetch recent orders
const { data: ordersData, pending: ordersPending } = await useApi('/order', {
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
### Ví dụ về tiện ích mở rộng quản lý dữ liệu
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
      <PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST'] }">
        <UButton @click="showCreateModal = true" color="primary">
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
            <PermissionGate :condition="{ route: '/enfyra_user', methods: ['PATCH'] }">
              <UButton 
                @click="editUser(row)" 
                size="sm" 
                variant="outline"
              >
                Edit
              </UButton>
            </PermissionGate>
            
            <PermissionGate :condition="{ route: '/enfyra_user', methods: ['DELETE'] }">
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
    <CommonModal
      v-model:open="showCreateModal"
      :cancel-action="{ label: 'Cancel', onClick: () => (showCreateModal = false) }"
      :primary-action="{ label: editingUser ? 'Update' : 'Create', loading: saving, onClick: saveUser }"
    >
      <template #header>
        <h3>{{ editingUser ? 'Edit User' : 'Create User' }}</h3>
      </template>

      <template #body>
        <div class="space-y-4">
          <UInput v-model="userForm.name" label="Name" />
          <UInput v-model="userForm.email" label="Email" type="email" />
          <USelect v-model="userForm.roleId" label="Role" :options="roleOptions" />
          <UToggle v-model="userForm.isActive" label="Active" />
        </div>
      </template>
    </CommonModal>
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
const { data: usersData, pending, execute: fetchUsers } = useApi('/enfyra_user', {
  query: apiQuery
});

// Execute on mount and when query changes
onMounted(() => {
  fetchUsers();
});

watch(apiQuery, () => {
  fetchUsers();
}, { deep: true });

// Fetch roles for dropdown
const { data: rolesData, execute: fetchRoles } = useApi('/enfyra_role', {
  query: {
    fields: 'id,name',
    sort: 'name'
  }
});

onMounted(() => {
  fetchRoles();
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
  fetchUsers();
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
      const { execute: updateUser } = useApi('/enfyra_user', {
        method: 'patch',
        body: userForm
      });
      await updateUser({ id: editingUser.value.id });
      
      toast.add({
        title: 'User updated',
        description: `${userForm.name} has been updated successfully`,
        color: 'success'
      });
    } else {
      // Create new user
      const { execute: createUser } = useApi('/enfyra_user', {
        method: 'post',
        body: userForm
      });
      await createUser();
      
      toast.add({
        title: 'User created',
        description: `${userForm.name} has been created successfully`,
        color: 'success'
      });
    }
    
    showCreateModal.value = false;
    resetForm();
    fetchUsers();
    
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
    const { execute: deleteUserApi } = useApi('/enfyra_user', {
      method: 'delete'
    });
    await deleteUserApi({ id: userId });
    
    toast.add({
      title: 'User deleted',
      description: 'User has been deleted successfully',
      color: 'success'
    });
    
    fetchUsers();
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
## Các mẫu xử lý lỗi
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
  const { data, error, execute } = useApi('/endpoint');
  await execute();
  
  if (error.value) {
    handleApiError(error.value);
    throw error.value;
  }
  
  return data.value;
};
</script>
```
## Bộ nhớ đệm và hiệu suất
```vue
<script setup>
// Cache key strategies
const userListKey = computed(() => 
  `users-${currentPage.value}-${searchTerm.value}-${selectedRole.value}`
);

// Fetch with reactive query
const { data, pending, execute: fetchUsers } = useApi('/enfyra_user', {
  query: apiQuery
});

// Execute query
onMounted(() => {
  fetchUsers();
});

// Refresh data
const invalidateCache = () => {
  fetchUsers();
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
    const { execute: updateUser } = useApi('/enfyra_user', {
      method: 'patch',
      body: updates
    });
    await updateUser({ id: userId });
  } catch (error) {
    // Revert on error
    fetchUsers();
    throw error;
  }
};
</script>
```
## Các phương pháp hay nhất

### 1. Sử dụng truy vấn được tính toán
```vue
<script setup>
// Good - reactive query
const apiQuery = computed(() => ({
  filter: { name: { _contains: searchTerm.value } },
  page: currentPage.value
}));

const { data, execute } = useApi('/users', {
  query: apiQuery
});

onMounted(() => {
  execute();
});
</script>
```
### 2. Xử lý trạng thái tải
```vue
<template>
  <div>
    <USkeleton v-if="pending" class="h-20" />
    <UserCard v-else :user="data" />
  </div>
</template>
```
### 3. Thực hiện ranh giới lỗi thích hợp
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
### 4. Sử dụng TypeScript (nếu có)
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

const { data, execute } = useApi<{ data: User[], total: number }>('/enfyra_user');

onMounted(() => {
  execute();
});
```
## Tài liệu liên quan

- **[Hệ thống tiện ích mở rộng](./extension-system.md)** - Sử dụng API trong tiện ích mở rộng và tiện ích
- **[Hệ thống biểu mẫu](./form-system.md)** – Cách biểu mẫu tích hợp với API backend  
- **[Thành phần quyền](./permission-components.md)** – Kiểm soát quyền truy cập giao diện quanh lệnh gọi API  
- **[Trình dựng quyền](./permission-builder.md)** – Cách đánh giá và cấu hình quyền
- **[Tài liệu máy chủ](../server/README.md)** - Tổng quan về ngữ cảnh và API phụ trợ

Thành phần kết hợp `useApi` cung cấp mọi thứ bạn cần để tích hợp API mạnh mẽ với khả năng xử lý lỗi, hỗ trợ TypeScript và khả năng tương thích MongoDB.

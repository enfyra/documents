---
slug: ung-dung/thanh-phan-quyen
---

# Các thành phần phân quyền

Enfyra cung cấp hai công cụ chính để kiểm soát việc hiển thị giao diện theo quyền của người dùng: component `PermissionGate` để render theo kiểu khai báo và composable `usePermissions` để kiểm tra quyền bằng code.

## Component PermissionGate

Component `PermissionGate` bao bọc các phần tử giao diện và tự động hiển thị hoặc ẩn chúng theo quyền của người dùng.

### Cách sử dụng cơ bản

```vue
<PermissionGate :condition="{ route: '/enfyra_user', methods: ['GET'] }">
  <div>This content only shows if user can read users</div>
</PermissionGate>
```

### Nhiều hành động

Kiểm tra xem người dùng có BẤT KỲ hành động nào trong danh sách hay không:

```vue
<PermissionGate :condition="{ route: '/enfyra_user', methods: ['POST', 'PATCH'] }">
  <UButton>Edit User</UButton>
</PermissionGate>
```

### Điều kiện phức tạp

#### Logic AND

Người dùng phải có TẤT CẢ các quyền:

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

#### Logic OR

Người dùng chỉ cần có BẤT KỲ quyền nào sau đây:

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

#### Điều kiện lồng nhau

Kết hợp AND và OR để biểu diễn logic phức tạp:

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

## Composable usePermissions

Composable `usePermissions` cho phép kiểm tra quyền bằng code trong các component Vue.

### Thiết lập

```vue
<script setup lang="ts">
const { hasPermission, checkPermissionCondition } = usePermissions();
</script>
```

### Kiểm tra một quyền cụ thể

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

### Kiểm tra điều kiện phức tạp

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

### Ánh xạ phương thức HTTP

Hệ thống ánh xạ các hành động sang phương thức HTTP như sau:

| Hành động | Phương thức HTTP | Mục đích |
|-----------|------------------|----------|
| `read` | GET | Xem hoặc liệt kê dữ liệu |
| `create` | POST | Tạo bản ghi mới |
| `update` | PATCH | Sửa bản ghi hiện có |
| `delete` | DELETE | Xóa bản ghi |

## Tích hợp với hệ thống menu

Hệ thống menu sử dụng cả `PermissionGate` và `usePermissions` ở bên trong để kiểm soát việc hiển thị menu.

### Cách menu sử dụng quyền

Khi bạn đặt quyền cho một mục menu:

```javascript
// Menu configuration
{
  label: 'User Management',
  route: '/settings/users',
  permission: {
    or: [
      { route: '/users', methods: ['GET'] },
      { route: '/users', methods: ['POST'] }
    ]
  }
}
```

Hệ thống menu sẽ:

1. Dùng `checkPermissionCondition` để đánh giá điều kiện quyền.
2. Chỉ render mục menu khi điều kiện quyền được đáp ứng.
3. Tự động ẩn menu cha nếu người dùng không thể truy cập bất kỳ mục con nào.

### Các component menu

**Hệ thống menu** sử dụng quyền:

```vue
// Internally filters menu items based on permissions
const visibleItems = menuGroups.filter(item => {
  if (!item.permission) return true;
  return checkPermissionCondition(item.permission);
});
```

**Các mục menu** được bao bọc bằng `PermissionGate`:

```vue
<PermissionGate :condition="menuItem.permission">
  <MenuItem :item="menuItem" />
</PermissionGate>
```

**Kết quả**: Menu tự động thay đổi theo quyền của người dùng mà không cần cấu hình thủ công.

## Các mẫu sử dụng phổ biến

### Nút theo điều kiện

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

### Hành động trong bảng

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

### Gửi biểu mẫu

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

### Hành động trên đầu trang

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

## Các trường hợp đặc biệt

### Quản trị viên root

Quản trị viên root bỏ qua mọi bước kiểm tra quyền:

```vue
<script setup lang="ts">
const { me } = useAuth();

// Root admin has all permissions automatically
if (me.value?.isRootAdmin) {
  // All permission checks return true
}
</script>
```

### Cho phép tất cả

Cấp quyền truy cập không giới hạn; chỉ nên dùng khi thật sự cần thiết:

```vue
<PermissionGate :condition="{ allowAll: true }">
  <div>Always visible content</div>
</PermissionGate>
```

### Quyền gán trực tiếp cho người dùng

Người dùng có thể được gán quyền trực tiếp, không phụ thuộc vào vai trò của họ:

```javascript
// User's direct permissions override role permissions
// Checked automatically by usePermissions
```

## Thực hành tốt

### Dùng PermissionGate cho giao diện

- Bao bọc nút, mục menu và từng khu vực giao diện.
- Giữ template gọn gàng và theo kiểu khai báo.
- Tự động xử lý khi quyền thay đổi.

### Dùng usePermissions cho logic

- Dùng trong logic nghiệp vụ và bước xác thực dữ liệu.
- Dùng computed property cho các điều kiện phức tạp.
- Kiểm tra trước khi gọi API hoặc xử lý dữ liệu.

### Lưu kết quả kiểm tra quyền bằng computed

```vue
<script setup lang="ts">
// Good - computed property caches result
const canEdit = computed(() => hasPermission('/users', 'PATCH'));

// Avoid - checking in template repeatedly
// <div v-if="hasPermission('/users', 'PATCH')">
</script>
```

### Khớp với route API

Luôn sử dụng đường dẫn endpoint API thực tế:

```javascript
// Good - matches API endpoint
{ route: '/enfyra_user', methods: ['GET'] }

// Bad - doesn't match actual route
{ route: '/users', methods: ['GET'] }
```

## Gỡ lỗi

### Kiểm tra các quyền hiện tại

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

### Kiểm tra điều kiện quyền

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

## Tài liệu liên quan

- **[Trình tạo quyền](./permission-builder.md)** - Kiến trúc phân quyền phía backend
- **[Trình tạo quyền](./permission-builder.md)** - Cấu hình quyền bằng giao diện trực quan
- **[Quản lý menu](./menu-management.md)** - Cách menu sử dụng quyền
- **[Hệ thống biểu mẫu](./form-system.md)** - Tích hợp quyền trong biểu mẫu

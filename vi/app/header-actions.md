---
slug: ung-dung/hanh-dong-dau-trang
---

# Hệ thống hành động tiêu đề

Tiện ích mở rộng có thể tự động đưa các hành động tùy chỉnh vào khu vực tiêu đề và tiêu đề phụ của ứng dụng, cung cấp khả năng tích hợp liền mạch với giao diện cốt lõi. **Điều này thể hiện sức mạnh đáng kinh ngạc của hệ thống tiện ích mở rộng của Enfyra - các tiện ích mở rộng có thể can thiệp và tùy chỉnh BẤT KỲ phần nào của giao diện ứng dụng.**

 **Bản demo trực tiếp**: [https://demo.enfyra.io/](https://demo.enfyra.io/) - Xem các hành động tiêu đề đang hoạt động!

## Mục lục

- [Tổng quan](#tong-quan)
- [Hành động tiêu đề và hành động tiêu đề phụ](#hanh-dong-tieu-de-so-voi-hanh-dong-tieu-de-phu)
- [Loại hành động](#loai-hanh-dong)
- [Tích hợp quyền](#tich-hop-quyen)
- [Cách sử dụng cơ bản](#cach-su-dung-co-ban)
- [Tính năng nâng cao](#tinh-nang-nang-cao)
- [Định vị và bố cục](#dinh-vi-va-bo-cuc)
- [Hành động dựa trên tuyến đường](#hanh-dong-dua-tren-lo-trinh)
- [Thành phần tùy chỉnh](#thanh-phan-tuy-chinh)
- [Ví dụ trong thế giới thực](#vi-du-trong-the-gioi-thuc)
- [Các phương pháp hay nhất](#cac-phuong-phap-hay-nhat)
- [Tham khảo API](#tham-khao-api)

## Tổng quan

** Quyền lực mở rộng **: Tiện ích mở rộng hoàn toàn có thể kiểm soát giao diện ứng dụng thông qua các hành động tiêu đề. Đây chỉ là một ví dụ về cách tiện ích mở rộng Enfyra có thể **can thiệp vào BẤT KỲ phần nào của ứng dụng**, khiến chúng trở nên cực kỳ mạnh mẽ để tùy chỉnh.

Hệ thống Hành động tiêu đề cung cấp hai lĩnh vực chính cho hành động tùy chỉnh:

- **Hành động tiêu đề**: Nằm ở góc trên bên phải của tiêu đề chính
- **Hành động tiêu đề phụ**: Nằm trong tiêu đề phụ của trang, có thể được đặt ở vị trí bên trái hoặc bên phải

Cả hai hệ thống đều **nhận biết quyền** và **phân biệt tuyến đường**, cho phép tùy chỉnh phức tạp dựa trên vai trò của người dùng và ngữ cảnh trang hiện tại.

## Hành động tiêu đề so với Hành động tiêu đề phụ

### Hành động tiêu đề (Góc trên bên phải)
```
┌─────────────────────────────────────────────────────┐
│ Enfyra App                       [Hành động tiêu đề] │
└─────────────────────────────────────────────────────┘
```
### Hành động tiêu đề phụ (Cấp trang)
```
┌─────────────────────────────────────────────────────┐
│ [Hành động trái]              [Hành động phải] │
└─────────────────────────────────────────────────────┘
│                    Nội dung trang                     │
```
## Loại hành động

### 1. Thao tác với nút
Các nút tiêu chuẩn có biểu tượng, nhãn và trình xử lý nhấp chuột:
```javascript
const buttonAction = {
  id: 'export-data',
  label: 'Export',
  variant: 'solid',
  color: 'primary',
  onClick: () => exportData()
};
```
### 2. Hành động thành phần tùy chỉnh
Tiêm các thành phần hoàn toàn tùy chỉnh:
```javascript
const componentAction = {
  id: 'custom-widget',
  component: 'CustomWidget',
  props: { data: dynamicData }
};
```
## Tích hợp quyền

** Mọi hành động đều được tự động bao bọc bằng PermissionGate** - hệ thống cấp phép mạnh mẽ tương tự được sử dụng trên khắp Enfyra.

**[Tìm hiểu về hệ thống phân quyền](./permission-components.md)**
```javascript
const permissionAction = {
  id: 'admin-action',
  label: 'Admin Panel',
  onClick: () => openAdminPanel(),
  permission: {
    route: '/admin',
    methods: ['GET']
  }
};
```
**Tính năng cấp phép:**
- Hành động tự động ẩn nếu người dùng thiếu sự cho phép
- Hỗ trợ các điều kiện cấp phép AND/OR phức tạp
- Hoạt động với kiểm soát truy cập dựa trên vai trò
- Kiểm tra quyền động dựa trên ngữ cảnh dữ liệu

** [Hướng dẫn cấp phép đầy đủ](./permission-builder.md)**

## Cách sử dụng cơ bản

### Hành động tiêu đề trong tiện ích mở rộng
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
// Available globally in extensions
registerHeaderActions([
  {
    id: 'save-report',
    label: 'Save Report',
    color: 'success',
    onClick: () => {
      toast.add({
        title: 'Report saved!',
        color: 'success'
      });
    },
    permission: {
      route: '/reports',
      methods: ['POST']
    }
  }
]);
</script>
```
### Hành động tiêu đề phụ trong tiện ích mở rộng
```vue
<script setup>
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
// Available globally in extensions
registerSubHeaderActions([
  {
    id: 'filter-toggle',
    label: 'Filters',
    side: 'left', // Position on left side
    variant: 'soft',
    onClick: () => toggleFilters()
  }
]);
</script>
```
## Tính năng nâng cao

### Thuộc tính phản ứng
Tất cả các thuộc tính có thể phản ứng bằng cách sử dụng ref hoặc được tính toán:
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const isLoading = ref(false);
const itemCount = ref(0);

const dynamicLabel = computed(() =>
  `Export (${itemCount.value} items)`
);

registerHeaderActions([
  {
    id: 'dynamic-export',
    label: dynamicLabel,
    loading: isLoading,
    disabled: computed(() => itemCount.value === 0),
    onClick: async () => {
      isLoading.value = true;
      await exportItems();
      isLoading.value = false;
    }
  }
]);
</script>
```
### Đăng ký nhiều hành động
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const actions = [
  {
    id: 'refresh',
    onClick: () => refresh()
  },
  {
    id: 'export',
    label: 'Export',
    onClick: () => exportData()
  },
  {
    id: 'settings',
    onClick: () => openSettings()
  }
];

// Register all at once
registerHeaderActions(actions);
</script>
```
## Định vị và bố cục

### Định vị tiêu đề phụ
```vue
<script setup>
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
// Left side actions (typically filters, views)
registerSubHeaderActions([
  {
    id: 'view-toggle',
    label: 'Grid View',
    side: 'left',
    onClick: () => toggleView()
  },
  {
    id: 'export',
    label: 'Export',
    side: 'right', // Default
    onClick: () => exportData()
  }
]);
</script>
```
### Hành vi đáp ứng
Các hành động tự động thích ứng với kích thước màn hình:
- Di động: Chỉ hiển thị biểu tượng
- Desktop: Biểu tượng + nhãn hiển thị
- Máy tính bảng: Nút nhỏ hơn

## Hành động dựa trên lộ trình

### Hiện/Ẩn trên các tuyến đường cụ thể
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
registerHeaderActions([
  {
    id: 'route-specific',
    label: 'Data Tools',
    showOn: ['/data'], // Only show on data routes
    hideOn: ['/settings'], // Hide on settings routes
    onClick: () => openDataTools()
  }
]);
</script>
```
### Hành động toàn cầu và hành động cụ thể theo tuyến đường
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
// Register multiple actions
registerHeaderActions([
  {
    id: 'global-help',
    global: true, // Persists across all routes
    onClick: () => openHelp()
  },
  {
    id: 'page-specific',
    label: 'Page Action', // Cleared when navigating away
    onClick: () => doPageSpecificAction()
  }
]);
</script>
```
## Thành phần tùy chỉnh

### Chèn các thành phần tùy chỉnh
```vue
<template>
  <div id="my-extension">
    <!-- Extension content -->
  </div>
</template>

<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
// Define a custom component
const CustomStatusWidget = {
  template: `
    <div class="flex items-center gap-2">
      <UBadge :color="status.color">{{ status.text }}</UBadge>
      <UButton @click="refresh" variant="ghost" size="sm" />
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

// Register as header action
registerHeaderActions([
  {
    id: 'status-widget',
    component: CustomStatusWidget
  }
]);
</script>
```
### Thành phần có Đạo cụ
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const DataCounter = {
  template: `
    <UBadge variant="soft" color="primary">
      {{ count }} items
    </UBadge>
  `,
  props: ['count']
};

const itemCount = ref(150);

registerHeaderActions([
  {
    id: 'data-counter',
    component: DataCounter,
    props: {
      count: itemCount.value
    }
  }
]);
</script>
```
## Ví dụ trong thế giới thực

### 1. Tiện ích xuất dữ liệu
```vue
<template>
  <div class="p-6">
    <h2>Data Management</h2>
  </div>
</template>

<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const isExporting = ref(false);
const selectedFormat = ref('json');

const exportData = async () => {
  isExporting.value = true;
  try {
    const { data } = await useApi('/export', {
      method: 'POST',
      body: { format: selectedFormat.value }
    });

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

// Export dropdown component
const ExportDropdown = {
  template: `
    <UDropdown :items="exportItems">
      <UButton
        label="Export"
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
          click: () => {
            selectedFormat.value = 'json';
            exportData();
          }
        },
        {
          label: 'CSV',
          click: () => {
            selectedFormat.value = 'csv';
            exportData();
          }
        },
        {
          label: 'PDF',
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

// Register at top level - no onMounted needed
registerHeaderActions([
  {
    id: 'export-dropdown',
    component: ExportDropdown,
    permission: {
      route: '/data',
      methods: ['GET']
    }
  }
]);
</script>
```
### 2. Bảng thông tin trạng thái theo thời gian thực
```vue
<template>
  <div class="dashboard p-6">
    <h1>System Dashboard</h1>
    <!-- Dashboard content -->
  </div>
</template>

<script setup>
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
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

// Register status in sub-header left
registerSubHeaderActions([
  {
    id: 'connection-status',
    component: StatusIndicator,
    side: 'left'
  }
]);

// Register controls in sub-header right
registerSubHeaderActions([
  {
    id: 'auto-refresh-toggle',
    label: computed(() => autoRefresh.value ? 'Auto-refresh ON' : 'Auto-refresh OFF'),
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
  }
]);

onMounted(() => {

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
### 3. Hành động dựa trên quyền nâng cao
```vue
<template>
  <div class="admin-panel p-6">
    <h2>Administrative Tools</h2>
  </div>
</template>

<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
const { me } = useAuth();
const userRole = computed(() => me.value?.role?.name);

// Define actions at top level - no onMounted needed
const adminActions = [
  {
    id: 'system-settings',
    label: 'System',
    permission: {
      route: '/admin',
      methods: ['PATCH']
    },
    onClick: () => navigateTo('/admin/system')
  },
  {
    id: 'user-management',
    label: 'Users',
    permission: {
      route: '/enfyra_user',
      methods: ['POST', 'PATCH', 'DELETE']
    },
    onClick: () => navigateTo('/admin/users')
  },
  {
    id: 'audit-logs',
    label: 'Audit',
    permission: {
      route: '/audit_log',
      methods: ['GET']
    },
    onClick: () => navigateTo('/admin/audit')
  }
];

// Super admin only actions
const superAdminActions = [
  {
    id: 'danger-zone',
    label: 'Danger Zone',
    color: 'error',
    permission: {
      and: [
        { route: '/admin', methods: ['DELETE'] },
        { allowedUsers: [me.value?.id] }
      ]
    },
    onClick: () => openDangerZone()
  }
];

// Register actions
registerHeaderActions([...adminActions, ...superAdminActions]);

// Conditional registration based on user role
if (userRole.value === 'manager') {
  registerSubHeaderActions([
    {
      id: 'team-overview',
      label: `Team Overview (${me.value.team?.memberCount || 0})`,
      side: 'left',
      onClick: () => showTeamOverview()
    }
  ]);
}
</script>
```
## Các phương pháp hay nhất

### 1. Sử dụng ID có ý nghĩa
```javascript
// Good
{ id: 'export-user-data', label: 'Export Users' }

// Avoid
{ id: 'btn1', label: 'Export Users' }
```
### 2. Thực hiện kiểm tra quyền
```javascript
// Always include permission conditions
{
  id: 'delete-action',
  label: 'Delete',
  color: 'error',
  permission: {
    route: '/data',
    methods: ['DELETE']
  }
}
```
### 3. Cung cấp trạng thái tải
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
### 4. Sử dụng vị trí phù hợp
```javascript
// Left side: Views, filters, status
{ side: 'left', id: 'filter-toggle' }

// Right side: Actions, exports, settings
{ side: 'right', id: 'export-data' }
```
### 5. Tự động dọn dẹp
```javascript
// Actions are automatically cleaned up on route change
// Only actions with global: true persist across routes
// No manual cleanup needed in most cases
```
### 6. Tài liệu liên quan đến tham chiếu chéo
Khi làm việc với quyền:
- **[Trình dựng quyền](./permission-builder.md)** – Tạo quy tắc phân quyền phức tạp
- **[Thành phần quyền](./permission-components.md)** – Cách dùng `PermissionGate`
- **[Hệ thống quyền](./permission-components.md)** – Kiến trúc phân quyền ở backend

## Tham khảo API

### Giao diện hành động tiêu đề
```typescript
interface HeaderAction {
  // Core Properties
  id: string;                                    // Unique identifier
  label?: string | ComputedRef<string>;          // Button text

  
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
### Sổ đăng ký hành động tiêu đề
```typescript
const { register: registerHeaderActions } = useHeaderActionRegistry();
// Accepts a single action object or an array of actions
registerHeaderActions(actionOrActions)
```
### Sổ đăng ký hành động tiêu đề phụ
```typescript
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
// Accepts a single action object or an array of actions
registerSubHeaderActions(actionOrActions)
```
## Tóm tắt

Hệ thống Hành động tiêu đề thể hiện **sức mạnh đáng kinh ngạc của các tiện ích mở rộng Enfyra** - chúng có thể tích hợp liền mạch và kiểm soát BẤT KỲ phần nào của giao diện ứng dụng. Đây chỉ là một ví dụ về cách tiện ích mở rộng có thể:

 **Can thiệp vào các khu vực giao diện người dùng cốt lõi** - Tiêu đề, thanh bên, biểu mẫu, bảng
 **Thêm chức năng tùy chỉnh** - Nút, tiện ích, thành phần hoàn chỉnh  
 **Tôn trọng hệ thống cấp phép** - Tích hợp PermissionGate tự động
 **Phản hồi các thay đổi tuyến đường** - Hành vi động dựa trên trang hiện tại
 **Cung cấp thông tin cập nhật theo thời gian thực** - Thuộc tính phản ứng và dữ liệu trực tiếp

** Tiện ích mở rộng không giới hạn ở các trang tùy chỉnh - chúng có thể nâng cao và tùy chỉnh mọi khía cạnh của trải nghiệm Enfyra, khiến chúng trở nên cực kỳ mạnh mẽ để xây dựng chính xác những gì bạn cần.**

**Tài liệu liên quan:**
- **[Hệ thống tiện ích mở rộng](./extension-system.md)** - Hướng dẫn phát triển tiện ích mở rộng hoàn chỉnh
- **[Thành phần quyền](./permission-components.md)** – `PermissionGate` và tích hợp quyền
- **[Tích hợp API](./api-integration.md)** – Lấy dữ liệu cho hành động động

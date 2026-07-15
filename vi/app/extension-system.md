---
slug: ung-dung/he-thong-mo-rong
---

# Hệ thống mở rộng

Hệ thống tiện ích mở rộng cho phép bạn tạo các trang, tiện ích tùy chỉnh và tích hợp shell toàn cầu bằng cách sử dụng các thành phần Vue.js. Tiện ích mở rộng trang kết hợp với các mục menu để cung cấp nội dung định tuyến, tiện ích mở rộng tiện ích được nhúng bên trong các trang và tiện ích mở rộng chung được gắn kết một lần ở cấp vỏ ứng dụng cho logic đăng ký trên toàn ứng dụng.

**[Hướng dẫn quản lý menu](./menu-management.md)** – Tìm hiểu cách tạo và cấu hình menu

## Mục lục

- [Tìm hiểu về tiện ích mở rộng và menu](#tim-hieu-ve-tien-ich-mo-rong-va-menu)
- [Ví dụ về quy trình làm việc hoàn chỉnh](#vi-du-ve-quy-trinh-lam-viec-hoan-chinh-tu-menu-den-hien-thi-tien-ich-mo-rong)
- [Các loại tiện ích mở rộng](#cac-loai-tien-ich-mo-rong)
- [Quyền truy cập SDK đầy đủ](#quyen-truy-cap-sdk-day-du-trong-tien-ich-mo-rong)
- [Danh sách đầy đủ các tài nguyên được đưa vào](#danh-sach-day-du-cac-tai-nguyen-duoc-dua-vao)
  - [Thành phần giao diện người dùng](#thanh-phan-giao-dien-nguoi-dung-duoc-chen-tu-dong)
  - [Thành phần kết hợp Enfyra](#thanh-phan-ket-hop-enfyra-truy-cap-toan-cau)
  - [Thành phần kết hợp Nuxt](#thanh-phan-ket-hop-nuxt-truy-cap-toan-cau)
  - [Composition API của Vue 3](#composition-api-cua-vue-3-truy-cap-toan-cau)
  - [API trình duyệt](#api-trinh-duyet-co-san)
- [Tính năng tiện ích mở rộng nâng cao](#tinh-nang-mo-rong-nang-cao)
- [Tích hợp hành động tiêu đề](#tich-hop-hanh-dong-tieu-de)
- [Hệ thống widget](#he-thong-tien-ich)
- [Hệ thống mở rộng toàn cầu](#he-thong-mo-rong-toan-cau)
- [Hỗ trợ tải lên tệp](#ho-tro-tai-len-tep)
- [Sử dụng gói NPM trong tiện ích mở rộng](#su-dung-goi-npm-trong-tien-ich-mo-rong)
- [Quản lý tiện ích mở rộng](#quan-ly-tien-ich-mo-rong)
- [Các phương pháp hay nhất](#cac-phuong-phap-hay-nhat)
- [Các vấn đề thường gặp và giải pháp](#cac-van-de-thuong-gap-va-giai-phap)
- [Mẫu nâng cao](#mau-nang-cao)
- [Cân nhắc về bảo mật](#can-nhac-ve-bao-mat)
- [Tóm tắt](#tom-tat)

## Tìm hiểu về Tiện ích mở rộng và Menu

### Vấn đề
Khi bạn tạo một mục menu có đường dẫn tùy chỉnh như `/reports/sale`, việc nhấp vào mục đó sẽ dẫn đến một trang trống vì không có mã để xử lý tuyến đường đó.

### Giải pháp
Tiện ích mở rộng cung cấp thành phần Vue.js hiển thị khi người dùng điều hướng đến đường dẫn menu tùy chỉnh của bạn. Điều này tạo ra trải nghiệm người dùng hoàn chỉnh:
1. **Menu** = Điểm vào điều hướng
2. **Tiện ích mở rộng** = Nội dung trang thực tế

## Ví dụ về quy trình làm việc hoàn chỉnh: Từ Menu đến Hiển thị tiện ích mở rộng

Ví dụ này cho thấy toàn bộ quá trình từ tạo menu đến hiển thị nội dung tùy chỉnh.

### Bước 1: Tạo Mục Menu
1. Điều hướng đến **Cài đặt > Menu**
2. Nhấp vào **"Tạo"** để thêm menu mới
3. Cấu hình menu của bạn:
   - **Loại**: Chọn "Menu" (đối với các mục menu thông thường)
   - **Nhãn**: "Trang tổng quan phân tích"
   - **Đường dẫn**: `/custom/analytics` (đây sẽ là URL của tiện ích mở rộng của bạn)
   - **Biểu tượng**: Chọn ""
   - **Thanh bên**: Chọn "Trang tổng quan" (để nó xuất hiện dưới thanh bên Trang tổng quan)
4. Lưu mục menu

**Kết quả**: Bây giờ bạn có một mục menu nhưng việc nhấp vào nó sẽ hiển thị một trang trống vì không có tiện ích mở rộng nào được liên kết.

### Bước 2: Tạo tiện ích mở rộng
1. Điều hướng đến **Cài đặt > Tiện ích mở rộng**
2. Nhấp vào **"Tạo tiện ích mở rộng"**
3. Điền thông tin chi tiết về tiện ích mở rộng:
   - **Tên**: "Trang tổng quan phân tích"
   - **ID tiện ích mở rộng**: Được tạo tự động (ví dụ: "analytics-dashboard-1234")
   - **Loại**: Chọn "Trang" (đối với tiện ích mở rộng được liên kết với menu)
   - **Mô tả**: "Trang tổng quan phân tích tùy chỉnh với biểu đồ và số liệu"
   - **Menu**: Sử dụng bộ chọn quan hệ để **chọn menu bạn vừa tạo**
   - **Phiên bản**: "1.0.0"
   - **Đã bật**: Chọn hộp này

### Bước 3: Viết mã mở rộng của bạn
Trong trình chỉnh sửa mã, hãy viết Thành phần tệp đơn Vue.js (SFC) của bạn.

** Ví dụ hoàn chỉnh về việc sao chép-dán:**Ví dụ này thể hiện tất cả các tính năng và có thể được dán trực tiếp vào trình chỉnh sửa tiện ích mở rộng:
```vue
<template>
  <div class="p-6 space-y-6">
    <!-- Header Section -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-3xl font-bold">Dashboard Example</h1>
        <p class="text-gray-500 dark:text-gray-400">
          Complete working example - copy and paste this code
        </p>
      </div>
      <UBadge color="success" variant="soft">
        
        Live Data
      </UBadge>
    </div>

    <!-- Stats Cards Grid -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <UCard>
        <div class="text-center p-4">
          
          <div class="text-2xl font-bold text-blue-600">{{ stats.users }}</div>
          <div class="text-sm text-gray-500">Total Users</div>
        </div>
      </UCard>

      <UCard>
        <div class="text-center p-4">
          
          <div class="text-2xl font-bold text-green-600">{{ stats.revenue }}</div>
          <div class="text-sm text-gray-500">Monthly Revenue</div>
        </div>
      </UCard>

      <UCard>
        <div class="text-center p-4">
          
          <div class="text-2xl font-bold text-purple-600">{{ stats.orders }}</div>
          <div class="text-sm text-gray-500">Orders Today</div>
        </div>
      </UCard>
    </div>

    <!-- Action Buttons -->
    <div class="flex flex-wrap gap-4">
      <UButton
        @click="refreshData"
        :loading="loading"
        color="primary"
       
      >
        Refresh Data
      </UButton>

      <UButton
        @click="fetchFromAPI"
        :loading="apiLoading"
        variant="outline"
       
      >
        Fetch Real Data
      </UButton>

      <PermissionGate :condition="{ route: '/admin', methods: ['POST'] }">
        <UButton
          @click="generateReport"
          variant="soft"
          color="success"
         
        >
          Generate Report (Admin Only)
        </UButton>
      </PermissionGate>
    </div>

    <!-- Data Table -->
    <UCard>
      <template #header>
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-semibold">Recent Activity</h3>
          <UBadge variant="soft">{{ recentActivity.length }} items</UBadge>
        </div>
      </template>

      <UTable :rows="recentActivity" :columns="columns">
        <template #action-data="{ row }">
          <div class="flex items-center gap-2">
            
            {{ row.action }}
          </div>
        </template>

        <template #time-data="{ row }">
          <UBadge variant="soft" size="xs">{{ row.time }}</UBadge>
        </template>
      </UTable>
    </UCard>

    <!-- Form Example -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">Quick Add Form</h3>
      </template>

      <div class="space-y-4">
        <UInput
          v-model="formData.name"
          placeholder="Enter name"
          label="Name"
         
        />

        <UTextarea
          v-model="formData.description"
          placeholder="Enter description"
          label="Description"
          :rows="3"
        />

        <USelect
          v-model="formData.category"
          :options="categories"
          label="Category"
          placeholder="Select category"
        />

        <div class="flex items-center gap-4">
          <USwitch v-model="formData.isActive" label="Active" />
          <UCheckbox v-model="formData.isPublic" label="Public" />
        </div>

        <UButton
          @click="submitForm"
          color="primary"
          block
          :disabled="!isFormValid"
        >
          Submit Form
        </UButton>
      </div>
    </UCard>
  </div>
</template>

<script setup>
const {
  register: registerHeaderActions,
  unregister: unregisterHeaderAction
} = useHeaderActionRegistry();
// ==========================================
// ALL FUNCTIONS ARE GLOBALLY AVAILABLE
// No imports needed - just use them directly!
// ==========================================

// Nuxt & Enfyra Composables - Available globally
const toast = useToast();
const { me } = useAuth();
const router = useRouter();
const route = useRoute();

// Vue Composition API - Available globally
const loading = ref(false);
const apiLoading = ref(false);

// Reactive state
const stats = reactive({
  users: 1234,
  revenue: '$12,450',
  orders: 89
});

const recentActivity = ref([
  { id: 1, action: 'User login', user: 'john@example.com', time: '2 mins ago' },
  { id: 2, action: 'New order', user: 'jane@example.com', time: '5 mins ago' },
  { id: 3, action: 'Payment received', user: 'bob@example.com', time: '10 mins ago' },
  { id: 4, action: 'Profile update', user: 'alice@example.com', time: '15 mins ago' },
  { id: 5, action: 'Password reset', user: 'david@example.com', time: '20 mins ago' }
]);

// Table columns configuration
const columns = [
  { key: 'action', label: 'Action', sortable: true },
  { key: 'user', label: 'User', sortable: true },
  { key: 'time', label: 'Time' }
];

// Form data
const formData = reactive({
  name: '',
  description: '',
  category: null,
  isActive: true,
  isPublic: false
});

const categories = [
  { value: 'product', label: 'Product' },
  { value: 'service', label: 'Service' },
  { value: 'other', label: 'Other' }
];

// Computed properties
const isFormValid = computed(() => {
  return formData.name && formData.description && formData.category;
});

// Helper function for action icons
const getActionIcon = (action) => {
  const icons = {
    'User login': '',
    'New order': '',
    'Payment received': '',
    'Profile update': '',
    'Password reset': ''
  };
  return icons[action] || '';
};

// Methods
const refreshData = async () => {
  loading.value = true;

  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 1000));

  // Update stats with random values
  stats.users += Math.floor(Math.random() * 10);
  stats.orders += Math.floor(Math.random() * 5);

  toast.add({
    title: 'Success',
    description: 'Data has been refreshed',
    color: 'green',
  });

  loading.value = false;
};

// Fetch real data from API
const fetchFromAPI = async () => {
  apiLoading.value = true;

  // useApi already handles errors - no try-catch needed
  const { data, error } = await useApi('/enfyra_user', {
    query: {
      limit: 5,
      fields: 'id,email,created_at'
    }
  });

  if (error.value) {
    toast.add({
      title: 'API Error',
      description: error.value.message || 'Failed to fetch data',
      color: 'red'
    });
  } else if (data.value?.data) {
    // Update activity with real data
    recentActivity.value = data.value.data.map((user, index) => ({
      id: user.id,
      action: 'User registered',
      user: user.email,
      time: `${index + 1} days ago`
    }));

    toast.add({
      title: 'Data Loaded',
      description: `Fetched ${data.value.data.length} records from API`,
      color: 'green'
    });
  }

  apiLoading.value = false;
};

const generateReport = () => {
  toast.add({
    title: 'Report Generated',
    description: 'Your report is ready for download',
    color: 'blue',
    timeout: 5000,
    actions: [{
      label: 'Download',
      color: 'white',
      click: () => {
        toast.add({
          title: 'Downloading...',
          description: 'Report download started'
        });
      }
    }]
  });
};

const submitForm = async () => {
  if (!isFormValid.value) return;

  // Example: Send data to API with proper error handling
  // const { data, error } = await useApi('/my-endpoint', {
  //   method: 'POST',
  //   body: formData
  // });
  //
  // if (error.value) {
  //   toast.add({
  //     title: 'Submission Failed',
  //     description: error.value.message,
  //     color: 'red'
  //   });
  //   return;
  // }

  toast.add({
    title: 'Form Submitted',
    description: `Created: ${formData.name}`,
    color: 'green'
  });

  // Reset form
  formData.name = '';
  formData.description = '';
  formData.category = null;
  formData.isActive = true;
  formData.isPublic = false;
};

// Register header actions when component mounts
onMounted(() => {
  // Add custom actions to app header
  registerHeaderActions([
    {
      id: 'refresh-dashboard',
      label: 'Refresh',
      onClick: refreshData,
      color: 'primary',
      variant: 'soft'
    },
    {
      id: 'view-settings',
      label: 'Settings',
      variant: 'ghost',
      onClick: () => {
        navigateTo('/settings');
      }
    }
  ]);

  // Log current user info
  if (me.value) {
    console.log('Extension loaded for user:', me.value.email);
  }

  // Example: Check permissions
  const { hasPermission } = usePermissions();
  if (hasPermission('/admin', 'GET')) {
    console.log('User has admin read access');
  }
});

// Cleanup when component unmounts
onUnmounted(() => {
  // Unregister header actions
  unregisterHeaderAction('refresh-dashboard');
  unregisterHeaderAction('view-settings');
});
</script>

<style scoped>
/* Add any custom styles here if needed */
/* Tailwind classes are recommended */
</style>
```
### Bước 4: Lưu và kiểm tra
1. Nhấp vào **"Tạo"** để lưu tiện ích mở rộng của bạn
2. Hệ thống tự động biên dịch mã Vue của bạn
3. Điều hướng đến đường dẫn menu tùy chỉnh của bạn: `/custom/analytics`
4. **Nội dung tiện ích mở rộng của bạn hiện hiển thị!**

### Bước 5: Quy trình hoàn chỉnh
**Điều gì xảy ra khi người dùng nhấp vào menu:**

1. **Người dùng nhấp chuột** "Trang tổng quan phân tích" trong menu
2. **Điều hướng trình duyệt** tới `/custom/analytics`
3. **Bộ định tuyến động của Enfyra** bắt được lộ trình
4. **Truy vấn hệ thống** cho menu có đường dẫn `/custom/analytics`  
5. **Tìm tiện ích mở rộng được liên kết** thông qua mối quan hệ một-một
6. **Tải và biên dịch** mã Vue SFC mở rộng
7. **Tiện ích kết xuất** có toàn quyền truy cập vào các thành phần và thành phần kết hợp
8. **Người dùng nhìn thấy** nội dung trang tổng quan phân tích tùy chỉnh

**Điểm cốt lõi**: Menu cung cấp điều hướng, tiện ích mở rộng cung cấp nội dung, và hai phần này tạo nên trải nghiệm người dùng hoàn chỉnh.

## Các loại tiện ích mở rộng

### Tiện ích mở rộng trang
- **Mục đích**: Ứng dụng toàn trang được liên kết với các mục menu
- **Cách sử dụng**: Được chọn thông qua bộ chọn quan hệ menu
- **Ví dụ**: Trang tổng quan, báo cáo, biểu mẫu tùy chỉnh

### Tiện ích mở rộng tiện ích  
- **Mục đích**: Các thành phần có thể tái sử dụng để nhúng vào mọi nơi
- **Cách sử dụng**: Có thể được nhúng bằng `<Widget :id="databaseId" />`
- **Ví dụ**: Biểu đồ, thẻ trạng thái, biểu mẫu nhỏ

### Tiện ích mở rộng toàn cầu
- **Mục đích**: Đăng ký shell trên toàn ứng dụng và hành vi thời gian thực nền
- **Cách sử dụng**: Tạo tiện ích mở rộng có kiểu `global`; Ứng dụng quản trị viên Enfyra gắn kết các tiện ích mở rộng toàn cầu một cách vô hình trong quá trình bố trí init
- **Ví dụ**: Chuông thông báo trong bảng tài khoản, bộ đếm chưa đọc trên toàn cầu, trình nghe ổ cắm quản trị viên, cầu làm mới nền

## Quyền truy cập SDK đầy đủ trong Tiện ích mở rộng

**Tích hợp SDK hoàn chỉnh**: Các tiện ích mở rộng chạy bên trong ứng dụng quản trị dựa trên Nuxt và có toàn quyền truy cập vào tất cả các tính năng SDK của Enfyra Nuxt từ `@enfyra/sdk-nuxt`. Mọi tính năng có thể kết hợp, tiện ích và API có sẵn trong ứng dụng chính cũng có sẵn trong tiện ích mở rộng của bạn – không có giới hạn.

## Danh sách đầy đủ các tài nguyên được đưa vào

Tiện ích mở rộng có quyền truy cập vào bộ tài nguyên toàn diện được tự động đưa vào khi chạy:

### Thành phần giao diện người dùng (Được chèn tự động)
Tất cả các thành phần UI được hệ thống tiện ích mở rộng tự động đưa vào và có thể được sử dụng trực tiếp trong các mẫu mà không cần nhập:

**Thành phần giao diện người dùng Nuxt** ([ Tài liệu giao diện người dùng Nuxt](https://ui.nuxt.com/)):
- `UIcon`, `Icon` - Thành phần biểu tượng và SVG
- `UButton` - Các nút có biến thể và trạng thái
- `UCard` - Thẻ chứa có đầu trang/chân trang
- `UBadge` - Huy hiệu và nhãn trạng thái
- `UInput` - Trường nhập văn bản
- `UTextarea` - Vùng văn bản nhiều dòng
- `USelect` - Lựa chọn thả xuống
- `UCheckbox` - Đầu vào hộp kiểm
- `USwitch` - Chuyển đổi công tắc
- `UModal` - Hộp thoại phương thức
- `UPopover` - Lớp phủ Popover
- `UTooltip` - Chú giải công cụ di chuột
- `UAlert` - Tin nhắn cảnh báo
- `UAvatar` - Hình đại diện của người dùng
- `UProgress` - Chỉ báo tiến độ
- `UTable` - Bảng dữ liệu
- `UPagination` - Điều hướng trang
- `UBreadcrumb` - Điều hướng đường dẫn
- `UTabs` - Giao diện tab
- `UAcordion` - Nội dung có thể thu gọn
- `UForm` - Thùng chứa biểu mẫu

**Thành phần Enfyra tùy chỉnh:**
- `DataTable` - Bảng dữ liệu nâng cao có tính năng lọc
- `PermissionGate` - Hiển thị nội dung dựa trên quyền
- `FormEditor` - Tạo biểu mẫu động
- `FilterDrawer` - Giao diện lọc nâng cao
- `LoadingState` - Chỉ báo trạng thái đang tải
- `EmptyState` - Hiển thị trạng thái trống
- `SettingsCard` - Thẻ giao diện cài đặt
- `Image` - Thành phần hình ảnh nâng cao
- `UploadModal` - Giao diện tải file lên
- `Widget` - Nhúng widget độngSử dụng `CommonModal` cho quy trình làm việc theo phương thức và `CommonDrawer` cho quy trình chỉnh sửa bảng điều khiển bên. Đối với chân trang tiêu chuẩn, hãy chuyển các nút hành động thông qua các đạo cụ thay vì các nút Hủy và Lưu theo kiểu thủ công trong khe chân trang:
```vue
<CommonModal
  v-model:open="open"
  :cancel-action="{ label: 'Cancel', onClick: () => (open = false) }"
  :primary-action="{ label: 'Save', loading: saving, disabled: !canSave, onClick: save }"
>
  <template #header>
    <h3 class="text-lg font-semibold eapp-text-primary">Edit settings</h3>
  </template>
  <template #body>
    <FormEditor v-model="form" table-name="settings" />
  </template>
</CommonModal>
```
Hủy các hành động mặc định theo kiểu dáng phác thảo trung tính. Sử dụng `angerAction` cho hành động phá hoại không thể đảo ngược, chẳng hạn như xóa, thu hồi hoặc xóa vĩnh viễn. Trong hộp thoại loại bỏ, hãy sử dụng `tone: "chính"` cho `Tiếp tục chỉnh sửa` vì nó giúp người dùng duy trì quy trình chỉnh sửa an toàn.

Sử dụng `USkeleton` hoặc các thành phần tải được chia sẻ để tải phần giữ chỗ. Enfyra ánh xạ các màu khung trên toàn cầu, vì vậy mã mở rộng không được mã hóa cứng các gradient tải hoặc bảng màu.

Sử dụng `UTabs` cho các phần trang thay vì thanh tab tùy chỉnh. Các tab kiểu Enfyra trên toàn cầu, vì vậy các tab tiện ích mở rộng của bạn sẽ tự động khớp với các trang hệ thống.
```vue
<template>
  <!-- Components are injected and can be used directly -->
  <UCard class="p-6 space-y-4">
    <!-- Form Elements -->
    <UInput v-model="name" placeholder="Enter name" />
    <UTextarea v-model="description" placeholder="Enter description" />
    <USelect v-model="selectedOption" :options="options" />
    <USwitch v-model="enabled" label="Enable feature" />

    <!-- Buttons and Actions -->
    <UButton type="button" @click="handleClick" color="primary">
      Click Me
    </UButton>

    <!-- Data Display -->
    <UTable :rows="data" :columns="columns" />
    <UBadge color="success">Status: Active</UBadge>

    <!-- Advanced Components -->
    <PermissionGate :condition="{ route: '/users', methods: ['GET'] }">
      <UButton variant="outline">Admin Only Button</UButton>
    </PermissionGate>
  </UCard>
</template>

<script setup>
// Components are automatically available in template - no need to import
// All composables and Vue functions are available globally - use directly

const name = ref('');
const description = ref('');
const enabled = ref(false);
const selectedOption = ref(null);

const options = ['Option 1', 'Option 2', 'Option 3'];
const data = [
  { id: 1, name: 'Item 1', status: 'Active' },
  { id: 2, name: 'Item 2', status: 'Inactive' }
];
const columns = [
  { key: 'id', label: 'ID' },
  { key: 'name', label: 'Name' },
  { key: 'status', label: 'Status' }
];

const handleClick = () => {
  console.log('Button clicked!');
};
</script>
```
### Thành phần kết hợp Enfyra (Truy cập toàn cầu)
**Tất cả các thành phần kết hợp Enfyra được tự động đưa vào và có sẵn trên toàn cầu:**

**API & Dữ liệu:**
- `useApi()` - Trình bao bọc API tùy chỉnh có xử lý lỗi (được khuyến nghị)
- `useSchema()` - Xác thực lược đồ và tạo biểu mẫu
- `useFilterQuery()` - Lọc và truy vấn nâng cao [Hướng dẫn hệ thống bộ lọc](./filter-system.md)
- `useDataTableColumns()` - Quản lý cột bảng dữ liệu

**Xác thực & Quyền:**
- `useAuth()` - Trạng thái và phương thức xác thực (`me`, `isLoggedIn`, `login`, `logout`)
- `usePermissions()` - Kiểm tra và xác thực quyền

**Quản lý giao diện người dùng và trạng thái:**
- `useHeaderActionRegistry()` - Đăng ký các hành động tiêu đề
- `useSubHeaderActionRegistry()` - Đăng ký các hành động tiêu đề phụ  
- `useAccountPanelRegistry()` - Đăng ký các hàng trong bảng tài khoản thanh bên
- `useMenuNotificationRegistry()` - Đăng ký số lượng hoặc dấu chấm thông báo trên menu bên
- `useScreen()` - Kích thước màn hình và các tiện ích đáp ứng
- `useGlobalState()` - Quản lý nhà nước toàn cầu
- `useConfirm()` - Hộp thoại xác nhận

### Thành phần kết hợp Nuxt (Truy cập toàn cầu)
**Tất cả các thành phần kết hợp Nuxt đều có sẵn mà không cần nhập:**

**Điều hướng & Định tuyến:**
- `useRoute()` - Thông tin tuyến đường hiện tại
- `useRouter()` - Phiên bản bộ định tuyến để điều hướng
- `navigateTo()` - Điều hướng theo chương trình

**Quản lý nhà nước:**
- `useState()` - Quản lý trạng thái Nuxt
- `useCookie()` - Quản lý cookie

**Tìm nạp dữ liệu:**
- `useFetch()` - Tìm nạp dữ liệu phía máy chủ
- `useAsyncData()` - Xử lý dữ liệu không đồng bộ
- `useLazyFetch()` - Tải dữ liệu lười biếng

**Meta & SEO:**
- `useHead()` - Quản lý phần đầu tài liệu
- `useSeoMeta()` - Siêu dữ liệu SEO

**Ngữ cảnh ứng dụng:**
- `useNuxtApp()` - Phiên bản ứng dụng Nuxt
- `useToast()` - Thông báo bánh mì nướng

### Composition API của Vue 3 (Truy cập toàn cầu)
**Composition API Vue 3 hoàn chỉnh có sẵn trên toàn cầu:**

**Khả năng phản ứng cốt lõi:**
- `ref()` - Tạo tài liệu tham khảo phản ứng
- `reactive()` - Tạo các đối tượng phản ứng
- `computed()` - Thuộc tính được tính toán
- `readonly()` - Dữ liệu phản ứng chỉ đọc
- `shallowRef()` - Tham chiếu phản ứng nông
- `shallowReactive()` - Đối tượng phản ứng nông

**Móc vòng đời:**
- `onMounted()` - Thành phần được gắn kết
- `onUnmount()` - Thành phần chưa được gắn kết
- `onBeforeMount()` - Trước khi gắn kết thành phần
- `onBeforeUnmount()` - Trước khi ngắt kết nối thành phần
- `onUpdated()` - Đã cập nhật thành phần
- `onBeforeUpdate()` - Trước khi cập nhật thành phần

**Người theo dõi:**
- `watch()` - Xem dữ liệu phản ứng
- `watchEffect()` - Xem dựa trên hiệu ứng

**Thành phần thành phần:**
- `defineProps()` - Xác định đạo cụ thành phần
- `defineEmits()` - Xác định các sự kiện thành phần
- `defineExpose()` - Hiển thị các phương thức thành phần
- `defineComponent()` - Xác định thành phần Vue
- `h()` - Trình trợ giúp chức năng kết xuất
- `resolveComponent()` - Giải quyết thành phần theo tên

**Tiện ích:**
- `nextTick()` - Đợi cập nhật DOM
- `toRef()` - Chuyển đổi sang ref
- `toRefs()` - Chuyển đổi thành ref
- `unref()` - Mở gói giá trị ref
- `isRef()` - Kiểm tra xem giá trị có phải là ref không
- `markRaw()` - Đánh dấu là không phản ứng
- `toRaw()` - Lấy đối tượng thô
- `isProxy()`, `isReactive()`, `isReadonly()` - Kiểm tra kiểu
- `effectScope()` - Quản lý phạm vi hiệu ứng
- `getCurrentScope()` - Lấy phạm vi hiện tại
- `onScopeDispose()` - Dọn dẹp phạm vi

### API trình duyệt (Có sẵn)
**API trình duyệt tiêu chuẩn có thể truy cập được:**
- `tìm nạp()` - Yêu cầu HTTP
- `console` - Ghi nhật ký bảng điều khiển
- `window` - Đối tượng cửa sổ
- `tài liệu` - Thao tác DOM

**Để biết ví dụ đầy đủ về cách sử dụng API, hãy xem [Hướng dẫn tích hợp API](./api-integration.md)****Ví dụ về sử dụng tài nguyên được chèn cơ bản:**
```vue
<script setup>
// All functions and composables are available globally - just use them directly!

// API Access
const { data } = await useApi('/enfyra_extension', {
  query: { limit: 10 }
});

// Authentication
const { me, isLoggedIn, login, logout } = useAuth();

// Permissions
const { hasPermission } = usePermissions();
if (hasPermission('/users', 'POST')) {
  // User can create
}

// Notifications
const toast = useToast();
toast.add({
  title: 'Success!',
  color: 'success'
});

// Navigation
const router = useRouter();
const route = useRoute();

// Schema & Validation
const { validate, generateEmptyForm } = useSchema('enfyra_extension');

// Vue 3 Composition API - use directly from global
const loading = ref(false);
const state = reactive({ count: 0 });
const doubled = computed(() => state.count * 2);

// State management
const globalState = useState('myExtension', () => ({}));
</script>
```
### Cổng cấp phép
Kiểm soát khả năng hiển thị dựa trên quyền:
```vue
<template>
  <PermissionGate :condition="{ 
    route: '/users', 
    methods: ['POST'] 
  }">
    <UButton>Admin Only Button</UButton>
  </PermissionGate>
</template>
```
## Tính năng mở rộng nâng cao

## Tích hợp hành động tiêu đề

** Tiện ích mở rộng có thể đưa các hành động tùy chỉnh trực tiếp vào khu vực tiêu đề và tiêu đề phụ của ứng dụng** - thể hiện sức mạnh đáng kinh ngạc để can thiệp vào BẤT KỲ phần nào của giao diện ứng dụng.

### Ví dụ về hành động tiêu đề nhanh
```vue
<script setup>
const { register: registerHeaderActions } = useHeaderActionRegistry();
const { register: registerSubHeaderActions } = useSubHeaderActionRegistry();
onMounted(() => {
  // Register in main header (top-right)
  registerHeaderActions({
    id: 'save-report',
    label: 'Save Report',
    color: 'primary',
    onClick: () => saveReport(),
    permission: {
      route: '/reports',
      methods: ['POST']
    }
  });
  
  // Register in sub-header (page level)
  registerSubHeaderActions({
    id: 'filter-toggle',
    label: 'Filters',
    side: 'left',
    onClick: () => toggleFilters()
  });
});
</script>
```
### Tính năng mạnh mẽ
- **Tích hợp quyền**: Mọi hành động đều tự động sử dụng PermissionGate
- **Nhận thức về lộ trình**: Hiển thị/ẩn các hành động dựa trên trang hiện tại
- **Thành phần tùy chỉnh**: Đưa vào các tiện ích tùy chỉnh hoàn chỉnh
- **Thuộc tính phản ứng**: Nhãn động, trạng thái tải, khả năng hiển thị có điều kiện
- **Điều khiển định vị**: Định vị trái/phải trong tiêu đề phụ

** [Hướng dẫn thao tác tiêu đề hoàn chỉnh](./header-actions.md)** - Tài liệu đầy đủ với các ví dụ nâng cao

### Tìm nạp dữ liệu từ API
```vue
<script setup>
// Using custom API wrapper (recommended)
const { data: usersData, pending, error, refresh } = await useApi('/enfyra_user', {
  query: {
    limit: 10,
    fields: 'id,email,name,role.name',
    include: 'role'
  },
  key: 'users-list'
});

// Or use the standard useApi() composable for any custom call
const { data: directData, execute: loadDirect } = useApi('/enfyra_user', {
  query: { limit: 10 },
});
await loadDirect();

// Computed for easy access
const users = computed(() => usersData.value?.data || []);

// Manual refresh
const loadUsers = () => {
  refresh();
};

// Handle API responses
watch(error, (newError) => {
  if (newError) {
    toast.add({
      title: 'Error loading users',
      description: newError.message,
      color: 'error'
    });
  }
});

onMounted(() => {
  console.log('Users loaded:', users.value.length);
});
</script>

<template>
  <div>
    <!-- Loading state -->
    <div v-if="pending">Loading users...</div>
    
    <!-- Error state -->
    <UAlert v-else-if="error" color="red">
      {{ error.message }}
    </UAlert>
    
    <!-- Data display -->
    <div v-else>
      <div v-for="user in users" :key="user.id">
        {{ user.name }} - {{ user.role?.name }}
      </div>
      
      <UButton @click="loadUsers">Refresh</UButton>
    </div>
  </div>
</template>
```
**Xem [Tích hợp API](./api-integration.md) để biết hướng dẫn sử dụng API đầy đủ.**

### Tạo biểu mẫu

Các tiện ích mở rộng có thể sử dụng hệ thống biểu mẫu mạnh mẽ của Enfyra để tạo các biểu mẫu động, được xác thực:

** [Hướng dẫn hệ thống biểu mẫu hoàn chỉnh](./form-system.md)** - Tìm hiểu về các biểu mẫu động, xác thực và loại trường
```vue
<template>
  <UCard>
    <FormEditor
      :schema="schema"
      v-model="formData"
      @submit="handleSubmit"
    />
  </UCard>
</template>

<script setup>
const schema = await useSchema('products');
const formData = ref(schema.generateEmptyForm());

const handleSubmit = async () => {
  const { isValid, errors } = schema.validate(formData.value);
  
  if (!isValid) {
    toast.add({
      title: 'Validation failed',
      color: 'red'
    });
    return;
  }
  
  await useApi('/products', {
    method: 'POST',
    body: formData.value
  });
};
</script>
```
## Hệ thống tiện ích

### Sử dụng tiện ích mở rộng tiện ích
Widget là các thành phần có thể tái sử dụng và có thể được nhúng ở mọi nơi:
```vue
<template>
  <div class="grid grid-cols-2 gap-4">
    <!-- Embed widget by database ID (not extensionId) -->
    <Widget :id="5" />
    
    <!-- Another widget -->
    <Widget :id="6" />
  </div>
</template>
```
**Quan trọng**: Widget `id` là ID cơ sở dữ liệu số từ danh sách Tiện ích mở rộng, không phải chuỗi `extensionId`.

### Tạo tiện ích mở rộng widget

1. **Tạo tiện ích mở rộng** với loại "Widget":
   - **Tên**: Tên hiển thị của widget
   - **Loại**: Chọn "Widget" 
   - **Mô tả**: Tiện ích này làm gì
   - **Không cần liên kết menu** (các widget được nhúng, không được điều hướng đến)

2. **Viết mã tiện ích**:
```vue
<template>
  <UCard>
    <template #header>
      <h3>Sales Summary</h3>
    </template>
    
    <div class="text-2xl font-bold">
      ${{ totalSales.toLocaleString() }}
    </div>
    <p class="text-gray-500">This month</p>
  </UCard>
</template>

<script setup>
// All functions are available globally - use directly
const totalSales = ref(0);

// Load data using custom wrapper
onMounted(async () => {
  const { data, error } = await useApi('/sales_summary');
  if (!error.value && data.value) {
    totalSales.value = data.value.total;
  }
});
</script>
```
3. **Nhúng Widget**: Sử dụng `<Widget :id="database_id" />` trong bất kỳ tiện ích mở rộng hoặc trang nào

## Hệ thống mở rộng toàn cầu

Tiện ích mở rộng toàn cục là các bản ghi Vue SFC có `type="global"`. Ứng dụng quản trị Enfyra tìm nạp mọi tiện ích mở rộng toàn cầu được bật trong quá trình khởi tạo bố cục, giải quyết nó thông qua trình tải tiện ích mở rộng động thông thường và gắn kết nó một cách vô hình ở cấp shell. Sử dụng điều này cho logic phải tồn tại trên mọi trang.

### Khi nào nên sử dụng Tiện ích mở rộng toàn cầu

Sử dụng tiện ích mở rộng toàn cầu cho:
- Các mục trong bảng tài khoản như chuông thông báo
- Bộ đếm hoặc chỉ báo trạng thái chưa đọc toàn cầu
- Quản trị viên chia sẻ Trình nghe Socket.IO
- Cầu làm mới nền cập nhật trạng thái trên toàn ứng dụng
- Các mục đăng ký cấp Shell sẽ tồn tại khi thay đổi tuyến đường

Không sử dụng tiện ích mở rộng chung cho:
- Nội dung toàn trang
- Giao diện người dùng dành riêng cho tuyến đường
- Bảng điều khiển lớn hoặc trang hoạt động
- Các widget chỉ nên được nhúng trong một trang
- Thẻ nổi hoặc lớp phủ tùy chỉnh sao chép giao diện người dùng shell

### Tạo tiện ích mở rộng toàn cầu

Tạo bản ghi `enfyra_extension` với:
- **Loại**: `toàn cầu`
- **Menu**: trống
- **Mẫu**: trống hoặc ẩn
- **Script**: đăng ký và thiết lập thời gian thực

Các tiện ích mở rộng toàn cầu nên đăng ký giao diện người dùng hiển thị vào các cơ quan đăng ký shell hiện có. Họ không nên hiển thị trực tiếp giao diện người dùng nội dung trang.
```vue
<template></template>

<script setup>
const unread = ref(3)
const iconName = (name) => ['lucide', name].join(':')
const activeBellIcon = iconName('bell-ring')
const idleBellIcon = iconName('bell')
const chevronIcon = iconName('chevron-right')

const NotificationPanelItem = defineComponent({
  name: 'NotificationPanelItem',
  setup() {
    const openNotifications = () => navigateTo('/notifications')

    return () => h('button', {
      type: 'button',
      class: 'flex w-full items-center gap-3 rounded-lg px-3 py-2.5 text-left transition hover:bg-muted',
      onClick: openNotifications,
    }, [
      h('span', {
        class: 'flex h-9 w-9 shrink-0 items-center justify-center rounded-md bg-primary/10 text-primary',
      }, [
        h(UIcon, {
          name: unread.value > 0 ? activeBellIcon : idleBellIcon,
          class: 'h-5 w-5',
        }),
      ]),
      h('span', { class: 'min-w-0 flex-1' }, [
        h('span', { class: 'block truncate text-sm font-medium text-highlighted' }, 'Notifications'),
        h('span', { class: 'mt-0.5 block truncate text-xs text-muted' }, unread.value > 0 ? 'Needs review' : 'All caught up'),
      ]),
      unread.value > 0
        ? h(UBadge, { color: 'primary', variant: 'soft', size: 'sm' }, () => String(unread.value))
        : h(UIcon, { name: chevronIcon, class: 'h-4 w-4 text-muted' }),
    ])
  },
})

const notificationCount = computed(() => unread.value > 0 ? String(unread.value) : null)

const { register } = useAccountPanelRegistry()
register({
  id: 'notifications',
  order: 20,
  label: 'Notifications',

  description: computed(() => unread.value > 0 ? 'Needs review' : 'All caught up'),
  count: notificationCount,
  badgeColor: 'error',
  component: NotificationPanelItem,
})

const { register: registerMenuNotification, unregister: unregisterMenuNotification } = useMenuNotificationRegistry()
watchEffect(() => {
  registerMenuNotification({
    id: 'notifications-menu',
    target: { path: '/notifications' },
    value: notificationCount.value,
    color: unread.value > 0 ? 'error' : 'neutral',
    title: unread.value > 0 ? 'Unread notifications' : 'No unread notifications',
  })
})

const { adminSocket } = useAdminSocket()
const handleSummary = (payload) => {
  if (payload?.unread != null) unread.value = payload.unread
}

adminSocket.on('notification:summary', handleSummary)
onUnmounted(() => {
  adminSocket.off('notification:summary', handleSummary)
  unregisterMenuNotification('notifications-menu')
})
</script>
```
### Quy tắc giao diện người dùng mở rộng toàn cầu

- Giữ các mục trong bảng tài khoản dưới dạng một hàng nhỏ gọn: biểu tượng, nhãn, văn bản phụ ngắn, huy hiệu ở cuối hoặc chữ V.
- Sử dụng `count` cho các giá trị thông báo của bảng tài khoản. `huy hiệu` vẫn được hỗ trợ dưới dạng bí danh, nhưng trình kích hoạt tài khoản tổng hợp các giá trị `count`/`huy hiệu` hiển thị bằng số và giới hạn trình kích hoạt ở `99+`.
- Sử dụng `useMenuNotificationRegistry()` cho trạng thái thông báo menu thanh bên. Đăng ký một `id` ổn định, nhắm mục tiêu theo menu `id`, `path` hoặc `route`, chuyển `giá trị` cho số lượng hiển thị, bỏ qua `giá trị` cho dấu chấm và sử dụng `chính`, `thành công`, `cảnh báo`, `lỗi`, `thông tin` hoặc `trung tính` cho `màu`.
- Sử dụng các mã thông báo/lớp tương thích với shell như `bg-muted`, `text-muted` và `text-highlighted`.
- Sử dụng bán kính `rounded-lg` hoặc nhỏ hơn và khoảng đệm vừa phải để hàng khớp với bảng điều khiển thanh bên.
- Tạo toàn bộ hàng thành một `button` bằng `type="button"`.
- Không lồng các nút vào trong các hàng của bảng tài khoản.
- Không hiển thị thẻ tỷ lệ trang, vỏ phương thức hoặc giao diện người dùng kiểu anh hùng từ tiện ích mở rộng toàn cầu.
- Sử dụng id đăng ký ổn định để tải lại thay thế cùng một mục shell có thể dự đoán được.
- Dọn dẹp socket và trình nghe DOM trong `onUnmount`; Ứng dụng quản trị Enfyra ngắt kết nối các thành phần chung cũ khi tiện ích mở rộng tải lại hoặc bị tắt.

## Hỗ trợ tải lên tệp
Tiện ích mở rộng có thể xử lý tải lên tệp:
```vue
<template>
  <input 
    type="file" 
    @change="handleFileUpload"
    accept=".vue"
  />
</template>

<script setup>
const handleFileUpload = async (event) => {
  const file = event.target.files[0];
  if (!file) return;
  
  const content = await file.text();
  // Process the file content
  console.log('File content:', content);
};
</script>
```
## Sử dụng gói NPM trong tiện ích mở rộng

Tiện ích mở rộng có thể sử dụng gói npm để thêm chức năng mạnh mẽ như biểu đồ, tiện ích và xử lý dữ liệu.

### Bắt đầu nhanh

**1. Gói cài đặt**
- Đi tới **Gói** trong thanh bên
- Bấm vào **Cài đặt gói**
- Chọn loại **Gói ứng dụng**
- Tìm kiếm và cài đặt gói của bạn

**2. Sử dụng trong phần mở rộng**
```vue
<script setup>
onMounted(async () => {
  const { dayjs, lodash } = await getPackages();

  const date = dayjs().format('YYYY-MM-DD');
  const total = lodash.sum([1, 2, 3]);

  console.log('Date:', date, 'Total:', total);
});
</script>
```
### Mẫu sử dụng

** Phá hủy (Khuyến nghị) **
```javascript
const { chartjs, dayjs } = await getPackages();
```
**Mảng gói**
```javascript
const packages = await getPackages(['chartjs', 'dayjs']);
```
**Tất cả các gói**
```javascript
const allPackages = await getPackages();
```
### Ví dụ hoàn chỉnh: Tiện ích ngày
```vue
<template>
  <UCard>
    <template #header>
      <h3>Date Utilities</h3>
    </template>

    <div class="space-y-4">
      <div>
        <UButton @click="formatDate">Format Current Date</UButton>
      </div>

      <div v-if="formattedDate">
        <UBadge>{{ formattedDate }}</UBadge>
      </div>

      <div>
        <UButton @click="addDays" variant="outline">Add 7 Days</UButton>
      </div>

      <div v-if="futureDate">
        <UBadge color="success">{{ futureDate }}</UBadge>
      </div>
    </div>
  </UCard>
</template>

<script setup>
const formattedDate = ref(null);
const futureDate = ref(null);

const formatDate = async () => {
  const { dayjs } = await getPackages();
  formattedDate.value = dayjs().format('YYYY-MM-DD HH:mm:ss');
};

const addDays = async () => {
  const { dayjs } = await getPackages();
  futureDate.value = dayjs().add(7, 'day').format('YYYY-MM-DD');
};
</script>
```
### Ví dụ hoàn chỉnh: Biểu đồ có dữ liệu
```vue
<template>
  <UCard>
    <template #header>
      <div class="flex items-center justify-between">
        <h3>Revenue Chart</h3>
        <UButton @click="loadData" :loading="loading" size="sm">
          Refresh
        </UButton>
      </div>
    </template>

    <canvas ref="chartCanvas"></canvas>
  </UCard>
</template>

<script setup>
const chartCanvas = ref(null);
const loading = ref(false);
let chartInstance = null;

const loadData = async () => {
  loading.value = true;

  const { Chart } = await getPackages(['chart.js']);
  const ctx = chartCanvas.value.getContext('2d');

  if (chartInstance) {
    chartInstance.destroy();
  }

  chartInstance = new Chart(ctx, {
    type: 'line',
    data: {
      labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
      datasets: [{
        label: 'Revenue',
        data: [12000, 19000, 15000, 25000, 22000, 30000],
        borderColor: 'rgb(59, 130, 246)',
        backgroundColor: 'rgba(59, 130, 246, 0.1)',
        tension: 0.4
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: true
        }
      },
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });

  loading.value = false;
};

onMounted(() => {
  loadData();
});
</script>
```
### Các gói có sẵn

Bất kỳ gói npm nào cũng có thể được cài đặt. Lựa chọn phổ biến:

**Biểu đồ & Trực quan**
- `chart.js` - Biểu đồ và đồ thị
- `echarts` - Biểu đồ doanh nghiệp
- `apexcharts` - Biểu đồ hiện đại

**Tiện ích**
- `dayjs` - Thao tác ngày tháng
- `lodash` - Các hàm tiện ích
- `axios` - Yêu cầu HTTP

**Dữ liệu & Biểu mẫu**
- `vuedraggable` - Kéo và thả
- `sortablejs` - Danh sách có thể sắp xếp
- `papaparse` - Phân tích cú pháp CSV

**Để biết tài liệu đầy đủ, hãy xem [Quản lý gói](./hooks-handlers/package-management.md#using-app-packages-in-extensions)**

## Quản lý tiện ích mở rộng

### Bật/Tắt tiện ích mở rộng
1. Đi tới **Cài đặt > Tiện ích mở rộng**
2. Tìm tiện ích mở rộng của bạn trong danh sách
3. Chuyển đổi công tắc để bật/tắt
4. Tiện ích mở rộng bị vô hiệu hóa sẽ không tải ngay cả khi nhấp vào menu

### Chỉnh sửa tiện ích mở rộng
1. Nhấp vào bất kỳ thẻ mở rộng nào để mở trình chỉnh sửa
2. Sửa đổi mã trong trình chỉnh sửa
3. Nhấp vào **"Lưu"** để biên dịch lại
4. Thay đổi có hiệu lực ngay lập tức

### Kiểm soát phiên bản
- Cập nhật số phiên bản khi thực hiện thay đổi
- Hệ thống theo dõi dấu thời gian tạo và sửa đổi
- Thông tin người dùng được lưu trữ để theo dõi kiểm tra

## Các phương pháp hay nhất

### Tổ chức mã
```vue
<template>
  <!-- Keep template clean and organized -->
  <div class="extension-container">
    <ExtensionHeader />
    <ExtensionContent />
    <ExtensionFooter />
  </div>
</template>

<script setup>
// 1. Imports and composables
// All composables are available globally - just call them directly
const toast = useToast();

// 2. Reactive state
const state = reactive({
  loading: false,
  data: []
});

// 3. Computed properties
const filteredData = computed(() => {
  return state.data.filter(item => item.active);
});

// 4. Methods
const loadData = async () => {
  // Implementation
};

// 5. Lifecycle hooks
onMounted(() => {
  loadData();
});
</script>

<style scoped>
.extension-container {
  @apply p-6;
}
</style>
```
### Xử lý lỗi
```vue
<script setup>
const loadData = async () => {
  // useApi handles errors internally - no try-catch needed
  const { data, error } = await useApi('/endpoint');

  if (error.value) {
    console.error('Extension error:', error.value);
    toast.add({
      title: 'Error',
      description: error.value.message,
      color: 'red'
    });
    return;
  }

  // Handle success with data.value
  console.log('Data loaded:', data.value);
};
</script>
```
### Mẹo về hiệu suất
- Sử dụng `được tính` cho các giá trị dẫn xuất
- Triển khai phân trang cho tập dữ liệu lớn
- Các thành phần nặng tải lười biếng
- Dọn dẹp tài nguyên trong `onUnmount`

## Các vấn đề thường gặp và giải pháp

### Tiện ích mở rộng không tải
**Vấn đề**: Nhấp vào menu sẽ hiển thị trang trống
**Giải pháp**: 
1. Kiểm tra tiện ích mở rộng đã được bật chưa
2. Xác minh tiện ích mở rộng được liên kết với menu chính xác
3. Kiểm tra bảng điều khiển trình duyệt để tìm lỗi biên dịch
4. Đảm bảo người dùng có quyền truy cập

### Lỗi biên dịch
**Sự cố**: Tiện ích mở rộng không biên dịch được
**Giải pháp**:
1. Kiểm tra cú pháp Vue.js có đúng không
2. Đảm bảo tất cả các thành phần nhập khẩu đều tồn tại
3. Xác minh cú pháp thiết lập tập lệnh
4. Tìm kiếm thông báo lỗi trong biểu mẫu

### Thiếu thành phần
**Sự cố**: Thành phần không được nhận dạng
**Giải pháp**:
- Các thành phần được chèn tự động và có sẵn trực tiếp trong mẫu
- Không cần nhập hoặc truy cập thông qua đạo cụ
- Chỉ cần sử dụng trực tiếp: `<UButton>`, `<UCard>`, v.v.

### Cuộc gọi API không thành công
**Sự cố**: Không thể tìm nạp dữ liệu
**Giải pháp**:
1. Kiểm tra xem người dùng có quyền cần thiết không
2. Xác minh điểm cuối API tồn tại
3. Kiểm tra tab mạng xem có lỗi không
4. Đảm bảo xác thực phù hợp

## Mẫu nâng cao

### Quản lý nhà nước
```vue
<script setup>
// Use useState for cross-component state
const globalState = useState('myExtension', () => ({
  counter: 0,
  items: []
}));

// Update state
globalState.value.counter++;
</script>
```
### Thành phần thành phần
```vue
<script setup>
// Break large extensions into smaller components
const components = {
  Header: {
    template: '<div>Header</div>'
  },
  Footer: {
    template: '<div>Footer</div>'
  }
};
</script>

<template>
  <component :is="components.Header" />
  <component :is="components.Footer" />
</template>
```
### Đang tải động
```vue
<script setup>
const dynamicComponent = ref(null);

const loadComponent = async () => {
  // Load component based on conditions
  if (someCondition) {
    dynamicComponent.value = await loadExtension('widget-1');
  }
};
</script>

<template>
  <component :is="dynamicComponent" v-if="dynamicComponent" />
</template>
```
## Cân nhắc về bảo mật

### Kiểm tra quyền
Luôn xác minh quyền trong tiện ích mở rộng của bạn:
```vue
<script setup>
const { hasPermission } = usePermissions();

// Check before showing sensitive data
const canViewFinancials = computed(() => {
  return hasPermission('/financial_reports', 'GET');
});

// Check before allowing actions
const deleteRecord = async (id) => {
  if (!hasPermission('/records', 'DELETE')) {
    toast.add({
      title: 'Permission denied',
      color: 'red'
    });
    return;
  }
  
  await useApi(`/records/${id}`, { method: 'DELETE' });
};
</script>
```
### Xác thực đầu vào
Xác thực dữ liệu đầu vào của người dùng trước khi gửi tới API:
```vue
<script setup>
const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};

const submitForm = async () => {
  if (!validateEmail(formData.value.email)) {
    toast.add({
      title: 'Invalid email address',
      color: 'red'
    });
    return;
  }
  
  // Proceed with submission
};
</script>
```
## Tóm tắt

Hệ thống mở rộng cung cấp một cách mạnh mẽ để thêm chức năng tùy chỉnh vào Enfyra:
1. **Tạo Menu** Xác định mục điều hướng
2. **Tạo tiện ích mở rộng** Cung cấp nội dung trang  
3. **Liên kết với nhau** Menu và tiện ích mở rộng hoạt động như một
4. **Viết mã Vue** Hỗ trợ SFC Vue 3 đầy đủ với các thành phần được chèn tự động
5. **Quyền truy cập SDK đầy đủ** Quyền truy cập đầy đủ vào tất cả các tính năng và thành phần kết hợp của Enfyra SDK
6. **Truy cập tài nguyên** Các thành phần giao diện người dùng, API, quyền, hàm Vue
7. **Triển khai ngay lập tức** Không cần quá trình xây dựng

Các tiện ích mở rộng cho phép bạn linh hoạt tạo bất kỳ chức năng tùy chỉnh nào trong khi vẫn duy trì tính bảo mật và tính nhất quán của nền tảng Enfyra. Với sự tích hợp SDK đầy đủ, các tiện ích mở rộng có sức mạnh tương đương với ứng dụng cốt lõi.

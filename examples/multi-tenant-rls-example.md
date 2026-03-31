# Row-Level Security Example

> **Real-World Use Case:** Build a multi-tenant application where different teams share the same database tables but can only see and manage their own data.

**What You'll Build:**
- 1 shared `user_definition` table for all teams
- 2 teams (Team A, Team B) with complete data isolation
- Custom dashboards for each team
- Automatic RLS enforcement via Pre-Hooks
- Extension-based custom pages with RLS

**Time Required:** 30-45 minutes

---

## Table of Contents

- [Scenario Overview](#scenario-overview)
- [Phase 1: Database Setup](#phase-1-database-setup)
- [Phase 2: Pre-Hook RLS Enforcement](#phase-2-pre-hook-rls-enforcement)
- [Phase 3: Hide Default Menus](#phase-3-hide-default-menus)
- [Phase 4: Create Custom Dashboards](#phase-4-create-custom-dashboards)
- [Phase 5: Menu + Permission Setup](#phase-5-menu--permission-setup)
- [Phase 6: Testing](#phase-6-testing)
- [Troubleshooting](#troubleshooting)

---

## Scenario Overview

### The Problem

You have 2 websites with 2 different teams:

```
Website A (Team A):
- 10 users managing products
- Need their own dashboard
- Cannot see Team B's data

Website B (Team B):
- 8 users managing services
- Need their own dashboard
- Cannot see Team A's data
```

### The Solution

```
Shared Database:
├── user_definition (all users in 1 table)
│   ├── id
│   ├── email
│   ├── name
│   ├── role
│   └── project_id  ← RLS field: 'A' or 'B'
│
Enforcement:
├── Pre-Hooks: Auto-inject project_id filter
├── Extensions: Custom dashboards per team
└── Permissions: Role-based access control
```

---

## Phase 1: Database Setup

### Step 1.1: Add `project_id` Field to user_definition

1. Navigate to **Collections**  Click `user_definition`

2. In the **Columns** section, click **Add Column**

3. Configure the new column:
   - **Name:** `project_id`
   - **Type:** `varchar`
   - **Length:** `10`
   - **Nullable:** `No` (required field)
   - **Updatable:** `Yes`
   - **Default Value:** (leave empty - will be auto-assigned)

4. Click **Save** to update the table structure

### Step 1.2: Assign Existing Users to Projects

1. Navigate to **Data**  `user_definition`

2. For each existing user, edit and set:
   - Team A users: `project_id = "A"`
   - Team B users: `project_id = "B"`

3. Save each user

### Step 1.3: Create Sample Data Tables (Optional)

If you want to test with more than just users, create team-specific tables:

```
Table: products_a (Team A's products)
├── id
├── name
├── price
├── stock
└── project_id (always "A")

Table: products_b (Team B's products)
├── id
├── name
├── price
├── stock
└── project_id (always "B")
```

**Or use shared table with RLS:**

```
Table: products (shared)
├── id
├── name
├── price
├── stock
└── project_id ("A" or "B") ← RLS field
```

---

## Phase 2: Pre-Hook RLS Enforcement

### Step 2.1: Create Global Pre-Hook for user_definition

This Pre-Hook automatically injects `project_id` filter for ALL requests.

1. Navigate to **Settings**  **Routings**

2. Find and click on route: `/user_definition`

3. Scroll to **Hooks Management** section

4. Click **Add Pre-Hook**

5. Configure the hook:
   - **Name:** `RLS - Filter by Project`
   - **Description:** `Auto-inject project_id filter based on logged-in user`
   - **Priority:** `100` (high priority, runs first)
   - **Is Global:** `No` (specific to this route)
   - **Is Enabled:** `Yes`

6. Select **HTTP Methods:** Check ALL (GET, POST, PUT, PATCH, DELETE)

7. In the **Logic** editor, paste this code:

```javascript
// ============================================
// RLS Pre-Hook: Auto-inject project_id filter
// ============================================

// Get current user's project_id from JWT token
const userProject = @USER.project_id;
const isRootAdmin = @USER.isRootAdmin;

// Root admins can see all data - skip RLS
if (isRootAdmin) {
  return;
}

// Require project_id for non-admin users
if (!userProject) {
  throw new Error('User must have project_id assigned');
}

// Apply RLS based on HTTP method
const method = @API.method;

switch (method) {
  // === READ OPERATIONS ===
  case 'GET':
    // Inject filter: only show records from user's project
    if (@QUERY.filter) {
      @QUERY.filter = {
        _and: [
          { project_id: { _eq: userProject } },
          @QUERY.filter
        ]
      };
    } else {
      @QUERY.filter = { project_id: { _eq: userProject } };
    }
    break;

  // === CREATE OPERATIONS ===
  case 'POST':
    // Auto-assign project_id (prevent user from choosing)
    @BODY.project_id = userProject;

    // If user tried to set different project_id, reject
    if (@BODY.project_id && @BODY.project_id !== userProject) {
      throw new Error('Cannot create records for another project');
    }
    break;

  // === UPDATE OPERATIONS ===
  case 'PUT':
  case 'PATCH':
    // Prevent changing project_id
    delete @BODY.project_id;

    // Only allow updating records in user's project
    if (@QUERY.filter) {
      @QUERY.filter = {
        _and: [
          { project_id: { _eq: userProject } },
          @QUERY.filter
        ]
      };
    } else {
      // If no filter provided, add project_id to params
      @PARAMS.project_id = userProject;
    }
    break;

  // === DELETE OPERATIONS ===
  case 'DELETE':
    // Only allow deleting records in user's project
    if (@QUERY.filter) {
      @QUERY.filter = {
        _and: [
          { project_id: { _eq: userProject } },
          @QUERY.filter
        ]
      };
    } else {
      @PARAMS.project_id = userProject;
    }
    break;
}

// Log RLS enforcement (optional, for audit)
@LOGS(`RLS applied: project_id=${userProject}, method=${method}`);
```

8. Click **Save** to create the Pre-Hook

### Step 2.2: Create Pre-Hook for Other Tables (If Using Shared Tables)

If you have shared tables like `products`, `orders`, etc., repeat Step 2.1 for each:

```javascript
// Simplified version for any table with project_id
const userProject = @USER.project_id;
const isRootAdmin = @USER.isRootAdmin;

if (isRootAdmin) return;

if (!userProject) {
  throw new Error('User project_id not set');
}

const method = @API.method;

if (method === 'GET') {
  @QUERY.filter = {
    _and: [
      { project_id: { _eq: userProject } },
      @QUERY.filter || {}
    ]
  };
} else if (method === 'POST') {
  @BODY.project_id = userProject;
  delete @BODY.project_id; // Prevent manual override
} else if (method === 'PUT' || method === 'PATCH') {
  delete @BODY.project_id;
}
```

---

## Phase 3: Hide Default Menus

### Step 3.1: Hide Default user_definition Menu

1. Navigate to **Settings**  **Menu**

2. Find the default menu item for `user_definition` (usually under **Data**)

3. Click to edit the menu item

4. Set **Is Enabled:** to `No` (or delete the menu item)

5. Click **Save**

**Result:** Users won't see the default table view anymore.

---

## Phase 4: Create Custom Dashboards

### Step 4.1: Create Team A Dashboard Extension

1. Navigate to **Settings**  **Extensions**

2. Click **Create Extension**

3. Configure the extension:
   - **Name:** `Dashboard A`
   - **Extension ID:** `dashboard-a` (auto-generated, keep as is)
   - **Type:** `Page`
   - **Description:** `Team A dashboard with user management and stats`
   - **Version:** `1.0.0`
   - **Is Enabled:** `Yes`

4. In the **Code Editor**, paste this complete Vue component:

```vue
<template>
  <div class="p-6 space-y-6">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-bold text-blue-600">Team A Dashboard</h1>
        <p class="text-gray-500 dark:text-gray-400">
          Manage your team and users
        </p>
      </div>
      <UBadge color="blue" variant="soft">
        
        Project A Only
      </UBadge>
    </div>

    <!-- Stats Cards -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <UCard>
        <div class="flex items-center gap-4 p-4">
          <div class="w-12 h-12 rounded-lg bg-blue-100 dark:bg-blue-900/30 flex items-center justify-center">
            
          </div>
          <div>
            <div class="text-2xl font-bold">{{ stats.totalUsers }}</div>
            <div class="text-sm text-gray-500">Total Users</div>
          </div>
        </div>
      </UCard>

      <UCard>
        <div class="flex items-center gap-4 p-4">
          <div class="w-12 h-12 rounded-lg bg-green-100 dark:bg-green-900/30 flex items-center justify-center">
            
          </div>
          <div>
            <div class="text-2xl font-bold">{{ stats.newThisWeek }}</div>
            <div class="text-sm text-gray-500">New This Week</div>
          </div>
        </div>
      </UCard>

      <UCard>
        <div class="flex items-center gap-4 p-4">
          <div class="w-12 h-12 rounded-lg bg-purple-100 dark:bg-purple-900/30 flex items-center justify-center">
            
          </div>
          <div>
            <div class="text-2xl font-bold">{{ stats.activeToday }}</div>
            <div class="text-sm text-gray-500">Active Today</div>
          </div>
        </div>
      </UCard>
    </div>

    <!-- User Table -->
    <UCard>
      <template #header>
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-semibold">Team Members</h3>
          <UButton
            @click="showCreateModal = true"
            size="sm"
            color="primary"
          >
            
            Add User
          </UButton>
        </div>
      </template>

      <UTable
        :columns="columns"
        :rows="users"
        :loading="loading"
      >
        <template #role-data="{ row }">
          <UBadge variant="soft" size="xs">{{ row.role?.name || row.role || 'User' }}</UBadge>
        </template>

        <template #actions-data="{ row }">
          <div class="flex items-center gap-2">
            <UButton
              @click="editUser(row)"
              variant="ghost"
              size="xs"
            >
              Edit
            </UButton>
            <UButton
              @click="deleteUser(row)"
              variant="ghost"
              size="xs"
              color="red"
            >
              Delete
            </UButton>
          </div>
        </template>
      </UTable>
    </UCard>

    <!-- Create/Edit User Modal -->
    <UModal v-model:open="showCreateModal">
      <UCard>
        <template #header>
          <h3 class="text-lg font-semibold">{{ editingUser ? 'Edit User' : 'Create User' }}</h3>
        </template>

        <div class="space-y-4">
          <UInput
            v-model="form.email"
            label="Email"
            type="email"
            placeholder="user@example.com"
          />

          <UInput
            v-model="form.name"
            label="Name"
            placeholder="John Doe"
          />

          <UInput
            v-if="!editingUser"
            v-model="form.password"
            label="Password"
            type="password"
            placeholder="••••••••"
          />

          <USelect
            v-model="form.role"
            label="Role"
            :options="roleOptions"
            option-attribute="name"
            value-attribute="id"
          />

          <div class="flex justify-end gap-2">
            <UButton @click="showCreateModal = false" variant="outline">Cancel</UButton>
            <UButton @click="saveUser" :loading="saving">
              {{ editingUser ? 'Update' : 'Create' }}
            </UButton>
          </div>
        </div>
      </UCard>
    </UModal>
  </div>
</template>

<script setup>
// ==========================================
// RLS Extension: Team A Dashboard
// All queries automatically filtered by Pre-Hook
// ==========================================

const toast = useToast();
const { me } = useEnfyraAuth();
const confirm = useConfirm();

// State
const loading = ref(false);
const saving = ref(false);
const showCreateModal = ref(false);
const editingUser = ref(null);

// User data - RLS: Only fetches project_id = 'A'
const { data: usersData, refresh, pending } = useApi('/user_definition', {
  query: {
    fields: 'id,email,name,role,project_id',
    include: 'role',
    limit: 100
  }
});

const users = computed(() => usersData.value?.data || []);

// Stats - RLS: Only counts project_id = 'A'
const stats = computed(() => ({
  totalUsers: users.value.length,
  newThisWeek: users.value.filter(u => {
    const created = new Date(u.createdAt);
    const weekAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    return created >= weekAgo;
  }).length,
  activeToday: Math.floor(users.value.length * 0.3) // Example
}));

// Table columns
const columns = [
  { key: 'name', label: 'Name', sortable: true },
  { key: 'email', label: 'Email', sortable: true },
  { key: 'role', label: 'Role' },
  { key: 'actions', label: 'Actions' }
];

// Role options
const roleOptions = ref([]);

// Form data
const form = reactive({
  email: '',
  name: '',
  password: '',
  role: null
});

// Load roles on mount
onMounted(async () => {
  const { data } = await useApi('/role_definition', {
    query: { fields: 'id,name', limit: 100 }
  });
  roleOptions.value = data?.data || [];
});

// Edit user
const editUser = (user) => {
  editingUser.value = user;
  form.email = user.email;
  form.name = user.name;
  form.role = user.role?.id || user.role;
  showCreateModal.value = true;
};

// Save user (create or update)
const saveUser = async () => {
  saving.value = true;

  const body = {
    email: form.email,
    name: form.name,
    role: { id: form.role }
  };

  if (!editingUser.value) {
    body.password = form.password;
    // NOTE: project_id will be auto-assigned by Pre-Hook!
  }

  try {
    if (editingUser.value) {
      // Update existing user
      await useApi(`/user_definition/${editingUser.value.id}`, {
        method: 'PATCH',
        body
      });
      toast.add({
        title: 'Success',
        description: 'User updated successfully',
        color: 'green'
      });
    } else {
      // Create new user (project_id auto-assigned by Pre-Hook)
      await useApi('/user_definition', {
        method: 'POST',
        body
      });
      toast.add({
        title: 'Success',
        description: 'User created successfully',
        color: 'green'
      });
    }

    showCreateModal.value = false;
    resetForm();
    await refresh();
  } catch (error) {
    toast.add({
      title: 'Error',
      description: error.message,
      color: 'red'
    });
  } finally {
    saving.value = false;
  }
};

// Delete user
const deleteUser = async (user) => {
  const confirmed = await confirm({
    title: 'Delete User',
    content: `Are you sure you want to delete ${user.name}?`
  });

  if (!confirmed) return;

  try {
    await useApi(`/user_definition/${user.id}`, {
      method: 'DELETE'
    });
    toast.add({
      title: 'Success',
      description: 'User deleted',
      color: 'green'
    });
    await refresh();
  } catch (error) {
    toast.add({
      title: 'Error',
      description: error.message,
      color: 'red'
    });
  }
};

// Reset form
const resetForm = () => {
  editingUser.value = null;
  form.email = '';
  form.name = '';
  form.password = '';
  form.role = null;
};

// Register header actions
useHeaderActionRegistry().register({
  id: 'refresh-users',
  label: 'Refresh',
  variant: 'soft',
  onClick: refresh
});
</script>
```

5. Click **Create** to save the extension

### Step 4.2: Create Team B Dashboard (Duplicate and Modify)

1. Repeat Step 4.1, but change:
   - **Name:** `Dashboard B`
   - **Header color:** Change `blue-600` to `red-600`
   - **Badge text:** `Project B Only`
   - **Title:** `Team B Dashboard`

2. The **same code works** for both teams because:
   - Pre-Hook automatically filters by `@USER.project_id`
   - Team A users see only project A data
   - Team B users see only project B data

---

## Phase 5: Menu + Permission Setup

### Step 5.1: Create Menu for Dashboard A

1. Navigate to **Settings**  **Menu**

2. Click **Create Menu**

3. Configure:
   - **Type:** `Menu`
   - **Label:** `Dashboard`
   - **Path:** `/dashboard-a`
   - **Icon:** ``
   - **Category:** (select or create "Team A")
   - **Is Enabled:** `Yes`
   - **Extension:** Select `Dashboard A` (the extension you created)

4. In the **Permission** field, click to open Permission Builder:

5. Create permission rule:
   - Click **+ Add Group**
   - **Permission 1:**
     - **Route:** `/user_definition`
     - **Actions:** Read, Create, Update (check these)
   - This ensures only users with user_management permissions can access

6. Click **Save**

### Step 5.2: Create Menu for Dashboard B

Repeat Step 5.1 with:
- **Path:** `/dashboard-b`
- **Extension:** `Dashboard B`
- **Category:** "Team B"

---

## Phase 6: Testing

### Test Case 1: Team A User Login

1. Login as a Team A user (`project_id = "A"`)

2. **Verify:**
   -  Sidebar shows "Dashboard" under Team A
   -  Click Dashboard  Shows Team A Dashboard
   -  Stats show only Team A users count
   -  Table shows only Team A users
   -  Create user  Auto-assigned `project_id = "A"`
   -  Try to access `/dashboard-b`  403 or empty

### Test Case 2: Team B User Login

1. Login as a Team B user (`project_id = "B"`)

2. **Verify:**
   -  Sidebar shows "Dashboard" under Team B
   -  Click Dashboard  Shows Team B Dashboard
   -  Stats show only Team B users count
   -  Table shows only Team B users
   -  Create user  Auto-assigned `project_id = "B"`
   -  Try to access `/dashboard-a`  403 or empty

### Test Case 3: Root Admin Login

1. Login as Root Admin

2. **Verify:**
   -  Sidebar shows BOTH dashboards
   -  Can access `/dashboard-a` and `/dashboard-b`
   -  Stats show ALL users (no RLS filter for admin)
   -  Table shows ALL users from both teams

### Test Case 4: API-Level RLS

1. Login as Team A user

2. Open browser DevTools  Network tab

3. Make API call: `GET /api/user_definition`

4. **Verify:**
   -  Response only contains users with `project_id = "A"`
   -  No Team B users in response

5. Try to create user with different project_id:
   ```json
   POST /api/user_definition
   {
     "email": "test@example.com",
     "name": "Test",
     "project_id": "B"  // Try to assign to Team B
   }
   ```

6. **Verify:**
   -  Request fails with error "Cannot create records for another project"
   -  User is created with `project_id = "A"` (auto-assigned)

---

## Troubleshooting

### Issue: Users See All Data (RLS Not Working)

**Check:**
1. Pre-Hook is enabled and has correct priority (100)
2. User has `project_id` set in their profile
3. Pre-Hook logic has correct field name (`project_id`)

**Debug:**
```javascript
// Add to Pre-Hook for debugging
@LOGS(`User project: ${userProject}, Method: ${method}`);
```

Check logs in **Settings**  **Logs**

### Issue: Cannot Create New Users

**Check:**
1. User has permission for `POST /user_definition`
2. Pre-Hook doesn't have syntax errors
3. Form includes all required fields

### Issue: Dashboard Shows Empty

**Check:**
1. Extension is enabled
2. Menu is linked to correct extension
3. User has permission to access the route
4. There's actual data with matching `project_id`

### Issue: User Can Access Other Team's Dashboard

**Check:**
1. Menu permission is configured correctly
2. Extension doesn't have hardcoded project_id
3. Pre-Hook is running before the handler (priority 100)

---

## Next Steps

### Extend This Example

1. **Add More Tables:** Apply same RLS pattern to products, orders, etc.

2. **Multi-Level RLS:** Add region, department levels

3. **Audit Logging:** Log all RLS-enforced operations

4. **Custom Reports:** Create report extensions per team

5. **Data Sharing:** Allow cross-team collaboration with explicit permissions

### Related Documentation

- [Pre-Hooks Guide](../server/hooks-handlers/prehooks.md)
- [Permission Builder](../app/permission-builder.md)
- [Extension System](../app/extension-system.md)
- [Custom Handlers](../app/hooks-handlers/custom-handlers.md)

---

## Summary

```
What You Built:
├── 1 shared user_definition table
├── Pre-Hook RLS enforcement (automatic)
├── 2 custom dashboards (Team A + Team B)
├── Complete data isolation
└── Zero code changes for each new team

Key Concepts:
├── RLS Field: project_id in each table
├── Pre-Hook: Auto-inject filter based on @USER.project_id
├── Extensions: Custom UI per team
└── Permissions: Control access to dashboards

Benefits:
├ Single codebase for all teams
├ Automatic enforcement (no manual filtering)
├ Easy to add new teams (just set project_id)
└ Root admin sees everything for oversight
```

**Time Saved:** Instead of building separate admin panels for each team, you built ONE system that scales to 100+ teams with just metadata configuration.

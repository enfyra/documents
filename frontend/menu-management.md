# Menu Management

Menu Management lets you create custom navigation menus for your application. You can create sidebar icons, dropdown menus, and individual menu items with custom permissions and ordering.

## Accessing Menu Management

1. Navigate to **Settings → Menu** in the sidebar
2. You'll see all existing menus displayed as cards
3. System menus (like Dashboard, Settings) are protected and cannot be deleted

## Menu Types

### Mini Sidebar
- **Purpose**: Main sidebar icons (like Dashboard, Data, Collections)
- **Displays**: As icons in the left sidebar
- **Can contain**: Dropdown menus and direct menu items

### Dropdown Menu
- **Purpose**: Collapsible menu sections within a sidebar
- **Displays**: As expandable sections with arrow icons
- **Must belong to**: A Mini Sidebar
- **Can contain**: Individual menu items

### Menu
- **Purpose**: Individual navigation links
- **Displays**: As clickable menu items
- **Must belong to**: Either a Mini Sidebar or a Dropdown Menu

## Creating a New Menu

### Step 1: Start Creation
1. Click the **"Create"** button at the top right
2. You'll see the menu creation form

### Step 2: Basic Information
- **Label**: The display name for your menu item
- **Description**: Optional description of what this menu does
- **Icon**: Choose a Lucide icon (defaults to "menu")
- **Path**: The URL route this menu navigates to (e.g., `/my-custom-page`)

### Step 3: Choose Menu Type
Select the type from the dropdown:
- **Mini Sidebar**: Creates a new main sidebar section
- **Dropdown Menu**: Creates a collapsible menu section
- **Menu**: Creates an individual menu item

### Step 4: Set Hierarchy (for Dropdown Menu and Menu types)

**For Dropdown Menu:**
- **Sidebar**: Use the relation picker to select which Mini Sidebar this belongs to

**For Menu items:**
You can choose either:
- **Sidebar**: Place directly in a Mini Sidebar (top level)
- **Parent**: Place under a Dropdown Menu (nested)

**Note**: You cannot select both sidebar and parent - the form will automatically clear the other field when you make a selection.

### Step 5: Configure Settings
- **Order**: Number to control display order (lower numbers appear first)
- **Is Enabled**: Toggle to enable/disable the menu item
- **Permission**: Click "Configure Permission" to set access rules (see below)

### Step 6: Save
Click **"Create"** to save your new menu item

## Managing Existing Menus

### Quick Actions
- **Enable/Disable Toggle**: Click the switch on each menu card to enable or disable
- **Edit Menu**: Click on any menu card to open the edit form
- **Delete Menu**: Use the delete button (not available for system menus)

### Bulk Operations
- **Filter Menus**: Use the filter dropdown to show only specific menu types
- **Select Multiple**: Use checkboxes to select multiple menus for bulk operations

## Permission System  

### How Menu Permissions Work

The menu system uses `PermissionGate` and `usePermissions` internally to automatically show/hide menu items based on user permissions. When you configure permissions for a menu, the system:

1. **Evaluates permissions** when the menu loads
2. **Shows menu** only if user has required permissions
3. **Hides parent menus** if user can't access any child items
4. **Updates automatically** when permissions change

**See [Permission Components](./permission-components.md) for technical details**

### Setting Menu Permissions

1. Click **"Configure Permission"** in the menu form
2. Use the [Permission Builder](./permission-builder.md) to create rules:
   - **Allow All**: Makes menu visible to everyone
   - **Route + Actions**: Specify which route permissions are required
   - **AND/OR Logic**: Combine multiple permission conditions

### Common Permission Examples

**Admin Only:**
- Select route: `/admin`
- Enable action: Read

**Multiple Route Access (OR):**
- Add OR group
- Add permission 1: Route `/users`, Action: Read
- Add permission 2: Route `/settings`, Action: Update

**Complex Permissions (AND/OR):**
- Add AND group
- Add permission: Route `/products`, Action: Read
- Add OR sub-group:
  - Permission 1: Route `/products`, Action: Create
  - Permission 2: Route `/products`, Action: Update

## Menu Hierarchy Examples

### Simple Sidebar Menu
1. Create **Mini Sidebar**: "Reports" (`/reports`)
2. Create **Menu** items under Reports sidebar:
   - "Sales Report" (`/reports/sales`)
   - "User Report" (`/reports/users`)

### Complex Nested Structure
1. Create **Mini Sidebar**: "Management" (`/management`)
2. Create **Dropdown Menu**: "User Management" (under Management sidebar)
3. Create **Menu** items under User Management dropdown:
   - "User List" (`/management/users`)
   - "User Roles" (`/management/roles`)

## Important Notes

### System Menus
- **Protected**: System menus (Dashboard, Data, Collections, Settings, Files) cannot be deleted
- **Limited Editing**: You can modify labels and icons but core functionality is protected
- **Recognition**: System menus show a "System" badge

### Path Requirements
- **Must start with /**: All paths must begin with a forward slash
- **Unique paths**: Each menu should have a unique path to avoid conflicts
- **Route matching**: Ensure your path matches actual routes in your application

### Order and Organization
- **Numerical ordering**: Lower numbers appear first (0, 1, 2, etc.)
- **Visual hierarchy**: Mini Sidebars contain Dropdown Menus, which contain Menu items
- **Logical grouping**: Group related functionality under the same sidebar or dropdown

## Troubleshooting

### Menu Not Appearing
- Check that **Is Enabled** is turned on
- Verify the user has required permissions
- Ensure the menu type and hierarchy are set correctly

### Permission Issues
- Confirm permission rules are configured correctly
- Check that the user's role includes the required route permissions
- Test with "Allow All" to isolate permission vs. other issues

### Hierarchy Problems
- **Dropdown Menu**: Must have a sidebar selected
- **Menu item**: Must have either sidebar OR parent selected (not both)
- **Mini Sidebar**: Should not have parent or sidebar selected
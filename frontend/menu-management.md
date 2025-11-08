# Menu Management

Menu Management lets you create custom navigation menus for your application. You can create top-level menus, dropdown menus, and individual menu items with custom permissions and ordering.

## Accessing Menu Management

1. Navigate to **Settings → Menu** in the sidebar
2. You'll see all existing menus displayed as cards
3. System menus (like Dashboard, Settings) are protected and cannot be deleted

## Menu Types

### Menu
- **Purpose**: Top-level navigation items or individual menu links
- **Displays**: As clickable menu items in the sidebar
- **Can be**: 
  - A simple menu with a direct route (navigates to a page)
  - A menu group with children items (displays as expandable section)
- **Can contain**: Child menu items (optional)

### Dropdown Menu
- **Purpose**: Collapsible menu sections that group related menu items
- **Displays**: As expandable sections with arrow icons
- **Can contain**: Individual menu items
- **Behavior**: Click to expand/collapse and reveal child items

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
- **Menu**: Creates a top-level menu item (can have a route or contain child items)
- **Dropdown Menu**: Creates a collapsible menu section for grouping related items

### Step 4: Set Hierarchy (Optional)

**For Menu:**
- **Parent**: Use the relation picker to select a parent menu (Dropdown Menu or Menu with children)
- If no parent is selected, the menu appears at the top level
- **Path**: Set the route path if this menu should navigate to a page

**For Dropdown Menu:**
- **Parent**: Use the relation picker to select a parent menu (optional)
- If no parent is selected, the dropdown appears at the top level
- Dropdown menus typically don't have routes - they're containers for other items

**Note**: You can nest menus under other menus or dropdown menus to create hierarchical structures.

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

### Simple Top-Level Menu
1. Create **Menu**: "Reports" with path `/reports`
2. This appears as a clickable menu item that navigates to the reports page

### Menu with Children
1. Create **Menu**: "Management" (no path, or set a default path)
2. Create **Menu** items with parent set to "Management":
   - "User List" (`/management/users`)
   - "User Roles" (`/management/roles`)
3. The Management menu becomes expandable and shows its children when clicked

### Dropdown Menu Structure
1. Create **Dropdown Menu**: "User Management" (no path needed)
2. Create **Menu** items with parent set to "User Management":
   - "User List" (`/management/users`)
   - "User Roles" (`/management/roles`)
3. The dropdown menu appears as a collapsible section with arrow icon

### Complex Nested Structure
1. Create **Menu**: "Settings" (top level)
2. Create **Dropdown Menu**: "User Management" with parent "Settings"
3. Create **Menu** items with parent "User Management":
   - "User List" (`/settings/users`)
   - "User Roles" (`/settings/roles`)
4. This creates: Settings → User Management → (User List, User Roles)

## Important Notes

### System Menus
- **Protected**: System menus (Dashboard, Data, Collections, Settings, Storage) cannot be deleted
- **Limited Editing**: You can modify labels and icons but core functionality is protected
- **Recognition**: System menus show a "System" badge

### Path Requirements
- **Must start with /**: All paths must begin with a forward slash
- **Unique paths**: Each menu should have a unique path to avoid conflicts
- **Route matching**: Ensure your path matches actual routes in your application

### Order and Organization
- **Numerical ordering**: Lower numbers appear first (0, 1, 2, etc.)
- **Visual hierarchy**: Menus can contain Dropdown Menus or other Menu items
- **Logical grouping**: Group related functionality under the same parent menu or dropdown
- **Position**: Use the position field to place menus at "top" or "bottom" of the sidebar

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
- **Menu**: Can have a parent (Menu or Dropdown Menu) or be top-level
- **Dropdown Menu**: Can have a parent (Menu or Dropdown Menu) or be top-level
- **Circular references**: Ensure menus don't reference themselves as parents
- **Path conflicts**: Each menu with a route should have a unique path
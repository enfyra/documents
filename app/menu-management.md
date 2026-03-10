# Menu Management

Menu Management provides a visual interface for creating and organizing navigation menus in your application. You can create hierarchical menu structures with drag-and-drop reordering, configure permissions, and manage menu items all through an intuitive visual editor.

## Accessing Menu Management

1. Navigate to **Settings  Menus** in the sidebar
2. You'll see a visual tree representation of all menus
3. System menus (like Dashboard, Settings) are protected and cannot be deleted

## Menu Types

### Menu
- **Purpose**: Individual menu items that navigate to specific pages or act as containers for child items
- **Displays**: As clickable menu items in the sidebar
- **Can be**: 
  - A simple menu with a direct route (navigates to a page)
  - A menu group with children items (displays as expandable section)
- **Can contain**: Child menu items (optional)

### Dropdown Menu
- **Purpose**: Collapsible menu sections that group related menu items
- **Displays**: As expandable sections with arrow icons
- **Can contain**: Individual menu items or other dropdown menus
- **Behavior**: Click to expand/collapse and reveal child items

## Menu Visual Editor

The Menu Visual Editor provides an intuitive drag-and-drop interface for managing menus:

- **Tree View**: Hierarchical representation of your menu structure
- **Drag and Drop**: Reorder menus by dragging them within the tree
- **Move Between Levels**: Drag menus to different parent menus or to root level
- **Visual Hierarchy**: See parent-child relationships clearly
- **Quick Actions**: Edit, delete, enable/disable directly from the tree

## Creating a New Menu

### Step 1: Start Creation
1. Click the **"Create Menu"** button in the header actions
2. You'll see the menu creation form in a modal

### Step 2: Basic Information
- **Label**: The display name for your menu item
- **Description**: Optional description of what this menu does
- **Icon**: Choose a Lucide icon (defaults to "menu")
- **Path**: The URL route this menu navigates to (e.g., `/my-custom-page`)
  - **Important**: Path must start with `/`
  - Leave empty if this menu is just a container for child items

### Step 3: Choose Menu Type
Select the type from the dropdown:
- **Menu**: Creates a menu item (can have a route or contain child items)
- **Dropdown Menu**: Creates a collapsible menu section for grouping related items
  - Dropdown menus typically don't have routes - they're containers

### Step 4: Set Hierarchy (Optional)

**For Menu:**
- **Parent**: Use the relation picker to select a parent menu (Dropdown Menu or Menu with children)
- If no parent is selected, the menu appears at the top level
- **Path**: Set the route path if this menu should navigate to a page

**For Dropdown Menu:**
- **Parent**: Use the relation picker to select a parent menu (optional)
- If no parent is selected, the dropdown appears at the top level
- Dropdown menus typically don't have routes - they're containers for other items

**Note**: You can nest menus under other menus or dropdown menus to create hierarchical structures. Use the visual editor to reorganize after creation.

### Step 5: Configure Settings
- **Order**: Number to control display order (lower numbers appear first)
- **Is Enabled**: Toggle to enable/disable the menu item
- **Permission**: Click "Configure Permission" to set access rules (see below)

### Step 6: Save
Click **"Create"** to save your new menu item

## Managing Menus with Visual Editor

### Drag and Drop Reordering

The visual editor allows you to reorganize menus easily:

1. **Reorder Siblings**: Drag a menu item up or down within its current level
2. **Move to Different Parent**: Drag a menu item onto a different parent menu
3. **Move to Root**: Use the "Move to Root" option for menus that have a parent
4. **Visual Feedback**: The interface shows where the menu will be placed as you drag

### Quick Actions from Tree View

Each menu item in the tree provides quick actions:

- **Edit**: Click the edit icon to modify the menu item
- **Delete**: Click the delete icon (not available for system menus)
- **Enable/Disable Toggle**: Click the toggle switch to enable or disable
- **Add Child**: Click to create a new menu item with this menu as parent
- **Add Extension**: Link an extension to this menu (if applicable)

### Editing Menus

1. **Click on any menu item** in the visual tree to open the edit modal
2. **Modify any fields** (label, icon, path, permissions, etc.)
3. **Click "Save"** to apply changes

**Note**: Changes to menu properties (except order/parent) are saved immediately. For reordering, use drag-and-drop.

### Deleting Menus

1. **Click the delete icon** on any menu item (except system menus)
2. **Confirm deletion** in the confirmation dialog
3. **Child menus** are automatically moved to the parent's parent or root level

**Warning**: Deleting a menu also removes any child menus that depend on it as a parent. The system will automatically reorganize the hierarchy.

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
4. This creates: Settings  User Management  (User List, User Roles)

**Tip**: Use the visual editor to create the hierarchy first, then reorganize with drag-and-drop as needed.

## Extension Integration

Menus can be linked to **Extensions** to provide dynamic content or functionality:

1. **Create or select a menu** in the visual editor
2. **Click "Add Extension"** or use the extension option
3. **Select an extension** from the relation picker
4. The menu will display the extension's content when clicked

**Note**: Extensions provide custom page content or functionality. See [Extension System](./extension-system.md) for details.

## Important Notes

### System Menus
- **Protected**: System menus (Dashboard, Data, Collections, Settings, Storage) cannot be deleted
- **Limited Editing**: You can modify labels and icons but core functionality is protected
- **Recognition**: System menus show a "System" badge

### Path Requirements
- **Must start with /**: All paths must begin with a forward slash
- **Unique paths**: Each menu should have a unique path to avoid conflicts (when a path is provided)
- **Route matching**: Ensure your path matches actual routes in your application
- **Empty paths**: Allowed for container menus (menus that only have children, no direct navigation)

### Order and Organization
- **Numerical ordering**: Lower numbers appear first (0, 1, 2, etc.)
- **Visual hierarchy**: Menus can contain Dropdown Menus or other Menu items
- **Logical grouping**: Group related functionality under the same parent menu or dropdown
- **Position**: Use the position field to place menus at "top" or "bottom" of the sidebar
- **Drag-and-drop**: The visual editor allows easy reordering without editing each menu

### Visual Editor Benefits
- **Intuitive Organization**: See the entire menu structure at a glance
- **Easy Reordering**: Drag menus to reorganize without editing individual items
- **Quick Actions**: Edit, delete, enable/disable directly from the tree
- **Hierarchy Visualization**: Clearly see parent-child relationships

## Troubleshooting

### Menu Not Appearing
- Check that **Is Enabled** is turned on
- Verify the user has required permissions
- Ensure the menu type and hierarchy are set correctly
- Check that the menu's parent is also enabled and visible

### Permission Issues
- Confirm permission rules are configured correctly
- Check that the user's role includes the required route permissions
- Test with "Allow All" to isolate permission vs. other issues
- Remember that parent menus hide if no child items are accessible

### Hierarchy Problems
- **Menu**: Can have a parent (Menu or Dropdown Menu) or be top-level
- **Dropdown Menu**: Can have a parent (Menu or Dropdown Menu) or be top-level
- **Circular references**: The system prevents menus from referencing themselves as parents
- **Path conflicts**: Each menu with a route should have a unique path
- **Orphaned menus**: Menus with deleted parents automatically move to root level

### Drag and Drop Issues
- Ensure you're dragging to a valid drop zone (another menu or root area)
- The visual feedback shows where the menu will be placed
- If a move seems incorrect, use "Move to Root" and then reorganize
- Check that the target parent can contain children (some menu types may have restrictions)

## Best Practices

### Menu Organization
- **Group Related Items**: Use Dropdown Menus to group related functionality
- **Logical Hierarchy**: Create a clear, intuitive navigation structure
- **Consistent Naming**: Use clear, descriptive labels
- **Limit Nesting Depth**: Avoid deep nesting (3-4 levels max) for better UX

### Permissions
- **Least Privilege**: Only grant permissions to users who need them
- **Role-Based**: Use roles for common permission patterns
- **Test Thoroughly**: Test menu visibility with different user roles
- **Document Complex Rules**: Add descriptions to permission configurations

### Visual Editor Usage
- **Plan Structure First**: Sketch your menu hierarchy before creating
- **Use Drag-and-Drop**: Reorganize menus visually rather than editing order numbers
- **Regular Review**: Periodically review and optimize your menu structure
- **Backup Before Major Changes**: Consider documenting your structure before major reorganizations

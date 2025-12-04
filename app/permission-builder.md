# Permission Builder

The Permission Builder is a visual interface for creating complex access control rules. Instead of writing permission code, you can build permission conditions using a drag-and-drop style interface with logical operators.

## Accessing Permission Builder

Permission fields appear in various forms throughout Enfyra:
- **Menu Management**: Set menu visibility permissions
- **Route Management**: Configure route access permissions  
- **User Management**: Set user-specific permissions
- **Custom Forms**: Any form field with permission type

**How to Identify Permission Fields:**
- Look for fields with shield icons ()
- Click the field to open the Permission Builder drawer

## Building Permissions

### Step 1: Choose Permission Type

When you open the Permission Builder, you have two options:

**Allow All Access:**
- Toggle **"Allow All"** to grant unrestricted access
- This bypasses all permission checks
- Use sparingly for admin-level access only

**Build Custom Permissions:**
- Leave "Allow All" off to create specific permission rules
- Build detailed conditions using groups and rules

### Step 2: Create Permission Groups

**Add a Group:**
1. Click **"+ Add Group"** to create a permission container
2. Choose the group logic:
   - **AND**: All conditions in the group must be true
   - **OR**: Any condition in the group can be true

**Group Examples:**
- **AND Group**: User must have both "read posts" AND "edit posts"
- **OR Group**: User needs either "admin role" OR "editor role"

### Step 3: Add Permission Rules

**Within each group:**
1. Click **"+ Add Permission"** to create a rule
2. **Select Route**: Click the route picker to choose which API endpoint
3. **Choose Actions**: Toggle the actions this rule allows:
   - **Create** (POST): Can create new records
   - **Read** (GET): Can view/list records
   - **Update** (PATCH): Can edit existing records
   - **Delete** (DELETE): Can remove records

**Route Picker:**
- Search for routes by name or path
- Routes are displayed as `/table-name` or custom paths
- Select the specific endpoint you want to control

**Action Toggles:**
- Click action badges to enable/disable permissions
- Enabled actions show in color, disabled are grayed out
- Must select at least one action for the rule to be valid

### Step 4: Build Complex Logic

**Nested Groups:**
- Add groups within groups for complex conditions
- Example: `(Admin OR Editor) AND (Read Posts OR Write Posts)`

**Multiple Rules:**
- Add multiple permission rules to the same group
- Each rule can target different routes
- Combine different routes with AND/OR logic

## Permission Builder Examples

### Simple Permission
**Goal**: Allow reading user profiles

1. **Add Group** (AND logic - default)
2. **Add Permission** in the group:
   - **Route**: `/users`
   - **Actions**: Read 

### Complex Permission  
**Goal**: Admin or Editor can manage posts, OR any user can read posts

1. **Main Group** (OR logic)

2. **First Sub-Group** (AND logic):
   - **Permission 1**: Route `/roles`, Actions: Read   
   - **Permission 2**: Route `/posts`, Actions: Create , Update , Delete 

3. **Second Permission** (in main OR group):
   - **Route**: `/posts`
   - **Actions**: Read 

### Real-World Menu Example
**Goal**: Show "User Management" menu only to users who can manage users OR view user reports

1. **Main Group** (OR logic)
2. **Permission 1**: Route `/users`, Actions: Create , Update , Delete 
3. **Permission 2**: Route `/reports/users`, Actions: Read 

## Visual Interface Guide

### Permission Field Display
- **Shield with Check** : Permissions configured
- **Plain Shield** : Allow All enabled  
- **Shield with X** : No permissions configured

### Group Headers
- **AND/OR Toggle**: Switch between logical operators
- **Group Actions**: Add permissions, add sub-groups, delete group

### Permission Rules
- **Route Badge**: Shows selected route path
- **Action Badges**: Colored badges for enabled actions
- **Edit Button**: Modify rule settings
- **Delete Button**: Remove rule

### Validation Indicators
- **Red Border**: Required fields missing
- **Error Messages**: Specific validation issues
- **Save Button State**: Disabled until form is valid

## Integration with Other Systems

### Menu System
Permission Builder controls menu visibility:
- Menus only show for users with required permissions
- See [Menu Management](./menu-management.md) for menu-specific usage

### Route System  
Controls API endpoint access:
- Works with Published Methods and Role Permissions
- See [Permission System](../server/permission-system.md) for technical details

### Form System
Integrated into Enfyra's form rendering:
- Automatic permission field detection
- Consistent UI across all permission fields
- See [Form System](#form-system-integration) below

## Form System Integration

### Automatic Field Detection
Enfyra automatically detects permission fields in forms and renders the Permission Builder interface.

### Field Types
- **Permission Field**: Renders Permission Builder on click
- **Inline Editor**: Shows permission summary with edit capability
- **JSON Editor**: Advanced users can edit permission JSON directly

### Form Validation
- Permission fields validate automatically
- Required routes and actions are checked
- Form submission blocked until permissions are valid

### Data Binding
- Permission changes save automatically when you close the builder
- No need to click additional save buttons
- Real-time validation during editing

## Best Practices

### Permission Design
- **Start Simple**: Begin with basic permissions, add complexity as needed
- **Logical Grouping**: Group related permissions together with appropriate AND/OR logic
- **Clear Purpose**: Each permission rule should have a clear business purpose

### User Experience
- **Test Thoroughly**: Verify permissions work correctly with test users
- **Document Complex Rules**: Add descriptions for complex permission structures
- **Regular Review**: Periodically audit permissions to ensure they're still needed

### Performance Considerations
- **Avoid Over-Complexity**: Too many nested groups can impact performance
- **Route Specificity**: Use specific routes rather than overly broad permissions
- **Cache-Friendly**: Simple permission structures cache better

## Troubleshooting

### Permission Not Working
- **Check Route Path**: Ensure the route path exactly matches the API endpoint
- **Verify Actions**: Confirm the correct actions (CRUD) are selected
- **Test Group Logic**: Make sure AND/OR operators create the intended logic

### Form Validation Errors
- **Required Route**: Each permission rule needs a route selected
- **Required Actions**: At least one action must be enabled per rule
- **Empty Groups**: Remove groups that contain no permission rules

### Performance Issues
- **Simplify Structure**: Reduce nested groups if experiencing slowness
- **Specific Routes**: Use targeted routes instead of broad permissions
- **Cache Reset**: Complex permission changes may require cache refresh
# Permission System

Enfyra uses a role-based access control (RBAC) system with special bypass mechanisms. The permission system is fully integrated between backend and frontend - permissions configured in the backend automatically control what users can see and do in the UI.

## Permission Hierarchy

Access to API endpoints follows this priority order:

1. **Published Methods**: Public access, no authentication required
2. **Allowed Users**: Specific users bypass role restrictions for configured methods
3. **Role Permissions**: Users with assigned roles get access to role-specific methods
4. **No Access**: Users without proper permissions are denied

## Role-Based Access Control

### How RBAC Works
- Users are assigned a **role** (e.g., Admin, Editor, Viewer)
- Each role has specific **route permissions** for API endpoints
- Permissions specify which HTTP methods (GET, POST, PATCH, DELETE) the role can access
- Users inherit all permissions from their assigned role

### Route Permissions
Route permissions define access for specific API endpoints:
- **Route**: The API endpoint (e.g., `/users`, `/products`)
- **Methods**: Which HTTP methods are allowed (GET, POST, PATCH, DELETE)
- **Role**: Which role has this permission
- **Enabled**: Whether this permission is active

## AllowedUsers Bypass System

### What is AllowedUsers?
`allowedUsers` is a special feature that allows specific users to bypass role restrictions and get direct access to API endpoints.

### How it Works
- Individual users can be granted direct permissions to specific routes
- These permissions **completely bypass role checks**
- The user's role becomes irrelevant for that specific route
- Still respects HTTP method restrictions

### When to Use AllowedUsers
- **Exceptions**: When a specific user needs access that their role doesn't provide
- **Testing**: Temporary access for debugging or testing purposes  
- **Special Cases**: One-off permissions that don't warrant creating a new role
- **Granular Control**: Fine-tuned permissions for power users

### Priority Example
```
User: john@company.com
Role: Editor (has GET, POST access to /articles)
AllowedUsers: Direct DELETE access to /articles

Result: John can GET, POST, DELETE on /articles
- GET, POST from role permissions
- DELETE from allowedUsers bypass
```

## Published Methods (Public Access)

### What are Published Methods?
Published methods make specific HTTP methods publicly accessible without any authentication.

### Common Usage
- **GET methods**: Read-only public APIs
- **Public endpoints**: Contact forms, product catalogs, public data
- **File serving**: Public file downloads and media

### Security Considerations
- Only publish methods that should be truly public
- Avoid publishing write operations (POST, PATCH, DELETE) unless absolutely necessary
- Consider data sensitivity before making endpoints public

## Permission Management

### Through Admin Interface
1. **Settings → Roles**: Manage roles and their permissions
2. **Settings → Users**: Assign roles to users and configure allowedUsers
3. **Settings → Routings**: Set published methods for routes

### Permission Checking
- **Backend**: Automatic permission validation on every API request
- **Frontend**: UI elements automatically hide/show based on user permissions
- **Real-time**: Permissions are checked dynamically, changes take effect immediately

## Frontend-Backend Integration

### How Permissions Sync

**Automatic UI Updates:**
- Menus only show when users have required permissions
- Buttons and actions hide if user lacks permission
- Form fields become read-only without edit permission
- Data tables remove actions user cannot perform

**Permission Checking:**
1. **Backend**: Every API call validates permissions
2. **Frontend**: UI elements check permissions before rendering
3. **Real-time**: Changes to permissions take effect immediately
4. **Cached**: Frontend caches permissions for performance

### UI Components

**PermissionGate Component:**
- Wraps UI elements that require permissions
- Automatically hides content if permission check fails
- Supports complex AND/OR permission conditions

**usePermissions Composable:**
- Programmatic permission checking in code
- Check specific routes and actions
- Evaluate complex permission conditions

### Menu System Integration

Menus automatically respect permissions:
- Menu items only appear if user has permission
- Dropdown menus hide if no child items are accessible
- Sidebars disappear if user cannot access any items within
- Permissions are configured using the [Permission Builder](../frontend/permission-builder.md) UI

**How it works:**
1. Configure menu permissions through the UI using Permission Builder
2. System uses `PermissionGate` and `usePermissions` internally
3. Menus automatically show/hide based on user's permissions
4. No manual coding required - all handled through UI

See **[Permission Components](../frontend/permission-components.md)** for technical implementation details.

## Best Practices

### Role Design
- **Principle of Least Privilege**: Give users only the minimum permissions they need
- **Logical Grouping**: Create roles that match real job functions (Admin, Editor, Viewer)
- **Avoid Over-Complexity**: Too many roles can become hard to manage

### AllowedUsers Usage
- **Use Sparingly**: Prefer role-based permissions for most cases
- **Document Exceptions**: Keep track of why specific users have direct permissions
- **Regular Review**: Periodically audit allowedUsers assignments
- **Temporary Access**: Consider removing allowedUsers when no longer needed

### Security Guidelines
- **Sensitive Routes**: Be extra careful with user management, settings, and system routes
- **Write Permissions**: Restrict POST, PATCH, DELETE methods carefully
- **Public Methods**: Only publish truly public endpoints
- **Regular Audits**: Review permissions periodically for compliance
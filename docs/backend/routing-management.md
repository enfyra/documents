# Routing Management

Enfyra automatically generates API endpoints for every table you create (see [Getting Started](../getting-started/getting-started.md#automatic-api-generation)). The Routing Manager allows you to create custom endpoints that override these defaults, giving you full control over your API structure.

## Route Properties

### Basic Configuration
- **Path**: The custom endpoint path (e.g., `/assets/:id` instead of `/file_definition`)
- **Icon**: Visual identifier for the route (using Lucide icons)
- **Description**: Human-readable description of the route's purpose
- **Status**: Whether the route is enabled or disabled
- **System Route**: Indicates if this is a core system route

### Advanced Configuration
- **Main Table**: The primary table this route serves
- **Target Tables**: Additional tables this route can access (also provides automatic CRUD operations)
- **Route Permissions**: Access control rules for this endpoint
- **Handlers**: Custom request processing logic (see [Custom Handlers](./custom-handlers.md))
- **Hooks**: Lifecycle events and custom processing
- **Published Methods**: Defines which HTTP methods are public (no authentication required) vs private (role-protected)

## Creating Custom Routes

### Step 1: Access Route Manager
1. Navigate to **Settings → Routings**
2. Click **"Create Route"** button

### Step 2: Basic Configuration
Configure the essential route properties:
- **Path**: Set your custom endpoint (e.g., `/api/v1/products`, `/user-profiles`)
- **Icon**: Choose a Lucide icon for visual identification
- **Description**: Add a clear description of the route's purpose
- **Is Enabled**: Toggle to activate the route

### Step 3: Link to Main Table
The most important step is connecting your route to a data source:
1. Click the **relation picker** (pencil icon) next to **Main Table**
2. Search and select the table this route should serve
3. **Automatic CRUD Operations**: Once linked, your custom route can provide:
   - `GET /your-route` - List all records
   - `GET /your-route/:id` - Get single record
   - `POST /your-route` - Create new record
   - `PATCH /your-route/:id` - Update record
   - `DELETE /your-route/:id` - Delete record

4. **Published Methods Control**: HTTP methods specified in **Published Methods** become **public** (accessible by guests without authentication). Methods not listed remain **private** and require proper authentication and role permissions.

The route inherits all the table's fields, validation rules, and relationships without requiring additional configuration.

See [Relation Picker System](../frontend/relation-picker.md) for detailed usage of the relation selection interface.

### Step 4: Save and Test
1. Click **Save** to create the route
2. The custom endpoint becomes active immediately
3. All data management pages automatically use the new endpoint
4. API calls throughout the system switch to the custom path

### Step 5: Configure Route Permissions (Optional)
After creating the route, you can set up fine-grained access control:

1. **Access the edit page** of your newly created route
2. **Scroll to Route Permissions section** (only available on edit page, not during creation)
3. **Click "Add Permission"** to open the permission configuration drawer
4. **Configure permission settings:**
   - **Role**: Select which role gets access (single selection via relation picker)
   - **Methods**: Choose which HTTP methods this role can access (multiple selection)
   - **Allowed Users**: Select specific users who bypass role restrictions (multiple selection via relation picker)
   - **Is Enabled**: Toggle to activate/deactivate this permission rule
   - **Description**: Document the purpose of this permission

5. **Save the permission** - it takes effect immediately

**Important**: Route Permissions operate independently from the main route configuration. Changes to permissions (create, edit, delete) are applied instantly without requiring the "Save" button in the header actions.

### Managing Route Permissions
After creating permissions, they appear as a list in the Route Permissions section:
- **Permission entries** display the description (or "No description" if empty)
- **Status indicator** shows if the permission is enabled or disabled
- **Actions** include edit and delete options for each permission
- **Multiple permissions** can be configured for the same route with different roles/methods
- **Instant updates**: All permission changes take effect immediately, separate from route modifications

See [Relation Picker System](../frontend/relation-picker.md) for details on selecting roles and users through the relation interface.

## System Routes

System routes (marked with "System Route" badge) are core Enfyra endpoints that power the admin interface. These routes are automatically created and should generally not be modified unless you understand the implications.

Examples include routes for:
- File management and serving
- Route definition management
- Menu configuration
- User and permission systems

## Method Publishing and Access Control

### Published Methods vs Private Methods
The **Published Methods** field controls the authentication requirements for each HTTP method:

**Published Methods** (Public Access):
- **GET**: Anyone can read data without authentication
- **POST**: Guests can create records without login
- **PATCH**: Public record updates (no auth required)
- **DELETE**: Public record deletion (no auth required)

**Unpublished Methods** (Private Access):
- Require proper authentication and role permissions
- Protected by Enfyra's role-based access control system
- Checked against Route Permissions configuration
- Users must have appropriate role OR be listed in allowedUsers

### Permission Hierarchy
Access to private methods follows this priority order:
1. **Published Methods**: Public access, no authentication required
2. **Allowed Users**: Specific users bypass role restrictions for configured methods
3. **Role Permissions**: Users with assigned roles get access to role-specific methods
4. **No Access**: Users without proper permissions are denied

### Security Considerations
- **Typical Setup**: Only GET methods are published for read-only public APIs
- **Write Operations**: POST, PATCH, DELETE usually remain private for security
- **Allowed Users**: Use sparingly for exceptions, prefer role-based access
- **Empty Published Methods**: All methods require authentication and permissions

## Custom Route Behavior

### Default CRUD Operations
When you create a route and link it to targetTables, Enfyra automatically provides standard CRUD operations:
- `GET /your-route` - List records
- `GET /your-route/:id` - Get single record  
- `POST /your-route` - Create record
- `PATCH /your-route/:id` - Update record
- `DELETE /your-route/:id` - Delete record

### Custom Handlers Override
You can replace any of these default operations with custom business logic by creating handlers:

1. **Create the route** with targetTables configured
2. **Add custom handlers** via **Settings → Handlers** to override specific HTTP methods
3. **Handler takes precedence** - when a handler exists for a route+method, it executes instead of default CRUD

For detailed handler creation and examples, see [Custom Handlers](./custom-handlers.md).

### Route Status
- **Enabled/Disabled**: Controls whether the route is active
- **System Integration**: System routes are automatically integrated with the admin interface

## Custom Endpoint Benefits

### API Consistency
Create RESTful endpoints that match your application's naming conventions:
- `/products` instead of `/table_definition`
- `/orders/:id` instead of `/order_table`

### Versioning Support
Implement API versioning:
- `/api/v1/users`
- `/api/v2/users`

### Business Logic Integration
Routes can be enhanced with:
- Custom validation through handlers
- Data transformation via hooks
- Specialized permission rules

## Impact on Data Pages

When you create custom routes, all related functionality automatically adapts:
- **Data management pages** use the custom endpoint
- **Relation pickers** respect custom routes
- **Permission systems** apply to custom paths
- **API calls** throughout the system use the new endpoints

Changes to routes take effect immediately without requiring application restart.

## Best Practices

### Naming Conventions
- Use lowercase, hyphen-separated paths: `/user-profiles`
- Include version prefixes for API stability: `/api/v1/`
- Be descriptive but concise: `/product-categories` not `/pc`

### Route Organization
- Group related routes under common prefixes
- Use consistent patterns across your API
- Document the purpose of each custom route

### Permission Alignment
- Ensure route permissions align with table permissions
- Test custom routes with different user roles
- Consider the security implications of custom paths
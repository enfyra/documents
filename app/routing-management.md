# Routing Management

Routing Management lets you create custom API endpoints that are served by your **backend server**. By default, Enfyra automatically generates REST API endpoints for every table you create (like `/table_definition`). The Routing Manager allows you to create **custom endpoints** like `/products` or `/users` for any purpose you need.

**Important**: All routes are created and served by the **backend server**, not the frontend app. The frontend consumes these API endpoints via HTTP requests.

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
- **Handlers**: Custom request processing logic (see [Custom Handlers](hooks-handlers/custom-handlers.md))
- **Hooks**: Lifecycle events and custom processing
- **Published Methods**: Defines which HTTP methods are public (no authentication required) vs private (role-protected)

## Creating Custom Routes

### Step 1: Access Route Manager
1. Navigate to **Settings  Routings**
2. Click **"Create Route"** button

### Step 2: Basic Configuration
Configure the essential route properties:
- **Path**: Set your custom endpoint (e.g., `/v1/products`, `/user-profiles`)
- **Icon**: Choose a Lucide icon for visual identification
- **Description**: Add a clear description of the route's purpose
- **Is Enabled**: Toggle to activate the route

### Step 3: Link to Main Table
The most important step is connecting your route to a data source:
1. Click the **relation picker** (pencil icon) next to **Main Table**
2. Search and select the table this route should serve
3. **Automatic CRUD Operations**: Once linked, your custom route can provide:
   - `GET /your-route` - List all records
   - `POST /your-route` - Create new record
   - `PATCH /your-route/:id` - Update record
   - `DELETE /your-route/:id` - Delete record

4. **Published Methods Control**: HTTP methods specified in **Published Methods** become **public** (accessible by guests without authentication). Methods not listed remain **private** and require proper authentication and role permissions.

The route inherits all the table's fields, validation rules, and relationships without requiring additional configuration.

See [Relation Picker System](relation-picker.md) for detailed usage of the relation selection interface.

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
   - **Methods**: Choose which methods this role can access (multiple selection):
     - **REST API Methods**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`
     - **GraphQL Methods**: `GQL_QUERY`, `GQL_MUTATION`
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

See [Relation Picker System](relation-picker.md) for details on selecting roles and users through the relation interface.

### GraphQL Permission Control

GraphQL API access is controlled through two special methods in Route Permissions:

**`GQL_QUERY` Method:**
- Controls **read access** via GraphQL queries
- When enabled for a role, allows querying table data through GraphQL
- Example: `query { users { data { id name email } } }`
- Independent from REST `GET` permission

**`GQL_MUTATION` Method:**
- Controls **write access** (create, update, delete) via GraphQL mutations
- When enabled for a role, allows all mutation operations:
  - `create_table_name` - Create new records
  - `update_table_name` - Update existing records
  - `delete_table_name` - Delete records
- Independent from REST `POST`, `PUT`, `PATCH`, `DELETE` permissions

**Permission Examples:**

```
# Editor Role - Read-only GraphQL access
Role: Editor
Methods: [GQL_QUERY]
Result: Can query data but cannot create/update/delete via GraphQL

# Admin Role - Full GraphQL access  
Role: Admin
Methods: [GQL_QUERY, GQL_MUTATION]
Result: Can query and mutate data via GraphQL

# Developer Role - Mixed access
Role: Developer
Methods: [GET, POST, GQL_QUERY, GQL_MUTATION]
Result: Full REST and GraphQL access
```

**Important Notes:**
- GraphQL permissions are **separate** from REST permissions
- A role can have GraphQL access without REST access and vice versa
- `GQL_MUTATION` covers all mutation operations (create, update, delete) - you cannot separate them
- `GQL_QUERY` covers all query operations

For complete GraphQL API documentation, see **[GraphQL API Guide](../server/graphql-api.md)**.

## System Routes

System routes (marked with "System Route" badge) are core Enfyra endpoints that power the admin interface. These routes are automatically created and should generally not be modified unless you understand the implications.

Examples include routes for:
- File and storage management
- Route definition management
- Menu configuration
- User and permission systems

## Method Publishing and Access Control

The **Published Methods** field controls the authentication requirements for each HTTP method.

**For complete details on permissions, roles, and allowedUsers, see [Permission System Documentation](../server/permission-system.md).**

### Published Methods vs Private Methods
- **Published Methods**: Public access, no authentication required
- **Unpublished Methods**: Require authentication and proper permissions

### Quick Reference
1. **Published Methods**: Public access
2. **Allowed Users**: Bypass role restrictions  
3. **Role Permissions**: Standard role-based access
4. **No Access**: Denied

## Custom Route Behavior

### Default CRUD Operations
When you create a route and link it to targetTables, Enfyra automatically provides standard CRUD operations:

**REST API:**
- `GET /your-route` - List records
- `POST /your-route` - Create record
- `PATCH /your-route/:id` - Update record
- `DELETE /your-route/:id` - Delete record

**GraphQL API:**
- `query { your_table(...) }` - Query records
- `mutation { create_your_table(...) }` - Create record
- `mutation { update_your_table(...) }` - Update record
- `mutation { delete_your_table(...) }` - Delete record

### Custom Handlers Override
You can replace any of these default operations with custom business logic by creating handlers:

1. **Create the route** with targetTables configured
2. **Add custom handlers** via **Settings  Handlers** to override specific HTTP methods
3. **Handler takes precedence** - when a handler exists for a route+method, it executes instead of default CRUD

For detailed handler creation and examples, see [Custom Handlers](hooks-handlers/custom-handlers.md).

### Route Status
- **Enabled/Disabled**: Controls whether the route is active
- **System Integration**: System routes are automatically integrated with the admin interface

## Custom Endpoint Benefits

### API Consistency
Create RESTful endpoints that match your application's naming conventions:
- `/products` instead of `/table_definition`
- `/orders` instead of `/order_table`

### Versioning Support
Implement API versioning:
- `/v1/users`
- `/v2/users`

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
- Include version prefixes for API stability: `/v1/`
- Be descriptive but concise: `/product-categories` not `/pc`

### Route Organization
- Group related routes under common prefixes
- Use consistent patterns across your API
- Document the purpose of each custom route

### Permission Alignment
- Ensure route permissions align with table permissions
- Test custom routes with different user roles
- Consider the security implications of custom paths

## Related Documentation

- **[Custom Handlers](hooks-handlers/custom-handlers.md)** - Writing JavaScript logic for custom endpoints
- **[Hooks System](hooks-handlers/hooks.md)** - Adding validation and notifications
- **[Package Management](hooks-handlers/package-management.md)** - Installing NPM packages for external integrations

## Practical Examples

- **[User Registration Example](../examples/user-registration-example.md)** - Complete walkthrough including route creation, custom handler, and nodemailer integration
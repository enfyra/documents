# Hooks System

Hooks are a powerful feature that allows you to inject custom code at specific points in the API request lifecycle. Instead of creating full [Custom Handlers](./custom-handlers.md), you can use hooks to modify requests and responses with minimal code.

**For complete lifecycle understanding, see [API Lifecycle](../../server/api-lifecycle.md)**

## What are Hooks?

Hooks execute JavaScript code at two key moments:
- **PreHook**: Runs **before** the main operation (before database operations)
- **AfterHook**: Runs **after** the main operation (after database operations)

**Key Advantage**: Hooks share the same context throughout the entire request lifecycle, making them perfect for lightweight modifications without replacing the entire handler.

## When to Use Hooks vs Handlers

### Use Hooks When:
-  **Modify request data**: Transform input before database operations
-  **Add validation**: Check data before processing
-  **Modify query filters**: Change filtering programmatically  
-  **Transform responses**: Format output data after operations
-  **Add logging**: Track operations without changing core logic
-  **Auto-generate fields**: Create slugs, hash passwords, set timestamps

### Use Custom Handlers When:
-  **Complex business logic**: Multi-step operations across multiple tables
-  **Replace entire operation**: Completely different behavior than CRUD
-  **External API calls**: Third-party integrations requiring complex workflows

## The $ctx Context Object

Hooks have access to a shared context object (`$ctx`) that persists throughout the entire API request lifecycle. This context contains request data, database repositories, helper functions, and user information.

**For complete $ctx documentation and lifecycle details, see [API Lifecycle](../../server/api-lifecycle.md#context-sharing-ctx)**

### Key Context Properties Available in Hooks

```javascript
$ctx = {
  // Request data
  $body: {},           // Request body (can be modified)
  $params: {},         // URL parameters (/users/:id)
  $query: {},          // Query parameters (?limit=10)
  $user: {},           // Current authenticated user

  // Database access
  $repos: {            // Auto-generated repositories
    main: repository,  // Main table repository
    users: repository, // Named repositories for target tables
  },

  // NPM packages (installed via Package Management)
  $pkgs: {             // Access to installed NPM packages
    axios: axios,      // HTTP client library
    lodash: lodash,    // Utility functions
    moment: moment,    // Date manipulation
  },
  
  // Utilities  
  $helpers: {          // Helper functions
    $jwt: function,    // Create JWT tokens
    $bcrypt: object,   // Hash/compare passwords
    autoSlug: function // Generate URL slugs
  },
  
  // Logging & sharing
  $logs: function,     // Add logs to response
  $share: {            // Share data between hooks
    $logs: []
  }
};
```

**Important:** The `$ctx` object is the **same reference** across all hooks and handlers in a request. Changes made in preHooks affect afterHooks and handlers.

## Creating Hooks

### Step 1: Access Hooks Management
1. Navigate to **Settings  Hooks** in the sidebar
2. Click **"Create New Hook"** button

### Step 2: Configure Hook
You'll see the hook creation form with these fields:

- **Name**: Descriptive name for the hook
- **PreHook**: JavaScript code that runs before the operation
- **AfterHook**: JavaScript code that runs after the operation  
- **Priority**: Execution order (0 = highest priority)
- **IsEnabled**: Toggle to activate/deactivate the hook
- **Description**: Documentation for the hook's purpose
- **Route**: Click the relation picker to select which route this applies to
- **Methods**: Click the relation picker to select HTTP methods (GET, POST, PATCH, DELETE)

### Step 3: Link to Route and Methods
- **Route Selection**: Use the relation picker to search and select the target route
- **Method Selection**: Use the relation picker to choose which HTTP methods trigger this hook
- **Multiple Methods**: A single hook can apply to multiple HTTP methods on the same route

## Writing Hook Code

### Using NPM Packages in Hooks

Hooks have full access to NPM packages installed via [Package Management](./package-management.md):

```javascript
// PreHook example - Using lodash to validate data
const _ = $ctx.$pkgs.lodash;

if (!_.isObject($ctx.$body) || _.isEmpty($ctx.$body)) {
  throw new Error('Request body must be a non-empty object');
}

// AfterHook example - Using moment for timestamps
const moment = $ctx.$pkgs.moment;

$ctx.$response.data.forEach(record => {
  record.formatted_date = moment(record.created_at).format('YYYY-MM-DD HH:mm:ss');
});
```

**Complete Package Guide**: See [Package Management](./package-management.md) for installing and using NPM packages.

### Advanced Hook Development

For detailed information about writing JavaScript code within hooks, including the context object (`$ctx`), available functions, and comprehensive examples, see:

**[Hook Development Guide](../../server/hook-development.md)**

This covers:
- Hook context and available variables
- PreHook and AfterHook code examples
- Database access and utility functions
- Execution flow and priority system
- Best practices and debugging techniques

## Practical Examples

**[User Registration Example](../../examples/user-registration-example.md)** - See hooks in action with welcome email AfterHook using nodemailer package.

Hooks provide the perfect balance between simplicity and power, allowing you to customize API behavior without the complexity of full custom handlers.


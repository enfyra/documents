# Bootstrap Scripts System

> **📝 Template Syntax Note**: All examples use the traditional `$ctx.$property` syntax, but you can also use the shorter template syntax (`@CACHE`, `@REPOS`, `#table_name`). See [Template Syntax Guide](./template-syntax.md) for details. Both work identically and can be mixed freely.

## Overview

The Bootstrap Scripts system allows you to execute custom JavaScript code during application startup. These scripts run once when the application starts and have access to the full dynamic context, including database repositories, cache, and helper functions.

## Features

- **Startup Execution**: Scripts run automatically when the application starts
- **Priority-based**: Scripts execute in priority order (lower numbers first)
- **Timeout Support**: Configurable timeout for each script
- **Full Context Access**: Access to all database tables, cache, and helpers
- **Error Handling**: Failed scripts don't stop other scripts from executing
- **Hot Reload**: Scripts can be reloaded without restarting the application

## Database Schema

### bootstrap_script_definition Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | int | Primary key (auto-generated) |
| `name` | varchar | Unique name identifier for the script |
| `description` | text | Description of what the script does |
| `logic` | code | The actual JavaScript code to execute |
| `timeout` | int | Timeout in milliseconds (optional, default: 30000) |
| `priority` | int | Execution priority order (lower numbers execute first) |
| `isEnabled` | boolean | Whether the script is enabled for execution |
| `isSystem` | boolean | Whether this is a system-generated script |

## Creating Bootstrap Scripts

### 1. Via Admin Interface

Navigate to **Settings > Bootstrap Scripts** in the admin interface to create and manage bootstrap scripts.

### 2. Via Database

Insert directly into the `bootstrap_script_definition` table:

```sql
INSERT INTO bootstrap_script_definition (
  name, 
  description, 
  logic, 
  timeout, 
  priority, 
  isEnabled, 
  isSystem
) VALUES (
  'setup_default_data',
  'Initialize default application data',
  '// Your JavaScript code here',
  30000,
  0,
  true,
  false
);
```

## Script Context ($ctx)

Bootstrap scripts have access to a rich context object (`$ctx`) with the following properties:

### Available Properties

```javascript
$ctx = {
  // Error handling
  $throw: {
    // Throw HTTP errors
    badRequest: (message) => { /* ... */ },
    unauthorized: (message) => { /* ... */ },
    forbidden: (message) => { /* ... */ },
    notFound: (message) => { /* ... */ },
    conflict: (message) => { /* ... */ },
    internalServerError: (message) => { /* ... */ }
  },

  // Logging
  $logs: (...args) => {
    // Log messages to console
  },

  // Helper functions
  $helpers: {
    autoSlug: (text) => {
      // Convert text to URL-friendly slug
      return text.toLowerCase().replace(/\s+/g, '-');
    }
  },

  // Cache operations
  $cache: {
    get: async (key) => { /* ... */ },
    set: async (key, value, ttlMs) => { /* ... */ },
    deleteKey: async (key) => { /* ... */ },
    clearAll: async () => { /* ... */ }
  },

  // Database repositories (all tables)
  $repos: {
    user_definition: { 
      find: async (options?: {where?:any, fields?: string|string[]}) => Promise<{data: any[], meta: any}>,
      create: async (data: any) => Promise<any>,
      update: async (id: string|number, data: any) => Promise<any>,
      delete: async (id: string|number) => Promise<boolean>
    },
    table_definition: { 
      find: async (options?: {where?:any, fields?: string|string[]}) => Promise<{data: any[], meta: any}>,
      create: async (data: any) => Promise<any>,
      update: async (id: string|number, data: any) => Promise<any>,
      delete: async (id: string|number) => Promise<boolean>
    },
    // ... all other tables have the same interface
  },

  // Shared data
  $share: {
    $logs: [] // Array of log messages
  }
}
```

## Code Examples

### 1. Initialize Default Data

```javascript
// Create default admin user if not exists
const existingAdmin = await $ctx.$repos.user_definition.find({
  where: { email: { _eq: 'admin@example.com' } },
  fields: 'id,email,isRootAdmin' // Select only specific fields
});

if (!existingAdmin.data || existingAdmin.data.length === 0) {
  await $ctx.$repos.user_definition.create({
    email: 'admin@example.com',
    password: 'admin123',
    isRootAdmin: true,
    isSystem: true
  });
  
  $ctx.$logs('Default admin user created');
} else {
  $ctx.$logs('Admin user already exists');
}
```

### 2. Setup Cache Configuration

```javascript
// Store application configuration in cache
await $ctx.$cache.set('app_config', {
  version: '1.0.0',
  environment: 'production',
  features: {
    userRegistration: true,
    emailNotifications: true
  }
}, 0); // No expiration

$ctx.$logs('Application configuration cached');
```

### 3. Database Migration

```javascript
// Migrate existing data to new format
const usersResult = await $ctx.$repos.user_definition.find();

for (const user of usersResult.data) {
  if (!user.profile) {
    await $ctx.$repos.user_definition.update(user.id, {
      profile: {
        firstName: user.firstName || '',
        lastName: user.lastName || '',
        avatar: null
      }
    });
  }
}

$ctx.$logs(`Migrated ${usersResult.data.length} user profiles`);
```

### 4. External Service Integration

```javascript
// Initialize external service connections
try {
  const serviceConfig = {
    apiKey: process.env.EXTERNAL_SERVICE_KEY,
    baseUrl: 'https://api.external-service.com',
    timeout: 5000
  };
  
  await $ctx.$cache.set('external_service_config', serviceConfig, 0);
  $ctx.$logs('External service configuration initialized');
} catch (error) {
  $ctx.$logs('Failed to initialize external service:', error.message);
}
```

### 5. Data Validation and Cleanup

```javascript
// Clean up orphaned records
const orphanedResult = await $ctx.$repos.file_definition.find({
  where: { folder: { _is_null: true } },
  fields: 'id,name,folder' // Only fetch required fields
});

if (orphanedResult.data && orphanedResult.data.length > 0) {
  for (const record of orphanedResult.data) {
    await $ctx.$repos.file_definition.delete(record.id);
  }
  
  $ctx.$logs(`Cleaned up ${orphanedResult.data.length} orphaned files`);
}
```

## Best Practices

### 1. Error Handling

Always wrap your code in try-catch blocks:

```javascript
try {
  // Your bootstrap logic here
  $ctx.$logs('Bootstrap script completed successfully');
} catch (error) {
  $ctx.$logs('Bootstrap script failed:', error.message);
  // Don't throw - let other scripts continue
}
```

### 2. Idempotent Operations

Make your scripts idempotent (safe to run multiple times):

```javascript
// Check if data already exists before creating
const existing = await $ctx.$repos.setting_definition.find({
  where: { key: { _eq: 'app_version' } },
  fields: 'id,key' // Only check existence, minimal data
});

if (!existing.data || existing.data.length === 0) {
  await $ctx.$repos.setting_definition.create({
    key: 'app_version',
    value: '1.0.0'
  });
}
```

**Note**: Despite distributed locking, always write idempotent scripts for safety.

### 3. Use Appropriate Timeouts

Set reasonable timeouts for your scripts:

```javascript
// For simple operations
timeout: 5000  // 5 seconds

// For complex operations
timeout: 30000 // 30 seconds

// For data migrations
timeout: 60000 // 60 seconds
```

### 4. Logging

Use `$ctx.$logs()` for important operations:

```javascript
$ctx.$logs('Starting data migration...');
// ... migration logic
$ctx.$logs('Data migration completed successfully');
```

## Multi-Instance Support

### 🔒 Distributed Locking

Bootstrap scripts use **distributed locking** to ensure only one instance executes them:

1. **Lock Acquisition**: First instance acquires `bootstrap-script-execution` lock
2. **Execution**: Only the locked instance runs bootstrap scripts  
3. **Lock Release**: Lock is automatically released after completion or timeout (30s)
4. **Skip Mode**: Other instances skip execution and log the skip

### 📋 Instance Behavior

```bash
# Instance 1 (Got the lock)
🔒 Bootstrap scripts acquired lock - starting execution (uuid-1)
📋 Found 3 bootstrap scripts to execute
🔄 Executing script: setup_default_data (priority: 0)
✅ Bootstrap scripts execution completed
🔓 Bootstrap scripts lock released (uuid-1)

# Instance 2 (Same time)
🔴 Bootstrap scripts skipped - another instance is executing (uuid-2)
```

## Hot Reload

When you update a bootstrap script in the database, the system automatically:

1. **Clears all Redis cache** - Only on single instance  
2. **Distributed lock** - Only one instance acquires reload lock
3. **Reloads bootstrap scripts** - Only the locked instance executes
4. **Other instances skip** - No duplicate execution

### 🔒 Reload Lock Mechanism

```bash
# Instance A (Gets reload lock)
🔒 Bootstrap reload acquired lock - starting execution (uuid-A)
🧹 Cleared all Redis data
🔄 Executing script: updated_script (priority: 0) 
✅ BootstrapScriptService reload completed successfully
🔓 Bootstrap reload lock released (uuid-A)

# Instance B (Same reload event)  
🔴 Bootstrap reload skipped - another instance is executing (uuid-B)
```

This ensures **single execution** during reload operations, preventing data conflicts and race conditions.

## Troubleshooting

### Common Issues

1. **Script Timeout**: Increase the timeout value if your script takes longer to execute
2. **Database Errors**: Check table names and column names in your script
3. **Cache Issues**: Use `$ctx.$cache.clearAll()` if you need to reset cache state
4. **Permission Errors**: Ensure the script has access to required tables

### Debugging

Use `$ctx.$logs()` to debug your scripts:

```javascript
$ctx.$logs('Debug: Starting script execution');
const usersResult = await $ctx.$repos.user_definition.find({
  fields: 'id,name,email' // Limit fields for better performance
});
$ctx.$logs('.', usersResult.data.length, 'users');
$ctx.$logs('Debug: Script completed');
```

### Monitoring

Check application logs for bootstrap script execution:

```
[BootstrapScriptService] 🔄 Executing script: setup_default_data (priority: 0)
[BootstrapScriptService] ✅ Script setup_default_data completed successfully
```

## Security Considerations

- Bootstrap scripts run with full system privileges
- Validate all input data before processing
- Use parameterized queries when possible
- Don't expose sensitive information in logs
- Test scripts in development before deploying to production

## Performance Tips

- Use batch operations for large data sets
- Set appropriate timeouts
- Use database transactions for related operations
- Cache frequently accessed data
- Avoid blocking operations in scripts

## Integration with Other Systems

Bootstrap scripts can integrate with:

- **Cache System**: Store configuration and temporary data
- **Database**: Initialize data and perform migrations
- **External APIs**: Setup service connections
- **File System**: Initialize file structures
- **Environment Variables**: Access system configuration

## Examples for Common Use Cases

### Application Initialization

```javascript
// Initialize application with default settings
const defaultSettings = {
  siteName: 'My Application',
  siteDescription: 'A powerful application',
  maintenanceMode: false,
  userRegistration: true
};

for (const [key, value] of Object.entries(defaultSettings)) {
  const existing = await $ctx.$repos.setting_definition.find({
    where: { key: { _eq: key } }
  });
  
  if (!existing.data || existing.data.length === 0) {
    await $ctx.$repos.setting_definition.create({ key, value });
  }
}

$ctx.$logs('Application settings initialized');
```

### User Management Setup

```javascript
// Create default roles and permissions
const defaultRoles = [
  { name: 'admin', description: 'Administrator role' },
  { name: 'user', description: 'Regular user role' },
  { name: 'guest', description: 'Guest user role' }
];

for (const role of defaultRoles) {
  const existing = await $ctx.$repos.role_definition.find({
    where: { name: { _eq: role.name } }
  });
  
  if (!existing.data || existing.data.length === 0) {
    await $ctx.$repos.role_definition.create(role);
  }
}

$ctx.$logs('Default roles created');
```

### Data Seeding

```javascript
// Seed initial data for development
if (process.env.NODE_ENV === 'development') {
  const sampleUsers = [
    { email: 'user1@example.com', password: 'password123' },
    { email: 'user2@example.com', password: 'password123' }
  ];
  
  for (const user of sampleUsers) {
    const existing = await $ctx.$repos.user_definition.find({
      where: { email: { _eq: user.email } }
    });
    
    if (!existing.data || existing.data.length === 0) {
      await $ctx.$repos.user_definition.create(user);
    }
  }
  
  $ctx.$logs('Sample users created for development');
}
```

This bootstrap script system provides a powerful way to initialize and maintain your application state, ensuring consistency across deployments and environments.

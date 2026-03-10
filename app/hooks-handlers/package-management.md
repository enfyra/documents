# Package Management

Package Management lets you install NPM packages directly from the Enfyra interface and use them in your custom handlers and hooks. Instead of managing dependencies manually, you can search, install, and configure packages through the UI - they're automatically available in your custom code as `$ctx.$pkgs.packagename`.

**Related Documentation**: [Custom Handlers](./custom-handlers.md) | [Hooks and Handlers](../../server/hooks-handlers/) | [User Registration Example](../../examples/user-registration-example.md)

## Why Use Package Management?

- **Extend Functionality**: Add powerful libraries like axios, lodash, moment.js to your handlers
- **No Configuration**: Packages are instantly available in your custom code without setup
- **Visual Interface**: Search and install packages without touching package.json
- **Isolated Environment**: Packages run safely in your custom handler execution context

## Installing a Package

### Step 1: Access Package Installation
1. Navigate to **Packages** in the sidebar
2. Click **Install Package** button

### Step 2: Choose Package Type
You'll see two package type options:
- **Server Package**: For use in custom handlers and hooks (server-side)
- **App Package**: For use in extensions (pages, widgets) in the browser

Select the appropriate type for your use case.

### Step 3: Search NPM Packages
Start typing in the search field to find packages:
- **Real-time search** with auto-suggestions as you type
- **Rich package information** including descriptions and version numbers
- **Version badges** showing the latest available version
- **Package icons** for visual identification

**Popular package examples:**
```
axios          # HTTP requests and API calls
lodash         # Utility functions for data manipulation
moment         # Date and time handling
uuid           # Generate unique identifiers
joi            # Data validation schemas
```

### Step 4: Select and Configure
1. **Click on a package** from the dropdown results
2. **Form auto-fills** with package name, version, and description
3. **Customize details** if needed (description, flags, etc.)
4. **Click Install** to add the package to your project

**Important**: The package name field is automatically populated and cannot be edited - this ensures correct package resolution.

## Using Installed Packages

Once installed, packages are automatically available in your custom handlers and hooks through the `$ctx.$pkgs` object. No imports needed - just access them directly.

### Basic Usage Pattern
```javascript
// In any custom handler or hook
export default async function handler({ $ctx }) {
  // Access installed packages via $ctx.$pkgs
  const packageName = $ctx.$pkgs.packagename;

  // Use the package normally
  return packageName.someMethod();
}
```

### Real-World Examples

**HTTP Requests with Axios**
```javascript
// Custom handler for external API integration
export default async function handler({ $ctx }) {
  const axios = $ctx.$pkgs.axios;

  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/users');
    return {
      success: true,
      users: response.data
    };
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}
```

**Data Processing with Lodash**
```javascript
// Handler for complex data transformation
export default async function handler({ $ctx }) {
  const _ = $ctx.$pkgs.lodash;

  // Simulate processing user data
  const users = $ctx.$body.users || [];

  // Group users by department and get counts
  const grouped = _.groupBy(users, 'department');
  const summary = _.mapValues(grouped, group => ({
    count: group.length,
    active: _.filter(group, 'isActive').length
  }));

  return {
    departmentSummary: summary,
    totalUsers: users.length
  };
}
```

**Date Operations with Moment**
```javascript
// Handler for date-based operations
export default async function handler({ $ctx }) {
  const moment = $ctx.$pkgs.moment;

  const startDate = moment($ctx.$query.startDate);
  const endDate = moment($ctx.$query.endDate);

  return {
    period: {
      start: startDate.format('YYYY-MM-DD'),
      end: endDate.format('YYYY-MM-DD'),
      days: endDate.diff(startDate, 'days')
    },
    timestamps: {
      created: moment().unix(),
      formatted: moment().format('YYYY-MM-DD HH:mm:ss')
    }
  };
}
```

## Managing Installed Packages

### Viewing All Packages
1. Navigate to **Packages** in the sidebar
2. Click **Server Packages** to view all installed server packages

**Package card information:**
- **Package name** with npm icon
- **Current version** badge
- **Description** and installation date
- **Quick actions** (view details, uninstall)

### Updating Package Information
1. **Click on any package card** to open the detail view
2. **Modify information** like description or flags
3. **Click Save** to update the package metadata

> **Note**: Package name is locked after installation to prevent breaking existing code references.

### Updating Package Version
To update a package to a specific version:

1. **Navigate to the package detail page**
2. **Edit the version field** in the form to specify the desired version
3. **Click Save** to install the specified version
4. **Verify the update** - the version badge will show the new version number

**Example version update flow:**
```
Current: lodash v4.17.20
↓ Edit version field to "4.17.21"
↓ Click Save
Updated: lodash v4.17.21
```

**Version Update Notes:**
- **Manual specification**: You must enter the exact version number you want
- **Version validation**: Ensure the version exists on NPM before updating
- **Backward compatibility**: Test your handlers after version updates, especially major version changes
- **Check NPM**: Visit [npmjs.com/package/packagename](https://npmjs.com) to see available versions

### Removing Packages
1. **Open package details** by clicking on the package card
2. **Click Uninstall** button in the header (red trash icon)
3. **Confirm removal** when prompted

**Warning**: Uninstalling removes the package from all handlers and hooks immediately. Make sure no active code depends on it first.

## Common Package Use Cases

### Essential Packages for Most Projects

**HTTP & API Integration**
```javascript
// axios - HTTP requests
$ctx.$pkgs.axios.get('https://api.example.com/data')

// node-fetch - Alternative HTTP client
$ctx.$pkgs.fetch('https://api.example.com')
```

**Data Manipulation**
```javascript
// lodash - Comprehensive utilities
$ctx.$pkgs.lodash.groupBy(data, 'category')

// ramda - Functional programming
$ctx.$pkgs.ramda.filter(R.propEq('active', true))
```

**Date & Time Processing**
```javascript
// moment - Full-featured date library
$ctx.$pkgs.moment().format('YYYY-MM-DD')

// dayjs - Lightweight alternative
$ctx.$pkgs.dayjs().add(1, 'month')
```

**Validation & Security**
```javascript
// joi - Schema validation
$ctx.$pkgs.joi.object({ email: joi.string().email() })

// bcrypt - Password hashing
$ctx.$pkgs.bcrypt.hash(password, 10)
```

**Utilities**
```javascript
// uuid - Generate unique IDs
$ctx.$pkgs.uuid.v4()

// slugify - URL-friendly strings
$ctx.$pkgs.slugify('Hello World', { lower: true })
```

## Troubleshooting

### Package Not Found
```javascript
//  Wrong - package not installed
const axios = $ctx.$pkgs.axios; // undefined

//  Correct - install package first via UI
// Then access as shown above
```

**Solutions:**
1. Verify package is installed in **Packages  Server Packages**
2. Check package name spelling (case-sensitive)
3. Ensure you're using `$ctx.$pkgs.exactname` syntax

### Package Search Issues
- **Type at least 2 characters** before search activates
- **Try different keywords** if no results appear
- **Check internet connection** for external NPM queries

### Installation Problems
- **Verify package exists** on [npmjs.com](https://npmjs.com)
- **Try clearing browser cache** and refreshing
- **Check package name spelling** in search

### Version Update Issues
- **Check package compatibility** before updating major versions
- **Test handlers/hooks** after version updates to ensure compatibility
- **Revert if needed** by uninstalling and reinstalling the previous version

## Package Recommendations

**Starter Pack** - Essential packages for most projects:
- `axios` - HTTP requests and API integration
- `lodash` - Data manipulation and utilities
- `moment` - Date/time operations
- `uuid` - Generate unique identifiers
- `joi` - Input validation

**Data Processing Pack**:
- `csv-parser` - Parse CSV files
- `fast-xml-parser` - XML parsing
- `jsonwebtoken` - JWT token handling
- `bcrypt` - Password encryption

**Integration Pack**:
- `nodemailer` - Send emails
- `twilio` - SMS and communication
- `stripe` - Payment processing
- `aws-sdk` - Amazon Web Services

**Start with the Starter Pack packages** - they cover 80% of common backend needs and work well together.

## Best Practices

### Package Management
- **Check for updates**: Regularly check NPM for newer versions of your installed packages
- **Manual version control**: Specify exact versions when updating to maintain consistency
- **Test after updates**: Always test your handlers and hooks after version updates
- **Clean unused packages**: Remove packages that are no longer used to keep the system lean

### Version Strategy
- **Controlled updates**: Manually specify versions rather than auto-updating to avoid breaking changes
- **Research versions**: Check package release notes and changelogs before updating
- **Test in staging**: For production systems, test version updates in a staging environment first
- **Pin important versions**: Keep stable versions for critical functionality
- **Document dependencies**: Keep track of which handlers/hooks use which packages and versions

---

## Using App Packages in Extensions

App Packages let you use npm packages directly in your Extensions (custom pages and widgets). Unlike Backend Packages which run on the server, App Packages run in the browser and are perfect for:
- Charts and visualizations (chart.js, echarts)
- Utilities (lodash, dayjs, axios)
- UI components
- Data processing libraries

### Installing an App Package

1. Navigate to **Packages** in the sidebar
2. Click **Install Package**
3. Select **App Package** as the type
4. Search and select your package
5. Click **Install**

### Using Packages in Extensions

Once installed, packages are available through the `getPackages()` function:

**Method 1: Destructuring (Recommended)**
```vue
<script setup>
onMounted(async () => {
  const { chartjs, dayjs } = await getPackages();

  const formattedDate = dayjs().format('YYYY-MM-DD');
  console.log('Date:', formattedDate);
});
</script>
```

**Method 2: Array of Packages**
```vue
<script setup>
onMounted(async () => {
  const packages = await getPackages(['chartjs', 'dayjs']);

  const chart = packages.chartjs;
  const date = packages.dayjs().format('YYYY-MM-DD');
});
</script>
```

**Method 3: All Packages**
```vue
<script setup>
onMounted(async () => {
  const packages = await getPackages();

  const chart = packages.chartjs;
  const date = packages.dayjs().format('YYYY-MM-DD');
});
</script>
```

### Complete Example: Chart in Extension

```vue
<template>
  <UCard>
    <template #header>
      <h3>Sales Chart</h3>
    </template>

    <canvas ref="chartCanvas"></canvas>
  </UCard>
</template>

<script setup>
const chartCanvas = ref(null);

onMounted(async () => {
  const { Chart } = await getPackages(['chart.js']);

  const ctx = chartCanvas.value.getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: ['Jan', 'Feb', 'Mar'],
      datasets: [{
        label: 'Sales',
        data: [12, 19, 3],
        backgroundColor: 'rgba(59, 130, 246, 0.5)'
      }]
    }
  });
});
</script>
```

### Popular App Packages

**Charts & Visualization:**
- `chart.js` - Charts and graphs
- `vue-chartjs` - Vue wrapper for Chart.js
- `echarts` - Enterprise charts
- `apexcharts` - Modern charting library

**Utilities:**
- `dayjs` - Date manipulation (lightweight)
- `lodash` - Utility functions
- `axios` - HTTP requests
- `uuid` - Generate unique IDs

**Data & Forms:**
- `vuedraggable` - Drag and drop
- `sortablejs` - Sortable lists
- `papaparse` - CSV parsing

### How It Works

**Automatic Detection:**
The system automatically detects which packages you use by scanning your code:
- Scans for `getPackages()` calls
- Identifies package names from destructuring
- Only loads packages you actually use

**Lazy Loading:**
- Packages load when first accessed
- Subsequent calls use cached version
- Faster initial page load

**Dependency Management:**
- Dependencies load automatically
- Correct order guaranteed
- No manual setup needed

### Best Practices

1. **Specify packages explicitly** - Better performance than loading all
   ```javascript
   onMounted(async () => {
     const { chartjs } = await getPackages();
   });
   ```

2. **Load in onMounted** - Ensure component is ready
   ```javascript
   onMounted(async () => {
     const packages = await getPackages();
   });
   ```

3. **Check package availability**
   ```javascript
   const packages = await getPackages(['chartjs']);
   if (!packages.chartjs) {
     toast.add({
       title: 'Package not installed',
       description: 'Install chart.js from Packages',
       color: 'red'
     });
   }
   ```

### Troubleshooting

**Package not loading:**
- Verify package is installed as **App Package** (not Backend)
- Check browser console for errors
- Ensure package name matches installation

**Multiple versions:**
- Only one version per package name
- Update version in package settings if needed

**Performance:**
- Packages are cached after first load
- Use specific packages for better performance

**For more examples, see [Extension System](../extension-system.md#using-npm-packages-in-extensions)**


# Getting Started

After completing the installation of both Enfyra backend and app, follow these steps to get started.

## First Login

1. Navigate to your Enfyra app (default: `http://localhost:3000`)
2. You'll be redirected to the login page
3. Use the admin account created during backend setup:
   - The credentials you provided when setting up the backend
   - If you used default setup, check your backend console for the admin credentials

## After Login

Once logged in, you'll see the main Enfyra interface with several key components:

### Interface Layout

**Left Side:**
- **Mini Sidebar** - 16px wide icon bar with toggle button and navigation shortcuts
- **Expandable Sidebar** - Contains full navigation menu (visible on desktop by default, toggle on mobile)

**Main Area:**
- **Header** - Top section with page title and action buttons
- **Sub-Header** - Secondary navigation and breadcrumbs ("Home > settings > routings")
- **Content Area** - Main page content with gradient background and subtle patterns

**Header Actions:**
- Located in top-right corner of each page
- Context-specific buttons (Filter, Create, Save, Delete, etc.)
- Green "Create" buttons for adding new items

### Sidebar Navigation

- **Dashboard** (Grid icon) - Overview and quick stats of your system
- **Data** (List icon) - Browse, create, edit and delete records in your tables
- **Collections** (Database icon) - Create and manage database tables/schemas
- **Settings** (Gear icon) - System configuration, users, roles, and permissions
- **Files Management** (Folder icon) - Upload and manage media files and documents

**Sidebar Behavior:**
- Click items to expand submenus on the right side
- Desktop: Sidebar visible by default
- Mobile/Tablet: Collapsed by default, overlay when opened
- Toggle button in mini sidebar to show/hide full menu

## Next Steps: Create Your First Table

Now that you're familiar with the interface, it's time to create your first table and start building your application.

**→ [Table Creation Guide](./table-creation.md)** - Complete step-by-step guide to creating tables with all field types, relations, and constraints.

This comprehensive guide covers:
- **Table creation workflow** - Step-by-step process
- **All field types** - From basic text to rich content and relations
- **Advanced features** - Constraints, indexes, and relationships
- **What happens after creation** - Automatic API generation and integration

After creating your table, you'll have:
- **4 automatic CRUD endpoints** on your backend server
- **Frontend integration** in the Data section
- **Full API access** for external applications

**Then continue with:**
- **[Data Management](./data-management.md)** - Learn to add, edit, and manage records in your tables

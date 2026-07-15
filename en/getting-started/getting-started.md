# Getting Started

After completing the installation of both Enfyra backend and app, follow these steps to get started.

> **New to Enfyra?** Start with the [Installation Guide](./installation.md) to set up your app + server workspace.

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
- **Sidebar** - Contains full navigation menu with menus and dropdowns (visible on desktop by default, toggle on mobile)
- **Collapse Toggle** - Button to show/hide the sidebar (collapsed shows icon-only view)

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
- **Storage** (Folder icon) - Upload and manage media files, folders, and storage configurations

**Sidebar Behavior:**
- Click menu items to navigate or expand/collapse dropdown menus
- Desktop: Sidebar visible by default, can be collapsed to icon-only view
- Mobile/Tablet: Collapsed by default, overlay when opened
- Toggle button to show/hide full sidebar menu

## Next Step: Build A Working First App

Do not try to learn every collection option before seeing a result. Follow the short guided path to create a collection, enter a record, and call its generated API.

**[Build Your First App](./first-app.md)** - The recommended next step for a new Enfyra user.

This comprehensive guide covers:
- **Table creation workflow** - Step-by-step process
- **All field types** - From basic text to rich content and relations
- **Advanced features** - Constraints, indexes, and relationships
- **What happens after creation** - Automatic API generation and integration

After completing it, you'll have:
- **4 automatic CRUD endpoints** on your backend server
- **Frontend integration** in the Data section
- **Full API access** for external applications

**Then continue with:**
- **[Table Creation](./table-creation.md)** - All field types, relations, indexes, and constraints.
- **[Data Management](./data-management.md)** - Learn to add, edit, and manage records in your tables

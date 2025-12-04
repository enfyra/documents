# Enfyra App (Frontend) Documentation

This section covers everything that runs in the Enfyra **admin app (port 3000)** – the UI, extensions, forms, permissions, and how the app talks to the backend server.

The app is a **pure API client**: it never connects directly to your database. All data comes from the **Enfyra Server (port 1105)** via HTTP APIs.

## Quick Navigation

- **API Integration**  
  - **[API Integration](./api-integration.md)** – How the app calls backend APIs using the official Enfyra SDKs (`@enfyra/sdk-nuxt`, `@enfyra/sdk-next`), `useApi`, and `useEnfyraApi`

- **Extensions & Widgets**  
  - **[Extension System](./extension-system.md)** – Create custom pages and widgets with Vue components  
  - **[Header Actions](./header-actions.md)** – Inject custom buttons into the app header and sub-header

- **Forms & Data Entry**  
  - **[Form System](./form-system.md)** – Auto-generated forms, validation, relations, and change tracking  
  - **[Relation Picker](./relation-picker.md)** – Selecting related records inside forms  
  - **[Filter System](./filter-system.md)** – Advanced filtering for tables and pickers

- **Permissions & Visibility**  
  - **[Permission Builder](./permission-builder.md)** – Visual permission rule builder  
  - **[Permission Components](./permission-components.md)** – `PermissionGate` component and `usePermissions` composable

- **Navigation & Layout**  
  - **[Menu Management](./menu-management.md)** – Sidebar and menu configuration  
  - **[Page Header](./page-header.md)** – Page headers with stats and gradients

- **Hooks, Handlers & Packages (UI side)**  
  - **[Hooks](./hooks.md)** – Managing hooks from the app UI  
  - **[Custom Handlers](./custom-handlers.md)** – Managing custom handlers from the app UI  
  - **[Package Management](./package-management.md)** – Installing NPM packages for hooks and handlers

- **Storage & Files**  
  - **[Storage Management](./storage-management.md)** – File uploads, folders, and storage configurations

## Frontend Learning Path

If you are **building on top of the Enfyra App UI**, this is a suggested order:

1. **Understand the architecture**  
   - **[Architecture Overview](../architecture-overview.md)** – How frontend and backend work together  
   - **[Server Overview](../server/README.md)** – Server-side concepts and APIs

2. **Work with data from the app**  
   - **[Form System](./form-system.md)** – How forms are generated from tables  
   - **[Filter System](./filter-system.md)** – Filtering data in tables and relation pickers  
   - **[API Integration](./api-integration.md)** – Calling APIs from pages, extensions, and widgets

3. **Build custom UI and workflows**  
   - **[Extension System](./extension-system.md)** – Custom pages and widgets  
   - **[Header Actions](./header-actions.md)** – Custom header actions  
   - **[Menu Management](./menu-management.md)** – Connecting extensions to menus

4. **Secure and control the UI**  
   - **[Permission Builder](./permission-builder.md)** – Define permission rules  
   - **[Permission Components](./permission-components.md)** – Hide/show UI based on permissions  
   - **[Server Permission System](../server/permission-system.md)** – Backend permission evaluation

5. **Go deeper into backend customization (from the app UI)**  
   - **[Hooks](./hooks.md)** – Configure hooks that run on the server  
   - **[Custom Handlers](./custom-handlers.md)** – Override default CRUD behavior  
   - **[Package Management](./package-management.md)** – Install NPM packages for hooks/handlers  
   - **[Server Hooks & Handlers](../server/hooks-handlers/README.md)** – Full server-side behavior

## How This Relates to the Server Docs

- The **Server docs** (`../server/README.md`) describe **what APIs exist and how they behave**.  
- The **App docs** (this folder) describe **how the admin app uses those APIs** through forms, tables, extensions, and permissions.

When in doubt:

- If you are asking **“Which endpoint / field / filter can I use?”** – go to the **server docs**.  
- If you are asking **“How do I show this in the UI / extension?”** – stay in the **app docs**.


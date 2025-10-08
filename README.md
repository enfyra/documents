# Enfyra Documentation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Nuxt](https://img.shields.io/badge/Nuxt-4-green.svg)](https://nuxt.com/)
[![Vue](https://img.shields.io/badge/Vue-3-green.svg)](https://vuejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue.svg)](https://www.typescriptlang.org/)

## 🚀 What is Enfyra?

**Enfyra is a backend framework that automatically generates APIs from your database.** You create tables through a visual interface, and Enfyra instantly provides REST & GraphQL APIs for them - no coding required. It's like having a backend developer that never sleeps.

🔗 **Live Demo**: [https://demo.enfyra.io/](https://demo.enfyra.io/) - Pre-filled admin credentials, just click login!

**What can you build with Enfyra?**

- **E-commerce platforms** - Products, orders, customers, inventory
- **Content websites** - Blogs, news sites, documentation portals
- **Business applications** - CRM, project management, internal tools
- **Mobile app backends** - User management, data sync, push notifications
- **Any data-driven system** - If it needs a database and API, Enfyra can power it

### 💪 No-Code Simplicity, Full-Code Power

Unlike typical no-code platforms that limit you to predefined features, **Enfyra gives you complete control over every aspect of your API lifecycle**:

- **Before Hooks**: Validate, transform, or enrich data before it hits the database
- **Custom Handlers**: Replace default CRUD operations with your own business logic
- **After Hooks**: Trigger workflows, send notifications, or sync with external services
- **Dynamic Extensions**: Build custom UI components that integrate seamlessly with the admin panel
- **Query Interceptors**: Modify queries, add custom filters, or implement complex access rules

**The result?** Start building in minutes with no-code, but never hit a ceiling when your requirements grow complex. Every API endpoint can be customized, every workflow can be automated, and every business rule can be implemented - all without forking or modifying the core system.

### ⚡ Key Differentiators

**🔥 Backend-First Architecture**

- **API Generation**: All REST & GraphQL APIs are automatically generated and served by the backend server (port 1105)
- **Frontend as Client**: The frontend app (port 3000) is purely a client that consumes APIs from your backend URL
- **Single Source of Truth**: Backend generates APIs from database schema, frontend consumes them via HTTP requests

**🔥 Real-time Everything**

- **Live Extension System**: Write Vue/JavaScript extensions that compile and load instantly from the database - no server restarts, no deployments
- **Zero-downtime Schema Updates**: Change your data structure while your API stays 100% available
- **Instant API Generation**: Every table immediately becomes a full REST & GraphQL API with advanced querying

**🏗️ Cluster-Ready Architecture**

- **Born for Clustering**: Designed from day one to run multiple instances - no configuration, no setup, just scale
- **Automatic Coordination**: Schema changes sync instantly across all instances via Redis
- **Zero Additional Work**: Deploy 1 instance or 100 instances - the experience is identical
- **Stateless by Design**: Any instance handles any request seamlessly - [Learn how it works →](./server/cluster-architecture.md)

**🛡️ Enterprise-Grade Security**

- **Visual Permission Builder**: Create complex permission logic with AND/OR conditions and nested rules
- **Dynamic Role System**: Permissions that adapt based on data relationships and user context
- **Handler Isolation**: Custom code runs in isolated processes for security and stability

**🚀 Beyond Traditional CMS**

- **Meta-Programming Core**: The entire API structure is generated from database metadata in real-time
- **Multi-Instance Coordination**: Run multiple instances with automatic schema synchronization via Redis
- **Smart Caching**: SWR (Stale-While-Revalidate) pattern for optimal performance without sacrificing freshness
- **Flexible Syntax**: Choose between traditional `$ctx.$property` or modern template syntax `@TEMPLATE` & `#table_name`
- **Built-in Tables**: Leverage system tables like `user_definition` for immediate user management

### ⚡ Modern Template Syntax

Choose your coding style - both work seamlessly together:

**Traditional Syntax:**
```javascript
const user = await $ctx.$repos.user_definition.create({
  email: $ctx.$body.email,
  password: await $ctx.$helpers.$bcrypt.hash($ctx.$body.password)
});
```

**Template Syntax (Shortened):**
```javascript
const user = await #user_definition.create({
  email: @BODY.email,
  password: await @HELPERS.$bcrypt.hash(@BODY.password)
});
```

**See**: [Complete Example](./examples/user-registration-example.md) featuring both syntax styles!

### 🎯 Core Capabilities

| Feature               | How It Works                                                            |
| --------------------- | ----------------------------------------------------------------------- |
| **Extension System**  | Write custom features that compile and load instantly from the database |
| **Schema Changes**    | Modify your data structure with zero downtime - APIs stay available     |
| **Permission System** | Visual builder for complex access control                               |
| **API Generation**    | Every table instantly becomes a full REST & GraphQL API                 |
| **Custom Code**       | Execute business logic in isolated processes with full request context  |
| **Multi-Instance**    | Run multiple servers with automatic synchronization                     |
| **Flexible Syntax**   | Traditional `$ctx.$property` or modern `@TEMPLATE` & `#table_name` patterns |
| **Package Management**| Install NPM packages directly from UI for use in handlers and hooks     |
| **Built-in Auth**     | System tables like `user_definition` for immediate user management     |

### 💡 Perfect For

- **Rapid Development**: Go from idea to production API in minutes, not days
- **Complex Projects**: Handle sophisticated data relationships and business logic without limitations
- **Scale-Ready Applications**: Start small and scale to enterprise without architectural changes
- **Team Collaboration**: Intuitive interface for non-technical users, powerful tools for developers
- **Custom Solutions**: Build exactly what you need without fighting framework limitations

### 🏗️ Architecture Overview

**Two-Component System:**
```
Database → Backend (API Server) → Frontend (Admin App)
```

- **Backend (Port 1105)**: Generates & serves all REST & GraphQL APIs from your database schema
- **Frontend (Port 3000)**: Pure client application consuming APIs from backend URL
- **All API endpoints**: Originate from backend server, frontend makes HTTP requests

### 🏗️ Built With Modern Technology

**Backend**: NestJS + Express.js + TypeORM + Redis + GraphQL Yoga
**Frontend**: Nuxt 4 + Vue 3 + TypeScript + TailwindCSS
**Database**: MySQL, MariaDB, PostgreSQL (your choice)
**Extensions**: Dynamic Vue SFC compilation via Vite

#### Technology Stack Details

**NestJS + Express.js** - Enterprise Node.js framework built on Express with custom high-performance route engine that bypasses Express middleware stack for superior API performance

**TypeORM** - Database ORM with comprehensive support for:
- **MySQL** - Recommended for production environments
- **MariaDB** - MySQL-compatible with full feature support
- **PostgreSQL** - Advanced features and complex data types supported
- **SQLite** - Planned for future release (development and testing environments)

## Features

### 🏗️ Core Infrastructure
- **Dynamic Table Management** - Create and modify database tables on the fly
- **Package Management** - Install and manage NPM packages directly from the UI for use in handlers and hooks
- **Official SDK** - `useApi` and `useEnfyraAuth` from @enfyra/sdk-nuxt package with custom error handling
- **TypeScript Support** - Full type safety throughout the application

### 🎨 User Interface  
- **Extension System** - Extensible architecture with dynamic extension loading
- **Permission System** - Comprehensive role-based access control (RBAC)
- **Menu Registry** - Dynamic sidebar and menu management
- **Header Actions** - Configurable header button system
- **Dynamic Forms** - Auto-generated forms with validation and relations
- **Advanced Filtering** - Interactive UI filtering with MongoDB-like syntax

### ⚙️ Developer Experience
- **Flexible Code Syntax** - Choose between traditional `$ctx.$property` or modern `@TEMPLATE` & `#table_name`
- **Built-in Authentication** - System tables like `user_definition` for immediate user management
- **Hook System** - PreHook and AfterHook support for custom business logic
- **Custom Handlers** - Override default CRUD operations with your own code
- **Request Context** - Full `$ctx` object with repositories, helpers, packages, and more

## 📚 Documentation

```
enfyra-docs/
├── 🏗️ architecture-overview.md  # System architecture diagram and component responsibilities
├── 🚀 getting-started/
│   ├── installation.md          # Setup guide for backend and app
│   ├── getting-started.md       # First steps after installation and interface overview
│   ├── table-creation.md        # Complete guide to creating tables with all field types
│   └── data-management.md       # Complete guide to managing records in your tables
│
├── 📝 examples/
│   └── user-registration-example.md # Complete end-to-end example with template syntax
│
├── 🎨 frontend/
│   ├── api-integration.md        # API integration with Enfyra SDK and examples for extensions
│   ├── filter-system.md         # Interactive UI filtering for data tables and forms
│   ├── relation-picker.md       # Working with related data in forms (powered by Filter System)
│   ├── routing-management.md    # UI guide for creating custom API endpoints and route permissions
│   ├── custom-handlers.md       # UI guide for creating custom business logic handlers
│   ├── hooks.md                 # UI guide for creating lightweight request/response hooks
│   ├── package-management.md    # Install and manage NPM packages for handlers and hooks
│   ├── menu-management.md       # UI guide for creating custom navigation menus
│   ├── extension-system.md      # Create custom pages with Vue.js components (linked to menus)
│   ├── header-actions.md        # Inject custom actions into header and sub-header areas
│   ├── permission-builder.md    # Visual interface for creating complex permission rules
│   ├── permission-components.md # PermissionGate and usePermissions for UI control
│   └── form-system.md          # Dynamic form generation with validation and relations
│
└── ⚙️ server/
    ├── api-lifecycle.md         # 🔄 Request lifecycle, hook system, and context sharing
    ├── api-querying.md          # 🔥 MongoDB-like API querying with powerful operators (for developers)
    ├── graphql-api.md           # 📊 GraphQL queries and mutations with auto-generated schema
    ├── hook-development.md      # Advanced hook programming with context, examples, and best practices
    ├── context-reference.md     # 📖 Complete $ctx object reference for hooks and handlers
    ├── cluster-architecture.md  # 🏗️ Multi-instance coordination and distributed synchronization
    ├── permission-system.md     # 🔐 Role-based access control with allowedUsers bypass
    ├── template-syntax.md       # ✨ Modern template syntax for cleaner code
    └── bootstrap-scripts.md     # 🚀 Startup script execution system with full context access
```

## 🗺️ Learning Path

**New to Enfyra?** Choose your journey:

### ⚡ Quick Start (5 mins)
**Just want to see what Enfyra can do?**
1. 🌐 **[Try Live Demo](https://demo.enfyra.io/)** - Pre-filled admin credentials, just click login!
2. 📖 **[Getting Started Guide](./getting-started/getting-started.md)** - See table creation and data management in action

### 📚 Full Learning Path
**Ready to master Enfyra?** Follow this step-by-step path to become proficient:

### 🚀 Phase 1: Setup & Basics (30 mins)
1. **[Installation](./getting-started/installation.md)** - Set up Enfyra backend and app
2. **[Getting Started](./getting-started/getting-started.md)** - First login and interface overview
3. **[Table Creation](./getting-started/table-creation.md)** - Create your first table with all field types and relations
4. **[Data Management](./getting-started/data-management.md)** - Learn to manage records and relationships

### 🎨 Phase 2: Frontend Mastery (2-3 hours)
5. **[Form System](./frontend/form-system.md)** - Understand how forms work with your data
6. **[Filter System](./frontend/filter-system.md)** - Master data filtering and searching  
7. **[Permission Builder](./frontend/permission-builder.md)** - Set up access control rules
8. **[Menu Management](./frontend/menu-management.md)** - Customize navigation and user interface

### 🔧 Phase 3: Customization (3-4 hours)
9. **[API Integration](./frontend/api-integration.md)** - Learn to fetch and manipulate data programmatically
10. **[Package Management](./frontend/package-management.md)** - Install NPM packages for enhanced handler and hook functionality
11. **[Extension System](./frontend/extension-system.md)** - Build custom pages and functionality
12. **[Header Actions](./frontend/header-actions.md)** - Inject custom buttons and widgets into the app interface
13. **[Custom Handlers](./frontend/custom-handlers.md)** - Override default API behavior with business logic

### ⚙️ Phase 4: Advanced Development (4-5 hours)
14. **[API Querying](./server/api-querying.md)** - Master MongoDB-like querying for complex data retrieval
15. **[Context Reference](./server/context-reference.md)** - Complete reference for $ctx object in hooks and handlers
16. **[Hook Development](./server/hook-development.md)** - Create sophisticated request/response hooks
17. **[API Lifecycle](./server/api-lifecycle.md)** - Understand the complete request processing pipeline

### 🏗️ Phase 5: Production & Scale (2-3 hours)
18. **[Permission System](./server/permission-system.md)** - Deep dive into role-based access control
19. **[Cluster Architecture](./server/cluster-architecture.md)** - Deploy and scale across multiple instances
20. **[Bootstrap Scripts](./server/bootstrap-scripts.md)** - Initialize application state and perform startup tasks

### 🎯 Goal-Oriented Paths

**Have a specific goal?** Jump directly to what you need:

- 🚀 **Building an MVP?** → Phases 1-2 (4-5 hours total)
- 🔧 **Need custom functionality?** → Focus on Phase 3: API Integration + Extensions
- 📊 **Building a dashboard?** → [Extension System](./frontend/extension-system.md) + [API Integration](./frontend/api-integration.md)
- 🔒 **Need role-based access?** → [Permission Builder](./frontend/permission-builder.md) + [Permission System](./server/permission-system.md)
- ⚙️ **Complex business logic?** → [Custom Handlers](./frontend/custom-handlers.md) + [Context Reference](./server/context-reference.md) + [Hook Development](./server/hook-development.md)
- 🏢 **Enterprise deployment?** → [Cluster Architecture](./server/cluster-architecture.md) + [Permission System](./server/permission-system.md)
- 🚀 **Application initialization?** → [Bootstrap Scripts](./server/bootstrap-scripts.md) + [API Integration](./frontend/api-integration.md)

---

### Quick Navigation

**🏗️ Architecture Overview**
- **[Architecture Overview](./architecture-overview.md)** - System architecture diagram and component responsibilities

**🚀 Getting Started**
- **[Installation](./getting-started/installation.md)** - Setup guide for backend and app
- **[Getting Started](./getting-started/getting-started.md)** - First steps after installation and interface overview
- **[Table Creation](./getting-started/table-creation.md)** - Complete guide to creating tables with all field types
- **[Data Management](./getting-started/data-management.md)** - Complete guide to managing records in your tables

**🎨 Frontend (User Interface)**
- **[API Integration](./frontend/api-integration.md)** - API integration with Enfyra SDK and examples for extensions
- **[Filter System](./frontend/filter-system.md)** - Interactive UI filtering for data tables and forms
- **[Relation Picker](./frontend/relation-picker.md)** - Working with related data in forms (powered by Filter System)
- **[Routing Management](./frontend/routing-management.md)** - UI guide for creating custom API endpoints and route permissions
- **[Custom Handlers](./frontend/custom-handlers.md)** - UI guide for creating custom business logic handlers
- **[Hooks](./frontend/hooks.md)** - UI guide for creating lightweight request/response hooks
- **[Package Management](./frontend/package-management.md)** - Install and manage NPM packages for handlers and hooks
- **[Menu Management](./frontend/menu-management.md)** - UI guide for creating custom navigation menus
- **[Extension System](./frontend/extension-system.md)** - Create custom pages with Vue.js components (linked to menus)
- **[Header Actions](./frontend/header-actions.md)** - Inject custom actions into header and sub-header areas
- **[Permission Builder](./frontend/permission-builder.md)** - Visual interface for creating complex permission rules
- **[Permission Components](./frontend/permission-components.md)** - PermissionGate and usePermissions for UI control
- **[Form System](./frontend/form-system.md)** - Dynamic form generation with validation and relations

**⚙️ Backend (Developer Integration)**
- **[API Lifecycle](./server/api-lifecycle.md)** - 🔄 **Request lifecycle**, hook system, and context sharing
- **[API Querying](./server/api-querying.md)** - 🔥 **MongoDB-like API querying** with powerful operators, relation filtering, aggregation, and deep relations
- **[GraphQL API](./server/graphql-api.md)** - 📊 **GraphQL queries and mutations** with auto-generated schema and permission control
- **[Context Reference](./server/context-reference.md)** - 📖 **Complete $ctx object reference** for hooks and handlers with examples
- **[Hook Development](./server/hook-development.md)** - Advanced hook programming with context, examples, and best practices
- **[Cluster Architecture](./server/cluster-architecture.md)** - 🏗️ **Multi-instance coordination** and distributed synchronization
- **[Permission System](./server/permission-system.md)** - 🔐 **Role-based access control** with allowedUsers bypass
- **[Bootstrap Scripts](./server/bootstrap-scripts.md)** - 🚀 **Startup script execution** with full context access and hot reload

**📝 Examples & Templates**
- **[User Registration](./examples/user-registration-example.md)** - Complete end-to-end example featuring template syntax, hooks, handlers, and package management

## Installation

Enfyra requires both backend and frontend to work properly. See our complete installation guide:

**[→ Complete Installation Guide](./getting-started/installation.md)**

Quick overview:
1. First install Enfyra Server
2. Then install Enfyra App  
3. Connect them together

## Contributing

We welcome contributions! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Development Setup

1. Fork the repository
2. Clone your fork: `git clone https://github.com/your-username/enfyra.git`
3. Install dependencies: `npm install`
4. Create a feature branch: `git checkout -b feature/amazing-feature`
5. Make your changes and commit: `git commit -m 'Add amazing feature'`
6. Push to your branch: `git push origin feature/amazing-feature`
7. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- 📖 [Documentation](./)
- 🐛 [Issues](https://github.com/dothinh115/enfyra/issues)
- 💬 [Discussions](https://github.com/dothinh115/enfyra/discussions)

## Credits

Built with ❤️ using:

- [Nuxt.js](https://nuxt.com/) - The Vue.js Framework
- [Vue.js](https://vuejs.org/) - The Progressive JavaScript Framework
- [Nuxt UI](https://ui.nuxt.com/) - UI Components
- [TypeScript](https://www.typescriptlang.org/) - Type Safety

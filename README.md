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

**🔥 Real-time Everything**

- **Live Extension System**: Write Vue/JavaScript extensions that compile and load instantly from the database - no server restarts, no deployments
- **Zero-downtime Schema Updates**: Change your data structure while your API stays 100% available
- **Instant API Generation**: Every table immediately becomes a full REST & GraphQL API with advanced querying

**🛡️ Enterprise-Grade Security**

- **Visual Permission Builder**: Create complex permission logic with AND/OR conditions and nested rules
- **Dynamic Role System**: Permissions that adapt based on data relationships and user context
- **Handler Isolation**: Custom code runs in isolated processes for security and stability

**🚀 Beyond Traditional CMS**

- **Meta-Programming Core**: The entire API structure is generated from database metadata in real-time
- **Multi-Instance Coordination**: Run multiple instances with automatic schema synchronization via Redis
- **Smart Caching**: SWR (Stale-While-Revalidate) pattern for optimal performance without sacrificing freshness

### 🎯 Core Capabilities

| Feature               | How It Works                                                            |
| --------------------- | ----------------------------------------------------------------------- |
| **Extension System**  | Write custom features that compile and load instantly from the database |
| **Schema Changes**    | Modify your data structure with zero downtime - APIs stay available     |
| **Permission System** | Visual builder for complex access control                               |
| **API Generation**    | Every table instantly becomes a full REST & GraphQL API                 |
| **Custom Code**       | Execute business logic in isolated processes with full request context  |
| **Multi-Instance**    | Run multiple servers with automatic synchronization                     |

### 💡 Perfect For

- **Rapid Development**: Go from idea to production API in minutes, not days
- **Complex Projects**: Handle sophisticated data relationships and business logic without limitations
- **Scale-Ready Applications**: Start small and scale to enterprise without architectural changes
- **Team Collaboration**: Intuitive interface for non-technical users, powerful tools for developers
- **Custom Solutions**: Build exactly what you need without fighting framework limitations

### 🏗️ Built With Modern Technology

**Backend**: NestJS + TypeORM + Redis + GraphQL Yoga
**Frontend**: Nuxt 4 + Vue 3 + TypeScript + TailwindCSS
**Database**: MySQL, MariaDB, PostgreSQL (your choice)
**Extensions**: Dynamic Vue SFC compilation via Vite

## Features

- **Dynamic Table Management** - Create and modify database tables on the fly
- **Official SDK** - `useEnfyraApi` and `useEnfyraAuth` from @enfyra/sdk-nuxt package
- **TypeScript Support** - Full type safety throughout the application
- **Extension System** - Extensible architecture with dynamic extension loading
- **Authentication System** - Built-in user authentication and roles
- **Permission System** - Comprehensive role-based access control (RBAC)
- **Menu Registry** - Dynamic sidebar and menu management
- **Header Actions** - Configurable header button system

## 📚 Documentation

- **[Installation](./docs/installation.md)** - Setup guide for backend and app
- **[Getting Started](./docs/getting-started.md)** - First steps after installation, including table creation
- **[Relation Picker](./docs/relation-picker.md)** - Working with related data in forms (uses Filter System for searching)
- **[Filter System](./docs/filter-system.md)** - Advanced data filtering and search capabilities (used in tables and relation pickers)

## Installation

```bash
# Create new Enfyra app
npm create @enfyra/create-enfyra-app@latest my-app
cd my-app

# Start development server
npm run dev
```

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

- 📖 [Documentation](./docs/)
- 🐛 [Issues](https://github.com/dothinh115/enfyra/issues)
- 💬 [Discussions](https://github.com/dothinh115/enfyra/discussions)

## Credits

Built with ❤️ using:

- [Nuxt.js](https://nuxt.com/) - The Vue.js Framework
- [Vue.js](https://vuejs.org/) - The Progressive JavaScript Framework
- [Nuxt UI](https://ui.nuxt.com/) - UI Components
- [TypeScript](https://www.typescriptlang.org/) - Type Safety

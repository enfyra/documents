# Enfyra Cloud

Enfyra Cloud is the managed hosting option for Enfyra. Use it when you want an Enfyra project online without provisioning your own server, database, Redis, reverse proxy, TLS, and deployment pipeline.

Self-hosted Enfyra and Enfyra Cloud use the same product model: an Enfyra runtime, the admin app, generated REST APIs, optional GraphQL, realtime events, flows, extensions, users, roles, files, and metadata-driven schema management. The difference is operational ownership. With self-hosting, you own the infrastructure. With Cloud, Enfyra operates the runtime boundary, shared services, deployment, payment-gated provisioning, and platform maintenance.

## When To Use Cloud

Choose Enfyra Cloud when you want to:

- Start with a hosted Enfyra instance instead of installing Docker or managing a VPS.
- Create projects from the Cloud dashboard and receive the project URL and first admin credentials after provisioning.
- Use Enfyra for production apps while keeping infrastructure work outside your team.
- Keep a clear upgrade path from managed hosting to self-hosting if your infrastructure needs become more specialized.

Choose self-hosting when you need to:

- Control the server, database, Redis, storage, network, and deployment environment yourself.
- Run inside a private network or compliance boundary that cannot use managed hosting.
- Customize infrastructure beyond the Cloud plan model.

## Project Isolation

Each Cloud project runs in its own Enfyra runtime boundary. App logic, configuration, credentials, tenant metadata, and project access are isolated per project.

Cloud infrastructure uses shared supporting services where that improves operational efficiency. Database, edge, and related platform services are capacity-managed rather than carved into fixed idle slices for every project. This lets active projects benefit from available room when nearby workloads are quiet.

The platform uses a headroom model: capacity is planned with spare operating room, and edge/runtime guardrails prevent one active project from consuming the shared pool in a way that blocks other active projects. Higher plans reserve more headroom per project, so fewer projects share the same capacity envelope and each project has more operating room during busy periods.

Cloud plan pages describe customer-facing limits such as storage, transfer, region availability, checkout availability, and support level. Raw host package details, provider location codes, and internal CPU/RAM placement values are intentionally not part of the public plan contract.

## Creating A Cloud Project

1. Open the Cloud dashboard at `https://cloud.enfyra.io`.
2. Sign in or create an account.
3. Verify your email address if the project creation flow requires it.
4. Choose an available plan and region.
5. Enter the project name, subdomain, and project `SECRET_KEY`.
6. Review the project details.
7. Complete checkout when online payment is available.

After payment confirmation, Cloud provisions the project asynchronously. The project URL, login email, and one-time admin password are sent to the account email after provisioning succeeds.

If checkout is paused, use the early-access or contact flow from the public site instead of creating a paid project from the dashboard.

## Billing And Checkout

Cloud checkout is payment-gated. Creating or upgrading a project first creates a payment order; the project is provisioned only after the payment provider confirms the order.

Cloud pricing pages show the plan subtotal before buyer-country tax unless the checkout provider displays otherwise. Taxes and payment-provider checkout details are handled during checkout. For billing records, pending checkout recovery, and past orders, use the Cloud dashboard Orders page.

Cloud does not require a separate customer billing dashboard while the payment provider owns checkout and renewal management. Project billing details remain attached to project and order surfaces in the Cloud dashboard.

## Regions

Cloud regions are selected from the available region catalog in the dashboard. Customer-facing region labels describe the location in plain language. Internal provider names, host location codes, and server package details are not exposed in the customer UI.

Unavailable regions may be shown as future or disabled capacity. A project can only be created in a region that is currently available.

## Cloud And The Open-Source Runtime

Cloud projects are normal Enfyra projects from the builder's point of view. Once provisioned, you use the Enfyra admin app the same way you would in a self-hosted install:

- Create tables and relations.
- Configure roles, route permissions, hooks, handlers, flows, files, and extensions.
- Use generated REST APIs, optional GraphQL, and Socket.IO events.
- Connect external Nuxt, Next.js, SvelteKit, Remix, mobile, or server clients through the same API patterns documented in this repository.

For local development, Docker, or infrastructure ownership, start with the [Installation Guide](../getting-started/installation.md). For managed hosting, start with Enfyra Cloud.

# Enfyra Cloud

Enfyra Cloud is a management plane for Enfyra projects that run in infrastructure accounts you own. Enfyra connects to supported providers through OAuth, creates and manages the resources selected for each project, and gives you one console for provisioning, deployments, updates, credentials, domains, backups, and runtime operations.

Enfyra does not resell compute. The management subscription is `$11.99` per project per month. Your infrastructure provider bills compute, memory, storage, network traffic, backups, and provider features directly to your provider account.

Railway is the first supported provider. The Cloud project model is provider-neutral, so additional OAuth-capable providers can be added without changing the ownership contract.

## When To Use Cloud

Choose Enfyra Cloud when you want to:

- Operate Enfyra without maintaining a VPS or building a deployment control plane.
- Keep infrastructure, usage, data, and provider billing in your own account.
- Let Enfyra manage provisioning, deployments, pinned runtime updates, restarts, environment variables, domains, credentials, and backups.
- Choose a supported provider for each project instead of locking every project to one infrastructure vendor.

Choose self-hosting when you need complete control over the host, network, deployment pipeline, and every backing service without granting Enfyra management access.

## Project And Provider Ownership

The Enfyra Cloud project is the management unit. Each project selects one supported provider and maps to one provider project. You authorize Enfyra through the provider's OAuth flow; you never paste a personal access token into Enfyra Cloud.

Provider resources remain in your account. Disconnecting OAuth, stopping subscription renewal, losing management entitlement, or removing the Cloud management record does not silently stop or delete provider services and does not stop provider billing. A destructive provider-resource action is separate and always requires explicit confirmation.

## Creating A Project

1. Open `https://cloud.enfyra.io` and sign in.
2. Create the Enfyra Cloud project record.
3. Choose a supported infrastructure provider.
4. Connect the provider through OAuth if it is not connected yet.
5. Configure the provider project, region, subdomain, administrator email, and runtime options.
6. Review the separate Enfyra and provider billing responsibilities.
7. Activate the `$11.99` Enfyra management subscription through PayPal.
8. Start provisioning.

Payment is the final setup gate. Enfyra provisions provider resources only after the project configuration is complete and the management subscription is active.

## Current Railway Topology

For Railway, Enfyra creates:

- One Railway project for one Enfyra Cloud project.
- A dedicated always-on Railway PostgreSQL service with its own persistent database volume.
- An Enfyra service connected to PostgreSQL through Railway's private network.
- Embedded Redis inside the Enfyra service, with persistent runtime data under `/app/data`.
- A Railway service domain and an Enfyra-managed custom domain.

The production database is never embedded in the Enfyra container. Restarting or updating the runtime leaves the PostgreSQL volume attached.

## File Storage

File uploads require external object storage. Configure Amazon S3, Cloudflare R2, Google Cloud Storage, or another supported S3-compatible backend after provisioning. Railway runtime and database volumes are not the permanent upload store.

## Railway Serverless

Railway Serverless is optional for the Enfyra runtime service. PostgreSQL remains always on.

Enabling or disabling Serverless requires a new Railway deployment before the setting takes effect. Enfyra warns before applying the change and starts the redeployment after confirmation. A sleeping runtime can make the first request slower or temporarily return a provider error while the container wakes. Outbound traffic, open WebSocket or database connections, and background work can also prevent a service from sleeping.

## Billing And Cancellation

The Enfyra subscription and the provider bill are independent:

- PayPal processes `$11.99` per managed project per month for Enfyra management.
- The connected provider bills infrastructure usage directly.
- Enfyra does not add a provider-usage surcharge or include compute in the management price.

Cloud offers **Stop renewal**, not a voluntary refund. Stopping renewal prevents the next subscription charge and keeps management access until the end of the paid period. Paid subscription periods are not refunded or prorated except where applicable law requires a correction.

When management access ends, Enfyra-managed operations are disabled. Your provider resources continue under your provider account until you change or delete them there or use a separate explicitly confirmed provider-resource action.

## Backups And Drift

Backup capabilities depend on the selected provider and account tier. On Railway, built-in volume backups require a Pro or Enterprise workspace. Other workspaces must use an external PostgreSQL backup target, such as scheduled `pg_dump` uploads to S3-compatible storage.

A restore replaces current database data and always requires explicit confirmation.

Changes made directly in the provider console are detected during reconciliation. Missing or changed services, volumes, mount paths, domains, images, or regions appear as drift. Unsafe managed operations are disabled until the mapping is reviewed; Enfyra does not overwrite or recreate customer-owned resources automatically.

## Using The Enfyra Runtime

After provisioning, open the project URL and use Enfyra normally:

- Create tables and relations.
- Configure roles, route permissions, hooks, handlers, flows, and extensions.
- Use generated REST APIs, optional GraphQL, and Socket.IO events.
- Create application API tokens without exposing the Cloud management credential.

For a locally operated runtime, Docker, or full infrastructure ownership, start with the [Installation Guide](../getting-started/installation.md).

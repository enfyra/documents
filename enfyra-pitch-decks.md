# Enfyra Pitch Decks

Two slide-ready narratives for distinct audiences. The decks deliberately avoid invented market size, traction, revenue, customer, or fundraising numbers. Replace bracketed fields with verified business data before external distribution.

---

# Deck 1 — Investor Pitch Deck

**Audience:** pre-seed/seed investors evaluating the company, product thesis, market and funding case.

**Recommended length:** 13 slides, 10–12 minutes.

## Slide 1 — Cover

### Enfyra

**The programmable backend platform for teams that need to ship—and keep changing—production applications.**

Define data, APIs, permissions, automation, and realtime behavior as metadata. Enfyra turns those definitions into a live backend without a redeploy for every change.

`Open source · Self-hosted · Cloud-ready`

**Speaker note:** Enfyra is not just a database UI or an API generator. It is a runtime where backend capabilities are defined and operated as versioned, executable metadata.

---

## Slide 2 — The Problem

### Modern product teams repeatedly rebuild the same backend foundations

- Every new product needs data models, CRUD APIs, authentication, permissions, storage, workflows, and realtime.
- SaaS backends make early delivery fast, but introduce platform lock-in, opaque operational control, and limited customization.
- Custom backends offer control, but force teams to spend months rebuilding undifferentiated infrastructure.
- Low-code tools accelerate prototypes, then break down when teams need custom logic, policy, self-hosting, or multi-instance operation.

**Bottom line:** teams choose between speed and control when they should not have to.

**Speaker note:** The pain is strongest for technical teams building internal tools, SaaS products, vertical software, and AI-enabled apps that change their data and workflows frequently.

---

## Slide 3 — Why Now

### Application development is becoming more dynamic, not less

- AI increases the volume of product experiments and shortens iteration cycles.
- Teams need backend infrastructure that evolves as quickly as the product surface.
- Data residency, security reviews, and cost predictability are increasing demand for deployable and self-hosted platforms.
- APIs, automation, event-driven workflows, and realtime expectations are now baseline product capabilities.

**Thesis:** the winning backend platform combines the speed of a managed service with the extensibility and deployment control of a real backend runtime.

---

## Slide 4 — The Product

### Enfyra makes backend capabilities executable metadata

```text
Define                                  Run immediately
Tables and relations       ───────►     REST and GraphQL APIs
Roles and field policy     ───────►     Authenticated, governed access
Hooks and handlers         ───────►     Custom server-side behavior
Flows                      ───────►     Scheduled and event-driven automation
WebSocket events           ───────►     Realtime application features
Extensions                 ───────►     Dynamic operational interfaces
```

- Build visually or through APIs.
- Change supported runtime definitions without restarting the server.
- Keep a clear escape hatch: write server-side logic when metadata alone is insufficient.

**Speaker note:** The key product motion is “define, apply, run”—not generate code, copy it out, and maintain another backend.

---

## Slide 5 — How It Works

### One operating layer over data, APIs, and runtime behavior

```text
Admin UI / third-party applications
             │
             ▼
Enfyra App — same-origin API, auth, and Socket.IO bridge
             │
             ▼
Enfyra Server — policy, APIs, hooks, handlers, flows, realtime
             │
             ▼
PostgreSQL / MySQL / MongoDB  +  Redis
```

- Database stores business data and Enfyra’s runtime metadata.
- Enfyra Server loads metadata and executes the request lifecycle.
- Redis coordinates cache invalidation, workflow queues, realtime fanout, and multi-instance runtime behavior.

**Speaker note:** This architecture gives Enfyra a practical self-hosted model while still enabling a managed cloud product later.

---

## Slide 6 — Why Enfyra Is Different

### Speed without a dead end

| Typical category | Primary strength | Common ceiling | Enfyra position |
|---|---|---|---|
| Backend-as-a-service | Fast managed setup | Lock-in and limited runtime control | Deployable, programmable backend runtime |
| Headless CMS | Content modeling | Application logic and policy depth | Data plus executable APIs, workflow, and realtime |
| Low-code platform | Fast internal apps | Custom logic and engineering portability | Visual definitions with code-level escape hatches |
| Custom backend | Maximum flexibility | Slow, repetitive infrastructure work | Metadata automation plus custom handlers and hooks |

**Enfyra’s wedge:** a metadata runtime that generates the standard backend surface, then lets developers customize the exact points where product differentiation lives.

---

## Slide 7 — Initial Customers and Use Cases

### Start where backend change velocity is a competitive advantage

**Ideal customer profiles**

- Technical SaaS teams building data-heavy workflows.
- Agencies and system integrators delivering multiple bespoke applications.
- Enterprises building internal operations tools under data-control requirements.
- Product teams that need self-hosted deployment, custom policy, and faster iteration than a bespoke backend permits.

**High-frequency use cases**

- Internal operations portals and approval workflows.
- Multi-tenant B2B SaaS applications.
- Customer support, CRM, inventory, and order operations.
- Realtime collaboration, messaging, notification, and monitoring products.
- API backends for Nuxt, Next.js, mobile, and desktop applications.

---

## Slide 8 — Business Model

### Open source distribution with paid operational leverage

**Community / self-hosted**

- Free core for local development, evaluation, and self-managed deployments.
- Adoption through developers, templates, integrations, and community support.

**Paid cloud**

- Managed hosting, upgrades, backups, observability, scaling, and support.
- Usage- and environment-based plans for production workloads.

**Enterprise**

- Private deployment, SSO and governance capabilities, compliance support, priority support, and architecture assistance.

**Partner channel**

- Agency / systems-integrator plans, implementation support, and white-label or multi-project operational tooling.

**Speaker note:** Present pricing only after validating actual packaging, costs, and willingness-to-pay. Do not put forecasted ARR on this slide without a financial model.

---

## Slide 9 — Go-To-Market

### Developer adoption first; expand through repeatable delivery channels

1. **Open-source developer funnel** — simple local setup, strong documentation, reference applications, and framework integrations.
2. **Technical content** — show complete applications built around data, permissions, workflows, and realtime—not isolated feature demos.
3. **Agency and integrator partners** — help partners standardize their backend delivery layer across client projects.
4. **Cloud conversion** — remove self-hosting and operational burden once users reach production.
5. **Enterprise expansion** — land through a focused operational use case, then expand to additional teams and applications.

**North-star signal:** a team takes a product from schema to governed production API faster than it can with its existing backend stack.

---

## Slide 10 — Proof and Operating Metrics

### Insert verified evidence here; do not fabricate traction

Use only metrics that can be independently substantiated:

- `[#]` active self-hosted instances or qualified deployments.
- `[#]` weekly/monthly active builders.
- `[#]` production applications, partner implementations, or paying customers.
- `[time]` to build a representative application versus the team’s previous stack.
- `[#]` GitHub stars, contributors, package downloads, docs traffic, or community members.
- `[#]` pilot conversions, cloud waitlist applications, or signed partner commitments.

**What investors need to see next:** repeated adoption by a defined ICP, evidence that users deploy meaningful workloads, and an early path from open-source usage to paid operations.

---

## Slide 11 — Roadmap and Milestones

### De-risk adoption, operations, then expansion

**Near term**

- Strengthen getting-started experience, templates, and reference apps.
- Validate the core developer workflow across supported database targets.
- Package repeatable authentication, data, API, workflow, and realtime patterns.

**Next**

- Launch and validate managed-cloud operating workflows.
- Add the enterprise controls required by target buyers.
- Establish partner onboarding and implementation playbooks.

**Milestones to track**

- Time-to-first-production project.
- Active production deployments.
- Cloud conversion and retention.
- Partner-sourced projects and expansion revenue.

---

## Slide 12 — Funding Ask

### Fund the transition from capable platform to repeatable company

**Raising:** `[amount]` `[round type]`

**Use of funds**

- Product: managed cloud, onboarding, reliability, and productized templates.
- Go-to-market: developer growth and partner activation around the selected ICP.
- Customer success: design partners, migration support, and early enterprise requirements.

**18-month outcomes to target**

- `[target]` production deployments or active teams.
- `[target]` cloud / enterprise revenue milestone.
- `[target]` repeatable acquisition channel or partner motion.
- `[target]` reliability and deployment SLOs.

**Speaker note:** Replace the placeholders only with an approved fundraising plan and an operating model that connects spend to the milestones.

---

## Slide 13 — Closing

### Backend infrastructure should compound product velocity—not consume it

**Enfyra lets teams define, govern, and evolve a production backend at the speed their products change.**

`Open source foundation`  `Self-hosted control`  `Programmable runtime`  `Managed-cloud opportunity`

**Contact:** `[name] · [email] · [website]`

---

# Deck 2 — Technical Partner Pitch Deck

**Audience:** framework partners, agencies, system integrators, platform partners, developer-tool vendors, and technical ecosystem collaborators.

**Recommended length:** 14 slides, 15–20 minutes plus architecture discussion.

## Slide 1 — Cover

### Build applications on Enfyra

**A programmable backend runtime for partners who need faster delivery without surrendering engineering control.**

`APIs · Auth · Permissions · Workflows · Realtime · Storage · Custom code`

---

## Slide 2 — Partner Value Proposition

### Standardize the backend layer; differentiate in the product and delivery

With Enfyra, partners can:

- Start from a governed data model rather than rebuilding CRUD infrastructure.
- Deliver REST and GraphQL APIs directly from metadata-backed collections.
- Add custom business behavior through hooks, handlers, flow steps, and WebSocket events.
- Deploy self-hosted where customer architecture or data policy requires it.
- Reuse patterns across implementations while preserving per-customer customization.

**Partner outcome:** less repetitive backend construction, more time for domain workflows, UI, integrations, and customer value.

---

## Slide 3 — What Enfyra Provides

### A complete backend operating surface

| Capability | What partners use it for |
|---|---|
| Data modeling | Tables, columns, relations, indexes, validation rules |
| Generated APIs | Route-backed REST and optional GraphQL APIs |
| Identity and policy | Sessions, OAuth, JWT, route permissions, field permissions, guards |
| Programmable runtime | Pre-hooks, handlers, post-hooks, dynamic scripts |
| Automation | Event, webhook, and scheduled flows with execution history |
| Realtime | Socket.IO gateways and metadata-backed event handlers |
| Files | Storage configuration, asset permissions, and upload APIs |
| Operations UI | Dynamic admin pages, widgets, and extensions |
| Runtime coordination | Redis-backed cache reloads, queues, and multi-instance fanout |

---

## Slide 4 — Architecture

### Keep the application frontend independent from the backend runtime

```text
Nuxt / Next / React / Mobile / Desktop application
        │  same-origin /enfyra and /socket.io proxy
        ▼
Enfyra App bridge
        │
        ▼
Enfyra Server
  ├─ dynamic REST and GraphQL
  ├─ auth, permissions, guards
  ├─ hooks, handlers, flows
  ├─ Socket.IO gateways
  └─ storage and runtime reloads
        │
        ▼
PostgreSQL / MySQL / MongoDB  +  Redis
```

- The frontend does not connect to the database directly.
- The app bridge supports same-origin browser cookies and hides the server origin.
- Enfyra can operate as the backend behind a partner’s existing frontend stack.

---

## Slide 5 — From Schema to a Governed API

### A collection is more than a table

```text
Collection definition
  ├─ fields + validation
  ├─ relations
  ├─ route-backed REST projection
  ├─ optional GraphQL exposure
  ├─ route permissions and guards
  ├─ field-level policy
  └─ hooks / custom behavior
```

Example lifecycle:

```text
Create `ticket` collection
  → Enfyra stores metadata and updates physical storage
  → default `/ticket` REST route is available
  → add policy, relationships, hooks, and custom workflow
  → affected runtime cache reloads
```

**Key property:** supported metadata changes activate without restarting the server.

---

## Slide 6 — Request Lifecycle and Security Boundaries

### Make policy explicit in the runtime path

```text
Request
  → route detection
  → auth/session lookup
  → guards + route permissions
  → pre-hooks
  → custom handler OR canonical CRUD
  → post-hooks
  → field-permission cleanup
  → response
```

- Route permissions define who may call an operation.
- Guards enforce cross-cutting controls such as policy trees or rate limits.
- Field permissions govern read/create/update exposure at the data boundary.
- Pre-hooks can apply tenant/owner constraints before a query or mutation reaches business logic.

**Partner practice:** treat UI visibility as convenience; enforce authorization at the API and field boundary.

---

## Slide 7 — Programmable Escape Hatches

### Metadata accelerates the common case; scripts own the differentiated case

**Pre-hooks** — validate, transform, or inject request context before CRUD/handlers.

**Custom handlers** — provide endpoint-specific behavior and compose repository operations.

**Post-hooks** — transform responses, trigger downstream work, or emit events.

**Flow steps** — orchestrate scheduled, event-driven, and multi-step work.

**WebSocket handlers** — validate events, persist state, and emit targeted realtime updates.

```js
if (@BODY && @USER?.id) {
  @BODY.author = { id: @USER.id }
}
```

**Partner outcome:** teams do not fork the server to add application behavior.

---

## Slide 8 — Workflows and Realtime

### Build operational applications, not only data APIs

**Flows**

- Triggered manually, by a schedule, a table event, or webhook routes.
- Support ordered steps, execution history, and queued/background work.
- Typical uses: onboarding, notifications, payment processing, integrations, cleanup, and approvals.

**Socket.IO gateways**

- Define gateway paths and event handlers as runtime metadata.
- Authorize and validate messages in server-side scripts.
- Use Redis-backed fanout for multi-instance delivery when Redis is enabled.

**Example:** create a support ticket, trigger an assignment flow, and notify the relevant operator room in realtime.

---

## Slide 9 — Deployment and Operations Model

### Match customer deployment requirements without changing application semantics

| Environment | Typical configuration |
|---|---|
| Local development | Enfyra with a local supported database |
| Single-instance production | Enfyra Server + database; Redis when required features are enabled |
| High availability | Multiple Enfyra Server instances coordinated through Redis |
| Enterprise/private environment | Customer-controlled infrastructure, network, and database |
| Managed offering | Enfyra cloud operations layer when available |

Redis coordinates runtime invalidation, shared cache behavior, flows/queues, and Socket.IO fanout. When one instance changes supported metadata, other instances receive invalidation and reload affected runtime state.

---

## Slide 10 — Integration Patterns

### Keep integrations at stable boundaries

**Frontend applications**

- Use a same-origin proxy prefix such as `/enfyra/**` for API and `/socket.io/**` for realtime transport.
- Compatible integration patterns are documented for SSR frameworks including Nuxt, Next.js, SvelteKit, and Remix.

**External systems**

- Call generated REST endpoints or GraphQL where enabled.
- Trigger flows for asynchronous work and integration sequencing.
- Use custom routes/handlers when an endpoint requires a business-specific contract.

**SDKs**

- Use framework-appropriate SDK packages where available, or connect through standard HTTP and Socket.IO interfaces.

---

## Slide 11 — Suggested Partner Engagement Models

### Choose the partnership that matches your value chain

**Implementation partner**

- Build customer-specific applications and operational tools on Enfyra.
- Reuse schema, policy, workflow, and integration templates across engagements.

**Technology partner**

- Provide database, hosting, identity, observability, storage, or developer-tool integrations.
- Publish reference architectures and connector patterns.

**Framework / SDK partner**

- Deliver a first-class integration for a frontend ecosystem or vertical developer platform.

**Cloud / managed-services partner**

- Operate Enfyra for customers that want a managed platform while retaining deployment flexibility.

---

## Slide 12 — Proof-of-Concept Plan

### Validate the platform on one real workflow in 2–4 weeks

**Week 1 — Model and policy**

- Select a bounded production-like workflow: e.g. support intake, order approval, or partner onboarding.
- Define data, roles, field permissions, and user journeys.

**Week 2 — API and logic**

- Generate the CRUD/API surface.
- Add the minimum hooks, handlers, storage, and flows needed for the workflow.

**Week 3 — Frontend and realtime**

- Connect the partner’s frontend through the same-origin bridge.
- Add notification or collaboration behavior where relevant.

**Week 4 — Operational review**

- Test permissions, failure paths, deployment topology, and the upgrade/ownership model.
- Document reusable patterns and success metrics.

**Success criteria:** measurable reduction in backend setup time, no loss of required customization, and a clear operational path for the customer environment.

---

## Slide 13 — What We Need From a Technical Partner

### Co-design around a concrete customer and integration boundary

- A target use case with an accountable product or delivery owner.
- Access to the partner’s frontend, deployment, and identity constraints.
- A defined database and infrastructure preference.
- Technical feedback on onboarding, extension points, observability, security controls, and operational fit.
- Agreement on success criteria and whether the outcome can become a reusable reference architecture.

**What Enfyra brings:** platform expertise, architecture support, product feedback loop, and a commitment to turn validated patterns into reusable platform capability.

---

## Slide 14 — Closing

### Let’s make backend delivery repeatable without making applications generic

**Enfyra supplies the programmable backend layer. Partners bring the domain expertise, customer relationships, and differentiated experience.**

**Next step:** choose one integration pattern or customer workflow for a technical discovery session.

`[contact name] · [email] · [website]`

---

# Shared Design Direction

- **Language:** English for external decks; create a Vietnamese talk track only if the meeting audience needs it.
- **Visual style:** dark technical base, warm neutral accent, sparse diagrams, large type, and real screenshots only after they are stable and permission-cleared.
- **Investor deck:** outcome- and market-led. Limit architecture detail to one slide.
- **Technical partner deck:** show request lifecycle and deployment topology. Bring a live demo or architecture appendix rather than overcrowding the core deck.
- **Evidence discipline:** do not claim enterprise readiness, scale, compliance, cloud availability, customers, revenue, deployment counts, or performance results without verifiable evidence.

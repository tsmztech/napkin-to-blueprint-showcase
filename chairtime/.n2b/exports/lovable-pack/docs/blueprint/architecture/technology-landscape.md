---
document_type: technology-landscape
produced_by: technical-researcher
status: final
stage: 4
created: 2026-09-29
area_count: 21
option_count: 103
---

# Technology Landscape

## 1. Research Scope

| Decision Area | Activated By |
|---|---|
| Frontend Framework | Always active |
| Backend / API Layer | Always active |
| Database | Always active |
| ORM / Data Access | Always active |
| CSS / Styling | Always active |
| State Management | Always active |
| Build Tooling | Always active |
| Authentication & Identity | Always active |
| Hosting & Environments | Always active |
| CI/CD & Delivery | Always active |
| Observability & Operations | Always active |
| File & Object Storage | File upload (FEAT-27.SPEC-001, FEAT-27.SPEC-012; ASMP-35); Import/export (FEAT-29.SPEC-004, FEAT-29.SPEC-007, FEAT-16.SPEC-004, FEAT-08.SPEC-001) |
| Email & Messaging Delivery | Notifications (email/push/SMS) (24 Notification specs, e.g. FEAT-08.SPEC-001, FEAT-29.SPEC-014; Integration specs FEAT-08.SPEC-012, FEAT-08.SPEC-013, FEAT-26.SPEC-002) |
| Payments & Billing | Payments/billing (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011; ASMP-31) |
| Search | Search (FEAT-24.SPEC-001, FEAT-24.SPEC-002) |
| Background Jobs & Scheduling | Background processing (31 scheduled/timed Automation specs, e.g. FEAT-03.SPEC-003, FEAT-08.SPEC-007, FEAT-18.SPEC-003); Import/export (FEAT-29.SPEC-007); Notifications (email/push/SMS) (FEAT-08.SPEC-007) |
| Caching & Performance | Scale hints (ASMP-21, ASMP-22, ASMP-26); Offline (ASMP-27; FEAT-12.SPEC-001, FEAT-13.SPEC-001) |
| Real-time & Collaboration | Real-time (FEAT-03.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002; ASMP-21); Collaboration/concurrency (16 of 18 entities with contention; FEAT-03.SPEC-005, XBR-01) |
| Analytics & Product Telemetry | Scale hints (ASMP-21, ASMP-22, ASMP-26; BRIEF.md Scale & Non-Functional Expectations) |
| Internationalization | Internationalization (FEAT-07.SPEC-001, FEAT-15.SPEC-002, FEAT-22.SPEC-001, FEAT-27.SPEC-003, FEAT-27.SPEC-008; ASMP-25) |
| Calendar Sync | Product-mandated — "Pro's personal calendar (Google Calendar and Apple Calendar — both matter): two-way. Busy times there block Chairtime availability, and bookings made in Chairtime appear there." (BRIEF.md, ## Ecosystem & Integrations); ASMP-33 "Calendar-sync capability (reading and writing to a pro's personal calendar)" (assumptions-constraints.md, ## Dependencies); FEAT-04.SPEC-003, FEAT-03.SPEC-006 |

Inactive (not in scope): AI & Intelligent Behavior (AI/ML behavior = No), Geo & Maps (Geo/maps = No).

## 2. Decision Area Landscapes

### Frontend Framework

Serves a mobile-first web app for pros and clients, including the Instagram in-app browser (BRIEF.md Devices & platforms), 70 Screen specs, an 8-step setup wizard (FEAT-15), a multi-step booking flow (FEAT-05) and roughly one-second slot refresh (FEAT-03.SPEC-001).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Next.js (App Router) | Framework (React meta-framework) | SSR/SSG for fast public booking page loads, server actions and route handlers, large component ecosystem for wizard and dashboard screens | Open source (MIT); hosting cost separate | Low — mainstream React skills; first-class on Vercel, deployable elsewhere via Node | Very mature; framework conventions are Vercel-shaped but portable to Node hosts | knowledge-based — not fetched this run; figures from model knowledge |
| React Router v7 (framework mode, formerly Remix) | Framework (React) | Loader/action model suits form-heavy flows and progressive enhancement; SSR | Open source (MIT) | Low–medium — React skills; adapters for several runtimes | Mature; portable across Node, edge and serverless adapters | knowledge-based — not fetched this run; figures from model knowledge |
| SvelteKit | Framework (Svelte) | Small client bundles helpful in in-app browsers; form actions; smaller ecosystem than React | Open source (MIT) | Medium — Svelte skills less common; adapters for Node, Vercel, Cloudflare | Mature; ecosystem smaller, component libraries Svelte-specific | knowledge-based — not fetched this run; figures from model knowledge |
| Nuxt | Framework (Vue meta-framework) | SSR/hybrid rendering, Vue composition API for forms and wizard flows | Open source (MIT) | Medium — Vue skills; Nitro server adapters | Mature; Vue-specific ecosystem | knowledge-based — not fetched this run; figures from model knowledge |
| Astro with island components | Framework (content-first, multi-framework islands) | Fast static public pages; app-like dashboards need islands and more client code | Open source (MIT) | Medium — application state across islands needs extra design | Mature for content sites; less conventional for app-heavy dashboards | knowledge-based — not fetched this run; figures from model knowledge |

### Backend / API Layer

Serves 59 Automation specs, 13 Integration specs with inbound webhooks (payments, SMS status, calendar), idempotent payment outcomes (FEAT-07.SPEC-004) and hold/contention logic (FEAT-03.SPEC-002, FEAT-03.SPEC-005).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework server layer (Next.js route handlers and server actions) | Framework-bundled server | Single deployable for UI and API; webhooks as route handlers; long-running work must be delegated to a job runner | No separate cost; runs on framework host (serverless function pricing) | Low — no separate service | Coupled to the chosen frontend framework and its host runtime limits | knowledge-based — not fetched this run; figures from model knowledge |
| Hono on Node/edge runtimes | Library/framework (TypeScript HTTP) | Lightweight typed API and webhook endpoints; runs on Node, Cloudflare, Vercel, Deno, Bun | Open source (MIT) | Low — small API surface; shares TypeScript with frontend | Newer, fast-growing; portable across runtimes | knowledge-based — not fetched this run; figures from model knowledge |
| NestJS | Framework (TypeScript, opinionated) | Modular structure, DI and guards for role enforcement across 30 features; queue/scheduler modules | Open source (MIT) | Medium — framework conventions to learn | Mature; heavier than minimal frameworks; Node-bound | knowledge-based — not fetched this run; figures from model knowledge |
| FastAPI (Python) | Framework (Python async) | Typed API and webhook handling; strong for data-heavy jobs; separate language from a TypeScript frontend | Open source (MIT) | Medium — separate service and language | Mature; pairs with SQLAlchemy, not Prisma/Drizzle | knowledge-based — not fetched this run; figures from model knowledge |
| Ruby on Rails | Framework (full-stack MVC) | Batteries-included jobs, mailers, ActiveRecord; full-stack conventions for CRUD-heavy admin | Open source (MIT) | Medium — separate stack from a JS frontend unless used full-stack | Very mature; strong convention lock-in | knowledge-based — not fetched this run; figures from model knowledge |

### Database

Serves 18 entities, 56 named relationships, high-contention Booking records (feature-dependency-map.md), multi-year history for 100–500 clients per pro (ASMP-22), and the correctness bar of never silently double-booking or losing a deposit (ASMP-26); per-pro data isolation (ASMP-23).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Postgres | Managed service (Postgres plus platform) | Full Postgres (transactions, exclusion constraints, row-level security for per-pro isolation); bundled auth, storage, realtime | Free $0 (500 MB); Pro $25/mo (8 GB, then $0.125/GB) | Low — standard Postgres drivers plus supabase-js | Postgres-standard data, exit via pg_dump; platform extras add coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Neon | Managed service (serverless Postgres) | Standard Postgres with autoscaling and branching; scale-to-zero | Free $0 (0.5 GB); Launch pay-as-you-go $0.106/CU-hour, $0.35/GB-month | Low — standard drivers, serverless HTTP driver | Postgres-standard; exit via dump/restore | https://neon.com/pricing (accessed 2026-09-29) |
| Crunchy Bridge | Managed service (Postgres on AWS/Azure/GCP) | Plain managed Postgres with PITR and SOC 2 Type 2 | From ~$10/mo; storage $0.10/GB/mo | Low — standard drivers | Postgres-standard; exit via dump/restore | https://www.crunchydata.com/pricing (accessed 2026-09-29) |
| Amazon RDS / Aurora PostgreSQL | Managed service (cloud provider) | Postgres with Multi-AZ, read replicas, PITR | Instance-hour plus storage; entry instances roughly tens of dollars per month | Medium — VPC, IAM and parameter setup | Very mature; Postgres-standard data, AWS operational coupling | knowledge-based — not fetched this run; figures from model knowledge |
| Self-hosted PostgreSQL | Self-hosted | Full control of extensions and tuning | VM/host cost plus operations time | High — backups, failover, upgrades owned by team | Open-source standard; no vendor lock-in | knowledge-based — not fetched this run; figures from model knowledge |

### ORM / Data Access

Serves 18 entities with concurrency-sensitive writes (first-committed-wins, reject-with-refresh per feature-dependency-map.md) that need explicit transactions and versioning, plus schema migrations over many features.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Prisma | ORM (TypeScript) | Typed client, declarative schema, migrations; raw SQL escape hatch for constraints and locking | Open source (Apache-2.0) | Low — Node/TypeScript only | Mature; schema DSL is proprietary but SQL is exportable | knowledge-based — not fetched this run; figures from model knowledge |
| Drizzle ORM | ORM / query builder (TypeScript) | SQL-like typed queries, migration tooling, close to SQL semantics for transactions | Open source (Apache-2.0) | Low — Node/TypeScript, edge-friendly drivers | Newer; schemas in TypeScript, low lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Kysely | Query builder (TypeScript) | Type-safe SQL builder; no schema management (migrations separate) | Open source (MIT) | Low–medium — bring migration tool | Mature; minimal abstraction, low lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| SQLAlchemy with Alembic | ORM and migration tool (Python) | Full ORM and explicit transaction control | Open source (MIT) | Medium — Python backends only | Very mature; Python-only | knowledge-based — not fetched this run; figures from model knowledge |
| supabase-js / PostgREST client | Managed data API client | Direct client-to-database API guarded by row-level security; complex multi-step transactions require SQL functions | Included in Supabase plan (https://supabase.com/pricing) | Low — generated types available | Ties data access to Supabase/PostgREST; SQL functions remain portable | https://supabase.com/pricing (accessed 2026-09-29) |

### CSS / Styling

Serves mobile-first screens readable at phone width with scalable text and contrast (ASMP-28), 70 Screen specs, and a design-system passthrough that must map to tokens.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Tailwind CSS | Utility-class framework | Design tokens map to theme config; fast responsive layouts; works with all mainstream frameworks | Open source (MIT) | Low — build plugin | Very mature; utility markup couples templates to Tailwind | knowledge-based — not fetched this run; figures from model knowledge |
| CSS Modules with CSS custom properties | Vanilla CSS approach | Tokens as custom properties; scoped styles, no runtime | Free (web standard) | Low — supported by all bundlers | Standards-based; no lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Panda CSS | Build-time CSS-in-JS | Typed token-driven styles with zero runtime | Open source (MIT) | Medium — codegen step | Newer; moderate lock-in to its API | knowledge-based — not fetched this run; figures from model knowledge |
| vanilla-extract | Build-time CSS-in-TypeScript | Typed themes and tokens compiled to static CSS | Open source (MIT) | Medium — bundler plugin | Mature; TypeScript-authored styles | knowledge-based — not fetched this run; figures from model knowledge |

Component-layer candidates (evidence: multi-step wizard, booking flow, date/time pickers, dialogs and sheets across 70 Screen specs; ASMP-28 screen-reader support):
- Radix UI primitives (headless; React) — accessible unstyled behavior skinnable to tokens.
- shadcn/ui (copy-in components; React plus Tailwind CSS).
- React Aria Components (headless; React) — accessibility-first date and time components.
- Headless UI (headless; React and Vue).
- Custom components on the styling system alone (no library).

### State Management

Serves server-driven data (slot lists refreshed about every second, bookings, money list), an 8-step wizard with resumable progress (FEAT-15.SPEC-004) and multi-step booking flow state.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| TanStack Query | Library (server-cache) | Polling/refetch, caching, optimistic updates for slot lists and dashboards; framework adapters for React, Vue, Svelte | Open source (MIT) | Low | Very mature; no lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Zustand | Library (client store, React) | Small store for wizard and booking-flow state | Open source (MIT) | Low | Mature; React-oriented | knowledge-based — not fetched this run; figures from model knowledge |
| Redux Toolkit (with RTK Query) | Library (store, React-oriented) | Predictable global state and data-fetching layer | Open source (MIT) | Medium — more boilerplate | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| SWR | Library (data fetching, React) | Stale-while-revalidate polling for live lists | Open source (MIT) | Low | Mature; React-only | knowledge-based — not fetched this run; figures from model knowledge |
| Framework built-ins (React state/context, server components, URL state) | Framework built-in | Sufficient for form state and URL-carried wizard steps; polling written by hand | Free | Low | No lock-in; more hand-written code | knowledge-based — not fetched this run; figures from model knowledge |

### Build Tooling

Serves a single web application in one repository with a TypeScript codebase across 30 features and 219 specs.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework-bundled pipeline (Turbopack/Next.js, SvelteKit via Vite, Nuxt via Vite) | Framework-bundled build | No separate bundler config; pairs with the chosen framework | Included with framework | Low | Mature; overriding needs a documented driver | knowledge-based — not fetched this run; figures from model knowledge |
| Vite | Bundler / dev server | Fast dev server; used for SPA or framework builds | Open source (MIT) | Low | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| pnpm | Package manager | Strict, fast installs, workspace support | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| npm | Package manager | Ships with Node; universal | Free | Low | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| Turborepo | Monorepo build orchestrator | Task caching if the product splits into multiple packages | Open source (MIT); remote cache optional paid | Medium | Mature; optional Vercel-coupled remote cache | knowledge-based — not fetched this run; figures from model knowledge |

### Authentication & Identity

Serves Pro one-time-code sign-in with new-device alerts and session management (FEAT-29.SPEC-001, .002, .006, .014, .015; ASMP-30), client short-lived access links scoped to one pro (FEAT-06), and a read-only Platform Operator role (FEAT-19).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Auth | Managed service | Email/phone OTP, sessions, row-level-security integration; custom access-link flow needs own code | 50,000 MAUs free; Pro $25/mo with 100,000 MAUs then $0.00325/MAU | Low if Supabase Database used | Coupled to Supabase; open-source GoTrue server | https://supabase.com/pricing (accessed 2026-09-29) |
| Clerk | Managed service (identity SaaS) | Email/SMS OTP, passkeys, session and device management, organizations/roles | Hobby free; Pro $25/mo; 50,000 MRUs included then $0.02/MRU; SMS OTP $0.01 (US/CA) | Low — SDKs and prebuilt components | User data hosted by vendor; export possible | https://clerk.com/pricing (accessed 2026-09-29) |
| WorkOS AuthKit | Managed service | Magic auth, passkeys, MFA, SSO from one integration | First 1,000,000 MAUs free | Low–medium | Vendor-hosted user store; enterprise-oriented | https://workos.com/pricing (accessed 2026-09-29) |
| Better Auth / Auth.js | Library (self-hosted in app) | Framework-native sessions and email OTP plugins; full control of access-link and device-alert logic | Open source; delivery costs (email/SMS) separate | Medium — own session tables and security review | No vendor; team owns security upkeep | knowledge-based — not fetched this run; figures from model knowledge |
| Auth0 | Managed service | Passwordless OTP, sessions, roles, actions | Free tier plus paid plans by MAU | Low–medium | Mature; migrating users out requires export work | knowledge-based — not fetched this run; figures from model knowledge |

### Hosting & Environments

Serves a mobile web app with public booking pages, webhook receivers, scheduled jobs, one-second slot response (ASMP-21), and US-first launch with UK, Canada and Australia later (BRIEF.md Geography).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vercel | Managed platform (serverless/edge) | Preview environments per branch, native Next.js hosting, cron jobs on all plans | Hobby $0; Pro $20/mo; Enterprise custom | Low | Mature; serverless function limits and platform coupling | https://vercel.com/pricing (accessed 2026-09-29) |
| Render | Managed platform (containers, workers, cron) | Web services, background workers, cron and managed Postgres in one place | knowledge-based — pricing page fetch returned no price content | Low | Mature; container-based, portable | knowledge-based — Render pricing page fetched 2026-09-29 returned navigation content only; figures from model knowledge |
| Fly.io | Managed platform (Machines, multi-region) | Long-running processes near users; multi-region posture for future markets | Usage-based machines and volumes | Medium — Dockerfile and fly config | Container-based, portable | knowledge-based — fetch redirected and was not completed this run |
| AWS (ECS Fargate / App Runner) | Cloud provider | Full control, VPC, IAM, multi-region | Usage-based compute and networking | High — infrastructure-as-code needed | Very mature; AWS service coupling | knowledge-based — not fetched this run; figures from model knowledge |
| Cloudflare Workers / Pages | Managed edge platform | Edge execution, close to R2, KV and cron triggers; Workers runtime differs from Node | Free tier; paid plans usage-based | Medium — runtime constraints | Mature; Workers APIs are Cloudflare-specific | knowledge-based — not fetched this run; figures from model knowledge |

### CI/CD & Delivery

Serves 30 features shipped without regression given the correctness bar (ASMP-26): automated tests on booking, payment and refund logic, and dev/staging/prod promotion.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| GitHub Actions | SaaS platform (CI/CD) | Workflows on pull requests, matrix tests, environment protections and deploy hooks | 2,000 free min (Free), 3,000 (Pro/Team); Linux 2-core $0.006/min | Low | Mature; workflow YAML is GitHub-specific | https://docs.github.com/en/billing/managing-billing-for-your-products/about-billing-for-github-actions (accessed 2026-09-29) |
| GitLab CI/CD | SaaS platform / self-hosted | Pipelines with environments and approvals | Tiered per user plus compute minutes | Medium — needs GitLab repository | Mature; GitLab-specific config | knowledge-based — not fetched this run; figures from model knowledge |
| CircleCI | SaaS platform | Fast parallel pipelines and caching | Credit-based with free tier | Low–medium | Mature; config is CircleCI-specific | knowledge-based — not fetched this run; figures from model knowledge |
| Hosting-platform native deploys (e.g. Vercel Git integration) | Platform feature | Automatic preview and production deploys per branch; tests run elsewhere | Included with hosting plan (https://vercel.com/pricing) | Low | Coupled to hosting platform | https://vercel.com/pricing (accessed 2026-09-29) |
| Buildkite | SaaS orchestration with self-hosted agents | Pipelines on own infrastructure | Per-user seats plus own compute | Medium | Mature; agents self-operated | knowledge-based — not fetched this run; figures from model knowledge |

### Observability & Operations

Serves a correctness-first product (ASMP-26) needing error reporting, tracing across webhook/job chains, and alerting on failed refunds, sync health (FEAT-04.SPEC-006) and delivery failures (FEAT-08.SPEC-009); compliance posture on client data (ASMP-23, ASMP-24).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Sentry | SaaS platform (error and performance monitoring) | Error tracking, tracing, session replay | Developer free; Team $26/mo; Business $80/mo | Low — SDKs for JS and Python | Mature; proprietary backend, open-source SDKs | https://sentry.io/pricing/ (accessed 2026-09-29) |
| Better Stack | SaaS platform (uptime, logs, incident alerts) | Uptime checks, log management, on-call | knowledge-based — pricing fetch returned HTTP 503 | Low | Newer; log export possible | knowledge-based — Better Stack pricing page returned HTTP 503 on 2026-09-29 |
| Datadog | SaaS platform (full observability) | APM, logs, metrics, alerting in one product | Per-host and per-GB usage pricing | Medium | Very mature; costs scale with usage, proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Grafana Cloud | SaaS platform (metrics, logs, traces) | OpenTelemetry-native metrics, logs and traces with free tier | Free tier plus usage-based | Medium | Mature; open standards limit lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| OpenTelemetry with Honeycomb | Instrumentation standard plus SaaS backend | Vendor-neutral tracing across request, webhook and job chains | Free tier plus event-volume pricing | Medium — instrumentation work | Standards-based instrumentation | knowledge-based — not fetched this run; figures from model knowledge |

### File & Object Storage

Serves the Pro profile photo upload with size and format limits (FEAT-27.SPEC-001, FEAT-27.SPEC-012; ASMP-35) and generated artifacts: data export (FEAT-29.SPEC-007), dispute summary download (FEAT-16.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Storage | Managed service | Buckets with row-level-security policies; image transformations | 1 GB free; Pro 100 GB then $0.0213/GB | Low with Supabase stack | S3-compatible protocol; platform coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Cloudflare R2 | Managed object storage | S3-compatible with zero egress fees | Zero egress; storage per GB (pricing details not on fetched page) | Low–medium | S3 API limits lock-in | https://www.cloudflare.com/developer-platform/products/r2/ (accessed 2026-09-29) |
| Amazon S3 | Managed object storage | Industry-standard object store with pre-signed uploads | Per-GB storage plus request and egress fees | Medium — IAM and CORS | Very mature; S3 API is de facto standard | knowledge-based — not fetched this run; figures from model knowledge |
| Vercel Blob | Managed object storage | Simple uploads from serverless functions | Usage-based storage and transfer | Low — on Vercel | Coupled to Vercel | knowledge-based — not fetched this run; figures from model knowledge |
| Cloudinary | SaaS platform (media management) | Image upload, resize and format conversion | Free tier plus credit-based plans | Low | Proprietary transformation URLs | knowledge-based — not fetched this run; figures from model knowledge |

### Email & Messaging Delivery

Serves transactional SMS (opt-in, STOP handling, 8am–9pm timing, delivery status) and email fallback (FEAT-08.SPEC-012, FEAT-08.SPEC-013, FEAT-14.SPEC-004; ASMP-24, ASMP-29, ASMP-32) across 24 Notification specs; sign-in codes (FEAT-29.SPEC-014); WhatsApp later (FEAT-26.SPEC-002).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Twilio (Programmable Messaging and WhatsApp) | Managed service (SMS, WhatsApp) | SMS with delivery status, inbound STOP replies, A2P 10DLC registration; WhatsApp channel through the same provider | SMS $0.0083/segment plus carrier fees and 10DLC onboarding fees; WhatsApp $0.005/message plus Meta template fees from $0.0034 | Low — SDKs and webhooks | Very mature; number and 10DLC registrations tied to the account | https://www.twilio.com/en-us/sms/pricing/us and https://www.twilio.com/en-us/whatsapp/pricing (accessed 2026-09-29) |
| Telnyx | Managed service (SMS) | US SMS with 10DLC support, delivery reports | $0.004 per part plus carrier passthrough | Low–medium — REST API and SDKs | Mature; number registrations tied to account | https://telnyx.com/pricing/messaging (accessed 2026-09-29) |
| Resend | Managed email API | Transactional email for the fallback and code emails; Node SDK | Free 3,000/mo (100/day); Pro $20–35/mo for 50,000–100,000 | Low | Newer; SMTP limits lock-in | https://resend.com/pricing (accessed 2026-09-29) |
| Postmark | Managed email API | Transactional-first email with delivery-status tracking | Free 100/mo; from $15/mo for 10,000 | Low | Long-established | https://postmarkapp.com/pricing (accessed 2026-09-29) |
| Amazon SES with SNS | Cloud email service | Low-cost email with bounce/complaint notifications; SMS via separate AWS service | Per-1,000 email pricing | Medium — IAM and sender verification | Very mature; AWS coupling | knowledge-based — not fetched this run; figures from model knowledge |

### Payments & Billing

Serves deposit card charges with payout routing to the Pro, refunds, balance payments, tips, connected-account identity/bank verification, dispute notices, and Pro subscription billing (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-16.SPEC-003, FEAT-18.SPEC-006; ASMP-31); BRIEF.md: "Established card payment processor: takes client deposits and pays out to the pro, and owns all card data. Also used for the pro's subscription billing." Idempotency and outcome consistency (FEAT-07.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Stripe (Connect plus Billing) | Managed payment platform | Connected accounts with hosted identity/bank onboarding, destination charges, refunds, dispute webhooks, subscriptions | Card 2.9% + 30¢; Connect account fee $2 per monthly active account (platform-handled pricing) and payouts 0.25% + 25¢ | Low — SDKs in major languages; idempotency keys | Very mature; card data and vault tokens tied to Stripe | https://stripe.com/connect/pricing (accessed 2026-09-29) |
| Adyen for Platforms | Managed payment platform | Marketplace onboarding, split payments, payouts and disputes | Interchange-plus with per-transaction fee; enterprise sales process | Medium–high | Very mature; enterprise-oriented | knowledge-based — not fetched this run; figures from model knowledge |
| PayPal Complete Payments (Braintree/PayPal marketplaces) | Managed payment platform | Marketplace payments with seller onboarding and payouts | Per-transaction percentage plus fixed fee | Medium | Very mature; onboarding coverage varies by market | knowledge-based — not fetched this run; figures from model knowledge |
| Square (Payments and Connect) | Managed payment platform | Card payments with OAuth-based seller accounts | Per-transaction percentage plus fixed fee | Medium — seller accounts authorize the platform | Mature; seller-account model differs from platform-held funds | knowledge-based — not fetched this run; figures from model knowledge |

### Search

Serves client list search by match and filter (FEAT-24.SPEC-001, FEAT-24.SPEC-002) over 100–500 clients per pro (ASMP-22); no full-text or faceted search is specified.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostgreSQL built-ins (ILIKE, pg_trgm, full-text search) | Database feature | Match and filter over a per-pro record set with no extra service | Included in database cost | Low — SQL queries and indexes | Postgres-standard | knowledge-based — not fetched this run; figures from model knowledge |
| Typesense | Open-source engine / managed cloud | Typo-tolerant instant search | Self-hosted free; cloud priced via calculator (not retrieved) | Medium — data sync | Open-source; self-host exit | knowledge-based — Typesense pricing URL redirected to a calculator page not fetched this run |
| Meilisearch | Open-source engine / managed cloud | Typo-tolerant search with filters | Cloud from $20/mo; self-hosting free | Medium — data sync | Open-source; self-host exit | https://www.meilisearch.com/pricing (accessed 2026-09-29) |
| Algolia | SaaS platform (hosted search) | Hosted instant search; per-pro data isolation requires secured API keys | Free 10K requests/mo, 50K records; Grow $0.50 per additional 1K requests | Low–medium | Mature; proprietary, records must be synced | https://www.algolia.com/pricing (accessed 2026-09-29) |
| OpenSearch / Elasticsearch | Self-hosted or managed engine | Full-text and faceted search; heavy for the stated need | Managed cluster hourly pricing | High | Mature; operationally heavy | knowledge-based — not fetched this run; figures from model knowledge |

### Background Jobs & Scheduling

Serves 31 scheduled or timed Automation specs: slot hold expiry (FEAT-03.SPEC-003), reminder scheduling within timing windows (FEAT-08.SPEC-007), subscription renewals (FEAT-18.SPEC-003), auto-completion sweep (FEAT-12.SPEC-004), sync reconciliation (FEAT-04.SPEC-006), data export (FEAT-29.SPEC-007), retry semantics for messages and refunds (FEAT-08.SPEC-009, FEAT-09.SPEC-006).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Inngest | Managed workflow/job service | Durable step functions, delayed events, retries, cron | Free 50k executions/mo; Pro from $99/mo (1M executions) | Low — SDK and HTTP endpoint | Newer; step model is vendor-specific | https://www.inngest.com/pricing (accessed 2026-09-29) |
| Trigger.dev | Managed job platform (open-source core) | Long-running tasks, scheduling, retries | Free $5 credit; Hobby $10/mo; Pro $50/mo; per-run $0.000025 plus compute | Low–medium | Newer; open-source core allows self-hosting | https://trigger.dev/pricing (accessed 2026-09-29) |
| Postgres-native queue and cron (pg_cron with pgmq or similar) | Database-resident scheduler/queue | Jobs in same transactional store as bookings; no extra service | Included in database cost | Medium — polling worker or edge function | Postgres-standard | knowledge-based — not fetched this run; figures from model knowledge |
| BullMQ with Redis | Library plus self/managed Redis | Delayed and repeatable jobs with retries | Open source; Redis hosting cost (Upstash: https://upstash.com/pricing/redis) | Medium — needs a long-running worker | Mature; Node-specific | https://upstash.com/pricing/redis (accessed 2026-09-29) |
| Platform cron (Vercel Cron Jobs) | Platform feature | Scheduled HTTP invocations for sweeps; no built-in retry or queue | Cron included on Hobby, Pro and Enterprise | Low | Coupled to host; limited durability | https://vercel.com/pricing (accessed 2026-09-29) |

### Caching & Performance

Serves slot lists within roughly one second (ASMP-21), growth in per-pro history (ASMP-22), and read-only offline posture for the most recently loaded schedule, client list and money list (ASMP-27).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Database indexing and query design (no extra cache layer) | Database practice | Booking data per pro is small; indexed queries can serve slot computation | Included in database cost | Low | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Upstash Redis | Managed service (serverless Redis) | Short-lived cache, rate limiting and hold counters | Free 256 MB; pay-as-you-go $0.20 per 100K commands; fixed $10–$1,500/mo | Low — REST and Redis clients | Redis protocol limits lock-in | https://upstash.com/pricing/redis (accessed 2026-09-29) |
| Client-side cache (TanStack Query persistence, service worker) | Client library | Keeps last-loaded schedule readable offline | Free | Medium — persistence and invalidation | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| CDN/edge caching of public booking page (framework cache, Vercel or Cloudflare) | Platform feature | Fast public page shell; slot data still uncached | Included in hosting plan | Low | Platform-specific cache controls | https://vercel.com/pricing (accessed 2026-09-29) |
| Self-managed Redis / Amazon ElastiCache | Managed or self-hosted | General cache and locks | Instance-hour pricing | Medium — provisioning | Redis standard | knowledge-based — not fetched this run; figures from model knowledge |

### Real-time & Collaboration

Serves slot list refresh about every second while viewing (FEAT-03.SPEC-001), banners and notices updating without refresh (FEAT-08.SPEC-005, FEAT-28.SPEC-002); concurrent writes with last-write-wins, reject-with-refresh and first-committed-wins (feature-dependency-map.md, XBR-01). No WebSocket or collaborative editing is named in the profile.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Polling with TanStack Query / SWR | Client technique | Meets the roughly one-second refresh without extra infrastructure; load scales with viewers | No service cost | Low | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Supabase Realtime | Managed service | Postgres change broadcast and channels | Free 200 connections / 2M messages; Pro 500 connections / 5M messages | Low with Supabase | Coupled to Supabase | https://supabase.com/pricing (accessed 2026-09-29) |
| Ably | Managed service (pub/sub) | Channels and presence at scale | Free 200 connections; Standard $29/mo plus usage ($2.50/M messages) | Low–medium | Proprietary protocol with SDKs | https://ably.com/pricing (accessed 2026-09-29) |
| Pusher Channels | Managed service (pub/sub) | Simple channel push | Free tier plus plans by connections | Low | Proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Server-Sent Events over own server | Self-managed | One-way pushes from a long-running backend | Server cost | Medium — needs a stateful runtime | Standard web protocol | knowledge-based — not fetched this run; figures from model knowledge |

### Analytics & Product Telemetry

Serves the product's success metrics at a few hundred pros in year one (ASMP-22) and the one-minute booking and one-second slot benchmarks (ASMP-21); privacy posture limits client data flows (ASMP-23).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostHog | SaaS platform (open-source core) | Event analytics, funnels, session replay, feature flags | Free 1M events/mo; usage-based beyond | Low | Self-hosting available | https://posthog.com/pricing (accessed 2026-09-29) |
| Mixpanel | SaaS platform | Funnels and retention | Free 1M events/mo; Growth up to 20M events | Low | Proprietary | https://mixpanel.com/pricing/ (accessed 2026-09-29) |
| Amplitude | SaaS platform | Product analytics and funnels | Free tier plus paid plans | Low | Proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Google Analytics 4 | SaaS platform | Web traffic and funnels; privacy-consent overhead | Free | Low | Google-hosted | knowledge-based — not fetched this run; figures from model knowledge |
| Plausible Analytics | SaaS platform / self-hosted | Privacy-friendly traffic metrics; limited product funnels | Paid subscription by pageviews; self-hosting option | Low | Open-source core | knowledge-based — not fetched this run; figures from model knowledge |

### Internationalization

Serves per-account timezone and currency, never hard-coded (ASMP-25; FEAT-27.SPEC-003, FEAT-27.SPEC-008, FEAT-07.SPEC-001, FEAT-22.SPEC-001), with US first then UK, Canada, Australia; no multi-language behavior is specified.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Native Intl APIs (Intl.NumberFormat, Intl.DateTimeFormat) | Web platform standard | Currency and timezone formatting with no dependency | Free | Low | Standard | knowledge-based — not fetched this run; figures from model knowledge |
| date-fns with date-fns-tz | Library | Timezone-aware date arithmetic | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| Luxon | Library | Timezone-aware date/time with IANA zones | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| next-intl | Library (React/Next.js i18n) | Message catalogs plus formatting for later language support | Open source (MIT) | Low–medium | Mature; Next.js-specific | knowledge-based — not fetched this run; figures from model knowledge |
| i18next / react-i18next | Library | Framework-agnostic translations and plugins | Open source (MIT) | Medium | Very mature | knowledge-based — not fetched this run; figures from model knowledge |

### Calendar Sync

Serves two-way sync with a Pro's Google Calendar and Apple Calendar: busy time blocks availability and bookings appear there (FEAT-04.SPEC-003, FEAT-03.SPEC-006; ASMP-33), with sync health monitoring and degraded mode.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Direct integration (Google Calendar API plus CalDAV for iCloud) | Provider APIs, own code | Full control; Google deprecated CalDAV for third parties in 2019 so two protocols to build; iCloud via CalDAV | Free API usage | High — two protocols, push/watch channels, reconciliation | No vendor; more maintenance | https://www.nylas.com/blog/best-calendar-apis/ (accessed 2026-09-29) |
| Cronofy | Managed unified calendar API | Real-time sync and free/busy across Google, Microsoft and Apple | Base API from $819/month | Low–medium | Mature; vendor holds calendar tokens | https://www.cronofy.com/pricing and https://www.cronofy.com/blog/best-calendar-apis (accessed 2026-09-29) |
| Nylas | Managed unified calendar and email API | Reads and writes Google, Outlook, Exchange and iCloud (CalDAV wrapped behind JSON API) | Free; Essentials $15/mo; Pro $49/mo; per-account overage $1.35–$2.25 (calendar-only $1.35–$1.70) | Low–medium | Mature | https://www.nylas.com/pricing/ and https://developer.nylas.com/docs/cookbook/calendar/apple-calendar-api/ (accessed 2026-09-29) |
| OneCal Unified Calendar API | Managed unified calendar API | Unified API across calendar providers | Not retrieved | Low–medium | Smaller vendor | https://www.onecal.io/unified-calendar-api (accessed 2026-09-29, listed in search results, page not fetched) |
| Truto unified API | Managed unified API platform | Unified API covering Google Calendar, Outlook and Apple | Not retrieved | Medium | Smaller vendor | https://truto.one/blog/unified-api-for-google-calendar-outlook-and-apple-2026-architecture-guide/ (accessed 2026-09-29, listed in search results, page not fetched) |

## 3. Cross-Area Compatibility Notes

- Prisma, Drizzle and Kysely (ORM / Data Access) are TypeScript/Node data-access layers; they pair with the Node/TypeScript candidates in Backend / API Layer (Next.js server layer, Hono, NestJS), not with FastAPI. SQLAlchemy pairs with FastAPI, not with Node backends. Rails uses ActiveRecord.
- Every Postgres candidate in Database (Supabase, Neon, Crunchy Bridge, RDS/Aurora, self-hosted) exposes standard Postgres drivers supported by Prisma, Drizzle, Kysely and SQLAlchemy. supabase-js/PostgREST works only with Supabase.
- Supabase Auth, Storage and Realtime share one project with Supabase Postgres; row-level-security policies apply across them. Using them with another database splits identity and data.
- Clerk, WorkOS AuthKit, Better Auth/Auth.js and Auth0 keep identity separate from the database, so per-pro isolation is enforced in application code or database policies keyed to their user IDs.
- SWR and Redux Toolkit (State Management) are React-only; TanStack Query has adapters for React, Vue and Svelte; Zustand is React-oriented. Next.js is React; Nuxt is Vue; SvelteKit is Svelte.
- shadcn/ui and Radix UI assume React; shadcn/ui also assumes Tailwind CSS. Headless UI supports React and Vue. Tailwind CSS and CSS Modules pair with every Frontend Framework candidate.
- Vercel is the native host for Next.js; Next.js, React Router v7, SvelteKit, Nuxt and Astro all have adapters for other hosts. Cloudflare Workers uses a non-Node runtime, so Node-only libraries (some ORMs, BullMQ) may need compatible drivers.
- Vercel Cron Jobs (Background Jobs & Scheduling) run only on Vercel; Inngest and Trigger.dev call into an HTTP endpoint or run tasks on their own compute, so work with any host in Hosting & Environments.
- BullMQ requires a long-running worker process and Redis (Upstash, ElastiCache or self-managed); serverless-only hosting does not run it.
- Stripe, Twilio, Telnyx, Resend, Postmark, Cronofy and Nylas provide official SDKs or REST/webhook APIs usable from Node/TypeScript and Python; Adyen, PayPal and Square also have SDKs for both.
- Twilio provides both SMS and WhatsApp under one account (FEAT-26 channel), while Telnyx, Resend and Postmark cover a single channel each. Inbound STOP replies (FEAT-14.SPEC-004) and delivery-status callbacks require a public webhook endpoint in Backend / API Layer for every messaging candidate.
- Stripe Connect webhooks (disputes, payout status, renewals) and refund retry rely on idempotent handlers (FEAT-07.SPEC-004, FEAT-09.SPEC-006); every Backend / API Layer candidate can host them, and Background Jobs & Scheduling candidates can supply retries.
- Google deprecated CalDAV for third-party apps in 2019, so direct integration for Calendar Sync needs the Google Calendar API for Google and CalDAV for Apple; Cronofy and Nylas normalize both.
- Postgres exclusion constraints and transactions (Database candidates) can enforce non-overlapping slots at the data layer independent of the Real-time & Collaboration transport chosen.
- Supabase Realtime uses Postgres change streams from Supabase Postgres only; Ably and Pusher are database-independent.

## 4. Research Log

Every row marked `knowledge-based` in Section 2 was not fetched in this run: the pass fetched a bounded set of vendor pricing pages and covered the remaining candidates from model knowledge, with each affected Sources cell marked. Two fetches failed or were incomplete (Render pricing page returned navigation only; Better Stack returned HTTP 503), and two redirects (Typesense, Fly.io) were not followed.

| Decision Area | Method | Queries & Key Sources | Access Date |
|---|---|---|---|
| Frontend Framework | knowledge-based — web fetches were spent on pricing-bearing areas; open-source frameworks have no pricing pages | Candidates from model knowledge; no URLs cited | — |
| Backend / API Layer | knowledge-based — open-source frameworks; no pricing pages fetched | Candidates from model knowledge; no URLs cited | — |
| Database | web (one knowledge-based row for RDS and self-hosted) | supabase.com/pricing; neon.com/pricing; crunchydata.com/pricing; RDS and self-hosted rows knowledge-based | 2026-09-29 |
| ORM / Data Access | knowledge-based (one web-sourced row) — open-source libraries | supabase.com/pricing for supabase-js row; others from model knowledge | 2026-09-29 |
| CSS / Styling | knowledge-based — open-source libraries | No URLs cited | — |
| State Management | knowledge-based — open-source libraries | No URLs cited | — |
| Build Tooling | knowledge-based — open-source tooling | No URLs cited | — |
| Authentication & Identity | web (two knowledge-based rows) | supabase.com/pricing; clerk.com/pricing; workos.com/pricing; Better Auth/Auth.js and Auth0 from model knowledge | 2026-09-29 |
| Hosting & Environments | web (three knowledge-based rows) | vercel.com/pricing; Render pricing page fetch returned navigation content only; Fly.io fetch redirected to docs.fly.io and was not completed; AWS and Cloudflare from model knowledge | 2026-09-29 |
| CI/CD & Delivery | web (three knowledge-based rows) | docs.github.com billing page for Actions; vercel.com/pricing; GitLab, CircleCI and Buildkite from model knowledge | 2026-09-29 |
| Observability & Operations | web (four knowledge-based rows) | sentry.io/pricing; Better Stack pricing returned HTTP 503; Datadog, Grafana Cloud, OpenTelemetry/Honeycomb from model knowledge | 2026-09-29 |
| File & Object Storage | web (three knowledge-based rows) | supabase.com/pricing; cloudflare.com R2 product page (egress only, no storage price); S3, Vercel Blob, Cloudinary from model knowledge | 2026-09-29 |
| Email & Messaging Delivery | web (one knowledge-based row) | twilio.com SMS and WhatsApp pricing; telnyx.com messaging pricing; resend.com/pricing; postmarkapp.com/pricing; Amazon SES from model knowledge | 2026-09-29 |
| Payments & Billing | web (three knowledge-based rows) | stripe.com/connect/pricing; Adyen, PayPal, Square from model knowledge | 2026-09-29 |
| Search | web (three knowledge-based rows) | algolia.com/pricing; meilisearch.com/pricing; Typesense pricing URL redirected to cloud.typesense.org calculator, not fetched; Postgres built-ins and OpenSearch from model knowledge | 2026-09-29 |
| Background Jobs & Scheduling | web (one knowledge-based row) | inngest.com/pricing; trigger.dev/pricing; upstash.com/pricing/redis; vercel.com/pricing; Postgres-native queue from model knowledge | 2026-09-29 |
| Caching & Performance | web (three knowledge-based rows) | upstash.com/pricing/redis; vercel.com/pricing; other rows from model knowledge | 2026-09-29 |
| Real-time & Collaboration | web (three knowledge-based rows) | supabase.com/pricing; ably.com/pricing; polling, Pusher, SSE from model knowledge | 2026-09-29 |
| Analytics & Product Telemetry | web (three knowledge-based rows) | posthog.com/pricing; mixpanel.com/pricing; Amplitude, GA4, Plausible from model knowledge | 2026-09-29 |
| Internationalization | knowledge-based — open-source libraries and web standards | No URLs cited | — |
| Calendar Sync | web | Search "two-way calendar sync API Google Calendar and Apple iCloud CalDAV support Cronofy Nylas"; cronofy.com/pricing; nylas.com/pricing; result URLs from that search (nylas.com/blog/best-calendar-apis, cronofy.com/blog/best-calendar-apis, developer.nylas.com Apple cookbook, onecal.io, truto.one); OneCal and Truto pricing not retrieved | 2026-09-29 |

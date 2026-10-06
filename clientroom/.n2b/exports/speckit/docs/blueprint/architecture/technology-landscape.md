---
document_type: technology-landscape
produced_by: technical-researcher
status: final
stage: 4
created: 2026-09-29
area_count: 22
option_count: 89
---

# Technology Landscape

## 1. Research Scope

Derived mechanically from `technical-profile.md`: 11 always-active areas, 9 signal-activated areas (AI & Intelligent Behavior and Geo & Maps stay inactive: AI/ML behavior = No, Geo/maps = No), and 2 extension areas triggered textually by profile Section 7.

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
| File & Object Storage | File upload (FEAT-06.SPEC-001, FEAT-06.SPEC-003, FEAT-16.SPEC-002, FEAT-16.SPEC-007, FEAT-17.SPEC-001; ASMP-22, ASMP-30); Import/export (FEAT-22.SPEC-002, FEAT-24.SPEC-003) |
| Email & Messaging Delivery | Notifications (email/push/SMS) (26 Notification specs, e.g. FEAT-02.SPEC-011, FEAT-05.SPEC-008; FEAT-14.SPEC-001, FEAT-14.SPEC-002; ASMP-29) |
| Payments & Billing | Payments/billing (FEAT-10.SPEC-003, FEAT-23.SPEC-003, FEAT-32.SPEC-002, FEAT-09.SPEC-001 to FEAT-09.SPEC-010; ASMP-28, ASMP-31) |
| Search | Search (FEAT-28.SPEC-001, FEAT-28.SPEC-002, FEAT-28.SPEC-003, FEAT-28.SPEC-004) |
| Background Jobs & Scheduling | Background processing (FEAT-11.SPEC-001, FEAT-12.SPEC-004, FEAT-14.SPEC-003, FEAT-24.SPEC-003, FEAT-24.SPEC-005); Import/export (FEAT-22.SPEC-002, FEAT-24.SPEC-003); Notifications (email/push/SMS) (FEAT-14.SPEC-002) |
| Caching & Performance | Scale hints (ASMP-21, ASMP-22, ASMP-26; BRIEF.md ## Constraints budget); Offline (FEAT-07.SPEC-008, FEAT-29.SPEC-004; ASMP-27) |
| Real-time & Collaboration | Collaboration/concurrency (FEAT-03.SPEC-003, FEAT-08.SPEC-003, FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-11.SPEC-001, FEAT-18.SPEC-006; Real-time = No) |
| Analytics & Product Telemetry | Scale hints (ASMP-22; BRIEF.md ## Scale & Non-Functional Expectations) |
| Internationalization | Internationalization (FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-006, FEAT-15.SPEC-008, FEAT-22.SPEC-003) |
| Electronic Signature Attestation | Product-mandated — feature-dependency-map.md External Touchpoints: "Electronic-signature attestation for legally binding proposal signing — v1 phase ... FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability — submits the signature data captured by FEAT-26.SPEC-002 for attestation ... the jurisdictional standard it must meet is left to Stage 4)" |
| Custom Domain Verification & TLS Serving | Product-mandated — assumptions-constraints.md ASMP-32: "Domain-verification capability (Later phase) — Custom Domain per Freelancer (FEAT-27) requires the ability to verify that a freelancer controls a domain and serve her portal securely at it." (FEAT-27.SPEC-002) |

## 2. Decision Area Landscapes

### Frontend Framework

Serves 18 user-facing features and 65 Screen specs (profile Section 1), a mobile-first client portal with ~2 s interactivity (ASMP-21), and forms of at most 7 fields (Complex forms = No).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Next.js (App Router, v16) | React meta-framework | SSR/RSC for fast mobile first paint; server actions for form flows; largest hiring pool and ecosystem | Open source (MIT); free to self-host | Low — largest ecosystem of integrations and SDK examples | Very mature; softest lock-in is to Vercel-specific features (self-hosting supported) | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| React Router v7 (Remix lineage) | React framework | Loader/action model suited to form-rich, data-heavy flows and progressive enhancement | Open source (MIT) | Low–medium — smaller ecosystem of turnkey templates than Next.js | Mature; Remix merged into React Router v7; runtime-agnostic | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| SvelteKit (Svelte 5) | Svelte meta-framework | Small bundles and strong runtime performance for mobile clients; form actions built in | Open source (MIT) | Medium — smaller component and SDK ecosystem; different mental model from React | Mature; adapters for many hosts; Svelte-specific code does not port to other frameworks | https://prismic.io/blog/sveltekit-vs-nextjs (accessed 2026-09-29) |
| Nuxt 4 | Vue meta-framework | Full SSR/hybrid rendering with Vue ecosystem; mature i18n and form modules | Open source (MIT) | Medium — Vue-specific ecosystem | Mature; Vue-specific code does not port | https://medium.com/@aryavr2030/next-js-vs-nuxt-vs-remix-which-meta-framework-should-you-learn-in-2026-77c5eb8d3c86 (accessed 2026-09-29) |

### Backend / API Layer

Serves 62 Automation specs, 7 Integration specs (webhooks from payments/email/storage) and role-based portals (profile Sections 1 and 3); no real-time transport needed.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework server layer (Next.js route handlers and server actions) | Framework-bundled backend | Covers request/response, webhooks and form mutations in the same deployable as the frontend; long-running work must go to a job runner | Included in framework hosting cost | Low — no separate service | Tied to the frontend framework's runtime and hosting model | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| Hono on Node.js / edge runtimes | TypeScript API framework | Lightweight API routing with middleware, portable across Node, Workers and Bun | Open source (MIT) | Low — TypeScript, works with Drizzle/Kysely | Younger but widely adopted; runtime-portable | knowledge-based — no Hono-specific page fetched this run; characteristics from model knowledge |
| NestJS (Node.js) | TypeScript backend framework | Opinionated modules, DI, guards for role/authorization rules and scheduled tasks | Open source (MIT) | Medium — framework conventions to learn | Mature; large enterprise use | knowledge-based — no NestJS-specific page fetched this run; characteristics from model knowledge |
| Django + Django REST Framework (Python) | Python full-stack framework | Batteries-included ORM, admin, auth, migrations; strong for record-heavy systems | Open source (BSD) | Medium — separate Python service and separate frontend | Very mature; Python-side data-access tooling only | knowledge-based — no Django-specific page fetched this run; characteristics from model knowledge |

### Database

Serves 23 entities, 60 relationships, 35 cross-feature rules, immutable financial records (ASMP-25) and contention on 12 entities (profile Sections 2, 3); infrastructure budget under roughly $100/month (BRIEF.md ## Constraints).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Neon (serverless Postgres) | Managed Postgres | Relational transactions, row-level constraints, branching for environments; scale-to-zero | Free tier; Launch $0.106/CU-hour; Scale $0.222/CU-hour; storage $0.35/GB-month | Low — standard Postgres drivers | Standard Postgres; exit by dump/restore | https://neon.com/pricing (accessed 2026-09-29) |
| Supabase Postgres | Managed Postgres platform | Postgres plus bundled auth, storage and realtime; row-level security for client isolation | Pro $25/month incl. 8 GB DB, 250 GB egress; $0.125/GB beyond 8 GB | Low — Postgres wire protocol plus optional SDK | Standard Postgres; platform features add coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Amazon RDS for PostgreSQL | Cloud-managed Postgres | Production Postgres with Multi-AZ, backups and read replicas | Instance-hour plus storage; small instances from roughly tens of USD/month | Medium — VPC, IAM and parameter setup | Very mature; Postgres portability; ties to AWS networking | knowledge-based — AWS RDS pricing page not fetched this run; figures from model knowledge |
| Render Managed Postgres | Managed Postgres | Fixed-size instances with predictable billing | From $40/month for 1 vCPU / 2 GB | Low — connection string | Standard Postgres; exit by dump/restore | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |

### ORM / Data Access

Serves a relational model of 23 entities with reject-with-refresh concurrency and an immutable-record model (profile Sections 2, 3) — needs transactions, migrations and typed queries.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Drizzle ORM (v1) | TypeScript ORM / query builder | SQL-close typed queries; built-in migration generation (drizzle-kit); works in edge runtimes | Open source (Apache-2.0) | Low — TypeScript-native schema | v1 stable; code-first schema; TypeScript-only | https://www.pkgpulse.com/guides/drizzle-orm-v1-vs-prisma-6-vs-kysely-2026 (accessed 2026-09-29) |
| Prisma ORM (v6) | TypeScript ORM | Schema-first with generated client, Prisma Studio and Prisma Migrate; wide database support | Open source (Apache-2.0); optional paid Accelerate/Postgres services | Low — own schema language and codegen | Very mature; proprietary schema language; edge runtimes need Accelerate | https://makerkit.dev/blog/tutorials/drizzle-vs-prisma (accessed 2026-09-29) |
| Kysely | TypeScript query builder | Type-safe SQL builder for CTEs and window functions (financial aggregates); no migrations built in beyond a basic migrator | Open source (MIT) | Medium — schema types and migrations managed separately | Mature; thin abstraction, low lock-in | https://www.pkgpulse.com/guides/drizzle-orm-v1-vs-prisma-6-vs-kysely-2026 (accessed 2026-09-29) |
| SQLAlchemy + Alembic | Python ORM and migration tool | Full-featured ORM with mature migrations; Python backend only | Open source (MIT) | Medium — pairs only with a Python backend | Very mature; Python-only | knowledge-based — no SQLAlchemy page fetched this run; characteristics from model knowledge |

### CSS / Styling

Serves mobile-first client screens and freelancer-supplied brand colours/logo that must stay legible (FEAT-19.SPEC-001, FEAT-19.SPEC-003; ASMP-27), requiring token-driven theming.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Tailwind CSS v4 | Utility-first CSS framework | Fast builds (Oxide engine); CSS-variable theme tokens map to per-freelancer brand colours | Open source (MIT) | Low — first-class in all mainstream frameworks | Very mature; utility classes in markup are a migration cost | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |
| CSS Modules | Scoped CSS files | Plain CSS with local scope; tokens as CSS custom properties; zero runtime | Built into major bundlers | Low — no extra dependency | Very mature; web-standard, minimal lock-in | https://www.pkgpulse.com/guides/css-modules-vs-tailwind-2026 (accessed 2026-09-29) |
| vanilla-extract | Build-time TypeScript CSS | Type-safe tokens and themes compiled to static CSS | Open source (MIT) | Medium — bundler plugin per framework | Mature; TypeScript-authored styles | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |
| Panda CSS | Build-time CSS-in-JS | Type-safe tokens, variants and conditions; static extraction | Open source (MIT) | Medium — codegen step | Newer; smaller ecosystem (about 300K weekly downloads per source) | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |

Component-layer candidates (profile evidence: data tables in invoice/deliverable lists, dialogs for confirmations, no command palette; no user-supplied design system named in profile):
- Radix UI primitives (headless, React)
- shadcn/ui (copy-in components on Radix and Tailwind, React)
- Headless UI (headless, React and Vue)
- Melt UI (headless, Svelte)

### State Management

Serves mostly server-derived state (invoices, milestones, comments) with snapshot screens (Real-time = No), reject-with-refresh concurrency, and an offline comment queue (FEAT-07.SPEC-008).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| TanStack Query v5 | Server-state cache library | Fetching, caching, invalidation, retries and offline mutation persistence; multi-framework adapters | Open source (MIT) | Low | Very mature; server-state only | https://saschb2b.com/blog/react-state-management-2026 (accessed 2026-09-29) |
| Zustand | Client-store library (React) | Minimal store for UI state and the offline comment queue | Open source (MIT) | Low | Mature; React-only | https://dev.to/digitalunicon/state-management-in-2026-redux-vs-zustand-vs-react-context-5336 (accessed 2026-09-29) |
| Redux Toolkit (with RTK Query) | State store and data-fetching library (React) | Explicit action flow and RTK Query caching for interdependent client state | Open source (MIT) | Medium — slices, store setup | Very mature; React/Redux ecosystem | https://dev.to/digitalunicon/state-management-in-2026-redux-vs-zustand-vs-react-context-5336 (accessed 2026-09-29) |
| SWR | Data-fetching hook library (React) | Stale-while-revalidate caching for read-mostly screens | Open source (MIT) | Low | Mature; React-only, fewer mutation features | https://saschb2b.com/blog/react-state-management-2026 (accessed 2026-09-29) |

### Build Tooling

Serves a 33-feature web app on a single deployable; framework-bundled pipeline is the default (guide Section 5).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Turbopack (bundled with Next.js 16) | Framework-bundled bundler | Stabilized for production builds in Next.js 16 | Included with Next.js (open source) | None — bundled | Newer than webpack; tied to Next.js | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| Vite | Bundler / dev server | Default pipeline for React Router, SvelteKit and Nuxt; fast HMR | Open source (MIT) | Low | Very mature; plugin-based, low lock-in | knowledge-based — no Vite page fetched this run; characteristics from model knowledge |
| pnpm (package manager) | Package manager | Fast, disk-efficient installs; workspace support | Open source (MIT) | Low | Mature; lockfile format is pnpm-specific | knowledge-based — no pnpm page fetched this run; characteristics from model knowledge |
| Bun (package manager and bundler) | JS runtime and toolchain | All-in-one installer, bundler and test runner | Open source (MIT) | Low–medium — runtime compatibility to verify | Younger; runtime-specific behavior differences | knowledge-based — no Bun page fetched this run; characteristics from model knowledge |

### Authentication & Identity

Serves email magic-link sign-in for client contacts (FEAT-05.SPEC-001 to FEAT-05.SPEC-007), freelancer sign-up and login (FEAT-20.SPEC-001, FEAT-21.SPEC-003), sign-out of other sessions (FEAT-21.SPEC-006), four roles, and strict client isolation.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Clerk | Managed identity provider | Magic links, sessions and multi-session management; hosted UI components | Free to 50K monthly retained users; then $25/month plus per-MRU overage | Low — SDKs and prebuilt components | Mature; users held in vendor store, export possible | https://clerk.com/pricing (accessed 2026-09-29) |
| Better Auth | Open-source auth library | Magic-link plugin, 2FA, organizations; runs in own database | Open source (MIT); infrastructure cost only | Medium — own the sessions and tables | Newer; low lock-in (data in own DB) | https://dev.to/thiago_alvarez_a7561753aa/clerk-vs-better-auth-2026-we-verified-every-price-so-you-dont-have-to-13pk (accessed 2026-09-29) |
| Supabase Auth | Managed auth (open-source GoTrue) | Magic links, social login, MFA; ties to Postgres row-level security | Free 50,000 MAU; Pro $25/month incl. 100,000 MAU; $0.00325/MAU beyond | Low — SDK; best with Supabase Postgres | Open-source GoTrue gives an exit path | https://www.buildmvpfast.com/api-costs/authentication (accessed 2026-09-29) |
| WorkOS AuthKit | Managed identity provider | Passwordless and social login; enterprise SSO modules | Free to 1M MAU; SSO from $125/month per connection | Low — SDKs | Mature; hosted user store | https://www.buildmvpfast.com/api-costs/authentication (accessed 2026-09-29) |

### Hosting & Environments

Serves a web app with mobile-first clients worldwide (BRIEF.md Geography), webhook endpoints, scheduled work, and an infrastructure budget under roughly $100/month; no uptime target (ASMP-26).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vercel | Managed frontend/serverless platform | Native Next.js hosting, preview environments per branch, CDN, cron | Pro $20/seat/month with $20 usage credit, 1 TB transfer; Hobby is non-commercial only | Low — Git-based deploys | Mature; platform-specific features raise coupling | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |
| Fly.io | Container/VM platform | Long-running containers, regions near users, no cold starts | Per-second machine billing; no free tier; sample Node+Postgres stack ~$12/month | Medium — CLI-first, Dockerfile | Mature; standard containers, low lock-in | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |
| Railway | Managed app platform | Git deploys, one-click Postgres and workers | Hobby $5/month, Pro $20/month plus usage; sample stack ~$37/month | Low | Younger platform; standard containers | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |
| Render | Managed app platform | Web services, cron jobs, background workers, managed Postgres | Fixed instance prices plus workspace fee; sample stack ~$30/month | Low | Mature; standard containers | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |

### CI/CD & Delivery

Serves a 33-feature codebase with financial-correctness rules that require automated checks before promotion (ASMP-25) and multiple environments.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| GitHub Actions | Hosted CI/CD | Native to GitHub repos; environments with approval gates | Private repos: 2,000 min/month Free, 3,000 Team; Linux $0.006/min | Low | Very mature; workflow YAML is GitHub-specific | https://docs.github.com/billing/managing-billing-for-github-actions/about-billing-for-github-actions (accessed 2026-09-29) |
| GitLab CI/CD | Hosted CI/CD | Integrated pipelines and environments in GitLab | Free 400 min/month; Premium from $29/user/month | Medium — requires GitLab hosting of code | Very mature; pipeline syntax is GitLab-specific | https://www.eesel.ai/blog/gitlab-pricing (accessed 2026-09-29) |
| CircleCI | Hosted CI/CD | Fast parallel pipelines, reusable orbs | Free 6,000 credits/month; Performance credits $15 per 25,000 | Low–medium | Mature; config is CircleCI-specific | https://circleci.com/pricing/ (accessed 2026-09-29) |
| Platform-native deploy pipelines (e.g. Vercel / Render Git deploys) | Hosting-bundled CD | Build-and-deploy on push with preview environments; tests still need a separate runner | Included with hosting plan | Low | Tied to the hosting platform | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |

### Observability & Operations

Serves failed-email visibility within minutes (ASMP-26), an immutable audit trail (FEAT-13) and GDPR-class data in logs (Compliance/privacy = Yes) on a small budget.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Sentry | SaaS error and performance monitoring | Error reporting, tracing and replay; framework SDKs | Team $26/month incl. 50K errors; tiered overage from $0.0003625/error | Low — SDK per framework | Very mature; SDK is Sentry-specific, self-hosting available | https://blog.struct.ai/sentry-pricing-error-monitoring-2026/ (accessed 2026-09-29) |
| Grafana Cloud | SaaS metrics, logs and traces | Loki logs, Prometheus metrics, alerting in one bill | Logs from ~$0.50/GB ingested; free tier available | Medium — OpenTelemetry or agent setup | Mature; open-source components limit lock-in | https://kanopylabs.com/blog/axiom-vs-datadog-vs-grafana-cloud (accessed 2026-09-29) |
| Datadog | SaaS observability suite | Infra, APM and logs in one product | Infrastructure $15/host/month, APM $31/host/month, logs $0.10/GB | Medium | Very mature; proprietary agent and dashboards | https://blog.railway.com/p/best-cloud-observability-tools-2026 (accessed 2026-09-29) |
| Axiom | SaaS log and trace store | Serverless-friendly log ingestion for logs and traces at low cost | Usage-based; source describes it as a fraction of Datadog's cost | Low — HTTP ingest and platform integrations | Younger; scope narrower than full suites | https://kanopylabs.com/blog/axiom-vs-datadog-vs-grafana-cloud (accessed 2026-09-29) |

### File & Object Storage

Serves deliverables of tens of MB to over 1 GB with resumable upload, version history for the account's life, storage limits, purge on deletion, and generated export archives (FEAT-06.SPEC-003, FEAT-16.SPEC-002, FEAT-16.SPEC-004, FEAT-16.SPEC-006, FEAT-17, FEAT-24.SPEC-003); storage and bandwidth cost must fit the budget (BRIEF.md ## Scale & Non-Functional Expectations).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Cloudflare R2 | S3-compatible object storage | Multipart upload with presigned URLs; zero egress fees suit large downloads | $0.015/GB-month; no egress; Class A $4.50/M, Class B $0.36/M; 10 GB free | Low — S3 API | Mature; S3 API keeps exit path open | https://egresscost.com/cloudflare/ (accessed 2026-09-29) |
| Amazon S3 | Cloud object storage | Multipart and resumable patterns, lifecycle rules, deepest tooling | Standard $0.023/GB-month first 50 TB plus requests and data-transfer-out | Low–medium — IAM and CORS | Very mature; egress cost makes leaving costly | https://aws.amazon.com/s3/pricing/ (accessed 2026-09-29) |
| Backblaze B2 | S3-compatible object storage | Low-cost storage; free egress up to 3x stored data and free via Cloudflare Bandwidth Alliance | $0.00695/GB-month; egress beyond 3x at $0.01/GB | Low — S3-compatible API | Mature; S3-compatible | https://www.backblaze.com/cloud-storage/pricing (accessed 2026-09-29) |
| Supabase Storage | Managed storage on Postgres platform | Resumable (TUS) uploads, access rules tied to row-level security | Counted against plan egress quota (Pro 250 GB, $0.09/GB beyond) plus storage overage | Low if Supabase is in use | Couples to Supabase platform | https://supabase.com/pricing (accessed 2026-09-29) |

### Email & Messaging Delivery

Serves email-only notification delivery (26 Notification specs; FEAT-14.SPEC-001) with delivery and bounce status reporting (FEAT-14.SPEC-003) and branded presentation (FEAT-14.SPEC-005); push and SMS are not named in any spec.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Resend | Managed email API | Developer-first API, webhooks for delivery events, React email templates | Free 3,000 emails/month; Pro $20/month for 50,000 | Low — REST and SDKs | Newer entrant; SMTP interface limits lock-in | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| Postmark | Managed transactional email | Deliverability-focused, message streams, bounce webhooks | $15/month base plus $1.80 per extra 1K emails | Low — REST and SDKs | Long established; proprietary streams | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| Amazon SES | Cloud email service | Very low-cost sending with event publishing via SNS | $0.10 per 1,000 emails plus $0.12/GB attachments | Medium — IAM, domain and reputation setup | Very mature; ties event handling to AWS | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| SendGrid (Twilio) | Managed email platform | Transactional and marketing email, event webhooks | About $0.40 per 1,000 at paid tiers per comparison source | Low–medium | Very mature; Twilio account coupling | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |

### Payments & Billing

Serves card and bank-transfer payment into each freelancer's own processor account with the platform never touching card data (FEAT-10.SPEC-003, FEAT-32.SPEC-002; ASMP-24, ASMP-28), reversal notices (FEAT-25.SPEC-005), and separate recurring billing of freelancers for their own plan (FEAT-23.SPEC-003; ASMP-31); no cut of payments (BRIEF.md ## Business Context).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Stripe (Connect Standard with direct charges + Stripe Billing) | Payment platform | Freelancers connect their own account; funds land in their account; webhooks for status, disputes; Billing covers own-plan subscriptions | Direct charges on Standard accounts: Stripe fees charged to the connected account, no Connect fee to platform; 2.9% + 30c cards; Billing 0.7% of billing volume | Medium — OAuth/onboarding, webhooks, idempotency | Very mature; broadest SDK coverage; payment method and customer data are portable only via Stripe processes | https://docs.stripe.com/connect/charges (accessed 2026-09-29) |
| Adyen for Platforms | Payment platform | Global acquiring for marketplaces; sales-led onboarding | Interchange++ pricing with monthly minimums | High — sales process and heavier integration | Very mature; enterprise-oriented | https://www.chargeflow.io/blog/stripe-vs-adyen (accessed 2026-09-29) |
| PayPal Commerce Platform | Payment platform | Payment acceptance in 200+ markets; marketplace onboarding | Transaction fees per PayPal rate card | Medium | Very mature; consumer-brand recognition | https://www.inflowpay.com/blog/top-6-paypal-alternatives (accessed 2026-09-29) |
| Mollie Connect | Payment platform | Strong European local payment methods; blended per-transaction fees | Blended per-transaction fee | Medium | Mature; Europe-focused | https://www.mollie.com/growth/mollie-vs-adyen (accessed 2026-09-29) |
| Paddle (own-plan subscriptions only) | Merchant of Record billing | Handles subscription billing and worldwide tax for the platform's own plan, not per-freelancer payouts | About 5% + 50c per transaction | Medium | Mature; separates from client-payment processor | https://aliteq.com/stripe-vs-paddle-vs-lemon-squeezy-ai-saas-2026 (accessed 2026-09-29) |

### Search

Serves FEAT-28 cross-entity global search scoped by access rules (FEAT-28.SPEC-003) with rule-based ranking (FEAT-28.SPEC-004); no full-text or faceted keywords in specs; a few thousand freelancers with 3–15 clients each.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostgreSQL full-text search (tsvector/pg_trgm) | Database feature | Search inside existing data store, honoring same access rules | Included with database | Low — SQL queries and indexes | Mature; standard Postgres | knowledge-based — no Postgres documentation fetched this run; characteristics from model knowledge |
| Typesense | Open-source search engine / cloud | Typo-tolerant instant search; self-host or cloud | Cloud from ~$14/month or free self-hosted | Medium — sync index with database | Mature; open source | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |
| Meilisearch | Open-source search engine / cloud | Relevance-focused search with simple API | Cloud from ~$20–30/month or free self-hosted | Medium — sync index | Mature; open source | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |
| Algolia | Managed search service | Full-featured hosted search | ~$0.50 per 1K records and $0.40 per 1K searches on developer tier | Medium — indexing pipeline | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |

### Background Jobs & Scheduling

Serves reminder schedules (FEAT-11.SPEC-001), totals refresh (FEAT-12.SPEC-004), email retry (FEAT-14.SPEC-003), export archive generation (FEAT-24.SPEC-003), legal retention purge (FEAT-24.SPEC-005), support-session auto-close (FEAT-31.SPEC-004) and reliable webhook processing.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Inngest | Managed durable-function platform | Event-driven steps, retries, cron; calls own serverless endpoints | Free 25K runs/month; Basic $30/month; Pro $300/month | Low — SDK | Managed only, no self-host; proprietary function model | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| Trigger.dev v3 | Managed / self-hostable job platform | Long-running tasks without serverless timeouts; schedules | Free 50K runs/month; paid from $10/month | Low–medium | Open source (MIT); self-host via Docker | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| Upstash QStash | Managed HTTP message queue and scheduler | Scheduled and retried HTTP calls to endpoints | Usage-based per message | Low — HTTP | Managed; simple model, low lock-in | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| pg-boss / BullMQ workers | Open-source queue libraries | Postgres-backed (pg-boss) or Redis-backed (BullMQ) queues with cron; needs a long-running worker | Free; infrastructure cost of worker (and Redis for BullMQ) | Medium — operate workers | Mature; open source | knowledge-based — no pg-boss or BullMQ page fetched this run; characteristics from model knowledge |

### Caching & Performance

Serves ~2 s mobile interactivity (ASMP-21), dashboard totals within 1–2 s (ASMP-21) and degraded/offline read states (FEAT-29.SPEC-004; ASMP-27) at a few thousand freelancers.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Upstash Redis | Managed serverless Redis | Rate limiting and cached aggregates over HTTP | Free 256 MB / 500K commands; pay-as-you-go $0.20 per 100K commands; fixed from $10/month | Low — REST and SDK | Managed; Redis-protocol exit | https://upstash.com/pricing (accessed 2026-09-29) |
| Redis Cloud | Managed Redis | Conventional Redis with persistence options | Plan-based | Low–medium | Mature; Redis protocol portability | https://upstash.com/blog/redis-pricing-comparison-every-major-provider-in-2026-with-numbers (accessed 2026-09-29) |
| CDN and framework cache (Cloudflare CDN / Next.js cache) | Edge/HTTP caching | Static assets, cached public pages and revalidation without a separate service | Included with hosting/CDN plans | Low | Framework/CDN-specific semantics | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |
| Database-level caching (materialized views / read replicas) | Database feature | Precomputed financial totals without another service | Included with database, replica cost extra | Low–medium | Standard Postgres | knowledge-based — no Postgres documentation fetched this run; characteristics from model knowledge |

### Real-time & Collaboration

Real-time = No; the active signal is Collaboration/concurrency: exactly-once acceptance (FEAT-03.SPEC-003), approval concurrency guard (FEAT-08.SPEC-003) and reject-with-refresh on 12 contended entities, between one freelancer and client contacts rather than shared editing.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Database optimistic concurrency and transactions (row versions, unique constraints) | Database technique | Reject-with-refresh and exactly-once semantics inside the data store, no transport service | Included with database | Low–medium — implemented in data layer | Standard SQL | knowledge-based — technique not tied to a vendor page; from model knowledge |
| Supabase Realtime | Managed realtime over Postgres changes | Push updates of row changes if live refresh is wanted | Included in Supabase plans, usage quotas apply | Low if Supabase in use | Couples to Supabase | https://supabase.com/pricing (accessed 2026-09-29) |
| Ably | Managed pub/sub | Message fan-out with presence and history | Free 6M messages/month; consumption $2.50 per million | Low — SDK | Mature; proprietary protocol | https://ably.com/compare/ably-vs-pusher (accessed 2026-09-29) |
| Pusher Channels | Managed pub/sub | Simple channels for change notifications | From $49/month; free tier 200 connections | Low — SDK | Mature; tiered plans | https://ably.com/compare/ably-vs-pusher (accessed 2026-09-29) |

### Analytics & Product Telemetry

Serves product success metrics for a few thousand freelancers (ASMP-22) with GDPR-class data handling (ASMP-24); referral attribution capture exists as a product feature (FEAT-33).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostHog | SaaS / self-hostable product analytics | Events, funnels, feature flags, session replay | Free 1M events/month; then from $0.00005/event | Low — SDK | Open source; self-host option | https://posthog.com/pricing (accessed 2026-09-29) |
| Mixpanel | SaaS product analytics | Event funnels and retention | Free 20M events/month; Growth ~$0.00028/event | Low | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |
| Amplitude | SaaS product analytics | Behavioral analysis and experimentation | Free basic tier; growth from $49/month | Low | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |
| Plausible | Privacy-first web analytics | Cookie-less page-level analytics; light product event support | Subscription by pageviews | Low — one script | Mature; open source, self-hostable | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |

### Internationalization

Serves worldwide currencies, tax and time zones (FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-006, FEAT-15.SPEC-008); English only at launch, no translation specs; multi-currency dashboard does not aggregate (FEAT-15.SPEC-007).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Native Intl APIs (Intl.NumberFormat, Intl.DateTimeFormat) | Platform standard | Currency, date and time-zone formatting with no translation layer | Free (built into runtimes) | Low | Web standard; no lock-in | https://tolgee.io/blog/react-i18n-libraries-comparison (accessed 2026-09-29) |
| next-intl | Library (Next.js) | Server Component friendly message and format handling for future locales | Open source (MIT) | Low — Next.js-specific | Mature; Next.js-bound | https://www.pkgpulse.com/guides/next-intl-vs-react-i18next-vs-lingui-react-i18n-2026 (accessed 2026-09-29) |
| i18next / react-i18next | Library | Framework-agnostic translation and formatting | Open source (MIT) | Low | Very mature; ~2.8M weekly downloads per source | https://tolgee.io/blog/react-i18n-libraries-comparison (accessed 2026-09-29) |
| Lingui | Library | ICU-standard, compile-time extraction, ~3KB runtime | Open source (MIT) | Medium — macros and extraction | Mature; ICU/PO workflows | https://www.pkgpulse.com/guides/next-intl-vs-react-i18next-vs-lingui-react-i18n-2026 (accessed 2026-09-29) |

### Electronic Signature Attestation

Serves FEAT-26.SPEC-005: attesting signature data captured in-portal so signed acceptance has legal weight beyond a self-recorded timestamp; jurisdictional standard left to this stage; v1 phase; sent to both parties as signed copy (FEAT-26.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Documenso | Open-source e-signature platform | Embedded signing and API; white-labeling on Platform tier; self-hostable | Free 5 docs/month; Individual $30/month; Platform $250/month | Medium — embed and API | Newer; open source with self-host exit | https://dev.to/beton/documenso-pricing-teardown-2026-3ic6 (accessed 2026-09-29) |
| DocuSign eSignature API | Managed e-signature API | Embedded signing with widely recognized audit certificates | Essentials $75/month for 50 requests; Standard $250/month for 100; embedded signing on a ~$480/month tier | Medium–high | Very mature; per-envelope pricing | https://signb.ee/blog/embedded-esignature-api-cost-calculator (accessed 2026-09-29) |
| Dropbox Sign API | Managed e-signature API | Embedded signing, templates, webhooks | Subscription from ~$100/month billed monthly | Medium | Mature; per-request tiers | https://www.signwell.com/resources/dropbox-sign-api/ (accessed 2026-09-29) |
| SignWell API | Managed e-signature API | Embedded signing and API for lower-volume use | Plan-based | Medium | Mature; SaaS | https://www.signwell.com/resources/dropbox-sign-api/ (accessed 2026-09-29) |

### Custom Domain Verification & TLS Serving

Serves FEAT-27.SPEC-002: verify a freelancer controls a domain (DNS check) and serve the portal over HTTPS at it, with shared default address as fallback (FEAT-27.SPEC-003); Later phase, at most one per freelancer, a few thousand accounts.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Cloudflare for SaaS (Custom Hostnames) | Managed custom-hostname service | Automated certificate issuance and validation via API; fallback origin | 100 hostnames free on Free/Pro/Business; then $0.10/hostname/month to 50,000 | Medium — API and DNS setup | Mature; requires traffic through Cloudflare | https://domainee.dev/blog/cloudflare-for-saas-pricing (accessed 2026-09-29) |
| Vercel Domains API | Hosting-platform domain management | Add and verify domains per project via API with automatic TLS | Included in Vercel plans; limits per plan | Low if hosted on Vercel | Couples to Vercel | knowledge-based — Vercel domains documentation not fetched this run; from model knowledge |
| Caddy on-demand TLS | Open-source web server | Issues certificates on demand for verified hosts; self-hosted | Free; server infrastructure cost | Medium–high — operate proxy | Mature; open source | knowledge-based — Caddy documentation not fetched this run; from model knowledge |
| Fly.io custom domain certificates | Hosting-platform certificates | Certificates for custom hostnames via API on Fly apps | Included with Fly usage; certificate limits per app | Medium | Couples to Fly.io | knowledge-based — Fly.io certificate documentation not fetched this run; from model knowledge |

## 3. Cross-Area Compatibility Notes

- Drizzle ORM, Prisma and Kysely (ORM / Data Access) are TypeScript/Node data-access layers; they pair with the Node-runtime options in Backend / API Layer (framework server layer, Hono, NestJS) and not with Django. SQLAlchemy is Python-only and pairs with Django, not a Node backend.
- Every Postgres option in Database (Neon, Supabase Postgres, Amazon RDS, Render Postgres) is standard Postgres, supported by Drizzle, Prisma, Kysely and SQLAlchemy.
- Prisma cannot run in edge runtimes without Prisma Accelerate; Drizzle and Kysely run on edge runtimes (Vercel Edge, Workers).
- TanStack Query and Zustand have React bindings and TanStack Query has adapters for Vue and Svelte; SWR and Redux Toolkit are React-only, so they pair only with Next.js or React Router in Frontend Framework. Svelte projects use Svelte-native stores.
- shadcn/ui assumes React plus Tailwind CSS; Radix primitives are React; Headless UI supports React and Vue; Melt UI is Svelte.
- Tailwind CSS, CSS Modules and vanilla CSS approaches pair with all Frontend Framework options; next-intl is Next.js-specific, whereas i18next and Lingui support several frameworks.
- Next.js ships Turbopack; choosing Vite for a Next.js project overrides the bundled pipeline. React Router, SvelteKit and Nuxt use Vite by default.
- Vercel is native for Next.js; Fly.io, Railway and Render run any container. Vercel Hobby is limited to non-commercial use.
- Supabase Auth, Storage and Realtime bind to Supabase Postgres; Clerk, Better Auth and WorkOS AuthKit are independent of the database choice, and Better Auth stores users in the application's own database via the ORM.
- Cloudflare R2, Backblaze B2 and Amazon S3 share the S3 API, so one S3 client library works against all three; B2 egress via Cloudflare is free through the Bandwidth Alliance.
- Cloudflare for SaaS requires custom-hostname traffic to pass through Cloudflare; Vercel Domains API and Fly.io certificates require the portal to run on that platform; Caddy on-demand TLS requires a self-operated proxy.
- Stripe, Resend, Postmark, SendGrid, Amazon SES, Clerk, Sentry, PostHog and Inngest all publish official Node/TypeScript SDKs; Documenso and DocuSign offer HTTP APIs usable from any backend language. SDK availability for Python backends is present for Stripe, Sentry, PostHog, Amazon SES and Postmark; Inngest offers a Python SDK.
- Stripe Connect Standard with direct charges places Stripe fees on the connected account and does not apply platform pricing tools; Stripe Billing for the platform's own subscription is a separate charge stream from client payments; Paddle covers only the platform's own subscription and does not replace a client-payment processor.
- Trigger.dev and Inngest run job code against the application's own endpoints or compute; pg-boss requires Postgres and BullMQ requires Redis, which links Background Jobs & Scheduling to the Database and Caching & Performance options.
- Supabase Realtime and Supabase Storage are only available on Supabase; Ably and Pusher work with any backend.

## 4. Research Log

| Decision Area | Method | Queries & Key Sources | Access Date |
|---|---|---|---|
| Frontend Framework | web | "Next.js vs SvelteKit vs React Router Remix vs Nuxt 2026 comparison full-stack framework"; dev.to, prismic.io, medium.com comparisons | 2026-09-29 |
| Backend / API Layer | web (partial) with knowledge-based fallback | Same framework comparison query for framework server layer. Hono, NestJS and Django rows are knowledge-based — no dedicated page fetched this run; characteristics from model knowledge | 2026-09-29 |
| Database | web (partial) with knowledge-based fallback | "Neon Postgres pricing Launch Scale plan"; neon.com/pricing; supabase.com/pricing; Render via hosting comparison. Amazon RDS row knowledge-based — AWS RDS pricing page not fetched this run | 2026-09-29 |
| ORM / Data Access | web (partial) with knowledge-based fallback | "Drizzle ORM vs Prisma vs Kysely 2026"; makerkit.dev, pkgpulse.com. SQLAlchemy + Alembic row knowledge-based — no SQLAlchemy page fetched this run | 2026-09-29 |
| CSS / Styling | web | "Tailwind CSS v4 vs CSS Modules vs Panda CSS vs vanilla-extract 2026"; pkgpulse.com guides. Component-layer note list from decision guide Section 6 and model knowledge (no prices claimed) | 2026-09-29 |
| State Management | web | "TanStack Query vs Zustand vs Redux Toolkit vs SWR 2026"; saschb2b.com, dev.to | 2026-09-29 |
| Build Tooling | web (partial) with knowledge-based fallback | Turbopack status from the Next.js framework comparison. Vite, pnpm and Bun rows knowledge-based — no dedicated pages fetched this run | 2026-09-29 |
| Authentication & Identity | web | "Clerk pricing monthly active users magic link Auth.js Better Auth"; "Auth0 vs Supabase Auth vs WorkOS AuthKit pricing 2026"; clerk.com/pricing; buildmvpfast.com/api-costs/authentication; dev.to Clerk vs Better Auth | 2026-09-29 |
| Hosting & Environments | web | "Vercel pricing Pro plan"; "Fly.io Railway Render pricing 2026"; vercel.com/docs/plans/pro-plan; dev.to pricing comparison | 2026-09-29 |
| CI/CD & Delivery | web | "GitHub Actions pricing 2026; CircleCI pricing; GitLab CI"; docs.github.com billing; circleci.com/pricing; eesel.ai GitLab pricing | 2026-09-29 |
| Observability & Operations | web | "Sentry pricing Team plan 2026"; "Better Stack vs Grafana Cloud vs Datadog vs Axiom pricing"; blog.struct.ai; kanopylabs.com; blog.railway.com | 2026-09-29 |
| File & Object Storage | web | "Cloudflare R2 pricing"; "Backblaze B2 pricing; Amazon S3 pricing"; egresscost.com; backblaze.com/cloud-storage/pricing; aws.amazon.com/s3/pricing; supabase.com/pricing | 2026-09-29 |
| Email & Messaging Delivery | web | "Resend Postmark Amazon SES pricing transactional email 2026"; buildmvpfast.com/api-costs/email | 2026-09-29 |
| Payments & Billing | web | "Stripe Connect pricing standard accounts direct charges fees"; "Stripe Billing pricing ... Paddle Lemon Squeezy"; "Adyen for Platforms vs PayPal Commerce vs Mollie"; docs.stripe.com/connect/charges; chargeflow.io; mollie.com; aliteq.com. PayPal and Adyen pricing figures not retrieved as numbers; rows describe the pricing shape only | 2026-09-29 |
| Search | web (partial) with knowledge-based fallback | "Meilisearch Typesense Algolia pricing 2026"; buildmvpfast.com/api-costs/search. PostgreSQL full-text search row knowledge-based — no Postgres documentation fetched this run | 2026-09-29 |
| Background Jobs & Scheduling | web (partial) with knowledge-based fallback | "Inngest Trigger.dev pricing 2026"; buildmvpfast.com/api-costs/background-jobs. pg-boss / BullMQ row knowledge-based — no dedicated page fetched this run | 2026-09-29 |
| Caching & Performance | web (partial) with knowledge-based fallback | "Upstash Redis pricing 2026"; upstash.com/pricing; upstash.com blog. Database-level caching row knowledge-based — no Postgres documentation fetched this run | 2026-09-29 |
| Real-time & Collaboration | web (partial) with knowledge-based fallback | "Ably Pusher Channels Liveblocks pricing 2026"; ably.com/compare/ably-vs-pusher; supabase.com/pricing. Optimistic-concurrency row knowledge-based — technique not tied to a vendor page | 2026-09-29 |
| Analytics & Product Telemetry | web | "PostHog pricing"; "Plausible vs Mixpanel vs Amplitude pricing 2026"; posthog.com/pricing; buildmvpfast.com/api-costs/analytics. Plausible pricing figure not retrieved | 2026-09-29 |
| Internationalization | web | "next-intl vs i18next vs Lingui vs FormatJS 2026"; tolgee.io; pkgpulse.com | 2026-09-29 |
| Electronic Signature Attestation | web | "Documenso vs DocuSign API vs Dropbox Sign API pricing embedded signing 2026"; dev.to Documenso teardown; signb.ee; signwell.com. SignWell pricing figure not retrieved | 2026-09-29 |
| Custom Domain Verification & TLS Serving | web (partial) with knowledge-based fallback | "Cloudflare for SaaS custom hostnames pricing per hostname"; domainee.dev. Vercel Domains API, Caddy on-demand TLS and Fly.io certificate rows knowledge-based — documentation pages not fetched this run | 2026-09-29 |

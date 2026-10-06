# Research: Portal Referral Attribution (FEAT-33)

All technical decisions for this feature were made upstream in the blueprint and are
RESOLVED — there are no unknowns to research. Plan directly against the decisions below;
alternatives are documented in the blueprint for the humans who own this project, and are
not the planning agent's to choose.

## Decided stack (project-wide)

- Frontend: Next.js 16 (App Router, React 19, TypeScript) (ADR-001)
- Backend / API layer: Next.js framework server layer (server actions + Route Handlers on Node runtime); long-running work to job runner (ADR-002)
- Database: Neon serverless Postgres (Launch plan, Scale plan at growth) with branches per environment (ADR-003)
- ORM / data access: Drizzle ORM v1 + drizzle-kit, Neon Pool driver for interactive transactions (ADR-004)
- Styling and components: Tailwind CSS v4 with CSS-variable theme tokens and injected per-freelancer --brand variables (ADR-005)
- UI primitives: Component library: shadcn/ui (copy-in, Radix primitives + Tailwind) in src/shared/components/ui (ADR-024)
- Background jobs: Trigger.dev v3 (Cloud) fed by a Postgres transactional outbox with one-minute sweep (ADR-012)
- Authentication: Better Auth (freelancer/operator realm, sessions in Neon) + application-owned client-portal magic-link realm keyed by Client Contact with confirm-click consumption (ADR-026)

## Decisions specific to this feature

- **ADR-015 — Analytics & Product Telemetry:** PostHog Cloud (free tier) with server-side milestone events keyed by opaque ids; no replay on portal. Product signals are emitted as server-side events keyed by opaque IDs with no client-contact personal data.
- **ADR-021 — API & Routing:** Resource-oriented URLs in three route groups: /app (freelancer), /portal/[handle] (client, custom-domain rewrite), /ops (operator), plus public pages. Screens live in the decided route groups (/app, /portal/[handle], /ops) so each realm keeps its own shell and guards.

## Data model

This feature touches these canonical entities: referral_attribution, freelancer_account.
Full table definitions, relationships, and lifecycle rules:
`docs/blueprint/architecture/database-schema.md`.

## Full-depth sources

- `docs/blueprint/architecture/technical-architecture.md` — the recommended architecture,
  every decision area, and the consolidated ADR register (Section 14)
- `docs/blueprint/architecture/technical-feasibility.md` — feasibility analysis and
  approach classifications
- `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/` — this feature's full specifications
  (the verbatim source of every acceptance criterion in spec.md)

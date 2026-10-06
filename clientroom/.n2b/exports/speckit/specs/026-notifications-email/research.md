# Research: Notifications (Email) (FEAT-14)

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

- **ADR-009 — Email & Messaging Delivery:** Resend Pro with delivery/bounce webhooks and React Email branded templates. Every email this feature sends or tracks goes through the decided email delivery service, with its delivery and bounce webhooks feeding delivery status.
- **ADR-012 — Background Jobs & Scheduling:** Trigger.dev v3 (Cloud) fed by a Postgres transactional outbox with one-minute sweep. Scheduled, long-running and retried work runs on the decided job runner fed by the transactional outbox, so a write and its follow-up work commit atomically.
- **ADR-023 — Webhook Ingestion:** Signature-verified webhooks persisted to an inbound_event ledger (unique provider event id), acknowledged 200, processed by Trigger.dev in event-time order per subject with processor state authoritative. Provider events are persisted to the inbound_event ledger, acknowledged, and processed in event-time order so a duplicate event changes nothing.
- **ADR-005 — CSS / Styling:** Tailwind CSS v4 with CSS-variable theme tokens and injected per-freelancer --brand variables. Per-freelancer brand colours reach pages and emails through CSS variables, so branding needs no per-tenant stylesheet.

## Data model

This feature touches these canonical entities: notification, notification_type, notification_preference, freelancer_account, branding_profile.
Full table definitions, relationships, and lifecycle rules:
`docs/blueprint/architecture/database-schema.md`.

## Full-depth sources

- `docs/blueprint/architecture/technical-architecture.md` — the recommended architecture,
  every decision area, and the consolidated ADR register (Section 14)
- `docs/blueprint/architecture/technical-feasibility.md` — feasibility analysis and
  approach classifications
- `docs/blueprint/specifications/FEAT-14-notifications-email/` — this feature's full specifications
  (the verbatim source of every acceptance criterion in spec.md)

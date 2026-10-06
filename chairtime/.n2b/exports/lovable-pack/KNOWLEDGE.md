# Chairtime — Project Knowledge

Distilled build constitution for Chairtime. Full depth: docs/blueprint/.

## What this is

- Mobile-first booking page for ONE solo beauty/wellness pro (barber, nail tech, lash/brow artist, massage therapist, tattoo artist) that takes a card deposit and sends reminders [S: BRIEF.md]
- Roles: the Pro (owner, sees all bookings, clients, money) and the Client (books via the Pro's Instagram link, no password, sees only own bookings); read-only founder support access is not a product role [S: BRIEF.md]
- Value: no more DM haggling or chasing deposits; a no-show keeps the deposit automatically under the Pro's own policy; booking and payment must be correct always (never double-book, never lose a deposit) [S: BRIEF.md]

## Domain glossary

- **Deposit rule** — per-service fixed amount or 1–100% of price, computed once and locked at booking; balance is paid in person [S: features/product-features.md]
- **Cancellation window** — Pro-defined cutoff; outside it a cancel refunds the deposit, inside it or on no-show the deposit is forfeited to the Pro (binary, no partials) [S: features/product-features.md]
- **Slot hold** — short, time-limited reservation during checkout; the first client to complete payment wins [S: specifications/feature-dependency-map.md]
- **Access / manage link** — passwordless phone-plus-one-tap link that is the Client's only identity [S: features/product-features.md]
- **Messaging consent** — explicit SMS opt-in at booking; STOP revokes it; every message must respect it [S: features/product-features.md]
- **Zero platform fee** — Chairtime takes no cut of deposits, balances or tips; only the processor's card fee shows [S: specifications/feature-dependency-map.md]
- Slot rule: a time is offered, held or booked only if it passes the live check (duration plus buffer, no conflicting booking, block or calendar busy time) (XBR-01)
- Holds auto-release; minimum notice and booking horizon bind every client path, only the Pro may override (XBR-02, XBR-03)
- Service edits affect future bookings only; the deposit is computed once, client-unalterable, charged once (XBR-04, XBR-05)
- No deposit and no live link unless the Pro's payout account is active; platform fee is always zero (XBR-06, XBR-07)
- Each booking follows the policy version acknowledged at booking; outcomes are binary and symmetric (XBR-08, XBR-09)
- Correctness bar: never silently double-book or lose a deposit; no numeric uptime target (ASMP-26)
- **Handle** — the Pro's short public booking link (/{handle}), built for Instagram bios; client routes /c, Pro routes /app (ADR-020)
- **Pause** — the Pro can stop new bookings (holiday); existing bookings are honored [S: features/product-features.md]

## Data model — core entities

- **Booking** — appointment with DB-enforced non-overlap range (appointment plus buffer) [S: architecture/database-schema.md]
- **Service / Availability Rule / Time Block** — what is sold and when it is bookable [S: architecture/database-schema.md]
- **Client / Messaging Consent / Message** — per-Pro client record, opt-in state, sent messages [S: architecture/database-schema.md]
- **Deposit Transaction / Balance Payment / Payout Account** — money in, refunds, payout routing [S: architecture/database-schema.md]
- **Cancellation Policy** — versioned; each booking keeps the version acknowledged [S: architecture/database-schema.md]
- **Activity Event** — append-only dispute-evidence timeline [S: architecture/database-schema.md]
- **Calendar Connection / Access Link / Subscription** — calendar sync, client identity, Pro billing [S: architecture/database-schema.md]
- Every Pro-owned table carries pro_account_id for per-Pro isolation (RLS) [S: architecture/database-schema.md]

## Architecture (binding)

- The recommended architecture is BINDING; documented alternatives are informational only [S: architecture/technical-architecture.md]
- Frontend and backend: Next.js 15 App Router, React 19, TypeScript; server actions plus route handlers (ADR-001, ADR-002, ADR-021)
- Database: Supabase Postgres with btree_gist exclusion constraints and RLS; Drizzle ORM, migrations only (ADR-003, ADR-004, ADR-019)
- Styling: Tailwind CSS v4 tokens plus shadcn/ui (ADR-005, ADR-023); state: TanStack Query v5 (ADR-006)
- Payments: Stripe Connect direct charges, zero application fee, Stripe Billing for subscriptions (ADR-010)
- Messaging: Twilio SMS plus Postmark email; calendars: Nylas for Google and iCloud (ADR-009, ADR-017)
- Jobs: Inngest with a Postgres outbox; webhooks stored before 2xx, idempotent (ADR-012, ADR-022)
- Auth: Better Auth OTP for Pros; hashed-token access links for Clients (ADR-025, ADR-026)
- Hosting and ops: Vercel iad1, GitHub Actions, Sentry, PostHog (ADR-028, ADR-029, ADR-030, ADR-015)
- Conventions: strict TypeScript, Zod shared validation, date-fns-tz, integer money (ADR-027, ADR-024, ADR-016)

## Features — build order

- FEAT-01 Service & Pricing Management — Core: services, prices, deposit rules
- FEAT-02 Availability & Working Hours Setup — Core: hours, buffers
- FEAT-04 Two-Way Calendar Sync — Core: Google/Apple busy times in, bookings out
- FEAT-09 Cancellation & No-Show Policy Engine — Core: versioned policy, deposit outcomes
- FEAT-27 Pro Profile & Booking Page Settings — Important: profile, link, pause
- FEAT-28 Payout Account Connection & Payout Visibility — Core: payouts, money view
- FEAT-29 Pro Sign-In & Account Lifecycle — Important: OTP sign-in, export, closure
- FEAT-15 Pro Onboarding & Setup Wizard — Important: setup to live link
- FEAT-18 Pro Subscription Billing & Account Management — Important: flat monthly plan
- FEAT-03 Real-Time Slot Availability Engine — Core: live free-slot check, holds
- FEAT-05 Public Booking Page & Booking Flow — Core: one-minute client flow
- FEAT-06 Client Booking Identity — Core: phone plus link, my bookings
- FEAT-07 Deposit Payment at Booking — Core: card deposit
- FEAT-10 Client-Initiated Cancel/Reschedule — Core: self-serve within policy
- FEAT-11 No-Show Marking & Deposit Forfeiture — Core: mark, forfeit, undo
- FEAT-13 Client Record Management — Important: notes, deletion
- FEAT-14 Messaging Consent Management — Important: opt-in, STOP
- FEAT-08 Automated Booking Messaging — Core: confirmations, reminders
- FEAT-12 Pro Daily Schedule Dashboard — Core: today list, attention
- FEAT-16 Booking & Payment Activity Record — Important: append-only timeline
- FEAT-19 Platform Support Read-Only Access — Important: support view
- FEAT-20 Waitlist for Cancelled Slots — Nice-to-Have
- FEAT-21 Recurring/Standing Appointments — Nice-to-Have
- FEAT-22 In-App Balance Payment — Nice-to-Have
- FEAT-23 Tipping at Checkout — Nice-to-Have
- FEAT-24 Client List Search & Filter — Nice-to-Have
- FEAT-25 Booking & Revenue Insights — Nice-to-Have
- FEAT-26 WhatsApp Reminders — Nice-to-Have
- FEAT-30 Pro Booking Management — Core: cancel, reschedule, refund, book in
- FEAT-17 Manual Time Blocking — Important: blocks, recurrence

## DO-NOT-BUILD (scope exclusions)

- SC-01 — multi-staff or multi-chair accounts
- SC-02 — any admin/manager/staff role beyond read-only support
- SC-03 — cross-pro or cross-client visibility
- SC-04 — client passwords or shared client profiles
- SC-05 — support acting on a Pro's behalf
- SC-06 — Instagram integration beyond the bio link
- SC-07 — native apps, app stores
- SC-08 — health/intake forms
- SC-09 — importing data from prior tools
- SC-10 — multi-language content
- SC-11 — handling or storing card data
- SC-12 — social, reviews, marketplace
- SC-13 — card-on-file cancellation fees
- SC-14 — dynamic or demand pricing
- SC-15 — marketing text campaigns
- SC-16 — card-reader hardware
- SC-17 — Chairtime ruling on disputes
- SC-18 — partial refunds, tiered schedules
- SC-19 — scale target (hundreds of pros)
- SC-20 — US-first, timezone/currency per account
- SC-21 — correctness over uptime numbers
- SC-22 — keep history; de-identify after deletion
- Full verbatim exclusions with rationale: AGENTS.md and docs/blueprint/features/scope-boundaries.md [S: features/scope-boundaries.md]

## Design posture

- Design-agnostic: no design system is in the blueprint; the builder owns visual design and honors preferences in the brief's Constraints [S: BRIEF.md]

## Depth

- Full specs with all 2996 acceptance criteria, architecture alternatives, and the database schema: docs/blueprint/ [S: specifications/, architecture/]

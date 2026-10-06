---
document_type: technical-feasibility
produced_by: feasibility-planner
status: final
stage: 4
feature_count: 30
created: 2026-09-29
---

# Technical Feasibility Assessment

## 1. Feasibility Summary

| Feature | Verdict | Driving Factors |
|---------|---------|-----------------|
| FEAT-01 (Service & Pricing Management) | Straightforward | Small per-Pro CRUD with validation and a price/deposit snapshot at booking time (FEAT-01.SPEC-004, FEAT-01.SPEC-005 ## Field Validation Rules); no external service |
| FEAT-02 (Availability & Working Hours Setup) | Straightforward | Versioned rule records plus a conflict-flagging check on save (FEAT-02.SPEC-003, FEAT-02.SPEC-004 ## Processing Logic); timezone arithmetic is library-grade work |
| FEAT-03 (Real-Time Slot Availability Engine) | Hard | Roughly one-second slot computation over five data sources plus first-committed-wins hold contention with a never-double-book bar (FEAT-03.SPEC-001, FEAT-03.SPEC-002, FEAT-03.SPEC-005 ## Edge Cases; ASMP-21, ASMP-26) |
| FEAT-04 (Two-Way Calendar Sync) | Research-spike recommended | Two-way sync with Google and Apple within "a couple of minutes" (FEAT-04.SPEC-003 ## Inbound Events); Apple/iCloud change-detection latency and authorization model are not knowable from the specs or landscape |
| FEAT-05 (Public Booking Page & Booking Flow) | Standard-with-integration | Multi-step flow inside the Instagram in-app browser that embeds payment and reads a stored photo (FEAT-05.SPEC-004, FEAT-05.SPEC-006 ## Processing Logic; FEAT-27.SPEC-012 ## Degradation Behavior) |
| FEAT-06 (Client Booking Identity) | Standard-with-integration | Bearer-credential access links delivered by text/email with rate limiting (FEAT-06.SPEC-006 ## Channels, FEAT-06.SPEC-007 ## Authorization Rules) |
| FEAT-07 (Deposit Payment at Booking) | Standard-with-integration | Card authorization/capture with payout routing and exactly-once outcome (FEAT-07.SPEC-005 ## Degradation Behavior, FEAT-07.SPEC-004 ## Edge Cases) |
| FEAT-08 (Automated Booking Messaging) | Standard-with-integration | SMS and email providers with delivery-status webhooks, daytime-window scheduling and retry-then-fallback (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Degradation Behavior; FEAT-08.SPEC-007, FEAT-08.SPEC-009) |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Standard-with-integration | Processor refunds with indefinite idempotent retry (FEAT-09.SPEC-005 ## Degradation Behavior, FEAT-09.SPEC-006 ## Edge Cases) |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Standard-with-integration | Booking commit racing Pro actions under reject-with-refresh, triggering refunds or a new deposit charge (FEAT-10.SPEC-004 ## Processing Logic; dependency map Booking **Contention:** High) |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Straightforward | State transitions on already-captured money with a 24-hour undo window (FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-004); no processor call |
| FEAT-12 (Pro Daily Schedule Dashboard) | Straightforward | Read-heavy dashboard, an auto-completion sweep and flag aggregation (FEAT-12.SPEC-004, FEAT-12.SPEC-005 ## Trigger Definition); read-only offline posture (feature-overview.md ## Non-Functional Notes) |
| FEAT-13 (Client Record Management) | Straightforward | CRUD plus hard-delete with de-identified retention cascade (FEAT-13.SPEC-004 ## Processing Logic, FEAT-13.SPEC-006) |
| FEAT-14 (Messaging Consent Management) | Standard-with-integration | Inbound STOP replies from the text provider honored on the very next message (FEAT-14.SPEC-004 ## Trigger Definition; FEAT-14.SPEC-006 ## Edge Cases) |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Straightforward | Resumable progress record and a go-live rule evaluator (FEAT-15.SPEC-004, FEAT-15.SPEC-005, FEAT-15.SPEC-007); integrations are reached through FEAT-28/FEAT-18 |
| FEAT-16 (Booking & Payment Activity Record) | Standard-with-integration | Inbound processor dispute webhooks plus an append-only log and downloadable summary (FEAT-16.SPEC-003 ## Inbound Events, FEAT-16.SPEC-005) |
| FEAT-17 (Manual Time Blocking) | Straightforward | Block CRUD, recurring-occurrence generation and conflict detection against bookings (FEAT-17.SPEC-004, FEAT-17.SPEC-005 ## Processing Logic) |
| FEAT-18 (Pro Subscription Billing & Account Management) | Standard-with-integration | Processor-run subscription with inbound renewal outcomes and a 7-day grace pause (FEAT-18.SPEC-006 ## Degradation Behavior, FEAT-18.SPEC-003) |
| FEAT-19 (Platform Support Read-Only Access) | Straightforward | Role-scoped read-only session with field masking and access logging (FEAT-19.SPEC-004 ## Authorization Rules, FEAT-19.SPEC-002) |
| FEAT-20 (Waitlist for Cancelled Slots) | Standard-with-integration | Immediate opening notices via messaging with a 30-minute priority window layered onto slot availability (FEAT-20.SPEC-004, FEAT-20.SPEC-005, FEAT-20.SPEC-008) |
| FEAT-21 (Recurring/Standing Appointments) | Standard-with-integration | Scheduled occurrence generation, per-occurrence deposit links and release at cut-off (FEAT-21.SPEC-004, FEAT-21.SPEC-005 ## Trigger Definition) |
| FEAT-22 (In-App Balance Payment) | Standard-with-integration | Second card charge with payout routing and cancellation-race refunds (FEAT-22.SPEC-005 ## Degradation Behavior, FEAT-22.SPEC-004) |
| FEAT-23 (Tipping at Checkout) | Standard-with-integration | Tip rides on the FEAT-22 charge and refund (FEAT-23.SPEC-003; dependency map ## External Touchpoints) |
| FEAT-24 (Client List Search & Filter) | Straightforward | Partial match and filter over 100–500 records per Pro (FEAT-24.SPEC-002; ASMP-22) |
| FEAT-25 (Booking & Revenue Insights) | Straightforward | Rolling per-period aggregates maintained on events (FEAT-25.SPEC-004 ## Trigger Definition) |
| FEAT-26 (WhatsApp Reminders) | Standard-with-integration | WhatsApp send/status channel with fallback to text/email (FEAT-26.SPEC-002 ## Degradation Behavior, FEAT-26.SPEC-003) |
| FEAT-27 (Pro Profile & Booking Page Settings) | Standard-with-integration | Photo storage capability plus globally-unique link names with 12-month forwarding (FEAT-27.SPEC-012 ## Degradation Behavior, FEAT-27.SPEC-007, FEAT-27.SPEC-010) |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Standard-with-integration | Processor-hosted identity/bank verification and inbound status/money events (FEAT-28.SPEC-006 ## Degradation Behavior, ## Edge Cases) |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Standard-with-integration | One-time-code delivery over SMS/email, device/session management, data export and closure orchestration (FEAT-29.SPEC-006, FEAT-29.SPEC-007, FEAT-29.SPEC-008, FEAT-29.SPEC-014) |
| FEAT-30 (Pro Booking Management) | Standard-with-integration | Pro cancel/reschedule/goodwill/bulk commits calling processor refunds with per-booking retry (FEAT-30.SPEC-008, FEAT-30.SPEC-011 ## Edge Cases) |

## 2. Per-Feature Assessments

### FEAT-01 — Service & Pricing Management

**Verdict:** Straightforward — validated CRUD over a small per-Pro list (FEAT-01.SPEC-001..003 ## States, FEAT-01.SPEC-004 ## Field Validation Rules) with a booking-time snapshot of price/duration/deposit (FEAT-01.SPEC-005 ## Cross-Field Rules) and a synchronous impact check on archive (FEAT-01.SPEC-006 ## Processing Logic); no external service.

**Required Capabilities:**
- Deposit-rule validation (fixed or percentage, at or above the `minimum-chargeable-deposit` platform parameter) in the Pro's account currency (FEAT-01.SPEC-004; XBR-05, XBR-25)
- Immutable price/deposit snapshot copied onto each Booking at booking time so later edits never alter confirmed bookings (FEAT-01.SPEC-005; XBR-04)
- Archive impact query for upcoming bookings referencing a service (FEAT-01.SPEC-006 ## Processing Logic)
- Service data read live by the booking page and slot engine immediately after save (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Concurrency: last-write-wins between the Pro's own sessions; FEAT-01 and FEAT-02 write disjoint fields (buffer_override) so concurrent saves never overlap; a client mid-checkout keeps the price shown (feature-dependency-map.md, Service **Contention:**)
- Offline/degraded: N/A — Pro-only settings writes need a live connection per ASMP-27; no degraded mode is specified beyond the product-wide plain "needs connection" message
- Scale: a handful to a few dozen services per Pro; "no growth pattern here threatens responsiveness" (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Plain relational tables in any Database-area Postgres option (Supabase Postgres, Neon, Crunchy Bridge, Amazon RDS / Aurora PostgreSQL) with the snapshot enforced by copying fields into the Booking row inside the booking transaction; data access via Prisma, Drizzle ORM or Kysely (ORM / Data Access) for typed models, or supabase-js / PostgREST client with row-level security for per-Pro isolation. Currency formatting via Native Intl APIs (Internationalization). Form state via Framework built-ins or Zustand (State Management).

**Risks & Unknowns:** The `minimum-chargeable-deposit` value must sit at or above the chosen processor's own minimum charge in each currency (platform-parameters.md); this is a decide-before-build value, not a technical blocker.

**Spike Recommendation:** None

### FEAT-02 — Availability & Working Hours Setup

**Verdict:** Straightforward — settings screens (FEAT-02.SPEC-001, FEAT-02.SPEC-002), append-only rule versioning (FEAT-02.SPEC-003 ## Processing Logic) and a post-save conflict-flagging pass (FEAT-02.SPEC-004 ## Trigger Definition) are well-trodden; the only subtlety is timezone interpretation (FEAT-02.SPEC-005 ## Field Validation Rules).

**Required Capabilities:**
- Dated versions of the Availability Rule retained while any booking references them (FEAT-02.SPEC-003; feature-overview.md ## Entity-Lifecycle Coverage Matrix)
- Post-save check of confirmed bookings against new hours that flags, never cancels (FEAT-02.SPEC-004; XBR-11)
- Timezone-aware window interpretation in the Pro's IANA zone, including daylight-saving transitions (FEAT-02.SPEC-005; ASMP-25, XBR-25)
- Concurrency: last-write-wins between the Pro's sessions; a slot held under a previous version is re-validated at confirmation (feature-dependency-map.md, Availability Rule **Contention:**)
- Offline/degraded: N/A — setup writes require a live connection (ASMP-27); no degraded mode specified
- Scale: one active rule plus a slowly growing set of superseded versions per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Versioned rows (effective-from column) in any Database-area Postgres option; the conflict-flagging check can run in-request inside the Backend / API Layer (Framework server layer, Hono, NestJS) at this data size, or be enqueued via a Background Jobs & Scheduling option (Inngest, Trigger.dev, Postgres-native queue and cron). Timezone math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Daylight-saving boundaries (non-existent and repeated local hours) can produce off-by-one-hour windows if rules are stored as UTC offsets rather than IANA zone plus local time (FEAT-02.SPEC-005; ASMP-25).

**Spike Recommendation:** None

### FEAT-03 — Real-Time Slot Availability Engine

**Verdict:** Hard — the engine must compute genuinely free slots from working hours, buffers, bookings, time blocks, recurring reservations, holds and calendar busy time within roughly one second and refresh within a second of a slot being taken (FEAT-03.SPEC-001 ## Processing Logic; feature-overview.md ## Non-Functional Notes; ASMP-21), while guaranteeing exactly one winner for simultaneous holds and refusing both parties when commit order is ambiguous (FEAT-03.SPEC-005 ## Edge Cases; ASMP-26 correctness bar).

**Required Capabilities:**
- Slot computation merging six sources with duration-plus-buffer fit, minimum notice, horizon and Pro-only exceptions, labeled in the Pro's timezone (FEAT-03.SPEC-001, FEAT-03.SPEC-004 ## Cross-Field Rules; XBR-01, XBR-03, XBR-25)
- Atomic slot-hold creation at checkout and for Pro-created deposit requests (up to 24 hours or 2 hours before the appointment) (FEAT-03.SPEC-002, FEAT-03.SPEC-007; XBR-02)
- Timed hold expiration releasing slots back to public availability after `checkout-hold-timeout-minutes` (FEAT-03.SPEC-003 ## Trigger Definition)
- Live refresh of the client's slot list roughly every second while viewing (FEAT-03.SPEC-001; technical-profile.md Section 3, Real-time signal)
- Consumption of calendar busy periods with a Pro-only reduced-confidence flag (FEAT-03.SPEC-006 ## Data Exchanged)
- Concurrency: first-committed-wins across two clients, client vs. Pro deposit-request hold, and hold vs. Time Block; ambiguous commit order refuses both (FEAT-03.SPEC-005 ## Edge Cases; XBR-01; feature-dependency-map.md, Booking **Contention:** High, Time Block **Contention:**)
- Offline/degraded: calendar-sync outage falls back to Chairtime-only data, invisible to clients, with a Pro-visible confidence banner; stale or duplicate busy data never retroactively alters a returned result (FEAT-03.SPEC-006 ## Degradation Behavior, ## Edge Cases)
- Scale: a few hundred pros, 20–40 bookings a week each, with computation "equally fast as the volume of historical Bookings it must exclude" grows over years (feature-overview.md ## Non-Functional Notes; ASMP-22); concurrent viewers per Pro are the polling multiplier

**Candidate Approaches:** Data-layer enforcement of non-overlap via Postgres exclusion constraints and transactions (landscape Section 3 note) on any Database-area Postgres option, with row locking expressed through Prisma raw SQL, Drizzle ORM or Kysely (ORM / Data Access), or SQL functions called via supabase-js / PostgREST client. Hold counters or short-lived locks could alternatively sit in Upstash Redis or Self-managed Redis / Amazon ElastiCache (Caching & Performance), at the cost of a second source of truth. Refresh transport: Polling with TanStack Query / SWR (Real-time & Collaboration) meets the one-second cadence without extra infrastructure; Supabase Realtime, Ably or Pusher Channels push changes instead, reducing per-viewer query load; Server-Sent Events over own server needs a stateful runtime. Computation can stay uncached on Database indexing and query design (Caching & Performance) given small per-Pro data, while the page shell uses CDN/edge caching. Hold expiry can be lazy (holds carry an expires-at and are ignored once past) or swept via Postgres-native queue and cron, Inngest delayed events, Trigger.dev, or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling).

**Risks & Unknowns:** One-second polling multiplies database reads by concurrent viewers; a viral Instagram post could put many clients on one Pro's page at once (ASMP-21; BRIEF.md Devices & platforms) — the per-viewer query cost at that burst is not quantified upstream. Serverless cold starts on Vercel or Cloudflare Workers (Hosting & Environments) can consume much of the one-second budget. Correctness under concurrent writes depends on the constraint being enforced in the database, not application code (FEAT-03.SPEC-005 ## Edge Cases "the underlying single-Pro-Account data store enforces this"). DST and multi-version rule boundaries inside one requested range (FEAT-03.SPEC-001 ## Edge Cases) are a correctness trap.

**Spike Recommendation:** Build a thin prototype of hold creation plus slot computation on a Postgres candidate with an exclusion constraint, then load-test concurrent hold attempts on one slot and N simultaneous one-second pollers on one Pro. The spike answers: (a) whether database-level exclusion alone yields exactly one winner with no double hold under concurrency, and (b) the viewer count at which polling breaches the one-second target on the candidate hosting — which tells the Architect whether a push transport from Real-time & Collaboration is needed at launch.

### FEAT-04 — Two-Way Calendar Sync

**Verdict:** Research-spike recommended — the specs require both Google and Apple calendars to reflect busy periods and Chairtime bookings "within a couple of minutes" in both directions (FEAT-04.SPEC-003 ## Inbound Events; feature-overview.md ## Non-Functional Notes; success-metrics Calendar Sync Reliability). Google offers push channels, but whether Apple/iCloud busy-time changes can be detected within that window — through direct CalDAV polling or through a unified provider (Nylas, Cronofy) — and what authorization flow iCloud imposes on a Pro are not knowable from the specs or the landscape (landscape Calendar Sync area: "two protocols to build").

**Required Capabilities:**
- Account-linking handshake for Google and Apple, one connection per kind (FEAT-04.SPEC-001, FEAT-04.SPEC-003 ## Data Exchanged, FEAT-04.SPEC-008)
- Busy/free pull only (never event titles) feeding FEAT-03 (FEAT-04.SPEC-003, FEAT-04.SPEC-004; feature-overview.md Data sensitivity)
- Write/move/remove of minimal booking events on create/reschedule/cancel, queued and retried, never blocking the booking (FEAT-04.SPEC-005 ## Processing Logic; FEAT-04.SPEC-003 ## Degradation Behavior)
- Health monitoring, revoked-permission detection and reconciliation on reconnect (FEAT-04.SPEC-006 ## Trigger Definition); in-app reconnection banner (FEAT-04.SPEC-007 ## Channels)
- Concurrency: a Pro disconnect wins over in-flight sync; health status last-write-wins; out-of-order write/remove events resolved by Chairtime event timestamp; duplicate busy updates idempotent (feature-dependency-map.md, Calendar Connection **Contention:**; FEAT-04.SPEC-003 ## Edge Cases)
- Offline/degraded: slow provider keeps last synced busy periods; provider down falls back to Chairtime-only availability with Pro-only confidence narrowing; failed writes queue for retry; reconciliation keeps status at Syncing until drift is resolved (FEAT-04.SPEC-003 ## Degradation Behavior)
- Scale: at most two connections per Pro, a few hundred pros in year one; near-immediate sync latency in both directions (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Direct integration (Google Calendar API plus CalDAV for iCloud) (Calendar Sync) gives full control and no per-account fee but means two protocols, Google watch channels, CalDAV polling and own reconciliation, with a Background Jobs & Scheduling option (Inngest, Trigger.dev, Postgres-native queue and cron, BullMQ with Redis) driving polling and retries. Nylas (Calendar Sync) wraps Google and iCloud CalDAV behind one JSON API with per-account pricing ($1.35–$1.70 calendar-only overage), shrinking the protocol work. Cronofy (Calendar Sync) offers real-time sync and free/busy across Google and Apple at a base of $819/month, which is heavy against a few hundred pros. OneCal Unified Calendar API and Truto unified API are listed in the landscape but with pricing not retrieved. Webhook receipt for provider pushes can sit in any Backend / API Layer candidate.

**Risks & Unknowns:** Apple/iCloud has no third-party push channel noted in the landscape, so near-immediate busy-time detection may require frequent polling per connected Pro — a cost and rate-limit exposure not quantified upstream. iCloud CalDAV access typically depends on an app-specific password rather than a consent screen, which may conflict with the one-tap "authorize" flow the Setup screen describes (FEAT-04.SPEC-001) — unconfirmed in the landscape. A stale Apple busy period is the most direct path to the double-booking the product must never allow (ASMP-33, ASMP-26). Managed providers hold calendar tokens (landscape Maturity & Lock-in), a data-sensitivity consideration for a "Sensitive" personal calendar grant (feature-overview.md Data sensitivity).

**Spike Recommendation:** Connect one Google and one iCloud test calendar through (a) direct Google Calendar API plus CalDAV and (b) Nylas, and measure: the end-to-end latency from a personal-calendar change to busy time visible in Chairtime, the authorization steps a Pro must perform for iCloud, and the polling/request volume per connection per hour. The answer that unblocks the build team: whether the "within a couple of minutes" target is achievable for Apple at an acceptable per-Pro cost, and which Calendar Sync option does it — or whether the product owner must accept a longer Apple latency with the Pro-visible confidence banner as mitigation.

### FEAT-05 — Public Booking Page & Booking Flow

**Verdict:** Standard-with-integration — the multi-step flow (FEAT-05.SPEC-001..005 ## States) is conventional, but it embeds the payment step (FEAT-05.SPEC-004 → FEAT-07.SPEC-005), creates the checkout hold and Pending Payment booking (FEAT-05.SPEC-006 ## Processing Logic) and reads the stored profile photo (FEAT-27.SPEC-012 ## Degradation Behavior), all inside the Instagram in-app browser (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Fast-loading public page per booking link, including forwarded old link names and the "not accepting / not available" gate (FEAT-05.SPEC-001, FEAT-05.SPEC-008; XBR-14, XBR-27)
- Live slot list with hold placed at "Acknowledge & continue" and re-validated before payment (FEAT-05.SPEC-002, FEAT-05.SPEC-006)
- Consent capture with never-pre-checked opt-in and the exact wording stored as evidence (FEAT-05.SPEC-003, FEAT-05.SPEC-007; ASMP-24)
- Policy-version binding and integrity check between acknowledgment and payment (FEAT-05.SPEC-009; XBR-08)
- Accessible, phone-width UI inside an in-app browser (ASMP-28)
- Concurrency: a slot taken, a hold expired, or a policy edited mid-flow is refused with a refreshed list or re-acknowledgment (FEAT-05.SPEC-006, FEAT-05.SPEC-009 ## Edge Cases; feature-dependency-map.md, Cancellation Policy **Contention:**)
- Offline/degraded: booking and paying require a live connection and say so plainly; an already-loaded page stays visible (technical-profile.md Section 3, Offline signal: FEAT-05.SPEC-001, .004, .005); photo outage shows the no-photo page (FEAT-27.SPEC-012 ## Degradation Behavior)
- Scale: highest-traffic surface; slots within ~1 second and full booking in under one minute (ASMP-21; feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Server-rendered public pages via Next.js (App Router), React Router v7, SvelteKit or Nuxt (Frontend Framework), with SvelteKit's smaller bundles relevant to in-app browsers and Astro with island components suited to the static shell but less to the stateful flow. Page shell via CDN/edge caching of public booking page (Caching & Performance) with slot data fetched live. Flow state via Zustand, TanStack Query or Framework built-ins (State Management); accessible pickers via React Aria Components, Radix UI primitives or shadcn/ui (CSS / Styling component layer). Payment element per FEAT-07's Payments & Billing options; photo from FEAT-27's File & Object Storage options.

**Risks & Unknowns:** Embedded card entry, 3-D Secure challenges and wallet buttons can behave differently inside the Instagram in-app browser (BRIEF.md Devices & platforms); the landscape does not record per-processor in-app-browser behavior. Funnel analytics in the flow must respect the privacy posture (ASMP-23) — relevant when choosing among Analytics & Product Telemetry options.

**Spike Recommendation:** None

### FEAT-06 — Client Booking Identity

**Verdict:** Standard-with-integration — password-free access links are delivered through the transactional text and email capabilities (FEAT-06.SPEC-006 ## Channels, ## Delivery Rules; FEAT-08.SPEC-012, FEAT-08.SPEC-013) and governed by expiry, single-use and per-phone rate-limit rules (FEAT-06.SPEC-007 ## Authorization Rules).

**Required Capabilities:**
- Unguessable bearer tokens: on-demand single-use for 30 minutes, booking-specific valid until the appointment passes (FEAT-06.SPEC-007; XBR-18; ASMP-30)
- Per-phone-number request rate limiting at issuance (FEAT-06.SPEC-007 ## Authorization Rules)
- Phone-to-Client matching scoped to one Pro, with a hard no-cross-client boundary (FEAT-06.SPEC-008)
- Link delivery by text with consent, otherwise email, with retry on failure (FEAT-06.SPEC-006)
- Concurrency: a single-use link opened on two devices — first use wins, the second sees "request a new link" (feature-dependency-map.md, Access Link **Contention:**; FEAT-06.SPEC-002 ## Edge Cases)
- Offline/degraded: online-only identity check by design; already-loaded bookings list/detail stay visible (feature-overview.md ## Non-Functional Notes; technical-profile.md Section 3, Offline: FEAT-06.SPEC-001, .003, .004, .005); delivery failure follows FEAT-08.SPEC-009 retry/fallback
- Scale: active link volume bounded by 30-minute/per-appointment life, tracking booking volume (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Custom token issuance on any Backend / API Layer candidate with tokens hashed in a Database-area Postgres option; Supabase Auth or Better Auth / Auth.js (Authentication & Identity) can supply session plumbing after redemption, while the landscape notes the custom access-link flow "needs own code" under Supabase Auth. Rate limiting via Upstash Redis (Caching & Performance) or a Postgres counter table. Delivery via Twilio, Telnyx (text) and Resend, Postmark or Amazon SES with SNS (email) (Email & Messaging Delivery).

**Risks & Unknowns:** SMS pumping/toll fraud against the link-request endpoint can generate paid texts; the rate limit mitigates but its value is not specified. Link previews in messaging apps can "open" single-use links before the client taps them, consuming the single use (FEAT-06.SPEC-007 single-use rule) — a known pattern the redemption design must tolerate. The Access Link is classified Security-sensitive (dependency map Data Sensitivity).

**Spike Recommendation:** None

### FEAT-07 — Deposit Payment at Booking

**Verdict:** Standard-with-integration — card authorization/capture, payout routing to the Pro's connected account and fee reporting are a payment-processor contract (FEAT-07.SPEC-005 ## Capability Category, ## Data Exchanged); exactly-once outcomes via idempotency (FEAT-07.SPEC-004 ## Edge Cases) are established processor patterns.

**Required Capabilities:**
- Hosted/embedded card entry so the product never holds card data (FEAT-07.SPEC-001; SC-11; ASMP-31)
- Charge exactly the locked deposit, route to the Pro's payout account with zero platform fee, report the processor fee (FEAT-07.SPEC-003, FEAT-07.SPEC-005; XBR-05, XBR-07)
- Atomic Pending Payment → Confirmed flip with one Deposit Transaction per booking on the capture event (FEAT-07.SPEC-002 ## Processing Logic)
- Idempotent webhook handling for duplicate and out-of-order capture/decline events (FEAT-07.SPEC-005 ## Edge Cases)
- Concurrency: payment completing against an expiring hold or a Pro action — first committed wins; a capture for a booking that has left Pending Payment is resolved by FEAT-07.SPEC-004, not dropped (FEAT-07.SPEC-005 ## Edge Cases; feature-dependency-map.md, Deposit Transaction **Contention:**)
- Offline/degraded: slow processor shows "Processing payment, do not close this page"; processor down disables Pay while the hold keeps counting down; outcome unknown creates no Deposit Transaction until a result arrives (FEAT-07.SPEC-005 ## Degradation Behavior, ## Edge Cases)
- Scale: at most one deposit per booking at 20–40 bookings/week per Pro; payment step within the one-minute booking budget (feature-overview.md ## Non-Functional Notes; ASMP-21)

**Candidate Approaches:** Stripe (Connect plus Billing) (Payments & Billing) offers connected accounts, destination charges and idempotency keys in one platform that also covers FEAT-18; Adyen for Platforms offers split payments with an enterprise sales process; PayPal Complete Payments offers marketplace payouts with varying market coverage; Square (Payments and Connect) uses seller-authorized accounts, a different funds model from platform-held connected accounts. Webhook receipt on any Backend / API Layer candidate; failure tracing via Sentry or OpenTelemetry with Honeycomb (Observability & Operations).

**Risks & Unknowns:** The "zero platform fee" rule (XBR-07) interacts with connected-account pricing — Stripe Connect lists a $2 per monthly active account fee and payout fees (landscape Payments & Billing) that the platform absorbs; whether that fits the `subscription-price` is a business question. A capture arriving after hold expiry and a re-sold slot is the path to double-booking or a lost deposit (ASMP-26); FEAT-07.SPEC-004 governs it but the refund-or-honor behavior depends on FEAT-03's constraint design. In-app-browser 3-D Secure behavior is unverified (see FEAT-05).

**Spike Recommendation:** None

### FEAT-08 — Automated Booking Messaging

**Verdict:** Standard-with-integration — the feature is defined by two external capabilities, text and email, each with delivery-status reporting (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Inbound Events, ## Degradation Behavior), driven by scheduled reminders in an 8am–9pm window (FEAT-08.SPEC-007 ## Processing Logic) and retry-once-then-email fallback (FEAT-08.SPEC-009).

**Required Capabilities:**
- Transactional SMS with delivery status (Queued/Sent/Delivered/Failed) and a timeout that converts silence into Failed (FEAT-08.SPEC-012 ## Edge Cases)
- Transactional email fallback with delivery status (FEAT-08.SPEC-013)
- Reminder scheduling at `reminder-lead-time-days`, shifted into the Pro's local daytime window; confirmations within about a minute of payment (FEAT-08.SPEC-007; XBR-16; ASMP-29)
- Consent-based channel selection read on every send (FEAT-08.SPEC-011; XBR-15)
- One-tap reply routing and booking-specific manage-link issuance (FEAT-08.SPEC-008, FEAT-08.SPEC-010); .ics add-to-calendar link in the confirmation (FEAT-08.SPEC-001)
- In-app Pro notifications and attention alerts updating without refresh (FEAT-08.SPEC-005, FEAT-08.SPEC-006; technical-profile.md Section 3, Real-time)
- Concurrency: duplicate and out-of-order status events resolved by provider event time; a reply tap after the booking changed is evaluated against current state (FEAT-08.SPEC-012 ## Edge Cases)
- Offline/degraded: every send is asynchronous; provider slow/down records Failed after timeout and hands off to retry/fallback; the in-app copy never depends on the provider (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Degradation Behavior)
- Scale: at least a confirmation and a reminder per booking at 20–40 bookings/week per Pro; immutable Message records accumulate over years (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Text via Twilio (Programmable Messaging and WhatsApp) — which also carries FEAT-26's WhatsApp channel under one account — or Telnyx at a lower per-part price (Email & Messaging Delivery); email via Resend, Postmark (transactional-first delivery tracking) or Amazon SES with SNS. Scheduling and retry via Inngest (delayed events and retries), Trigger.dev, Postgres-native queue and cron, or BullMQ with Redis where a long-running worker exists; Platform cron (Vercel Cron Jobs) offers sweeps without built-in retry (Background Jobs & Scheduling). In-app updates via Polling with TanStack Query / SWR or Supabase Realtime / Ably / Pusher Channels (Real-time & Collaboration). Local-hour computation via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** US A2P 10DLC brand/campaign registration (landscape Twilio and Telnyx rows) gates any production texting and can take time; unregistered traffic is filtered by carriers. Carrier fees on top of per-segment price are not totaled upstream. No landscape option names a library for generating the .ics attachment/link (FEAT-08.SPEC-001) — a small landscape gap recorded in Section 5. Messages carry the Pro's studio address, which may be a home address (feature-overview.md Data sensitivity), so provider log retention matters.

**Spike Recommendation:** None

### FEAT-09 — Cancellation & No-Show Policy Engine

**Verdict:** Standard-with-integration — immutable policy versions and binary outcome rules are straightforward (FEAT-09.SPEC-002, FEAT-09.SPEC-003), but automatic full refunds through the processor with indefinite, idempotent retry (FEAT-09.SPEC-005 ## Degradation Behavior, FEAT-09.SPEC-006 ## Edge Cases) make an external payment contract part of the feature.

**Required Capabilities:**
- Immutable Cancellation Policy versions with each booking bound to the acknowledged version and a rendered cut-off time in the Pro's timezone (FEAT-09.SPEC-002; XBR-08)
- Instant outcome evaluation on every cancel/reschedule/no-show/Pro cancel (FEAT-09.SPEC-004; XBR-09)
- Refund request drawing on the Pro's payout account, retried every `refund-retry-interval-hours` until success, never dropped (FEAT-09.SPEC-005, FEAT-09.SPEC-006; XBR-10)
- Hand-off of a paid balance for full refund even when the deposit is forfeited (FEAT-09.SPEC-005 ## Edge Cases; XBR-23)
- Concurrency: one terminal outcome per deposit; automation and manual retries tied to the same idempotency reference; out-of-order refund events resolved to the true state (feature-dependency-map.md, Deposit Transaction **Contention:**; FEAT-09.SPEC-005 ## Edge Cases)
- Offline/degraded: processor unreachable leaves the deposit at Refund in Progress, shown as "in progress" to both parties, with a Pro attention flag (FEAT-09.SPEC-005 ## Degradation Behavior)
- Scale: outcome updates track cancellation/no-show volume, well within 20–40 bookings/week per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Refunds via Stripe (Connect plus Billing) with idempotency keys and refund-from-connected-account semantics, or Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing), each differing in how a refund draws on the seller's balance. Retry cadence via Inngest durable steps, Trigger.dev, Postgres-native queue and cron, or BullMQ with Redis (Background Jobs & Scheduling). Alerting on stuck refunds via Sentry, Better Stack or Grafana Cloud (Observability & Operations).

**Risks & Unknowns:** A refund drawn on a connected account with insufficient balance (because deposits were already paid out) is the most likely "cannot complete" cause (FEAT-09.SPEC-005 ## Degradation Behavior, Rejects column); retry succeeds only once the Pro's balance recovers, and processor rules for negative balances differ across Payments & Billing options. Indefinite retry needs monitoring to avoid silent long-tail refunds.

**Spike Recommendation:** None

### FEAT-10 — Client-Initiated Cancel/Reschedule

**Verdict:** Standard-with-integration — the commit coordinates deposit outcome, a new deposit charge on a late reschedule, calendar mirroring, activity logging and freed-slot hand-off (FEAT-10.SPEC-004 ## Processing Logic; FEAT-10.SPEC-003), relying on payment and calendar capabilities owned by FEAT-07/FEAT-09/FEAT-04; the contention rule is explicit and implementable (feature-dependency-map.md, Booking **Contention:**).

**Required Capabilities:**
- Eligibility and live cancellation-window countdown against the bound policy version (FEAT-10.SPEC-005)
- Reschedule through the same live slot list and hold machinery as a new booking (FEAT-10.SPEC-002; FEAT-03)
- Late reschedule: deposit kept plus a new deposit charged before confirming (FEAT-10.SPEC-003; XBR-09)
- Multi-side-effect commit: outcome, refund hand-off, calendar move/remove, activity event, freed slot to waitlist (FEAT-10.SPEC-004; XBR-13, XBR-21, XBR-28); client and Pro notices (FEAT-10.SPEC-006)
- Concurrency: High — client action races Pro cancel/reschedule/no-show/complete and automations; first committed transition wins, the other actor sees current state (feature-dependency-map.md, Booking **Contention:**)
- Offline/degraded: cancel/reschedule require a live connection; loaded booking screens stay visible (technical-profile.md Section 3, Offline: FEAT-10.SPEC-001..003); downstream refund/calendar failures do not roll back the commit (FEAT-04.SPEC-003, FEAT-09.SPEC-005 ## Degradation Behavior)
- Scale: target of 85% of cancellations/reschedules self-served — a meaningful share of weekly traffic; slot refresh within ~1 second (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** State-guarded transitions (compare-and-set on booking status/version) inside a transaction on any Database-area Postgres option, via Prisma, Drizzle ORM, Kysely or SQL functions through supabase-js / PostgREST client (ORM / Data Access). Side effects after commit either via an outbox table drained by Postgres-native queue and cron, or via events to Inngest or Trigger.dev (Background Jobs & Scheduling). New-deposit charge through FEAT-07's Payments & Billing option.

**Risks & Unknowns:** Late reschedule couples a new card charge to a slot hold — if the charge fails after the old booking is cancelled the client could lose both slots; FEAT-10.SPEC-003/SPEC-004 order the steps, but atomicity across processor and database needs care. Side-effect fan-out must be idempotent so retries do not double-notify or double-refund (XBR-10).

**Spike Recommendation:** None

### FEAT-11 — No-Show Marking & Deposit Forfeiture

**Verdict:** Straightforward — marking transitions Booking to No-Show and Deposit Transaction to Forfeited in one step, with a 24-hour undo restoring prior states (FEAT-11.SPEC-002, FEAT-11.SPEC-003 ## Processing Logic); the deposit is already captured, so no processor call is made (feature-overview.md Compliance flags).

**Required Capabilities:**
- Single-transaction dual-entity transition with outcome derived from the bound policy (FEAT-11.SPEC-002; XBR-09)
- Time-bounded undo within `no-show-undo-grace-window-hours` and marking window closing at auto-completion (FEAT-11.SPEC-004; XBR-12)
- Own-bookings-only authorization for the Pro (FEAT-11.SPEC-004 ## Authorization Rules)
- Concurrency: marking races a client cancel or a goodwill refund — reject-with-refresh, one terminal outcome per deposit (feature-dependency-map.md, Booking and Deposit Transaction **Contention:**)
- Offline/degraded: marking needs a live connection and says so; a failed write is retried and flagged, never silently queued (feature-overview.md ## Non-Functional Notes; ASMP-27; technical-profile.md Section 3, Offline: FEAT-11.SPEC-001)
- Scale: a subset of 20–40 bookings/week per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Transactional status updates with version checks on any Database-area Postgres option through Prisma, Drizzle ORM, Kysely or supabase-js / PostgREST client SQL functions (ORM / Data Access); undo-window enforcement by timestamp comparison at request time, needing no scheduler.

**Risks & Unknowns:** None identified — no external call and small volume; the only coupling is the dispute overlay (FEAT-16) that must coexist with Forfeited, which the Deposit Transaction contention rule already specifies.

**Spike Recommendation:** None

### FEAT-12 — Pro Daily Schedule Dashboard

**Verdict:** Straightforward — a read-heavy dashboard (FEAT-12.SPEC-001..003 ## States), a daily auto-completion sweep (FEAT-12.SPEC-004 ## Trigger Definition), attention-flag aggregation from other features' states (FEAT-12.SPEC-005) and masked support views (FEAT-12.SPEC-008); no external service.

**Required Capabilities:**
- Today/upcoming list with paid badge, balance due, "I'll be there" status and sync-reliability indicator (FEAT-12.SPEC-001, FEAT-12.SPEC-007)
- Auto-completion 7 days after appointment (FEAT-12.SPEC-004; XBR-12)
- De-duplicated attention items from sync health, delivery failures, refunds, disputes, setup conflicts (FEAT-12.SPEC-005)
- Past bookings browse by date over multi-year history (FEAT-12.SPEC-003)
- Live-updating notices without manual refresh (technical-profile.md Section 3, Real-time: FEAT-08.SPEC-005 citing FEAT-12)
- Concurrency: actions from the dashboard participate in the Booking reject-with-refresh rule (feature-dependency-map.md, Booking **Contention:**; FEAT-12.SPEC-006)
- Offline/degraded: most recently loaded schedule readable offline; actions require reconnecting (feature-overview.md ## Non-Functional Notes, Offline / degraded posture; ASMP-27)
- Scale: next booking's status identifiable within a few seconds; past browse equally responsive as history grows (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Offline readability via Client-side cache (TanStack Query persistence, service worker) (Caching & Performance). Live notices via Polling with TanStack Query / SWR, or Supabase Realtime, Ably or Pusher Channels (Real-time & Collaboration). Sweep via Platform cron (Vercel Cron Jobs), Postgres-native queue and cron, Inngest or Trigger.dev (Background Jobs & Scheduling). Aggregation either computed at read time with Database indexing and query design, or materialized by event handlers.

**Risks & Unknowns:** Service-worker persistence is unreliable in some in-app and private browsing contexts, so the offline-read promise (ASMP-27) may not hold uniformly on every Pro device; the Pro mostly uses a normal mobile browser, which lowers this risk.

**Spike Recommendation:** None

### FEAT-13 — Client Record Management

**Verdict:** Straightforward — contact/note CRUD (FEAT-13.SPEC-001, FEAT-13.SPEC-002), per-Pro phone uniqueness (FEAT-13.SPEC-005) and a hard-delete cascade that keeps de-identified financial and timeline records (FEAT-13.SPEC-004 ## Processing Logic, FEAT-13.SPEC-006) are established patterns.

**Required Capabilities:**
- Private-note field excluded from every Support view (FEAT-13.SPEC-005 ## Authorization Rules; XBR-24)
- Deletion eligibility blocked by upcoming bookings; irreversible delete with consent cascade and de-identification of retained records (FEAT-13.SPEC-003, FEAT-13.SPEC-004, FEAT-13.SPEC-006; XBR-19)
- Phone change invalidates access links and texting consent (feature-dependency-map.md, Client **Contention:**; XBR-15)
- Concurrency: Pro edits vs. client self-edits last-write-wins; two first bookings with the same phone merge to one record; a concurrent edit after deletion is refused with refresh (feature-dependency-map.md, Client **Contention:**)
- Offline/degraded: last loaded client list readable offline; opening, editing or deleting a record requires connection (feature-overview.md ## Non-Functional Notes, Offline / degraded posture)
- Scale: 100–500 clients per Pro; record loads instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Row-level security on Supabase Postgres, or application-layer tenant scoping with Prisma, Drizzle ORM or Kysely against Neon, Crunchy Bridge or Amazon RDS / Aurora PostgreSQL (Database, ORM / Data Access); a unique index on (pro, normalized phone) handles the merge race. De-identification as a transactional update of retained rows. Offline list via Client-side cache (Caching & Performance).

**Risks & Unknowns:** De-identification must also reach copies outside the primary database — messaging-provider logs, analytics events, error-tracking payloads (Observability & Operations, Analytics & Product Telemetry options) — for the deletion commitment (feature-overview.md Compliance flags) to be complete.

**Spike Recommendation:** None

### FEAT-14 — Messaging Consent Management

**Verdict:** Standard-with-integration — consent state is simple, but revocation arrives as inbound "STOP" replies through the text provider (FEAT-14.SPEC-004 ## Trigger Definition; FEAT-08.SPEC-012) and must be honored on the very next message (feature-overview.md ## Non-Functional Notes; XBR-15).

**Required Capabilities:**
- Consent record with state, timestamp, channel and exact wording shown (FEAT-14.SPEC-003; ASMP-24)
- Immediate revocation on opt-out link or inbound STOP; opt-out confirmation message (FEAT-14.SPEC-004, FEAT-14.SPEC-009)
- Re-grant from client preferences; phone change invalidates consent (FEAT-14.SPEC-005, FEAT-14.SPEC-008)
- Textability determination consumed by every send (FEAT-14.SPEC-007)
- Concurrency: STOP vs. in-app re-grant resolved by most recent explicit action timestamp, defaulting to no-text when uncertain (FEAT-14.SPEC-006; feature-dependency-map.md, Messaging Consent **Contention:**)
- Offline/degraded: preference screen needs a live connection; if consent state is uncertain the no-text state applies (feature-dependency-map.md, Messaging Consent **Contention:**; technical-profile.md Section 3, Offline: FEAT-14.SPEC-001)
- Scale: one consent record per client–Pro pair plus transition history (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Inbound STOP via Twilio (Programmable Messaging and WhatsApp) or Telnyx webhooks (Email & Messaging Delivery) received by any Backend / API Layer candidate (landscape Section 3: inbound STOP needs a public webhook). Twilio and Telnyx also apply carrier-level opt-out handling on their side, which the product's own consent record must mirror. Append-only consent history in any Database-area Postgres option.

**Risks & Unknowns:** Provider-level STOP handling and product-level consent can diverge (e.g., the provider blocks a number the product believes re-granted) — reconciliation between the two is not specified. STOP replies arrive on the sending number, so a shared number across all pros must map a STOP to the right client–Pro pair; number strategy (shared vs. per-Pro) is not decided upstream.

**Spike Recommendation:** None

### FEAT-15 — Pro Onboarding & Setup Wizard

**Verdict:** Straightforward — a wizard shell over other features' screens (FEAT-15.SPEC-001), a resumable progress record (FEAT-15.SPEC-004 ## Processing Logic) and a go-live rule re-evaluated on step completion or upstream status change (FEAT-15.SPEC-005, FEAT-15.SPEC-007; XBR-26); payout and subscription integrations are owned by FEAT-28 and FEAT-18.

**Required Capabilities:**
- Progress record with exact resume point, optional calendar step (FEAT-15.SPEC-004, FEAT-15.SPEC-006)
- Go-live evaluation triggered by asynchronous upstream events (payout verification, subscription) (FEAT-15.SPEC-005)
- First Cancellation Policy version with default window (FEAT-15.SPEC-002; `cancellation-window-default-hours`)
- Welcome confirmation when the link goes live (FEAT-15.SPEC-008)
- Concurrency: step completion and upstream status events can arrive together; go-live re-evaluation must be idempotent (FEAT-15.SPEC-005 ## Edge Cases; feature-dependency-map.md, Pro Account **Contention:**)
- Offline/degraded: failed step preserves entered values with retry; go-live preview screen keeps loaded content (feature-overview.md ## Non-Functional Notes; technical-profile.md Section 3, Offline: FEAT-15.SPEC-003)
- Scale: one progress record per new Pro, a few hundred in year one (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Wizard state carried in URL and server progress record via Framework built-ins, or Zustand/TanStack Query (State Management), on any Frontend Framework candidate. Go-live evaluation as a function called from payout/subscription webhook handlers on any Backend / API Layer candidate, or as an event consumer in Inngest or Trigger.dev (Background Jobs & Scheduling).

**Risks & Unknowns:** Payout verification can stay pending for days (FEAT-15.SPEC-003 pending hand-over), so the go-live trigger depends on FEAT-28's inbound events being reliable; a missed event would silently keep a ready Pro offline.

**Spike Recommendation:** None

### FEAT-16 — Booking & Payment Activity Record

**Verdict:** Standard-with-integration — the append-only log (FEAT-16.SPEC-002, FEAT-16.SPEC-005) is conventional, but inbound card-issuer dispute notices from the processor (FEAT-16.SPEC-003 ## Inbound Events, ## Degradation Behavior) and a downloadable dispute summary (FEAT-16.SPEC-004) add an external contract and file generation.

**Required Capabilities:**
- Immutable, append-only Activity Events written by many features (FEAT-16.SPEC-002; XBR-21)
- Dispute webhook: flag booking, set Disputed overlay without erasing outcome, record event (FEAT-16.SPEC-003; XBR-22)
- Plain-language dispute summary assembled from loaded timeline and handed over as a file (FEAT-16.SPEC-004)
- Concurrency: none on entries (append-only); duplicate and out-of-order dispute events idempotent, "concluded" held until the original notice arrives (feature-dependency-map.md, Activity Event **Contention:** None; FEAT-16.SPEC-003 ## Edge Cases)
- Offline/degraded: processor dispute channel down means notices arrive late with original timestamps; timeline shows only known disputes with no error state (FEAT-16.SPEC-003 ## Degradation Behavior); loaded timeline readable offline (technical-profile.md Section 3, Offline: FEAT-16.SPEC-001)
- Scale: a handful of events per booking over multi-year history; timeline loads instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Append-only table with update/delete privileges revoked, on any Database-area Postgres option; dispute webhooks from Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing). The summary can be generated on the fly and streamed, or stored in Supabase Storage, Cloudflare R2, Amazon S3 or Vercel Blob (File & Object Storage).

**Risks & Unknowns:** The file format of the dispute summary is not specified and the landscape names no document-generation library — recorded as a gap in Section 5. Retention after client deletion keeps only de-identified events (XBR-19), so event payloads must avoid embedding raw personal data that cannot be scrubbed.

**Spike Recommendation:** None

### FEAT-17 — Manual Time Blocking

**Verdict:** Straightforward — block CRUD (FEAT-17.SPEC-001, FEAT-17.SPEC-002), conflict detection against confirmed bookings (FEAT-17.SPEC-004 ## Processing Logic), recurring occurrence generation (FEAT-17.SPEC-005) and automatic expiry (FEAT-17.SPEC-007) are standard scheduling patterns; the one-second visibility bar rides on FEAT-03.

**Required Capabilities:**
- One-off and recurring blocks in the Pro's timezone with private labels (FEAT-17.SPEC-001, FEAT-17.SPEC-008; ASMP-25)
- Conflict review and explicit resolution (cancel, reschedule, keep as exception), never silent (FEAT-17.SPEC-003, FEAT-17.SPEC-006; XBR-11)
- Rolling generation of future occurrences with the same conflict detection (FEAT-17.SPEC-005)
- Concurrency: block vs. client checkout — first committed wins; block edits between Pro sessions last-write-wins (feature-dependency-map.md, Time Block **Contention:**; FEAT-03.SPEC-005 ## Edge Cases)
- Offline/degraded: creating/removing a block needs a live connection; loaded block list readable (feature-overview.md Compliance flags; technical-profile.md Section 3, Offline: FEAT-17.SPEC-001, .002)
- Scale: small per-Pro set; block appears/disappears from slot list within ~1 second (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Blocks stored as ranges in any Database-area Postgres option and included in the same exclusion-constraint/transaction scheme as FEAT-03. Recurring generation either materialized on a schedule (Postgres-native queue and cron, Inngest, Trigger.dev, Platform cron (Vercel Cron Jobs) — Background Jobs & Scheduling) or expanded at read time from the pattern; recurrence math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Materialized recurring occurrences create conflicts at generation time that the Pro was not present for (FEAT-17.SPEC-005); these route to the dashboard, which is specified, but the generation horizon choice affects how far ahead conflicts surface.

**Spike Recommendation:** None

### FEAT-18 — Pro Subscription Billing & Account Management

**Verdict:** Standard-with-integration — subscribe/update/cancel calls and inbound renewal outcomes through the payment processor (FEAT-18.SPEC-006 ## Data Exchanged, ## Degradation Behavior), with a 7-day grace clock and pause trigger (FEAT-18.SPEC-003, FEAT-18.SPEC-004).

**Required Capabilities:**
- Single-tier recurring subscription with card on file at the processor (FEAT-18.SPEC-001, FEAT-18.SPEC-005; SC-11)
- Renewal outcome processing and grace-period timer leading to a system-imposed pause (FEAT-18.SPEC-003, FEAT-18.SPEC-004; XBR-14)
- Cancellation at period end, including cancellation invoked by account closure (FEAT-18.SPEC-006; XBR-20)
- Billing notifications incl. 30-day price-change notice (FEAT-18.SPEC-007)
- Concurrency: payment-method update during a renewal retry — the processor's recorded outcome is authoritative; stale screen refused with refresh (feature-dependency-map.md, Subscription **Contention:**; FEAT-18.SPEC-006 ## Edge Cases)
- Offline/degraded: processor down disables subscribe/update/cancel with plain messages; a down processor during renewal is inconclusive, not a failure (FEAT-18.SPEC-006 ## Degradation Behavior); last-known plan status readable offline (feature-overview.md ## Non-Functional Notes)
- Scale: one subscription per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe (Connect plus Billing) (Payments & Billing) runs the renewal schedule itself and reports outcomes by webhook, sharing a platform with deposits; Adyen for Platforms, PayPal Complete Payments or Square would pair with their own recurring-billing products (not detailed in the landscape). Grace-period timer via Inngest delayed events, Trigger.dev, Postgres-native queue and cron or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling).

**Risks & Unknowns:** If the processor runs renewals, the product's grace clock must key off processor events rather than its own schedule, or two clocks disagree (FEAT-18.SPEC-003 vs. FEAT-18.SPEC-006). The landscape does not detail recurring-billing capabilities for non-Stripe options.

**Spike Recommendation:** None

### FEAT-19 — Platform Support Read-Only Access

**Verdict:** Straightforward — a role with structurally no write path, one-account-at-a-time scoping and field masking (FEAT-19.SPEC-004 ## Authorization Rules), plus view logging to the Pro-visible access log (FEAT-19.SPEC-002, FEAT-19.SPEC-003); no external service.

**Required Capabilities:**
- Read-only Platform Operator role; opening a new lookup ends the prior session (FEAT-19.SPEC-001, FEAT-19.SPEC-004)
- Exclusion of private notes, bank/identity details and sign-in codes from every support surface (XBR-24; ASMP-30)
- Support-view events logged with reason/ticket reference (FEAT-19.SPEC-002; XBR-21)
- Concurrency: N/A — support is view-only; its only writes are append-only log events (feature-dependency-map.md, Activity Event **Contention:** None)
- Offline/degraded: N/A — no offline behavior is specified; the view "loads instantly" on a live connection (feature-overview.md ## Non-Functional Notes)
- Scale: bounded by help-request count (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Role and claims from Supabase Auth, Clerk (organizations/roles), WorkOS AuthKit, Auth0 or Better Auth / Auth.js (Authentication & Identity); read-only enforcement either via row-level-security policies on Supabase Postgres or a dedicated read-only database role/connection for support queries on any Postgres option; masking in the Backend / API Layer.

**Risks & Unknowns:** "Structurally read-only" (FEAT-19.SPEC-004) is only as strong as the enforcement layer; application-level checks alone leave a regression path across 30 features' write endpoints.

**Spike Recommendation:** None

### FEAT-20 — Waitlist for Cancelled Slots

**Verdict:** Standard-with-integration — matching on freed-slot signals and a 30-minute claim window (FEAT-20.SPEC-004, FEAT-20.SPEC-005 ## Trigger Definition) depend on immediate delivery of opening notices through the messaging capability (FEAT-20.SPEC-008 ## Delivery Rules; FEAT-08.SPEC-012/013).

**Required Capabilities:**
- Event-driven matching on cancellation, Requested → Notified transitions (FEAT-20.SPEC-005; XBR-28)
- 30-minute priority window layered onto FEAT-03 availability, then return to general availability (FEAT-20.SPEC-004; XBR-02)
- Claim conversion through the ordinary booking flow; expiry of unclaimed and elapsed entries (FEAT-20.SPEC-006, FEAT-20.SPEC-007)
- Opening notice sent immediately (daytime rule read as not applying); expiry notice within the daytime window (FEAT-20.SPEC-008, FEAT-20.SPEC-009; feature-overview.md Compliance flags)
- Concurrency: several notified clients race for one slot — first to complete booking wins; leave wins over pending notification (feature-dependency-map.md, Waitlist Entry **Contention:**; XBR-01)
- Offline/degraded: messaging failures follow FEAT-08.SPEC-009 retry/fallback; My Waitlists screen readable when loaded (technical-profile.md Section 3, Offline: FEAT-20.SPEC-002)
- Scale: at most 3 active entries per client per Pro, low hundreds per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Freed-slot events delivered to a handler via Inngest or Trigger.dev events, or an outbox drained by Postgres-native queue and cron (Background Jobs & Scheduling); claim-window expiry via delayed jobs or lazy expiry checks. Priority window modeled as a waitlist-scoped hold in the same data-layer scheme as FEAT-03. Notices via FEAT-08's Email & Messaging Delivery options.

**Risks & Unknowns:** The priority window must be enforced by the slot engine, otherwise a public client can book a slot during the window — this extends FEAT-03's contention surface. Retry-then-email fallback delay eats into the 30-minute window (FEAT-08.SPEC-009).

**Spike Recommendation:** None

### FEAT-21 — Recurring/Standing Appointments

**Verdict:** Standard-with-integration — occurrence generation within the horizon with slot validation (FEAT-21.SPEC-004 ## Processing Logic), per-occurrence deposit links and release at cut-off (FEAT-21.SPEC-005 ## Trigger Definition) and three notification types (FEAT-21.SPEC-007..009) depend on payment and messaging capabilities and scheduled jobs.

**Required Capabilities:**
- Series with 1–12 week interval; occurrences generated up to the booking horizon (FEAT-21.SPEC-001, FEAT-21.SPEC-003)
- Generation through the same slot validation; conflicts produce advance notice with re-pick prompt (FEAT-21.SPEC-004, FEAT-21.SPEC-008)
- Deposit request `recurring-occurrence-deposit-lead-days` before each occurrence; release if unpaid at the cancellation cut-off (FEAT-21.SPEC-005; XBR-02, XBR-05)
- Whole-series vs. single-occurrence cancellation (FEAT-21.SPEC-006); Pro-side management (FEAT-21.SPEC-010)
- Concurrency: client and Pro change series/occurrence — reject-with-refresh, first committed wins (feature-dependency-map.md, Recurring Series **Contention:**)
- Offline/degraded: generation retried `occurrence-generation-retry-count` times before flagging the gap to the Pro (platform-parameters.md); messaging degradation per FEAT-08.SPEC-009
- Scale: working set bounded by the horizon (default 8 weeks); small relative to ordinary bookings (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Rolling generation via Postgres-native queue and cron, Inngest cron plus steps, Trigger.dev schedules or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling). Deposit links as processor-hosted payment links or the FEAT-07 payment page with Stripe (Connect plus Billing) or other Payments & Billing options — each occurrence paid fresh since card data is never stored (XBR-05). Recurrence math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Occurrences reserving slots at generation (FEAT-03.SPEC-001 ## Edge Cases) means a failed or late generation run opens those slots to the public; generation reliability therefore touches the double-booking bar. Interval arithmetic across DST changes can shift local times by an hour.

**Spike Recommendation:** None

### FEAT-22 — In-App Balance Payment

**Verdict:** Standard-with-integration — a second card charge with payout routing and full refunds on cancellation through the processor (FEAT-22.SPEC-005 ## Degradation Behavior, ## Edge Cases), plus a payment/cancellation race rule (FEAT-22.SPEC-004).

**Required Capabilities:**
- Balance computed once (price − deposit) and charged exactly (FEAT-22.SPEC-003; XBR-23)
- Capture → Balance Payment record → booking marked fully paid, visible on the dashboard without refresh (FEAT-22.SPEC-002; technical-profile.md Section 3, Real-time)
- Full refund of a paid balance on any cancellation, with idempotent indefinite retry (FEAT-22.SPEC-005)
- Concurrency: client paying while the Pro cancels — cancellation first blocks payment; payment first is refunded in full (feature-dependency-map.md, Balance Payment **Contention:**; FEAT-22.SPEC-004)
- Offline/degraded: processor down disables Pay with an in-person fallback; unknown outcome creates no record until resolved; refund "in progress" states (FEAT-22.SPEC-005 ## Degradation Behavior)
- Scale: at most one balance payment per booking, a fraction of booking volume (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Same Payments & Billing options as FEAT-07 — Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments, Square — reusing FEAT-07's webhook and idempotency machinery; refund retry via FEAT-09's Background Jobs & Scheduling option.

**Risks & Unknowns:** Balance refunds drawn on a connected account after payout share FEAT-09's insufficient-balance exposure. Processor minimum charge applies to small balances (`minimum-chargeable-deposit` reused, platform-parameters.md).

**Spike Recommendation:** None

### FEAT-23 — Tipping at Checkout

**Verdict:** Standard-with-integration — the tip is charged and refunded as part of FEAT-22's balance payment through FEAT-22.SPEC-005 (feature-dependency-map.md ## External Touchpoints; FEAT-23.SPEC-003), so it inherits an external payment contract while adding only validation and a UI step (FEAT-23.SPEC-001, FEAT-23.SPEC-002).

**Required Capabilities:**
- Optional, never-defaulted, non-negative tip step inside the balance flow (FEAT-23.SPEC-001, FEAT-23.SPEC-002)
- Tip passed wholly to the Pro with zero platform cut, refunded with the balance on cancellation (FEAT-23.SPEC-003; XBR-07, XBR-23)
- Concurrency: inherits FEAT-22's payment-vs-cancellation rule (feature-dependency-map.md, Balance Payment **Contention:**)
- Offline/degraded: inherits FEAT-22.SPEC-005 ## Degradation Behavior; a tip must never be silently dropped (feature-overview.md ## Non-Functional Notes; ASMP-26)
- Scale: N/A — one optional field on Balance Payment with no independent growth (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Tip added to the balance charge amount as one charge, or as a separately itemized line, on any Payments & Billing option (Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments, Square); the choice affects fee reporting on the money list (FEAT-28.SPEC-005).

**Risks & Unknowns:** Product description mentions tipping "at deposit or at balance payment" but FEAT-23.SPEC-001 places it in the balance flow only; if tipping at deposit is later wanted it touches FEAT-07's charge amount and lock rules.

**Spike Recommendation:** None

### FEAT-24 — Client List Search & Filter

**Verdict:** Straightforward — partial name/phone matching plus recency and upcoming-booking filters over 100–500 clients per Pro (FEAT-24.SPEC-001 ## States, FEAT-24.SPEC-002; ASMP-22); the profile classifies Search as "Simple filter" (technical-profile.md Section 3).

**Required Capabilities:**
- Instant narrowing as the Pro types, partial phone and name match (FEAT-24.SPEC-001, FEAT-24.SPEC-002)
- Recency (`client-recency-filter-window-days`) and upcoming-booking derivation from booking history (FEAT-24.SPEC-002)
- Private note never surfaced in results; Support sees masked results (feature-overview.md Data sensitivity; XBR-24)
- Concurrency: N/A — read-only feature; underlying record edits follow FEAT-13's rules
- Offline/degraded: loaded client list readable and filterable offline (technical-profile.md Section 3, Offline: FEAT-24.SPEC-001; ASMP-27)
- Scale: 100–500 clients per Pro with multi-year booking history; instant results (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** PostgreSQL built-ins (ILIKE, pg_trgm, full-text search) (Search) need no extra service at this volume; alternatively the whole per-Pro list can be loaded and filtered client-side with TanStack Query cache (State Management), which also serves offline filtering. Typesense, Meilisearch or Algolia (Search) add typo tolerance at the cost of syncing personal data to another service (Algolia needs secured API keys for per-Pro isolation).

**Risks & Unknowns:** None identified — volume is small and the landscape offers database-native options; an external search service would add a personal-data processor (ASMP-23).

**Spike Recommendation:** None

### FEAT-25 — Booking & Revenue Insights

**Verdict:** Straightforward — period summaries from rolling aggregates maintained as bookings and outcomes occur (FEAT-25.SPEC-002, FEAT-25.SPEC-004 ## Trigger Definition) with derivation rules (FEAT-25.SPEC-003); no external service.

**Required Capabilities:**
- Rolling per-period aggregates updated on booking, deposit outcome, no-show and payout events, retried `insights-aggregate-retry-count` times (FEAT-25.SPEC-004)
- Period summary with "saved" figure and most-booked services, "not enough data" threshold (FEAT-25.SPEC-001, FEAT-25.SPEC-003)
- Last successful result retained when a fresh computation fails (FEAT-25.SPEC-002)
- Concurrency: concurrent event updates to the same aggregate row must not lose increments (technical-profile.md Section 3, Collaboration: FEAT-25.SPEC-004)
- Offline/degraded: loaded summary stays visible; aggregate failure retains the last good result (FEAT-25.SPEC-002; technical-profile.md Section 3, Offline: FEAT-25.SPEC-001)
- Scale: aggregates over a multi-year history of 20–40 bookings/week without degrading (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Aggregates computed at read time with Database indexing and query design (Caching & Performance) — viable at this per-Pro size — or maintained incrementally by event handlers in Inngest, Trigger.dev or Postgres-native queue and cron (Background Jobs & Scheduling), or as Postgres materialized views refreshed on schedule. Product-level analytics (PostHog, Mixpanel — Analytics & Product Telemetry) serve the founder's metrics, not the Pro-facing view.

**Risks & Unknowns:** Incremental aggregates drift from source records if an event is missed; a periodic recompute pass would bound drift but is not specified.

**Spike Recommendation:** None

### FEAT-26 — WhatsApp Reminders

**Verdict:** Standard-with-integration — WhatsApp send with delivery status (FEAT-26.SPEC-002 ## Degradation Behavior) and fallback to text or email per consent (FEAT-26.SPEC-003), governed by channel-aware consent (FEAT-26.SPEC-004).

**Required Capabilities:**
- WhatsApp send and inbound status for confirmation, reminder and change-notice content (FEAT-26.SPEC-002)
- Channel preference and explicit channel-scoped consent separate from texting consent (FEAT-26.SPEC-001, FEAT-26.SPEC-004; ASMP-24)
- Automatic fallback on failure or unavailability, with a delivery flag (FEAT-26.SPEC-003)
- Concurrency: duplicate/out-of-order status events resolved by provider event time (FEAT-26.SPEC-002 ## Edge Cases); preference changes follow Messaging Consent resolution (feature-dependency-map.md, Messaging Consent **Contention:**)
- Offline/degraded: every send asynchronous; timeout converts silence to Failed and triggers fallback; preference screen never blocked by provider outage (FEAT-26.SPEC-002 ## Degradation Behavior)
- Scale: N/A at MVP (Later phase); then redistributes FEAT-08 volume across channels (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Twilio (Programmable Messaging and WhatsApp) (Email & Messaging Delivery) is the only landscape option carrying WhatsApp, sharing the account and webhook handling with SMS; Telnyx, Resend and Postmark cover a single channel each (landscape Section 3). Fallback orchestration via the same Background Jobs & Scheduling option as FEAT-08.

**Risks & Unknowns:** Business-initiated WhatsApp messages require pre-approved templates and Meta per-template fees (landscape Twilio row: "Meta template fees from $0.0034"); template approval delays are outside the product's control. The landscape offers a single WhatsApp-capable option, so provider choice for this channel is effectively constrained.

**Spike Recommendation:** None

### FEAT-27 — Pro Profile & Booking Page Settings

**Verdict:** Standard-with-integration — the photo storage capability is an Integration spec (FEAT-27.SPEC-012 ## Capability Category: File storage; ## Degradation Behavior); the rest is settings CRUD plus globally unique link names with 12-month forwarding (FEAT-27.SPEC-007, FEAT-27.SPEC-010) and pause precedence (FEAT-27.SPEC-009, FEAT-27.SPEC-011).

**Required Capabilities:**
- Photo upload within `profile-photo-max-file-size-mb` and format limits, replace and serve; page works without a photo (FEAT-27.SPEC-012; ASMP-35)
- Cross-Pro unique booking-link names with reject-with-refresh and 12-month reservation/forwarding of old names (FEAT-27.SPEC-007, FEAT-27.SPEC-010; XBR-27)
- Timezone/currency settings with currency locked at first deposit (FEAT-27.SPEC-003, FEAT-27.SPEC-008; XBR-25)
- Pro pause with end date and automatic resume; system-imposed pause precedence (FEAT-27.SPEC-004, FEAT-27.SPEC-009, FEAT-27.SPEC-011; XBR-14)
- Notification preferences; help request with acknowledgment (FEAT-27.SPEC-005, FEAT-27.SPEC-006, FEAT-27.SPEC-013)
- Concurrency: link name claimed by another pro first or currency locked mid-edit — reject-with-refresh; other fields last-write-wins between the Pro's devices (feature-dependency-map.md, Pro Account **Contention:**)
- Offline/degraded: photo service slow/down leaves other fields savable; booking page shows no photo without error (FEAT-27.SPEC-012 ## Degradation Behavior)
- Scale: one Pro Account per pro; changes visible on the public page immediately (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Photo storage via Supabase Storage (image transformations, RLS policies), Cloudflare R2 (zero egress), Amazon S3 (pre-signed uploads), Vercel Blob, or Cloudinary (resize and format conversion) (File & Object Storage). Unique names via a unique index plus a reservation table in any Database-area Postgres option. Scheduled forwarding expiry and auto-resume via Platform cron (Vercel Cron Jobs), Postgres-native queue and cron, Inngest or Trigger.dev, or lazy evaluation at read time (Background Jobs & Scheduling). Immediate public-page reflection interacts with CDN/edge caching of public booking page (Caching & Performance) — cache invalidation on save.

**Risks & Unknowns:** Phone photos often carry EXIF location metadata; stripping it matters because the studio address may be a home address (feature-overview.md Data sensitivity) — not specified upstream. Edge caching of the public page conflicts with "shows the change immediately" unless invalidation is wired.

**Spike Recommendation:** None

### FEAT-28 — Payout Account Connection & Payout Visibility

**Verdict:** Standard-with-integration — hand-off into the processor's own identity and bank verification, action-required resolution, and inbound status/payout/fee reporting (FEAT-28.SPEC-006 ## Data Exchanged, ## Degradation Behavior, ## Edge Cases) are the processor's hosted-onboarding pattern.

**Required Capabilities:**
- Processor-hosted onboarding launch and return; no bank or identity data held (FEAT-28.SPEC-001, FEAT-28.SPEC-006; SC-11)
- Status processing driving the go-live gate and notifications (FEAT-28.SPEC-003, FEAT-28.SPEC-007; XBR-06)
- One payout account per Pro, country/currency match (FEAT-28.SPEC-004; XBR-25)
- Money list of deposits, refunds, processor fees and bank payouts with net per period (FEAT-28.SPEC-002, FEAT-28.SPEC-005; XBR-07)
- Banner clearing without manual refresh (technical-profile.md Section 3, Real-time: FEAT-28.SPEC-002)
- Concurrency: processor-reported status authoritative, last-write-wins by processor event time; stale/out-of-order reports discarded (feature-dependency-map.md, Payout Account **Contention:**; FEAT-28.SPEC-006 ## Edge Cases)
- Offline/degraded: dashboard shows last-known data with Retry; resolution flow shows capability-down message leaving Action Required unchanged (FEAT-28.SPEC-006 ## Degradation Behavior); money list readable offline (ASMP-27)
- Scale: any item in the last 90 days findable within a few seconds over multi-year history (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe (Connect plus Billing) hosted identity/bank onboarding and account/payout webhooks; Adyen for Platforms marketplace onboarding; PayPal Complete Payments seller onboarding with market-dependent coverage; Square (Payments and Connect) OAuth seller accounts (Payments & Billing). Money list either mirrored locally from webhooks into any Database-area Postgres option or fetched live from the processor's reporting API; live banner via Polling with TanStack Query / SWR or a push option (Real-time & Collaboration).

**Risks & Unknowns:** Expansion to UK, Canada and Australia (BRIEF.md Geography) depends on the processor's connected-account coverage in each country — varying across options (landscape PayPal row). Mirroring payout and fee data locally risks drift from the processor's ledger; fetching live couples money-list responsiveness to processor latency.

**Spike Recommendation:** None

### FEAT-29 — Pro Sign-In & Account Lifecycle

**Verdict:** Standard-with-integration — one-time codes delivered by SMS and email (FEAT-29.SPEC-014 ## Channels; FEAT-08.SPEC-012/013), device/session management with new-device alerts (FEAT-29.SPEC-006, FEAT-29.SPEC-015), data export generation (FEAT-29.SPEC-007) and closure orchestration with a 30-day cooling-off and deletion (FEAT-29.SPEC-008, FEAT-29.SPEC-013) combine identity and messaging capabilities with scheduled jobs.

**Required Capabilities:**
- Email or mobile OTP sign-in with `sign-in-code-expiry-minutes`, lockout, anti-enumeration (FEAT-29.SPEC-001, FEAT-29.SPEC-011; XBR-29; ASMP-30)
- Signed-in device list, 30-day inactivity expiry, sign-out everywhere, new-device alerts (FEAT-29.SPEC-006, FEAT-29.SPEC-015)
- Recovery via remaining contact; dual-confirmation contact change (FEAT-29.SPEC-002, FEAT-29.SPEC-010, FEAT-29.SPEC-012)
- Spreadsheet-friendly export of clients, bookings and deposits with progress (FEAT-29.SPEC-004, FEAT-29.SPEC-007)
- Closure: bulk cancel with refunds, subscription cancel, page down, cooling-off, deletion keeping de-identified financial records; reopening (FEAT-29.SPEC-008, FEAT-29.SPEC-009; XBR-20)
- Concurrency: sessions on two devices; closure sequencing against in-flight bookings and refunds (feature-dependency-map.md, Pro Account **Contention:**; FEAT-29.SPEC-013)
- Offline/degraded: signed-in Pro keeps read-only access to last loaded schedule; code sending shows in-place indicator; code delivery failures follow FEAT-08.SPEC-009 (feature-overview.md ## Non-Functional Notes)
- Scale: export covers multi-year history (100–500 clients, 20–40 bookings/week) and must stay reliable (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Supabase Auth (email/phone OTP, sessions, RLS integration), Clerk (OTP, session and device management, SMS OTP pricing), WorkOS AuthKit (magic auth), Auth0 (passwordless OTP) or Better Auth / Auth.js (full control of device-alert logic, own session tables) (Authentication & Identity); managed options differ in whether new-device alerts, lockout and anti-enumeration rules match the spec out of the box. Export as a job in Inngest, Trigger.dev (long-running tasks), BullMQ with Redis or Postgres-native queue and cron (Background Jobs & Scheduling), stored in Supabase Storage, Cloudflare R2, Amazon S3 or Vercel Blob (File & Object Storage). Code delivery via Twilio/Telnyx and Resend/Postmark/Amazon SES with SNS (Email & Messaging Delivery).

**Risks & Unknowns:** Managed auth products may not natively support all spec'd rules (dual-confirmation contact change, 30-day device inactivity, new-device alert on existing contacts), pushing custom code around them. Deletion after cooling-off must also purge data held by third parties (auth vendor user store, messaging logs, file storage). SMS OTP is exposed to toll-fraud abuse.

**Spike Recommendation:** None

### FEAT-30 — Pro Booking Management

**Verdict:** Standard-with-integration — Pro cancel/reschedule/goodwill/bulk commits (FEAT-30.SPEC-007..010 ## Processing Logic) call processor refunds with per-booking independent retry (FEAT-30.SPEC-011 ## Degradation Behavior, ## Edge Cases) and deliver deposit requests by text, email or on-screen scan code (FEAT-30.SPEC-013); the contention rule is explicit (feature-dependency-map.md, Booking **Contention:**).

**Required Capabilities:**
- Pro cancel with full refund; Pro reschedule inside notice/horizon exception with deposit carried over and fresh manage link (FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-006; XBR-03, XBR-18)
- Goodwill refund until completion (FEAT-30.SPEC-003, FEAT-30.SPEC-009; XBR-12)
- Bulk cancellation with per-booking outcome reporting (FEAT-30.SPEC-005, FEAT-30.SPEC-008)
- Book-client-in with a deposit-request hold up to 24h or 2h before appointment (FEAT-30.SPEC-004, FEAT-30.SPEC-010; FEAT-03.SPEC-007)
- Deposit request delivered as link or on-screen scan code; delivery stops on expiry (FEAT-30.SPEC-013)
- Concurrency: High — Pro actions race client self-service and automations; first committed wins; bulk siblings refunded independently (feature-dependency-map.md, Booking and Deposit Transaction **Contention:**; FEAT-30.SPEC-011 ## Edge Cases)
- Offline/degraded: actions require connection; a refund that cannot complete shows "Cancelled — refund in progress", never a failure, and never reverses the committed cancellation (FEAT-30.SPEC-011 ## Degradation Behavior); loaded booking-in screen readable (technical-profile.md Section 3, Offline: FEAT-30.SPEC-004)
- Scale: cancel/reschedule in under 30 seconds from the dashboard; a bulk action can cover a full day's bookings (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Refunds via Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing) with idempotency per Deposit Transaction. Bulk fan-out with per-item retry via Inngest step functions, Trigger.dev, Postgres-native queue and cron or BullMQ with Redis (Background Jobs & Scheduling). Deposit-request links as processor-hosted payment links or the FEAT-07 payment page.

**Risks & Unknowns:** The on-screen scan code (FEAT-30.SPEC-013) needs QR-code generation, for which the landscape names no option — recorded as a gap in Section 5. Bulk cancellation on a sick day concentrates refunds against one connected account, raising the insufficient-balance exposure noted for FEAT-09.

**Spike Recommendation:** None

## 3. Cross-Feature Technical Themes

| Theme / Shared Subsystem | Features Involved | Evidence That Makes It Shared |
|--------------------------|-------------------|-------------------------------|
| Slot reservation and booking-contention core (holds, exclusion, first-committed-wins) | FEAT-03, FEAT-05, FEAT-07, FEAT-10, FEAT-17, FEAT-20, FEAT-21, FEAT-30 | FEAT-03.SPEC-002/SPEC-005/SPEC-007 holds and contention; FEAT-05.SPEC-006 checkout hold; FEAT-17 block vs. hold (Time Block **Contention:**); FEAT-20.SPEC-004 priority window; FEAT-21 occurrence reservations (FEAT-03.SPEC-001 ## Edge Cases); FEAT-30.SPEC-010 deposit-request hold; XBR-01, XBR-02 |
| Background job execution, timers and retries | FEAT-02, FEAT-03, FEAT-04, FEAT-08, FEAT-09, FEAT-12, FEAT-17, FEAT-18, FEAT-20, FEAT-21, FEAT-25, FEAT-27, FEAT-29, FEAT-30 | 31 scheduled/timed Automation specs (technical-profile.md Section 3, Background processing), e.g. FEAT-03.SPEC-003 hold expiry, FEAT-08.SPEC-007 reminders, FEAT-09.SPEC-006 refund retry, FEAT-12.SPEC-004 sweep, FEAT-18.SPEC-003 renewals, FEAT-29.SPEC-008 cooling-off |
| Outbound client/Pro messaging with consent gating, timing window and retry-then-fallback | FEAT-06, FEAT-08, FEAT-10, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | All deliver through FEAT-08.SPEC-012/SPEC-013 (feature-dependency-map.md ## External Touchpoints, text and email rows); consent rule XBR-15, timing XBR-16, fallback XBR-17 |
| Payment-processor integration: charges, refunds, connected accounts, disputes, subscriptions, idempotent webhooks | FEAT-07, FEAT-09, FEAT-16, FEAT-18, FEAT-22, FEAT-23, FEAT-28, FEAT-30 | Seven Payment-processing Integration specs (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011), all specifying duplicate and out-of-order event handling in ## Edge Cases; ASMP-31 |
| Refund execution with indefinite idempotent retry and "in progress" surfacing | FEAT-09, FEAT-22, FEAT-23, FEAT-30, FEAT-12, FEAT-29 | FEAT-09.SPEC-006, FEAT-22.SPEC-005, FEAT-30.SPEC-011 share `refund-retry-interval-hours` and identical Degradation Behavior wording; FEAT-12.SPEC-005 attention flag; FEAT-29.SPEC-008 bulk refunds on closure; XBR-10 |
| Inbound webhook ingestion with event-time ordering and deduplication | FEAT-04, FEAT-07, FEAT-08, FEAT-14, FEAT-16, FEAT-18, FEAT-22, FEAT-26, FEAT-28 | Every Integration spec's ## Edge Cases lists "same event delivered twice" and "events arrive out of order … by its own event time" (e.g., FEAT-04.SPEC-003, FEAT-08.SPEC-012, FEAT-28.SPEC-006); landscape Section 3 notes public webhook endpoints are needed |
| Booking commit side-effect fan-out (deposit outcome, calendar mirror, activity event, freed slot, notices) | FEAT-05, FEAT-07, FEAT-10, FEAT-11, FEAT-16, FEAT-20, FEAT-30, FEAT-04 | FEAT-10.SPEC-004 and FEAT-30.SPEC-007/SPEC-008 coordinate the same side effects; FEAT-04.SPEC-005 mirrors every create/reschedule/cancel (XBR-13); FEAT-16.SPEC-002 records every event (XBR-21); XBR-28 freed-slot hand-off |
| Live-updating views (slot list, notices, banners) | FEAT-03, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-22, FEAT-28 | technical-profile.md Section 3, Real-time: FEAT-03.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002; ASMP-21 |
| Read-only offline cache of last-loaded Pro data | FEAT-12, FEAT-13, FEAT-18, FEAT-24, FEAT-28, FEAT-29 | ASMP-27 schedule, client list and money list readable offline; feature-overview.md ## Non-Functional Notes of FEAT-12, FEAT-13, FEAT-18, FEAT-29; technical-profile.md Section 3, Offline |
| Per-Pro data isolation and role-scoped access (Pro, Client, read-only Support with masking) | FEAT-06, FEAT-12, FEAT-13, FEAT-16, FEAT-19, FEAT-24, FEAT-25, FEAT-27, FEAT-28, FEAT-29 | ASMP-23 privacy posture; XBR-24 support restrictions; FEAT-06.SPEC-008 isolation rule; FEAT-12.SPEC-008, FEAT-19.SPEC-004 authorization rules |
| Deletion and de-identified retention across stores | FEAT-13, FEAT-16, FEAT-29, FEAT-14 | XBR-19 client deletion and XBR-20 account closure; FEAT-13.SPEC-004 cascade; FEAT-16 retention (SC-22); FEAT-29.SPEC-013 retention rules |
| Timezone- and currency-aware computation | FEAT-02, FEAT-03, FEAT-07, FEAT-08, FEAT-17, FEAT-21, FEAT-22, FEAT-27 | XBR-25; ASMP-25; FEAT-02.SPEC-005, FEAT-03.SPEC-004 Pro-timezone labeling, FEAT-08.SPEC-007 local daytime window, FEAT-27.SPEC-003/SPEC-008 |
| Generated downloadable files | FEAT-16, FEAT-29 | FEAT-16.SPEC-004 dispute summary download; FEAT-29.SPEC-007 spreadsheet-friendly export; landscape File & Object Storage activation |

## 4. Key Technical Risks

| Risk | Features Affected | Driving Evidence | Possible Mitigation Directions |
|------|-------------------|------------------|--------------------------------|
| Double-booking under concurrent holds, blocks, deposit requests or waitlist claims | FEAT-03, FEAT-05, FEAT-17, FEAT-20, FEAT-21, FEAT-30 | FEAT-03.SPEC-005 ## Edge Cases; XBR-01; ASMP-26 ("never silently double-book"); Booking **Contention:** High | Enforce non-overlap in the database (exclusion constraints or serializable transactions) rather than application checks; the FEAT-03 spike measures the chosen approach under load |
| Apple/iCloud busy time not detected within "a couple of minutes", letting a client book over a personal commitment | FEAT-04, FEAT-03 | FEAT-04.SPEC-003 ## Inbound Events; landscape Calendar Sync area (CalDAV for iCloud, no push noted); ASMP-33 | Run the FEAT-04 spike; directions include frequent CalDAV polling, a unified provider (Nylas, Cronofy), or accepting a longer Apple latency with the Pro-visible confidence banner |
| One-second slot refresh breaks under burst traffic or serverless cold starts | FEAT-03, FEAT-05, FEAT-10 | ASMP-21; FEAT-03 feature-overview.md ## Non-Functional Notes; landscape Real-time & Collaboration polling row ("load scales with viewers") | Push transport options (Supabase Realtime, Ably, Pusher Channels) or warm/long-running hosting (Render, Fly.io); measured in the FEAT-03 spike |
| Refunds cannot complete because the Pro's connected balance was already paid out | FEAT-09, FEAT-22, FEAT-30, FEAT-29 | FEAT-09.SPEC-005 and FEAT-30.SPEC-011 ## Degradation Behavior (Rejects column); XBR-10 | Processor-specific negative-balance/debit behavior as a selection criterion in Payments & Billing; payout-delay settings; monitoring on long-running "in progress" refunds |
| Lost or out-of-order webhooks create wrong payment, consent or calendar state | FEAT-04, FEAT-07, FEAT-14, FEAT-16, FEAT-18, FEAT-22, FEAT-28 | Duplicate/out-of-order cases in every Integration spec's ## Edge Cases; landscape Section 3 idempotent-handler note | Event-time comparison and idempotency keys stored per event; periodic reconciliation against provider APIs; tracing via Observability & Operations options |
| US texting blocked or filtered until A2P 10DLC registration completes; STOP mapping ambiguous on shared numbers | FEAT-08, FEAT-14, FEAT-06, FEAT-29 | landscape Email & Messaging Delivery (10DLC onboarding); FEAT-14.SPEC-004; ASMP-24 | Start registration early; decide shared vs. per-Pro sending numbers; email fallback path (XBR-17) already covers interim delivery |
| Payment and card entry misbehave inside the Instagram in-app browser | FEAT-05, FEAT-07, FEAT-22 | BRIEF.md Devices & platforms; FEAT-05 feature-overview.md ## Non-Functional Notes; ASMP-28 | Device testing of the candidate processor's hosted/embedded elements and 3-D Secure in Instagram's browser before selection; hosted-page fallback |
| Personal data persists in third-party stores after deletion | FEAT-13, FEAT-29, FEAT-08, FEAT-16 | XBR-19, XBR-20; ASMP-23; landscape Observability & Operations and Analytics & Product Telemetry vendors | Minimize personal data sent to vendors; provider data-deletion APIs in the closure/deletion orchestration; scrubbing in error-tracking SDKs |
| Correctness regressions across 30 features with dense cross-feature rules | All | ASMP-26; 29 XBRs and 320 touchpoint rows (technical-profile.md Section 2) | Automated tests on booking, payment and refund logic in CI/CD & Delivery options; concurrency tests derived from the Contention lines |

## 5. Open Questions for the Build Team

| # | Question | Why It Matters | What Would Resolve It |
|---|----------|----------------|------------------------|
| 1 | Can Apple/iCloud busy time be detected within "a couple of minutes", and what does a Pro have to do to authorize iCloud access? | Determines whether FEAT-04 leaves Research-spike status and which Calendar Sync option fits (FEAT-04.SPEC-001, FEAT-04.SPEC-003) | The FEAT-04 spike outcome; if not achievable, a product decision on an accepted Apple latency |
| 2 | Does database-level exclusion deliver exactly one winner under concurrent holds, and at what viewer count does one-second polling breach ASMP-21? | Confirms the FEAT-03 Hard path and whether a push transport is needed at launch | The FEAT-03 spike (load test on a Postgres candidate and the candidate hosting) |
| 3 | Which payment processor's connected-account model, negative-balance/refund behavior and country coverage fit the zero-platform-fee rule and UK/CA/AU expansion? | Affects FEAT-07, FEAT-09, FEAT-22, FEAT-28, FEAT-30 refund feasibility and XBR-07 economics | Architect's Payments & Billing selection informed by processor documentation on connected-account refunds and supported countries |
| 4 | Shared sending number or per-Pro numbers for SMS, and how are inbound STOP replies mapped to a client–Pro pair? | FEAT-14.SPEC-004 revocation correctness and FEAT-08 10DLC registration scope | A product/build decision plus confirmation of the chosen provider's opt-out behavior |
| 5 | What file format do the dispute summary (FEAT-16.SPEC-004) and data export (FEAT-29.SPEC-007, "spreadsheet-friendly") take, and which generation library produces them? | The landscape names storage options but no document/CSV generation library — a landscape gap | A build-team choice of format and library; CSV likely needs none, a formatted summary may |
| 6 | How are the .ics add-to-calendar link (FEAT-08.SPEC-001) and the on-screen deposit-request scan code (FEAT-30.SPEC-013) generated? | No landscape option covers iCalendar or QR-code generation — a landscape gap affecting FEAT-08 and FEAT-30 | A build-team choice of small libraries or hand-written generation |
| 7 | Does the processor run subscription renewals (with the product following its events), or does the product schedule renewals itself? | Avoids two disagreeing grace clocks in FEAT-18.SPEC-003 / FEAT-18.SPEC-006 | Architect decision aligned with the Payments & Billing selection |
| 8 | What values are chosen for the 33 decide-before-build platform parameters, notably `checkout-hold-timeout-minutes`, `refund-retry-interval-hours` and the access-link rate limit? | Hold duration shapes FEAT-03 contention exposure; retry cadence shapes FEAT-09/FEAT-22/FEAT-30 job load; rate limit shapes FEAT-06 abuse exposure | Product-owner decisions recorded in platform-parameters.md |
| 9 | Is tipping at deposit time in scope, or only at balance payment? | FEAT-23 description mentions both; FEAT-23.SPEC-001 covers balance only; deposit tipping would touch FEAT-07 amount-lock rules | A product decision |
| 10 | Must EXIF metadata (including location) be stripped from uploaded profile photos? | Studio address may be a home address (FEAT-27 feature-overview.md Data sensitivity); affects FEAT-27.SPEC-012 processing | A product/privacy decision; File & Object Storage options differ in built-in image processing (Supabase Storage, Cloudinary) |

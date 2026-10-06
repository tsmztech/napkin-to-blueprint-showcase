# Chairtime — Build Prompt Sequence

Paste ONE prompt per message, in order. Prompt 0 goes into Lovable's Plan mode; review the plan before building. Prompts 1–30 are build prompts, one feature each, and each is self-contained. Never paste several prompts at once — these tools degrade on bulk input. KNOWLEDGE.md (already loaded as project knowledge) carries the standing rules, and the full specifications live under docs/blueprint/.

Order notes — the dependency graph contains cycles, so the sequence below makes these forced breaks:

- FEAT-03 depends on FEAT-17, which the sequence places later (sequenced as part of breaking a dependency cycle) — when building FEAT-03 (Real-Time Slot Availability Engine), stub the FEAT-17-facing interface and complete it in FEAT-17's prompt (Manual Time Blocking).
- FEAT-03 ↔ FEAT-21 are mutually dependent (each lists the other in the dependency map); the sequence places FEAT-03 first — when building FEAT-03 (Real-Time Slot Availability Engine), stub the FEAT-21-facing interface and complete it in FEAT-21's prompt (Recurring/Standing Appointments).
- FEAT-05 ↔ FEAT-06 are mutually dependent (each lists the other in the dependency map); the sequence places FEAT-05 first — when building FEAT-05 (Public Booking Page & Booking Flow), stub the FEAT-06-facing interface and complete it in FEAT-06's prompt (Client Booking Identity).
- FEAT-05 ↔ FEAT-07 are mutually dependent (each lists the other in the dependency map); the sequence places FEAT-05 first — when building FEAT-05 (Public Booking Page & Booking Flow), stub the FEAT-07-facing interface and complete it in FEAT-07's prompt (Deposit Payment at Booking).

---

## Prompt 0 — Plan-mode seed

I am building Chairtime: a mobile-first booking page for one solo beauty or wellness professional (barber, nail tech, lash and brow artist, massage therapist, tattoo artist). A client opens the pro's Instagram bio link, picks a service and a genuinely free time, pays a card deposit and gets automatic reminders; a no-show keeps the deposit under the pro's own cancellation policy. The standing rules and scope exclusions are in the project knowledge; the complete blueprint lives under docs/blueprint/.

The build runs as 30 features in this fixed dependency order:

1. FEAT-01 — Service & Pricing Management
2. FEAT-02 — Availability & Working Hours Setup
3. FEAT-04 — Two-Way Calendar Sync
4. FEAT-09 — Cancellation & No-Show Policy Engine
5. FEAT-27 — Pro Profile & Booking Page Settings
6. FEAT-28 — Payout Account Connection & Payout Visibility
7. FEAT-29 — Pro Sign-In & Account Lifecycle
8. FEAT-15 — Pro Onboarding & Setup Wizard
9. FEAT-18 — Pro Subscription Billing & Account Management
10. FEAT-03 — Real-Time Slot Availability Engine
11. FEAT-05 — Public Booking Page & Booking Flow
12. FEAT-06 — Client Booking Identity
13. FEAT-07 — Deposit Payment at Booking
14. FEAT-10 — Client-Initiated Cancel/Reschedule
15. FEAT-11 — No-Show Marking & Deposit Forfeiture
16. FEAT-13 — Client Record Management
17. FEAT-14 — Messaging Consent Management
18. FEAT-08 — Automated Booking Messaging
19. FEAT-12 — Pro Daily Schedule Dashboard
20. FEAT-16 — Booking & Payment Activity Record
21. FEAT-19 — Platform Support Read-Only Access
22. FEAT-20 — Waitlist for Cancelled Slots
23. FEAT-21 — Recurring/Standing Appointments
24. FEAT-22 — In-App Balance Payment
25. FEAT-23 — Tipping at Checkout
26. FEAT-24 — Client List Search & Filter
27. FEAT-25 — Booking & Revenue Insights
28. FEAT-26 — WhatsApp Reminders
29. FEAT-30 — Pro Booking Management
30. FEAT-17 — Manual Time Blocking

Architecture (binding): Next.js 15 (App Router, React 19, TypeScript) with server actions and route handlers, Supabase Postgres with Drizzle, Tailwind CSS v4 with shadcn/ui, Stripe Connect for deposits and Stripe Billing for subscriptions, Twilio and Postmark for messaging, Nylas for calendars, Inngest for jobs. Nothing on the DO-NOT-BUILD list in the project knowledge may be built.

Generate the plan for this build: confirm the feature order above, propose the project scaffold, and tell me what you need before feature 1. Do not write code yet.

---

## Prompt 1 — Service & Pricing Management (FEAT-01)

Build feature FEAT-01 — Service & Pricing Management (Core). The Pro defines the services they offer — name, price, duration and a per-service deposit rule (fixed amount or percentage) — and controls which services are bookable, including add, edit, archive and reorder. Edits and archiving apply to future bookings only.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-01-service-pricing-management/):

- **FEAT-01.SPEC-001 — Service List** (screen): The Pro views their full service catalog, reorders how services appear on the public booking page, switches between Active and Archived views, and reactivates an archived service.
- **FEAT-01.SPEC-002 — Add Service** (screen): The Pro creates a new bookable service by entering its name, price, duration, and deposit rule, with an inline preview of how it will appear on the client-facing booking page.
- **FEAT-01.SPEC-003 — Edit Service** (screen): The Pro updates an existing service's fields or archives it; Platform Operator (Support) views the same service details read-only for troubleshooting.
- **FEAT-01.SPEC-004 — Service Field & Deposit Rule Validation** (logic-rule): Defines every field-level validation rule, cross-field deposit-rule rule, authorization rule, and default/derivation for the Service entity, shared by the Add and Edit screens.
- **FEAT-01.SPEC-005 — Price & Deposit Lock at Booking Time** (logic-rule): Governs that editing or archiving a service never changes the price, duration, or deposit already agreed on a confirmed booking -- this feature's elaboration of XBR-04.
- **FEAT-01.SPEC-006 — Archive Impact Check** (automation): On an archive request, checks for upcoming bookings referencing the service and surfaces an impact warning before the Pro confirms the archive.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-024) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-01.SPEC-001 — 10 acceptance criteria (FEAT-01.SPEC-001-AC-01…FEAT-01.SPEC-001-AC-10) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-001-service-list.md
- FEAT-01.SPEC-002 — 10 acceptance criteria (FEAT-01.SPEC-002-AC-01…FEAT-01.SPEC-002-AC-10) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-002-add-service.md
- FEAT-01.SPEC-003 — 11 acceptance criteria (FEAT-01.SPEC-003-AC-01…FEAT-01.SPEC-003-AC-11) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-003-edit-service.md
- FEAT-01.SPEC-004 — 17 acceptance criteria (FEAT-01.SPEC-004-AC-01…FEAT-01.SPEC-004-AC-17) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-004-service-field-deposit-rule-validation.md
- FEAT-01.SPEC-005 — 14 acceptance criteria (FEAT-01.SPEC-005-AC-01…FEAT-01.SPEC-005-AC-14) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-005-price-deposit-lock-at-booking-time.md
- FEAT-01.SPEC-006 — 10 acceptance criteria (FEAT-01.SPEC-006-AC-01…FEAT-01.SPEC-006-AC-10) — docs/blueprint/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-006-archive-impact-check.md

All 72 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 2 — Availability & Working Hours Setup (FEAT-02)

Build feature FEAT-02 — Availability & Working Hours Setup (Core). The Pro sets recurring working hours and buffer time between clients, forming the base schedule the availability engine works from, with per-service buffer overrides and versioned rules.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/):

- **FEAT-02.SPEC-001 — Working Hours, Buffer, Notice & Horizon Setup** (screen): Talia (the Pro) sets her recurring weekly working windows, default buffer time between bookings, minimum booking notice, and booking horizon — the base schedule the availability engine computes from.
- **FEAT-02.SPEC-002 — Per-Service Buffer Override** (screen): Talia (the Pro) sets a buffer-time override for an individual service that genuinely needs more or less gap than her default buffer.
- **FEAT-02.SPEC-003 — Availability Rule Versioning** (automation): System saves every passing Availability Rule edit as a new dated version rather than overwriting the prior one, so past bookings keep the rule that was live when they were made.
- **FEAT-02.SPEC-004 — Confirmed Booking Conflict Flagging** (automation): System checks every existing confirmed booking against a newly saved Availability Rule version and flags any that now fall outside working hours, without ever cancelling them.
- **FEAT-02.SPEC-005 — Availability Setup Validation & Limits** (logic-rule): Defines every validation and boundary rule governing the Availability Rule and its per-service buffer override, plus authorization rules for every action on both, shared by FEAT-02.SPEC-001 and FEAT-02.SPEC-002 so neither screen duplicates the rules.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-016) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-02.SPEC-001 — 21 acceptance criteria (FEAT-02.SPEC-001-AC-01…FEAT-02.SPEC-001-AC-21) — docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-001-working-hours-buffer-notice-horizon-setup.md
- FEAT-02.SPEC-002 — 15 acceptance criteria (FEAT-02.SPEC-002-AC-01…FEAT-02.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-002-per-service-buffer-override.md
- FEAT-02.SPEC-003 — 11 acceptance criteria (FEAT-02.SPEC-003-AC-01…FEAT-02.SPEC-003-AC-11) — docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-003-availability-rule-versioning.md
- FEAT-02.SPEC-004 — 11 acceptance criteria (FEAT-02.SPEC-004-AC-01…FEAT-02.SPEC-004-AC-11) — docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-004-confirmed-booking-conflict-flagging.md
- FEAT-02.SPEC-005 — 24 acceptance criteria (FEAT-02.SPEC-005-AC-01…FEAT-02.SPEC-005-AC-24) — docs/blueprint/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-005-availability-setup-validation-limits.md

All 82 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 3 — Two-Way Calendar Sync (FEAT-04)

Build feature FEAT-04 — Two-Way Calendar Sync (Core). The Pro connects a personal Google or Apple calendar: busy time there blocks Chairtime availability and confirmed bookings appear on that calendar automatically, with connection status and sync-health handling.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/):

- **FEAT-04.SPEC-001 — Calendar Connection Setup** (screen): The Pro chooses Google or Apple as their personal calendar kind and authorizes Chairtime to connect to it, kicking off the account-linking handshake.
- **FEAT-04.SPEC-002 — Calendar Connection Status & Management** (screen): The Pro views the health of each connected calendar, reconnects a lapsed connection, or disconnects a calendar at any time; this is the default entry point for the calendar feature area.
- **FEAT-04.SPEC-003 — Calendar Provider Sync** (integration): The product connects to a Pro's personal Google or Apple calendar through a calendar-sync capability, performing the account-linking handshake, pulling busy/free time, and writing, moving, or removing Chairtime bookings on that calendar.
- **FEAT-04.SPEC-004 — Busy-Time Availability Feed** (automation): Turns the busy/free periods synced from a Pro's connected personal calendar into the blocked-availability signal the slot engine consumes, so external commitments block Chairtime availability.
- **FEAT-04.SPEC-005 — Booking-to-Calendar Sync** (automation): On any Chairtime booking create, reschedule, or cancel, writes, moves, or removes the matching event on the Pro's connected personal calendar, so the personal calendar never shows a stale or ghost appointment.
- **FEAT-04.SPEC-006 — Sync Health Monitor & Reconciliation** (automation): Watches each connection's validity, degrades the Pro-visible availability confidence on failure, and reconciles drift once a lapsed connection is restored.
- **FEAT-04.SPEC-007 — Calendar Reconnection Alert** (notification): Tells the Pro, via a dashboard banner, that a connected calendar needs reconnecting, so a lapsed connection is never a silent gap.
- **FEAT-04.SPEC-008 — Calendar Connection Rules** (logic-rule): Enforces the one-connection-per-kind limit and the contention/precedence rules governing connect, disconnect, and in-flight sync for the Calendar Connection entity.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-017, ADR-022) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-04.SPEC-001 — 11 acceptance criteria (FEAT-04.SPEC-001-AC-01…FEAT-04.SPEC-001-AC-11) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-001-calendar-connection-setup.md
- FEAT-04.SPEC-002 — 13 acceptance criteria (FEAT-04.SPEC-002-AC-01…FEAT-04.SPEC-002-AC-13) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-002-calendar-connection-status-management.md
- FEAT-04.SPEC-003 — 15 acceptance criteria (FEAT-04.SPEC-003-AC-01…FEAT-04.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-003-calendar-provider-sync.md
- FEAT-04.SPEC-004 — 10 acceptance criteria (FEAT-04.SPEC-004-AC-01…FEAT-04.SPEC-004-AC-10) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-004-busy-time-availability-feed.md
- FEAT-04.SPEC-005 — 13 acceptance criteria (FEAT-04.SPEC-005-AC-01…FEAT-04.SPEC-005-AC-13) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md
- FEAT-04.SPEC-006 — 12 acceptance criteria (FEAT-04.SPEC-006-AC-01…FEAT-04.SPEC-006-AC-12) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-006-sync-health-monitor-reconciliation.md
- FEAT-04.SPEC-007 — 9 acceptance criteria (FEAT-04.SPEC-007-AC-01…FEAT-04.SPEC-007-AC-09) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-007-calendar-reconnection-alert.md
- FEAT-04.SPEC-008 — 15 acceptance criteria (FEAT-04.SPEC-008-AC-01…FEAT-04.SPEC-008-AC-15) — docs/blueprint/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-008-calendar-connection-rules.md

All 98 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 4 — Cancellation & No-Show Policy Engine (FEAT-09)

Build feature FEAT-09 — Cancellation & No-Show Policy Engine (Core). The Pro defines a cancellation window and what happens to the deposit inside versus outside it; the system enforces the versioned policy automatically on every cancellation and no-show, exactly as the client agreed at booking.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/):

- **FEAT-09.SPEC-001 — Cancellation Policy Setup** (screen): Talia sets or edits her cancellation/reschedule window in whole hours and reviews the resulting plain-language wording before saving, so every client sees exactly what will happen to their deposit.
- **FEAT-09.SPEC-002 — Policy Versioning & Cutoff Rendering** (logic-rule): Governs how an edit to the Cancellation Policy always creates a new, immutable version rather than overwriting the current one, how every Booking stays permanently bound to the version it acknowledged, and how this booking's exact cutoff time and plain-language wording are computed wherever another feature needs to display them.
- **FEAT-09.SPEC-003 — Deposit Outcome Rules** (logic-rule): Defines the complete, binary, symmetric rule set governing what happens to a booking's deposit for every combination of who acts (client or Pro), what they do (cancel, reschedule, no-show), and when they do it relative to the policy's cutoff -- the single source of truth every evaluation and preview in the product reads instead of re-deriving.
- **FEAT-09.SPEC-004 — Cancellation & No-Show Outcome Evaluation** (automation): On every cancellation, reschedule, or no-show marking, evaluates the booking's bound policy version against FEAT-09.SPEC-003's rule set and writes the resulting refund-due or forfeiture-due outcome to the Deposit Transaction, instantaneously and with no user-visible loading state.
- **FEAT-09.SPEC-005 — Automatic Deposit Refund** (integration): The product requests a full deposit refund from the payment-processing capability whenever FEAT-09.SPEC-004 determines one is due, drawing on the Pro's connected payout account, and reflects the outcome to both parties without either having to chase it.
- **FEAT-09.SPEC-006 — Refund Idempotency & Retry Rule** (logic-rule): Guarantees a refund completes exactly once per Deposit Transaction, retries automatically and indefinitely when it cannot complete immediately, and keeps the Pro's dashboard flag and the client's "in progress" status consistent with the true state until the refund resolves.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-012) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-09.SPEC-001 — 12 acceptance criteria (FEAT-09.SPEC-001-AC-01…FEAT-09.SPEC-001-AC-12) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-001-cancellation-policy-setup.md
- FEAT-09.SPEC-002 — 15 acceptance criteria (FEAT-09.SPEC-002-AC-01…FEAT-09.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-002-policy-versioning-cutoff-rendering.md
- FEAT-09.SPEC-003 — 16 acceptance criteria (FEAT-09.SPEC-003-AC-01…FEAT-09.SPEC-003-AC-16) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-003-deposit-outcome-rules.md
- FEAT-09.SPEC-004 — 15 acceptance criteria (FEAT-09.SPEC-004-AC-01…FEAT-09.SPEC-004-AC-15) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-004-cancellation-no-show-outcome-evaluation.md
- FEAT-09.SPEC-005 — 16 acceptance criteria (FEAT-09.SPEC-005-AC-01…FEAT-09.SPEC-005-AC-16) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-005-automatic-deposit-refund.md
- FEAT-09.SPEC-006 — 13 acceptance criteria (FEAT-09.SPEC-006-AC-01…FEAT-09.SPEC-006-AC-13) — docs/blueprint/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-006-refund-idempotency-retry-rule.md

All 87 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 5 — Pro Profile & Booking Page Settings (FEAT-27)

Build feature FEAT-27 — Pro Profile & Booking Page Settings (Important). The Pro controls how they appear and how the booking page behaves: display name, photo, intro, booking link name, timezone and currency, pausing new bookings, notification preferences and help requests.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/):

- **FEAT-27.SPEC-001 — Profile & Booking Page Settings** (screen): Talia edits her public profile (display name, photo, intro, general area) and her confirmation-only studio address, and reaches every other settings sub-flow (booking link, timezone/currency, pause, notifications, help) and the booking-page preview from one hub.
- **FEAT-27.SPEC-002 — Booking Link Rename** (screen): Talia views and changes her booking link name, sees the forwarding guarantee on the old name before she confirms, and gets suggestions when a name is already taken.
- **FEAT-27.SPEC-003 — Timezone & Currency Settings** (screen): Talia sets her account timezone and currency, sees a warning before a timezone change is saved, and sees currency locked once it has been.
- **FEAT-27.SPEC-004 — Pause Bookings** (screen): Talia pauses new bookings with an optional message and end date, or resumes them, and sees whether a system-imposed pause is also in effect.
- **FEAT-27.SPEC-005 — Notification Preferences** (screen): Talia chooses which Pro notifications she receives and on which channel(s) -- in-app, text, or email.
- **FEAT-27.SPEC-006 — Help Request** (screen): Talia describes a problem and sends a help request to support from settings.
- **FEAT-27.SPEC-007 — Booking Link Name Validation & Uniqueness Rule** (logic-rule): Defines the format, length, and cross-pro uniqueness rules for booking_link_name, and the reject-with-refresh behavior when another pro claims a candidate name first.
- **FEAT-27.SPEC-008 — Currency Lock Rule** (logic-rule): Determines whether the Pro Account's currency is still editable, locking it permanently the moment the account's first deposit is taken.
- **FEAT-27.SPEC-009 — Pause State Precedence Rule** (logic-rule): Governs how a Pro-chosen pause and a system-imposed (subscription-lapse) pause coexist, including that the Pro's resume toggle cannot clear a system-imposed pause, and that a pause end date cannot be in the past.
- **FEAT-27.SPEC-010 — Booking Link Forwarding & Reservation Expiry** (automation): On a booking-link rename, keeps the old link name forwarding to the new one for at least platform parameter: `booking-link-forward-window-months`, reserving it from reuse by any other pro until that window lapses, and then releases the reservation.
- **FEAT-27.SPEC-011 — Automatic Pause Resume** (automation): Resumes bookings automatically when a Pro-chosen pause reaches its end date, without disturbing a still-active system-imposed pause.
- **FEAT-27.SPEC-012 — Profile Photo Storage Capability** (integration): The product stores, replaces, and serves Talia's profile photo through an external file-storage capability, so the booking page still works correctly even when no photo has been set.
- **FEAT-27.SPEC-013 — Help Request Acknowledgment** (notification): Confirms to Talia that her help request was received, carrying the reference support's read-only lookup (FEAT-19) points to.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-008, ADR-016, ADR-020) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-27.SPEC-001 — 15 acceptance criteria (FEAT-27.SPEC-001-AC-01…FEAT-27.SPEC-001-AC-15) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-001-profile-booking-page-settings.md
- FEAT-27.SPEC-002 — 12 acceptance criteria (FEAT-27.SPEC-002-AC-01…FEAT-27.SPEC-002-AC-12) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-002-booking-link-rename.md
- FEAT-27.SPEC-003 — 12 acceptance criteria (FEAT-27.SPEC-003-AC-01…FEAT-27.SPEC-003-AC-12) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-003-timezone-currency-settings.md
- FEAT-27.SPEC-004 — 13 acceptance criteria (FEAT-27.SPEC-004-AC-01…FEAT-27.SPEC-004-AC-13) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-004-pause-bookings.md
- FEAT-27.SPEC-005 — 11 acceptance criteria (FEAT-27.SPEC-005-AC-01…FEAT-27.SPEC-005-AC-11) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-005-notification-preferences.md
- FEAT-27.SPEC-006 — 10 acceptance criteria (FEAT-27.SPEC-006-AC-01…FEAT-27.SPEC-006-AC-10) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-006-help-request.md
- FEAT-27.SPEC-007 — 12 acceptance criteria (FEAT-27.SPEC-007-AC-01…FEAT-27.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-007-booking-link-name-validation-uniqueness-rule.md
- FEAT-27.SPEC-008 — 9 acceptance criteria (FEAT-27.SPEC-008-AC-01…FEAT-27.SPEC-008-AC-09) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-008-currency-lock-rule.md
- FEAT-27.SPEC-009 — 11 acceptance criteria (FEAT-27.SPEC-009-AC-01…FEAT-27.SPEC-009-AC-11) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-009-pause-state-precedence-rule.md
- FEAT-27.SPEC-010 — 10 acceptance criteria (FEAT-27.SPEC-010-AC-01…FEAT-27.SPEC-010-AC-10) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-010-booking-link-forwarding-reservation-expiry.md
- FEAT-27.SPEC-011 — 9 acceptance criteria (FEAT-27.SPEC-011-AC-01…FEAT-27.SPEC-011-AC-09) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-011-automatic-pause-resume.md
- FEAT-27.SPEC-012 — 11 acceptance criteria (FEAT-27.SPEC-012-AC-01…FEAT-27.SPEC-012-AC-11) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-012-profile-photo-storage-capability.md
- FEAT-27.SPEC-013 — 10 acceptance criteria (FEAT-27.SPEC-013-AC-01…FEAT-27.SPEC-013-AC-10) — docs/blueprint/specifications/FEAT-27-pro-profile-booking-page-settings/FEAT-27.SPEC-013-help-request-acknowledgment.md

All 145 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 6 — Payout Account Connection & Payout Visibility (FEAT-28)

Build feature FEAT-28 — Payout Account Connection & Payout Visibility (Core). The Pro connects a payout account so every client deposit lands directly with them, and sees what came in, what was refunded, the processor's card fee and when money reaches the bank; Chairtime takes nothing.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/):

- **FEAT-28.SPEC-001 — Payout Account Connection** (screen): Talia launches the payment-processing capability's own secure identity and bank verification flow from the "getting paid" step of setup, and sees the outcome of that handoff (pending, active, or a failed handoff) before continuing.
- **FEAT-28.SPEC-002 — Payout Status & Money Dashboard** (screen): Talia's ongoing view of her payout account's status (with an action-required banner and resolution path when needed) and the money list of deposits, refunds, processor fees, and payouts, in all its data states.
- **FEAT-28.SPEC-003 — Payout Account Status Processing** (automation): Creates and updates the Payout Account record from the payment-processing capability's reported status changes, drives the go-live gate signal (XBR-06), and triggers the status notification.
- **FEAT-28.SPEC-004 — Payout Account Eligibility & Constraints** (logic-rule): Governs one-payout-account-per-Pro, country/currency matching to the Pro Account, and the standing zero-Chairtime-fee rule that the money list and go-live gate both depend on.
- **FEAT-28.SPEC-005 — Money List Composition & Net Calculation** (logic-rule): Defines how deposits, refunds, processor fees, and payouts are assembled into the money list and how net amount received per period is derived.
- **FEAT-28.SPEC-006 — Payout Account Connection & Verification** (integration): Handles the outbound handoff to, and inbound status/money data from, the payment-processing capability for account connection, identity and bank verification, action-required resolution, and payout/fee reporting.
- **FEAT-28.SPEC-007 — Payout Status Notification** (notification): Notifies Talia when verification completes and the booking link can go live, and when the payout account needs action, so she never discovers either condition only by happening to open the dashboard.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-022) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-28.SPEC-001 — 18 acceptance criteria (FEAT-28.SPEC-001-AC-01…FEAT-28.SPEC-001-AC-18) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-001-payout-account-connection.md
- FEAT-28.SPEC-002 — 22 acceptance criteria (FEAT-28.SPEC-002-AC-01…FEAT-28.SPEC-002-AC-22) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-002-payout-status-money-dashboard.md
- FEAT-28.SPEC-003 — 14 acceptance criteria (FEAT-28.SPEC-003-AC-01…FEAT-28.SPEC-003-AC-14) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-003-payout-account-status-processing.md
- FEAT-28.SPEC-004 — 19 acceptance criteria (FEAT-28.SPEC-004-AC-01…FEAT-28.SPEC-004-AC-19) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-004-payout-account-eligibility-constraints.md
- FEAT-28.SPEC-005 — 20 acceptance criteria (FEAT-28.SPEC-005-AC-01…FEAT-28.SPEC-005-AC-20) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-005-money-list-composition-net-calculation.md
- FEAT-28.SPEC-006 — 20 acceptance criteria (FEAT-28.SPEC-006-AC-01…FEAT-28.SPEC-006-AC-20) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-006-payout-account-connection-verification.md
- FEAT-28.SPEC-007 — 14 acceptance criteria (FEAT-28.SPEC-007-AC-01…FEAT-28.SPEC-007-AC-14) — docs/blueprint/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-007-payout-status-notification.md

All 127 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 7 — Pro Sign-In & Account Lifecycle (FEAT-29)

Build feature FEAT-29 — Pro Sign-In & Account Lifecycle (Important). The Pro signs in securely from their phone, can recover access, export their data, manage sessions and devices, change contact details and close their account with data deleted afterward.

**Already built (dependencies):** none — this is the starting feature

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/):

- **FEAT-29.SPEC-001 — Sign-In Screen** (screen): Talia enters her sign-in email or mobile number, requests a one-time code, and enters it to sign in -- and it is the screen every unauthenticated visitor to a Pro-only area lands on (XBR-29).
- **FEAT-29.SPEC-002 — Account Recovery Screen** (screen): Talia, having lost access to one of her two sign-in contact methods, regains sign-in through whichever contact method she still controls.
- **FEAT-29.SPEC-003 — Account & Sign-In Settings Screen** (screen): Talia's ongoing hub for her signed-in devices, signing out everywhere, and starting a sign-in-contact change, data export, or account closure; also Support's view-only entry point for account status.
- **FEAT-29.SPEC-004 — Data Export Screen** (screen): Talia requests and downloads a spreadsheet-friendly file of her own clients and booking history.
- **FEAT-29.SPEC-005 — Account Closure & Reopening Screen** (screen): Talia reviews upcoming bookings and confirms closure, or -- during the cooling-off period -- reopens her account with everything intact.
- **FEAT-29.SPEC-006 — Session & Device Management** (automation): Requests and verifies one-time sign-in codes, creates or refreshes a signed-in device on successful verification, expires devices after inactivity, and executes "sign out everywhere."
- **FEAT-29.SPEC-007 — Data Export Generation** (automation): Assembles the requested spreadsheet-friendly file from the Pro's own Client, Booking, and Deposit Transaction records on request.
- **FEAT-29.SPEC-008 — Account Closure Orchestration** (automation): Sequences closure -- cancels the subscription, takes the booking page down, starts the 30-day cooling-off clock, and executes deletion (retaining only legally required de-identified financial records) when the cooling-off period expires unreversed.
- **FEAT-29.SPEC-009 — Account Reopening** (automation): Restores a Closing account to Active when the Pro signs back in during the cooling-off period, leaving the booking page down until the Pro separately resumes it.
- **FEAT-29.SPEC-010 — Contact-Detail Change Processing** (automation): Carries a sign-in email or mobile-number change through code-entry confirmation on both the old and the new contact -- the same code-entry pattern used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- before committing it.
- **FEAT-29.SPEC-011 — Sign-In & Recovery Rules** (logic-rule): Governs one-time-code expiry, the failed-attempt lockout, session duration, and the anti-enumeration rule that a failed sign-in never reveals whether an account exists.
- **FEAT-29.SPEC-012 — Contact-Change Confirmation Rules** (logic-rule): Governs the dual-confirmation requirement for sign-in-contact changes -- each side proving control of its contact by entering a one-time code, identically to the Shared UI Pattern's code-entry step used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- and what a partial or abandoned change leaves in place.
- **FEAT-29.SPEC-013 — Account Closure & Retention Rules** (logic-rule): Governs the 30-day cooling-off period, the closure sequencing required before deletion, the scope of what deletion removes versus retains, and the scope of the data export.
- **FEAT-29.SPEC-014 — Sign-In Code Notification** (notification): Delivers the one-time sign-in code to whichever contact method Talia is signing in with.
- **FEAT-29.SPEC-015 — New-Device Sign-In Alert** (notification): Alerts Talia on her existing contact methods when her account is signed in on a device not seen before, so an unrecognized sign-in is never silent.
- **FEAT-29.SPEC-016 — Contact-Change Confirmation Notification** (notification): Delivers the one-time confirmation code for a sign-in email or mobile-number change to each side -- the same code-entry pattern used to deliver a sign-in code (FEAT-29.SPEC-014) -- and confirms the change to both the old and the new contact once it commits.
- **FEAT-29.SPEC-017 — Account Closure & Deletion Notifications** (notification): Sends the account-closure confirmation when closure is requested and the final notice when data is permanently deleted.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-025, ADR-026, ADR-008) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-29.SPEC-001 — 13 acceptance criteria (FEAT-29.SPEC-001-AC-01…FEAT-29.SPEC-001-AC-13) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-001-sign-in-screen.md
- FEAT-29.SPEC-002 — 10 acceptance criteria (FEAT-29.SPEC-002-AC-01…FEAT-29.SPEC-002-AC-10) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-002-account-recovery-screen.md
- FEAT-29.SPEC-003 — 19 acceptance criteria (FEAT-29.SPEC-003-AC-01…FEAT-29.SPEC-003-AC-19) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-003-account-sign-in-settings-screen.md
- FEAT-29.SPEC-004 — 12 acceptance criteria (FEAT-29.SPEC-004-AC-01…FEAT-29.SPEC-004-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-004-data-export-screen.md
- FEAT-29.SPEC-005 — 15 acceptance criteria (FEAT-29.SPEC-005-AC-01…FEAT-29.SPEC-005-AC-15) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-005-account-closure-reopening-screen.md
- FEAT-29.SPEC-006 — 14 acceptance criteria (FEAT-29.SPEC-006-AC-01…FEAT-29.SPEC-006-AC-14) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-006-session-device-management.md
- FEAT-29.SPEC-007 — 12 acceptance criteria (FEAT-29.SPEC-007-AC-01…FEAT-29.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-007-data-export-generation.md
- FEAT-29.SPEC-008 — 12 acceptance criteria (FEAT-29.SPEC-008-AC-01…FEAT-29.SPEC-008-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-008-account-closure-orchestration.md
- FEAT-29.SPEC-009 — 9 acceptance criteria (FEAT-29.SPEC-009-AC-01…FEAT-29.SPEC-009-AC-09) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-009-account-reopening.md
- FEAT-29.SPEC-010 — 12 acceptance criteria (FEAT-29.SPEC-010-AC-01…FEAT-29.SPEC-010-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-010-contact-detail-change-processing.md
- FEAT-29.SPEC-011 — 12 acceptance criteria (FEAT-29.SPEC-011-AC-01…FEAT-29.SPEC-011-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-011-sign-in-recovery-rules.md
- FEAT-29.SPEC-012 — 11 acceptance criteria (FEAT-29.SPEC-012-AC-01…FEAT-29.SPEC-012-AC-11) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-012-contact-change-confirmation-rules.md
- FEAT-29.SPEC-013 — 12 acceptance criteria (FEAT-29.SPEC-013-AC-01…FEAT-29.SPEC-013-AC-12) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-013-account-closure-retention-rules.md
- FEAT-29.SPEC-014 — 10 acceptance criteria (FEAT-29.SPEC-014-AC-01…FEAT-29.SPEC-014-AC-10) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-014-sign-in-code-notification.md
- FEAT-29.SPEC-015 — 9 acceptance criteria (FEAT-29.SPEC-015-AC-01…FEAT-29.SPEC-015-AC-09) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-015-new-device-sign-in-alert.md
- FEAT-29.SPEC-016 — 11 acceptance criteria (FEAT-29.SPEC-016-AC-01…FEAT-29.SPEC-016-AC-11) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-016-contact-change-confirmation-notification.md
- FEAT-29.SPEC-017 — 11 acceptance criteria (FEAT-29.SPEC-017-AC-01…FEAT-29.SPEC-017-AC-11) — docs/blueprint/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-017-account-closure-deletion-notifications.md

All 204 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 8 — Pro Onboarding & Setup Wizard (FEAT-15)

Build feature FEAT-15 — Pro Onboarding & Setup Wizard (Important). A guided one-time setup wizard takes a brand-new Pro from signup to a live, shareable booking link — services, hours, deposit rule, cancellation policy and calendar — with sensible defaults and resumable progress.

**Already built (dependencies):** FEAT-28 (Payout Account Connection & Payout Visibility), FEAT-29 (Pro Sign-In & Account Lifecycle)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/):

- **FEAT-15.SPEC-001 — Setup Wizard Shell, Step Navigation & Guidance** (screen): The persistent wizard frame that shows Talia her step order and progress, hands her into each step's owning-feature screen in sequence, and surfaces a short plain-language tip for whichever step she is on.
- **FEAT-15.SPEC-002 — Cancellation Policy Default & First-Version Setup Step** (screen): Talia accepts or adjusts a recommended cancellation window and saves it, creating the first Cancellation Policy version for her account.
- **FEAT-15.SPEC-003 — Go-Live Preview & Booking Link Hand-Over** (screen): Once every required setup step is satisfied, this screen reveals Talia's live shareable booking link with a plain hand-over note on what to do with it -- or, if her payout account is still verifying, shows a "finish verifying to start taking bookings" waiting state instead.
- **FEAT-15.SPEC-004 — Setup Progress Tracking & Resume** (automation): Creates the Pro Account's setup-progress record the moment Talia first enters the wizard, updates it as each step (including a skipped calendar step) completes, and computes the exact resume point whenever she returns -- the same progress state Support reads read-only.
- **FEAT-15.SPEC-005 — Go-Live Evaluation & Booking Link Activation** (automation): Re-evaluates readiness against the Go-Live Prerequisite Rule after every relevant step completion or upstream status change, and activates Talia's booking link the moment it is satisfied.
- **FEAT-15.SPEC-006 — Setup Step Order & Optional-Step Rules** (logic-rule): Defines the fixed step sequence, which single step (calendar connection) is optional and resumable from settings later, and how a skipped step is represented in progress.
- **FEAT-15.SPEC-007 — Go-Live Prerequisite Rule (XBR-26 Authority)** (logic-rule): Defines and owns the exact set of steps that must be complete before the booking link can go live, per XBR-26; consumed by FEAT-15.SPEC-005 and referenced by other features that gate on go-live status.
- **FEAT-15.SPEC-008 — Onboarding Welcome Confirmation** (notification): Tells Talia, the moment her booking link goes live, that setup is done and her link is ready to share -- a one-time confirmation, not a recurring nag.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-006, ADR-023) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-15.SPEC-001 — 14 acceptance criteria (FEAT-15.SPEC-001-AC-01…FEAT-15.SPEC-001-AC-14) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-001-setup-wizard-shell-step-navigation-guidance.md
- FEAT-15.SPEC-002 — 12 acceptance criteria (FEAT-15.SPEC-002-AC-01…FEAT-15.SPEC-002-AC-12) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-002-cancellation-policy-default-first-version-setup-step.md
- FEAT-15.SPEC-003 — 13 acceptance criteria (FEAT-15.SPEC-003-AC-01…FEAT-15.SPEC-003-AC-13) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-003-go-live-preview-booking-link-hand-over.md
- FEAT-15.SPEC-004 — 12 acceptance criteria (FEAT-15.SPEC-004-AC-01…FEAT-15.SPEC-004-AC-12) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-004-setup-progress-tracking-resume.md
- FEAT-15.SPEC-005 — 11 acceptance criteria (FEAT-15.SPEC-005-AC-01…FEAT-15.SPEC-005-AC-11) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-005-go-live-evaluation-booking-link-activation.md
- FEAT-15.SPEC-006 — 15 acceptance criteria (FEAT-15.SPEC-006-AC-01…FEAT-15.SPEC-006-AC-15) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-006-setup-step-order-optional-step-rules.md
- FEAT-15.SPEC-007 — 14 acceptance criteria (FEAT-15.SPEC-007-AC-01…FEAT-15.SPEC-007-AC-14) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-007-go-live-prerequisite-rule-xbr-26-authority.md
- FEAT-15.SPEC-008 — 12 acceptance criteria (FEAT-15.SPEC-008-AC-01…FEAT-15.SPEC-008-AC-12) — docs/blueprint/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-008-onboarding-welcome-confirmation.md

All 103 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 9 — Pro Subscription Billing & Account Management (FEAT-18)

Build feature FEAT-18 — Pro Subscription Billing & Account Management (Important). The Pro's flat monthly Chairtime subscription — one price tier, card-based, cancel anytime — with plan status, payment-method updates, renewal and failure handling, and lapse pausing.

**Already built (dependencies):** FEAT-15 (Pro Onboarding & Setup Wizard)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/):

- **FEAT-18.SPEC-001 — Subscribe Screen** (screen): Talia enters her card details during onboarding, sees the single all-inclusive price and the zero-fee statement, and starts her Chairtime subscription.
- **FEAT-18.SPEC-002 — Billing & Subscription Management Screen** (screen): Talia's ongoing view of her subscription's plan status and next billing date, with actions to update her payment method or cancel; Support's read-only entry point into the same status for billing support questions.
- **FEAT-18.SPEC-003 — Subscription Renewal & Payment-Failure Processing** (automation): Runs the monthly renewal charge against the payment-processing capability, branches on success or failure, updates the billing date on success, and starts the 7-day grace-period clock on failure.
- **FEAT-18.SPEC-004 — Subscription-Lapse Account Pause Trigger** (automation): Triggers the Pro Account's system-imposed pause when the grace period expires unresolved, and lifts it the moment billing is restored.
- **FEAT-18.SPEC-005 — Subscription Billing Rules** (logic-rule): Governs the single price tier, the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`), cancellation-at-period-end timing, the 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`), and contention/authority resolution between the Pro and the payment-processing capability for the Subscription entity.
- **FEAT-18.SPEC-006 — Subscription Billing Integration** (integration): Carries outbound subscribe, update-payment-method, and cancel requests to the payment-processing capability, and receives inbound renewal-outcome and dispute events, so the Pro's subscription billing is handled without the product ever holding card data.
- **FEAT-18.SPEC-007 — Subscription Billing Notifications** (notification): Sends Talia the payment-failure grace notice, renewal receipt, cancellation confirmation, and price-change notice, so she is never surprised by her own billing.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-18.SPEC-001 — 11 acceptance criteria (FEAT-18.SPEC-001-AC-01…FEAT-18.SPEC-001-AC-11) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-001-subscribe-screen.md
- FEAT-18.SPEC-002 — 13 acceptance criteria (FEAT-18.SPEC-002-AC-01…FEAT-18.SPEC-002-AC-13) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-002-billing-subscription-management-screen.md
- FEAT-18.SPEC-003 — 11 acceptance criteria (FEAT-18.SPEC-003-AC-01…FEAT-18.SPEC-003-AC-11) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-003-subscription-renewal-payment-failure-processing.md
- FEAT-18.SPEC-004 — 9 acceptance criteria (FEAT-18.SPEC-004-AC-01…FEAT-18.SPEC-004-AC-09) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-004-subscription-lapse-account-pause-trigger.md
- FEAT-18.SPEC-005 — 19 acceptance criteria (FEAT-18.SPEC-005-AC-01…FEAT-18.SPEC-005-AC-19) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md
- FEAT-18.SPEC-006 — 15 acceptance criteria (FEAT-18.SPEC-006-AC-01…FEAT-18.SPEC-006-AC-15) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-006-subscription-billing-integration.md
- FEAT-18.SPEC-007 — 12 acceptance criteria (FEAT-18.SPEC-007-AC-01…FEAT-18.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-007-subscription-billing-notifications.md

All 90 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 10 — Real-Time Slot Availability Engine (FEAT-03)

Build feature FEAT-03 — Real-Time Slot Availability Engine (Core). The engine that computes, at the moment a client is looking, exactly which slots are genuinely free by combining working hours, buffers, bookings, time blocks and calendar busy time, plus short checkout holds and contention resolution.

**Already built (dependencies):** FEAT-02 (Availability & Working Hours Setup), FEAT-04 (Two-Way Calendar Sync). FEAT-17 (Manual Time Blocking), FEAT-21 (Recurring/Standing Appointments) follow later (see the order notes in the intro); build against a stub for now

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/):

- **FEAT-03.SPEC-001 — Slot Availability Computation** (automation): Computes, live and on demand, the complete set of genuinely open time slots for a chosen service and date range by combining the Pro's working hours and buffer, existing Bookings, manual Time Blocks, Recurring Series occurrences, active Slot Holds, and the Pro's connected-calendar busy time.
- **FEAT-03.SPEC-002 — Slot Hold Creation & Checkout Reservation** (automation): Creates a time-limited Slot Hold the instant a client begins paying for a chosen slot, instantly excluding it from every other client's computed availability so no second client can grab it mid-checkout.
- **FEAT-03.SPEC-003 — Slot Hold Expiration** (automation): Automatically expires and deletes a checkout Slot Hold that reaches its fixed timeout without completed payment, returning the slot to public availability.
- **FEAT-03.SPEC-004 — Slot Validation & Timing Rules** (logic-rule): Defines what makes any candidate time slot offerable -- the duration-plus-buffer fit, minimum notice, booking horizon, the Pro-only exception to notice and horizon, and the rule that every slot is always computed and labeled in the Pro's timezone.
- **FEAT-03.SPEC-005 — Slot Contention Resolution Rules** (logic-rule): Governs how a contested slot -- two clients attempting the same time, or a client colliding with a Pro-side change -- resolves: the first committed action wins, and every other attempt sees a plain re-pick message, never a payment error.
- **FEAT-03.SPEC-006 — Calendar Busy-Time Consumption & Degraded Mode** (integration): Consumes busy/free periods from the Pro's connected personal calendar (the connection itself owned by FEAT-04) as an additional availability constraint, and defines the fallback behavior -- Chairtime-only data with a Pro-only reduced-confidence flag -- when that sync becomes unavailable.
- **FEAT-03.SPEC-007 — Pro-Created Deposit Request Hold & Expiration** (automation): Reserves a slot the instant the Pro books a client in with a deposit request through FEAT-30, holding it up to 24 hours (platform parameter: `deposit-request-hold-max-hours`) or until 2 hours before the appointment (platform parameter: `deposit-request-hold-appointment-cutoff-hours`), whichever comes first, and expires the booking with a Pro notification if the deposit is never paid.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-012, ADR-014) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-03.SPEC-001 — 12 acceptance criteria (FEAT-03.SPEC-001-AC-01…FEAT-03.SPEC-001-AC-12) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-001-slot-availability-computation.md
- FEAT-03.SPEC-002 — 10 acceptance criteria (FEAT-03.SPEC-002-AC-01…FEAT-03.SPEC-002-AC-10) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-002-slot-hold-creation-checkout-reservation.md
- FEAT-03.SPEC-003 — 8 acceptance criteria (FEAT-03.SPEC-003-AC-01…FEAT-03.SPEC-003-AC-08) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-003-slot-hold-expiration.md
- FEAT-03.SPEC-004 — 14 acceptance criteria (FEAT-03.SPEC-004-AC-01…FEAT-03.SPEC-004-AC-14) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-004-slot-validation-timing-rules.md
- FEAT-03.SPEC-005 — 11 acceptance criteria (FEAT-03.SPEC-005-AC-01…FEAT-03.SPEC-005-AC-11) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-005-slot-contention-resolution-rules.md
- FEAT-03.SPEC-006 — 11 acceptance criteria (FEAT-03.SPEC-006-AC-01…FEAT-03.SPEC-006-AC-11) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-006-calendar-busy-time-consumption-degraded-mode.md
- FEAT-03.SPEC-007 — 12 acceptance criteria (FEAT-03.SPEC-007-AC-01…FEAT-03.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md

All 78 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 11 — Public Booking Page & Booking Flow (FEAT-05)

Build feature FEAT-05 — Public Booking Page & Booking Flow (Core). The single mobile-first public page reached from the Pro's Instagram bio link, where a client picks a service and a free time, enters name and phone, opts into texts, acknowledges the policy and pays the deposit in one continuous flow.

**Already built (dependencies):** FEAT-01 (Service & Pricing Management), FEAT-03 (Real-Time Slot Availability Engine), FEAT-09 (Cancellation & No-Show Policy Engine), FEAT-27 (Pro Profile & Booking Page Settings). FEAT-06 (Client Booking Identity), FEAT-07 (Deposit Payment at Booking) follow later (see the order notes in the intro); build against a stub for now

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/):

- **FEAT-05.SPEC-001 — Public Booking Page (Landing & Service List)** (screen): Entry point a client reaches from the Pro's Instagram bio link, showing the Pro's public profile and service list with prices, durations, and the deposit rule in plain words, from which the client picks a service to begin booking.
- **FEAT-05.SPEC-002 — Slot Selection** (screen): Client picks a genuinely free time for the chosen service from the live slot list, confirmed against the live slot check at the instant of the pick; the checkout hold itself is placed later, when the client advances into the payment step (FEAT-05.SPEC-004 -> FEAT-07.SPEC-001, per FEAT-03.SPEC-002 and XBR-02).
- **FEAT-05.SPEC-003 — Client Details & Consent** (screen): Client enters their name and phone, opts into text messages, provides an email if declining texts, and adds an optional note for the Pro.
- **FEAT-05.SPEC-004 — Policy Acknowledgment & Deposit Checkout** (screen): Client sees this booking's exact deposit and cancellation terms, explicitly acknowledges them, and continues into the deposit payment step (FEAT-07.SPEC-001), where the checkout hold starts and the deposit is paid.
- **FEAT-05.SPEC-005 — Booking Confirmation** (screen): Client sees an immediate on-screen confirmation of the completed, paid booking.
- **FEAT-05.SPEC-006 — Slot Hold & Re-Validation at Checkout** (automation): Creates the client's Booking in Pending Payment state and places a checkout hold on the chosen slot when the client advances into the deposit payment step (FEAT-05.SPEC-004 -> FEAT-07.SPEC-001), aligned to FEAT-03.SPEC-002; re-validates the hold immediately before charging, and resolves expiry or contention outcomes by returning the client to a fresh slot list.
- **FEAT-05.SPEC-007 — Booking Details Field Validation** (logic-rule): Defines all validation rules, conditional requirements, and authorization rules for the name, phone, texting opt-in, email, and note fields captured on the Client Details & Consent screen.
- **FEAT-05.SPEC-008 — Booking Page Availability Gate** (logic-rule): Determines, on every load of the booking link, whether to render the normal booking flow, a plain "not accepting bookings" message, or a plain "this booking page isn't available" message.
- **FEAT-05.SPEC-009 — Policy Acknowledgment Capture & Integrity Check** (logic-rule): Computes this booking's exact deposit amount and cancellation cut-off, captures the acknowledged cancellation policy version and wording into the in-progress checkout (carried onto the Booking when FEAT-05.SPEC-006 creates it), and re-validates that version still matches at payment time.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-001, ADR-020, ADR-021, ADR-023) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-05.SPEC-001 — 12 acceptance criteria (FEAT-05.SPEC-001-AC-01…FEAT-05.SPEC-001-AC-12) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md
- FEAT-05.SPEC-002 — 14 acceptance criteria (FEAT-05.SPEC-002-AC-01…FEAT-05.SPEC-002-AC-14) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-002-slot-selection.md
- FEAT-05.SPEC-003 — 13 acceptance criteria (FEAT-05.SPEC-003-AC-01…FEAT-05.SPEC-003-AC-13) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-003-client-details-consent.md
- FEAT-05.SPEC-004 — 15 acceptance criteria (FEAT-05.SPEC-004-AC-01…FEAT-05.SPEC-004-AC-15) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-004-policy-acknowledgment-deposit-checkout.md
- FEAT-05.SPEC-005 — 12 acceptance criteria (FEAT-05.SPEC-005-AC-01…FEAT-05.SPEC-005-AC-12) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-005-booking-confirmation.md
- FEAT-05.SPEC-006 — 12 acceptance criteria (FEAT-05.SPEC-006-AC-01…FEAT-05.SPEC-006-AC-12) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-006-slot-hold-re-validation-at-checkout.md
- FEAT-05.SPEC-007 — 16 acceptance criteria (FEAT-05.SPEC-007-AC-01…FEAT-05.SPEC-007-AC-16) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-007-booking-details-field-validation.md
- FEAT-05.SPEC-008 — 13 acceptance criteria (FEAT-05.SPEC-008-AC-01…FEAT-05.SPEC-008-AC-13) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-008-booking-page-availability-gate.md
- FEAT-05.SPEC-009 — 14 acceptance criteria (FEAT-05.SPEC-009-AC-01…FEAT-05.SPEC-009-AC-14) — docs/blueprint/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md

All 121 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 12 — Client Booking Identity (FEAT-06)

Build feature FEAT-06 — Client Booking Identity (Core). The password-free way a client proves identity when returning: a phone number plus a one-tap link, giving a my-bookings list, per-booking manage links and consent and email preferences.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-06-client-booking-identity/):

- **FEAT-06.SPEC-001 — Access Link Request** (screen): Client enters the phone number they used at booking and requests a one-tap access link sent to that phone, so they can view or manage their bookings with this one Pro without a password.
- **FEAT-06.SPEC-002 — Access Link Validation & Redemption** (automation): System validates a tapped access link -- on-demand or booking-specific -- checks its scope, expiry, and used/unused state, writes the resulting state, and routes the client to the right screen.
- **FEAT-06.SPEC-003 — My Bookings List** (screen): Client views their own past and upcoming bookings with this one Pro after redeeming an on-demand access link.
- **FEAT-06.SPEC-004 — Booking Detail via Manage Link** (screen): Client views one booking's full detail, opened either from the My Bookings list or directly via a booking-specific manage link, and starts a cancel, reschedule, balance payment, or preferences action from it.
- **FEAT-06.SPEC-005 — Consent & Email Preferences** (screen): Client updates their own texting consent (including re-granting it after opting out) and their email address for this Pro.
- **FEAT-06.SPEC-006 — Access Link Delivery** (notification): Sends the requested one-tap on-demand access link by text (with active consent) or email, so the client can view their bookings, with an immediate retry on delivery failure.
- **FEAT-06.SPEC-007 — Access Link Lifecycle & Scope Rules** (logic-rule): Governs the Access Link entity's expiry (30-minute single-use vs. booking-specific until the appointment passes), single-use enforcement, scope, and the per-phone-number request rate limit.
- **FEAT-06.SPEC-008 — Client Identity & Privacy Isolation Rule** (logic-rule): Governs phone-to-Client matching scoped to one Pro, and the hard boundary that no client can ever see another phone number's or Pro's bookings.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-025, ADR-026) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-06.SPEC-001 — 12 acceptance criteria (FEAT-06.SPEC-001-AC-01…FEAT-06.SPEC-001-AC-12) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-001-access-link-request.md
- FEAT-06.SPEC-002 — 11 acceptance criteria (FEAT-06.SPEC-002-AC-01…FEAT-06.SPEC-002-AC-11) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-002-access-link-validation-redemption.md
- FEAT-06.SPEC-003 — 12 acceptance criteria (FEAT-06.SPEC-003-AC-01…FEAT-06.SPEC-003-AC-12) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-003-my-bookings-list.md
- FEAT-06.SPEC-004 — 11 acceptance criteria (FEAT-06.SPEC-004-AC-01…FEAT-06.SPEC-004-AC-11) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md
- FEAT-06.SPEC-005 — 16 acceptance criteria (FEAT-06.SPEC-005-AC-01…FEAT-06.SPEC-005-AC-16) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-005-consent-email-preferences.md
- FEAT-06.SPEC-006 — 10 acceptance criteria (FEAT-06.SPEC-006-AC-01…FEAT-06.SPEC-006-AC-10) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-006-access-link-delivery.md
- FEAT-06.SPEC-007 — 13 acceptance criteria (FEAT-06.SPEC-007-AC-01…FEAT-06.SPEC-007-AC-13) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-007-access-link-lifecycle-scope-rules.md
- FEAT-06.SPEC-008 — 12 acceptance criteria (FEAT-06.SPEC-008-AC-01…FEAT-06.SPEC-008-AC-12) — docs/blueprint/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-008-client-identity-privacy-isolation-rule.md

All 97 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 13 — Deposit Payment at Booking (FEAT-07)

Build feature FEAT-07 — Deposit Payment at Booking (Core). The client pays a card deposit — fixed or percentage per the Pro's rule — at booking, with the balance due in person; payment outcome comes only from the processor webhook and is idempotent.

**Already built (dependencies):** FEAT-01 (Service & Pricing Management), FEAT-05 (Public Booking Page & Booking Flow), FEAT-28 (Payout Account Connection & Payout Visibility)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/):

- **FEAT-07.SPEC-001 — Deposit Payment** (screen): Riley enters card details for the exact, already-computed deposit and sees processing, decline/retry, and success states before handing off to the booking confirmation.
- **FEAT-07.SPEC-002 — Deposit Capture & Booking Confirmation** (automation): On a successful card charge, the system creates the Deposit Transaction record and atomically flips the Booking from Pending Payment to Confirmed, enforcing exactly one charge per booking.
- **FEAT-07.SPEC-003 — Deposit Amount & Eligibility Rules** (logic-rule): Governs how the deposit amount is computed exactly once from the service's rule, that it can never be altered by the client, and the preconditions that must hold before any charge is attempted.
- **FEAT-07.SPEC-004 — Payment Outcome Consistency & Idempotency** (logic-rule): Guarantees every payment attempt ends in exactly one of a clean success or a clean, actionable failure -- never a double charge and never an ambiguous booking state, including when the confirmation UI itself fails to load or the connection drops mid-payment.
- **FEAT-07.SPEC-005 — Card Deposit Charge & Payout Routing** (integration): Authorizes and captures Riley's card charge through the payment-processing capability, reports the processor's own card fee, and routes the captured deposit to Talia's connected payout account with zero platform fee.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-021, ADR-022) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-07.SPEC-001 — 19 acceptance criteria (FEAT-07.SPEC-001-AC-01…FEAT-07.SPEC-001-AC-19) — docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-001-deposit-payment.md
- FEAT-07.SPEC-002 — 14 acceptance criteria (FEAT-07.SPEC-002-AC-01…FEAT-07.SPEC-002-AC-14) — docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-002-deposit-capture-booking-confirmation.md
- FEAT-07.SPEC-003 — 16 acceptance criteria (FEAT-07.SPEC-003-AC-01…FEAT-07.SPEC-003-AC-16) — docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-003-deposit-amount-eligibility-rules.md
- FEAT-07.SPEC-004 — 14 acceptance criteria (FEAT-07.SPEC-004-AC-01…FEAT-07.SPEC-004-AC-14) — docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-004-payment-outcome-consistency-idempotency.md
- FEAT-07.SPEC-005 — 16 acceptance criteria (FEAT-07.SPEC-005-AC-01…FEAT-07.SPEC-005-AC-16) — docs/blueprint/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-005-card-deposit-charge-payout-routing.md

All 79 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 14 — Client-Initiated Cancel/Reschedule (FEAT-10)

Build feature FEAT-10 — Client-Initiated Cancel/Reschedule (Core). A client can cancel or reschedule their own booking within the Pro's policy, from the reminder's one-tap option or their manage link, with the deposit outcome applied per the policy.

**Already built (dependencies):** FEAT-03 (Real-Time Slot Availability Engine), FEAT-06 (Client Booking Identity), FEAT-09 (Cancellation & No-Show Policy Engine)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/):

- **FEAT-10.SPEC-001 — Cancel Booking** (screen): Client views the cancellation window countdown and deposit outcome for their own upcoming booking and confirms or backs out of cancelling it.
- **FEAT-10.SPEC-002 — Reschedule -- Select New Time** (screen): Client picks a new, genuinely free time for the same service from the same live slot list a fresh booking would use.
- **FEAT-10.SPEC-003 — Reschedule -- Outcome & Confirm** (screen): Client sees the deposit outcome for the chosen new time (carried-over deposit, or late-reschedule deposit-kept-plus-new-deposit-needed) before confirming.
- **FEAT-10.SPEC-004 — Booking Update Commit** (automation): Commits the client's confirmed cancellation or reschedule to the Booking record, coordinating the deposit outcome, calendar mirroring, activity logging, and freed-slot handoff this triggers in other features.
- **FEAT-10.SPEC-005 — Cancellation Window & Eligibility Rule** (logic-rule): Governs whether a booking is currently eligible to be cancelled or rescheduled by its client, and computes the window countdown that determines which deposit-outcome branch applies.
- **FEAT-10.SPEC-006 — Cancellation/Reschedule Notification** (notification): Tells the client their cancellation or reschedule went through (with the exact deposit outcome) and tells the Pro that a client-initiated change happened, once FEAT-10.SPEC-004's commit succeeds.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-021, ADR-012) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-10.SPEC-001 — 17 acceptance criteria (FEAT-10.SPEC-001-AC-01…FEAT-10.SPEC-001-AC-17) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-001-cancel-booking.md
- FEAT-10.SPEC-002 — 15 acceptance criteria (FEAT-10.SPEC-002-AC-01…FEAT-10.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-002-reschedule-select-new-time.md
- FEAT-10.SPEC-003 — 17 acceptance criteria (FEAT-10.SPEC-003-AC-01…FEAT-10.SPEC-003-AC-17) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-003-reschedule-outcome-confirm.md
- FEAT-10.SPEC-004 — 16 acceptance criteria (FEAT-10.SPEC-004-AC-01…FEAT-10.SPEC-004-AC-16) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-004-booking-update-commit.md
- FEAT-10.SPEC-005 — 14 acceptance criteria (FEAT-10.SPEC-005-AC-01…FEAT-10.SPEC-005-AC-14) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-005-cancellation-window-eligibility-rule.md
- FEAT-10.SPEC-006 — 12 acceptance criteria (FEAT-10.SPEC-006-AC-01…FEAT-10.SPEC-006-AC-12) — docs/blueprint/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-006-cancellation-reschedule-notification.md

All 91 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 15 — No-Show Marking & Deposit Forfeiture (FEAT-11)

Build feature FEAT-11 — No-Show Marking & Deposit Forfeiture (Core). The Pro marks a booking as a no-show and the deposit is forfeited to the Pro automatically under the agreed policy, with an undo window and clear authorization rules.

**Already built (dependencies):** FEAT-07 (Deposit Payment at Booking), FEAT-09 (Cancellation & No-Show Policy Engine)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-11-no-show-marking-deposit-forfeiture/):

- **FEAT-11.SPEC-001 — No-Show Mark & Undo Prompt** (screen): Talia reaches this one prompt from a past-due booking row to mark a no-show, choose a goodwill refund instead, or -- within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) -- undo a mark she already made, seeing the current deposit outcome reflected inline the instant she confirms.
- **FEAT-11.SPEC-002 — No-Show Marking & Deposit Forfeiture** (automation): On Talia's confirmed mark, atomically transitions the Booking to No-Show and its Deposit Transaction to Forfeited in one step, deriving the forfeiture outcome from the booking's acknowledged cancellation policy version, with no separate invoicing step and no manual chasing.
- **FEAT-11.SPEC-003 — No-Show Mark Undo** (automation): Within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`), atomically reverses a no-show mark -- restoring the Booking to Completed and the Deposit Transaction to its prior Captured status -- when Talia confirms the mistake was hers.
- **FEAT-11.SPEC-004 — No-Show Marking Window & Authorization Rules** (logic-rule): Governs who may mark or undo a no-show (the Pro, on their own bookings only), the eligible marking window (after the appointment start time, before the booking auto-completes), and the 24-hour undo grace window (platform parameter: `no-show-undo-grace-window-hours`), shared by the prompt and both automations rather than duplicated in each.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-021) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-11.SPEC-001 — 15 acceptance criteria (FEAT-11.SPEC-001-AC-01…FEAT-11.SPEC-001-AC-15) — docs/blueprint/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-001-no-show-mark-undo-prompt.md
- FEAT-11.SPEC-002 — 11 acceptance criteria (FEAT-11.SPEC-002-AC-01…FEAT-11.SPEC-002-AC-11) — docs/blueprint/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-002-no-show-marking-deposit-forfeiture.md
- FEAT-11.SPEC-003 — 11 acceptance criteria (FEAT-11.SPEC-003-AC-01…FEAT-11.SPEC-003-AC-11) — docs/blueprint/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-003-no-show-mark-undo.md
- FEAT-11.SPEC-004 — 17 acceptance criteria (FEAT-11.SPEC-004-AC-01…FEAT-11.SPEC-004-AC-17) — docs/blueprint/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md

All 54 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 16 — Client Record Management (FEAT-13)

Build feature FEAT-13 — Client Record Management (Important). The Pro keeps a simple record per client — contact details, private notes, booking history — and can permanently delete a client's record on request, subject to retention rules.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-13-client-record-management/):

- **FEAT-13.SPEC-001 — Client Record Detail** (screen): The Pro views a client's contact details, private note, and full booking history with this Pro, and edits the private note directly on this screen.
- **FEAT-13.SPEC-002 — Client Contact Edit** (screen): The Pro corrects a client's name, email, or phone number.
- **FEAT-13.SPEC-003 — Client Deletion Confirmation** (screen): The Pro requests permanent deletion of a client's record, sees the eligibility check result, and confirms an irreversible delete.
- **FEAT-13.SPEC-004 — Client Deletion Execution** (automation): Hard-deletes the client's contact details and private note, cascades to remove Messaging Consent, and retains de-identified financial and timeline history.
- **FEAT-13.SPEC-005 — Client Field Validation & Access Rules** (logic-rule): Governs name/phone/email/note field validation, phone-number identity uniqueness within one Pro, and the private-note field's Pro-only visibility.
- **FEAT-13.SPEC-006 — Deletion Eligibility & Retention Rule** (logic-rule): Governs when a client record may be deleted (blocked by an upcoming booking), the irreversibility of deletion, and the retention/de-identification of related records afterward.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-011) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-13.SPEC-001 — 15 acceptance criteria (FEAT-13.SPEC-001-AC-01…FEAT-13.SPEC-001-AC-15) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-001-client-record-detail.md
- FEAT-13.SPEC-002 — 13 acceptance criteria (FEAT-13.SPEC-002-AC-01…FEAT-13.SPEC-002-AC-13) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-002-client-contact-edit.md
- FEAT-13.SPEC-003 — 13 acceptance criteria (FEAT-13.SPEC-003-AC-01…FEAT-13.SPEC-003-AC-13) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-003-client-deletion-confirmation.md
- FEAT-13.SPEC-004 — 10 acceptance criteria (FEAT-13.SPEC-004-AC-01…FEAT-13.SPEC-004-AC-10) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-004-client-deletion-execution.md
- FEAT-13.SPEC-005 — 17 acceptance criteria (FEAT-13.SPEC-005-AC-01…FEAT-13.SPEC-005-AC-17) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-005-client-field-validation-access-rules.md
- FEAT-13.SPEC-006 — 14 acceptance criteria (FEAT-13.SPEC-006-AC-01…FEAT-13.SPEC-006-AC-14) — docs/blueprint/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-006-deletion-eligibility-retention-rule.md

All 82 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 17 — Messaging Consent Management (FEAT-14)

Build feature FEAT-14 — Messaging Consent Management (Important). Captures a client's explicit text opt-in at booking, lets them withdraw consent any time (including by STOP), and ensures every message respects the current consent state.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-14-messaging-consent-management/):

- **FEAT-14.SPEC-001 — Consent & Preferences** (screen): The consent section of the client's Preferences screen: the client's current texting consent status for this Pro, with an action to opt back in to texting when it is currently off. This section is hosted inside FEAT-06.SPEC-005 (Consent & Email Preferences), the single client-facing Preferences screen; it is not a separately reachable screen.
- **FEAT-14.SPEC-002 — Opt-Out Link Landing** (screen): The page a client lands on after tapping the opt-out link included in a text message, confirming that texting has been turned off.
- **FEAT-14.SPEC-003 — Consent Capture at Booking** (automation): Records the client's opt-in or opt-out choice made at booking as a Messaging Consent record, with state, timestamp, channel, and the exact consent wording shown, creating the record on a client's first booking with a Pro and updating it on a later booking if the choice changes.
- **FEAT-14.SPEC-004 — Opt-Out / STOP Processing** (automation): Revokes a client's texting consent immediately, with no grace period, whether triggered by a tapped opt-out link or an inbound "STOP" text reply.
- **FEAT-14.SPEC-005 — Consent Re-Grant Action** (automation): Records a client's opt-back-in to texting when they tap "Turn texting back on" on the Consent & Preferences screen.
- **FEAT-14.SPEC-006 — Concurrent Consent Update Resolution** (logic-rule): Resolves a STOP reply and an in-app re-grant (or any two consent-changing writes) arriving close together for the same client-Pro relationship, by most-recent-explicit-action timestamp, defaulting to the no-text state when the outcome is uncertain.
- **FEAT-14.SPEC-007 — Textability Determination Rule** (logic-rule): Computes, as a single authoritative answer, whether a given client is currently textable for a given Pro relationship -- the value every other feature reads instead of re-deriving consent logic itself.
- **FEAT-14.SPEC-008 — Phone Number Change Consent Invalidation Rule** (logic-rule): Invalidates a client's existing texting consent whenever their phone number changes, so fresh consent is required before the new number is ever texted.
- **FEAT-14.SPEC-009 — Opt-Out Confirmation Message** (notification): Acknowledges to a client, immediately after a STOP reply is processed, that texting has been turned off and that their confirmations and reminders will now arrive by email instead.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-009) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-14.SPEC-001 — 9 acceptance criteria (FEAT-14.SPEC-001-AC-01…FEAT-14.SPEC-001-AC-09) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-001-consent-and-preferences.md
- FEAT-14.SPEC-002 — 8 acceptance criteria (FEAT-14.SPEC-002-AC-01…FEAT-14.SPEC-002-AC-08) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-002-opt-out-link-landing.md
- FEAT-14.SPEC-003 — 10 acceptance criteria (FEAT-14.SPEC-003-AC-01…FEAT-14.SPEC-003-AC-10) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-003-consent-capture-at-booking.md
- FEAT-14.SPEC-004 — 11 acceptance criteria (FEAT-14.SPEC-004-AC-01…FEAT-14.SPEC-004-AC-11) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-004-opt-out-stop-processing.md
- FEAT-14.SPEC-005 — 8 acceptance criteria (FEAT-14.SPEC-005-AC-01…FEAT-14.SPEC-005-AC-08) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-005-consent-re-grant-action.md
- FEAT-14.SPEC-006 — 11 acceptance criteria (FEAT-14.SPEC-006-AC-01…FEAT-14.SPEC-006-AC-11) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-006-concurrent-consent-update-resolution.md
- FEAT-14.SPEC-007 — 10 acceptance criteria (FEAT-14.SPEC-007-AC-01…FEAT-14.SPEC-007-AC-10) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-007-textability-determination-rule.md
- FEAT-14.SPEC-008 — 9 acceptance criteria (FEAT-14.SPEC-008-AC-01…FEAT-14.SPEC-008-AC-09) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-008-phone-number-change-consent-invalidation-rule.md
- FEAT-14.SPEC-009 — 10 acceptance criteria (FEAT-14.SPEC-009-AC-01…FEAT-14.SPEC-009-AC-10) — docs/blueprint/specifications/FEAT-14-messaging-consent-management/FEAT-14.SPEC-009-opt-out-confirmation-message.md

All 86 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 18 — Automated Booking Messaging (FEAT-08)

Build feature FEAT-08 — Automated Booking Messaging (Core). The client gets an immediate confirmation once a booking is paid and an automatic reminder with a one-tap response, plus change and refund notices, Pro notifications, retry and email fallback, all consent-aware.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow), FEAT-07 (Deposit Payment at Booking), FEAT-14 (Messaging Consent Management), FEAT-27 (Pro Profile & Booking Page Settings)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-08-automated-booking-messaging/):

- **FEAT-08.SPEC-001 — Booking Confirmation Message** (notification): Tells the client, the moment their deposit payment succeeds, that their appointment is confirmed -- carrying every detail they need (what, when, where, what was paid, what's still owed, when they can no longer cancel free, and how to manage or add the booking to their own calendar) without needing to ask the Pro anything.
- **FEAT-08.SPEC-002 — Appointment Reminder Message** (notification): Reminds the client before their appointment and offers a one-tap "I'll be there" or "I need to reschedule" response, replacing the Pro's habit of texting reminders by hand.
- **FEAT-08.SPEC-003 — Reminder Reply Acknowledgment** (screen): Confirms to the client, after they tap "I'll be there" in a reminder, that their attendance was recorded -- and handles the cases where the tapped link no longer works.
- **FEAT-08.SPEC-004 — Booking Change & Refund Notice** (notification): Tells the client, promptly, when their booking is cancelled, rescheduled, or refunded by either themselves or the Pro -- including exactly what happened to their deposit -- so no client is ever left wondering whether a change went through or whether their money is safe.
- **FEAT-08.SPEC-005 — Pro Booking Activity Notification** (notification): Tells the Pro, on her own channels, that a new booking arrived or a client cancelled or rescheduled their own appointment -- so Talia learns about routine schedule changes without having to keep the dashboard open.
- **FEAT-08.SPEC-006 — Pro Attention Alert** (notification): Tells the Pro immediately when something needs her attention -- a message delivery failure, a calendar connection needing reconnection, a refund that failed to complete, a card-issuer dispute, a reminder that failed to schedule, or a deposit request that expired unpaid -- so nothing about her business is ever a silent failure she discovers late.
- **FEAT-08.SPEC-007 — Reminder Scheduling & Timing Window Enforcement** (automation): Computes when each confirmed booking's reminder should fire, keeps every reminder inside the daytime send window (roughly 8am--9pm per XBR-16; platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`) in the Pro's timezone, and suppresses a separate reminder when a booking is made after its own reminder point has already passed.
- **FEAT-08.SPEC-008 — Reminder Reply Routing** (automation): Processes whichever one-tap reply a client makes on a reminder -- recording "I'll be there" as an acknowledgment, or handing "I need to reschedule" off into the reschedule flow.
- **FEAT-08.SPEC-009 — Message Delivery Retry & Fallback** (automation): When a text fails to deliver, retries it once and then falls back to email, flagging the delivery gap on the Pro's dashboard so no message this feature sends is ever silently dropped.
- **FEAT-08.SPEC-010 — Booking-Specific Manage Link Issuance** (automation): Mints the booking-specific manage link carried in every confirmation and reminder, scoped to exactly one booking, and reissues a fresh one whenever the Pro reschedules that booking.
- **FEAT-08.SPEC-011 — Messaging Consent & Channel Selection Rule** (logic-rule): Decides text vs. email for every outbound client-directed message this feature sends, based on the client's active Messaging Consent, honoring a revoke on the very next message and requiring fresh consent after a phone number change.
- **FEAT-08.SPEC-012 — Transactional Text Messaging Capability** (integration): Sends every text message the product needs to deliver -- confirmations, reminders, change notices, Pro notifications, access links, and every other feature's text-based messages -- through an external text-messaging capability, and reports back each message's delivery status.
- **FEAT-08.SPEC-013 — Transactional Email Capability** (integration): Sends every email message the product needs to deliver -- as the fallback channel after a failed text and as the primary channel for clients who decline texting -- through an external transactional-email capability, and reports back each message's delivery status.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-009, ADR-012, ADR-022) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-08.SPEC-001 — 15 acceptance criteria (FEAT-08.SPEC-001-AC-01…FEAT-08.SPEC-001-AC-15) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-001-booking-confirmation-message.md
- FEAT-08.SPEC-002 — 12 acceptance criteria (FEAT-08.SPEC-002-AC-01…FEAT-08.SPEC-002-AC-12) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-002-appointment-reminder-message.md
- FEAT-08.SPEC-003 — 11 acceptance criteria (FEAT-08.SPEC-003-AC-01…FEAT-08.SPEC-003-AC-11) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-003-reminder-reply-acknowledgment.md
- FEAT-08.SPEC-004 — 13 acceptance criteria (FEAT-08.SPEC-004-AC-01…FEAT-08.SPEC-004-AC-13) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-004-booking-change-refund-notice.md
- FEAT-08.SPEC-005 — 12 acceptance criteria (FEAT-08.SPEC-005-AC-01…FEAT-08.SPEC-005-AC-12) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-005-pro-booking-activity-notification.md
- FEAT-08.SPEC-006 — 16 acceptance criteria (FEAT-08.SPEC-006-AC-01…FEAT-08.SPEC-006-AC-16) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-006-pro-attention-alert.md
- FEAT-08.SPEC-007 — 12 acceptance criteria (FEAT-08.SPEC-007-AC-01…FEAT-08.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-007-reminder-scheduling-timing-window-enforcement.md
- FEAT-08.SPEC-008 — 11 acceptance criteria (FEAT-08.SPEC-008-AC-01…FEAT-08.SPEC-008-AC-11) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-008-reminder-reply-routing.md
- FEAT-08.SPEC-009 — 12 acceptance criteria (FEAT-08.SPEC-009-AC-01…FEAT-08.SPEC-009-AC-12) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-009-message-delivery-retry-fallback.md
- FEAT-08.SPEC-010 — 11 acceptance criteria (FEAT-08.SPEC-010-AC-01…FEAT-08.SPEC-010-AC-11) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-010-booking-specific-manage-link-issuance.md
- FEAT-08.SPEC-011 — 15 acceptance criteria (FEAT-08.SPEC-011-AC-01…FEAT-08.SPEC-011-AC-15) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-011-messaging-consent-channel-selection-rule.md
- FEAT-08.SPEC-012 — 14 acceptance criteria (FEAT-08.SPEC-012-AC-01…FEAT-08.SPEC-012-AC-14) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-012-transactional-text-messaging-capability.md
- FEAT-08.SPEC-013 — 13 acceptance criteria (FEAT-08.SPEC-013-AC-01…FEAT-08.SPEC-013-AC-13) — docs/blueprint/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-013-transactional-email-capability.md

All 167 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 19 — Pro Daily Schedule Dashboard (FEAT-12)

Build feature FEAT-12 — Pro Daily Schedule Dashboard (Core). The Pro's phone-first home screen: today's and upcoming bookings with paid badges, client notes and balance still due, an attention list, past bookings and automatic completion.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow), FEAT-07 (Deposit Payment at Booking), FEAT-08 (Automated Booking Messaging), FEAT-13 (Client Record Management), FEAT-29 (Pro Sign-In & Account Lifecycle)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/):

- **FEAT-12.SPEC-001 — Today's & Upcoming Schedule** (screen): The Pro's main dashboard: today's remaining bookings in time order plus upcoming bookings beyond today, each with a paid badge, balance due, "I'll be there" status, sync-reliability marking, and quick-action entry points.
- **FEAT-12.SPEC-002 — Attention List** (screen): Surfaces everything needing the Pro's attention -- sync issues, message delivery failures, refunds in progress, card-issuer disputes, bookings left outside changed hours, and waitlist demand -- in one place.
- **FEAT-12.SPEC-003 — Past Bookings Browse** (screen): The Pro finds and reviews a past booking by browsing by date, using the same booking row pattern as the main schedule.
- **FEAT-12.SPEC-004 — Auto-Completion Sweep** (automation): Automatically marks a Booking Completed 7 days (platform parameter: `booking-auto-completion-window-days`) after its appointment time if the Pro never marked it Completed or No-Show.
- **FEAT-12.SPEC-005 — Attention Flag Aggregation** (automation): Gathers and de-duplicates attention-worthy signals owned by other features (calendar sync health, message delivery, refund progress, card-issuer disputes, setup-change conflicts) into a single Attention List feed, and tracks each item's resolution.
- **FEAT-12.SPEC-006 — Booking Completion Rules** (logic-rule): Governs when a Booking may be marked Completed (by the Pro or automatically), and how completion interacts with the booking's remaining lifecycle actions.
- **FEAT-12.SPEC-007 — Balance Due & Status Display Rules** (logic-rule): Derives the balance-due amount and the paid/unpaid, "I'll be there," and sync-reliability display states shown consistently on every booking row across this feature's three screens.
- **FEAT-12.SPEC-008 — Dashboard Access Authorization** (logic-rule): Enforces who may open this feature's three screens and what each role sees: the Pro's own schedule in full, Support's masked read-only view for troubleshooting, and a redirect to sign-in for anyone else.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-006, ADR-014) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-12.SPEC-001 — 19 acceptance criteria (FEAT-12.SPEC-001-AC-01…FEAT-12.SPEC-001-AC-19) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-001-todays-upcoming-schedule.md
- FEAT-12.SPEC-002 — 15 acceptance criteria (FEAT-12.SPEC-002-AC-01…FEAT-12.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-002-attention-list.md
- FEAT-12.SPEC-003 — 15 acceptance criteria (FEAT-12.SPEC-003-AC-01…FEAT-12.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-003-past-bookings-browse.md
- FEAT-12.SPEC-004 — 9 acceptance criteria (FEAT-12.SPEC-004-AC-01…FEAT-12.SPEC-004-AC-09) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-004-auto-completion-sweep.md
- FEAT-12.SPEC-005 — 12 acceptance criteria (FEAT-12.SPEC-005-AC-01…FEAT-12.SPEC-005-AC-12) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-005-attention-flag-aggregation.md
- FEAT-12.SPEC-006 — 14 acceptance criteria (FEAT-12.SPEC-006-AC-01…FEAT-12.SPEC-006-AC-14) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-006-booking-completion-rules.md
- FEAT-12.SPEC-007 — 15 acceptance criteria (FEAT-12.SPEC-007-AC-01…FEAT-12.SPEC-007-AC-15) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-007-balance-due-status-display-rules.md
- FEAT-12.SPEC-008 — 13 acceptance criteria (FEAT-12.SPEC-008-AC-01…FEAT-12.SPEC-008-AC-13) — docs/blueprint/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-008-dashboard-access-authorization.md

All 112 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 20 — Booking & Payment Activity Record (FEAT-16)

Build feature FEAT-16 — Booking & Payment Activity Record (Important). An append-only record of every booking's key events — created, paid, messaged, cancelled, rescheduled, no-show, refunded — giving the Pro (or Support) a trustworthy timeline for disputes, with a downloadable summary.

**Already built (dependencies):** FEAT-05 (Public Booking Page & Booking Flow), FEAT-07 (Deposit Payment at Booking), FEAT-08 (Automated Booking Messaging), FEAT-10 (Client-Initiated Cancel/Reschedule), FEAT-11 (No-Show Marking & Deposit Forfeiture)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/):

- **FEAT-16.SPEC-001 — Booking Activity Timeline** (screen): The Pro (or Platform Operator Support, read-only) opens a single booking's full, ordered event history to reference when preparing to respond to a client dispute, and can act from it by starting a goodwill refund or requesting a dispute evidence summary.
- **FEAT-16.SPEC-002 — Activity Event Recording** (automation): Writes one immutable, append-only Activity Event for every qualifying action across Booking, Deposit Payment, Messaging, Cancellation/Reschedule, No-Show, Client Deletion, Payout, Pro-initiated cancel/reschedule, and Support View, so a complete timeline exists for FEAT-16.SPEC-001 to render.
- **FEAT-16.SPEC-003 — Card-Issuer Dispute Integration** (integration): Receives an inbound card-issuer dispute notice from the payment-processing capability, flags the affected booking, sets the Deposit Transaction's Disputed overlay without erasing its underlying outcome, and records the dispute as an Activity Event.
- **FEAT-16.SPEC-004 — Dispute Summary Download** (automation): Assembles a plain-language, shareable summary of a disputed booking's timeline -- policy shown and acknowledged, booking time, messages sent, and no-show mark -- and hands it to the Pro as a downloadable file to submit as evidence with the payment processor.
- **FEAT-16.SPEC-005 — Activity Record Immutability & Visibility Rules** (logic-rule): Governs append-only enforcement (no role, including the Pro, may edit an entry), View-only access for the Pro and Support, retention tied to the Booking's life, de-identification on client deletion, and exclusion of the Pro's private client notes from the Support view.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-008) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-16.SPEC-001 — 19 acceptance criteria (FEAT-16.SPEC-001-AC-01…FEAT-16.SPEC-001-AC-19) — docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-001-booking-activity-timeline.md
- FEAT-16.SPEC-002 — 23 acceptance criteria (FEAT-16.SPEC-002-AC-01…FEAT-16.SPEC-002-AC-23) — docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-002-activity-event-recording.md
- FEAT-16.SPEC-003 — 13 acceptance criteria (FEAT-16.SPEC-003-AC-01…FEAT-16.SPEC-003-AC-13) — docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-003-card-issuer-dispute-integration.md
- FEAT-16.SPEC-004 — 14 acceptance criteria (FEAT-16.SPEC-004-AC-01…FEAT-16.SPEC-004-AC-14) — docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-004-dispute-summary-download.md
- FEAT-16.SPEC-005 — 18 acceptance criteria (FEAT-16.SPEC-005-AC-01…FEAT-16.SPEC-005-AC-18) — docs/blueprint/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-005-activity-record-immutability-visibility-rules.md

All 87 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 21 — Platform Support Read-Only Access (FEAT-19)

Build feature FEAT-19 — Platform Support Read-Only Access (Important). The founder, in a support capacity, can open a read-only view of a specific Pro's account to troubleshoot, with every view logged and no edit or client-facing access.

**Already built (dependencies):** FEAT-12 (Pro Daily Schedule Dashboard), FEAT-15 (Pro Onboarding & Setup Wizard), FEAT-16 (Booking & Payment Activity Record), FEAT-18 (Pro Subscription Billing & Account Management)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-19-platform-support-read-only-access/):

- **FEAT-19.SPEC-001 — Pro Account Lookup & Support Session Entry** (screen): Support looks up one specific Pro by request, records a reason or ticket reference, and opens a single-account, structurally read-only session that hands into the already-validated read-only surfaces of other features for services, schedule, bookings, billing status, and disputed booking timelines.
- **FEAT-19.SPEC-002 — Support View Logging** (automation): On session open, and on each disputed-booking timeline opened within it, assembles the support-view event (actor, time, reason/ticket reference) and hands it to FEAT-16.SPEC-002 to write as the Pro Account's Activity Event -- this feature never writes a second event store.
- **FEAT-19.SPEC-003 — Support Access Log** (screen): Renders the Pro's (and Support's own) account-level list of every past support view -- who, when, and the reason/ticket reference -- distinct from FEAT-16.SPEC-001's per-booking timeline.
- **FEAT-19.SPEC-004 — Support Session Scope & Access Rules** (logic-rule): Governs the structural no-write-path rule, one-Pro-account-at-a-time scoping (opening a new lookup ends the prior session), the "only after a Pro's help request" precondition, and the field-level exclusions (private client notes, bank/identity details, sign-in codes) that ground XBR-24.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-025, ADR-003) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-19.SPEC-001 — 20 acceptance criteria (FEAT-19.SPEC-001-AC-01…FEAT-19.SPEC-001-AC-20) — docs/blueprint/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-001-pro-account-lookup-support-session-entry.md
- FEAT-19.SPEC-002 — 15 acceptance criteria (FEAT-19.SPEC-002-AC-01…FEAT-19.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-002-support-view-logging.md
- FEAT-19.SPEC-003 — 14 acceptance criteria (FEAT-19.SPEC-003-AC-01…FEAT-19.SPEC-003-AC-14) — docs/blueprint/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-003-support-access-log.md
- FEAT-19.SPEC-004 — 27 acceptance criteria (FEAT-19.SPEC-004-AC-01…FEAT-19.SPEC-004-AC-27) — docs/blueprint/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-004-support-session-scope-access-rules.md

All 76 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 22 — Waitlist for Cancelled Slots (FEAT-20)

Build feature FEAT-20 — Waitlist for Cancelled Slots (Nice-to-Have). A client can ask to be notified when a specific service and day opens up from a cancellation; the first to complete a booking wins, with claim windows and expiry.

**Already built (dependencies):** FEAT-03 (Real-Time Slot Availability Engine), FEAT-10 (Client-Initiated Cancel/Reschedule)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/):

- **FEAT-20.SPEC-001 — Join Waitlist** (screen): Riley joins the waitlist for a specific service and a day (or up to a 7-day range) when the public booking page shows no free time, so she is notified the moment a cancellation opens a matching slot.
- **FEAT-20.SPEC-002 — My Waitlists** (screen): Riley views her own waitlist entries (position/status), sees the plain empty state when she holds none, and leaves any entry.
- **FEAT-20.SPEC-003 — Waitlist Entry Validation & Limits** (logic-rule): Governs what a valid waitlist join looks like -- the service/date-range shape, the 3-active-entries-per-Pro cap, and the notice/horizon bounds a joined date range must respect.
- **FEAT-20.SPEC-004 — Waitlist Priority & Claim Window Rule** (logic-rule): Governs which Requested entries match a freed slot, the 30-minute claim window, how the window interacts with general public availability, and how contested or withdrawn claims resolve.
- **FEAT-20.SPEC-005 — Cancellation-Triggered Waitlist Matching** (automation): On a freed-slot signal from a cancellation, finds every matching Requested entry, transitions each to Notified, and hands off to the opening notification.
- **FEAT-20.SPEC-006 — Waitlist Claim Conversion** (automation): When a notified client completes the ordinary booking flow for the matching slot, converts their entry to Converted and leaves the other notified entries untouched.
- **FEAT-20.SPEC-007 — Waitlist Entry Expiry** (automation): Expires a Notified entry whose 30-minute claim window lapses unclaimed, and separately expires a Requested entry whose joined date range elapses with no matching opening ever found.
- **FEAT-20.SPEC-008 — Waitlist Opening Notification** (notification): Notifies a matching client the moment their slot opens, states the 30-minute claim window, and carries the claim link into the booking flow.
- **FEAT-20.SPEC-009 — Waitlist Expiry Notification** (notification): Informs a client that their waitlist entry has expired -- either an unclaimed opening or an unmatched date range -- so they are never left wondering.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-012, ADR-009) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-20.SPEC-001 — 17 acceptance criteria (FEAT-20.SPEC-001-AC-01…FEAT-20.SPEC-001-AC-17) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-001-join-waitlist.md
- FEAT-20.SPEC-002 — 15 acceptance criteria (FEAT-20.SPEC-002-AC-01…FEAT-20.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-002-my-waitlists.md
- FEAT-20.SPEC-003 — 16 acceptance criteria (FEAT-20.SPEC-003-AC-01…FEAT-20.SPEC-003-AC-16) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-003-waitlist-entry-validation-limits.md
- FEAT-20.SPEC-004 — 15 acceptance criteria (FEAT-20.SPEC-004-AC-01…FEAT-20.SPEC-004-AC-15) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-004-waitlist-priority-claim-window-rule.md
- FEAT-20.SPEC-005 — 14 acceptance criteria (FEAT-20.SPEC-005-AC-01…FEAT-20.SPEC-005-AC-14) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-005-cancellation-triggered-waitlist-matching.md
- FEAT-20.SPEC-006 — 13 acceptance criteria (FEAT-20.SPEC-006-AC-01…FEAT-20.SPEC-006-AC-13) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-006-waitlist-claim-conversion.md
- FEAT-20.SPEC-007 — 13 acceptance criteria (FEAT-20.SPEC-007-AC-01…FEAT-20.SPEC-007-AC-13) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-007-waitlist-entry-expiry.md
- FEAT-20.SPEC-008 — 14 acceptance criteria (FEAT-20.SPEC-008-AC-01…FEAT-20.SPEC-008-AC-14) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-008-waitlist-opening-notification.md
- FEAT-20.SPEC-009 — 13 acceptance criteria (FEAT-20.SPEC-009-AC-01…FEAT-20.SPEC-009-AC-13) — docs/blueprint/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-009-waitlist-expiry-notification.md

All 130 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 23 — Recurring/Standing Appointments (FEAT-21)

Build feature FEAT-21 — Recurring/Standing Appointments (Nice-to-Have). A client can set up a standing appointment pattern (for example every 3 weeks) that generates individual bookings automatically, with conflict handling and deposit requests per occurrence.

**Already built (dependencies):** FEAT-03 (Real-Time Slot Availability Engine), FEAT-05 (Public Booking Page & Booking Flow)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/):

- **FEAT-21.SPEC-001 — Set Up Recurring Series** (screen): Lets Riley turn the booking she just made into a standing appointment by choosing how often it repeats, so future visits with Talia are generated automatically instead of booked one at a time.
- **FEAT-21.SPEC-002 — My Recurring Series** (screen): Lets Riley see her standing appointment series and its upcoming occurrences grouped together, and cancel the whole series or just one occurrence, with no extra UI at all when she holds no series.
- **FEAT-21.SPEC-003 — Recurring Series Setup & Generation Limits** (logic-rule): Defines the valid interval range for a Recurring Series, the booking-horizon ceiling that governs how far ahead occurrences may ever be generated, and who may create a series.
- **FEAT-21.SPEC-004 — Occurrence Generation & Conflict Handling** (automation): Generates each occurrence's Booking within the Pro's booking horizon as an Active series' due date arrives, subject to the same slot validation as any booking, and hands an occurrence whose usual time is no longer available to a client pick-a-new-time flow without breaking the rest of the series.
- **FEAT-21.SPEC-005 — Occurrence Deposit Request & Release** (automation): Sends each generated occurrence's own fresh deposit link about a week before it, and releases the occurrence if the deposit is never paid by its cancellation cut-off, without disturbing the rest of the series.
- **FEAT-21.SPEC-006 — Series & Occurrence Cancellation Rules** (logic-rule): Governs what cancelling the whole series does to its not-yet-occurred occurrences versus cancelling a single occurrence, and how a concurrent Client/Pro change to the same series or occurrence resolves.
- **FEAT-21.SPEC-007 — Occurrence Generated Notification** (notification): Confirms to the client, each time a standing appointment's next occurrence is generated, exactly which appointment has just been scheduled from her series.
- **FEAT-21.SPEC-008 — Occurrence Time Change Advance Notice** (notification): Gives the client advance notice, with a prompt to pick a new time, when a standing appointment's usual slot is no longer available for an upcoming occurrence -- without disturbing the rest of her series.
- **FEAT-21.SPEC-009 — Occurrence Deposit Lifecycle Notification** (notification): Sends the client her occurrence's own fresh deposit link about a week before it, and, separately, tells both the client and the Pro when an unpaid occurrence is released.
- **FEAT-21.SPEC-010 — Pro Recurring Series Management** (screen): Lets Talia set up a standing appointment for a client while the client is at the chair, then see that client's series and upcoming occurrences on her own schedule, and cancel one occurrence or end the whole series.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-012, ADR-003) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-21.SPEC-001 — 13 acceptance criteria (FEAT-21.SPEC-001-AC-01…FEAT-21.SPEC-001-AC-13) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-001-set-up-recurring-series.md
- FEAT-21.SPEC-002 — 14 acceptance criteria (FEAT-21.SPEC-002-AC-01…FEAT-21.SPEC-002-AC-14) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-002-my-recurring-series.md
- FEAT-21.SPEC-003 — 15 acceptance criteria (FEAT-21.SPEC-003-AC-01…FEAT-21.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-003-recurring-series-setup-generation-limits.md
- FEAT-21.SPEC-004 — 14 acceptance criteria (FEAT-21.SPEC-004-AC-01…FEAT-21.SPEC-004-AC-14) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-004-occurrence-generation-conflict-handling.md
- FEAT-21.SPEC-005 — 13 acceptance criteria (FEAT-21.SPEC-005-AC-01…FEAT-21.SPEC-005-AC-13) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-005-occurrence-deposit-request-release.md
- FEAT-21.SPEC-006 — 16 acceptance criteria (FEAT-21.SPEC-006-AC-01…FEAT-21.SPEC-006-AC-16) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-006-series-occurrence-cancellation-rules.md
- FEAT-21.SPEC-007 — 12 acceptance criteria (FEAT-21.SPEC-007-AC-01…FEAT-21.SPEC-007-AC-12) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-007-occurrence-generated-notification.md
- FEAT-21.SPEC-008 — 12 acceptance criteria (FEAT-21.SPEC-008-AC-01…FEAT-21.SPEC-008-AC-12) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-008-occurrence-time-change-advance-notice.md
- FEAT-21.SPEC-009 — 15 acceptance criteria (FEAT-21.SPEC-009-AC-01…FEAT-21.SPEC-009-AC-15) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-009-occurrence-deposit-lifecycle-notification.md
- FEAT-21.SPEC-010 — 22 acceptance criteria (FEAT-21.SPEC-010-AC-01…FEAT-21.SPEC-010-AC-22) — docs/blueprint/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md

All 146 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 24 — In-App Balance Payment (FEAT-22)

Build feature FEAT-22 — In-App Balance Payment (Nice-to-Have). A client can optionally pay the remaining balance in the app before or at the appointment instead of paying the Pro in person.

**Already built (dependencies):** FEAT-07 (Deposit Payment at Booking), FEAT-28 (Payout Account Connection & Payout Visibility)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-22-in-app-balance-payment/):

- **FEAT-22.SPEC-001 — Balance Payment** (screen): Riley views her deposit-paid-vs-balance-remaining running record on her own confirmed booking and optionally pays the balance in-app, seeing processing, decline, and success states.
- **FEAT-22.SPEC-002 — Balance Capture & Booking Status Update** (automation): On a successful in-app balance card charge, the system creates the Balance Payment record and updates the Booking's balance-due status to fully paid, so Talia's dashboard reflects it without a manual refresh.
- **FEAT-22.SPEC-003 — Balance Amount & Eligibility Rules** (logic-rule): Governs how the balance amount is derived once from the service price and the deposit already paid, that it can never be altered by the client, and the preconditions that must hold before any in-app balance charge is attempted.
- **FEAT-22.SPEC-004 — Balance Payment Outcome Consistency & Cancellation Contention** (logic-rule): Guarantees every balance payment attempt ends in exactly one clean outcome -- never a double charge and never a partial or ambiguous "balance due" state -- and resolves the case where a Pro cancellation and a client's balance payment race each other, per XBR-23.
- **FEAT-22.SPEC-005 — Balance Charge, Payout Routing & Refund** (integration): Authorizes and captures Riley's in-app balance card charge through the payment-processing capability, routes the captured balance to Talia's connected payout account with zero platform fee, and executes the outbound refund call when a paid balance must be refunded in full.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-021) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-22.SPEC-001 — 18 acceptance criteria (FEAT-22.SPEC-001-AC-01…FEAT-22.SPEC-001-AC-18) — docs/blueprint/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-001-balance-payment.md
- FEAT-22.SPEC-002 — 12 acceptance criteria (FEAT-22.SPEC-002-AC-01…FEAT-22.SPEC-002-AC-12) — docs/blueprint/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-002-balance-capture-booking-status-update.md
- FEAT-22.SPEC-003 — 15 acceptance criteria (FEAT-22.SPEC-003-AC-01…FEAT-22.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-003-balance-amount-eligibility-rules.md
- FEAT-22.SPEC-004 — 16 acceptance criteria (FEAT-22.SPEC-004-AC-01…FEAT-22.SPEC-004-AC-16) — docs/blueprint/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-004-balance-payment-outcome-consistency-cancellation-contention.md
- FEAT-22.SPEC-005 — 18 acceptance criteria (FEAT-22.SPEC-005-AC-01…FEAT-22.SPEC-005-AC-18) — docs/blueprint/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-005-balance-charge-payout-routing-refund.md

All 79 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 25 — Tipping at Checkout (FEAT-23)

Build feature FEAT-23 — Tipping at Checkout (Nice-to-Have). A client can optionally add a tip when paying in-app, passing through to the Pro with no platform fee.

**Already built (dependencies):** FEAT-22 (In-App Balance Payment)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-23-tipping-at-checkout/):

- **FEAT-23.SPEC-001 — Tip Selection** (screen): The Client is offered an optional, never-pre-selected tip amount as a step embedded inside the in-app balance payment flow, and can enter an amount or skip it without the underlying payment ever being blocked.
- **FEAT-23.SPEC-002 — Tip Amount Validation** (logic-rule): Governs the tip amount's own constraints on the Balance Payment record's `tip` field -- non-negative when given, valid as a monetary amount, and never pre-selected to a default -- independent of the balance amount's own rules, which FEAT-22 owns.
- **FEAT-23.SPEC-003 — Tip Payout & Refund Rule** (logic-rule): Governs where a validated tip's money goes once a balance payment succeeds -- the whole amount to the Pro's payout account with no platform cut -- and what must happen to it if the appointment is later cancelled: refunded in full together with the balance, never forfeited.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-23.SPEC-001 — 11 acceptance criteria (FEAT-23.SPEC-001-AC-01…FEAT-23.SPEC-001-AC-11) — docs/blueprint/specifications/FEAT-23-tipping-at-checkout/FEAT-23.SPEC-001-tip-selection.md
- FEAT-23.SPEC-002 — 9 acceptance criteria (FEAT-23.SPEC-002-AC-01…FEAT-23.SPEC-002-AC-09) — docs/blueprint/specifications/FEAT-23-tipping-at-checkout/FEAT-23.SPEC-002-tip-amount-validation.md
- FEAT-23.SPEC-003 — 10 acceptance criteria (FEAT-23.SPEC-003-AC-01…FEAT-23.SPEC-003-AC-10) — docs/blueprint/specifications/FEAT-23-tipping-at-checkout/FEAT-23.SPEC-003-tip-payout-refund-rule.md

All 30 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 26 — Client List Search & Filter (FEAT-24)

Build feature FEAT-24 — Client List Search & Filter (Nice-to-Have). The Pro can search and filter their client list by name, phone or recent activity.

**Already built (dependencies):** FEAT-13 (Client Record Management)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-24-client-list-search-filter/):

- **FEAT-24.SPEC-001 — Client Search & Filter** (screen): The Pro or Support views the Pro's full client list and narrows it by typing a partial name or phone number and/or selecting a recency or upcoming-booking filter, so a growing client base (100–500 clients) stays navigable instead of requiring an unfiltered scroll.
- **FEAT-24.SPEC-002 — Search Match & Filter Derivation Rules** (logic-rule): Governs how a search term matches a client's name or phone number, how the recency and upcoming-booking filter conditions are derived from a client's booking history, how search and an active filter combine, and the fallback behavior when the search/filter computation fails.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-011) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-24.SPEC-001 — 18 acceptance criteria (FEAT-24.SPEC-001-AC-01…FEAT-24.SPEC-001-AC-18) — docs/blueprint/specifications/FEAT-24-client-list-search-filter/FEAT-24.SPEC-001-client-search-filter.md
- FEAT-24.SPEC-002 — 15 acceptance criteria (FEAT-24.SPEC-002-AC-01…FEAT-24.SPEC-002-AC-15) — docs/blueprint/specifications/FEAT-24-client-list-search-filter/FEAT-24.SPEC-002-search-match-filter-derivation-rules.md

All 33 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 27 — Booking & Revenue Insights (FEAT-25)

Build feature FEAT-25 — Booking & Revenue Insights (Nice-to-Have). A simple summary for the Pro of booking volume, deposits collected and no-shows recovered over time — proof of value, not a full analytics suite.

**Already built (dependencies):** FEAT-07 (Deposit Payment at Booking), FEAT-11 (No-Show Marking & Deposit Forfeiture), FEAT-28 (Payout Account Connection & Payout Visibility)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-25-booking-revenue-insights/):

- **FEAT-25.SPEC-001 — Insights Summary Screen** (screen): A read-only period summary of the Pro's own booking volume, deposits collected, the no-show "saved" figure, most-booked services, and amounts received into her payout account, with a period selector and full Empty / Loading / Error / Offline-degraded coverage.
- **FEAT-25.SPEC-002 — Period Insights Aggregation** (automation): Computes the requested period's summary figures (total bookings, deposits collected, the "saved" figure, most-booked services, and amounts received) on view or period change, and retains the most recently successful result for reuse when a fresh computation fails or the Pro is offline.
- **FEAT-25.SPEC-003 — Insights Derivation, Period & Access Rules** (logic-rule): Defines the period bounds and data-sufficiency threshold, the "saved" and most-booked-service derivation formulas, and the own-figures-only / no-cross-pro-comparison access rule, once, for both FEAT-25.SPEC-001 and FEAT-25.SPEC-002 to reference.
- **FEAT-25.SPEC-004 — Historical Aggregate Maintenance** (automation): Maintains rolling per-period aggregates as bookings, deposit outcomes, no-show marks, and payout figures occur, so the summary stays responsive as a Pro's history grows across years.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-012) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-25.SPEC-001 — 24 acceptance criteria (FEAT-25.SPEC-001-AC-01…FEAT-25.SPEC-001-AC-24) — docs/blueprint/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-001-insights-summary-screen.md
- FEAT-25.SPEC-002 — 19 acceptance criteria (FEAT-25.SPEC-002-AC-01…FEAT-25.SPEC-002-AC-19) — docs/blueprint/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-002-period-insights-aggregation.md
- FEAT-25.SPEC-003 — 20 acceptance criteria (FEAT-25.SPEC-003-AC-01…FEAT-25.SPEC-003-AC-20) — docs/blueprint/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-003-insights-derivation-period-access-rules.md
- FEAT-25.SPEC-004 — 24 acceptance criteria (FEAT-25.SPEC-004-AC-01…FEAT-25.SPEC-004-AC-24) — docs/blueprint/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-004-historical-aggregate-maintenance.md

All 87 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 28 — WhatsApp Reminders (FEAT-26)

Build feature FEAT-26 — WhatsApp Reminders (Nice-to-Have). Confirmations and reminders can optionally go over WhatsApp instead of or alongside SMS, with fallback and consent rules.

**Already built (dependencies):** FEAT-08 (Automated Booking Messaging), FEAT-14 (Messaging Consent Management)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-26-whatsapp-reminders/):

- **FEAT-26.SPEC-001 — WhatsApp Channel Preference** (screen): Riley (the Client) opts for WhatsApp as her delivery channel for confirmations, reminders and change notices from her manage link, or switches back to text.
- **FEAT-26.SPEC-002 — WhatsApp Send & Delivery-Status Capability** (integration): Sends FEAT-08's confirmation, reminder and change-notice content over WhatsApp for clients whose channel is eligible, and reports back each message's delivery status.
- **FEAT-26.SPEC-003 — WhatsApp Delivery Fallback** (automation): When a WhatsApp send fails or the recipient's number is unreachable on WhatsApp, automatically falls back to text or email per the client's existing texting consent, and flags the delivery gap the way FEAT-08 already does for a failed text.
- **FEAT-26.SPEC-004 — WhatsApp Channel Eligibility & Consent Rule** (logic-rule): Decides, for every outbound confirmation, reminder or change notice, whether WhatsApp is used -- checking the client's channel preference against their channel-aware Messaging Consent state -- before handing the send to FEAT-26.SPEC-002 or deferring to FEAT-08.SPEC-011's text/email decision.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-009) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-26.SPEC-001 — 12 acceptance criteria (FEAT-26.SPEC-001-AC-01…FEAT-26.SPEC-001-AC-12) — docs/blueprint/specifications/FEAT-26-whatsapp-reminders/FEAT-26.SPEC-001-whatsapp-channel-preference.md
- FEAT-26.SPEC-002 — 12 acceptance criteria (FEAT-26.SPEC-002-AC-01…FEAT-26.SPEC-002-AC-12) — docs/blueprint/specifications/FEAT-26-whatsapp-reminders/FEAT-26.SPEC-002-whatsapp-send-delivery-status-capability.md
- FEAT-26.SPEC-003 — 12 acceptance criteria (FEAT-26.SPEC-003-AC-01…FEAT-26.SPEC-003-AC-12) — docs/blueprint/specifications/FEAT-26-whatsapp-reminders/FEAT-26.SPEC-003-whatsapp-delivery-fallback.md
- FEAT-26.SPEC-004 — 13 acceptance criteria (FEAT-26.SPEC-004-AC-01…FEAT-26.SPEC-004-AC-13) — docs/blueprint/specifications/FEAT-26-whatsapp-reminders/FEAT-26.SPEC-004-whatsapp-channel-eligibility-consent-rule.md

All 49 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 29 — Pro Booking Management (FEAT-30)

Build feature FEAT-30 — Pro Booking Management (Core). The Pro can cancel, reschedule or refund any of their own bookings, cancel several at once, grant goodwill refunds and book a client in, with the deposit handled fairly every time and the client told what happened.

**Already built (dependencies):** FEAT-03 (Real-Time Slot Availability Engine), FEAT-07 (Deposit Payment at Booking), FEAT-09 (Cancellation & No-Show Policy Engine), FEAT-28 (Payout Account Connection & Payout Visibility)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-30-pro-booking-management/):

- **FEAT-30.SPEC-001 — Cancel Booking (Pro-Initiated)** (screen): Talia views a single booking and confirms cancelling it, seeing plainly that the client's deposit will be refunded in full whatever the timing.
- **FEAT-30.SPEC-002 — Reschedule Booking (Pro-Initiated)** (screen): Talia picks a new genuinely free time for a client's booking, inside her own notice/horizon exception, seeing that the deposit carries over and a fresh manage link will go to the client.
- **FEAT-30.SPEC-003 — Goodwill Deposit Refund** (screen): Talia confirms a full goodwill refund on a booking's deposit, reached from a no-show prompt, a dispute timeline, or a booking row, available any time before the booking completes.
- **FEAT-30.SPEC-004 — Book Client In** (screen): Talia chooses a service, a time, and an existing or new client to book the client in directly, then issues a held deposit request (link or on-screen scan code) instead of collecting payment inside this screen.
- **FEAT-30.SPEC-005 — Cancel Several Bookings at Once** (screen): Talia reviews the set of bookings a new time block conflicts with and confirms cancelling them together, seeing the full-refund outcome for each before confirming.
- **FEAT-30.SPEC-006 — Pro Booking Action Rules** (logic-rule): Governs the eligibility, ownership, and limits shared by every Pro-initiated booking action -- cancel, reschedule, goodwill refund, book-client-in, and bulk cancel -- so each screen and automation in this feature references one authoritative set of rules instead of restating them.
- **FEAT-30.SPEC-007 — Pro Cancel/Reschedule Commit** (automation): Commits a Pro-initiated single cancellation or reschedule to the Booking, coordinating the deposit-outcome handoff, calendar mirroring, activity logging, freed-slot handoff, and client notice this triggers.
- **FEAT-30.SPEC-008 — Bulk Cancellation Commit** (automation): Commits a Pro-initiated cancellation of several bookings at once, reporting a per-booking outcome and coordinating each booking's refund, calendar removal, and client notice independently.
- **FEAT-30.SPEC-009 — Goodwill Refund Commit** (automation): Processes a confirmed goodwill refund against a booking's deposit -- a standalone Pro override, independent of the cancellation window and available until the booking completes.
- **FEAT-30.SPEC-010 — Pro-Created Booking & Deposit Request Hold** (automation): Creates the pending Booking from a Pro-entered service, time, and client, invokes the slot hold that reserves the time, confirms the booking on payment, and reflects the hold's expiry if the deposit is never paid.
- **FEAT-30.SPEC-011 — Goodwill & Bulk-Cancellation Refund Execution** (integration): Requests each goodwill or bulk-cancellation refund from the payment-processing capability, drawing on the Pro's connected payout account, guarantees each refund completes exactly once with automatic retry, and reports back any refund that cannot complete immediately.
- **FEAT-30.SPEC-012 — Pro Action Client Notice** (notification): The trigger-and-audience contract that ensures the client is told when the Pro cancelled their booking (with refund status), rescheduled it (with the new time and a fresh manage link), or issued a goodwill refund -- so no client is ever left wondering whether a Pro-initiated change went through or what happened to their deposit. The message content is owned by FEAT-08.SPEC-004; this spec defines when it fires, for whom, and what data this feature supplies.
- **FEAT-30.SPEC-013 — Deposit Request & Expiry Notice** (notification): Delivers a Pro-created deposit request to the client -- by text, by email, or as an on-screen code to scan -- and, when an unpaid request expires, stops any pending delivery for it. The Pro's expiry notice is not defined here: FEAT-03.SPEC-007 is its trigger and FEAT-08.SPEC-006 is its content owner.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-010, ADR-012, ADR-021) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-30.SPEC-001 — 16 acceptance criteria (FEAT-30.SPEC-001-AC-01…FEAT-30.SPEC-001-AC-16) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-001-cancel-booking-pro-initiated.md
- FEAT-30.SPEC-002 — 14 acceptance criteria (FEAT-30.SPEC-002-AC-01…FEAT-30.SPEC-002-AC-14) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-002-reschedule-booking-pro-initiated.md
- FEAT-30.SPEC-003 — 15 acceptance criteria (FEAT-30.SPEC-003-AC-01…FEAT-30.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-003-goodwill-deposit-refund.md
- FEAT-30.SPEC-004 — 16 acceptance criteria (FEAT-30.SPEC-004-AC-01…FEAT-30.SPEC-004-AC-16) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-004-book-client-in.md
- FEAT-30.SPEC-005 — 15 acceptance criteria (FEAT-30.SPEC-005-AC-01…FEAT-30.SPEC-005-AC-15) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-005-cancel-several-bookings-at-once.md
- FEAT-30.SPEC-006 — 19 acceptance criteria (FEAT-30.SPEC-006-AC-01…FEAT-30.SPEC-006-AC-19) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-006-pro-booking-action-rules.md
- FEAT-30.SPEC-007 — 16 acceptance criteria (FEAT-30.SPEC-007-AC-01…FEAT-30.SPEC-007-AC-16) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md
- FEAT-30.SPEC-008 — 13 acceptance criteria (FEAT-30.SPEC-008-AC-01…FEAT-30.SPEC-008-AC-13) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md
- FEAT-30.SPEC-009 — 12 acceptance criteria (FEAT-30.SPEC-009-AC-01…FEAT-30.SPEC-009-AC-12) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-009-goodwill-refund-commit.md
- FEAT-30.SPEC-010 — 14 acceptance criteria (FEAT-30.SPEC-010-AC-01…FEAT-30.SPEC-010-AC-14) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md
- FEAT-30.SPEC-011 — 14 acceptance criteria (FEAT-30.SPEC-011-AC-01…FEAT-30.SPEC-011-AC-14) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-011-goodwill-bulk-cancellation-refund-execution.md
- FEAT-30.SPEC-012 — 13 acceptance criteria (FEAT-30.SPEC-012-AC-01…FEAT-30.SPEC-012-AC-13) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-012-pro-action-client-notice.md
- FEAT-30.SPEC-013 — 13 acceptance criteria (FEAT-30.SPEC-013-AC-01…FEAT-30.SPEC-013-AC-13) — docs/blueprint/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-013-deposit-request-expiry-notice.md

All 190 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

---

## Prompt 30 — Manual Time Blocking (FEAT-17)

Build feature FEAT-17 — Manual Time Blocking (Important). The Pro can block off spans of time — one-off or recurring — removing them from bookable availability, with conflict review and resolution against existing bookings.

**Already built (dependencies):** FEAT-30 (Pro Booking Management)

**Build these specifications** (full text under docs/blueprint/specifications/FEAT-17-manual-time-blocking/):

- **FEAT-17.SPEC-001 — Create/Edit Time Block** (screen): Talia sets a span of time on a specific date, or a recurring weekly pattern, with an optional private label, to create a new Time Block or edit an existing one.
- **FEAT-17.SPEC-002 — Manage Time Blocks** (screen): Talia (Full) and Platform Operator Support (View-only) see the list of upcoming Time Blocks, with a plain empty state when none exist and an entry point to edit or remove each one.
- **FEAT-17.SPEC-003 — Time Block Conflict Review** (screen): Talia sees every confirmed booking a new or edited Time Block conflicts with and chooses, per booking, to cancel it, reschedule it, or keep the block with that booking as an exception, before the block commits.
- **FEAT-17.SPEC-004 — Time Block Save Commit & Conflict Detection** (automation): Validates and commits a created or edited Time Block, checking it against existing confirmed bookings and routing to the Conflict Review screen when any are found.
- **FEAT-17.SPEC-005 — Recurring Time Block Occurrence Generation** (automation): Generates and maintains the future dated occurrences of a recurring block pattern (e.g., every Sunday), running each new occurrence through the same conflict detection as a single-date block.
- **FEAT-17.SPEC-006 — Time Block Conflict Resolution Commit** (automation): Commits Talia's explicit choice on a conflicting booking set -- hand off to cancellation, hand off to reschedule, or mark the booking as a kept exception -- and finalizes the block once every conflicting booking has a resolved outcome.
- **FEAT-17.SPEC-007 — Time Block Removal & Expiry** (automation): Deletes a block Talia removes early, or automatically retires a block once its end time has passed, in either case restoring that time to bookable availability immediately.
- **FEAT-17.SPEC-008 — Time Block Validation & Conflict Handling Rules** (logic-rule): Defines the shared rules every screen and automation in this feature references: end-after-start validation, what counts as a conflicting booking, the never-silently-affect-a-booking rule, and who can see or act on a block.

**Architecture rules:** follow the binding recommended architecture in the project knowledge (ADR-003, ADR-012) — never substitute a documented alternative, and touch nothing on the DO-NOT-BUILD list.

**Definition of done:**

- FEAT-17.SPEC-001 — 17 acceptance criteria (FEAT-17.SPEC-001-AC-01…FEAT-17.SPEC-001-AC-17) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-001-create-edit-time-block.md
- FEAT-17.SPEC-002 — 14 acceptance criteria (FEAT-17.SPEC-002-AC-01…FEAT-17.SPEC-002-AC-14) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-002-manage-time-blocks.md
- FEAT-17.SPEC-003 — 15 acceptance criteria (FEAT-17.SPEC-003-AC-01…FEAT-17.SPEC-003-AC-15) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-003-time-block-conflict-review.md
- FEAT-17.SPEC-004 — 12 acceptance criteria (FEAT-17.SPEC-004-AC-01…FEAT-17.SPEC-004-AC-12) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-004-time-block-save-commit-conflict-detection.md
- FEAT-17.SPEC-005 — 12 acceptance criteria (FEAT-17.SPEC-005-AC-01…FEAT-17.SPEC-005-AC-12) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-005-recurring-time-block-occurrence-generation.md
- FEAT-17.SPEC-006 — 13 acceptance criteria (FEAT-17.SPEC-006-AC-01…FEAT-17.SPEC-006-AC-13) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-006-time-block-conflict-resolution-commit.md
- FEAT-17.SPEC-007 — 11 acceptance criteria (FEAT-17.SPEC-007-AC-01…FEAT-17.SPEC-007-AC-11) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-007-time-block-removal-expiry.md
- FEAT-17.SPEC-008 — 20 acceptance criteria (FEAT-17.SPEC-008-AC-01…FEAT-17.SPEC-008-AC-20) — docs/blueprint/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-008-time-block-validation-conflict-handling-rules.md

All 114 acceptance criteria above must pass end-to-end — verify them as a user would before calling this feature done.

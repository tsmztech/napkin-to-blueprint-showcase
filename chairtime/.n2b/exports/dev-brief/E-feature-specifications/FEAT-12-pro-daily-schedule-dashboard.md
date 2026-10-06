# FEAT-12 — Pro Daily Schedule Dashboard

This chapter covers Pro Daily Schedule Dashboard (FEAT-12), a Core-tier feature. It carries 8 specifications carrying 112 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | screen | 19 |
| FEAT-12.SPEC-002 | Attention List | screen | 15 |
| FEAT-12.SPEC-003 | Past Bookings Browse | screen | 15 |
| FEAT-12.SPEC-004 | Auto-Completion Sweep | automation | 9 |
| FEAT-12.SPEC-005 | Attention Flag Aggregation | automation | 12 |
| FEAT-12.SPEC-006 | Booking Completion Rules | logic-rule | 14 |
| FEAT-12.SPEC-007 | Balance Due & Status Display Rules | logic-rule | 15 |
| FEAT-12.SPEC-008 | Dashboard Access Authorization | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Pro Daily Schedule Dashboard

## Summary

**Feature:** Pro Daily Schedule Dashboard
**ID:** FEAT-12
**Description:** The Pro's primary, phone-first view: today's (and upcoming) bookings, each with a paid badge, a client note, and how much balance is still due in person — the screen the Pro glances at between clients.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Vision describes this exactly: "As the pro, you glance at your phone between clients: today's list, each booking with a paid badge, a client note, and how much is still due in person." This is the Pro's single most frequent touchpoint with the product.

**Key Capabilities:**
- View today's bookings at a glance, in time order, with paid/unpaid and balance-due status
- View upcoming bookings beyond today
- Take quick actions directly from the list: mark no-show, view client note, jump to reschedule/cancel
- Mark a finished appointment as completed (recording the balance as settled in person), or let it complete automatically
- See which clients tapped "I'll be there", and an attention list of anything needing action (sync issue, message delivery failure, refund failure, card-issuer dispute, bookings left outside changed hours)
- Browse past bookings by date

**Connected Entities:** Booking (read, update — quick actions and completion), Client (read), Deposit Transaction (read), Message (read — delivery flags), plus Time Block, Calendar Connection, Messaging Consent and Waitlist Entry (all read-only per the dependency map slice).

**Access (from the Access Matrix):** The Pro has Full access to their own schedule only. Clients have no access to this view (they see only their own bookings, through FEAT-06); anyone not signed in as the Pro is sent to the Pro sign-in screen (FEAT-29). Platform Operator (Support) has View-only access for troubleshooting a specific reported issue, and never sees the Pro's private client notes.

**Communications:** N/A — this is a viewing surface; it does not itself send messages. (No Communications lines to elaborate into Notification specs; `notification_count: 0` is legal and expected here.)

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | Screen | The Pro, Platform Operator (Support) | The Pro's main dashboard: today's remaining bookings in time order plus upcoming bookings, each with paid badge, balance due, "I'll be there" status, sync-reliability marking, and quick-action entry points |
| FEAT-12.SPEC-002 | Attention List | Screen | The Pro, Platform Operator (Support) | Surfaces everything needing the Pro's attention — sync issues, message delivery failures, refunds in progress, card-issuer disputes, bookings left outside changed hours, and waitlist demand |
| FEAT-12.SPEC-003 | Past Bookings Browse | Screen | The Pro, Platform Operator (Support) | The Pro finds and reviews a past booking by browsing by date |
| FEAT-12.SPEC-004 | Auto-Completion Sweep | Automation | The Pro | Automatically marks a booking Completed 7 days after its appointment if the Pro never marked it completed or no-show |
| FEAT-12.SPEC-005 | Attention Flag Aggregation | Automation | The Pro | Gathers and de-duplicates attention-worthy signals from other features' owned states (calendar sync health, message delivery, refund progress, disputes, setup-change conflicts, waitlist demand) into a single Attention List feed, and tracks resolution |
| FEAT-12.SPEC-006 | Booking Completion Rules | Logic/Rule | The Pro | Governs when a booking may be marked Completed, when it auto-completes, and how completion interacts with cancellation/reschedule eligibility (XBR-12) |
| FEAT-12.SPEC-007 | Balance Due & Status Display Rules | Logic/Rule | The Pro, Platform Operator (Support) | Derives the balance-due amount and the paid/unpaid, "I'll be there," and sync-reliability display states shown consistently across all three screens |
| FEAT-12.SPEC-008 | Dashboard Access Authorization | Logic/Rule | The Pro, Platform Operator (Support) | Enforces who may open this dashboard and what each role sees: the Pro's own schedule only, Support's masked read-only view (never private notes, sign-in codes, or card/bank details), and redirect-to-sign-in for anyone else (XBR-29) |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View today's bookings at a glance, in time order, with paid/unpaid and balance-due status | FEAT-12.SPEC-001, FEAT-12.SPEC-007 | Primary layout of the schedule screen; balance/paid states computed by the display-rules spec | Phase 2 (Explicit) |
| View upcoming bookings beyond today | FEAT-12.SPEC-001 | Same screen, upcoming section/view beyond today's date | Phase 2 (Explicit) |
| Take quick actions directly from the list: mark no-show, view client note, jump to reschedule/cancel | FEAT-12.SPEC-001 | Inline per-row action affordances that route to the owning feature (FEAT-11, FEAT-13, FEAT-30) | Phase 2 (Explicit) |
| Mark a finished appointment as completed, or let it complete automatically | FEAT-12.SPEC-001, FEAT-12.SPEC-004, FEAT-12.SPEC-006 | Inline "mark completed" action on the schedule screen, validated by the completion rules spec; unmarked bookings are swept by the auto-completion automation | Phase 2 (Explicit) + Phase 4 (Trigger-Response, for auto-completion) |
| See which clients tapped "I'll be there," and an attention list of anything needing action | FEAT-12.SPEC-001 ("I'll be there"), FEAT-12.SPEC-002, FEAT-12.SPEC-005 | Per-row attendance-reply display; dedicated Attention List screen fed by the aggregation automation | Phase 2 (Explicit) + Phase 4 (Trigger-Response / External Dependencies lens) |
| Browse past bookings by date | FEAT-12.SPEC-003 | Dedicated past-bookings screen with date browsing | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-12.SPEC-006 | Booking Completion Rules | Phase 5 (Rule-Constraint Discovery) | The feature's Validation & Limits field states three conditional rules governing one state transition (only after start time, 7-day auto-complete, interaction with cancel/reschedule eligibility per XBR-12) — this is conditional logic shared across Phase 2's inline action and Phase 4's automation, crossing the standalone-spec threshold |
| FEAT-12.SPEC-007 | Balance Due & Status Display Rules | Phase 5 (Rule-Constraint Discovery) | Balance due is a non-trivial derivation (price − deposit − any in-app balance payment, per XBR-23) reused identically across SPEC-001 and SPEC-003; consolidating avoids restating the formula and its badge/status conventions in two Screen specs |
| FEAT-12.SPEC-008 | Dashboard Access Authorization | Phase 5 (Rule-Constraint Discovery) | The Access field describes materially different behavior per role (Pro: Full/own-only; Support: View-only with masked private notes; everyone else: redirect to sign-in per XBR-29) — an authorization rule set shared by all three screens |
| FEAT-12.SPEC-005 | Attention Flag Aggregation | Phase 4 (Trigger-Response / External Dependencies lens) | The Access field and assumptions-constraints.md's Dependencies slice both point at signals owned by other features' external-capability integrations (calendar sync health from FEAT-04/ASMP-33, message delivery from FEAT-08/ASMP-32, disputes from FEAT-16/ASMP-31, refund progress from FEAT-09); this feature does not own any of those capabilities but must collect, de-duplicate, and track resolution of what they report — a cross-feature aggregation behavior, not a bare display row |

**Note on Integration specs (`integration_count: 0`):** The context package's own Non-Functional and dependency slice states this explicitly: "whether it needs its own Integration spec for the dispute-notice row is the Analyst's decomposition call — the dependency map expects that row to be specified in the batch covering FEAT-16." Having reviewed all three category-level external capabilities in the Dependencies slice (ASMP-31 payment processing, ASMP-32 texting, ASMP-33 calendar sync), none is owned by FEAT-12: FEAT-12 is a pure consumer of outcomes that FEAT-04, FEAT-08, FEAT-09/FEAT-28, and FEAT-16 each already own via their own Integration specs. `integration_count: 0` is therefore correct, not a coverage gap.

## Entity-Lifecycle Coverage Matrix

**Entity: Booking**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-05, FEAT-30, and FEAT-21 per the dependency map — this feature never creates a Booking | -- |
| Read (single) | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003 | Booking detail expands inline on the schedule row, the attention item, and the past-bookings row | -- |
| Read (list) | FEAT-12.SPEC-001, FEAT-12.SPEC-003 | Today/upcoming list and the past-bookings-by-date list | -- |
| Update | FEAT-12.SPEC-001, FEAT-12.SPEC-006 | Mark-completed action on the schedule screen, validated against the Completion Rules spec | This feature's only Update ownership on Booking is the completion mark, per the dependency map's Updated-by list |
| Delete/Archive | N/A | Bookings are never deleted by any feature — kept for the life of the account per scope-boundaries.md SC-22 (multi-year history retained for dispute evidence and insights); cancelled/expired/no-show/completed bookings remain as permanent history, not archived state this feature manages | Intentional non-goal, not an omission — see Non-Goals |
| State Transition | FEAT-12.SPEC-001, FEAT-12.SPEC-004, FEAT-12.SPEC-006 | Pro-initiated → Completed (SPEC-001, validated by SPEC-006); time-based → Completed after 7 days with no Pro action (SPEC-004, the Auto-Completion Sweep, per XBR-12 which names FEAT-12 as the owner of completion and auto-completion) | Every other Booking state transition (Confirmed, No-Show, Cancelled, Rescheduled, Expired) is owned by FEAT-07, FEAT-10, FEAT-11, or FEAT-30 and only read/displayed here |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-12.SPEC-001, FEAT-12.SPEC-003 | Client name and private note preview shown on each booking row; full record and edit belong to FEAT-13 |
| Deposit Transaction | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003, FEAT-12.SPEC-007 | Paid status and any forfeit/refund/dispute outcome shown per booking; refund-in-progress and disputed states feed the Attention List |
| Message | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Delivery-failure flags surfaced per booking row and in the Attention List (XBR-17) |
| Time Block | FEAT-12.SPEC-001 | Manual time blocks shown on the schedule alongside bookings so a blocked period reads as occupied, not empty |
| Calendar Connection | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Sync-health status drives the per-booking reliability marking and the reconnect attention item |
| Messaging Consent | FEAT-12.SPEC-001 | Pro sees whether a client is reachable by text (textability), never able to override consent |
| Waitlist Entry | FEAT-12.SPEC-002 | Aggregate waitlist demand shown as an informational item; the Pro sees counts only, never individual entries (Access Matrix: Waitlist = View for the Pro) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro opens the dashboard | Load today's remaining bookings, upcoming bookings, and per-row status (paid, balance due, attendance reply, sync reliability) | Inline in triggering screen | FEAT-12.SPEC-001 |
| Pro taps "mark completed" on a booking | Validate the booking is past its start time and not already Completed/No-Show/Cancelled; if valid, transition Booking to Completed and record balance settled in person | Standalone Logic/Rule (validation) + inline write in triggering screen | FEAT-12.SPEC-006 / FEAT-12.SPEC-001 |
| 7 days elapse after a booking's appointment time with no Pro mark | Auto-transition Booking to Completed | Standalone Automation | FEAT-12.SPEC-004 |
| Calendar sync health changes to Needs Reconnection, a message delivery fails, a refund cannot complete, a dispute notice arrives, or a setup change conflicts with an existing booking | Add or update an item on the Attention List; de-duplicate repeat signals for the same booking/cause | Standalone Automation | FEAT-12.SPEC-005 |
| Pro taps an Attention List item (reconnect, money action, dispute) | Navigate to the owning feature's screen (FEAT-04, FEAT-28, FEAT-16) | Cross-feature | Owning feature's responsibility; FEAT-12.SPEC-002 initiates navigation only |
| Pro taps "no-show" on a booking row | Navigate to no-show marking/undo | Cross-feature | FEAT-11 responsibility |
| Pro taps a booking's reschedule/cancel/rebook action | Navigate to Pro Booking Management | Cross-feature | FEAT-30 responsibility |
| Pro taps a client on a booking row | Navigate to the client record and private note | Cross-feature | FEAT-13 responsibility |
| Pro taps "block time" from the schedule view | Navigate to manual time blocking | Cross-feature | FEAT-17 responsibility |
| Pro opens a past booking | Navigate to its full activity timeline | Cross-feature | FEAT-16 responsibility |
| Booking is displayed anywhere on this feature's screens | Compute balance due and status badges from underlying entity data | Standalone Logic/Rule | FEAT-12.SPEC-007 |
| Any of this feature's three screens is opened | Verify the signed-in identity's role and apply the correct view (Pro full, Support masked-view-only, anyone else redirected) | Standalone Logic/Rule | FEAT-12.SPEC-008 |
| A booking's calendar-sync status is uncertain | Show that specific booking's reliability as uncertain rather than presenting it with false confidence | Inline in triggering screen, sourced from FEAT-12.SPEC-005's aggregation | FEAT-12.SPEC-001 |
| Empty day (zero bookings) | Show a friendly "nothing booked yet today" state with a shortcut to share the booking link | Inline in triggering screen | FEAT-12.SPEC-001 |

## Shared Context

**Shared Entities:**
- Booking -- read by all three screens (SPEC-001, SPEC-002, SPEC-003); updated only by the completion mark (SPEC-001, validated by SPEC-006) and the auto-completion sweep (SPEC-004). Fields relevant to this feature: service, start_time, client, price_agreed, deposit_amount, state, attendance_reply, balance_due, cancellation/reschedule timestamps.
- Deposit Transaction -- read by SPEC-001, SPEC-002, SPEC-003, and consumed by SPEC-007's balance-due derivation. Fields relevant here: status (including Refund in Progress and Disputed), amount, currency.
- Client -- read-only across SPEC-001 and SPEC-003 for name and private_note preview; full edit stays in FEAT-13.

**Shared UI Patterns:**
- Booking row -- the same visual and interaction pattern (paid badge, balance due, attendance reply, sync-reliability marking, quick-action affordances) appears on SPEC-001's today/upcoming list and SPEC-003's past-bookings list; both Spec Writers should describe it once, consistently, referencing FEAT-12.SPEC-007 for the derived values shown.
- Attention item -- the same card pattern (cause, affected booking or account, action button) is used for every attention source aggregated by SPEC-005 and rendered by SPEC-002.

**Shared Validation:**
- FEAT-12.SPEC-006 defines the completion-eligibility window; FEAT-12.SPEC-001's mark-completed action and FEAT-12.SPEC-004's auto-completion sweep both reference it rather than restating the rule.
- FEAT-12.SPEC-008 defines role-based view rules; all three screens reference it rather than each re-describing what Support sees versus the Pro.

**Discrepancy flagged, not resolved (per methodology -- Interactions consistency check):** The feature's own Interactions field in product-features.md lists this feature as depending on FEAT-05, FEAT-07, FEAT-08, and FEAT-13, but the Feature Dependency Map's Features table row lists FEAT-12's "Depends On" as FEAT-05, FEAT-07, FEAT-08, FEAT-13, **and FEAT-29** (Pro sign-in). The dependency map's own Navigation Connections row ("FEAT-29 sign-in → FEAT-12 today's schedule, trigger: Pro signs in / opens the app") and Cross-Feature Business Rule XBR-29 ("every Pro-facing screen requires a signed-in Pro... FEAT-12" listed among affected features) both corroborate the dependency-map version. This Brief treats FEAT-29 as a genuine dependency (reflected in the Cross-Feature Touchpoints table and FEAT-12.SPEC-008) and flags the product-features.md Interactions field's omission for the Requirements Architect to reconcile — it is not this Analyst's decision to resolve.

**Downstream reader noted, not owned here:** The dependency map's Features table shows FEAT-19 (Booking & Payment Activity Record) as "Depended On By" this feature. FEAT-19 reads the Booking entity this feature also reads and updates; no direct screen-to-screen touchpoint exists between FEAT-12 and FEAT-19 in the Navigation Connections table, so no Cross-Feature Touchpoints row is created for it — the relationship is entity-level only.

## Internal Dependency Map

```
SPEC-001 (Today's & Upcoming Schedule) -> [Pro taps a booking row] -> inline expand: client note preview + quick actions
SPEC-001 -> [Pro taps "mark completed"] -> validated by SPEC-006 (Booking Completion Rules) -> Booking updated to Completed
SPEC-004 (Auto-Completion Sweep) -> [7 days elapse, no Pro action] -> validated by SPEC-006 -> Booking updated to Completed -> reflected in SPEC-001 / SPEC-003
SPEC-001 -> [renders paid badge / balance due / "I'll be there" / reliability marking using] -> SPEC-007 (Balance Due & Status Display Rules)
SPEC-003 (Past Bookings Browse) -> [renders the same booking row using] -> SPEC-007
SPEC-005 (Attention Flag Aggregation) -> [feeds] -> SPEC-002 (Attention List)
SPEC-001 -> [Pro taps the attention banner] -> SPEC-002
SPEC-001 -> [Pro navigates to] -> SPEC-003
SPEC-001 / SPEC-002 / SPEC-003 -> [each checked against on open] -> SPEC-008 (Dashboard Access Authorization)
```

**Default Entry:** SPEC-001 (Today's & Upcoming Schedule) -- the screen shown when the Pro signs in and opens the app (FEAT-29).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-12.SPEC-001 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Dashboard is the entry point after sign-in | Pro signs in / opens the app |
| FEAT-12.SPEC-001 | Outbound | FEAT-13 (Client Record Management) | Client record and private note | Pro taps a client on a booking row |
| FEAT-12.SPEC-001 | Outbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Mark no-show / undo | Pro taps "no-show" |
| FEAT-12.SPEC-001 | Outbound | FEAT-30 (Pro Booking Management) | Cancel, reschedule, refund, book next visit | Pro taps a booking action |
| FEAT-12.SPEC-001 | Outbound | FEAT-17 (Manual Time Blocking) | Add a time block | Pro taps "block time" |
| FEAT-12.SPEC-003 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Booking activity timeline | Pro opens a past booking's history |
| FEAT-12.SPEC-002 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Reconnect calendar | Pro taps the reconnect banner |
| FEAT-12.SPEC-002 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Refund in progress / payout action required | Pro taps a money attention item |
| FEAT-12.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Card-issuer dispute flag and summary download | Pro taps a dispute flag |
| FEAT-12.SPEC-001 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Money list | Pro opens the money list from navigation |
| FEAT-12.SPEC-001 | Outbound | FEAT-27 (Pro Profile & Booking Page Settings) | Profile and booking page settings | Pro opens settings from navigation |
| FEAT-12.SPEC-001 | Outbound | FEAT-25 (Booking & Revenue Insights) | Insights (v1) | Pro opens insights from navigation |
| FEAT-12.SPEC-005 | Inbound | FEAT-04 (Two-Way Calendar Sync) | Calendar sync health feeds an attention flag | Sync status changes (XBR-13) |
| FEAT-12.SPEC-005 | Inbound | FEAT-08 (Automated Booking Messaging) | Message delivery failure feeds an attention flag | Text retried then falls back to email, gap flagged (XBR-17) |
| FEAT-12.SPEC-005 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Refund-in-progress feeds an attention flag | An automatic refund cannot complete immediately (XBR-10) |
| FEAT-12.SPEC-005 | Inbound | FEAT-16 (Booking & Payment Activity Record) | Card-issuer dispute feeds an attention flag | Dispute notice received from payment processing (XBR-22) |
| FEAT-12.SPEC-005 | Inbound | FEAT-01, FEAT-02, FEAT-17, FEAT-18, FEAT-27 (Setup features) | Bookings left outside changed hours feed an attention flag | A setup change (hours, blocks, archived service, pause, subscription lapse) conflicts with an existing booking (XBR-11) |
| FEAT-12.SPEC-002 | Referenced | FEAT-20 (Waitlist for Cancelled Slots) | Aggregate waitlist demand shown as an informational item (v1) | Pro opens the Attention List / dashboard |

## Non-Functional Notes

**Data volumes / growth:** A few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week (ASMP-22); the Past Bookings Browse screen (SPEC-003) must stay equally responsive as a Pro's booking history grows over multiple years, since Booking history is retained for the life of the account (scope-boundaries.md SC-22).

**Responsiveness:** Success-metrics.md's Daily Dashboard Glance Speed target: a Pro can identify their next booking's status (paid/unpaid, balance due) within a few seconds of opening the dashboard, with no extra navigation required — this is the product's single most frequent touchpoint and defines its perceived speed (BRIEF.md's Vision). Loading states (ASMP-27) must never block this glance with a blank screen; a lightweight in-place indicator is required on slow connections.

**Data sensitivity / privacy:** This feature surfaces personal data linked to identifiable clients — appointment time, service, and the Pro's private client note preview — visible only to the Pro (ASMP-23). Platform Operator (Support) sees a read-only, masked view: never the Pro's private client notes, sign-in codes, card data, or bank/identity details (user-persona.md Access Matrix; ASMP-20, ASMP-30). Calendar Connection data shown here is limited to sync health only, never event titles or details (product-features.md FEAT-04 Data Notes).

**Compliance flags:** Messaging Consent is displayed as textability status only — the Pro can see it but never override a client's consent (US texting-consent rules, ASMP-24). No card data is ever displayed on this feature's screens; card data is owned entirely by payment processing (ASMP-15, BRIEF.md Constraints). Every screen in this feature must remain readable and fully operable at phone width inside the Instagram in-app browser, with paid/attention status never conveyed by color alone — for example, the paid badge also carries a word (ASMP-28 Accessibility baseline).

**Analytics linkage (from Signals):** dashboard_viewed (SPEC-001 open), quick_action_taken with action type (any inline action on SPEC-001), booking_marked_completed (SPEC-001/SPEC-006), booking_auto_completed (SPEC-004), attention_item_resolved (SPEC-002/SPEC-005). These signals are the feature's contribution to the Daily Dashboard Glance Speed and Pro Change Correctness success metrics.

**Offline / degraded posture:** Per ASMP-27 and this feature's own States field, the most recently loaded schedule remains viewable read-only when offline; actions like marking no-show or completed require reconnecting. This is annotated on SPEC-001 and SPEC-003 rather than restated as a separate spec.

## Non-Goals

- **Balance payment taken in-app** -- Excluded at MVP per scope-boundaries.md SC-16: the balance segment is deliberately settled in person, off-platform, at MVP; this feature only records that a booking was completed with the balance settled in person. In-App Balance Payment (FEAT-22) is a deferred v1 enhancement this feature will read from once it exists (per XBR-23), but does not implement now.
- **Chairtime adjudicating a card-issuer dispute** -- Excluded per scope-boundaries.md SC-17: this feature flags a dispute on the dashboard and routes to the evidence-download flow (FEAT-16), but never rules on who is right; that is owned by the payment processor's dispute process.
- **Computing or displaying revenue/booking insights** -- The Booking & Revenue Insights feature is deferred to v1 per scope-boundaries.md's deferral notes ("deferred until pros have enough booking history for a summary to be meaningful"); this feature only provides a navigation entry point (FEAT-25) and does not compute or render insight metrics itself.
- **Per-staff or multi-chair schedule views** -- Excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so this feature shows exactly one Pro's own schedule with no staff-filtering or salon-wide view.
- **Client-facing access to this dashboard** -- Per the Access Matrix, Clients have no access to this view at all; they see only their own bookings through Client Booking Identity (FEAT-06). Building any client-visible variant of this screen would violate BRIEF.md's Privacy constraint that a client's data is never visible to any other pro or client, and would blur the single-purpose-per-role boundary the persona set establishes.
- **Deleting or archiving Booking records from this feature** -- Intentional lifecycle decision per scope-boundaries.md SC-22: a Pro's full booking history is retained for the life of the account for dispute evidence and insights; this feature never offers a delete or archive action on a Booking, and no retention window applies to it here.
- **Support acting on a Pro's behalf from this dashboard** -- Excluded per scope-boundaries.md SC-05: Support's access here is strictly View-only for troubleshooting a reported issue; it cannot mark completions, no-shows, or any other write action from this screen, and cannot sign in as the Pro.



# Screen Spec: Today's & Upcoming Schedule

## Overview

**Name:** Today's & Upcoming Schedule
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** The Pro's main dashboard: today's remaining bookings in time order plus upcoming bookings beyond today, each with a paid badge, balance due, "I'll be there" status, sync-reliability marking, and quick-action entry points.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying today's remaining bookings in time order and upcoming bookings beyond today
- Showing manual time blocks (FEAT-17) alongside bookings so blocked periods read as occupied
- Per-row paid badge, balance due, "I'll be there" status, and sync-reliability marking (derived by FEAT-12.SPEC-007)
- Quick-action entry points: view client note, mark no-show, jump to reschedule/cancel, mark completed
- Navigating to the Attention List (FEAT-12.SPEC-002), Past Bookings Browse (FEAT-12.SPEC-003), and cross-feature destinations reachable from the dashboard's navigation
- The empty-day state and the loading/offline/error states for this screen

**Non-Goals:**
- Displaying the aggregated Attention List itself -- owned by FEAT-12.SPEC-002; this screen only shows a summary banner that links to it
- Browsing past bookings by date -- owned by FEAT-12.SPEC-003
- Performing the no-show mark, cancel, reschedule, or refund actions themselves -- owned by FEAT-11 and FEAT-30 respectively; this screen only provides the entry point and navigates there
- Taking an in-app balance payment -- excluded per scope-boundaries.md SC-16: the balance is deliberately settled in person at MVP; this screen only offers the mark-completed action, which records the balance as settled in person, never a payment flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-001 (Sign-In Screen) / FEAT-29.SPEC-002 (Account Recovery Screen) (Pro Sign-In & Account Lifecycle) | Pro signs in / opens the app | None -- this is the default landing screen after sign-in |
| FEAT-12.SPEC-002 (Attention List) | Pro navigates back from the Attention List | None -- schedule reloads current data |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Pro navigates back from past bookings | None -- schedule reloads current data |
| FEAT-13 (Client Record Management) | Pro returns after viewing/editing a client record | None -- schedule reloads current data |
| FEAT-30 (Pro Booking Management), FEAT-11 (No-Show Marking), FEAT-17 (Manual Time Blocking) | Pro completes or cancels a cross-feature action and returns | None -- schedule reloads to reflect any change |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own schedule only | All quick actions and navigation | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review, except the client private-note preview, which is omitted entirely (FEAT-12.SPEC-008) | View only -- no quick actions, no mark-completed control | Any write control is simply not present; there is no denial dialog because no write path is ever rendered for Support |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29); Clients see only their own bookings through FEAT-06, never this screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen to preserve, since it is a viewing surface with no form state |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title showing the current date (e.g., "Today"), with an Attention banner directly below the title when one or more Open Attention Items exist (FEAT-12.SPEC-005) -- the banner states the count and a short label (e.g., "3 things need your attention") and is tappable. Navigation to Settings (FEAT-27), Money (FEAT-28), and Insights (FEAT-25) is reachable from a persistent navigation affordance in the header area.

**Body:** A single vertically scrolling list organized into two sections in order:
1. **Today** -- today's remaining bookings and any manual time blocks (FEAT-17), in strict time order. A booking whose start_time has already passed and is not yet marked Completed, No-Show, or Cancelled remains visible in this section (it does not disappear once its time passes) until the Pro acts or the Auto-Completion Sweep (FEAT-12.SPEC-004) resolves it.
2. **Upcoming** -- bookings beyond today, grouped by date, each date heading followed by that date's bookings in time order.

Each **booking row** (the shared pattern used identically here and on FEAT-12.SPEC-003, per the Brief's Shared UI Patterns) shows, left to right / top to bottom: start time, service name, client name (tappable), the paid badge and balance-due figure (derived by FEAT-12.SPEC-007), the "I'll be there" attendance label when present, a sync-reliability marking when the booking's reliability is uncertain, and a client note preview (a short excerpt of the Pro's private note for that client, when one exists) truncated to fit one line. A row-level action affordance exposes: view full client note, mark no-show, jump to reschedule/cancel, and (only once start_time has passed and the booking is still Confirmed or Awaiting Outcome) mark completed.

Each **time block row** shows its start/end and, for the Pro only, its private label; it has no quick actions except "add/edit time block," which navigates to FEAT-17.

**Footer:** None -- all actions are inline within rows or the header banner.

### Responsive Behavior

- **Compact breakpoint (phone width, including inside the Instagram in-app browser):** Single-column list as described above, full width; the Attention banner remains directly under the header at all times without requiring a scroll.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping, since the product's primary use is phone-first (ASMP: mobile-first for both roles).
- **Booking row on very narrow widths:** The client note preview truncates further or is omitted first, before any status badge (paid, balance due, attendance, reliability) is dropped -- badges never disappear to save space, since paid/attention status must remain visible at a glance.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Attention banner | Tap | Navigate to FEAT-12.SPEC-002 (Attention List) | Screen transitions | Standard navigation transition |
| Booking row -- client name | Tap | Navigate to FEAT-13 (Client Record Management) for that client | Screen transitions | Standard navigation transition |
| Booking row -- client note preview | Tap | Expand the full private note inline on the row | Row expands to show full note text | Note text becomes fully visible without navigating away |
| Booking row -- "no-show" action | Tap | Navigate to FEAT-11 (No-Show Marking & Deposit Forfeiture) for that booking | Screen transitions | Standard navigation transition |
| Booking row -- "reschedule/cancel" action | Tap | Navigate to FEAT-30 (Pro Booking Management) for that booking | Screen transitions | Standard navigation transition |
| Booking row -- "mark completed" action (shown only once eligible per FEAT-12.SPEC-006) | Tap | Validate eligibility via FEAT-12.SPEC-006; if valid, transition the booking to Completed and record the balance as settled in person | Row updates in place to show Completed status; action affordance for that row is removed | Brief inline confirmation (e.g., a check mark and "Completed") replaces the action; no full-screen transition |
| Booking row -- "mark completed" action (attempted before eligible) | Tap | No state change -- the control is disabled before start_time has passed | Control remains disabled | Control shows a disabled visual treatment; it is not tappable |
| "Add time block" affordance | Tap | Navigate to FEAT-17 (Manual Time Blocking) | Screen transitions | Standard navigation transition |
| Empty-day shortcut ("share your booking link") | Tap | Opens the Pro's own booking-link share action | Share action presented | Standard platform share affordance appears |
| Navigation -- Money | Tap | Navigate to FEAT-28 (Payout Account Connection & Payout Visibility) | Screen transitions | Standard navigation transition |
| Navigation -- Settings | Tap | Navigate to FEAT-27 (Pro Profile & Booking Page Settings) | Screen transitions | Standard navigation transition |
| Navigation -- Insights | Tap | Navigate to FEAT-25 (Booking & Revenue Insights) | Screen transitions | Standard navigation transition |
| Navigation -- Past Bookings | Tap | Navigate to FEAT-12.SPEC-003 (Past Bookings Browse) | Screen transitions | Standard navigation transition |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches today's and upcoming bookings, time blocks, and derived status | List reloads | Loading indicator during refresh; list content updates in place |

### Accessibility Notes

- **Focus order:** Screen title -> Attention banner (when present) -> navigation affordances -> Today section heading -> each booking/time-block row in time order (within a row: time, service, client name, badges, note preview, action affordances) -> Upcoming section date headings and their rows in order.
- **Dynamic-change announcements:** When a "mark completed" action succeeds, the row's updated status ("Completed") is announced to assistive technology. When the Attention banner's count changes on refresh, the updated count is announced. Loading and offline-state transitions are announced when they occur.
- **Status conveyed beyond color:** Every paid, balance-due, attendance, and reliability badge carries a word, never color alone (ASMP-28), so no accessibility gap exists for badge meaning.
- **Keyboard alternatives:** Every action on this screen (navigation, expand note, mark completed, refresh) is reachable without a pointer-only gesture; pull-to-refresh has an equivalent tappable refresh control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has bookings) | Today and Upcoming sections populated as described in Layout and Content | Data fetch succeeds with at least one booking or time block | Data changes (new fetch, action taken) |
| Empty day | Today section shows a friendly "Nothing booked yet today" message with a shortcut to share the booking link; Upcoming section still shows if it has content | Today section has zero bookings and zero time blocks | A booking or time block for today is created, or the date rolls to a day with bookings |
| Loading | A lightweight in-place indicator appears without blanking existing content already on screen; on first-ever load, a full-screen lightweight loading indicator appears briefly | Screen first opens, or a refresh is triggered | Data fetch completes (success or failure) |
| Error | Error banner at the top of the list: "Couldn't load your schedule. Check your connection and try again." with a Retry control; any previously loaded content remains visible below the banner if this is a refresh failure, not a first load | Data fetch fails | Pro taps Retry and the fetch succeeds, or connectivity is restored and an automatic retry succeeds |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded schedule." at the top; the most recently loaded schedule remains viewable read-only; quick actions that write data (mark no-show, mark completed, reschedule/cancel) are disabled with a note that they require reconnecting; navigation to other features that themselves require connectivity shows their own offline handling | Connectivity is lost while this screen is open, or the screen is opened while already offline with cached data available | Connectivity is restored -- the banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry form fields; its only rule-governed interaction is the "mark completed" action.

Validation and eligibility for "mark completed" is governed by FEAT-12.SPEC-006 (Booking Completion Rules). See that spec for the full eligibility window and denied-state messages. Checked at the moment the Pro taps the action, before the write is attempted.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Attention banner tap | FEAT-12.SPEC-002 (Attention List) | -- |
| Client name tap | FEAT-13.SPEC-001 (Client Record Detail) | FEAT-13 |
| "No-show" action tap | FEAT-11 (No-Show Marking & Deposit Forfeiture) | FEAT-11 |
| "Reschedule/cancel" action tap | FEAT-30 (Pro Booking Management) | FEAT-30 |
| "Add time block" tap | FEAT-17 (Manual Time Blocking) | FEAT-17 |
| Navigation -- Past Bookings | FEAT-12.SPEC-003 (Past Bookings Browse) | -- |
| Navigation -- Money | FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28 |
| Navigation -- Settings | FEAT-27 (Pro Profile & Booking Page Settings) | FEAT-27 |
| Navigation -- Insights | FEAT-25.SPEC-001 (Insights Summary Screen) | FEAT-25 |

## Data Model

**Creates:** None.
**Reads:** Booking (service, start_time, duration, client reference, price_agreed, deposit_amount, state, attendance_reply, balance_due -- derived by FEAT-12.SPEC-007) for today and upcoming dates; Client (name, private_note preview); Deposit Transaction (status, amount, via FEAT-12.SPEC-007); Message (delivery_status flags); Time Block (start, end, label); Calendar Connection (status, via FEAT-12.SPEC-007); Messaging Consent (state, textability display only); Open Attention Item count (via FEAT-12.SPEC-005, for the banner).
**Updates:** Booking -- `state` field, transitioned to `Completed` by the "mark completed" action (validated by FEAT-12.SPEC-006).
**Deletes:** None.

## Business Rules

- The "mark completed" action is governed entirely by FEAT-12.SPEC-006 -- this screen enforces but does not define the eligibility window.
- Paid badge, balance due, attendance label, and sync-reliability marking are derived entirely by FEAT-12.SPEC-007 -- this screen renders but does not compute them.
- Every screen in this feature, including this one, requires an authorized viewer per FEAT-12.SPEC-008 -- checked before any data loads and on every refresh.
- XBR-11: a setup change never silently removes a booking from this list -- a booking left outside changed hours remains visible here and is separately flagged through the Attention List (FEAT-12.SPEC-002/FEAT-12.SPEC-005), never hidden or auto-cancelled.
- XBR-13: if calendar sync lapses, this screen shows reduced confidence (the sync-reliability marking) to the Pro only, never to the client.
- A booking whose start_time has passed remains visible in the Today section (not silently removed) until it is completed, marked no-show, or the Auto-Completion Sweep resolves it (FEAT-12.SPEC-004) -- this keeps the Pro's glance trustworthy about what still needs a decision.

## Edge Cases

- **Pro taps "mark completed" on a booking that another device (the Pro's own second session) or the Auto-Completion Sweep (FEAT-12.SPEC-004) has already completed** -- Concurrent-edit conflict, governed by the Booking entity's reject-with-refresh contention resolution (dependency map): the action is refused, the row refreshes to show its current Completed state, and no error is shown beyond the row simply reflecting the up-to-date status.
- **Pro taps "no-show" or "reschedule/cancel" on a booking that was cancelled by the Client moments earlier** -- The Pro is navigated to FEAT-11 or FEAT-30, which itself reads the booking's current state and shows that destination feature's own defined current-state message for a cancelled booking (reject-with-refresh); this screen's own row refreshes to the new state on return.
- **More bookings exist for a date than fit on screen** -- The list scrolls; no pagination or truncation of booking rows occurs within a day's list.
- **Pro double-taps "mark completed" rapidly** -- The second tap is ignored while the first write is in progress; the control shows a brief disabled/loading treatment during the write.
- **Pro navigates away mid-action (e.g., taps a quick action, then backs out before the destination screen loads)** -- No partial state is created on this screen, since this screen never writes data itself except the atomic mark-completed transition; navigating away before mark-completed is tapped leaves the booking untouched.
- **Time block overlaps a confirmed booking** -- Both are shown on the schedule (the block is shown as occupied time alongside the booking); resolving the overlap is an explicit Pro action through FEAT-17/FEAT-30, not something this screen resolves automatically.
- **Empty day with zero bookings and zero time blocks** -- The friendly "Nothing booked yet today" message and share-link shortcut appear, per the Empty day state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-006 (Booking Completion Rules) | References (outbound) | Governs the mark-completed action's eligibility and mechanics |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid badge, balance due, attendance label, and reliability marking shown on every row |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | References (inbound) | Supplies the Open Attention Item count shown in the header banner |
| FEAT-12.SPEC-002 (Attention List) | Navigation (outbound) | Attention banner navigates here |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Navigation (outbound) | Past Bookings navigation entry navigates here |
| FEAT-12.SPEC-004 (Auto-Completion Sweep) | References (inbound) | Automatic completion outcomes are reflected here on refresh |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound) | Default landing screen after sign-in |
| FEAT-13 (Client Record Management) | Navigation (outbound) | Client name tap navigates here |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Navigation (outbound) | No-show action navigates here |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | Reschedule/cancel action navigates here |
| FEAT-17 (Manual Time Blocking) | Navigation (outbound) | Add time block navigates here |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Navigation (outbound) | Money navigation entry |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound) | Settings navigation entry |
| FEAT-25 (Booking & Revenue Insights) | Navigation (outbound) | Insights navigation entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dashboard_viewed | booking_count_today, has_attention_items (boolean) | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| quick_action_taken | action_type (view_note / no_show / reschedule_cancel / add_time_block) | Pro taps a row-level or header quick action | supports success-metrics.md: "Daily Dashboard Glance Speed" (view_note actions) or supports success-metrics.md: "Pro Change Correctness" (no_show and reschedule_cancel actions, which initiate the pro-side change flows that metric measures) |
| booking_marked_completed | time_since_start_time | "Mark completed" action succeeds | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| empty_day_share_link_tapped | -- | Pro taps the share-link shortcut on an empty day | supports success-metrics.md: "Daily Dashboard Glance Speed" (measures whether the empty state still leads to useful action rather than a dead end) |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Talia opens the app after signing in, when the dashboard loads, then she sees today's remaining bookings in time order with paid badge, balance due, and attendance status on each row within a few seconds, per the Daily Dashboard Glance Speed target.

**FEAT-12.SPEC-001-AC-02:** Given Talia has zero bookings today, when she opens the dashboard, then she sees "Nothing booked yet today" with a shortcut to share her booking link.

**FEAT-12.SPEC-001-AC-03:** Given Talia has bookings scheduled beyond today, when she scrolls past the Today section, then she sees the Upcoming section grouped by date.

**FEAT-12.SPEC-001-AC-04:** Given Talia taps a booking row's client name, when the tap registers, then she is navigated to that client's record (FEAT-13).

**FEAT-12.SPEC-001-AC-05:** Given Talia taps "no-show" on a booking row, when the tap registers, then she is navigated to FEAT-11 for that booking.

**FEAT-12.SPEC-001-AC-06:** Given Talia taps "reschedule/cancel" on a booking row, when the tap registers, then she is navigated to FEAT-30 for that booking.

**FEAT-12.SPEC-001-AC-07:** Given a booking's start_time has passed and it is still Confirmed, when Talia taps "mark completed," then the booking transitions to Completed and the row updates in place to show that status.

**FEAT-12.SPEC-001-AC-08:** Given a booking's start_time has not yet arrived, when Talia looks at its row, then the "mark completed" control is disabled and not tappable.

**FEAT-12.SPEC-001-AC-09:** Given one or more Open Attention Items exist, when Talia opens the dashboard, then the Attention banner shows the count and is tappable to FEAT-12.SPEC-002.

**FEAT-12.SPEC-001-AC-10:** Given zero Open Attention Items exist, when Talia opens the dashboard, then no Attention banner is shown.

**FEAT-12.SPEC-001-AC-11:** Given the dashboard's data fetch fails, when the failure occurs, then an error banner "Couldn't load your schedule. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-001-AC-12:** Given Talia loses connectivity while viewing the dashboard, when the offline state activates, then the banner "You're offline -- showing your most recently loaded schedule." appears, the schedule remains viewable, and write actions are disabled with a reconnect note.

**FEAT-12.SPEC-001-AC-13:** Given Talia's connectivity is restored after the offline state, when reconnection is detected, then the offline banner clears and the schedule refreshes automatically.

**FEAT-12.SPEC-001-AC-14:** Given a booking's Calendar Connection reliability is uncertain, when Talia views that row, then it shows the "Reliability uncertain" marking rather than presenting false confidence.

**FEAT-12.SPEC-001-AC-15:** Given Talia taps "mark completed" twice in rapid succession on the same row, when the second tap registers while the first write is in progress, then the second tap has no additional effect.

**FEAT-12.SPEC-001-AC-16:** Given a booking Talia is about to mark completed was already auto-completed by FEAT-12.SPEC-004 moments earlier, when she taps "mark completed," then the action is refused and the row refreshes to show the already-Completed status.

**FEAT-12.SPEC-001-AC-17:** Given Platform Operator (Support) is viewing Talia's dashboard during an active support session, when Support looks at any booking row, then no client private-note preview is shown and no quick-action controls appear.

**FEAT-12.SPEC-001-AC-18:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any booking data.

**FEAT-12.SPEC-001-AC-19:** Given a manual time block overlaps a confirmed booking on today's schedule, when Talia views the Today section, then both the block and the booking are shown, with the block visible as occupied time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 13 | 13 |
| States | 5 (loaded, empty day, loading, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Screen Spec: Attention List

## Overview

**Name:** Attention List
**ID:** FEAT-12.SPEC-002
**Type:** Screen
**Purpose:** Surfaces everything needing the Pro's attention -- sync issues, message delivery failures, refunds in progress, card-issuer disputes, bookings left outside changed hours, and waitlist demand -- in one place.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying every Open Attention Item aggregated by FEAT-12.SPEC-005
- Displaying aggregate waitlist demand as a separate informational item
- Routing each item's action to the owning feature (FEAT-04, FEAT-28, FEAT-16, FEAT-30)
- The empty state (nothing needs attention) and loading/error/offline states

**Non-Goals:**
- De-duplicating or resolving attention signals -- owned by FEAT-12.SPEC-005 (Attention Flag Aggregation); this screen only displays its output
- Resolving the underlying cause (reconnecting a calendar, retrying a refund, submitting dispute evidence) -- owned by the destination feature (FEAT-04, FEAT-28, FEAT-16); this screen only navigates there
- Showing individual waitlist entries -- excluded per the Access Matrix (Waitlist = View, aggregate counts only for the Pro); showing individual client waitlist requests would expose data the Pro is not entitled to see per-entry
- Letting Support act on any attention item -- excluded per scope-boundaries.md SC-05: Support's access here is View-only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro taps the Attention banner | None -- list loads current Open items |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own account's attention items only | Tap through to any item's owning feature | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review; the same items the Pro sees, since none of this screen's content is private client-note material | View only -- can tap through to an owning feature's read-only view where that feature permits Support access; cannot take any write action there either | Any write action reachable from an item is unavailable to Support in the owning feature itself, consistent with that feature's own Access Matrix row |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title "Attention" with a back arrow returning to FEAT-12.SPEC-001.

**Body:** A single vertically scrolling list of attention cards (the shared "attention item" pattern per the Brief's Shared UI Patterns), each showing: cause label (e.g., "Reconnect calendar," "Message delivery gap," "Refund in progress," "Card-issuer dispute," "Booking outside changed hours"), the affected booking's date/time and client name when the cause is booking-specific, or the account-level context when it is not (e.g., calendar reconnection), and a single action button whose label matches the destination (e.g., "Reconnect," "View money list," "Download summary," "Review booking"). Cards are ordered most-recently-detected first.

A separate, visually distinct **waitlist demand card** appears at the top of the list (or, when no other attention items exist, as the sole card) showing an aggregate count of clients waiting for openings (e.g., "4 clients waiting for an opening") with no individual entries -- this card has no action button, since acting on waitlist demand is not a defined capability of this screen (the Pro sees demand only, per the Access Matrix).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column card list, full width, as described above.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 | Screen closes | Standard navigation transition |
| Attention card -- "Reconnect calendar" | Tap | Navigate to FEAT-04 (Two-Way Calendar Sync) | Screen transitions | Standard navigation transition |
| Attention card -- "Refund in progress" / money action | Tap | Navigate to FEAT-28 (Payout Account Connection & Payout Visibility) | Screen transitions | Standard navigation transition |
| Attention card -- "Card-issuer dispute" | Tap | Navigate to FEAT-16 (Booking & Payment Activity Record) for the dispute flag and evidence summary download | Screen transitions | Standard navigation transition |
| Attention card -- "Message delivery gap" | Tap | Navigate to FEAT-12.SPEC-001's booking row context (no dedicated resolution screen exists for a delivery gap; the Pro reviews the booking and may re-send or contact the client through the booking's own context) | Screen transitions to the relevant booking on FEAT-12.SPEC-001 | Standard navigation transition |
| Attention card -- "Booking outside changed hours" | Tap | Navigate to FEAT-30 (Pro Booking Management) for that booking | Screen transitions | Standard navigation transition |
| Waitlist demand card | Tap | No action -- display-only, non-interactive | None | None (card is visually inert beyond its count display) |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the current Open Attention Item set and waitlist demand count | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> waitlist demand card (when present) -> each attention card in order (most-recently-detected first), each announced with its cause label before its action button.
- **Dynamic-change announcements:** When an item resolves and drops off the list on refresh, the updated count is announced; when a new item appears, it is announced as part of the refreshed list content.
- **Status conveyed beyond color:** Every card's cause label is text, never conveyed by color or icon alone (ASMP-28).
- **Keyboard alternatives:** Every action (navigation, refresh) is reachable without a pointer-only gesture.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Has items | List of attention cards and/or the waitlist demand card as described in Layout and Content | One or more Open Attention Items exist, or waitlist demand is greater than zero | Data changes (refresh, item resolves) |
| Empty (nothing needs attention) | Friendly message "Nothing needs your attention right now" -- shown when there are zero Open Attention Items and zero waitlist demand | Data fetch succeeds with no items and no waitlist demand | An item appears or waitlist demand becomes greater than zero |
| Loading | Lightweight in-place indicator; on first-ever load, a brief full-screen lightweight loading indicator | Screen first opens, or a refresh is triggered | Data fetch completes |
| Error | Error banner "Couldn't load your attention list. Check your connection and try again." with Retry; prior content remains visible below the banner on a refresh failure | Data fetch fails | Retry succeeds, or automatic retry succeeds after reconnection |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded attention list." at the top; the most recently loaded list remains viewable read-only; tapping a card's action navigates to the destination feature, which applies its own offline handling | Connectivity lost while this screen is open, or screen opened while offline with cached data available | Connectivity restored -- banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry fields; it is a display and navigation surface only. N/A -- no validation rules apply.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | -- |
| "Reconnect calendar" card tap | FEAT-04 (Two-Way Calendar Sync) | FEAT-04 |
| Money-related card tap | FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28 |
| Dispute card tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
| Message-delivery-gap card tap | FEAT-12.SPEC-001 (booking row context) | -- |
| Setup-conflict card tap | FEAT-30 (Pro Booking Management) | FEAT-30 |

## Data Model

**Creates:** None.
**Reads:** Open Attention Item set (cause category, affected booking or account reference, first-detected time -- via FEAT-12.SPEC-005); Waitlist Entry (aggregate count only, per Pro Account, via FEAT-20).
**Updates:** None -- this screen never writes; resolution of an item happens through the destination feature and is reflected here only on the next refresh, or through FEAT-12.SPEC-005's own periodic re-check.
**Deletes:** None.

## Business Rules

- Booking-linked cards (sync-reliability and dispute items) show the underlying booking's paid, balance, and reliability values exactly as derived by FEAT-12.SPEC-007 (Balance Due & Status Display Rules), which this screen enforces on every such booking reference and never recomputes.
- Every item shown here is sourced exclusively from FEAT-12.SPEC-005's aggregation -- this screen never computes or de-duplicates a cause itself.
- Waitlist demand is shown as an aggregate count only, never individual entries, per the Access Matrix (Waitlist = View for the Pro).
- Support's access is View-only here, consistent with SC-05 -- Support can navigate to a destination feature from a card only where that feature's own Access Matrix row permits Support's read-only view; no write action is ever available to Support from this screen or through it.
- A resolved item is never shown as an active card -- once FEAT-12.SPEC-005 marks an item Resolved, it disappears from this screen's next load, consistent with XBR-10, XBR-13, XBR-17, XBR-22, and XBR-11's respective resolution definitions.

## Edge Cases

- **All attention items resolve while the Pro is viewing this screen** -- The list does not change mid-view; the Empty state appears only on the next refresh (pull-to-refresh or re-opening the screen), since this is a snapshot view, not a live-updating one.
- **A new attention item appears while the Pro is viewing this screen** -- Similarly not shown until the next refresh; this screen is a snapshot, consistent with the dependency map treating Attention Item aggregation as owned entirely by FEAT-12.SPEC-005's own periodic processing.
- **Waitlist demand count is exactly zero but other attention items exist** -- The waitlist demand card is omitted entirely (not shown with a "0" count), and only the other attention cards appear.
- **Pro taps a card whose destination feature (e.g., FEAT-04) is itself unreachable due to connectivity loss** -- The destination feature's own offline handling applies once navigation completes; this screen's own navigation action itself always succeeds locally (it is a local screen transition, not a network call).
- **Two attention cards reference the same booking with different causes (e.g., a message delivery gap and a dispute)** -- Both cards are shown separately, since FEAT-12.SPEC-005 de-duplicates only within a (booking, cause) pair, not across causes for the same booking.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | References (inbound) | Supplies the Open Attention Item set this screen displays |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Attention banner navigates here |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid, balance, and reliability values shown on booking-linked cards |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-04 (Two-Way Calendar Sync) | Navigation (outbound) | Reconnect calendar action |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Navigation (outbound) | Money action |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (outbound) | Dispute flag and evidence summary download |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | Setup-conflict resolution action |
| FEAT-20 (Waitlist for Cancelled Slots) | References (inbound) | Aggregate waitlist demand count shown here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| attention_list_viewed | open_item_count, waitlist_demand_count | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| attention_item_resolved | cause_category, time_to_resolution | Pro views the list and an item has resolved since it was first detected (the resolution itself is computed by FEAT-12.SPEC-005; this event marks that the Pro's view reflects it) | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| attention_item_action_tapped | cause_category, destination_feature | Pro taps a card's action button | supports success-metrics.md: "Daily Dashboard Glance Speed" |

## Acceptance Criteria

**FEAT-12.SPEC-002-AC-01:** Given Talia has one or more Open Attention Items, when she opens this screen from the dashboard banner, then she sees a card for each item, most-recently-detected first.

**FEAT-12.SPEC-002-AC-02:** Given Talia has zero Open Attention Items and zero waitlist demand, when she opens this screen, then she sees "Nothing needs your attention right now."

**FEAT-12.SPEC-002-AC-03:** Given Talia has 4 clients waiting for an opening, when she opens this screen, then a waitlist demand card shows "4 clients waiting for an opening" with no individual entries and no action button.

**FEAT-12.SPEC-002-AC-04:** Given Talia's Calendar Connection needs reconnection, when she taps that card, then she is navigated to FEAT-04.

**FEAT-12.SPEC-002-AC-05:** Given a booking's deposit is Disputed, when Talia taps the dispute card, then she is navigated to FEAT-16 for the evidence summary download.

**FEAT-12.SPEC-002-AC-06:** Given a refund is in progress for a booking, when Talia taps that card, then she is navigated to FEAT-28's money list.

**FEAT-12.SPEC-002-AC-07:** Given a booking is flagged as outside her changed hours, when Talia taps that card, then she is navigated to FEAT-30 for that booking.

**FEAT-12.SPEC-002-AC-08:** Given the screen's data fetch fails, when the failure occurs, then the error banner "Couldn't load your attention list. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-002-AC-09:** Given Talia loses connectivity while viewing this screen, when the offline state activates, then the banner "You're offline -- showing your most recently loaded attention list." appears and the most recently loaded list remains viewable.

**FEAT-12.SPEC-002-AC-10:** Given an attention item resolves while Talia is actively viewing this screen without refreshing, when she looks at the list, then the resolved item still appears until her next refresh, since this is a snapshot view.

**FEAT-12.SPEC-002-AC-11:** Given Talia refreshes the screen after an item has resolved, when the refresh completes, then the resolved item's card no longer appears.

**FEAT-12.SPEC-002-AC-12:** Given Platform Operator (Support) is viewing this screen during an active support session, when Support looks at the list, then Support sees the same cards Talia would see, since none of this content is private client-note material.

**FEAT-12.SPEC-002-AC-13:** Given Platform Operator (Support) taps a card's action, when the destination feature loads, then no write action is available to Support there either, consistent with that feature's own read-only Access Matrix row.

**FEAT-12.SPEC-002-AC-14:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any attention data.

**FEAT-12.SPEC-002-AC-15:** Given two attention cards reference the same booking for different causes, when Talia views the list, then both cards are shown separately.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (has items, empty, loading, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Past Bookings Browse

## Overview

**Name:** Past Bookings Browse
**ID:** FEAT-12.SPEC-003
**Type:** Screen
**Purpose:** The Pro finds and reviews a past booking by browsing by date, using the same booking row pattern as the main schedule.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Browsing past bookings organized by date, most recent first
- Rendering the shared booking row pattern (paid badge, balance due, attendance reply, sync-reliability marking) for past bookings, identical to FEAT-12.SPEC-001
- Navigating from a past booking to its full activity timeline (FEAT-16)
- Staying equally responsive as a Pro's booking history grows over multiple years (SC-22)

**Non-Goals:**
- Editing or acting on a past booking (mark completed, no-show, cancel, reschedule) -- all of those actions require the booking to be in an active, non-terminal state; a genuinely past booking has already reached a terminal state (Completed, No-Show, Cancelled, Rescheduled, Expired) and this screen is read-only
- Deriving the balance-due or paid-badge values -- owned by FEAT-12.SPEC-007, reused identically here
- Computing revenue or booking insights from past bookings -- excluded per scope-boundaries.md's deferral notes; that is FEAT-25 (Booking & Revenue Insights, v1), not this screen
- Deleting or archiving past bookings -- excluded per scope-boundaries.md SC-22: booking history is retained for the life of the account for dispute evidence and insights; no delete or archive action exists on this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro navigates to "Past Bookings" | None -- list loads defaulting to the most recent past date with bookings |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own past bookings only | Browse and open a past booking's activity timeline (read-only navigation, no edits) | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review, except the client private-note preview, which is omitted entirely (FEAT-12.SPEC-008) | Browse only; can open the activity timeline where FEAT-16's own Access Matrix row permits Support | -- |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title "Past Bookings" with a back arrow returning to FEAT-12.SPEC-001, and a date browser control (e.g., a date picker or scrollable date strip) that lets the Pro jump to any past date.

**Body:** A single vertically scrolling list of past dates, most recent first, each date heading followed by that date's bookings using the identical booking row pattern described in FEAT-12.SPEC-001's Layout and Content (start time, service, client name, paid badge, balance due, attendance label, sync-reliability marking, client note preview) -- rendered here as read-only (no quick-action affordances, since every booking here is in a terminal state). Loading additional older dates happens as the Pro scrolls further back or selects an earlier date from the date browser.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described above, full width; the date browser remains accessible from the header without scrolling.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 | Screen closes | Standard navigation transition |
| Date browser control | Select a date | Jump the list to the selected date's bookings | List scrolls to or loads that date | List content updates to center on the selected date |
| Booking row -- client name | Tap | Navigate to FEAT-13 (Client Record Management) for that client | Screen transitions | Standard navigation transition |
| Booking row -- client note preview | Tap | Expand the full private note inline on the row | Row expands | Note text becomes fully visible |
| Booking row (elsewhere on the row) | Tap | Navigate to FEAT-16 (Booking & Payment Activity Record) for that booking's full activity timeline | Screen transitions | Standard navigation transition |
| Scroll to top / bottom of loaded range | Scroll | Loads the next older (or more recent) page of dates | List extends | Loading indicator appears briefly at the loaded edge while more history fetches |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the currently viewed date range | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> date browser control -> each date heading and its bookings in order (within a row: time, service, client name, badges, note preview).
- **Dynamic-change announcements:** When the date browser jumps the list to a new date, the new date heading is announced. When older history finishes loading during scroll, no interrupting announcement occurs (content simply extends).
- **Status conveyed beyond color:** Every badge carries a word, never color alone (ASMP-28), consistent with FEAT-12.SPEC-007.
- **Keyboard alternatives:** The date browser, every row tap target, and refresh are all reachable without a pointer-only gesture; infinite-scroll loading has an equivalent "load more" control for keyboard and assistive-technology use.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has past bookings) | Dates and booking rows as described in Layout and Content | Data fetch for the current date range succeeds with at least one booking | Data changes (date browser selection, further scroll, refresh) |
| Empty (no past bookings yet) | Friendly message "No past bookings yet -- they'll show up here once your first appointment happens" | The Pro Account has zero bookings with a start_time in the past | A booking's start_time passes into the past |
| Empty selected date | When the Pro jumps to a specific date via the date browser and that date has no bookings, a message "Nothing booked on this date" appears for that date only, with the surrounding dates' content (if loaded) still visible | Selected date has zero bookings | Pro selects a different date |
| Loading | Lightweight in-place indicator, at the loaded edge during scroll-triggered pagination, or a brief full-screen indicator on first load | Screen first opens, a refresh is triggered, or the Pro scrolls to the loaded edge | Data fetch completes |
| Error | Error banner "Couldn't load your past bookings. Check your connection and try again." with Retry; already-loaded content remains visible below the banner on a refresh or pagination failure | Data fetch fails | Retry succeeds, or automatic retry succeeds after reconnection |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded past bookings." at the top; the most recently loaded range remains viewable read-only; the date browser can still be used within already-loaded dates, but jumping to an unloaded date shows the offline banner instead of new content until reconnected | Connectivity lost while this screen is open, or opened while offline with cached data available | Connectivity restored -- banner clears and the requested range loads |

## Validation Rules

This screen has no user-entry form fields and performs no writes. N/A -- no validation rules apply.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | -- |
| Client name tap | FEAT-13 (Client Record Management) | FEAT-13 |
| Booking row tap (elsewhere) | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |

## Data Model

**Creates:** None.
**Reads:** Booking (service, start_time, duration, client reference, price_agreed, deposit_amount, state, attendance_reply, balance_due -- derived by FEAT-12.SPEC-007) for dates in the past; Client (name, private_note preview); Deposit Transaction (status, amount, via FEAT-12.SPEC-007); Calendar Connection (status, via FEAT-12.SPEC-007, for historical reliability marking where applicable); Messaging Consent (state, textability display).
**Updates:** None -- this screen is entirely read-only.
**Deletes:** None.

## Business Rules

- Every booking shown here is, by definition, past its start_time and therefore in a terminal or near-terminal state; this screen never offers the write actions available on FEAT-12.SPEC-001 (mark completed, no-show, reschedule/cancel), since those require an active, non-terminal booking.
- Paid badge, balance due, attendance label, and sync-reliability marking are derived identically to FEAT-12.SPEC-001, via FEAT-12.SPEC-007 -- the same booking presented on both screens shows the same values.
- Booking history is retained for the life of the account (SC-22); this screen's responsiveness must not degrade as that history grows across multiple years, so older history loads incrementally (pagination on scroll) rather than all at once.
- Every screen in this feature, including this one, requires an authorized viewer per FEAT-12.SPEC-008 -- checked before any data loads and on every refresh.

## Edge Cases

- **Pro's account has years of booking history** -- The date browser and incremental loading (pagination on scroll) keep the screen responsive; only the currently viewed date range is fetched at once, consistent with SC-22's retention-without-degradation expectation.
- **Pro selects a date in the future from the date browser** -- The date browser only offers dates up to and including today, since this screen is scoped to past bookings; today's and future bookings are shown on FEAT-12.SPEC-001, not here.
- **A booking's state changes (e.g., an auto-completion sweep resolves it) while the Pro is viewing this screen** -- Since this screen only ever shows bookings whose start_time has passed, a status change here does not remove the booking from view; the row's badge updates to reflect the new state on the next refresh (this is a display update, not a concurrent-edit conflict, since no write is ever attempted from this read-only screen).
- **Pro scrolls rapidly through many months of history** -- Pagination loads sequentially as the scroll reaches each loaded edge; a rapid scroll may show a brief loading indicator at the edge without blocking the already-loaded content above it.
- **Client record referenced by a very old booking has since been deleted (FEAT-13 hard delete)** -- The booking row still shows, since Booking records are retained regardless of Client deletion (XBR-19: financial and timeline records are retained in de-identified form); the client name shows a de-identified placeholder (e.g., "Former client") and no note preview is offered, since the note itself was deleted along with the Client record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Past Bookings entry navigates here |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid badge, balance due, attendance label, and reliability marking shown on every row, identically to FEAT-12.SPEC-001 |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-13 (Client Record Management) | Navigation (outbound) | Client name tap navigates here |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (outbound) | Booking row tap navigates to the full activity timeline |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| past_bookings_viewed | date_range_loaded | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" (measures whether the Pro's broader glance at their history stays fast as it grows, per SC-22) |
| past_booking_timeline_opened | -- | Pro taps a booking row to open its activity timeline | N/A -- reason: opening a historical timeline is a low-frequency lookup action with no defined success-metrics.md target of its own; it is tracked for feature-usage visibility rather than tied to a Stage 2 metric |

## Acceptance Criteria

**FEAT-12.SPEC-003-AC-01:** Given Talia has past bookings, when she navigates to this screen from FEAT-12.SPEC-001, then she sees her most recent past date's bookings first, in the same booking row pattern used on the main schedule.

**FEAT-12.SPEC-003-AC-02:** Given Talia has zero bookings in the past, when she opens this screen, then she sees "No past bookings yet -- they'll show up here once your first appointment happens."

**FEAT-12.SPEC-003-AC-03:** Given Talia selects a specific past date with no bookings, when the date browser jumps there, then she sees "Nothing booked on this date" for that date.

**FEAT-12.SPEC-003-AC-04:** Given Talia taps a booking row (outside the client name), when the tap registers, then she is navigated to FEAT-16's full activity timeline for that booking.

**FEAT-12.SPEC-003-AC-05:** Given Talia taps a booking row's client name, when the tap registers, then she is navigated to that client's record (FEAT-13).

**FEAT-12.SPEC-003-AC-06:** Given a past booking's Deposit Transaction status is Captured, when Talia views its row, then the paid badge reads "Paid," matching FEAT-12.SPEC-007's derivation used identically on FEAT-12.SPEC-001.

**FEAT-12.SPEC-003-AC-07:** Given Talia scrolls to the bottom of her currently loaded history, when she continues scrolling, then the next older page of dates loads with a brief loading indicator, without disrupting already-loaded content.

**FEAT-12.SPEC-003-AC-08:** Given Talia's account has multiple years of booking history, when she opens this screen, then only the currently viewed date range is fetched, keeping the screen responsive.

**FEAT-12.SPEC-003-AC-09:** Given this screen's data fetch fails, when the failure occurs, then the error banner "Couldn't load your past bookings. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-003-AC-10:** Given Talia loses connectivity while viewing this screen, when the offline state activates, then the banner "You're offline -- showing your most recently loaded past bookings." appears and already-loaded dates remain viewable.

**FEAT-12.SPEC-003-AC-11:** Given Talia is offline and jumps the date browser to a date outside what is already loaded, when the selection registers, then the offline banner is shown in place of new content until reconnected.

**FEAT-12.SPEC-003-AC-12:** Given a client linked to an old booking has since been deleted, when Talia views that booking's row, then the client name shows a de-identified placeholder and no note preview is offered.

**FEAT-12.SPEC-003-AC-13:** Given Talia looks for a mark-completed, no-show, or reschedule/cancel action on any row on this screen, when she inspects the row, then none of those actions are present, since every booking here is already in a terminal state.

**FEAT-12.SPEC-003-AC-14:** Given Platform Operator (Support) is viewing this screen during an active support session, when Support views a booking row, then no client private-note preview is shown.

**FEAT-12.SPEC-003-AC-15:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any past-booking data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (loaded, empty, empty selected date, loading, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Auto-Completion Sweep

## Overview

**Name:** Auto-Completion Sweep
**ID:** FEAT-12.SPEC-004
**Type:** Automation
**Purpose:** Automatically marks a Booking Completed 7 days (platform parameter: `booking-auto-completion-window-days`) after its appointment time if the Pro never marked it Completed or No-Show.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Periodically scanning for bookings eligible for automatic completion
- Applying the completion eligibility rules from FEAT-12.SPEC-006 to each candidate
- Transitioning eligible bookings to Completed and recording the balance as settled in person
- Emitting the resulting outcome so it is reflected on the schedule and past-bookings screens

**Non-Goals:**
- Defining the eligibility window and completion mechanics themselves -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this automation only applies that spec's rules on a schedule
- Marking a booking No-Show -- excluded per the dependency map's Entity-Lifecycle Coverage Matrix: No-Show is a distinct, Pro-initiated transition owned by FEAT-11, never an automatic outcome of this sweep
- Notifying the client that their booking was completed -- product-features.md's Communications field for this feature is N/A (this is a viewing surface); no client-facing message is defined for auto-completion, and none is added here without a Stage 2 source
- Taking or recording an in-app balance payment -- excluded per scope-boundaries.md SC-16; this automation only marks the balance as settled in person, never processes a payment

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Scheduled sweep pass | system (schedule-based; no user-facing trigger spec) | Runs on a recurring schedule frequent enough that no eligible booking waits more than a small fraction of a day past its 7-day mark before being swept | For each candidate Booking: state, start_time, price_agreed, deposit_amount, and the Pro Account it belongs to |

## Processing Logic

1. On each scheduled pass, read every Booking currently in `Confirmed` or `Awaiting Outcome` state across all Pro Accounts.
2. For each such Booking, evaluate whether the current time is at least 7 days after its `start_time`.
3. For every Booking that meets the window condition, apply the eligibility check defined in FEAT-12.SPEC-006 (state must still be `Confirmed` or `Awaiting Outcome` at the moment of transition, to guard against a race with a Pro action taken between step 1's read and this step).
4. For each Booking that still passes the check, transition its `state` to `Completed` and record the balance as settled in person (per FEAT-12.SPEC-006's completion mechanics -- no in-app balance charge is created).
5. For any Booking that no longer passes the check (because the Pro or another automation already moved it out of `Confirmed`/`Awaiting Outcome` since step 1), skip it without effect.
6. Log the sweep pass's outcome counts (bookings evaluated, bookings completed, bookings skipped) for operational visibility; this logging is internal and not itself a user-facing feature.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Booking auto-completed | Booking was `Confirmed` or `Awaiting Outcome`, start_time was at least 7 days ago, and it still passed the eligibility check at transition time | Booking.state -> Completed; balance recorded as settled in person | The booking now shows a Completed status the next time the Pro opens FEAT-12.SPEC-001 or FEAT-12.SPEC-003 -- no interrupting notification, since this is a background sweep, not a screen the Pro is actively watching | FEAT-12.SPEC-001, FEAT-12.SPEC-003 (both display the updated state); FEAT-25.SPEC-004 (aggregates the completed outcome) |
| No action needed (not yet eligible) | Booking is `Confirmed`/`Awaiting Outcome` but start_time is less than 7 days in the past | None | None -- silent, re-evaluated on the next pass | -- |
| No action needed (already resolved) | Booking already left `Confirmed`/`Awaiting Outcome` before this pass reached it (Pro marked it, cancelled it, or a prior sweep pass already completed it) | None | None -- silent | -- |
| Sweep pass failure | The scheduled pass itself cannot run to completion (e.g., an internal processing error interrupts the scan) | No partial state changes are left inconsistent -- any Booking not reached by a failed pass is picked up cleanly on the next scheduled pass | None directly; no booking is left in an ambiguous state, since the sweep only ever moves a Booking forward to Completed in a single step | FEAT-12.SPEC-001, FEAT-12.SPEC-003 (unaffected until the next successful pass catches up) |

## Data Model

**Reads:** Booking -- `state`, `start_time`, `price_agreed`, `deposit_amount`, and the owning Pro Account reference, across all Pro Accounts, for every Booking currently `Confirmed` or `Awaiting Outcome`.
**Creates:** None.
**Updates:** Booking -- `state` (to `Completed`) for each eligible Booking found by this pass.
**Deletes:** None.

## Business Rules

- The completion window is 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time`, per XBR-12 and FEAT-12.SPEC-006.
- This sweep never marks a Booking No-Show -- No-Show is exclusively a Pro-initiated action owned by FEAT-11.
- This sweep is non-destructive: it only ever transitions a Booking forward from `Confirmed`/`Awaiting Outcome` to `Completed`; it never reverts, cancels, or reschedules a Booking.
- The sweep runs across every Pro Account uniformly -- there is no per-Pro configuration of the completion window (it is one value for every Pro (platform parameter: `booking-auto-completion-window-days`), not a per-account setting).
- A Booking already moved out of `Confirmed`/`Awaiting Outcome` by the time this sweep reaches it (by the Pro, by a client cancellation, or by a prior sweep pass) is left untouched, consistent with the Booking entity's reject-with-refresh contention resolution: the first committed transition wins.

## Edge Cases

- **Booking passes its 7-day mark while a client-initiated cancellation is being processed at the same instant** -- Concurrent trigger firing: whichever transition (the sweep's completion, or the cancellation) commits first wins; the other finds the Booking already out of `Confirmed`/`Awaiting Outcome` at its eligibility check and skips it without effect, per FEAT-12.SPEC-006's reject-with-refresh resolution.
- **The Pro marks a booking completed manually a moment before a scheduled sweep pass reaches it** -- The sweep's eligibility check (step 3) re-verifies state at transition time and finds the Booking already `Completed`; it is skipped without effect and without any error.
- **A sweep pass is still processing a large batch when the next scheduled pass would normally start** -- The next pass does not start a second concurrent scan while one is in flight; it waits for the current pass to finish, so no Booking is evaluated by two overlapping passes at once.
- **A Booking's start_time falls in a time zone whose day boundary is ambiguous relative to the sweep's own scheduling clock** -- The comparison is always against the Booking's own `start_time` (stored and interpreted in the Pro's account timezone, per XBR-25), not the sweep's own clock's local day boundary; the 7-day window is computed from that same instant, so timezone handling introduces no separate ambiguity.
- **Pro Account is Paused (subscription lapse or Pro-chosen pause) when a sweep pass reaches one of its bookings** -- The pause affects new bookings and deposits only (XBR-14); it does not exempt existing bookings from this sweep, so eligible bookings on a paused account are still completed on schedule.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-006 (Booking Completion Rules) | References (outbound) | This automation applies SPEC-006's eligibility window and transition mechanics on every pass |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Affects (outbound) | Displays the resulting Completed state on the Pro's next visit |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Affects (outbound) | Displays the resulting Completed state once the booking is in the past |
| FEAT-25.SPEC-004 (Insights Aggregates) | Triggers (outbound) | Each booking auto-completed by this sweep is a booking-outcome event that FEAT-25.SPEC-004 picks up when refreshing its insights aggregates |

## Analytics and Success Signals

- **booking_auto_completed** (days_since_start_time) -- N/A -- reason: this is a background, automatic outcome with no Pro-facing speed or correctness dimension of its own to measure; it supports operational visibility (sweep pass outcome counts, per Processing Logic step 6) rather than any Stage 2 success metric. The Pro-facing signals for this feature's contribution to Daily Dashboard Glance Speed and Pro Change Correctness are emitted by FEAT-12.SPEC-001 (dashboard_viewed, quick_action_taken, booking_marked_completed) and FEAT-12.SPEC-002/FEAT-12.SPEC-005 (attention_item_resolved), not by this automation.
- **auto_completion_sweep_pass_summary** (bookings_evaluated, bookings_completed, bookings_skipped) -- N/A -- reason: an internal operational log for the sweep's own health, not a product success signal tied to a persona-facing outcome.

## Acceptance Criteria

**FEAT-12.SPEC-004-AC-01:** Given a Confirmed booking whose start_time was 7 days ago and Talia never marked it completed or no-show, when the scheduled sweep pass runs, then the booking's state transitions to Completed.

**FEAT-12.SPEC-004-AC-02:** Given a Confirmed booking whose start_time was only 2 days ago, when the sweep pass runs, then the booking is left unchanged.

**FEAT-12.SPEC-004-AC-03:** Given a booking already marked Completed by Talia before the sweep pass reaches it, when the sweep evaluates it, then it is skipped without effect.

**FEAT-12.SPEC-004-AC-04:** Given a booking already marked No-Show by Talia, when the sweep pass runs, then the booking is left unchanged, since this sweep never marks or overrides a No-Show.

**FEAT-12.SPEC-004-AC-05:** Given a booking is cancelled by the Client at effectively the same moment the sweep would complete it, when the cancellation commits first, then the sweep's eligibility check finds the booking already out of Confirmed/Awaiting Outcome and skips it.

**FEAT-12.SPEC-004-AC-06:** Given the sweep transitions a booking to Completed, when Talia next opens FEAT-12.SPEC-001, then the booking shows as Completed with its balance recorded as settled in person.

**FEAT-12.SPEC-004-AC-07:** Given a sweep pass is interrupted by a processing error partway through, when the failure occurs, then no booking is left in a partially-updated state, and the next scheduled pass picks up every still-eligible booking cleanly.

**FEAT-12.SPEC-004-AC-08:** Given a scheduled sweep pass is still running when the next pass would normally start, when the next scheduled time arrives, then the new pass does not start until the current one finishes.

**FEAT-12.SPEC-004-AC-09:** Given a Pro Account is Paused due to a subscription lapse, when an eligible booking on that account reaches its 7-day mark, then the sweep still completes it, since the pause affects only new bookings and deposits (XBR-14).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Attention Flag Aggregation

## Overview

**Name:** Attention Flag Aggregation
**ID:** FEAT-12.SPEC-005
**Type:** Automation
**Purpose:** Gathers and de-duplicates attention-worthy signals owned by other features (calendar sync health, message delivery, refund progress, card-issuer disputes, setup-change conflicts) into a single Attention List feed, and tracks each item's resolution.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Receiving attention-worthy signals from the owning features' own capabilities (FEAT-04, FEAT-08, FEAT-09, FEAT-16, and the setup features named in XBR-11)
- De-duplicating repeat signals for the same booking or cause so the Pro sees one item, not a flood
- Maintaining each item's open/resolved status as the underlying cause clears
- Feeding the resulting item list to FEAT-12.SPEC-002 (Attention List) for display

**Non-Goals:**
- Diagnosing or resolving the underlying cause (reconnecting a calendar, retrying a refund, resolving a dispute) -- each owned by its source feature (FEAT-04, FEAT-28, FEAT-16 respectively); this automation only surfaces and tracks the flag
- Displaying the aggregated items -- owned by FEAT-12.SPEC-002 (Attention List), which is this automation's sole consumer
- Aggregating waitlist demand -- excluded per this feature's own Side-Effect Inventory: waitlist demand is shown as a separate informational item directly by FEAT-12.SPEC-002, not routed through this de-duplication pipeline, since it is a standing count rather than a discrete resolvable cause
- Notifying the client about any of these attention causes -- each source feature owns its own client-facing notifications (e.g., FEAT-08 for delivery, FEAT-16 for dispute evidence requests); this automation's output is Pro-facing only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Calendar sync health degrades | FEAT-04.SPEC-003 (Two-Way Calendar Sync -- Integration spec) | Fires when a Pro's Calendar Connection status changes to Needs Reconnection or Disconnected (XBR-13) | Pro Account reference, Calendar Connection status, time of status change |
| Message delivery gap after retry and fallback | FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Fires when a text delivery fails, is retried once, and falls back to email, per XBR-17 -- the fallback itself is the signal, not a single first-attempt failure | Booking reference, Message type and channel, delivery_status, time of the fallback |
| Refund cannot complete immediately | FEAT-09.SPEC-005 (Cancellation & No-Show Policy Engine -- refund Integration spec) | Fires when an automatic full deposit refund (client cancellation outside the window, or any Pro cancellation) cannot complete immediately and is reported back for retry, per XBR-10 | Booking reference, Deposit Transaction reference, amount, retry status |
| Card-issuer dispute notice received | FEAT-16.SPEC-003 (Booking & Payment Activity Record -- dispute Integration spec) | Fires when a card-issuer dispute notice is received for a captured deposit, per XBR-22 | Booking reference, Deposit Transaction reference (now Disputed), dispute reference, time received |
| Setup change conflicts with an existing booking | FEAT-01, FEAT-02, FEAT-17, FEAT-18, or FEAT-27 (whichever setup feature made the change) | Fires when a change to hours, a new time block, an archived service, a Pro pause, or a subscription lapse leaves an existing confirmed booking outside the Pro's now-current setup, per XBR-11 -- the confirmed booking itself is never altered by the change | Booking reference, the setup change's kind (hours / block / archived service / pause / subscription lapse), time of the change |

## Processing Logic

1. Receive an incoming signal from one of the five trigger sources above, carrying the affected booking or account reference, the signal's cause category, and a timestamp.
2. Check whether an existing, unresolved Attention Item already exists for the same booking (or account, for a calendar-sync signal) and the same cause category.
3. If a matching unresolved item exists, update its last-seen timestamp rather than creating a duplicate -- the Pro continues to see one item for that cause.
4. If no matching unresolved item exists, create a new Attention Item recording: cause category, affected booking or account reference, first-detected time, and current status (Open).
5. For every open Attention Item, periodically re-check whether its underlying cause has cleared (calendar reconnected, delivery ultimately delivered, refund completed, dispute closed by the processor, or the conflicting booking resolved by an explicit Pro choice through FEAT-30) by consulting the same source feature's current state.
6. When a cause has cleared, mark the corresponding Attention Item Resolved and record the resolution time; a resolved item drops off the active Attention List (FEAT-12.SPEC-002) but its resolution is available in the booking's own activity history (FEAT-16), not restated here.
7. Feed the current set of Open Attention Items to FEAT-12.SPEC-002 on every request from that screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New attention item created | A signal arrives with no existing unresolved item for the same booking/account and cause | New Attention Item recorded, status Open | A new card appears on the Attention List the next time the Pro opens it | FEAT-12.SPEC-002 |
| Repeat signal de-duplicated | A signal arrives matching an already-open item for the same booking/account and cause | Existing item's last-seen timestamp updated; no new item created | No change visible to the Pro -- the existing card remains as-is | FEAT-12.SPEC-002 |
| Attention item resolved | The underlying cause is found cleared on a periodic re-check | Item status -> Resolved, resolution time recorded | The card is removed from the active Attention List on the Pro's next view | FEAT-12.SPEC-002 |
| No action (cause already resolved before first check) | A signal's underlying cause clears before this automation's next re-check runs | No open item is ever shown, or an already-open item resolves on the very next check | The Pro may never see the item at all if it clears within the same check interval it was created in, or sees it briefly then see it clear | FEAT-12.SPEC-002 |
| Aggregation failure | This automation cannot process an incoming signal (e.g., an internal processing error) | The signal is not lost -- the source feature's own record (Calendar Connection status, Message delivery_status, Deposit Transaction status, dispute record, or the conflicting booking's own flag) remains the source of truth and is re-read on the next periodic re-check, so the item still surfaces once processing succeeds | No immediate feedback; the item appears on the Attention List once this automation successfully processes the underlying signal on a later pass | FEAT-12.SPEC-002 |

## Data Model

**Reads:** Calendar Connection (status), Message (delivery_status), Deposit Transaction (status, amount), Booking (reference, state), and the setup-change record from whichever of FEAT-01/FEAT-02/FEAT-17/FEAT-18/FEAT-27 raised the conflict -- read-only, per the dependency map's "Referenced Entities" list for this feature.
**Creates:** Attention Item entries (a record owned by this feature, not part of the shared Domain Entity Inventory) -- each with cause category, affected booking or account reference, first-detected time, and status.
**Updates:** Attention Item entries -- last-seen timestamp (on a de-duplicated repeat signal) and status/resolution time (on resolution).
**Deletes:** None -- resolved items are marked Resolved and removed from the active list view, not deleted; the dependency map's Booking/Deposit Transaction/Message records they reference are never deleted either.

## Business Rules

- One Attention Item per distinct (booking or account, cause category) pair while the cause remains open -- repeat signals for the same pair never create a second card (Side-Effect Inventory: "de-duplicate repeat signals for the same booking/cause").
- This automation never alters the underlying entity it reads from (Calendar Connection, Message, Deposit Transaction, Booking, or a setup record) -- it only observes and reflects their state; every write to those entities is owned by their respective feature.
- A setup-change conflict (XBR-11) never causes this automation, or any feature, to silently cancel the affected booking -- the booking is only ever changed by an explicit Pro choice through FEAT-30, and until that choice is made the Attention Item stays open.
- Resolution is derived, not asserted by the Pro directly on this screen -- an item resolves because its source feature's state changed (e.g., the Pro reconnected the calendar through FEAT-04), not because the Pro dismissed the card here.
- This automation's output is Pro-only; it never surfaces to the Client or to any external party.

## Edge Cases

- **Two different causes arrive for the same booking at once (e.g., a message delivery gap and a dispute notice)** -- Two separate Attention Items are created, one per cause category, since de-duplication is scoped to (booking/account, cause) pairs, not to the booking alone.
- **The same cause fires again for the same booking after its prior item was already resolved** -- A new Attention Item is created (not treated as a duplicate of the resolved one), since de-duplication only suppresses repeats of a currently-open item.
- **Concurrent trigger firing (a calendar-sync degradation and a message-delivery fallback signal for the same account arrive at effectively the same time)** -- Each signal is processed independently against its own cause category; both can result in new Attention Items in the same pass with no interference between them, since the de-duplication check is scoped per (booking/account, cause) pair.
- **Trigger fires while a previous aggregation pass for the same (booking/account, cause) pair is still in flight** -- The second signal's de-duplication check waits for the first to finish creating or updating its item, then finds that item and updates its last-seen timestamp rather than racing to create a second item for the same pair.
- **A setup-change conflict clears because the Pro's explicit choice (via FEAT-30) resolves the conflicting booking, but this automation's periodic re-check has not yet run** -- The Attention Item remains visible until the next re-check confirms resolution; it is never resolved purely by the passage of time without the underlying state actually changing.
- **The underlying source feature itself is degraded (e.g., FEAT-04's own sync is down) when this automation tries to re-check for resolution** -- The Attention Item stays Open (the safer default) until a successful re-check confirms the cause has actually cleared; it is never auto-resolved on an inconclusive check.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Two-Way Calendar Sync) | Triggered by (inbound) | Calendar sync health degradation feeds a new or repeat attention signal |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | A delivery gap after retry and fallback feeds an attention signal |
| FEAT-09.SPEC-005 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | A refund that cannot complete immediately feeds an attention signal |
| FEAT-16.SPEC-003 (Booking & Payment Activity Record) | Triggered by (inbound) | A card-issuer dispute notice feeds an attention signal |
| FEAT-01, FEAT-02, FEAT-17, FEAT-18, FEAT-27 (setup features) | Triggered by (inbound) | A setup change that conflicts with an existing booking feeds an attention signal (XBR-11) |
| FEAT-12.SPEC-002 (Attention List) | Affects (outbound) | Supplies the current set of Open Attention Items for display |
| FEAT-30 (Pro Booking Management) | References (outbound) | The explicit Pro choice that ultimately resolves a setup-change conflict is made there, not on this feature |

## Analytics and Success Signals

- **attention_item_created** (cause_category) -- N/A -- reason: item creation is a Pro-facing signal reflected as the `attention_item_resolved` event's counterpart, but success-metrics.md defines no metric tracking how often attention items arise (only how the Pro's glance experience and resolution speed perform); tracked here for completeness so the create/resolve pair is not silently one-sided.
- **attention_item_resolved** (cause_category, time_to_resolution) -- supports success-metrics.md: "Daily Dashboard Glance Speed" (a resolved item reflects that the Pro's glance-and-act workflow surfaced and cleared something needing attention, which is part of what makes the daily glance trustworthy and complete)

## Acceptance Criteria

**FEAT-12.SPEC-005-AC-01:** Given Talia's Calendar Connection status changes to Needs Reconnection, when this automation processes the signal, then a new Attention Item is created for her account with cause "Reconnect calendar."

**FEAT-12.SPEC-005-AC-02:** Given a message to a client for a booking fails, is retried, and falls back to email (XBR-17), when this automation processes the fallback signal, then a new Attention Item is created for that booking with cause "Message delivery gap."

**FEAT-12.SPEC-005-AC-03:** Given an automatic refund for a booking cannot complete immediately, when this automation processes the signal, then a new Attention Item is created for that booking with cause "Refund in progress."

**FEAT-12.SPEC-005-AC-04:** Given a card-issuer dispute notice is received for a booking's deposit, when this automation processes the signal, then a new Attention Item is created for that booking with cause "Card-issuer dispute."

**FEAT-12.SPEC-005-AC-05:** Given Talia changes her working hours in a way that leaves an existing confirmed booking outside her new hours, when this automation processes the resulting conflict signal, then a new Attention Item is created for that booking with cause "Booking outside changed hours," and the booking itself is left unchanged.

**FEAT-12.SPEC-005-AC-06:** Given an Attention Item is already open for a booking's message-delivery gap, when a second delivery-gap signal arrives for the same booking, then no second item is created -- the existing item's last-seen time is updated instead.

**FEAT-12.SPEC-005-AC-07:** Given Talia reconnects her calendar through FEAT-04, when this automation's next periodic re-check runs, then the "Reconnect calendar" Attention Item is marked Resolved and no longer appears on FEAT-12.SPEC-002.

**FEAT-12.SPEC-005-AC-08:** Given a refund-in-progress Attention Item exists and the refund later completes successfully, when the next re-check runs, then that item is marked Resolved.

**FEAT-12.SPEC-005-AC-09:** Given two different causes (a dispute and a delivery gap) arise for the same booking at effectively the same time, when this automation processes both signals, then two separate Attention Items are created, one per cause.

**FEAT-12.SPEC-005-AC-10:** Given a setup-change conflict's underlying cause has not actually cleared, when a periodic re-check runs while the source feature is temporarily degraded and cannot confirm status, then the Attention Item remains Open rather than being resolved on an inconclusive check.

**FEAT-12.SPEC-005-AC-11:** Given this automation fails to process an incoming dispute signal due to an internal error, when the underlying Deposit Transaction remains Disputed, then the Attention Item is still created once a later processing pass succeeds -- the signal is not permanently lost.

**FEAT-12.SPEC-005-AC-12:** Given a previously resolved "Message delivery gap" Attention Item exists for a booking and a new delivery gap occurs for that same booking later, when this automation processes the new signal, then a new Attention Item is created rather than being treated as a duplicate of the resolved one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Booking Completion Rules

## Overview

**Name:** Booking Completion Rules
**ID:** FEAT-12.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs when a Booking may be marked Completed (by the Pro or automatically), and how completion interacts with the booking's remaining lifecycle actions.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (the `state` field's transition into `Completed`, and the conditions gating that transition)

## Scope and Non-Goals

**In Scope:**
- Eligibility conditions for marking a Booking Completed, whether Pro-initiated or automatic
- The timing window for automatic completion
- How a completed state interacts with cancellation, reschedule, and refund eligibility (XBR-12)
- Authorization for the mark-completed action, per role
- What happens when completion is attempted against a Booking in an ineligible state

**Non-Goals:**
- The mark-completed interaction's on-screen presentation -- owned by FEAT-12.SPEC-001 (Today's & Upcoming Schedule), which enforces this spec's rules
- The 7-day sweep's trigger scheduling and processing steps -- owned by FEAT-12.SPEC-004 (Auto-Completion Sweep), which enforces this spec's eligibility window
- Deriving the balance-due amount shown alongside a completed booking -- owned by FEAT-12.SPEC-007 (Balance Due & Status Display Rules)
- Taking an in-app balance payment at completion -- excluded per scope-boundaries.md SC-16: the balance is deliberately settled in person, off-platform, at MVP; this spec only records that the balance was settled in person, never a payment transaction
- No-show marking and its own 24-hour undo window -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture); this spec only notes that a No-Show state closes off completion eligibility

## Governed Entity

**Entity:** Booking
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | text (reference) | The Service this booking is for; fixed at booking |
| start_time | date/time | Appointment start, in the Pro's timezone; fixed at booking |
| duration | number | Appointment length in minutes; fixed at booking |
| client | reference | The Client this booking belongs to |
| price_agreed | number | Price agreed at booking time |
| deposit_amount | number | Deposit agreed at booking time |
| policy_version | reference | Cancellation policy version shown and acknowledged at booking |
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) |
| attendance_reply | enum | "I'll be there" / reschedule requested, from reminders |
| balance_due | derived | price_agreed − deposit_amount − any in-app balance payment |
| source | enum | Client link, Pro booked-in, recurring occurrence |
| cancellation / reschedule timestamps | date/time (+ optional text) | Timestamps and an optional private Pro reason for a cancellation or reschedule |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On the "mark completed" quick action -- checked immediately when the Pro taps the action, before the write is attempted |
| FEAT-12.SPEC-004 | Auto-Completion Sweep | On each scheduled sweep pass -- checked for every Booking still in Confirmed or Awaiting Outcome state |
| FEAT-30 (Pro Booking Management) | Pro-initiated cancel/reschedule | Consulted (not owned here) to determine whether a Booking's Completed state closes off cancel/reschedule eligibility |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | No-show marking | Consulted (not owned here) to determine whether a Booking already marked Completed is ineligible for no-show marking |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| state | May transition to `Completed` only from `Confirmed` or `Awaiting Outcome`, and only once `start_time` has passed | Always, for both the Pro-initiated and automatic paths | On mark-completed attempt (screen) and on each sweep pass (automation) | "This appointment can't be marked completed yet -- it hasn't started." (before start_time) / "This booking can no longer be marked completed." (state is already Completed, No-Show, Cancelled, Rescheduled, Expired, or Pending Payment) | Yes |
| start_time | No validation beyond data type -- read as the eligibility gate for completion; the field itself is set and validated at booking time by FEAT-05, FEAT-21, or FEAT-30 | -- | -- | -- | -- |
| duration | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-01 | Always | -- | -- | -- |
| service | No validation beyond data type -- not governed by this spec; owned by FEAT-01/FEAT-05 | Always | -- | -- | -- |
| client | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-30 | Always | -- | -- | -- |
| price_agreed | No validation beyond data type -- not governed by this spec; owned by FEAT-07 | Always | -- | -- | -- |
| deposit_amount | No validation beyond data type -- not governed by this spec; owned by FEAT-07 | Always | -- | -- | -- |
| policy_version | No validation beyond data type -- not governed by this spec; owned by FEAT-09 | Always | -- | -- | -- |
| attendance_reply | No validation beyond data type -- not governed by this spec; owned by FEAT-08 | Always | -- | -- | -- |
| balance_due | No validation beyond data type -- derivation owned by FEAT-12.SPEC-007; this spec only consumes the state transition that causes the balance to be recorded as settled in person | On completion, the settled-in-person status is recorded (see Business Rules) | On mark-completed (screen) and on sweep completion (automation) | -- | -- |
| source | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-30/FEAT-21 | Always | -- | -- | -- |
| cancellation / reschedule timestamps | No validation beyond data type -- not governed by this spec; owned by FEAT-10/FEAT-30 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Completion eligibility window | state, start_time | `state` may become `Completed` only when the current time is strictly after `start_time` AND `state` is currently `Confirmed` or `Awaiting Outcome` | "This appointment can't be marked completed yet -- it hasn't started." |
| Auto-completion window | state, start_time | If `state` remains `Confirmed` or `Awaiting Outcome` 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` with no Pro action, the Auto-Completion Sweep (FEAT-12.SPEC-004) transitions `state` to `Completed` automatically | N/A -- automatic transition, no user-facing error |
| Completion closes cancel/reschedule/no-show eligibility | state | Once `state` is `Completed`, FEAT-30's cancel/reschedule actions and FEAT-11's no-show action are no longer available for this Booking (XBR-12) | Owned by FEAT-30/FEAT-11: "This booking is already completed and can no longer be changed." |
| Completion closes goodwill-refund eligibility | state | A goodwill refund (owned by FEAT-30) remains available only until `state` becomes `Completed` (XBR-12) | Owned by FEAT-30: goodwill refund action is not offered once the booking is Completed |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Mark a booking Completed (manual) | The Pro | Only bookings belonging to the requesting Pro's own account, in `Confirmed` or `Awaiting Outcome` state, with `start_time` in the past | Mark-completed control is not shown on a booking outside this Pro's account (never reachable -- FEAT-12.SPEC-008 scopes the dashboard to the Pro's own schedule); when shown but the eligibility condition fails, the control is disabled and tapping it (e.g. via a stale screen) shows "This appointment can't be marked completed yet -- it hasn't started." or "This booking can no longer be marked completed." depending on which condition failed |
| Mark a booking Completed (manual) | Platform Operator (Support) | Never | Mark-completed control is not shown to Support; Support's view is read-only per SC-05 |
| Mark a booking Completed (manual) | The Client | Never | The Client has no access to this dashboard at all (per FEAT-12.SPEC-008); the action is never reachable |
| Trigger the automatic 7-day completion sweep | No human role -- the sweep (FEAT-12.SPEC-004) runs on a schedule with no user-initiated trigger | Always, subject to the Cross-Field Rules eligibility window | N/A -- there is no denial path for a system-triggered action |
| View a booking's completion state | The Pro | Own bookings only | -- |
| View a booking's completion state | Platform Operator (Support) | The one Pro account under active support review, read-only (XBR-24); never the Pro's private client note | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state (on completion) | Set to `Completed` | When either the Pro's mark-completed action or the Auto-Completion Sweep (FEAT-12.SPEC-004) succeeds | No -- the Pro chooses whether to act before the automatic window closes, but cannot set an arbitrary state directly |
| balance settled-in-person flag | Implicitly recorded the moment `state` becomes `Completed` (whether Pro-marked or auto-completed), meaning the balance shown by FEAT-12.SPEC-007 is understood to have been collected off-platform | On completion (either path) | No -- this is not a separate payment record at MVP (SC-16); FEAT-22 (In-App Balance Payment, v1) is the only path that would record an actual balance payment |

## Business Rules

- A Booking can be marked Completed only after its `start_time` has passed; there is no requirement that the full `duration` has elapsed (XBR-12).
- If the Pro takes no action, the Auto-Completion Sweep (FEAT-12.SPEC-004) transitions the Booking to `Completed` automatically 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` (XBR-12).
- Marking a Booking Completed records the balance as settled in person; it never triggers an in-app balance charge (SC-16). Once FEAT-22 (In-App Balance Payment) exists, FEAT-12.SPEC-007 will read from it, but this spec's completion mechanics do not change.
- A Booking already in `Completed`, `No-Show`, `Cancelled by Client`, `Cancelled by Pro`, `Rescheduled`, `Expired (unpaid)`, or `Pending Payment` cannot be marked Completed again or for the first time by either path; the completion attempt is refused.
- A `Completed` or `No-Show` Booking can no longer be cancelled or rescheduled (XBR-12) -- FEAT-30 and FEAT-10 enforce this on their own actions by reading `state`; this spec is the source of truth for when `state` reaches `Completed`.
- A goodwill refund on the Booking's deposit (owned by FEAT-30) remains available only until the Booking reaches `Completed` (XBR-12).
- Contention resolution follows the dependency map's Booking entity note: reject-with-refresh -- the first committed state transition wins, and any other actor (the Pro on a second device, the Auto-Completion Sweep, or a client-initiated cancellation) sees the Booking's current state and must re-decide rather than having transitions merged.
- This spec does not govern no-show marking's own 24-hour undo window (FEAT-11) or cancellation/reschedule eligibility windows (FEAT-09, FEAT-10); it governs only the point at which those windows close because the Booking has become Completed.
- Every transition of `state` into `Completed`, by either path, is a booking-outcome event that FEAT-25.SPEC-004 (Insights Aggregates) consumes to update its insights aggregates; FEAT-25.SPEC-004 reads the resulting state and never writes it, so this spec's eligibility rules are unaffected.
- Completion is a one-way transition: the Access Matrix and product definition provide no undo action for a Completed booking (unlike No-Show, which FEAT-11 allows undoing for a limited window).

## Edge Cases

- **Pro taps "mark completed" at the exact instant of start_time** -- The condition requires the current time to be strictly after `start_time`; at the exact instant, the action is refused with "This appointment can't be marked completed yet -- it hasn't started." A retry a moment later succeeds.
- **Pro attempts to mark completed a Booking that the Auto-Completion Sweep already completed moments earlier** -- The Pro's screen reflects the current `Completed` state on next load or refresh (per the dependency map's reject-with-refresh resolution); the mark-completed control is no longer shown, since the Booking is already Completed.
- **Client cancels their own Booking (via FEAT-10) at the same moment the Pro taps mark-completed** -- First committed transition wins. If the cancellation commits first, the Pro's mark-completed attempt is refused with "This booking can no longer be marked completed." and the Pro's screen refreshes to show the Cancelled state. If completion commits first, the Client's cancellation attempt is refused by FEAT-10 with its own current-state message.
- **Auto-Completion Sweep runs while the Pro is actively viewing the booking on the dashboard** -- The dashboard reflects the new `Completed` state on its next refresh (FEAT-12.SPEC-001); no destructive action is taken against any in-flight Pro interaction, since the sweep only ever moves a Booking forward from `Confirmed`/`Awaiting Outcome` to `Completed`.
- **Booking is in `Awaiting Outcome` exactly at the 7-day boundary** -- The sweep evaluates at each scheduled pass (FEAT-12.SPEC-004); a Booking that crosses the boundary between passes is completed on the next pass that finds it still eligible, not the instant the boundary is crossed.
- **A Booking that was never confirmed (still `Pending Payment` or already `Expired (unpaid)`) reaches its 7-day mark** -- Never eligible for completion by either path, since eligibility requires `Confirmed` or `Awaiting Outcome` as the starting state; it remains in its own terminal state.

## Acceptance Criteria

**FEAT-12.SPEC-006-AC-01:** Given Talia has a Confirmed booking whose start_time has passed, when she taps "mark completed" on FEAT-12.SPEC-001, then the booking's state transitions to Completed and the balance is recorded as settled in person.

**FEAT-12.SPEC-006-AC-02:** Given Talia has a Confirmed booking whose start_time has not yet arrived, when she attempts to mark it completed, then the action is refused with "This appointment can't be marked completed yet -- it hasn't started." and the state does not change.

**FEAT-12.SPEC-006-AC-03:** Given Talia has a booking already in Completed state, when she looks at it on the dashboard, then no mark-completed control is shown for it.

**FEAT-12.SPEC-006-AC-04:** Given Talia has a booking already marked No-Show, when she attempts to mark it completed, then the action is refused with "This booking can no longer be marked completed."

**FEAT-12.SPEC-006-AC-05:** Given a Confirmed booking's start_time passed 7 days ago and Talia never acted on it, when the Auto-Completion Sweep (FEAT-12.SPEC-004) runs, then the booking's state transitions to Completed automatically.

**FEAT-12.SPEC-006-AC-06:** Given a Confirmed booking whose start_time passed only 2 days ago, when the Auto-Completion Sweep runs, then the booking is left unchanged because it has not yet reached the 7-day window.

**FEAT-12.SPEC-006-AC-07:** Given a booking has been marked Completed, when Talia opens FEAT-30 (Pro Booking Management) for that booking, then no cancel or reschedule action is available for it, per XBR-12.

**FEAT-12.SPEC-006-AC-08:** Given a booking has been marked Completed, when Talia looks for a goodwill refund option on that booking, then none is offered, per XBR-12.

**FEAT-12.SPEC-006-AC-09:** Given Talia (the Pro) is viewing her own dashboard, when she marks an eligible booking completed, then the action succeeds, since only the Pro may perform this action on her own bookings.

**FEAT-12.SPEC-006-AC-10:** Given Platform Operator (Support) is viewing a Pro's account during a support session, when Support looks for a mark-completed control, then none is shown, since Support's access is read-only (SC-05).

**FEAT-12.SPEC-006-AC-11:** Given a Client attempts to reach this dashboard, when the access check runs, then the mark-completed action is never reachable, because Clients have no access to this dashboard at all (FEAT-12.SPEC-008).

**FEAT-12.SPEC-006-AC-12:** Given Talia's booking is simultaneously cancelled by the Client (FEAT-10) and marked completed by Talia at effectively the same moment, when the cancellation commits first, then Talia's mark-completed attempt is refused with "This booking can no longer be marked completed." and her dashboard refreshes to show the cancelled state.

**FEAT-12.SPEC-006-AC-13:** Given a booking's start_time is exactly the current instant, when Talia attempts to mark it completed at that exact moment, then the action is refused because the transition requires the current time to be strictly after start_time.

**FEAT-12.SPEC-006-AC-14:** Given a booking is still in Pending Payment state, when the Auto-Completion Sweep evaluates it after the 7-day window, then the booking is left unchanged because it never reached Confirmed or Awaiting Outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 12 | 12 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 9 | 9 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Balance Due & Status Display Rules

## Overview

**Name:** Balance Due & Status Display Rules
**ID:** FEAT-12.SPEC-007
**Type:** Logic/Rule
**Purpose:** Derives the balance-due amount and the paid/unpaid, "I'll be there," and sync-reliability display states shown consistently on every booking row across this feature's three screens.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (balance_due derivation and attendance_reply display), plus Deposit Transaction (paid/unpaid status display) and Calendar Connection (sync-reliability display) as read-only inputs to the same derived booking-row presentation

## Scope and Non-Goals

**In Scope:**
- Deriving `balance_due` from Booking and Deposit Transaction data (XBR-23)
- Deriving the paid/unpaid badge shown on every booking row from Deposit Transaction status
- Deriving the "I'll be there" attendance display from Booking's `attendance_reply`
- Deriving the per-booking sync-reliability marking from Calendar Connection status
- The exact display conventions (badge wording, never color-alone) so SPEC-001 and SPEC-003 render the shared booking row identically

**Non-Goals:**
- Whether a Booking may be marked Completed -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this spec only derives what is displayed once a state is reached
- Who may view a booking at all -- owned by FEAT-12.SPEC-008 (Dashboard Access Authorization); this spec assumes the viewer is already authorized and only governs what is shown once visible
- Collecting or recording the deposit or balance payment itself -- owned by FEAT-07 (Deposit Payment at Booking) and FEAT-22 (In-App Balance Payment, v1, not yet built at MVP); this spec only reads their recorded outcomes for display
- Diagnosing or resolving a calendar sync failure -- owned by FEAT-04 (Two-Way Calendar Sync); this spec only reads the reported `status` field to decide how a booking's reliability marking reads

## Governed Entity

**Entity:** Booking (primary), with read-only display inputs from Deposit Transaction and Calendar Connection
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | Booking -- price agreed at booking time |
| deposit_amount | number | Booking -- deposit agreed at booking time |
| balance_due | derived | Booking -- price_agreed − deposit_amount − any in-app balance payment |
| attendance_reply | enum | Booking -- "I'll be there" / reschedule requested, from reminders, or no reply yet |
| state | enum | Booking -- current lifecycle state, read here only to decide whether balance/paid display still applies |
| deposit_transaction.status | enum | Deposit Transaction -- Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed |
| deposit_transaction.amount | number | Deposit Transaction -- the captured deposit amount |
| calendar_connection.status | enum | Calendar Connection -- Connected \| Syncing \| Needs Reconnection \| Disconnected |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On every booking row render, for today's and upcoming bookings |
| FEAT-12.SPEC-003 | Past Bookings Browse | On every booking row render, for past bookings |
| FEAT-12.SPEC-002 | Attention List | On the underlying booking reference shown inside a sync-reliability or dispute attention item |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| price_agreed | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| deposit_amount | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| balance_due | Must equal price_agreed − deposit_amount − any in-app balance payment; never displayed as a negative amount | Always (derivation is display-only, not user input) | On every row render | If the derivation would be negative, display as fully settled (see Business Rules) rather than a negative figure | No -- this is a display derivation, not a user-input validation |
| attendance_reply | No validation beyond data type -- this spec only reads it for the display label | Always | -- | -- | -- |
| state | No validation beyond data type -- read only to gate whether balance/paid display still applies (e.g., a Cancelled booking shows no balance-due badge) | Always | -- | -- | -- |
| deposit_transaction.status | No validation beyond data type -- this spec only reads it to select the paid-badge wording | Always | -- | -- | -- |
| deposit_transaction.amount | No validation beyond data type -- this spec only reads it for derivation | Always | -- | -- | -- |
| calendar_connection.status | No validation beyond data type -- this spec only reads it to select the sync-reliability wording | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Balance-due formula | price_agreed, deposit_amount, balance_due | balance_due = price_agreed − deposit_amount − any in-app balance payment (XBR-23); at MVP, with no in-app balance payment capability, balance_due = price_agreed − deposit_amount | N/A -- derived value, no user-facing error |
| Paid-badge wording | deposit_transaction.status, state | If deposit_transaction.status is Captured or Applied, the badge reads "Paid" (with a word, never color alone, per ASMP-28); if Refund in Progress, the badge reads "Refund in progress"; if Refunded, the booking is no longer shown as a balance-due item since the booking itself is Cancelled; if Disputed, the badge reads "Paid" with a separate dispute flag surfaced through the Attention List (FEAT-12.SPEC-005), never replacing the paid badge itself | N/A -- display-only |
| Attendance display | attendance_reply, state | If attendance_reply is "I'll be there," the row shows that label; if it is a reschedule request, the row shows "Asked to reschedule" and links to FEAT-10's context (no direct action taken here); if no reply has been received, the row shows no attendance label at all (absence of a label is intentional, not an error state) | N/A -- display-only |
| Sync-reliability marking | calendar_connection.status | If Connected or Syncing, the booking's reliability marking reads normally (no special marking); if Needs Reconnection or Disconnected, the booking is marked "Reliability uncertain" per FEAT-12.SPEC-005's aggregation, since the calendar connection cannot currently confirm this time is conflict-free | N/A -- display-only |
| Balance-due suppressed on terminal non-completed states | state, balance_due | If state is Cancelled by Client, Cancelled by Pro, No-Show, Rescheduled, or Expired (unpaid), the balance-due figure is not shown as an outstanding amount on the row -- these states have their own status display (owned by FEAT-09, FEAT-10, FEAT-11, FEAT-30) and this spec defers to them rather than showing a stale balance-due badge | N/A -- display-only |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking on own bookings | The Pro | Own bookings only | -- |
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking | Platform Operator (Support) | The one Pro account under active support review, read-only (XBR-24) -- masked the same way as the Pro's own view since none of these derived values are private client-note content | -- |
| View derived balance-due, paid badge, attendance reply, and sync-reliability marking | The Client | Never on this feature's screens | The Client has no access to this dashboard at all (FEAT-12.SPEC-008); their own balance/paid status is shown through FEAT-06 instead |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| balance_due | price_agreed − deposit_amount − any in-app balance payment (XBR-23); floors at zero for display -- never shown as negative | Recomputed on every render | No -- this is a read-only derived display value |
| paid-badge label | Selected from deposit_transaction.status per the Cross-Field Rules table above | Recomputed on every render | No |
| attendance label | Selected from attendance_reply per the Cross-Field Rules table above | Recomputed on every render | No |
| sync-reliability marking | Selected from calendar_connection.status per the Cross-Field Rules table above, sourced through FEAT-12.SPEC-005's aggregation | Recomputed on every render | No |

## Business Rules

- Balance due is always computed, never stored as a separately editable field: price_agreed − deposit_amount − any in-app balance payment (XBR-23). At MVP, with FEAT-22 (In-App Balance Payment) not yet built, the formula simplifies to price_agreed − deposit_amount.
- If the computed balance_due would be zero or negative (e.g., the deposit equals or exceeds the price, or a balance payment already covers the remainder once FEAT-22 exists), the row shows the booking as fully settled rather than a zero or negative figure.
- Paid/unpaid and attention status are never conveyed by color alone -- every badge carries a word alongside any color treatment (ASMP-28 Accessibility baseline).
- A Disputed Deposit Transaction never overwrites or hides the underlying paid status on the booking row; the dispute is a separate flag raised through the Attention List (FEAT-12.SPEC-002, fed by FEAT-12.SPEC-005), consistent with XBR-22's rule that a dispute overlay never erases the underlying outcome.
- Sync-reliability marking never claims false confidence: if the calendar connection cannot currently confirm a time is genuinely free of external conflicts, the booking is marked uncertain rather than shown as reliable (XBR-13; this feature's own Side-Effect Inventory entry "A booking's calendar-sync status is uncertain").
- Messaging Consent (textability) is displayed as a read-only status alongside the booking row, per FEAT-12.SPEC-001 -- this spec does not compute or alter it; it is read directly from Messaging Consent's `state` field.
- Card data is never part of any derived display value on this feature's screens (ASMP-15, SC-11); only amounts and statuses are shown.
- These derivation rules apply identically wherever the shared booking row pattern is used -- FEAT-12.SPEC-001's today/upcoming list and FEAT-12.SPEC-003's past-bookings list -- so a client's balance-due figure and paid badge read the same regardless of which screen shows it.

## Edge Cases

- **Deposit equals the full price (100% deposit rule)** -- balance_due computes to zero; the row shows "Paid in full" rather than a $0 balance-due badge.
- **Booking is Cancelled by Client with the deposit refunded** -- No balance-due badge is shown; the row instead shows the cancellation/refund status owned by FEAT-10/FEAT-09, and this spec's paid badge is not rendered for a cancelled booking.
- **Deposit Transaction is Disputed while the booking is still upcoming** -- The paid badge continues to read "Paid" (the underlying outcome is unchanged); a separate dispute flag appears via the Attention List (FEAT-12.SPEC-002).
- **Calendar Connection status flips from Needs Reconnection back to Connected while the Pro is viewing the schedule** -- The reliability marking updates to reflect the current status on the screen's next refresh (FEAT-12.SPEC-001 defines the refresh behavior); this spec does not itself define polling frequency.
- **Client has not yet replied to a reminder and the appointment is imminent** -- No attendance label is shown; absence of a reply is not treated as a negative signal or rendered as an error state.
- **In-app balance payment exists (post-MVP, FEAT-22) and partially covers the balance** -- balance_due recomputes to price_agreed − deposit_amount − balance payment amount; this spec's formula already accounts for that term, so no rule change is needed when FEAT-22 ships.

## Acceptance Criteria

**FEAT-12.SPEC-007-AC-01:** Given a booking with price_agreed and deposit_amount recorded, when Talia views it on FEAT-12.SPEC-001, then the balance-due figure shown equals price_agreed minus deposit_amount.

**FEAT-12.SPEC-007-AC-02:** Given a booking's deposit equals its full price, when Talia views the row, then it shows "Paid in full" rather than a $0 balance-due badge.

**FEAT-12.SPEC-007-AC-03:** Given a booking's Deposit Transaction status is Captured, when Talia views the row, then the badge reads "Paid" with the word visible alongside any color treatment.

**FEAT-12.SPEC-007-AC-04:** Given a booking's Deposit Transaction status is Refund in Progress, when Talia views the row, then the badge reads "Refund in progress."

**FEAT-12.SPEC-007-AC-05:** Given a booking's Deposit Transaction becomes Disputed, when Talia views the row, then the paid badge still reads "Paid" and a separate dispute flag appears through the Attention List, per XBR-22.

**FEAT-12.SPEC-007-AC-06:** Given a client has tapped "I'll be there" on a reminder, when Talia views that booking's row, then the attendance label shows "I'll be there."

**FEAT-12.SPEC-007-AC-07:** Given a client has not replied to any reminder for a booking, when Talia views that row, then no attendance label is shown.

**FEAT-12.SPEC-007-AC-08:** Given the Pro's Calendar Connection status is Needs Reconnection, when Talia views a booking whose reliability cannot currently be confirmed, then that booking is marked "Reliability uncertain" rather than shown as normally reliable.

**FEAT-12.SPEC-007-AC-09:** Given the Pro's Calendar Connection status is Connected, when Talia views any booking row, then no reliability-uncertain marking appears.

**FEAT-12.SPEC-007-AC-10:** Given a booking is Cancelled by Client, when Talia views its row, then no balance-due badge is shown for it.

**FEAT-12.SPEC-007-AC-11:** Given Talia (the Pro) views her own dashboard, when a booking row renders, then the derived balance, paid, attendance, and reliability values are shown for it, since she is authorized to view her own bookings.

**FEAT-12.SPEC-007-AC-12:** Given Platform Operator (Support) is reviewing a Pro's account, when Support views a booking row, then the same derived balance, paid, attendance, and reliability values are shown as the Pro sees, since none of these values are private client-note content.

**FEAT-12.SPEC-007-AC-13:** Given a Client attempts to reach this dashboard, when the access check runs, then no booking row or derived value is ever shown to them here, since Clients have no access to this feature (FEAT-12.SPEC-008).

**FEAT-12.SPEC-007-AC-14:** Given the same booking appears on both FEAT-12.SPEC-001 (upcoming) and, once past, FEAT-12.SPEC-003 (past bookings), when Talia views it on either screen, then the balance-due figure and paid badge read identically on both.

**FEAT-12.SPEC-007-AC-15:** Given a booking's price_agreed is less than its deposit_amount due to a later price adjustment path that does not exist in this product (hypothetical negative derivation), when the derivation computes a negative value, then the row shows the booking as fully settled rather than a negative balance-due figure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Dashboard Access Authorization

## Overview

**Name:** Dashboard Access Authorization
**ID:** FEAT-12.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces who may open this feature's three screens and what each role sees: the Pro's own schedule in full, Support's masked read-only view for troubleshooting, and a redirect to sign-in for anyone else.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (the entity every screen in this feature reads), gated together with the Client, Deposit Transaction, Message, Time Block, Calendar Connection, Messaging Consent, and Waitlist Entry fields those screens display alongside it

## Scope and Non-Goals

**In Scope:**
- Who may open FEAT-12.SPEC-001, FEAT-12.SPEC-002, and FEAT-12.SPEC-003, and under what identity conditions
- What the Pro's Full access covers versus Support's View-only access on this feature's three screens
- Exactly which fields Support never sees, regardless of role-level View access (private client notes, sign-in codes, card or bank/identity details)
- The unauthenticated and expired-session experience for every screen in this feature
- Confirming that Support's every view is logged, per XBR-24

**Non-Goals:**
- Whether a specific booking action (mark completed) is allowed once the dashboard is open -- owned by FEAT-12.SPEC-006 (Booking Completion Rules)
- Deriving what a booking row displays once visible -- owned by FEAT-12.SPEC-007 (Balance Due & Status Display Rules)
- The Pro sign-in mechanism itself (one-time code, device trust, new-device alerts) -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle); this spec only enforces that a signed-in Pro identity is required
- Support's own audit-log screen or how support sessions are opened/closed -- owned by FEAT-19 (Platform Support Read-Only Access); this spec only enforces the resulting view restrictions on FEAT-12's three screens

## Governed Entity

**Entity:** Booking (primary), gated together with the read-only entities every screen in this feature displays
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client (reference), and via it Client.private_note | reference / text | Client linked to the booking; the Pro's private note about that client |
| service, start_time, duration, price_agreed, deposit_amount, state, attendance_reply, balance_due, source, cancellation/reschedule timestamps | mixed | Booking's own fields (see FEAT-12.SPEC-006 for the full list); visible to the Pro and, masked as below, to Support |
| deposit_transaction.status, amount | enum / number | Deposit Transaction fields shown per booking; card data is never part of this entity (ASMP-15) |
| message.delivery_status | enum | Message delivery-failure flags shown per booking |
| time_block.start / end / label | date/time / text | Manual time blocks shown alongside bookings; `label` is Pro-private |
| calendar_connection.status | enum | Sync-health status; never event titles or details |
| messaging_consent.state | enum | Textability status only |
| waitlist_entry (aggregate count only) | number | Aggregate waitlist demand; never individual entries shown to the Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On screen entry (before any data loads) and on every subsequent data refresh |
| FEAT-12.SPEC-002 | Attention List | On screen entry and on every subsequent data refresh |
| FEAT-12.SPEC-003 | Past Bookings Browse | On screen entry and on every subsequent data refresh |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| client.private_note | Never shown to Platform Operator (Support), regardless of the booking otherwise being visible to Support | When the viewer is Support | On every render that would otherwise include the note preview | The note-preview element is omitted entirely from Support's view (not shown blank, not shown as "hidden" -- simply not present in the layout) | Yes |
| All other Booking, Deposit Transaction, Message, Time Block (except label), Calendar Connection, and Messaging Consent fields listed above | No validation beyond data type -- this spec governs view authorization, not field-level validation of these fields (owned by their originating features) | Always | -- | -- | -- |
| time_block.label | Never shown to Platform Operator (Support) -- Pro-private per the dependency map's Data Sensitivity note for Time Block | When the viewer is Support | On every render | The label is omitted from Support's view of a time block; the block itself (its start/end) still shows as occupied time | Yes |
| Card, bank, or identity details | Never displayed on any of this feature's three screens, for any role | Always | -- | N/A -- these fields are never surfaced here at all (ASMP-15, SC-11); this is a structural exclusion, not a per-role denial | Yes |
| Pro sign-in codes | Never displayed on any of this feature's three screens, for any role including Support | Always | -- | N/A -- sign-in codes are never surfaced here (ASMP-20, XBR-24) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Support masking is uniform across all three screens | client.private_note, time_block.label, sign-in codes, card/bank/identity details | The same masking rules apply identically whether Support is viewing FEAT-12.SPEC-001, FEAT-12.SPEC-002, or FEAT-12.SPEC-003 -- no screen in this feature exposes to Support what another screen in this feature withholds | N/A -- structural consistency rule, no user-facing error |
| Support access is scoped to one Pro account at a time | (account-level, not a Booking field) | Support's view, when active, is scoped entirely to the single Pro account under active review (XBR-24); this spec never allows a cross-account view | N/A -- enforced by FEAT-19, referenced here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Open FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-001 | Platform Operator (Support) | Only while an active support session for that specific Pro account is open, following a Pro help request (XBR-24) | -- |
| Open FEAT-12.SPEC-001 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29; a client is never told whether the destination account exists |
| Open FEAT-12.SPEC-001 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-002 (Attention List) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-002 | Platform Operator (Support) | Only during an active support session for that Pro account | -- |
| Open FEAT-12.SPEC-002 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-002 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-003 (Past Bookings Browse) | The Pro | Signed in as the Pro, viewing their own account only | -- |
| Open FEAT-12.SPEC-003 | Platform Operator (Support) | Only during an active support session for that Pro account | -- |
| Open FEAT-12.SPEC-003 | The Client | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| Open FEAT-12.SPEC-003 | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29) |
| View a client's private note preview on any booking row | The Pro | Own bookings only | -- |
| View a client's private note preview on any booking row | Platform Operator (Support) | Never | The note-preview element is omitted from the row entirely |
| Perform any write action on this feature's three screens (mark completed, resolve an attention item) | Platform Operator (Support) | Never | Every write control is omitted from Support's view; Support's access is strictly View-only (SC-05) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Viewer role for the current session | Derived from the signed-in identity: Pro (own account), Support (active support session on a named Pro account), or neither | On every screen entry in this feature | No -- the viewer cannot elevate their own role from within the screen |
| Masked-field set applied to the current render | Derived from the viewer role above: Pro sees everything owned by the account; Support sees everything except private client notes, time-block labels, sign-in codes, and card/bank/identity details | Recomputed on every render | No |

## Business Rules

- Every screen in this feature requires a signed-in Pro identity (or an active Support session scoped to that Pro), per XBR-29: "every Pro-facing screen requires a signed-in Pro; anyone else is sent to the Pro sign-in screen."
- A failed sign-in never reveals whether a Pro account exists (XBR-29) -- this spec never distinguishes "no such account" from "wrong code" in any redirect behavior on these three screens.
- Support's access is read-only, one account at a time, used only after a Pro's help request, and never shows private client notes, bank or identity details, or sign-in codes (XBR-24) -- this is enforced identically across all three of this feature's screens.
- Every Support view of this feature's screens is logged in the Pro's visible account activity (XBR-24); the Pro can see when Support looked, through FEAT-19's own audit surface (not rendered inside this feature's screens themselves).
- Support cannot act on the Pro's behalf from this dashboard (SC-05): no write control (mark completed, resolve an attention item, navigate into an editing flow on another feature) is shown to Support.
- Calendar Connection data shown on this feature's screens is limited to sync health only, never event titles or details, for either the Pro or Support (product-features.md FEAT-04 Data Notes).
- Messaging Consent is shown as textability status only; neither the Pro nor Support can override a client's consent from this feature's screens (US texting-consent rules, ASMP-24).
- Waitlist demand shown on FEAT-12.SPEC-002 is an aggregate count only, for the Pro; individual waitlist entries are never surfaced here (Access Matrix: Waitlist = View for the Pro), and this rule applies identically to Support's masked view.

## Edge Cases

- **A Pro's sign-in session expires while a screen in this feature is open** -- The screen's own Expired session row (defined per-screen in FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003) applies; this spec's authorization check re-runs on the next data refresh and, finding no valid session, redirects to the Pro sign-in screen (FEAT-29).
- **A support session is closed (or times out) while Support is viewing one of these screens** -- The next authorization check finds no active support session and the screen redirects Support out; no further data loads.
- **Support attempts to reach one of these three screens without an active session tied to a specific Pro help request** -- Access is refused at the authorization check, consistent with XBR-24's "used only after a Pro's help request"; enforcement of when a session may be opened belongs to FEAT-19, but this spec never renders Pro data absent that active session.
- **A Client, signed in through their own client-side access link (FEAT-06), tries to open a Pro-facing URL for this feature** -- A client access link never grants a Pro-facing session; the visitor is treated as unauthenticated for this feature's purposes and redirected to the Pro sign-in screen (FEAT-29), never shown any booking data.
- **The Pro views the dashboard on two signed-in devices at once** -- Both are the same authorized role (the Pro, own account); both see the full, unmasked view. This is a read-scenario, not a write conflict; any write conflict (e.g., both marking the same booking completed) is governed by FEAT-12.SPEC-006, not this spec.
- **A booking's client note is edited by the Pro (FEAT-13) while Support is viewing the same booking's row on this feature** -- Irrelevant to Support, since the private note is never shown to Support in the first place; no visibility change occurs for Support regardless of the underlying edit.

## Acceptance Criteria

**FEAT-12.SPEC-008-AC-01:** Given Talia is signed in as the Pro, when she opens FEAT-12.SPEC-001, then she sees her own schedule in full, with all fields visible.

**FEAT-12.SPEC-008-AC-02:** Given Platform Operator (Support) has an active support session on Talia's account following her help request, when Support opens FEAT-12.SPEC-001, then Support sees the same bookings with the client private-note preview omitted from every row.

**FEAT-12.SPEC-008-AC-03:** Given a visitor is not signed in as any Pro, when they attempt to open FEAT-12.SPEC-001, then they are redirected to the Pro sign-in screen (FEAT-29), per XBR-29.

**FEAT-12.SPEC-008-AC-04:** Given Riley (the Client) attempts to reach FEAT-12.SPEC-001 directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, since Clients have no access to this feature.

**FEAT-12.SPEC-008-AC-05:** Given Platform Operator (Support) is viewing Talia's booking list, when Support looks for a mark-completed or resolve-attention-item control, then none is shown, since Support's access is strictly View-only.

**FEAT-12.SPEC-008-AC-06:** Given Talia's sign-in session expires while FEAT-12.SPEC-002 is open, when the screen's next data refresh runs, then she is redirected to the Pro sign-in screen (FEAT-29).

**FEAT-12.SPEC-008-AC-07:** Given Support's active session on Talia's account is closed, when Support's screen attempts its next refresh, then Support is redirected out and no further Pro data loads.

**FEAT-12.SPEC-008-AC-08:** Given Talia is viewing a booking row on FEAT-12.SPEC-003, when the row renders, then her private note preview for that client is visible to her.

**FEAT-12.SPEC-008-AC-09:** Given Support is viewing the same booking row on FEAT-12.SPEC-003, when the row renders, then no private note preview element appears at all for Support.

**FEAT-12.SPEC-008-AC-10:** Given Support is viewing FEAT-12.SPEC-002, when a time block appears alongside bookings, then its start/end are shown but its private label is omitted.

**FEAT-12.SPEC-008-AC-11:** Given Talia opens FEAT-12.SPEC-002, when the waitlist demand item renders, then only an aggregate count is shown, never individual waitlist entries.

**FEAT-12.SPEC-008-AC-12:** Given a failed sign-in attempt is made against this feature's entry point, when the sign-in fails, then the response never reveals whether a Pro account exists for the attempted identifier, per XBR-29.

**FEAT-12.SPEC-008-AC-13:** Given Talia is signed in on two devices at once, when she opens FEAT-12.SPEC-001 on both, then both show her full, unmasked schedule, since both are the same authorized Pro role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |


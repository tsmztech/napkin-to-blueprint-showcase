---
document_type: feature-overview
feature_number: FEAT-12
feature_name: Pro Daily Schedule Dashboard
feature_slug: pro-daily-schedule-dashboard
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 3
automation_count: 2
logic_rule_count: 3
integration_count: 0
notification_count: 0
---

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

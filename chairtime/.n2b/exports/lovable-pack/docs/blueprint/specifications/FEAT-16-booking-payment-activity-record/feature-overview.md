---
document_type: feature-overview
feature_number: FEAT-16
feature_name: Booking & Payment Activity Record
feature_slug: booking-payment-activity-record
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Booking & Payment Activity Record

## Summary

**Feature:** Booking & Payment Activity Record
**ID:** FEAT-16
**Description:** An always-on, append-only record of every booking's key events -- created, paid, confirmed, messaged, cancelled/rescheduled, marked no-show, refunded/forfeited -- so the Pro (or, when asked to help, Platform Operator Support) has a trustworthy timeline to point to if a client ever disputes a charge.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Problem Statement names this precisely as a current failure: "no record when a client disputes a no-show charge." Ranked Important rather than Core because it is a record-keeping layer that supports the Core booking/deposit/no-show features rather than something a user directly seeks out day to day; it must still ship at MVP because the dispute scenario it prevents is a launch-day risk, not a later refinement. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- View a chronological timeline of everything that happened to a specific booking
- See exactly which cancellation policy version applied and when it was shown to the client
- Reference this record when responding to a client's dispute
- See a booking flagged when the client raises a dispute with their card issuer, and download a plain, shareable summary of the booking's timeline (policy shown and acknowledged, booking time, messages sent, no-show mark) to use as evidence with the payment processor [AUDIT-ADDED: 1 -- counterpart symmetry: a client can contest a kept deposit through their card issuer, not only by messaging the Pro; the Pro needed the record in a form they can submit]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-16.SPEC-001 | Booking Activity Timeline | Screen | The Pro, Platform Operator (Support) | Read-only, chronological view of one booking's full event history -- policy version shown/acknowledged, appointment time, messages, cancellations, no-show mark, gaps shown plainly -- with an entry point to a goodwill refund |
| FEAT-16.SPEC-002 | Activity Event Recording | Automation | All | Writes a single, immutable, append-only Activity Event for every qualifying action across Booking, Deposit Payment, Messaging, Cancellation/Reschedule, No-Show, Client Deletion, Payout, and Pro-initiated cancel/reschedule |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | Integration | The Pro, Platform Operator (Support) | Receives an inbound dispute notice from the payment-processing capability, flags the booking, sets the Deposit Transaction's Disputed overlay, and records the dispute event |
| FEAT-16.SPEC-004 | Dispute Summary Download | Automation | The Pro | Assembles a plain-language, shareable summary of a disputed booking's timeline and hands it to the Pro as a downloadable file to submit to the payment processor |
| FEAT-16.SPEC-005 | Activity Record Immutability & Visibility Rules | Logic/Rule | The Pro, Platform Operator (Support) | Governs append-only enforcement (no role, including the Pro, may edit an entry), View-only access for the Pro and Support, retention tied to the Booking's life, de-identification on client deletion, and exclusion of the Pro's private client notes from the Support view |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View a chronological timeline of everything that happened to a specific booking | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | The screen renders the ordered event list; the automation is what guarantees every qualifying action across five other features actually lands in that list | Phase 2 (Explicit) |
| See exactly which cancellation policy version applied and when it was shown to the client | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | The screen surfaces the policy version and acknowledgment timestamp as a distinct line item; the automation is the one that accepts and stores that event from FEAT-09.SPEC-002 | Phase 2 (Explicit) |
| Reference this record when responding to a client's dispute | FEAT-16.SPEC-001 | The same timeline the Pro reads to prepare her explanation to the client -- no separate view exists for this use | Phase 2 (Explicit) |
| See a booking flagged when the client raises a dispute with their card issuer, and download a plain, shareable summary of the booking's timeline to use as evidence with the payment processor | FEAT-16.SPEC-003, FEAT-16.SPEC-004 | The Integration spec receives the dispute notice and flags the booking; the Automation spec turns the same timeline data into a downloadable evidence summary | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-16.SPEC-002 | Activity Event Recording | Phase 4 (Trigger-Response) | Every Key Capability depends on entries actually existing; the feature's own Lifecycle line ("Created by FEAT-16 ... written automatically as FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11, FEAT-30 act") names a fan-in of eleven distinct inbound writer specs across seven features -- too consequential and too cross-feature to leave as an unstated implementation detail of the screen |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | Phase 4 (External Dependencies lens) | ASMP-31's payment-processing capability explicitly includes "notify the product of card-issuer disputes," and the External Touchpoints row for this integration names FEAT-16 as the feature expected to define it in this batch; an inbound event crossing the product boundary from an external capability is, by the standalone-spec decision rule, an Integration spec |
| FEAT-16.SPEC-005 | Activity Record Immutability & Visibility Rules | Phase 5 (Rule Discovery) | The Access field ("nobody, including the Pro, can edit an entry"), the Validation & Limits field ("append-only and immutable"), and XBR-24's private-notes exclusion for Support are three interacting authorization/visibility rules shared across the screen, the recording automation, and the dispute integration -- past the inline-validation threshold and requiring one shared definition rather than three restatements |

## Entity-Lifecycle Coverage Matrix

**Entity: Activity Event**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-16.SPEC-002 | Writes one immutable entry per qualifying trigger from FEAT-05, FEAT-07.SPEC-002, FEAT-08 (SPEC-001/002/004/005/006/009), FEAT-09.SPEC-002, FEAT-10.SPEC-004, FEAT-11.SPEC-002/003, FEAT-13.SPEC-004, FEAT-28.SPEC-003, FEAT-30.SPEC-007/008, and FEAT-16.SPEC-003's own dispute event | Support-view logging is a second creator of Activity Event, written via FEAT-19.SPEC-002 (Support View Logging) and recorded in FEAT-16.SPEC-002 as an accepted writer (SG-11 decision) |
| Read (single) | N/A | An Activity Event is never opened individually; it is always read as part of a booking's ordered timeline | -- |
| Read (list) | FEAT-16.SPEC-001 | Renders the full ordered event list for one booking, including the Support read-only view | -- |
| Update | N/A | No spec ever updates an Activity Event -- enforced by FEAT-16.SPEC-005 (Validation & Limits field: "entries are append-only and immutable once written") | This is the property that makes the record trustworthy as dispute evidence, not an accidental omission |
| Delete/Archive | FEAT-16.SPEC-002 | Hard delete never occurs. On a client-deletion event (inbound from FEAT-13.SPEC-004), affected entries are converted in place to de-identified form (contact and note content stripped, financial and timeline facts retained) per XBR-19; no restore path exists because the conversion is a one-directional, legally-required redaction, not a reversible action. While the associated Booking exists and no client deletion has occurred, entries are retained indefinitely (SC-22) | -- |
| State Transition | N/A | An Activity Event carries no state beyond its fixed fields at creation -- it is a point-in-time fact, not a stateful record | -- |

**Deposit Transaction (narrow update, not fully managed by this feature):**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-07 (Deposit Payment at Booking) -- this feature never creates a transaction | -- |
| Read (single) | FEAT-16.SPEC-001, FEAT-16.SPEC-003 | The timeline reads status and outcome to display outcomes; the dispute integration reads current status before applying the Disputed overlay | -- |
| Read (list) | N/A | This feature produces no list view of Deposit Transactions -- that is FEAT-28's money list | -- |
| Update | FEAT-16.SPEC-003 | Sets a Disputed overlay on card-issuer dispute notice. Per the Requirements Architect's coordination note, this is treated as this feature's single, narrowly-scoped update: the dependency map's Deposit Transaction Lifecycle line and XBR-22 (authority: FEAT-16) both assign the Disputed transition to this feature, even though the Stage 2 Connected Entities list marks Deposit Transaction as read-only elsewhere in this Brief -- the overlay is additive and never erases or replaces the underlying outcome (Captured/Forfeited/Refunded), per the entity's own Contention line | This is the one deliberate exception to this feature's otherwise read-only relationship with Deposit Transaction; recorded explicitly so a downstream builder does not read the Connected Entities line and miss it |
| Delete/Archive | N/A | Owned by FEAT-07/FEAT-30; financial records are retained per SC-22 and de-identified only after account closure or client deletion, never deleted outright | -- |
| State Transition | FEAT-16.SPEC-003 | Applies the Disputed overlay only; every other transition (Authorized/Captured/Applied/Refunded/Forfeited) belongs to FEAT-07, FEAT-09, FEAT-11, FEAT-30 | A deposit's terminal outcome (per the entity's Contention rule) is set once by another feature; the Disputed overlay sits alongside it, not in place of it |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-16.SPEC-001, FEAT-16.SPEC-002, FEAT-16.SPEC-003, FEAT-16.SPEC-004 | The timeline, the recording automation, the dispute flag, and the summary download all key off one Booking; this feature never creates, updates (beyond the Deposit Transaction exception above), or deletes a Booking |
| Message | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | Delivery events (sent, retried, failed, fallen back per XBR-17) are recorded as Activity Events and shown on the timeline; message content itself remains owned by FEAT-08 |
| Cancellation Policy | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | The policy version and wording shown/acknowledged at booking (FEAT-09.SPEC-002) is recorded and displayed; this feature never edits policy terms |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Deposit payment succeeds (FEAT-05) | Write a "deposit paid" event | Standalone Automation | FEAT-16.SPEC-002 |
| Deposit attempted / succeeded / failed (FEAT-07.SPEC-002) | Write the corresponding deposit event | Standalone Automation | FEAT-16.SPEC-002 |
| A message is sent, retried, fails, or falls back to email (FEAT-08.SPEC-001/002/004/005/006/009) | Write the message-delivery event, including a plainly-visible gap when a text fails (XBR-17) | Standalone Automation | FEAT-16.SPEC-002 |
| Policy version shown and acknowledged at booking (FEAT-09.SPEC-002) | Write the policy-acknowledgment event with the version and acknowledgment time | Standalone Automation | FEAT-16.SPEC-002 |
| Client commits a cancel/reschedule (FEAT-10.SPEC-004) | Write the cancellation/reschedule event | Standalone Automation | FEAT-16.SPEC-002 |
| No-show marked or forfeiture undone (FEAT-11.SPEC-002/003) | Write the no-show mark / undo event | Standalone Automation | FEAT-16.SPEC-002 |
| Client deletion is processed (FEAT-13.SPEC-004) | Convert this booking's entries to de-identified form (XBR-19) rather than deleting them | Standalone Automation | FEAT-16.SPEC-002 |
| Payout account status changes (FEAT-28.SPEC-003) | Write the payout-status event to the Pro Account's activity | Standalone Automation | FEAT-16.SPEC-002 |
| Pro commits a single or bulk cancel/reschedule (FEAT-30.SPEC-007/008) | Write the corresponding event(s), one per affected booking | Standalone Automation | FEAT-16.SPEC-002 |
| A card-issuer dispute notice arrives from the payment-processing capability | Flag the booking, set the Deposit Transaction's Disputed overlay, and write the dispute event | Standalone Integration | FEAT-16.SPEC-003 |
| A card-issuer dispute notice arrives | Notify the Pro | Cross-feature -- owned by FEAT-08.SPEC-006 (Pro Attention Alert), per this feature's own Communications field ("A Pro notification (via FEAT-08)") | FEAT-08.SPEC-006 responsibility |
| A card-issuer dispute notice arrives | Feed the attention flag shown on the Pro's dashboard | Cross-feature -- owned by FEAT-12.SPEC-002/SPEC-005 | FEAT-12 responsibility |
| Pro taps the dispute flag and requests the evidence summary | Assemble policy shown/acknowledged, booking time, messages sent, and no-show mark into a plain, downloadable file | Standalone Automation | FEAT-16.SPEC-004 |
| Pro opens a booking's timeline and taps through to a refund | Navigate to the goodwill refund flow | Cross-feature -- navigation only; refund logic owned by FEAT-30.SPEC-003 | FEAT-30 responsibility |
| Any role attempts to edit or delete an Activity Event | Refused outright -- no edit or delete path exists in any spec | Standalone Logic/Rule | FEAT-16.SPEC-005 |
| Support opens a timeline | Show the read-only view with the Pro's private client notes excluded (XBR-24) | Standalone Logic/Rule, rendered by the Screen | FEAT-16.SPEC-005, FEAT-16.SPEC-001 |

## Shared Context

**Shared Entities:**
- Activity Event -- created by SPEC-002 (qualifying booking events) and, for support-view logging, written via FEAT-19.SPEC-002; read and rendered by SPEC-001; de-identified in place by SPEC-002 on client deletion. Fields: event_type, time, actor (Client, Pro, the product automatically, or a support view), details (e.g., policy version and wording shown, message sent, deposit outcome).
- Deposit Transaction (narrow slice) -- read by SPEC-001 and SPEC-003; the Disputed overlay is written only by SPEC-003, and only as an addition, never a replacement, to the outcome another feature already set.
- Booking, Message, Cancellation Policy -- read-only across SPEC-001 through SPEC-004; none of their fields are ever written by this feature.

**Shared UI Patterns:**
- Single ordered-timeline surface -- SPEC-001 is the one screen for both the Pro's own use and Support's read-only use; it toggles visible content (private notes hidden from Support) rather than presenting two separate screens. Spec Writers should describe both audiences of this one screen consistently, including how a delivery gap (XBR-17) renders inline rather than being hidden.

**Shared Validation:**
- SPEC-005 defines the immutability, access, retention, and private-notes-exclusion rules once. SPEC-001 (rendering), SPEC-002 (write enforcement), SPEC-003 (Disputed-overlay scoping), and SPEC-004 (private-notes exclusion on the downloadable summary) all reference it rather than restating the rules.

## Internal Dependency Map

```
SPEC-002 (Activity Event Recording) -> [inbound events from FEAT-05, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-13, FEAT-28, FEAT-30] -> Activity Event written
SPEC-001 (Booking Activity Timeline) -> [Pro or Support opens a booking's history] -> reads Activity Event (via SPEC-002's writes), Booking, Message, Cancellation Policy, Deposit Transaction
SPEC-003 (Card-Issuer Dispute Integration) -> [dispute notice received] -> writes a dispute Activity Event (via SPEC-002's write path) and sets the Deposit Transaction's Disputed overlay
SPEC-003 (Card-Issuer Dispute Integration) -> [booking flagged] -> SPEC-001 (Pro opens the flagged booking's timeline)
SPEC-001 (Booking Activity Timeline) -> [Pro taps the dispute flag / requests evidence] -> SPEC-004 (Dispute Summary Download)
SPEC-001 (Booking Activity Timeline) -> [Pro decides to refund] -> FEAT-30 (Goodwill Deposit Refund)
SPEC-001 (Booking Activity Timeline) -> [governed by] -> SPEC-005 (Immutability & Visibility Rules)
SPEC-002 (Activity Event Recording) -> [governed by] -> SPEC-005 (Immutability & Visibility Rules)
SPEC-003 (Card-Issuer Dispute Integration) -> [governed by] -> SPEC-005 (Immutability & Visibility Rules)
SPEC-004 (Dispute Summary Download) -> [governed by] -> SPEC-005 (Immutability & Visibility Rules)
```

**Default Entry:** SPEC-001 (Booking Activity Timeline) -- reached only from a specific booking (FEAT-12's past-bookings browse or attention list); this feature has no standalone entry point of its own, consistent with its Interactions field ("Reads from ... FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11").

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-16.SPEC-002 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Deposit payment success writes an activity event | Deposit payment succeeds |
| FEAT-16.SPEC-002 | Inbound | FEAT-07 (Deposit Payment at Booking) | Deposit attempted/succeeded/failed writes an activity event | Deposit outcome determined |
| FEAT-16.SPEC-002 | Inbound | FEAT-08 (Automated Booking Messaging) | Every message sent, retried, failed, and fallen back writes an activity event, per XBR-17 | Message delivery attempt or outcome |
| FEAT-16.SPEC-002 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Policy version shown and acknowledged at booking is recorded, per XBR-08 | Booking created and policy acknowledged |
| FEAT-16.SPEC-002 | Inbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | Client cancel/reschedule commit writes an activity event | Client commits a cancel or reschedule |
| FEAT-16.SPEC-002 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | No-show mark and its undo write activity events, later used as dispute evidence | Booking marked no-show / mark undone |
| FEAT-16.SPEC-002 | Inbound | FEAT-13 (Client Record Management) | Client deletion writes/converts activity events to de-identified form, per XBR-19 | Client deletion is processed |
| FEAT-16.SPEC-002 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Payout account status change writes an activity event | Payout account status changes |
| FEAT-16.SPEC-002 | Inbound | FEAT-30 (Pro Booking Management) | Pro single and bulk cancel/reschedule commits write activity events | Pro commits a cancel or reschedule |
| FEAT-16.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro opens a past booking from the Past Bookings Browse, landing on its timeline | Pro browses past bookings and selects one |
| FEAT-16.SPEC-003 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Card-issuer dispute feeds the Attention List / Attention Flag Aggregation shown on the dashboard | Dispute notice received |
| FEAT-16.SPEC-003 | Outbound | FEAT-08 (Automated Booking Messaging) | Dispute notice hands off to Pro Attention Alert (FEAT-08.SPEC-006) for the Pro notification named in this feature's Communications field | Dispute notice received |
| FEAT-16.SPEC-001 | Outbound | FEAT-30 (Pro Booking Management) | Pro decides to refund as goodwill from the timeline, navigating to the Goodwill Deposit Refund flow | Pro taps refund from the timeline |
| FEAT-16.SPEC-002 | Inbound | FEAT-19 (Platform Support Read-Only Access) | Each support view of a timeline is logged as an Activity Event written via FEAT-19.SPEC-002 (Support View Logging), per the dependency map's Lifecycle line and XBR-24; recorded in FEAT-16.SPEC-002 as a second writer (SG-11 decision) | Support opens a Pro's account |

## Non-Functional Notes

**Data volumes / growth:** Each booking accumulates only a handful of Activity Event entries (created, paid, confirmed, messaged, cancelled/rescheduled, no-show, refunded/forfeited, disputed), but the record spans a Pro's full multi-year history at 20-40 bookings a week (ASMP-22, SC-22); the timeline must stay equally responsive as that history accumulates.

**Responsiveness:** The States field is explicit that this is a small per-booking dataset that "loads instantly" -- no loading state beyond an instantaneous local render is expected; the download in FEAT-16.SPEC-004 assembles from already-loaded timeline data, so it is likewise near-instantaneous rather than a long-running export.

**Data sensitivity / privacy:** Activity Events carry personal and financial event details and form dispute evidence, so immutability is itself a trust requirement (dependency map, Data Sensitivity line); visible only to the Pro (View) and Platform Operator Support (View-only, and never the Pro's private client notes, per XBR-24); Clients never see this internal timeline directly (Access field) -- they see outcomes through their own booking view.

**Compliance flags:** SC-11 applies directly to FEAT-16.SPEC-003 -- the dispute integration reads only dispute metadata and outcome flags from the payment-processing capability, never card data, which the payment processor alone owns. SC-22 governs retention: full history is kept for the life of the account for dispute evidence and insights, and only de-identified financial and timeline records survive a client deletion or account closure (XBR-19).

## Non-Goals

- **Editing, correcting, or deleting an Activity Event, by any role including the Pro** -- Excluded per the Access field ("nobody, including the Pro, can edit an entry") and the Validation & Limits field ("append-only and immutable once written"); this is the property that makes the record usable as dispute evidence, not an oversight. Enforced by FEAT-16.SPEC-005 with no override path anywhere in this feature.
- **A standalone Notification spec for the card-issuer dispute alert** -- Excluded per this feature's own Communications field, which states the Pro notification is delivered "via FEAT-08"; the disposition is an explicit cross-feature hand-off (FEAT-08.SPEC-006, Pro Attention Alert) rather than a duplicate notification path, recorded in the Side-Effect Inventory and Cross-Feature Touchpoints above.
- **An automated hand-over of the evidence summary to the payment processor** -- Excluded per this feature's own Key Capabilities line, which describes the Pro downloading a file "to use as evidence with the payment processor," and per SC-17 (Chairtime never rules on or manages the dispute process itself); the Pro submits the downloaded summary herself through the processor's own channel.
- **Chairtime deciding or adjudicating a dispute** -- Excluded per SC-17: this feature supplies the trustworthy record and the flag; who is right is resolved between the Pro and the card issuer, never by the product.
- **Support editing, refunding, or acting on the Pro's behalf from the timeline** -- Excluded per SC-05: Platform Operator (Support) has View-only access to the same timeline; no action control exists in this feature's screen for that role.
- **Displaying the Pro's private client notes to Support** -- Excluded per XBR-24: Support's view of the timeline never includes the Pro's private notes about the client, enforced by FEAT-16.SPEC-005.
- **A user-facing search, filter, or export-to-spreadsheet view across many bookings' activity records at once** -- Not named by any Key Capability, Primary Flow, or journey step; the feature is scoped to one booking's timeline at a time (Data Notes: "a chronological event list per booking"). A cross-booking activity view, if ever wanted, belongs to Booking & Revenue Insights (FEAT-25), not this feature.
- **Permanent hard deletion of Activity Events after a client deletion or account closure** -- Excluded per SC-22: only de-identification occurs; the underlying financial and timeline facts are retained indefinitely for legal and dispute purposes, never purged outright.

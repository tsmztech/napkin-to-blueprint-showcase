# FEAT-16 — Booking & Payment Activity Record

This chapter covers Booking & Payment Activity Record (FEAT-16), a Important-tier feature. It carries 5 specifications carrying 87 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-16.SPEC-001 | Booking Activity Timeline | screen | 19 |
| FEAT-16.SPEC-002 | Activity Event Recording | automation | 23 |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | integration | 13 |
| FEAT-16.SPEC-004 | Dispute Summary Download | automation | 14 |
| FEAT-16.SPEC-005 | Activity Record Immutability & Visibility Rules | logic-rule | 18 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Booking Activity Timeline

## Overview

**Name:** Booking Activity Timeline
**ID:** FEAT-16.SPEC-001
**Type:** Screen
**Purpose:** The Pro (or Platform Operator Support, read-only) opens a single booking's full, ordered event history to reference when preparing to respond to a client dispute, and can act from it by starting a goodwill refund or requesting a dispute evidence summary.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Rendering the ordered event list for one Booking's Activity Events, including the policy version shown/acknowledged, appointment time, message delivery events (including gaps), cancellation/reschedule, no-show mark, deposit outcome, and any dispute event
- Distinguishing the Pro's own view from Platform Operator (Support)'s read-only, private-notes-excluded view of the same screen
- An entry point from this screen into the goodwill refund flow and into the dispute evidence download

**Non-Goals:**
- Editing, correcting, or deleting any Activity Event -- excluded per FEAT-16.SPEC-005 (Validation & Limits: "append-only and immutable once written"); this screen has no write path to an entry, ever, for any role
- Executing the goodwill refund itself -- excluded per the Internal Dependency Map; this screen only navigates to FEAT-30.SPEC-003 (Goodwill Deposit Refund), which owns the refund logic
- Assembling or generating the downloadable dispute evidence file -- excluded per the Internal Dependency Map; this screen only navigates to FEAT-16.SPEC-004 (Dispute Summary Download), which owns the assembly
- A cross-booking search, filter, or list of activity records -- excluded per feature-overview.md's Non-Goals: this feature is scoped to one booking's timeline at a time; a cross-booking view belongs to Booking & Revenue Insights (FEAT-25)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-003 (Past Bookings Browse) | Pro selects a past booking from the browse list | The selected Booking's identifier |
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | Pro taps a card-issuer dispute flag on the dashboard | The disputed Booking's identifier, with the screen opening scrolled to the dispute event |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a disputed booking from the support view of a Pro's account, per a help request | The Booking's identifier; the screen renders in the Support read-only variant |
| FEAT-19.SPEC-003 (Support Access Log) | Support taps an access-log entry that references a booking | The referenced Booking identifier; Support read-only variant |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full timeline for any of her own bookings, including her own private client notes surfaced elsewhere on the booking (not duplicated on this screen) | Navigate to the goodwill refund (FEAT-30.SPEC-003) and to the dispute summary download (FEAT-16.SPEC-004) when the booking is flagged disputed; no edit action exists for any role | -- |
| Platform Operator (Support) | Full timeline for the one Pro account they are actively viewing under a help request, with the Pro's private client notes excluded (XBR-24) | View-only -- no refund entry point, no download entry point, no action of any kind (SC-05) | If Support attempts to reach a refund or download control (neither is rendered for this role, so no control exists to attempt): the screen carries no such element for Support in the first place |
| The Client (Riley) | None -- this internal operational record is never shown to the Client | None | Riley's own booking view shows her outcomes (confirmation, deposit status) but has no link, deep link, or route into this screen; a direct attempt to reach it is treated as an unauthenticated request |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen; after signing in, the user lands on FEAT-12.SPEC-001 (Today's & Upcoming Schedule), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress work exists on this read-only screen to preserve; after re-authentication the user returns to their prior context (dashboard or support view) |

## Layout and Content

**Header:** Booking summary bar showing the client's name, service, and appointment date/time (Pro's timezone), with a back arrow (returns to the entry source: FEAT-12.SPEC-003, the attention list, or the support view). When viewed by Support, the header additionally shows a small "Support view -- read-only" label.

**Body:** A single-column, reverse-chronological (most recent first) ordered list of Activity Event entries. Each entry shows:
- A timestamp (Pro's timezone, per XBR-25)
- An actor indicator (Client, Pro, "Chairtime" for automatic entries, or "Support view" for a logged support access)
- A plain-language description of the event (e.g., "Deposit paid," "Policy shown and acknowledged: {version}," "Text reminder sent," "Marked no-show," "Deposit kept per cancellation policy")
- Where the event is a message-delivery gap (a failed text that fell back to email, per XBR-17), the entry renders plainly inline -- e.g., "Text reminder failed to send; sent by email instead" -- never hidden or smoothed over
- Where the event is the card-issuer dispute (FEAT-16.SPEC-003), the entry is visually distinguished (e.g., a flagged marker) and sits inline in chronological order like every other entry

At the top of the body, above the first (most recent) entry, a persistent banner appears only when the booking carries an active dispute: "This booking has a card-issuer dispute" with a "Download evidence summary" action (Pro only; navigates to FEAT-16.SPEC-004).

At the bottom of the body, a "Refund as goodwill" action appears (Pro only, and only while the booking's deposit has not already reached a terminal refunded/forfeited-and-undone-unavailable state per FEAT-30.SPEC-003's own eligibility rules); navigates to FEAT-30.SPEC-003.

**Footer:** None.

Support's view omits: the dispute banner's download action, the goodwill refund action, and any entry whose details field would surface the Pro's private client note (per XBR-24 and FEAT-16.SPEC-005) -- such entries render with their non-note details intact and the note portion simply absent, never a placeholder.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described, full width; the header summary bar wraps to two lines if the client name and service do not fit on one.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.
- **Dispute banner and goodwill refund action:** Remain full-width and pinned at their respective ends of the list at every size class.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source (FEAT-12.SPEC-003, attention list, or support view) | Screen closes | Animated transition back |
| Activity Event entry | Tap | Display-only -- entries are not individually interactive beyond what is already shown inline (per the Entity-Lifecycle Coverage Matrix: "An Activity Event is never opened individually") | None | None |
| "Download evidence summary" (Pro only, disputed bookings only) | Tap | Navigate to FEAT-16.SPEC-004 (Dispute Summary Download), carrying the Booking's identifier | Screen closes | Animated transition to the download flow |
| "Refund as goodwill" (Pro only) | Tap | Navigate to FEAT-30.SPEC-003 (Goodwill Deposit Refund), carrying the Booking's identifier | Screen closes | Animated transition to the refund flow |

### Accessibility Notes

- **Focus order:** Back arrow -> dispute banner (when present) -> timeline entries in displayed order (most recent first) -> "Refund as goodwill" action (when present).
- **Dynamic content announcements:** Because this is a read-only, load-once screen, no validation or save-state announcements apply; if new Activity Events are written by another process while the screen is open, no live update occurs (see States, Offline/Degraded) so nothing is announced mid-view.
- **Keyboard alternatives:** Every action on this screen (back, download, refund) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Full ordered timeline rendered as described | Screen opens for a booking with at least one Activity Event | N/A -- this is a snapshot read-only screen; the state persists until the user navigates away |
| Loading | N/A -- feature-overview.md's Non-Functional Notes state this is a small per-booking dataset that "loads instantly"; no loading state beyond an instantaneous local render is expected | -- | -- |
| Error | N/A -- this is a read-only, append-only log; there is no user-facing write path on this screen to fail, and a read failure is treated as the Offline/Degraded case below rather than a distinct error state | -- | -- |
| Gap present | Same as Loaded, with one or more entries rendering a plainly-visible gap (e.g., a failed-then-fallback message event) inline, per XBR-17 | The booking's timeline includes at least one such event | N/A -- persists for the life of the timeline; a gap is a permanent, immutable fact once recorded |
| Offline/Degraded | The most recently loaded timeline for this booking remains viewable read-only; the dispute-download and goodwill-refund actions (which require a live connection to their own flows) are disabled with the note "This action needs a connection." | Connectivity is lost after the timeline has loaded at least once | Connectivity restored -- the disabled actions re-enable; the timeline itself does not need to reload since it is a small, immutable dataset already held |

## Validation Rules

N/A -- this is a read-only screen with no user input fields; validation for the actions it navigates to (goodwill refund, dispute download) is owned entirely by FEAT-30.SPEC-003 and FEAT-16.SPEC-004 respectively.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12.SPEC-003 (Past Bookings Browse), the attention list, or the support view (whichever was the entry source) | FEAT-12 (or FEAT-19) |
| "Download evidence summary" tap | FEAT-16.SPEC-004 (Dispute Summary Download) | -- |
| "Refund as goodwill" tap | FEAT-30.SPEC-003 (Goodwill Deposit Refund) | FEAT-30 |

## Data Model

**Creates:** None -- this screen never writes an Activity Event; all entries are written by FEAT-16.SPEC-002 (and, for support-view entries handed off by FEAT-19.SPEC-002, through FEAT-16.SPEC-002 as the sole writer).
**Reads:** Activity Event (event_type, time, actor, details) for the one Booking, ordered by time; Booking (service, start_time, client reference, state, policy_version); Message (type, channel, send time, delivery_status) for delivery-event rendering; Cancellation Policy (version, plain_language_wording) for the policy-acknowledgment entry; Deposit Transaction (status, outcome_reason, Disputed overlay) for outcome and dispute-flag rendering.
**Updates:** None.
**Deletes:** None.

## Business Rules

- No role, including the Pro, can edit or delete an entry rendered on this screen -- governed entirely by FEAT-16.SPEC-005 (Validation & Limits: append-only and immutable).
- Support's view excludes the Pro's private client notes from any entry's details, per XBR-24 and FEAT-16.SPEC-005 (Authorization Rules).
- A message-delivery gap renders inline exactly as it occurred -- it is never smoothed over, summarized away, or hidden, per XBR-17.
- The policy version and wording shown to the client at booking (FEAT-09.SPEC-002) is rendered as its own distinct entry, never merged into the "created" entry, so the exact version and acknowledgment time are independently visible.
- The "Refund as goodwill" action is available only while FEAT-30.SPEC-003's own eligibility rules permit a goodwill refund on this booking; this screen defers entirely to that spec's eligibility determination rather than re-deriving it.
- The dispute banner and its download action appear only while the Deposit Transaction carries the Disputed overlay set by FEAT-16.SPEC-003.

## Edge Cases

- **A booking has no events beyond "created"** -- The timeline shows a single entry; no empty state applies (a timeline only exists for bookings that have happened, per feature-overview.md's States field).
- **Support opens a timeline for a booking with a private Pro note attached to an event** -- The entry renders with its non-note details intact; the note content is simply absent from the entry, never replaced with a placeholder like "[hidden]" (per XBR-24).
- **The Pro taps "Refund as goodwill" on a booking whose deposit was already refunded** -- The action is not shown in this case per FEAT-30.SPEC-003's eligibility rules; if the underlying state changes between page load and tap (see concurrent-access entry below), the Pro is routed into FEAT-30.SPEC-003, which independently re-checks eligibility and refuses with its own current-state message.
- **A new Activity Event is written (by another feature acting on this booking) while the Pro has this screen open** -- No live update occurs; the screen shows a snapshot as of load time (per the Offline/Degraded state's rationale that this is a small, rarely-changing dataset). Reopening the screen shows the new entry.
- **Concurrent access: the Deposit Transaction's terminal outcome changes between this screen's load and the Pro tapping an action into FEAT-30.SPEC-003** -- No conflict resolution occurs on this screen itself, because it never writes to the Deposit Transaction; FEAT-30.SPEC-003 (per the dependency map's Contention note: reject-with-refresh, one terminal outcome per deposit) is the one that detects and resolves the conflict when the Pro's action reaches it.
- **A dispute notice arrives while the Pro is already viewing this booking's timeline** -- No live update occurs (per Offline/Degraded rationale); the dispute banner and its download action appear the next time the screen is opened.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | This screen renders the entries that automation writes |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | References (inbound) | The dispute banner and flagged entry reflect this spec's write |
| FEAT-16.SPEC-004 (Dispute Summary Download) | Navigation (outbound) | "Download evidence summary" navigates here |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the absence of any edit control and the Support private-notes exclusion |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Navigation (inbound) | Pro arrives from a selected past booking |
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | Navigation (inbound) | Pro arrives by tapping a dispute flag |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry), FEAT-19.SPEC-002 (Support View Logging), FEAT-19.SPEC-003 (Support Access Log) -- within FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support arrives from the support view of a Pro's account |
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Navigation (outbound) | "Refund as goodwill" navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_record_viewed | viewer role (Pro / Support), booking has an active dispute (yes/no) | The timeline finishes loading | N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of how often the record is consulted, most notably around disputes |
| dispute_download_entry_tapped | -- | The Pro taps "Download evidence summary" | N/A -- no connected success-metrics.md metric; retained to observe how often the flagged record leads into evidence assembly |
| goodwill_refund_entry_tapped | -- | The Pro taps "Refund as goodwill" from this screen | N/A -- no connected success-metrics.md metric; the resulting refund outcome is measured downstream by FEAT-30.SPEC-011's own signals |

## Acceptance Criteria

**FEAT-16.SPEC-001-AC-01:** Given Talia opens a past booking from FEAT-12.SPEC-003, when the timeline finishes loading, then she sees every Activity Event for that booking in reverse-chronological order, including the policy version shown and acknowledged and its acknowledgment time.

**FEAT-16.SPEC-001-AC-02:** Given Talia is viewing a booking's timeline where a reminder text failed and fell back to email, when she looks at that entry, then it reads plainly, for example "Text reminder failed to send; sent by email instead," rather than being hidden.

**FEAT-16.SPEC-001-AC-03:** Given Talia is viewing a booking whose Deposit Transaction carries the Disputed overlay, when the screen loads, then a banner "This booking has a card-issuer dispute" appears at the top with a "Download evidence summary" action.

**FEAT-16.SPEC-001-AC-04:** Given Talia taps "Download evidence summary" on a disputed booking's timeline, when the tap registers, then she is navigated to FEAT-16.SPEC-004 with that booking's identifier carried along.

**FEAT-16.SPEC-001-AC-05:** Given Talia is viewing a booking eligible for a goodwill refund, when she taps "Refund as goodwill," then she is navigated to FEAT-30.SPEC-003 with that booking's identifier carried along.

**FEAT-16.SPEC-001-AC-06:** Given Talia is viewing any booking's timeline, when she looks for an edit or delete control on any entry, then none exists anywhere on the screen.

**FEAT-16.SPEC-001-AC-07:** Given Platform Operator Support opens a booking's timeline after Talia's help request, when the screen renders, then it shows the "Support view -- read-only" label, omits the goodwill-refund action and the dispute-download action entirely, and excludes Talia's private client notes from any entry.

**FEAT-16.SPEC-001-AC-08:** Given Riley (the Client) attempts to reach this screen, when the attempt is made, then she is treated as an unauthorized/unauthenticated user and is not shown any part of this timeline.

**FEAT-16.SPEC-001-AC-09:** Given an unauthenticated visitor reaches this screen's route directly, when the screen would otherwise load, then they are redirected to the Pro sign-in screen and land on FEAT-12.SPEC-001 after signing in, not on this timeline.

**FEAT-16.SPEC-001-AC-10:** Given Talia's session expires while this screen is open, when she next interacts with it, then the dialog "Your session has expired. Sign in to continue." appears, and after re-authenticating she returns to her prior context.

**FEAT-16.SPEC-001-AC-11:** Given Talia loses connectivity after this booking's timeline has already loaded, when she looks at the screen, then the previously loaded entries remain visible read-only and the dispute-download and goodwill-refund actions (if present) show "This action needs a connection." and are disabled.

**FEAT-16.SPEC-001-AC-12:** Given Talia is viewing a booking with only a "created" event so far, when the screen loads, then exactly that one entry is shown with no empty-state message.

**FEAT-16.SPEC-001-AC-13:** Given Talia has this screen open and another feature writes a new Activity Event to this same booking in the background, when Talia continues viewing without navigating away, then the new entry does not appear until she reopens the screen.

**FEAT-16.SPEC-001-AC-14:** Given Talia's deposit for this booking was already refunded before she opens the timeline, when the screen loads, then the "Refund as goodwill" action is not shown, per FEAT-30.SPEC-003's eligibility rules.

**FEAT-16.SPEC-001-AC-15:** Given Talia taps "Refund as goodwill" and the deposit's state changed to a terminal outcome between load and tap, when FEAT-30.SPEC-003 re-checks eligibility, then that spec refuses with its own current-state message rather than this screen silently proceeding.

**FEAT-16.SPEC-001-AC-16:** Given Support is viewing a timeline entry whose details would normally include Talia's private client note, when the entry renders, then the note content is simply absent -- no placeholder text appears in its place.

**FEAT-16.SPEC-001-AC-17:** Given Talia opens this screen and taps the back arrow, when the tap registers, then she returns to whichever entry point she arrived from (Past Bookings Browse, the attention list, or nowhere else).

**FEAT-16.SPEC-001-AC-18:** Given Talia is viewing a booking's timeline, when she taps any individual entry, then nothing happens -- entries are display-only and are never opened individually.

**FEAT-16.SPEC-001-AC-19:** Given Talia is viewing a booking's timeline that includes a cancellation, a no-show mark, and a deposit outcome, when she reads the list, then each of those three events appears as its own distinct, correctly ordered entry rather than being merged into one summary line.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (loaded, gap present, offline/degraded, N/A loading/error justified) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Automation Spec: Activity Event Recording

## Overview

**Name:** Activity Event Recording
**ID:** FEAT-16.SPEC-002
**Type:** Automation
**Purpose:** Writes one immutable, append-only Activity Event for every qualifying action across Booking, Deposit Payment, Messaging, Cancellation/Reschedule, No-Show, Client Deletion, Payout, Pro-initiated cancel/reschedule, and Support View, so a complete timeline exists for FEAT-16.SPEC-001 to render.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Writing a new Activity Event for every qualifying trigger listed in the Trigger Definition below, including the support-view event handed off by FEAT-19.SPEC-002 (written against the Pro Account, actor "a support view")
- Converting this booking's (or Pro Account's) existing Activity Events to de-identified form when a client deletion is processed, per XBR-19
- Guaranteeing that every write is append-only and immutable at the moment of creation (the entry is never revisited by this automation once written)

**Non-Goals:**
- Rendering the timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline); this automation only produces the data that screen displays
- Writing the card-issuer dispute event -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration), which writes through this same append-only mechanism but is triggered by an external event this spec does not itself receive
- Editing or correcting a previously written entry, by any role or process, ever -- excluded per feature-overview.md's Validation & Limits ("append-only and immutable once written") and enforced by FEAT-16.SPEC-005; a mistaken entry is never overwritten, only ever superseded by a later, independent entry describing what actually happened next
- Assembling the support-view event's own content (reason/ticket reference, which timeline was viewed) -- owned by FEAT-19.SPEC-002 (Support View Logging), which hands the assembled event to this automation; this spec only writes it

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking created | FEAT-05.SPEC-005 (Booking Confirmation) | A new Booking reaches a confirmed state after deposit payment | Booking reference, service, appointment time, client reference, source (client link / Pro booked-in / recurring occurrence) |
| Policy shown and acknowledged | FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | The client acknowledges the cancellation policy during booking | Booking reference, Cancellation Policy version, plain-language wording shown, acknowledgment timestamp |
| Deposit attempted / succeeded / failed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Every deposit attempt reaches an outcome | Booking reference, Deposit Transaction reference, outcome (attempted / succeeded / failed), amount, timestamp |
| Message sent, retried, failed, or falls back to email | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-08.SPEC-009 (Automated Booking Messaging specs) | Every message delivery attempt or outcome, including a failed-then-fallback sequence per XBR-17 | Booking reference (or Pro Account reference for Pro notifications), message type, channel, delivery_status, timestamp |
| Client commits a cancel/reschedule | FEAT-10.SPEC-004 (Booking Update Commit) | The client's cancel or reschedule action commits | Booking reference, action (cancelled / rescheduled), new time (if rescheduled), deposit outcome triggered, timestamp |
| No-show marked | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | The Pro marks a booking as a no-show | Booking reference, Deposit Transaction outcome (forfeited), timestamp |
| No-show mark undone | FEAT-11.SPEC-003 (No-Show Mark Undo) | The Pro undoes a no-show mark within the grace window | Booking reference, reversed Deposit Transaction outcome, timestamp |
| Client deletion processed | FEAT-13.SPEC-004 (Client Deletion Execution) | A client deletion request completes | Client reference, list of affected Bookings, de-identification instruction |
| Payout account status changes | FEAT-28.SPEC-003 (Payout Account Status Processing) | The Pro's Payout Account status changes | Pro Account reference, new status, timestamp |
| Pro commits a single cancel/reschedule | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | The Pro's single cancel or reschedule action commits | Booking reference, action, new time (if rescheduled), deposit outcome, timestamp |
| Pro commits a bulk cancel/reschedule | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | The Pro's bulk cancel or reschedule action commits | List of affected Booking references, action, deposit outcome per booking, timestamp |
| Support view logged (session opened, or disputed booking timeline opened within a session) | FEAT-19.SPEC-002 (Support View Logging) | Support opens a help-request-gated session or opens a disputed booking's timeline within it | Pro Account reference, event_type (support_view_opened / support_view_booking_timeline), reason/ticket reference, Booking reference (timeline view only), timestamp |
| Card-issuer dispute recorded (external-event trigger) | FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | FEAT-16.SPEC-003 receives an inbound card-issuer dispute notice | Booking reference, Deposit Transaction reference, dispute timestamp |

## Processing Logic

1. Receive the triggering event's data from the source spec, identified by its trigger type from the table above.
2. Determine the correct event_type for the entry (e.g., "created," "policy_acknowledged," "deposit_attempted" / "deposit_succeeded" / "deposit_failed," "message_sent" / "message_failed" / "message_fallback," "cancelled" / "rescheduled," "no_show_marked" / "no_show_undone," "payout_status_changed," "support_view_opened" / "support_view_booking_timeline," "disputed").
3. Determine the actor for the entry: Client, Pro, "the product automatically," or a support view (when the trigger is FEAT-19.SPEC-002).
4. Assemble the details field from the trigger's available data (e.g., policy version and wording shown, message channel and outcome, deposit outcome and amount, new appointment time).
5. Write one new Activity Event, associated with the Booking (or, for a Pro-Account-level event such as a payout status change or a support view, the Pro Account -- a support-view entry never belongs to the Booking, even when the trigger was a timeline view) referenced by the trigger. The entry's time, actor, event_type, and details are fixed at the moment of this write and never revisited.
6. If the trigger is a bulk action affecting multiple bookings (FEAT-30.SPEC-008), repeat steps 2--5 once per affected booking so each booking's timeline carries its own entry.
7. If the trigger is a client deletion (FEAT-13.SPEC-004), do not write a new event for the deletion itself as a fresh fact on the booking; instead, convert every existing Activity Event belonging to the affected client's bookings to de-identified form: strip contact and note content from each entry's details field while retaining the financial and timeline facts (amounts, outcomes, timestamps, event types) unchanged, per XBR-19.
8. Confirm the write (or, for client deletion, the conversion) completed before returning control to the triggering spec; no triggering spec's own success path depends on waiting for this automation, since Activity Event recording is a side effect, not a precondition of any other spec's completion.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Entry written | The trigger's data is complete and valid | One new, immutable Activity Event created for the referenced Booking or Pro Account | None directly -- the entry becomes visible the next time FEAT-16.SPEC-001 is opened | FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Bulk entries written | A bulk trigger (FEAT-30.SPEC-008) affects multiple bookings | One new Activity Event per affected booking | None directly | FEAT-16.SPEC-001 |
| Entries de-identified | A client deletion completes (FEAT-13.SPEC-004) | All existing Activity Events for the client's bookings have contact and note content stripped from their details field; financial and timeline facts retained | None directly -- Talia sees the de-identified entries the next time she opens an affected timeline | FEAT-16.SPEC-001 |
| No-op (nothing to record) | A trigger fires but its underlying action produced no state genuinely worth recording (this does not occur for any trigger in the table above -- every listed trigger corresponds to a qualifying, recordable action) | None | None | -- |
| Write failure | The automation cannot complete the write against a Booking or Pro Account reference (e.g., the referenced record cannot be found) | No entry is created | Non-blocking to the triggering spec -- the triggering action (e.g., the deposit capture, the message send) completes on its own terms regardless of whether its activity entry succeeded; the gap is retried automatically | The triggering spec proceeds unaffected; the timeline (FEAT-16.SPEC-001) shows a gap until the retry succeeds |

## Data Model

**Reads:** Booking (reference, service, start_time, client reference, state), Deposit Transaction (reference, status, amount, outcome_reason), Message (type, channel, delivery_status), Cancellation Policy (version, plain_language_wording) -- read only to assemble each entry's details field from the triggering spec's own available data, never independently re-queried beyond what the trigger provides.
**Creates:** Activity Event -- event_type, time, actor, details, associated to one Booking or the Pro Account.
**Updates:** Activity Event -- the only update path this spec has is the client-deletion de-identification conversion (details field content stripped of contact/note content); no other field of any entry is ever changed after creation.
**Deletes:** None -- hard deletion never occurs, per XBR-19 and the Entity-Lifecycle Coverage Matrix.

## Business Rules

- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit.
- Every entry is written once, at the moment its qualifying action occurs; no batching or delayed writing that could reorder entries relative to when their actions actually happened.
- A message-delivery gap (failed text, then email fallback) is recorded as its own visible fact, never merged into or replaced by the eventual fallback's success entry, per XBR-17.
- The Disputed overlay's own event (written by FEAT-16.SPEC-003) uses this same append-only mechanism and the same immutability guarantee; this spec's Processing Logic (steps 2--5) applies identically to that trigger.
- Client deletion never removes financial or timeline facts -- only contact and note content is stripped, per XBR-19 and SC-22.
- This automation never blocks or delays the triggering spec's own completion; recording is a side effect that runs alongside, not a gate the triggering action must pass through.

## Edge Cases

- **The triggering spec's own action later needs correction (e.g., a mis-marked no-show is undone)** -- The undo (FEAT-11.SPEC-003) is its own distinct trigger producing its own new entry; the original no-show-marked entry is never edited or removed, so the timeline shows both facts in order.
- **Two triggers fire for the same booking at effectively the same time (e.g., a client reschedule commits at the same moment a reminder message is sent)** -- Each trigger writes its own independent entry; entries are ordered by their own recorded time, and no coordination between the two writes is needed since neither reads or depends on the other's outcome.
- **A trigger fires while a previous recording run for the same booking is still in flight** -- Each write is independent and additive (append-only), so a second write for the same booking never needs to wait for, merge with, or overwrite the first; both entries land in the timeline in their own time order.
- **A client deletion is processed for a client with an in-flight, not-yet-recorded event (e.g., a message send that has not yet reported delivery status)** -- The de-identification conversion applies to entries that already exist at the time of deletion; a still-in-flight event that lands afterward is written and immediately carries no contact or note content for that now-deleted client's record, consistent with the fields already stripped elsewhere.
- **A trigger references a Booking or Pro Account that cannot be found (e.g., a data inconsistency upstream)** -- The write fails per the Write failure outcome above; the triggering spec's own action is unaffected, and the gap is retried automatically without blocking any user-facing flow.
- **A bulk cancel/reschedule (FEAT-30.SPEC-008) affects zero bookings (e.g., the Pro's selection ends up empty)** -- No entries are written; this is not a failure, simply nothing to record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Triggered by (inbound) | Booking creation writes the "created" entry |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | Triggered by (inbound) | Policy acknowledgment writes its own entry |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Deposit outcome writes its own entry |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-08.SPEC-009 | Triggered by (inbound) | Every message delivery attempt or outcome writes an entry |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | Client cancel/reschedule writes an entry |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) | No-show mark writes an entry |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | Triggered by (inbound) | No-show undo writes an entry |
| FEAT-13.SPEC-004 (Client Deletion Execution) | Triggered by (inbound) | Client deletion triggers de-identification conversion |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Payout status change writes an entry |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | Pro single cancel/reschedule writes an entry |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Pro bulk cancel/reschedule writes one entry per affected booking |
| FEAT-19.SPEC-002 (Support View Logging) | Triggered by (inbound) | Support session open and disputed-timeline view hand off an assembled support-view event; this automation is its sole writer |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | Triggered by (inbound, external-event) | The dispute event is written through this same mechanism |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Affects (outbound) | Every entry this automation writes is what that screen renders |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only, immutable guarantee this automation upholds |

## Analytics and Success Signals

- **activity_event_recorded** (event_type, actor) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal is retained for operational observability of recording volume and coverage across the inbound writer specs.
- **activity_event_recording_failed** (trigger source spec, reason) -- N/A -- no connected success-metrics.md metric; retained to observe how often a gap occurs before automatic retry closes it, since a silent gap would undermine the record's trustworthiness as dispute evidence.
- **activity_events_deidentified** (count of entries converted) -- N/A -- no connected success-metrics.md metric; retained to observe that XBR-19's de-identification obligation is actually being fulfilled on client deletion.

## Acceptance Criteria

**FEAT-16.SPEC-002-AC-01:** Given Riley completes a booking and her deposit is captured, when FEAT-05.SPEC-005 confirms the booking, then a "created" Activity Event is written for that booking with the appointment time, service, and client reference.

**FEAT-16.SPEC-002-AC-02:** Given Riley acknowledges the cancellation policy during booking, when FEAT-09.SPEC-002 records the acknowledgment, then a distinct "policy_acknowledged" Activity Event is written carrying the exact policy version and wording shown and the acknowledgment timestamp.

**FEAT-16.SPEC-002-AC-03:** Given a deposit attempt fails and is then retried and succeeds, when each outcome is determined by FEAT-07.SPEC-002, then two separate Activity Events are written -- one for the failed attempt and one for the succeeded attempt -- neither overwriting the other.

**FEAT-16.SPEC-002-AC-04:** Given a reminder text fails and falls back to email per XBR-17, when FEAT-08.SPEC-009 reports the fallback, then an Activity Event is written showing the failure and a second showing the successful email fallback, both visible on the timeline.

**FEAT-16.SPEC-002-AC-05:** Given Riley commits a reschedule through FEAT-10.SPEC-004, when the commit succeeds, then an Activity Event is written recording the reschedule, the new appointment time, and any resulting deposit outcome.

**FEAT-16.SPEC-002-AC-06:** Given Talia marks a booking as a no-show, when FEAT-11.SPEC-002 forfeits the deposit, then an Activity Event is written recording the no-show mark and the forfeiture outcome.

**FEAT-16.SPEC-002-AC-07:** Given Talia undoes a no-show mark within the grace window, when FEAT-11.SPEC-003 reverses the forfeiture, then a new Activity Event is written recording the undo; the original no-show-marked entry remains unchanged and visible.

**FEAT-16.SPEC-002-AC-08:** Given a client deletion completes via FEAT-13.SPEC-004, when the deletion is processed, then every existing Activity Event for that client's bookings has its contact and note content stripped from the details field while amounts, outcomes, and timestamps remain intact.

**FEAT-16.SPEC-002-AC-09:** Given Talia's Payout Account status changes via FEAT-28.SPEC-003, when the change is processed, then an Activity Event is written against her Pro Account recording the new status.

**FEAT-16.SPEC-002-AC-10:** Given Talia commits a single cancel through FEAT-30.SPEC-007, when the commit succeeds, then an Activity Event is written for that booking recording the cancellation and its deposit outcome.

**FEAT-16.SPEC-002-AC-11:** Given Talia commits a bulk cancellation affecting 5 bookings through FEAT-30.SPEC-008, when the commit succeeds, then 5 separate Activity Events are written, one per affected booking.

**FEAT-16.SPEC-002-AC-12:** Given FEAT-16.SPEC-003 receives an inbound card-issuer dispute notice, when it hands off the dispute event, then this automation writes a "disputed" Activity Event for the affected booking through the same append-only mechanism.

**FEAT-16.SPEC-002-AC-13:** Given any Activity Event has already been written, when any role, including Talia, attempts to change it, then no path exists anywhere in the product to do so -- the write in Processing Logic step 5 is the entry's only ever write.

**FEAT-16.SPEC-002-AC-14:** Given a trigger references a Booking that cannot be found, when the write is attempted, then the write fails without affecting the triggering spec's own success path, and the gap is retried automatically.

**FEAT-16.SPEC-002-AC-15:** Given a client cancel/reschedule commit and a message-delivery event both fire for the same booking at effectively the same time, when both automations run, then each writes its own independent entry and both appear correctly ordered by their own recorded time.

**FEAT-16.SPEC-002-AC-16:** Given a recording run for one booking is still in flight, when a second, unrelated trigger fires for the same booking, then the second write proceeds independently and does not wait for or merge with the first.

**FEAT-16.SPEC-002-AC-17:** Given Talia's bulk cancellation selection ends up affecting zero bookings, when FEAT-30.SPEC-008 completes with no bookings changed, then no Activity Event is written and this is not treated as a failure.

**FEAT-16.SPEC-002-AC-18:** Given a still-in-flight message send for a client completes delivery reporting after that client's deletion has already been processed, when the delivery event is recorded, then the new entry carries no contact or note content for that client, consistent with the client's other de-identified entries.

**FEAT-16.SPEC-002-AC-19:** Given a reschedule and an automatic policy-acknowledgment write occur for two different bookings at the same moment, when both automations run, then neither booking's timeline is affected by the other's write.

**FEAT-16.SPEC-002-AC-20:** Given Talia views a booking's timeline after a payout status change was recorded against her Pro Account, when she looks at her account-level activity (surfaced via the Pro Account's own activity context), then the payout-status entry appears alongside her booking-level entries' shared account context.

**FEAT-16.SPEC-002-AC-21:** Given a deposit outcome and its corresponding Activity Event write are both in progress, when the deposit outcome itself completes, then the deposit's own success or failure path is never blocked or delayed by whether this automation's write has finished.

**FEAT-16.SPEC-002-AC-22:** Given a client is deleted and later books again with the same Pro, when the new booking is created, then a fresh Client record and a fresh set of Activity Events begin -- the de-identified historical entries from the earlier relationship are never resurrected or merged into the new record's timeline.

**FEAT-16.SPEC-002-AC-23:** Given Support opens a session or a disputed booking's timeline and FEAT-19.SPEC-002 hands off the assembled support-view event, when this automation processes it, then one immutable Activity Event with actor "a support view" is written against Talia's Pro Account (not the Booking), and Support's view is not blocked by the write.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 13 | 13 |
| Outcome Paths | 5 | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Integration Spec: Card-Issuer Dispute Integration

## Overview

**Name:** Card-Issuer Dispute Integration
**ID:** FEAT-16.SPEC-003
**Type:** Integration
**Purpose:** Receives an inbound card-issuer dispute notice from the payment-processing capability, flags the affected booking, sets the Deposit Transaction's Disputed overlay without erasing its underlying outcome, and records the dispute as an Activity Event.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Receiving the payment-processing capability's card-issuer dispute notice for a Deposit Transaction
- Setting the Disputed overlay on the affected Deposit Transaction, additive to (never replacing) its existing outcome
- Handing off the dispute for the Pro notification and dashboard flag (owned elsewhere, cited below)
- Recording the dispute event through FEAT-16.SPEC-002's append-only mechanism
- Degradation behavior when the payment-processing capability is slow, down, or rejects, for every screen this integration affects
- Disclosure of what dispute metadata is read from the capability

**Non-Goals:**
- Deciding or adjudicating the dispute -- excluded per SC-17: Chairtime never rules on who is right between the Pro and the client; this spec supplies the record and the flag, nothing more
- Reading or storing card data of any kind -- excluded per SC-11: this spec reads only dispute metadata and outcome flags from the payment-processing capability, which alone owns card data
- Sending the Pro's dispute notification -- owned by FEAT-08.SPEC-006 (Pro Attention Alert), which this spec hands off to per feature-overview.md's own Communications field ("A Pro notification (via FEAT-08)")
- Feeding the dashboard's attention flag -- owned by FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation), which consume the Disputed overlay this spec sets
- Assembling or delivering the downloadable evidence summary -- owned by FEAT-16.SPEC-004 (Dispute Summary Download)
- Any outbound submission of evidence to the payment processor on the Pro's behalf -- excluded per feature-overview.md's own Non-Goals and SC-17; the Pro submits the downloaded summary herself through the processor's own channel

## Capability Category

**Category:** Payment processing -- card-issuer dispute notifications
**Dependency Source:** ASMP-31 -- "Payment-processing capability ... notify the product of card-issuer disputes" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing — card-issuer dispute notifications (ASMP-31)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-16, FEAT-12)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision; BRIEF.md records no user mandate for a specific payment processor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia sees a booking flagged the moment a client raises a card-issuer dispute | See a booking flagged when the client raises a dispute with their card issuer | FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation), FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Talia is notified of the dispute without checking the dashboard | See a booking flagged... | FEAT-08.SPEC-006 (Pro Attention Alert) |
| The disputed booking's Deposit Transaction carries a Disputed marker alongside its existing outcome | See a booking flagged... | FEAT-16.SPEC-001, FEAT-28 (money list, read-only reflection) |
| The dispute is recorded as a permanent, immutable fact on the booking's timeline | Reference this record when responding to a client's dispute | FEAT-16.SPEC-002 (Activity Event Recording), FEAT-16.SPEC-001 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Deposit Transaction reference | Deposit Transaction -- processor transaction reference (already held from FEAT-07.SPEC-005's original charge) | The capability's own dispute process needs to be tied to the original charge (this reference already exists from the original charge and is not newly sent for this integration -- listed here as context data already known to the capability) | Ties the dispute back to the exact charge in the capability's own systems |

No new outbound data is sent by this integration beyond what the original deposit charge (FEAT-07.SPEC-005) already shared with the capability. This spec is inbound-only: the payment-processing capability reports the dispute to the product; the product sends nothing further to initiate or advance the dispute itself. Booking details, client contact information, Pro Account details, and every other product entity never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Dispute notice (deposit reference, dispute reason category, dispute timestamp) | The payment-processing capability reports a card-issuer dispute against a captured deposit | Deposit Transaction -- status overlay set to Disputed; outcome_reason and timestamps updated to include the dispute event; Booking -- flagged for dashboard display |
| Dispute outcome metadata (won / lost / withdrawn), if reported later by the capability | The capability reports the dispute process concluding | Deposit Transaction -- outcome_reason updated to reflect the concluded dispute status; the original Captured/Forfeited/Refunded outcome is never erased |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Card-issuer dispute notice received | The payment-processing capability reports a client dispute against a captured deposit | Deposit Transaction's Disputed overlay set (additive, never replacing the underlying Captured/Forfeited/Refunded outcome); Booking flagged; a "disputed" Activity Event is written via FEAT-16.SPEC-002 | Talia's dashboard shows the dispute flag (FEAT-12.SPEC-002/FEAT-12.SPEC-005); Talia receives a Pro notification (FEAT-08.SPEC-006); the booking's timeline (FEAT-16.SPEC-001) shows the dispute banner and download entry point | FEAT-16.SPEC-002 (Activity Event Recording), FEAT-16.SPEC-001 (Booking Activity Timeline), FEAT-12.SPEC-002, FEAT-12.SPEC-005, FEAT-08.SPEC-006 |
| Dispute process concludes (won / lost / withdrawn) | The capability reports the dispute process has ended | Deposit Transaction's outcome_reason updated with the concluded dispute status alongside the retained Disputed overlay and original outcome; a corresponding Activity Event is written via FEAT-16.SPEC-002 | The booking's timeline (FEAT-16.SPEC-001) shows the concluded dispute event; no separate Pro notification is defined for this event beyond the existing dashboard flag update, since the founder has no further action to take once the processor's own process has concluded | FEAT-16.SPEC-002, FEAT-16.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-005 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12.SPEC-002 / FEAT-12.SPEC-005 (Attention List / Attention Flag Aggregation) | N/A -- this integration is purely inbound; there is no Talia-initiated request from the dashboard that can be "slow" toward the capability | If the capability's dispute-notification channel is unavailable, no new dispute notices arrive during the outage; the dashboard shows exactly the dispute flags it already knows about, with no error state, since a missing notification looks identical to "no dispute has occurred yet" from the product's perspective -- there is no in-progress request to fail visibly | N/A -- the dashboard never sends a request this capability could reject; it only reflects notices already received |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | N/A -- same reasoning: this integration never initiates a request from this screen | Same as above: a dispute notice delayed by an outage simply has not arrived yet; the timeline shows no dispute banner until the notice lands, with no error state | N/A -- no outbound request from this screen to reject |

## Consent and Disclosure

- **No client-facing or Pro-facing consent moment exists for this integration** -- the payment-processing capability's own terms (accepted once, when the Pro connected her payout account through FEAT-28) already cover dispute-notification reporting as part of standard card-processing service; this spec introduces no new outbound data element requiring a fresh disclosure, since the "Leaves the product" section above confirms no new data is sent to enable this integration beyond what the original deposit charge already shared.
- **What is never shared or requested** -- this integration never reads or requests card data, cardholder identity details, or any product entity (Booking, Client, Message) beyond the already-known Deposit Transaction reference; SC-11 keeps card data entirely with the payment processor.

## Edge Cases

- **A dispute notice arrives for a Deposit Transaction that has since been de-identified (client deletion processed, per XBR-19)** -- The Disputed overlay is applied to the retained, de-identified financial record; the dashboard flag and Pro notification still fire, since the dispute is a financial fact independent of the client's contact details having been removed.
- **The same dispute notice is delivered twice** -- The second delivery changes nothing: the Deposit Transaction's Disputed overlay is already set, the dispute Activity Event already exists, and no duplicate notification or duplicate timeline entry is created.
- **A "dispute concluded" event arrives before the original "dispute notice" event (out-of-order delivery)** -- The product holds the concluded-outcome data and applies it only once the original dispute notice is also received and the Disputed overlay is set; if the original notice never arrives, the concluded event alone is insufficient to flag a booking that was never shown as disputed, and this out-of-order case is flagged as a data inconsistency for support to investigate rather than silently applied.
- **A dispute is raised on a deposit that was already fully refunded** -- The Disputed overlay is set alongside the existing Refunded outcome per the entity's Contention rule (a Disputed overlay never erases the underlying outcome); Talia sees both facts on the timeline: the refund and the dispute.
- **The payment-processing capability is down when the dispute notice would otherwise have arrived** -- The notice simply has not been delivered yet; once the capability's channel recovers, the notice arrives and is processed normally with its own original dispute timestamp, not the recovery time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Activity Event Recording) | Triggers (outbound, external-event source) | The dispute notice is the external-event trigger for the "disputed" Activity Event write |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Affects (outbound) | The dispute banner and flagged entry reflect this spec's write |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Governed by | Rule spec listing this spec in its Enforced By table: the "disputed" Activity Event this spec hands to FEAT-16.SPEC-002 is subject to the same append-only, immutable, non-editable guarantee, and the Disputed overlay is scoped to Deposit Transaction, not Activity Event |
| FEAT-16.SPEC-004 (Dispute Summary Download) | References (inbound) | That spec's download entry is available only once this spec has flagged the booking |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | The dispute notice hands off to this notification |
| FEAT-12.SPEC-002 (Attention List) | Affects (outbound) | Consumes the dispute flag for dashboard display |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | Affects (outbound) | Aggregates the dispute flag among other attention items |
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | References (inbound) | The original deposit charge this dispute is raised against |

## Analytics and Success Signals

- **card_dispute_flagged** (booking reference) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of dispute frequency, since Chairtime never adjudicates disputes and has no target rate to measure them against.
- **card_dispute_concluded** (outcome: won / lost / withdrawn) -- N/A -- no connected success-metrics.md metric; retained purely for operational visibility into dispute resolution outcomes over time.

## Acceptance Criteria

**FEAT-16.SPEC-003-AC-01:** Given a client raises a card-issuer dispute against a booking's captured deposit, when the payment-processing capability reports the dispute, then the Deposit Transaction's Disputed overlay is set without changing its existing Captured/Forfeited/Refunded outcome.

**FEAT-16.SPEC-003-AC-02:** Given a dispute notice is received, when the overlay is set, then a "disputed" Activity Event is written for the affected booking through FEAT-16.SPEC-002.

**FEAT-16.SPEC-003-AC-03:** Given a dispute notice is received, when the flag is applied, then Talia's dashboard attention list (FEAT-12.SPEC-002/FEAT-12.SPEC-005) shows the dispute flag and Talia receives a Pro notification via FEAT-08.SPEC-006.

**FEAT-16.SPEC-003-AC-04:** Given the payment-processing capability later reports the dispute concluded as "lost," when the event arrives, then the Deposit Transaction's outcome_reason is updated to reflect the concluded status while the Disputed overlay and original outcome remain visible.

**FEAT-16.SPEC-003-AC-05:** Given the payment-processing capability's dispute-notification channel is down, when Talia views her dashboard during the outage, then it shows exactly the dispute flags already known, with no error state suggesting something is broken.

**FEAT-16.SPEC-003-AC-06:** Given the same dispute notice is delivered twice by the capability, when the second delivery is processed, then no duplicate Activity Event or duplicate Pro notification is created.

**FEAT-16.SPEC-003-AC-07:** Given a dispute is raised on a deposit that was already refunded, when the notice is processed, then Talia's timeline (FEAT-16.SPEC-001) shows both the refund and the dispute as separate, coexisting facts.

**FEAT-16.SPEC-003-AC-08:** Given a dispute notice arrives for a client whose record has since been de-identified per XBR-19, when the notice is processed, then the Disputed overlay is still applied to the retained financial record and the dashboard flag still fires.

**FEAT-16.SPEC-003-AC-09:** Given Talia asks what data this integration reads, when she reviews the disclosure, then it states plainly that only dispute metadata and outcome flags are read, never card data, per SC-11.

**FEAT-16.SPEC-003-AC-10:** Given a "dispute concluded" event arrives before its corresponding original dispute notice, when the out-of-order delivery is detected, then the concluded outcome is held rather than silently applied to a booking never shown as disputed.

**FEAT-16.SPEC-003-AC-11:** Given Talia wants to know who is right in a dispute, when she looks to Chairtime for a ruling, then no such feature exists anywhere in this spec or its connected specs -- only the record and the flag are provided, per SC-17.

**FEAT-16.SPEC-003-AC-12:** Given the payment-processing capability's channel was down and then recovers, when a delayed dispute notice finally arrives, then it is processed with its own original dispute timestamp, not the time of recovery.

**FEAT-16.SPEC-003-AC-13:** Given no new outbound data element is introduced by this integration beyond the original deposit charge's own transaction reference, when a downstream reviewer checks Consent and Disclosure, then no new disclosure moment is required, and this is stated explicitly rather than left unaddressed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 2 | 2 |
| Degradation Paths | 2 (2 screens; N/A cells justified) | 2 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



# Automation Spec: Dispute Summary Download

## Overview

**Name:** Dispute Summary Download
**ID:** FEAT-16.SPEC-004
**Type:** Automation
**Purpose:** Assembles a plain-language, shareable summary of a disputed booking's timeline -- policy shown and acknowledged, booking time, messages sent, and no-show mark -- and hands it to the Pro as a downloadable file to submit as evidence with the payment processor.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Assembling a plain-language summary from the disputed booking's existing Activity Event, Booking, Deposit Transaction, Cancellation Policy, and Message data
- Producing that summary as a file the Pro can download to her own device
- Restricting this action to bookings that currently carry the Disputed overlay

**Non-Goals:**
- Submitting the summary to the payment processor on the Pro's behalf -- excluded per feature-overview.md's Key Capabilities ("to use as evidence with the payment processor") and SC-17; the Pro submits it herself through the processor's own channel
- Deciding the dispute's outcome -- excluded per SC-17: this automation only assembles a factual record, it never argues a position or renders a verdict
- Detecting or flagging the dispute itself -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration), which sets the Disputed overlay this automation checks for
- Including the Pro's private client notes in the summary -- excluded per XBR-24 and FEAT-16.SPEC-005: the summary is built for external submission to the payment processor, so it carries strictly less than even Support's internal view, which itself already excludes private notes
- Generating a summary for a non-disputed booking -- this automation has no trigger path that reaches a booking without an active Disputed overlay, since its only entry point (FEAT-16.SPEC-001) hides the download action for non-disputed bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests the evidence summary | FEAT-16.SPEC-001 (Booking Activity Timeline) | The Pro taps "Download evidence summary" on a booking whose Deposit Transaction currently carries the Disputed overlay | Booking reference |

## Processing Logic

1. Confirm the referenced Booking's Deposit Transaction currently carries the Disputed overlay (set by FEAT-16.SPEC-003). If it does not, refuse the request (see Outcome Definitions).
2. Read the booking's full ordered Activity Event history (the same data FEAT-16.SPEC-001 renders): the policy version and wording shown and its acknowledgment timestamp, the appointment time, every message sent and its delivery outcome (including any gap per XBR-17), any cancellation/reschedule event, the no-show mark (if present) and its timestamp, and the deposit outcome including the dispute event itself.
3. Assemble this data into a plain-language summary document, organized chronologically, using the same event descriptions the timeline screen already shows -- no new wording or interpretation is introduced beyond what the timeline already states as fact.
4. Exclude the Pro's private client notes from the assembled summary entirely, per XBR-24 and FEAT-16.SPEC-005.
5. Render the assembled summary as a downloadable file and hand it to the Pro's device.
6. Since this automation assembles from data already loaded by the timeline (feature-overview.md's Responsiveness note: "assembles from already-loaded timeline data"), the assembly completes near-instantaneously rather than as a long-running export.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Summary produced | The booking's Deposit Transaction carries the Disputed overlay at request time | None -- this automation is read-only against product data; it produces a file, it does not write one | The file downloads to Talia's device; FEAT-16.SPEC-001 shows a brief confirmation that the download started | FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Request refused (not disputed) | The booking's Deposit Transaction does not carry the Disputed overlay at request time (e.g., the overlay was somehow cleared or the request is stale) | None | Talia sees: "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." and no file is produced | FEAT-16.SPEC-001 |
| Assembly failure | The automation cannot read one or more required data elements (e.g., a transient read failure) | None | Talia sees: "The evidence summary could not be prepared right now. Try again in a moment." with a retry option; no partial or corrupted file is ever handed to her device | FEAT-16.SPEC-001 |

## Data Model

**Reads:** Activity Event (event_type, time, actor, details) for the booking; Booking (service, start_time, state); Deposit Transaction (status including Disputed overlay, outcome_reason, timestamps); Cancellation Policy (version, plain_language_wording); Message (type, channel, send time, delivery_status) -- all read-only, identical in source to what FEAT-16.SPEC-001 already renders.
**Creates:** A downloadable summary file, handed to the Pro's device; this file is not itself a product entity and is not stored by the product beyond the download hand-off.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-22: the plain timeline summary is made available to submit as evidence once a card-issuer dispute has flagged the booking; this automation is exactly that mechanism.
- The summary never includes the Pro's private client notes, per XBR-24 and FEAT-16.SPEC-005, regardless of how much detail the notes might otherwise add to the Pro's case.
- The summary is available only for a booking whose Deposit Transaction currently carries the Disputed overlay -- there is no path to request one for a non-disputed booking.
- The summary states facts only, drawn verbatim from the same Activity Event history the timeline already shows the Pro; this automation introduces no new characterization, argument, or recommendation, consistent with SC-17 (Chairtime never rules on or manages the dispute process itself).
- The Pro alone submits the downloaded file to the payment processor; no automated hand-over exists, per feature-overview.md's own Non-Goals.

## Edge Cases

- **The dispute is resolved (won, lost, or withdrawn) between the Pro's first and second download of the summary** -- The Disputed overlay remains set (per FEAT-16.SPEC-003's Data Exchanged: the concluded outcome is added alongside the overlay, which is not cleared), so a second download remains available and reflects the same underlying timeline plus the now-visible concluded-dispute entry.
- **Two rapid taps on "Download evidence summary" on the same device (trigger fires while a previous run is in flight)** -- The second tap while the first assembly is in flight is ignored (the action shows a brief in-progress state); only one file is handed to the device per request.
- **Talia requests the summary from two signed-in devices at effectively the same time (concurrent trigger firing)** -- Each device's request runs its own independent assembly against the same read-only source data; both succeed independently and each device receives its own downloaded file, since assembly never writes shared state that the two runs could conflict over.
- **A gap exists in the timeline (e.g., a failed-then-fallback message)** -- The gap is included in the summary exactly as the timeline shows it, per XBR-17; the summary never presents a falsely clean record.
- **The booking has very few events (e.g., created, policy acknowledged, deposit paid, then immediately disputed with no messages or no-show mark)** -- The summary includes exactly the events that exist; it is not padded with placeholder sections for events that never occurred.
- **The Pro requests the summary while offline** -- The action is disabled with "This action needs a connection." per FEAT-16.SPEC-001's Offline/Degraded state, since assembly and hand-off require connectivity even though the underlying data was already loaded.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Triggered by (inbound) | "Download evidence summary" initiates this automation |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | References (inbound) | The Disputed overlay this automation checks for is set there |
| FEAT-16.SPEC-002 (Activity Event Recording) | References (inbound) | The Activity Event history this automation assembles from is written there |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Governed by | Rule spec that governs this automation: its Enforced By table names this spec for the private-notes exclusion on assembly (XBR-24), and its Authorization Rules define who may request a summary |

## Analytics and Success Signals

- **dispute_summary_downloaded** (booking reference) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal (named in feature-overview.md's Signals field) is retained for operational observability of how often the record is actually used as evidence.
- **dispute_summary_assembly_failed** (reason) -- N/A -- no connected success-metrics.md metric; retained to observe whether the evidence path is reliable when Talia needs it most.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Talia is viewing a disputed booking's timeline, when she taps "Download evidence summary," then a plain-language file downloads to her device containing the policy shown and acknowledged, the booking time, messages sent, and the no-show mark (if any).

**FEAT-16.SPEC-004-AC-02:** Given a booking's timeline includes a message-delivery gap, when Talia downloads the summary, then the gap appears in the summary exactly as it appears on the timeline.

**FEAT-16.SPEC-004-AC-03:** Given the summary is assembled, when Talia opens the downloaded file, then it contains no reference to her private client notes about that client.

**FEAT-16.SPEC-004-AC-04:** Given a booking's Deposit Transaction does not carry the Disputed overlay, when a request for its evidence summary somehow reaches this automation, then it is refused with "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." and no file is produced.

**FEAT-16.SPEC-004-AC-05:** Given the automation cannot read the booking's required data at request time, when the assembly fails, then Talia sees "The evidence summary could not be prepared right now. Try again in a moment." with a retry option, and no partial file is produced.

**FEAT-16.SPEC-004-AC-06:** Given Talia downloads a disputed booking's summary and the dispute later resolves as "lost," when she downloads it again, then the file still assembles, now also reflecting the concluded-dispute entry alongside the original facts.

**FEAT-16.SPEC-004-AC-07:** Given Talia taps "Download evidence summary" twice rapidly, when the second tap registers while the first is still assembling, then it is ignored and exactly one file is handed to her device.

**FEAT-16.SPEC-004-AC-08:** Given a disputed booking has only a handful of events (created, policy acknowledged, deposit paid, disputed), when Talia downloads the summary, then it contains exactly those events with no placeholder sections for events that never occurred.

**FEAT-16.SPEC-004-AC-09:** Given Talia loses connectivity while viewing a disputed booking's timeline, when she looks for the download action, then it is disabled with "This action needs a connection."

**FEAT-16.SPEC-004-AC-10:** Given Talia has downloaded the summary, when she looks for a way to send it to the payment processor from within the product, then no such feature exists -- she must submit it herself through the processor's own channel.

**FEAT-16.SPEC-004-AC-11:** Given the assembled summary states the sequence of events, when Talia reads it, then it contains no argument, recommendation, or verdict about who is right -- only the recorded facts.

**FEAT-16.SPEC-004-AC-12:** Given a booking's cancellation policy version and acknowledgment time are part of its timeline, when Talia downloads the summary, then that exact version and timestamp appear in the file.

**FEAT-16.SPEC-004-AC-13:** Given the disputed booking includes a no-show mark, when Talia downloads the summary, then the no-show mark and its timestamp appear as one of the summarized facts.

**FEAT-16.SPEC-004-AC-14:** Given the summary is assembled from data already loaded by the timeline screen, when Talia taps download, then the file is produced near-instantaneously rather than showing a long-running export progress state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Activity Record Immutability & Visibility Rules

## Overview

**Name:** Activity Record Immutability & Visibility Rules
**ID:** FEAT-16.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs append-only enforcement (no role, including the Pro, may edit an entry), View-only access for the Pro and Support, retention tied to the Booking's life, de-identification on client deletion, and exclusion of the Pro's private client notes from the Support view.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record
**Governed Entity:** Activity Event

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Activity Event entity (what may be set, when, and by what)
- Authorization rules for every action the product defines on Activity Event, across every role in the Access Matrix
- Retention and de-identification rules tied to the Booking's life and client deletion
- The Support-view exclusion of the Pro's private client notes

**Non-Goals:**
- Deciding what event_type values exist and what triggers each one -- owned by FEAT-16.SPEC-002 (Activity Event Recording), which enumerates every trigger and its resulting event_type; this spec governs the entity's rules once an entry is written, not the catalog of writers
- Rendering the timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline), which references this spec's Access rules for its own Access and Visibility table rather than restating them
- The dispute-specific overlay rule on Deposit Transaction (a different entity) -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration); this spec governs Activity Event only, per its Governed Entity
- Client deletion's own execution mechanics (removing contact details and notes from the Client record itself) -- owned by FEAT-13.SPEC-004 (Client Deletion Execution); this spec governs only the resulting Activity Event de-identification, not the Client-record deletion process itself

## Governed Entity

**Entity:** Activity Event
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of qualifying action this entry records (e.g., created, policy_acknowledged, deposit_succeeded, message_sent, cancelled, no_show_marked, disputed) |
| time | date (with time) | When the recorded action occurred |
| actor | enum | Who or what caused the event: Client, Pro, the product automatically, or a support view (written via FEAT-19.SPEC-002 through FEAT-16.SPEC-002) |
| details | text (structured) | The event's specifics -- e.g., policy version and wording shown, message sent and channel, deposit outcome and amount; may include a Pro's private client note reference before de-identification, per the entity's own Data Sensitivity line |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Booking Activity Timeline | On screen render: Access and Visibility (View for the Pro and Support, private-notes exclusion for Support), no edit/delete control ever rendered |
| FEAT-16.SPEC-002 | Activity Event Recording | On every write: append-only enforcement (no update path beyond the client-deletion de-identification conversion), retention (never hard-deleted while the Booking exists) |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | On its own write (the dispute event): the same append-only enforcement applies, since it writes through FEAT-16.SPEC-002's mechanism |
| FEAT-16.SPEC-004 | Dispute Summary Download | On assembly: private-notes exclusion (the downloadable summary excludes private notes exactly as the Support view does) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| event_type | Must be one of the fixed set of qualifying event types defined by FEAT-16.SPEC-002's Trigger Definition | Always | On write | N/A -- this is a system-internal write, not a user-entered field; there is no user-facing error message because no role ever enters an event_type directly | Yes (a write with an unrecognized event_type does not occur -- no enforcing spec offers a path to attempt one) |
| time | Required, set automatically to the moment of the qualifying action; never entered or edited by any role | Always | On write | N/A -- not user-entered | Yes |
| actor | Required, derived automatically from the triggering spec (Client, Pro, the product automatically, or a support view) | Always | On write | N/A -- not user-entered | Yes |
| details | No validation beyond data type once assembled by the triggering spec; content requirements (what must be included per event_type) are FEAT-16.SPEC-002's concern, not this spec's | Always | -- | -- | -- |

No field on this entity is ever entered directly by a person through a form -- every field is system-derived at the moment of a qualifying action (per FEAT-16.SPEC-002's Processing Logic), so no field-level rule above carries a user-facing error message; each row exists to confirm the field was considered, not accidentally skipped.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| actor consistency with event_type | actor, event_type | The actor recorded must be consistent with who or what genuinely caused the event (e.g., "no_show_marked" always has actor Pro; "message_sent" always has actor "the product automatically"; a client-initiated cancellation has actor Client) -- this consistency is guaranteed structurally by FEAT-16.SPEC-002's Processing Logic, which derives actor from the specific trigger, never independently | N/A -- this is a structural guarantee of the recording automation, not a user-facing validation that can fail |
| details content matches event_type | details, event_type | The details field's structure follows from event_type (e.g., a "policy_acknowledged" entry's details always includes a policy version and wording; a "message_sent" entry's details always includes a channel and delivery_status) | N/A -- structural guarantee of FEAT-16.SPEC-002, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create (write) an entry | The product automatically (via FEAT-16.SPEC-002 and FEAT-16.SPEC-003) | Only in response to a qualifying trigger defined in FEAT-16.SPEC-002's Trigger Definition | N/A -- no role ever attempts to create an entry directly; there is no create control anywhere in the product for any role |
| View an entry (single or as part of a timeline) | The Pro | Only entries belonging to her own bookings (or her own Pro Account for account-level entries such as payout status) | -- |
| View an entry (single or as part of a timeline) | Platform Operator (Support) | Only while actively viewing the one Pro account under a help request (per XBR-24), and never the Pro's private client notes within any entry's details | Any entry's private-note content is simply absent from Support's rendering; no "hidden" placeholder appears -- the rest of that entry's details render normally |
| View an entry | The Client | Never | The Client is never shown this internal timeline directly (FEAT-16.SPEC-001's Access and Visibility); she sees only her own booking's outcomes through her own booking view, not this operational record |
| Edit an entry | Any role, including the Pro | Never -- no exception exists | No edit control is ever rendered for any role, on any screen, for any entry; the entry is permanent from the moment it is written |
| Delete (hard) an entry | Any role, including the Pro | Never -- no exception exists | No delete control is ever rendered for any role; hard deletion never occurs while the associated Booking exists, and never occurs at all even after client deletion (only de-identification occurs) |
| Convert an entry to de-identified form | The product automatically (via FEAT-16.SPEC-002) | Only when a client deletion is processed for the client whose bookings the entries belong to (per XBR-19) | N/A -- this is not a role-initiated action; it is the one automatic, system-only exception to full immutability, and it strips contact/note content only, never financial or timeline facts |
| Download a plain-language summary of an entry set | The Pro | Only for a booking whose Deposit Transaction currently carries the Disputed overlay (FEAT-16.SPEC-004) | The download action is not shown for a non-disputed booking; a stale request that reaches the automation anyway is refused with "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." |
| Download a plain-language summary of an entry set | Platform Operator (Support) | Never (SC-05: Support has View-only access with no action control) | The download action is never rendered for Support |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| time | Set to the exact moment the qualifying action occurs, as reported by the triggering spec | On create only | No |
| actor | Derived from which triggering spec fired and its own context (e.g., FEAT-10.SPEC-004 always yields actor Client; FEAT-30.SPEC-007 always yields actor Pro; FEAT-08's message specs always yield actor "the product automatically") | On create only | No |
| event_type | Derived from the specific trigger row in FEAT-16.SPEC-002's Trigger Definition that fired | On create only | No |
| details | Assembled from the triggering spec's own available data at the moment of the trigger | On create only | No |

## Business Rules

- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit.
- XBR-19: on client deletion, this entity's entries are converted to de-identified form (contact and note content stripped) rather than deleted; financial and timeline facts are retained per SC-22.
- XBR-24: Support's access to this entity is read-only, used only after a Pro's help request, and never includes the Pro's private client notes.
- Retention: entries are retained for as long as the associated Booking exists (feature-overview.md's Validation & Limits); after a client deletion, the de-identified entries persist indefinitely per SC-22, since financial and timeline facts required for dispute/audit purposes are never purged outright.
- The one and only automatic write-after-creation this entity ever undergoes is the de-identification conversion on client deletion (per XBR-19); this is not treated as a violation of immutability, since it strips only contact/note content and never alters an event's recorded facts (event_type, time, actor category, financial amounts, or outcomes).
- An Activity Event is never read or opened individually -- it exists only as part of a booking's ordered timeline (per the Entity-Lifecycle Coverage Matrix's Read (single): N/A).

## Edge Cases

- **A client deletion is requested while a new Activity Event for that client's booking is being written at the same moment** -- The de-identification conversion applies to entries that exist at the moment the deletion completes; any entry written after that moment for the now-deleted client carries no contact or note content from the outset, consistent with the already-converted entries (see FEAT-16.SPEC-002's own concurrency handling).
- **Support's help-request session ends while they are mid-view of a timeline** -- Access is revoked immediately; any further attempt to view the timeline requires a fresh help-request-gated session, per XBR-24's "used only after a Pro's help request" condition.
- **A Pro attempts to edit an entry through any indirect path (e.g., editing the underlying Booking or Message record after the fact)** -- Editing the underlying Booking, Message, or Deposit Transaction record (where those edits are themselves permitted by their own owning specs) never rewrites an already-written Activity Event's recorded details; the entry remains a fixed snapshot of what was true at the moment it was written, even if the source record later changes through its own governing rules.
- **The Booking a set of entries belongs to reaches its retention end (the account itself is closed, per XBR-20)** -- Entries are retained through the account's 30-day cooling-off period; only after the account closure's own data-deletion step (owned by FEAT-29) do the entries become subject to that closure's own de-identified-financial-record retention, following the same de-identification principle as an individual client deletion rather than a hard delete.
- **A de-identified entry's remaining financial/timeline facts are later needed for a dispute** -- The retained facts (event_type, time, amounts, outcomes) remain fully usable as dispute evidence even without the stripped contact/note content, since a dispute concerns the transaction facts, not the client's contact details.

## Acceptance Criteria

**FEAT-16.SPEC-005-AC-01:** Given a qualifying action occurs (e.g., Talia marks a no-show), when FEAT-16.SPEC-002 writes the resulting entry, then its event_type, time, actor, and details are all set automatically with no user-entered value anywhere in the write.

**FEAT-16.SPEC-005-AC-02:** Given Talia views her own booking's timeline, when she looks for an edit or delete control on any entry, then none exists anywhere on the screen or in any connected spec.

**FEAT-16.SPEC-005-AC-03:** Given Talia attempts to alter an Activity Event through any indirect path, when she edits the underlying Booking or Message record instead, then the previously written Activity Event's own recorded details remain unchanged.

**FEAT-16.SPEC-005-AC-04:** Given Talia views her own booking's timeline, when the screen renders, then she sees every entry belonging to that booking, including her own private client notes where relevant to an entry's details.

**FEAT-16.SPEC-005-AC-05:** Given Support opens a Pro's booking timeline after a help request, when an entry's details would normally include the Pro's private client note, then that note content is absent from Support's rendering while the rest of the entry's details render normally.

**FEAT-16.SPEC-005-AC-06:** Given Riley (the Client) attempts to view this internal timeline, when the attempt is made, then she is never shown any part of it -- she sees only her own booking's outcomes through her own booking view.

**FEAT-16.SPEC-005-AC-07:** Given a client deletion is processed for one of Talia's clients, when FEAT-16.SPEC-002 executes the conversion, then every existing Activity Event for that client's bookings has its contact and note content stripped while event_type, time, financial amounts, and outcomes remain unchanged.

**FEAT-16.SPEC-005-AC-08:** Given a de-identified Activity Event, when Talia or Support later views it, then it displays its retained financial and timeline facts with no contact or note content and no "[hidden]" placeholder in their place.

**FEAT-16.SPEC-005-AC-09:** Given a booking's Activity Events, when the associated Booking still exists, then the entries are retained indefinitely with no expiry or automatic purge.

**FEAT-16.SPEC-005-AC-10:** Given Talia's account is closed and its 30-day cooling-off period elapses, when the account closure's data-deletion step runs, then the account's Activity Events are de-identified following the same principle as an individual client deletion, never hard-deleted outright.

**FEAT-16.SPEC-005-AC-11:** Given Talia is viewing a disputed booking's timeline, when she taps the download action, then FEAT-16.SPEC-004's own authorization check (Disputed overlay present) determines whether the summary is produced.

**FEAT-16.SPEC-005-AC-12:** Given Support is viewing a Pro's timeline, when Support looks for a download-summary action, then none is rendered, per the Authorization Rules row denying that action to Support entirely.

**FEAT-16.SPEC-005-AC-13:** Given a new Activity Event write is attempted for a trigger not listed in FEAT-16.SPEC-002's Trigger Definition, when the write path is examined, then no such path exists anywhere in the product -- the only writers are the enumerated triggers and the client-deletion conversion.

**FEAT-16.SPEC-005-AC-14:** Given Support's help-request-gated session ends, when Support attempts to continue viewing a timeline, then access is denied and a fresh help-request-gated session is required.

**FEAT-16.SPEC-005-AC-15:** Given an Activity Event's actor is derived from its triggering spec, when a client-initiated cancellation writes its entry, then the actor field reads Client, never Pro or "the product automatically."

**FEAT-16.SPEC-005-AC-16:** Given a message-delivery automation writes its entry, when the entry is created, then its actor field reads "the product automatically," consistent with every automated message trigger.

**FEAT-16.SPEC-005-AC-17:** Given a client is deleted and later re-books with the same Pro, when the new booking begins accumulating Activity Events, then those new entries start a fresh record entirely separate from the earlier, now de-identified entries -- the de-identified history is never merged into or resurrected for the new record.

**FEAT-16.SPEC-005-AC-18:** Given the entity's fields are examined for coverage, when each of event_type, time, actor, and details is checked against the Field Validation Rules table, then every field has an explicit rule or an explicit "not user-entered" rationale -- none is left unaddressed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |


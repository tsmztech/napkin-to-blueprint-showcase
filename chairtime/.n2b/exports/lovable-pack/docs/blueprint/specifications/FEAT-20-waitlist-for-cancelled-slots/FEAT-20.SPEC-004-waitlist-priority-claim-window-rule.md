---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-20.SPEC-004
spec_name: Waitlist Priority & Claim Window Rule
spec_slug: waitlist-priority-claim-window-rule
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Waitlist Priority & Claim Window Rule

## Overview

**Name:** Waitlist Priority & Claim Window Rule
**ID:** FEAT-20.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs which Requested entries match a freed slot, the 30-minute claim window, how the window interacts with general public availability, and how contested or withdrawn claims resolve.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots
**Governed Entity:** Waitlist Entry -- specifically its matching, notification-window, and contention behavior once a slot frees

## Scope and Non-Goals

**In Scope:**
- The matching test that decides whether a Requested entry corresponds to a freed slot
- The 30-minute claim window's exact boundaries and what happens at each edge
- How the claim window interacts with the slot's general public availability (the recorded reading in this Brief's Shared Context)
- How a leave request resolves against a pending Notified claim (the contention rule this spec is Enforced By FEAT-20.SPEC-002 for)
- How a contested claim (two or more Notified entries for the same freed slot) resolves

**Non-Goals:**
- Detecting that a cancellation has freed a slot in the first place -- owned by FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching); this spec supplies the matching test and window rule that automation applies, not the detection of the triggering cancellation itself
- Converting a claimed slot into a Booking -- owned by FEAT-20.SPEC-006 (Waitlist Claim Conversion); this spec governs only which entry is eligible to claim and for how long, not the booking-completion mechanics
- The underlying free/not-free slot truth and the generic first-to-pay-wins tie-break mechanism for any booking path -- owned by FEAT-03.SPEC-005 (Slot Contention Resolution Rules, XBR-01); this spec builds the waitlist-specific priority window on top of that mechanism, per FEAT-03.SPEC-005's own stated Non-Goal excluding the waitlist window from its scope

## Governed Entity

**Entity:** Waitlist Entry, plus its contention against the freed slot's general availability
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| service | reference | The Service this entry is waitlisted for |
| start_date / end_date | date | The requested day or range this entry matches against |
| state | enum (Requested, Notified, Converted, Expired) | The entry's current lifecycle state |
| claim_deadline | date/time | Set to the moment of notification plus platform parameter: `waitlist-claim-window-minutes`, once Notified |

**Referenced (read-only):** Booking (freed slot's service, date, and time), via FEAT-03's live slot truth.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-20.SPEC-005 | Cancellation-Triggered Waitlist Matching | At the moment a freed-slot signal arrives -- applies the matching test to find every eligible Requested entry |
| FEAT-20.SPEC-002 | My Waitlists | At the moment Riley confirms Leave on an entry that may have a pending Notified claim |
| FEAT-20.SPEC-006 | Waitlist Claim Conversion | At the moment a notified client completes a booking for the matching slot -- resolves any contested claim |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | At the moment a Notified entry's claim_deadline passes unclaimed |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | Supplies the underlying free/not-free slot truth this spec's window rule builds on |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| service | No validation beyond data type -- already validated at creation by FEAT-20.SPEC-003; this spec only reads it for matching | Always | -- | -- | -- |
| start_date / end_date | No validation beyond data type -- already validated at creation; this spec only reads them for matching | Always | -- | -- | -- |
| state | Must be Requested for an entry to be eligible for matching by FEAT-20.SPEC-005 | Always | At the moment a freed-slot signal is evaluated | N/A -- read-only evaluation, no user-facing error; a non-Requested entry is simply excluded from the match set | No |
| claim_deadline | Must be unset (entry not yet Notified) for a match to set it; once set, it is never recomputed or extended | Always | At the moment of notification | N/A -- read-only evaluation | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Matching test | service, start_date, end_date, state | An entry matches a freed slot when: state is Requested, the freed slot's service equals the entry's service, and the freed slot's date falls within [start_date, end_date] inclusive | N/A -- a computed boolean, not a user-facing error |
| Claim window boundary | claim_deadline | A Notified entry remains claimable strictly before claim_deadline; at or after claim_deadline, the claim window has lapsed and the entry is no longer eligible to claim (governed for expiry by FEAT-20.SPEC-007) | N/A -- expressed to the client as the countdown on FEAT-20.SPEC-002 and the deadline stated in FEAT-20.SPEC-008 |
| Priority-before-public rule | claim_deadline | For the full platform parameter: `waitlist-claim-window-minutes` following notification, the freed slot does not appear on FEAT-05's general public slot list; it is reachable only through a matching client's claim link. Once the window lapses (or immediately, if zero entries matched), the slot appears on the general public list like any other open time -- this is the recorded reading of XBR-28 in this Brief's Shared Context, distinguishing the underlying slot truth (always free the instant it is freed, per FEAT-03) from public *listing* (deferred for the window) | N/A -- this is a display/listing rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Be matched and notified for a freed slot | The Client (Riley) | Only her own Requested entries that pass the matching test | -- |
| Claim a notified opening | The Client (Riley) | Only while her own entry's claim_deadline has not yet passed, and only for the exact matched slot | If the window has lapsed: FEAT-20.SPEC-007's expiry handling applies (no claim action remains available; see that spec) |
| Claim a notified opening after another notified client already booked it | The Client (Riley) | Never for that specific slot -- the slot is gone the instant the first booking completes | Riley sees FEAT-03's plain "just taken" message (XBR-01), and her own entry remains Requested for the next opportunity (per this spec's contention resolution below), not deleted or expired by this event |
| Leave (delete) an entry with a pending Notified claim | The Client (Riley) | Always -- a leave request wins over a pending notification, with no condition attached | -- (the leave always succeeds; there is no denial path for this action) |
| See who else is notified for the same opening | The Client (Riley) | Never | She is never shown whether or how many other clients were notified for the same slot |
| See individual notified clients for a freed slot | The Pro (Talia) | Never -- her View access is the aggregate demand count only (FEAT-12), never per-client notification state | -- |
| View a specific client's notification/claim state | Platform Operator (Support) | View-only, for troubleshooting, via FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| claim_deadline | Derived: the exact moment of notification plus platform parameter: `waitlist-claim-window-minutes` | Set once, at the moment FEAT-20.SPEC-005 transitions the entry to Notified | No -- fixed and never extended, paused, or recalculated for any reason |
| match set | Derived: every Requested entry (across all clients waitlisted with this Pro) whose service and date satisfy the Matching test | Computed fresh each time a freed-slot signal arrives | No |

## Business Rules

- XBR-28 governs this spec entirely: a slot freed by a cancellation becomes publicly bookable immediately at the underlying slot-truth level (FEAT-03 never blocks it), while matching waitlisted clients are notified first and have platform parameter: `waitlist-claim-window-minutes` of priority before it returns to general public listing (FEAT-05).
- Notification is simultaneous, not sequential: every matching Requested entry is transitioned to Notified and notified at the same moment (FEAT-20.SPEC-005) -- there is no queue or turn order, per this Brief's Non-Goals ("Sequential, turn-based waitlist claiming" is explicitly excluded).
- First-to-complete-booking wins a contested claim (XBR-01, via FEAT-03.SPEC-005's underlying tie-break mechanism): when two or more Notified clients attempt to claim the same freed slot, whichever completes payment first wins it; every other notified client's entry remains Requested, unconverted and unexpired by this event, so they remain eligible for the next matching opening.
- A leave request unconditionally wins over a pending Notified claim (enforced by FEAT-20.SPEC-002): there is no scenario in which a leave is refused or delayed because a notification is in flight.
- The claim window is fixed at platform parameter: `waitlist-claim-window-minutes` for every entry, every Pro, and every slot -- it is never configured per Pro or extended for any individual claim.

## Edge Cases

- **Zero Requested entries match a freed slot** -- The slot appears on the general public list immediately, with no priority window elapsing, since there is no one to notify (per the Priority-before-public rule's "or immediately, if zero entries matched" clause).
- **Exactly one Requested entry matches** -- That entry is Notified alone; the window and its claim behave identically to the multi-match case, just with a single eligible claimant.
- **Multiple Requested entries match the same freed slot** -- All are transitioned to Notified simultaneously and all receive the opening notification (FEAT-20.SPEC-008) at the same moment; the first to complete a booking wins per the contention rule above.
- **A matching entry belongs to a client who already holds another Notified entry for a different freed slot at the same time** -- Each entry and its claim window are independent; claiming one has no effect on the other, and both remain separately actionable within their own windows.
- **A client's Notified entry's claim_deadline is reached at the exact instant she taps "Claim now"** -- Whichever event's timestamp is earlier governs: if the claim action reaches the booking flow before claim_deadline, it proceeds as an ordinary in-window claim; if claim_deadline has already passed, FEAT-20.SPEC-007's expiry handling applies and the claim link no longer completes a priority-window booking (the slot is then evaluated against general availability like any other attempt).
- **A client leaves an entry that has already converted (a rare race where Leave is tapped just as her own claim commits)** -- Not possible in practice: FEAT-20.SPEC-002 disables the Leave action the instant a claim conversion (FEAT-20.SPEC-006) commits for that entry, since a Converted entry is terminal and offers no Leave action.
- **The Pro cancels her own cancellation policy's window mid-flight while entries are Notified** -- Has no effect on this spec's rules: the claim window and matching logic depend only on the Waitlist Entry and the freed slot, never on the Cancellation Policy that produced the original cancellation.

## Acceptance Criteria

**FEAT-20.SPEC-004-AC-01:** Given a cancellation frees a slot for a service and date matching Riley's Requested entry, when FEAT-20.SPEC-005 evaluates the matching test, then Riley's entry is included in the match set.

**FEAT-20.SPEC-004-AC-02:** Given a freed slot's service does not match Riley's Requested entry's service, when the matching test runs, then her entry is excluded from the match set.

**FEAT-20.SPEC-004-AC-03:** Given a freed slot's date falls outside Riley's requested [start_date, end_date] range, when the matching test runs, then her entry is excluded.

**FEAT-20.SPEC-004-AC-04:** Given Riley's entry is matched and Notified, when claim_deadline is computed, then it is set to the notification moment plus platform parameter: `waitlist-claim-window-minutes`, and never recalculated afterward.

**FEAT-20.SPEC-004-AC-05:** Given no Requested entry matches a freed slot, when the matching test finds zero matches, then the slot appears on FEAT-05's general public list immediately with no priority window elapsing.

**FEAT-20.SPEC-004-AC-06:** Given one or more entries are Notified for a freed slot, when the claim window is in effect, then the slot does not appear on FEAT-05's general public list until the window lapses.

**FEAT-20.SPEC-004-AC-07:** Given two clients are both Notified for the same freed slot, when one completes a booking first, then that client's entry converts (FEAT-20.SPEC-006) and the other's entry remains Requested, unconverted and unexpired.

**FEAT-20.SPEC-004-AC-08:** Given a second notified client attempts to claim a slot after the first already booked it, then she sees the plain "just taken" message (XBR-01) and her entry remains Requested.

**FEAT-20.SPEC-004-AC-09:** Given Riley has a pending Notified claim, when she confirms Leave on FEAT-20.SPEC-002, then her entry is deleted immediately regardless of the pending notification.

**FEAT-20.SPEC-004-AC-10:** Given Talia (the Pro) views her Attention List, when she looks for which specific clients are notified for an opening, then she sees only the aggregate demand count, never individual notification state.

**FEAT-20.SPEC-004-AC-11:** Given Riley's claim window has already lapsed when she taps "Claim now," then FEAT-20.SPEC-007's expiry handling applies rather than a priority-window booking.

**FEAT-20.SPEC-004-AC-12:** Given Riley holds two separate Notified entries for two different freed slots at the same time, when she claims one, then the other's window and claimability are entirely unaffected.

**FEAT-20.SPEC-004-AC-13:** Given three Requested entries all match the same freed slot, when the matching runs, then all three are transitioned to Notified simultaneously with no queue or turn order among them.

**FEAT-20.SPEC-004-AC-14:** Given Support views a Pro's account via FEAT-19, when they inspect a specific client's notification/claim state, then they see it as View-only, with no action available.

**FEAT-20.SPEC-004-AC-15:** Given a Notified entry converts to a Booking, when Riley then looks for a Leave action on that entry in FEAT-20.SPEC-002, then none is offered, since the entry is already terminal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-11.SPEC-004
spec_name: No-Show Marking Window & Authorization Rules
spec_slug: no-show-marking-window-authorization-rules
parent_feature: FEAT-11
parent_feature_name: No-Show Marking & Deposit Forfeiture
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 25
acceptance_criteria_count: 17
---

# Logic/Rule Spec: No-Show Marking Window & Authorization Rules

## Overview

**Name:** No-Show Marking Window & Authorization Rules
**ID:** FEAT-11.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs who may mark or undo a no-show (the Pro, on their own bookings only), the eligible marking window (after the appointment start time, before the booking auto-completes), and the 24-hour undo grace window (platform parameter: `no-show-undo-grace-window-hours`), shared by the prompt and both automations rather than duplicated in each.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture
**Governed Entity:** Booking (the no-show marking and undo transition, and the Deposit Transaction status it depends on)

## Scope and Non-Goals

**In Scope:**
- The marking-window bounds (lower: start_time has passed; upper: before the booking's auto-completion boundary)
- The fixed 24-hour undo grace window, measured from the Deposit Transaction's Forfeited outcome timestamp
- Authorization for the "mark no-show" and "undo no-show" actions, by role
- Ownership: a Pro may only mark or undo bookings they own
- The cross-entity condition that a mark or undo is only valid against a Deposit Transaction in the matching status (Captured to mark, Forfeited to undo)

**Non-Goals:**
- Writing the Booking or Deposit Transaction transitions themselves -- owned by FEAT-11.SPEC-002 (mark) and FEAT-11.SPEC-003 (undo); this spec only defines whether the attempt is eligible
- The prompt's layout and interaction handling -- owned by FEAT-11.SPEC-001 (Screen); this spec is referenced by it, not embedded in it
- Governing any other Booking state transition (Pending Payment, Confirmed, Cancelled, Rescheduled, Expired (unpaid), or the auto-completion itself) -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: those transitions and their windows are owned by FEAT-05, FEAT-07, FEAT-10, FEAT-12, and FEAT-30
- Deciding the forfeiture outcome's substance (deposit kept vs. refunded) -- owned by the Cancellation Policy Engine (FEAT-09) and read as a binary, acknowledged fact by FEAT-11.SPEC-002; this spec governs only the window and authorization for the action, not the financial outcome it produces

## Governed Entity

**Entity:** Booking (narrow slice relevant to the no-show marking and undo transition), with the Deposit Transaction's status read as a cross-entity precondition.
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | text | Service booked -- fixed at booking, not evaluated by this spec |
| start_time | date/time | Appointment start time, in the Pro's timezone -- the lower bound of the marking window |
| duration | number | Appointment duration -- not evaluated by this spec |
| client | reference | The booked Client -- not evaluated by this spec beyond confirming a booking exists |
| price_agreed / deposit_amount | number | Fixed at booking -- not evaluated by this spec (the amount itself is FEAT-07's and FEAT-09's concern) |
| policy_version | reference | The acknowledged Cancellation Policy version -- read by FEAT-11.SPEC-002 for the forfeiture outcome, not evaluated as an eligibility condition here |
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- the marking window is only open while state is Awaiting Outcome, and the undo window only while state is No-Show |
| attendance_reply | text | Reminder reply -- not evaluated by this spec |
| balance_due | number (derived) | Not evaluated by this spec |
| source | enum | Booking origin -- not evaluated by this spec |
| cancellation / reschedule timestamps and optional private Pro reason | text/date | Not evaluated by this spec -- a cancelled or rescheduled booking cannot be in Awaiting Outcome or No-Show state, so these fields are already excluded by the state condition |
| owning Pro Account | reference | The Pro who created this booking -- the ownership condition for both actions |

**Cross-entity precondition (Deposit Transaction):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed -- must be Captured for a mark attempt to be eligible, and Forfeited for an undo attempt to be eligible |
| outcome_reason / timestamps | text/date | The Forfeited outcome timestamp anchors the 24-hour undo grace window |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-001 | No-Show Mark & Undo Prompt | On prompt open (determines which of the three prompt states to show) and again on confirm (re-validates before triggering the automation) |
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | Re-checks the marking window, ownership, and Deposit Transaction status immediately before the atomic write |
| FEAT-11.SPEC-003 | No-Show Mark Undo | Re-checks the undo grace window, ownership, and Deposit Transaction status immediately before the atomic write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Booking.start_time | Marking window lower bound: the current time must be at or after this value | Mark action only | On prompt open and again on confirm | "This booking can be marked no-show only after its appointment time has passed." | Yes |
| Booking.start_time (via auto-completion boundary) | Marking window upper bound: the current time must be before start_time + platform parameter: `booking-auto-completion-window-days` (boundary owned by FEAT-12) | Mark action only | On prompt open and again on confirm | "This booking has already auto-completed and can no longer be marked as a no-show." | Yes |
| Booking.state | Must be Awaiting Outcome | Mark action only | On prompt open and again on confirm | "This booking's state has changed and can no longer be marked as a no-show." | Yes |
| Booking.state | Must be No-Show | Undo action only | On prompt open and again on confirm | "This booking is not currently marked as a no-show." | Yes |
| Deposit Transaction.status | Must be Captured | Mark action only | On confirm (re-checked by FEAT-11.SPEC-002 immediately before write) | "This booking's deposit is not in a state that can be forfeited." | Yes |
| Deposit Transaction.status | Must be Forfeited | Undo action only | On confirm (re-checked by FEAT-11.SPEC-003 immediately before write) | "This booking's deposit is not in a state that can be restored." | Yes |
| Deposit Transaction.outcome_reason/timestamps | Undo grace window: the current time must be within the fixed span (platform parameter: `no-show-undo-grace-window-hours`) of the Forfeited outcome timestamp | Undo action only | On prompt open (whether Undo is offered at all) and again on confirm | "This no-show mark can no longer be undone." | Yes |
| owning Pro Account | Must match the requesting Pro's account | Both actions | On prompt open and again on confirm | "This booking could not be found." (the prompt does not reveal another Pro's booking exists at all -- per user-persona.md's "No role can ever see another pro's data") | Yes |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Marking window is fully bounded | Booking.start_time, auto-completion boundary (platform parameter: `booking-auto-completion-window-days`) | The marking window is open only in the span from start_time (inclusive) to the auto-completion boundary (exclusive); outside either edge, the mark action is unavailable | "This booking can be marked no-show only after its appointment time has passed." (below the window) / "This booking has already auto-completed and can no longer be marked as a no-show." (past the window) |
| Undo window never outlasts auto-completion | Deposit Transaction's Forfeited outcome_timestamp + platform parameter: `no-show-undo-grace-window-hours`, Booking.start_time + platform parameter: `booking-auto-completion-window-days` | The fixed 24-hour undo grace window is always shorter than the 7-day auto-completion boundary measured from the same appointment, so an undo attempt can never collide with the booking auto-completing out from under it; the two boundaries are evaluated independently and never need reconciling against each other | N/A -- this rule states a designed non-collision, not a validation failure |
| Action must match current cross-entity state pair | Booking.state, Deposit Transaction.status | Mark requires (Awaiting Outcome, Captured); undo requires (No-Show, Forfeited). Any other pairing observed at confirm time means the state has already moved (through cancellation, a goodwill refund, or a dispute) and the action is refused | "This booking's state has changed. {current state and deposit status shown}." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Mark booking as no-show | The Pro | Only bookings the requesting Pro owns, and only within the marking window (Booking.state is Awaiting Outcome, Deposit Transaction.status is Captured) | If the window or state condition fails: the exact message from Field Validation Rules above. If the Pro does not own the booking: "This booking could not be found." (the booking is never revealed to exist for another Pro's account) |
| Mark booking as no-show | The Client | Never | The prompt (FEAT-11.SPEC-001) is not reachable from any Client-facing surface -- there is no control to deny; the Client's Cancellation & No-Show Handling access is Own-only visibility of the resulting outcome, never this action |
| Mark booking as no-show | Platform Operator (Support) | Never | No mark control is shown anywhere in Support's read-only surfaces (FEAT-19); Support's View access to Cancellation & No-Show Handling never includes an action control |
| Undo a no-show mark | The Pro | Only bookings the requesting Pro owns, and only within the fixed grace window (Booking.state is No-Show, Deposit Transaction.status is Forfeited, current time within platform parameter: `no-show-undo-grace-window-hours` of the Forfeited outcome timestamp) | If the window or state condition fails: the exact message from Field Validation Rules above. If the Pro does not own the booking: "This booking could not be found." |
| Undo a no-show mark | The Client | Never | Same as above -- no reachable surface exists for the Client |
| Undo a no-show mark | Platform Operator (Support) | Never | Same as above -- no action control exists on any Support surface; SC-05 excludes Support from acting on the Pro's behalf |
| View the resulting deposit-kept or restored outcome (not this action itself) | The Pro | Always, for their own bookings | -- |
| View the resulting deposit-kept or restored outcome (not this action itself) | The Client | Only in their own booking history, own bookings only (FEAT-16) | -- |
| View the resulting deposit-kept or restored outcome (not this action itself) | Platform Operator (Support) | View-only, for dispute troubleshooting, through FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Marking window upper bound | Booking.start_time + platform parameter: `booking-auto-completion-window-days` | Computed on every eligibility check (prompt open and confirm) | No -- one value for every Pro (7 days per XBR-12), owned by FEAT-12 |
| Undo grace window boundary | Deposit Transaction's Forfeited outcome_timestamp + platform parameter: `no-show-undo-grace-window-hours` | Computed on every eligibility check (prompt open and confirm) | No -- one value for every Pro and every booking (24 hours per XBR-12) |
| Prompt state selection (Mark / Undo / Grace window elapsed) | Derived entirely from Booking.state, Deposit Transaction.status, and the two window boundaries above -- never a stored field or a manual toggle | On every prompt open | No |

## Business Rules

- XBR-12: a booking can be marked no-show or completed only after its start time; it auto-completes 7 days after the appointment (platform parameter: `booking-auto-completion-window-days`); a no-show mark can be undone for 24 hours (platform parameter: `no-show-undo-grace-window-hours`); a completed or no-show booking can no longer be cancelled or rescheduled.
- XBR-08: the forfeiture outcome this window and authorization govern is derived from the policy version the booking's client acknowledged at booking -- this spec does not decide the outcome itself, only whether the action that produces it is currently eligible.
- Ownership is the sole authorization gate for both actions -- there is no additional role tier among Pros; every Pro Account has identical authority over its own bookings, and none over any other Pro's.
- The Deposit Transaction status condition (Captured for mark, Forfeited for undo) exists because a deposit can be forfeited or refunded only once overall (dependency map's Contention rule) -- these rules prevent this feature's actions from firing against a deposit that has already moved to a different outcome through another path (a client cancellation refund via FEAT-09, or a goodwill refund via FEAT-30).
- The marking window's lower bound (start_time passed) and the undo window's anchor (the Forfeited outcome timestamp, not the original start_time) are deliberately distinct measurements -- the undo window is about how recently Talia acted, not about the appointment's timing.

## Edge Cases

- **Talia's confirm arrives at the exact instant start_time passes** -- The lower bound is inclusive (current time "at or after" start_time), so a confirm timestamped exactly at start_time is eligible.
- **Talia's confirm arrives at the exact instant the auto-completion boundary is reached** -- The upper bound is exclusive ("before" the boundary), so a confirm timestamped exactly at the boundary is denied with the auto-completed message.
- **The undo grace window has exactly 0 seconds remaining** -- Treated as elapsed; the boundary is exclusive in Talia's favor only up to and not including the exact expiry instant, so a confirm timestamped at or after the boundary is denied.
- **A goodwill refund is issued through FEAT-30 while the booking is still in Awaiting Outcome (before any no-show mark)** -- The Deposit Transaction's status moves away from Captured, so a subsequent mark attempt fails the "must be Captured" condition and is denied with the deposit-status message, even though the marking window itself may still be open.
- **The Pro's account ownership of a booking changes** -- Cannot occur in this product: a Booking belongs to exactly one Pro Account for its entire lifecycle (dependency map's Booking Relationships), so no ownership-transfer edge case exists for this entity.
- **Two eligibility checks (prompt open and confirm) disagree because time passed between them** -- The confirm-time check is authoritative; if the window closed between open and confirm, the confirm is denied even though the prompt initially showed the action as available (per FEAT-11.SPEC-001's re-validation-on-confirm behavior).
- **A booking's state and its Deposit Transaction's status momentarily disagree with the expected pairing (e.g., mid-write from a concurrent action)** -- The cross-field rule "Action must match current cross-entity state pair" catches this at confirm time and denies with the current-state message rather than allowing an action against an inconsistent pairing.

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given Talia views a booking whose start_time has just passed, when she opens this booking's prompt, then the mark action is eligible and no denial message appears.

**FEAT-11.SPEC-004-AC-02:** Given Talia views a booking whose start_time has not yet arrived, when she attempts to mark it no-show (a state unreachable through the normal dashboard path), then the eligibility check denies with "This booking can be marked no-show only after its appointment time has passed."

**FEAT-11.SPEC-004-AC-03:** Given a booking's auto-completion boundary (platform parameter: `booking-auto-completion-window-days` after start_time) has passed, when Talia attempts to mark it no-show, then the eligibility check denies with "This booking has already auto-completed and can no longer be marked as a no-show."

**FEAT-11.SPEC-004-AC-04:** Given a booking is in Awaiting Outcome state with its Deposit Transaction Captured, when Talia's mark confirm reaches the eligibility check, then both conditions pass and the mark proceeds to FEAT-11.SPEC-002.

**FEAT-11.SPEC-004-AC-05:** Given a booking's Deposit Transaction is no longer Captured (e.g., already Forfeited from an earlier mark), when Talia attempts to mark it no-show again, then the eligibility check denies with "This booking's deposit is not in a state that can be forfeited."

**FEAT-11.SPEC-004-AC-06:** Given Talia marked a booking as a no-show 3 hours ago, when she opens the prompt, then the undo action is eligible because the current time is within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) of the Forfeited outcome timestamp.

**FEAT-11.SPEC-004-AC-07:** Given Talia marked a booking as a no-show more than 24 hours ago, when she opens the prompt, then the undo action is not offered and the eligibility check denies with "This no-show mark can no longer be undone."

**FEAT-11.SPEC-004-AC-08:** Given a booking's Deposit Transaction is no longer Forfeited (e.g., a goodwill refund has since been issued through FEAT-30), when Talia attempts to undo the no-show mark, then the eligibility check denies with "This booking's deposit is not in a state that can be restored."

**FEAT-11.SPEC-004-AC-09:** Given Talia (the Pro) attempts to mark or undo a no-show on a booking she owns, when the ownership check runs, then it passes and the action-specific window checks proceed.

**FEAT-11.SPEC-004-AC-10:** Given Talia attempts to reach the mark/undo prompt for a booking that belongs to a different Pro Account (e.g., a stale or manipulated link), when the ownership check runs, then it denies with "This booking could not be found." and never reveals that the booking exists.

**FEAT-11.SPEC-004-AC-11:** Given Riley (the Client) has no path to this prompt, when the product's screens are reviewed for a mark or undo control reachable to her, then none exists -- her Cancellation & No-Show Handling access remains Own-only visibility of the resulting outcome.

**FEAT-11.SPEC-004-AC-12:** Given Platform Operator (Support) is viewing this booking through FEAT-19's read-only surfaces, when Support looks for a mark or undo control, then none is shown -- Support's access to Cancellation & No-Show Handling is View-only.

**FEAT-11.SPEC-004-AC-13:** Given a booking's start_time has passed but is not yet at the auto-completion boundary, when the marking window is evaluated, then it is found open (both the lower and upper bound conditions are satisfied).

**FEAT-11.SPEC-004-AC-14:** Given a confirm request for a mark action arrives at the exact instant the auto-completion boundary is reached, when the eligibility check evaluates the upper bound, then it is treated as past the window (the boundary is exclusive) and the action is denied.

**FEAT-11.SPEC-004-AC-15:** Given a confirm request for an undo action arrives at the exact instant the 24-hour grace boundary is reached, when the eligibility check evaluates the window, then it is treated as elapsed and the action is denied.

**FEAT-11.SPEC-004-AC-16:** Given the Booking's state and the Deposit Transaction's status do not match the expected pairing for the requested action at confirm time (e.g., the booking is No-Show but the deposit is already Refunded through another path), when the cross-field consistency rule evaluates the pairing, then the action is denied with the current-state message rather than proceeding against an inconsistent pairing.

**FEAT-11.SPEC-004-AC-17:** Given the marking window and the undo window are both computed against the same appointment, when their boundaries are compared, then the fixed 24-hour undo window always falls before the 7-day auto-completion boundary, so the two never need to be reconciled against each other for the same booking.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

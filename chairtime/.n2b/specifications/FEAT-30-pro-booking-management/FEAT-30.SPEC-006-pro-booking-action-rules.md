---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-30.SPEC-006
spec_name: Pro Booking Action Rules
spec_slug: pro-booking-action-rules
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 16
acceptance_criteria_count: 19
---

# Logic/Rule Spec: Pro Booking Action Rules

## Overview

**Name:** Pro Booking Action Rules
**ID:** FEAT-30.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs the eligibility, ownership, and limits shared by every Pro-initiated booking action -- cancel, reschedule, goodwill refund, book-client-in, and bulk cancel -- so each screen and automation in this feature references one authoritative set of rules instead of restating them.
**Parent Feature:** FEAT-30 -- Pro Booking Management
**Governed Entity:** Booking (the Pro-action eligibility slice: `state` and `start_time`), with Deposit Transaction.status and Availability Rule's notice/horizon fields read as cross-entity preconditions

## Scope and Non-Goals

**In Scope:**
- The Pro-only exception to minimum booking notice and booking horizon for reschedule (FEAT-30.SPEC-002) and book-client-in (FEAT-30.SPEC-004)
- The Pro-created deposit-request hold window (24 hours or 2 hours before the appointment, whichever comes first)
- The once-only refund limit shared by every refund path this feature triggers or executes
- The until-completion availability window for a goodwill refund
- The completed/auto-completed cutoff that blocks cancel, reschedule, and new bulk-cancel actions
- Ownership and authorization for every action this feature defines, by role

**Non-Goals:**
- Determining the deposit outcome's substance (refund vs. keep) for a single Pro cancellation or reschedule -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec governs only whether the triggering action is currently eligible, not the financial outcome it produces
- Creating or expiring the Pro-created deposit-request hold itself -- owned by FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration); this spec states the shared window value that spec computes against, not the hold mechanics
- Governing the Booking's transition into `Completed` -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this spec only reads that boundary as the cutoff for its own actions
- The no-show marking window and its own 24-hour undo grace period -- owned by FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules); this spec governs a different action (goodwill refund) that happens to interact with the same Deposit Transaction

## Governed Entity

**Entity:** Booking (Pro-action eligibility slice), with Deposit Transaction and Availability Rule read as cross-entity preconditions
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- cancel/reschedule eligibility depends on this value |
| start_time | date/time | Appointment start time, Pro's timezone -- the anchor for the deposit-request hold's cutoff and the auto-completion boundary read from FEAT-12.SPEC-006 |
| owning Pro Account | reference | The Pro who owns this booking -- the ownership condition for every action in this spec |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps and optional private Pro reason | various | Not evaluated as eligibility conditions by this spec; read and written by the automations this spec governs (FEAT-30.SPEC-007/008/009/010) |

**Cross-entity precondition (Deposit Transaction):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Authorized \| Captured \| Applied \| Refunded \| Refund in Progress \| Forfeited \| Disputed -- must be Captured or Forfeited for a goodwill refund to be eligible; a deposit already Refunded, Refund in Progress, or Disputed can never be refunded again (once-only limit) |

**Cross-entity reference (Availability Rule, read-only):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| minimum_booking_notice | number (days) | Exempted for the Pro when rescheduling (FEAT-30.SPEC-002) or booking a client in (FEAT-30.SPEC-004) |
| booking_horizon | number (weeks/months) | Exempted for the Pro when rescheduling or booking a client in |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-30.SPEC-001 | Cancel Booking (Pro-Initiated) | On screen entry (eligibility gate) and again on confirm |
| FEAT-30.SPEC-002 | Reschedule Booking (Pro-Initiated) | On screen entry, on candidate-slot selection (notice/horizon exception), and again on confirm |
| FEAT-30.SPEC-003 | Goodwill Deposit Refund | On screen entry (until-completion, once-only limits) and again on confirm |
| FEAT-30.SPEC-004 | Book Client In | On candidate-slot selection (notice/horizon exception) |
| FEAT-30.SPEC-005 | Cancel Several Bookings at Once | On screen entry, per booking in the reviewed set, and again on confirm |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | Re-checks eligibility and ownership immediately before the atomic write |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | Re-checks eligibility and ownership per booking immediately before each atomic write |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | Re-checks the once-only and until-completion limits immediately before the atomic write |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | Reads the deposit-request hold window this spec states, and applies the notice/horizon exception to the candidate slot |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Consumes this spec's stated hold-window values and notice/horizon exception when creating and expiring the hold |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Booking.state (cancel/reschedule/bulk-cancel actions) | Must be Confirmed or Awaiting Outcome | Cancel, reschedule, and bulk-cancel actions only | On screen entry and again on confirm | "This booking is already completed and can no longer be changed." (state is Completed or No-Show) / "This booking has already been cancelled or rescheduled." (state is Cancelled by Client, Cancelled by Pro, Rescheduled, or Expired) | Yes |
| Booking.state (goodwill refund action) | Must not be Completed | Goodwill refund only | On screen entry and again on confirm | "A goodwill refund is no longer available once a booking is completed." | Yes |
| Deposit Transaction.status (goodwill refund action) | Must be Captured or Forfeited | Goodwill refund only | On screen entry and again on confirm | "This booking's deposit is not in a state that can be refunded." (already Refunded, Refund in Progress, or Disputed) | Yes |
| owning Pro Account | Must match the requesting Pro's account | All actions | On screen entry and again on confirm | "This booking could not be found." (never reveals another Pro's booking exists, per user-persona.md's "No role can ever see another pro's data") | Yes |
| Candidate slot's minimum_booking_notice (Availability Rule) | Exempted for this feature's Pro-initiated actions | Reschedule (FEAT-30.SPEC-002) and Book Client In (FEAT-30.SPEC-004) only | On candidate-slot re-validation | N/A -- exemption, not a validation failure | No |
| Candidate slot's booking_horizon (Availability Rule) | Exempted for this feature's Pro-initiated actions | Reschedule and Book Client In only | On candidate-slot re-validation | N/A -- exemption, not a validation failure | No |
| Candidate slot's duration + buffer fit | Never exempted -- must pass FEAT-03.SPEC-004's fit rule regardless of who is booking | Reschedule and Book Client In only | On candidate-slot re-validation | "That time doesn't fit -- pick another." (per FEAT-03.SPEC-004's own wording) | Yes |
| service, duration, client, price_agreed/deposit_amount, policy_version, attendance_reply, balance_due, source, cancellation/reschedule timestamps | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Once-only refund limit | Deposit Transaction.status | Any refund this feature triggers or executes (cancel-with-refund, reschedule-carry-over, goodwill, bulk cancel) is refused once the Deposit Transaction has already reached Refunded, Refund in Progress, or Disputed for that booking -- a deposit is refunded at most once (dependency map's Deposit Transaction Contention rule) | "This booking's deposit has already been resolved and cannot be refunded again." |
| Goodwill availability spans both pre- and post-no-show states | Booking.state, Deposit Transaction.status | A goodwill refund is available whenever Deposit Transaction.status is Captured (offered instead of marking a no-show, or against an as-yet-unresolved inside-window client cancellation) or Forfeited (converting an already-kept deposit -- from a no-show mark or an inside-window client cancellation -- into a full refund), for as long as Booking.state has not reached Completed | "A goodwill refund is no longer available once a booking is completed." |
| Deposit-request hold window is the earlier of two limits | Booking.start_time, hold-created timestamp | The Pro-created deposit-request hold's expiry (computed and enforced by FEAT-03.SPEC-007) is the earlier of platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment | N/A -- this spec states the value; FEAT-03.SPEC-007 enforces it and owns its own messaging |
| Notice/horizon exception never exempts the fit rule | Candidate slot's duration + buffer, Availability Rule | The Pro-only exception to minimum_booking_notice and booking_horizon never extends to whether the candidate slot's full duration plus buffer genuinely fits an open window with no conflict (XBR-03) | "That time doesn't fit -- pick another." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Cancel a booking (single) | The Pro | Only bookings the requesting Pro owns, and only while Booking.state is Confirmed or Awaiting Outcome | If ownership fails: "This booking could not be found." If the state condition fails: the exact message from Field Validation Rules above |
| Reschedule a booking | The Pro | Only bookings the requesting Pro owns, only while Booking.state is Confirmed or Awaiting Outcome, and only to a candidate slot that passes FEAT-03.SPEC-004's fit rule (notice/horizon exempted) | Same as Cancel above; a candidate slot that fails the fit rule shows "That time doesn't fit -- pick another." |
| Issue a goodwill refund | The Pro | Only bookings the requesting Pro owns, only while Booking.state has not reached Completed, and only while Deposit Transaction.status is Captured or Forfeited | If ownership fails: "This booking could not be found." If the state or deposit-status condition fails: the exact messages from Field Validation Rules above |
| Book a client in (create a new Booking) | The Pro | Always, for the Pro's own account, to a candidate slot that passes FEAT-03.SPEC-004's fit rule (notice/horizon exempted) | A candidate slot that fails the fit rule shows "That time doesn't fit -- pick another." |
| Cancel several bookings at once | The Pro | Only bookings the requesting Pro owns, evaluated per booking against the same state condition as a single cancellation | Per booking: the exact messages from Field Validation Rules above; a booking that fails is reported as a failed outcome in the reviewed set (FEAT-30.SPEC-005/008) while the others still proceed |
| Any action in this spec | The Client | Never | No control for any of these actions is reachable from any Client-facing surface; the Client's Booking & Payment access is Own-only, exercised entirely through FEAT-10 (Client-Initiated Cancel/Reschedule), never through this feature |
| Any action in this spec | Platform Operator (Support) | Never | No action control exists anywhere on Support's read-only surfaces (FEAT-19); Support's access to Booking & Payment and Cancellation & No-Show Handling is View-only, per SC-05 |
| View the outcome of any action in this spec (not the action itself) | The Client | Own-only, for their own affected booking | -- |
| View the outcome of any action in this spec (not the action itself) | Platform Operator (Support) | View-only, through FEAT-19 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deposit-request hold expiry | The earlier of (creation time + platform parameter: `deposit-request-hold-max-hours`) and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`) | Computed by FEAT-03.SPEC-007 whenever FEAT-30.SPEC-010 creates a Pro-created deposit request | No -- one value for every Pro and every booking |
| Notice/horizon exception applicability | Derived from the requesting role: applied automatically whenever the triggering action is FEAT-30.SPEC-002 (reschedule) or FEAT-30.SPEC-004 (book-client-in); never applied to any client-facing booking or reschedule path (FEAT-05, FEAT-10) | On every candidate-slot re-validation for this feature's actions | No |
| Goodwill and once-only refund eligibility | Derived entirely from Booking.state and Deposit Transaction.status at the moment of the action -- never a stored flag or manual toggle | On screen entry and again on confirm, for every refund-bearing action | No |

## Business Rules

- XBR-03: minimum booking notice and booking horizon limit every client-facing booking path; the Pro alone may book inside notice or beyond horizon when booking a client in (FEAT-30.SPEC-004) or rescheduling (FEAT-30.SPEC-002).
- XBR-02: the Pro-created deposit-request hold holds its slot for up to platform parameter: `deposit-request-hold-max-hours` or until platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, whichever comes first; this spec states the value, FEAT-03.SPEC-007 owns the hold mechanics that enforce it.
- XBR-10: refunds are always full and happen at most once per deposit; this spec's once-only limit is the eligibility gate every refund-bearing action in this feature (cancel, reschedule-inside-window-never-applies-per-XBR-09, goodwill, bulk cancel) checks before proceeding.
- XBR-12: a completed or auto-completed booking can no longer be cancelled or rescheduled; a goodwill refund remains available only until completion. This spec is the shared reference every screen and automation in this feature reads instead of restating the rule.
- Ownership is the sole authorization gate among Pros -- there is no tier among Pro accounts; every Pro has identical authority over their own bookings and none over any other Pro's, consistent with FEAT-11.SPEC-004's identical treatment of the same ownership condition.
- A goodwill refund never changes Booking.state -- only Deposit Transaction.status; this distinguishes it from every other action this spec governs, all of which do transition Booking.state (per the Entity-Lifecycle Coverage Matrix).

## Edge Cases

- **Talia's confirm on a cancel or reschedule arrives the instant the Auto-Completion Sweep (FEAT-12.SPEC-004) completes the same booking** -- The eligibility re-check at confirm time (reject-with-refresh, per the Booking entity's Contention rule) finds the booking already Completed and denies with "This booking is already completed and can no longer be changed."; Talia's screen reflects the current state on refresh.
- **Talia attempts a goodwill refund on a booking whose deposit a client-side outside-window cancellation has already refunded automatically (FEAT-09)** -- The Deposit Transaction.status is already Refunded, so the once-only limit denies with "This booking's deposit has already been resolved and cannot be refunded again."
- **Talia reschedules a client to a time inside her own minimum_booking_notice** -- The candidate slot is not excluded on notice grounds (Pro-only exception applies); the fit rule (duration + buffer, no conflict) still applies unexempted.
- **Two eligibility checks for the same booking (screen entry and confirm) disagree because time passed between them** -- The confirm-time check is authoritative; a window or state condition that closed between entry and confirm is denied at confirm even though the screen initially showed the action as available.
- **Talia issues a goodwill refund on a booking already marked No-Show (Deposit Transaction Forfeited)** -- The refund is eligible per the Cross-Field Rule above (Forfeited is a refundable status), converting the kept deposit to Refunded; per FEAT-11.SPEC-004's own edge case, this also closes the no-show mark's undo window (the Deposit Transaction is no longer Forfeited).
- **A bulk-cancel review set includes one booking that has since been completed by the Auto-Completion Sweep** -- That single booking is denied with the standard completed-state message and reported as a failed outcome in the reviewed set (FEAT-30.SPEC-005/008); the other bookings in the set proceed independently.

## Acceptance Criteria

**FEAT-30.SPEC-006-AC-01:** Given Talia opens the Cancel Booking screen for a Confirmed booking she owns, when eligibility is checked, then the cancel action is available with no denial message.

**FEAT-30.SPEC-006-AC-02:** Given Talia opens the Cancel Booking screen for a booking already Completed, when eligibility is checked, then she sees "This booking is already completed and can no longer be changed." and no cancel action is offered.

**FEAT-30.SPEC-006-AC-03:** Given Talia opens the Reschedule screen for a booking already Cancelled by Client, when eligibility is checked, then she sees "This booking has already been cancelled or rescheduled." and no reschedule action is offered.

**FEAT-30.SPEC-006-AC-04:** Given Talia picks a candidate reschedule time that falls inside her own minimum_booking_notice, when the candidate is re-validated, then it is not excluded on notice grounds, per the Pro-only exception.

**FEAT-30.SPEC-006-AC-05:** Given Talia picks a candidate time for booking a client in beyond her own booking_horizon, when the candidate is re-validated, then it is not excluded on horizon grounds, per the Pro-only exception.

**FEAT-30.SPEC-006-AC-06:** Given Talia picks a candidate reschedule time whose duration and buffer do not fit any open window, when the candidate is re-validated, then she sees "That time doesn't fit -- pick another." regardless of the Pro-only exception.

**FEAT-30.SPEC-006-AC-07:** Given a booking's Deposit Transaction is Captured and Booking.state is Awaiting Outcome, when Talia opens the Goodwill Deposit Refund screen, then the goodwill action is eligible.

**FEAT-30.SPEC-006-AC-08:** Given a booking's Deposit Transaction is Forfeited following a no-show mark, when Talia opens the Goodwill Deposit Refund screen, then the goodwill action is still eligible, converting the kept deposit to a full refund.

**FEAT-30.SPEC-006-AC-09:** Given a booking's Deposit Transaction is already Refunded, when Talia opens the Goodwill Deposit Refund screen, then she sees "This booking's deposit is not in a state that can be refunded." and no goodwill action is offered.

**FEAT-30.SPEC-006-AC-10:** Given a booking has reached Completed, when Talia looks for a goodwill refund option, then none is offered, and a direct attempt is denied with "A goodwill refund is no longer available once a booking is completed."

**FEAT-30.SPEC-006-AC-11:** Given Talia attempts to reach any of this feature's screens for a booking belonging to a different Pro account, when the ownership check runs, then it denies with "This booking could not be found." and never reveals the booking exists.

**FEAT-30.SPEC-006-AC-12:** Given Riley (the Client) has no path to any of this feature's screens, when the product's screens are reviewed for a reachable control, then none exists -- her Booking & Payment access remains Own-only, exercised through FEAT-10.

**FEAT-30.SPEC-006-AC-13:** Given Platform Operator (Support) is viewing a booking through FEAT-19's read-only surfaces, when Support looks for any of this feature's action controls, then none is shown.

**FEAT-30.SPEC-006-AC-14:** Given Talia creates a Pro-created deposit request for an appointment more than 24 hours away, when the hold's expiry is computed by FEAT-03.SPEC-007, then it is set to platform parameter: `deposit-request-hold-max-hours` from creation, per this spec's stated window.

**FEAT-30.SPEC-006-AC-15:** Given Talia creates a Pro-created deposit request for an appointment less than platform parameter: `deposit-request-hold-max-hours` away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment.

**FEAT-30.SPEC-006-AC-16:** Given Talia's confirm on a cancellation arrives at the exact instant the Auto-Completion Sweep completes the same booking, when the confirm-time eligibility check runs, then it denies with the completed-state message rather than proceeding against a stale screen state.

**FEAT-30.SPEC-006-AC-17:** Given a booking's deposit has already been refunded through an automatic outside-window client cancellation (FEAT-09), when Talia attempts a goodwill refund on the same booking, then she sees "This booking's deposit has already been resolved and cannot be refunded again."

**FEAT-30.SPEC-006-AC-18:** Given Talia reviews a bulk-cancel set where one booking has since been completed, when eligibility is checked per booking, then that booking is denied and reported as a failed outcome while the remaining bookings in the set proceed.

**FEAT-30.SPEC-006-AC-19:** Given Talia owns the booking she is acting on, when the ownership check runs for any action in this spec, then it passes and the action-specific eligibility checks proceed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-22.SPEC-002
spec_name: Balance Capture & Booking Status Update
spec_slug: balance-capture-booking-status-update
parent_feature: FEAT-22
parent_feature_name: In-App Balance Payment
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Balance Capture & Booking Status Update

## Overview

**Name:** Balance Capture & Booking Status Update
**ID:** FEAT-22.SPEC-002
**Type:** Automation
**Purpose:** On a successful in-app balance card charge, the system creates the Balance Payment record and updates the Booking's balance-due status to fully paid, so Talia's dashboard reflects it without a manual refresh.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Creating the Balance Payment record the instant a balance charge is reported captured
- Recomputing and reflecting the Booking's derived balance_due status as fully paid in the same step as that creation
- The exactly-zero-or-one-Balance-Payment-per-Booking guarantee at the point of capture (working with FEAT-22.SPEC-003's precondition check and FEAT-22.SPEC-004's idempotency guarantee)
- Feeding the fully-paid booking into the Pro's schedule and booking-management surfaces, and into the success state Riley sees

**Non-Goals:**
- Requesting authorization and capture from the payment-processing capability, or routing the captured amount to Talia's payout account -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this automation only reacts to that spec's reported outcome
- Computing the balance amount -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this automation only persists the amount already locked by that spec
- Guaranteeing the charge is never duplicated across a dropped connection, an interrupted status update, or a race with a Pro cancellation -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention); this automation implements the create-and-update step that spec's guarantee wraps around
- Composing or sending a confirmation message -- excluded per the Brief's Communications field: the balance payment confirmation is a same-screen success state on FEAT-22.SPEC-001, not a delivered message this automation triggers, per the inline-communication exception this feature's `notification_count: 0` reflects

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Balance charge reported captured | FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Fires when the payment-processing capability reports a successful capture for an attempt that passed FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check | Booking reference, captured amount, currency, capture timestamp |

This automation has exactly one trigger. It is fired once per successful capture event; a capture reported for a Booking that already has a Succeeded Balance Payment (per the idempotency guarantee in FEAT-22.SPEC-004) does not reach this automation as a new run -- see Edge Cases.

## Processing Logic

1. Receive the capture event from FEAT-22.SPEC-005: the Booking reference, captured amount, currency, and capture timestamp.
2. Confirm the referenced Booking is still in a payable state and that no Balance Payment with status Succeeded already exists for it (the one-succeeded-payment-per-booking guarantee, enforced jointly with FEAT-22.SPEC-004). If a Succeeded Balance Payment already exists for this Booking, treat this as a duplicate delivery of the same capture event and take no further action (see Edge Cases).
3. Create the Balance Payment record: amount from the capture event, state set directly to Succeeded, and the capture timestamp recorded. tip is left unset (owned by FEAT-23, Later, if it ever ships).
4. In the same atomic step as creating the Balance Payment, recompute the Booking's derived balance_due (price_agreed minus deposit_amount minus this Balance Payment's amount) to zero, and reflect the Booking's balance-due status as fully paid.
5. Signal FEAT-22.SPEC-001 (Balance Payment) that the Booking's balance-due status is now fully paid, so the screen can show its Success state.
6. Signal FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management) that the Booking now shows "fully paid" instead of "balance due," so Talia's schedule reflects it without a manual refresh.
7. Signal FEAT-28 (Payout Account Connection & Payout Visibility) that the captured balance is available for its money list, once FEAT-22.SPEC-005's payout routing confirms.
8. Signal FEAT-16 (Booking & Payment Activity Record) that a balance_payment_succeeded event occurred, for the append-only activity record.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Capture confirmed | The referenced Booking is still payable and has no existing Succeeded Balance Payment | Balance Payment created (state: Succeeded); Booking's derived balance_due recomputed to zero and its balance-due status reflected as fully paid | Riley sees the Success state on FEAT-22.SPEC-001; Talia sees "fully paid" on her schedule and booking-management surfaces | FEAT-22.SPEC-001, FEAT-12, FEAT-16, FEAT-28, FEAT-30 |
| Duplicate capture event ignored | A Succeeded Balance Payment already exists for the referenced Booking (the capture event was delivered more than once) | None -- the existing Balance Payment and fully-paid status are left unchanged | None -- Riley already saw the Success state from the first delivery | None beyond the existing state |
| Referenced Booking not eligible | The referenced Booking is no longer in a payable state at the moment this automation runs (for example, a Pro cancellation committed first, per FEAT-22.SPEC-004's contention rule) | None -- no Balance Payment is created against an ineligible Booking | The capture is reported back to FEAT-22.SPEC-005 as unappliable, which FEAT-22.SPEC-004 resolves per its correctness guarantee (never silently keep money for a cancelled booking) -- the amount is refunded rather than recorded as a kept balance payment | FEAT-22.SPEC-004, FEAT-22.SPEC-005 |
| Automation failure (processing error after capture confirmed) | The capture event is received but this automation cannot complete the create-and-update step (for example, an internal fault interrupts step 3 or 4) | No partial state is left visible: either both the Balance Payment and the fully-paid status are committed together, or neither is | Riley's screen (FEAT-22.SPEC-001) shows the Offline/Degraded resolution defined by FEAT-22.SPEC-004 -- she is never shown an ambiguous or double-charged state; the automation retries the create-and-update step automatically | FEAT-22.SPEC-001, FEAT-22.SPEC-004 |

## Data Model

**Reads:** Booking -- balance-payable state, price_agreed. Deposit Transaction -- amount (to compute the derived balance_due alongside this automation's own captured amount). Balance Payment -- read internally to check for an existing Succeeded record before creating a new one (the one-succeeded-payment-per-booking guarantee).
**Creates:** Balance Payment -- amount, state (set to Succeeded), capture timestamp. Exactly zero or one Succeeded Balance Payment per Booking, ever, from this automation.
**Updates:** Booking -- the derived balance_due field (recomputed to zero) and the balance-due status shown to the Pro. This is the only Booking-facing update this automation writes.
**Deletes:** None.

## Business Rules

- The Balance Payment creation and the Booking's fully-paid status update happen as a single atomic step -- one can never persist without the other.
- At most one Succeeded Balance Payment is ever created per Booking; a second capture event for the same Booking is a duplicate delivery, never a second charge (FEAT-22.SPEC-004).
- The amount written to the Balance Payment is exactly the value FEAT-22.SPEC-005 reports as captured, which must equal the amount FEAT-22.SPEC-003 locked -- this automation does not recompute or adjust the amount.
- This automation is the sole writer of the Booking's fully-paid balance-due status arising from an in-app balance payment; the Booking's other fields and state transitions (cancel, reschedule, no-show, complete) are owned by FEAT-05, FEAT-10, FEAT-11, FEAT-12, FEAT-21, and FEAT-30, per the Entity-Lifecycle Coverage Matrix.
- The fully-paid status is instant from Riley's perspective: the Booking reflects it the moment capture is reported, not on a delay or batch cycle, consistent with the deposit feature's own instant-confirmation practice (FEAT-07.SPEC-002).
- XBR-23: a balance payment this automation creates is never subject to forfeiture and is refunded in full if either party later cancels -- this automation itself performs no refund; it only records the successful capture.

## Edge Cases

- **Duplicate capture event for the same Booking** -- The second (and any subsequent) delivery finds an existing Succeeded Balance Payment and takes no action; the Booking remains fully paid with its original capture timestamp. No duplicate feedback fires.
- **Capture event arrives for a Booking a Pro cancellation has since resolved against (per FEAT-22.SPEC-004's contention rule)** -- No Balance Payment is created; the outcome is escalated to FEAT-22.SPEC-004 as a payment succeeded against an ineligible booking, which that spec's correctness guarantee resolves by refunding the captured amount rather than silently keeping it or leaving it unrecorded.
- **Status-update-step processing fails after the charge succeeded** -- The create-and-update step either fully commits or does not commit at all; a partial state (Balance Payment created but the Booking still showing balance due, or the reverse) never exists. If the step has not yet committed, it is retried automatically; Riley's screen shows the safe-retry behavior FEAT-22.SPEC-004 defines rather than a false failure.
- **Concurrent trigger firing (two capture events for two different Bookings at effectively the same time)** -- Each runs independently against its own Booking and Balance Payment; there is no shared state between two different Bookings' captures, so neither run affects the other.
- **Trigger fires while a previous run for the same Booking is still in flight** -- Cannot occur under normal operation, because FEAT-22.SPEC-004's one-succeeded-payment-per-booking guarantee ensures a second charge attempt for the same payable Booking is never authorized while the first is being captured; if a second capture event nonetheless arrives before the first run has finished committing, it is treated exactly as the duplicate-capture-event case once the first run's Balance Payment becomes visible.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Triggered by (inbound) | A reported successful capture fires this automation |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | References (inbound) | Confirms the captured amount matches the locked computation |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Governs the atomicity guarantee this automation implements and the resolution when a capture cannot be applied |
| FEAT-22.SPEC-001 (Balance Payment) | Affects (outbound) | The fully-paid status update is what moves that screen into its Success state |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on Talia's schedule |
| FEAT-30 (Pro Booking Management) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on the Pro's booking management surface |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The captured balance is what that feature's money list reflects, once payout routing confirms |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | balance_payment_succeeded is written to the append-only activity record |

## Analytics and Success Signals

- **balance_capture_confirmed** (booking reference, balance amount) -- supports success-metrics.md: "Payout Transparency"
- **balance_capture_duplicate_ignored** (booking reference) -- N/A -- no Stage 2 metric measures duplicate-delivery frequency directly; retained so the one-succeeded-payment-per-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric
- **balance_capture_not_applied** (booking reference, reason: booking_no_longer_payable) -- N/A -- no Stage 2 metric measures this exact non-applied path; retained so a captured-but-not-recorded amount (always resolved by a refund per FEAT-22.SPEC-004) is never silently unobservable

## Acceptance Criteria

**FEAT-22.SPEC-002-AC-01:** Given Riley's card is authorized and captured for a Booking that is still payable, when FEAT-22.SPEC-005 reports the capture, then a Balance Payment is created with state Succeeded and the Booking's balance_due is recomputed to zero in the same step.

**FEAT-22.SPEC-002-AC-02:** Given a Succeeded Balance Payment was just created for a Booking, when the same capture event is delivered a second time, then no second Balance Payment is created and the Booking's fully-paid status is unchanged.

**FEAT-22.SPEC-002-AC-03:** Given a capture event arrives for a Booking a Pro cancellation has already resolved against, when the automation checks the Booking's state, then no Balance Payment is created and the outcome is escalated to FEAT-22.SPEC-004.

**FEAT-22.SPEC-002-AC-04:** Given the Balance Payment is created and the Booking's balance-due status is updated, when the transition completes, then FEAT-22.SPEC-001 shows its Success state.

**FEAT-22.SPEC-002-AC-05:** Given Talia is viewing her schedule (FEAT-12) at the moment a client's balance is captured, when the automation confirms the Booking, then the booking's status updates to "fully paid" without Talia needing to refresh.

**FEAT-22.SPEC-002-AC-06:** Given a Booking's balance has just been captured by this automation, when the update completes, then a balance_payment_succeeded event is written to the append-only activity record (FEAT-16).

**FEAT-22.SPEC-002-AC-07:** Given the create-and-update step is interrupted by a processing error after the charge succeeded, when the automation retries, then the Balance Payment and the fully-paid status either both persist or neither does -- Riley is never shown a state where one exists without the other.

**FEAT-22.SPEC-002-AC-08:** Given two different clients' balance captures are reported at effectively the same time, when both automations run, then each creates its own Balance Payment and updates its own Booking independently, with no interference between the two runs.

**FEAT-22.SPEC-002-AC-09:** Given a captured amount reported by FEAT-22.SPEC-005 for a Booking, when this automation writes the Balance Payment, then the recorded amount exactly matches the balance amount FEAT-22.SPEC-003 locked on that Booking.

**FEAT-22.SPEC-002-AC-10:** Given a Succeeded Balance Payment has been created for a Booking, when any later capture event for that same Booking is evaluated, then it is recognized as a duplicate and produces no new Balance Payment.

**FEAT-22.SPEC-002-AC-11:** Given a Booking's balance is captured by this automation, when FEAT-30 (Pro Booking Management) next loads that booking, then its status reflects fully paid.

**FEAT-22.SPEC-002-AC-12:** Given the captured amount is available for Talia's money list, when FEAT-22.SPEC-005's payout routing confirms, then FEAT-28 reflects the entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (confirmed, duplicate ignored, not applied, automation failure) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-07.SPEC-002
spec_name: Deposit Capture & Booking Confirmation
spec_slug: deposit-capture-booking-confirmation
parent_feature: FEAT-07
parent_feature_name: Deposit Payment at Booking
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Deposit Capture & Booking Confirmation

## Overview

**Name:** Deposit Capture & Booking Confirmation
**ID:** FEAT-07.SPEC-002
**Type:** Automation
**Purpose:** On a successful card charge, the system creates the Deposit Transaction record and atomically flips the Booking from Pending Payment to Confirmed, enforcing exactly one charge per booking.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Creating the Deposit Transaction record the instant a charge is reported captured
- Atomically transitioning the Booking from Pending Payment to Confirmed as one step with that creation
- The one-charge-per-booking guarantee at the point of capture (working with FEAT-07.SPEC-003's precondition check and FEAT-07.SPEC-004's idempotency guarantee)
- Feeding the confirmed booking into the confirmation message and the Pro's schedule and activity surfaces

**Non-Goals:**
- Requesting authorization and capture from the payment-processing capability -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this automation only reacts to that spec's reported outcome
- Computing the deposit amount -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this automation only persists the amount already locked on the Booking
- Guaranteeing the charge is never duplicated across a dropped connection or an interrupted confirmation step -- owned by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency); this automation implements the create-and-confirm step that spec's guarantee wraps around
- Composing or sending the client's confirmation message -- excluded per product-features.md's Communications field: a successful deposit "feeds the confirmation message in Automated Booking Messaging (FEAT-08)," whose own Notification spec owns content and delivery

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Card charge reported captured | FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Fires when the payment-processing capability reports a successful capture for an attempt that passed FEAT-07.SPEC-003's eligibility check | Booking reference, captured amount, currency, processor_fee, capture timestamp |

This automation has exactly one trigger. It is fired once per successful capture event; a capture reported for a Booking that is not Pending Payment (per the idempotency guarantee in FEAT-07.SPEC-004) does not reach this automation as a new run -- see Edge Cases.

## Processing Logic

1. Receive the capture event from FEAT-07.SPEC-005: the Booking reference, captured amount, currency, processor_fee, and capture timestamp.
2. Confirm the referenced Booking's current state is Pending Payment and that no Deposit Transaction already exists for it (the one-charge-per-booking guarantee, enforced jointly with FEAT-07.SPEC-004). If a Deposit Transaction already exists for this Booking, treat this as a duplicate delivery of the same capture event and take no further action (see Edge Cases).
3. Create the Deposit Transaction record: amount and currency from the capture event, status set directly to Captured, processor_fee recorded, outcome_reason set to "deposit captured at booking," and the capture timestamp recorded.
4. In the same atomic step as creating the Deposit Transaction, transition the Booking's state from Pending Payment to Confirmed.
5. Signal FEAT-07.SPEC-001 (Deposit Payment) that the Booking is now Confirmed, so the screen can show its Success state and hand off to FEAT-05.SPEC-005 (Booking Confirmation).
6. Hand the confirmed Booking to FEAT-08 (Automated Booking Messaging) so the client's confirmation message can be composed and sent -- this automation does not compose or send that message itself.
7. Signal FEAT-16 (Booking & Payment Activity Record) that a deposit_payment_succeeded event occurred, for the append-only activity record.
8. Signal FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management) that the Booking's paid/unpaid status is now Confirmed/paid, so the Pro's schedule reflects it without a manual refresh.
9. Fire FEAT-25.SPEC-004 (Historical Aggregate Maintenance) with the Booking's Pro Account reference, service reference, and start_time, so the day's booking-count and per-service aggregates increment exactly once for this Booking-confirmed transition; a failure of that aggregate update never blocks or reverses this automation.
10. Fire FEAT-04.SPEC-005 (Booking-to-Calendar Sync) with the Booking reference (service, start_time, duration), so the confirmed Booking is written to the Pro's connected calendar; a failure of that write never blocks or reverses this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Capture confirmed | The referenced Booking is Pending Payment and has no existing Deposit Transaction | Deposit Transaction created (status: Captured); Booking transitions Pending Payment -> Confirmed | Riley sees the Success state and hand-off on FEAT-07.SPEC-001, then the full confirmation on FEAT-05.SPEC-005; Talia sees the booking as paid on her schedule | FEAT-07.SPEC-001, FEAT-05.SPEC-005, FEAT-08 (FEAT-08.SPEC-001, FEAT-08.SPEC-005, FEAT-08.SPEC-007), FEAT-12, FEAT-16 (FEAT-16.SPEC-002), FEAT-25 (FEAT-25.SPEC-004), FEAT-04 (FEAT-04.SPEC-005), FEAT-30 (FEAT-30.SPEC-010) |
| Duplicate capture event ignored | A Deposit Transaction already exists for the referenced Booking (the capture event was delivered more than once, or arrived after the Booking was already confirmed by an earlier delivery) | None -- the existing Deposit Transaction and Confirmed state are left unchanged | None -- Riley already saw the Success state from the first delivery; no second confirmation message fires | None beyond the existing state |
| Referenced Booking not eligible | The referenced Booking is not Pending Payment at the moment this automation runs (for example, it was already cancelled by an automation-expired hold, or a different terminal path already resolved it) | None -- no Deposit Transaction is created against an ineligible Booking | The capture is reported back to FEAT-07.SPEC-005 as unappliable, which FEAT-07.SPEC-004 resolves per its correctness guarantee (never silently drop a successful charge) | FEAT-07.SPEC-004, FEAT-07.SPEC-005 |
| Automation failure (processing error after capture confirmed) | The capture event is received but this automation cannot complete the create-and-confirm step (for example, an internal fault interrupts step 3 or 4) | No partial state is left visible: either both the Deposit Transaction and the Confirmed transition are committed together, or neither is | Riley's screen (FEAT-07.SPEC-001) shows the Offline/Degraded resolution defined by FEAT-07.SPEC-004 -- she is never shown an ambiguous or double-charged state; the automation retries the create-and-confirm step automatically | FEAT-07.SPEC-001, FEAT-07.SPEC-004 |

## Data Model

**Reads:** Booking -- state (must be Pending Payment), service, price_agreed, deposit_amount (to confirm the captured amount matches the locked computation), client reference. Deposit Transaction -- read internally to check for an existing record before creating a new one (the one-charge-per-booking guarantee).
**Creates:** Deposit Transaction -- amount, currency, status (set to Captured), processor_fee, outcome_reason, timestamps. Exactly one per Booking, ever, from this automation.
**Updates:** Booking -- state, from Pending Payment to Confirmed. This is the only Booking field this automation writes.
**Deletes:** None.

## Business Rules

- The Deposit Transaction creation and the Booking's Pending Payment -> Confirmed transition happen as a single atomic step -- one can never persist without the other (XBR-05).
- Exactly one Deposit Transaction is ever created per Booking; a second capture event for the same Booking is a duplicate delivery, never a second charge (FEAT-07.SPEC-004).
- The amount and currency written to the Deposit Transaction are exactly the values FEAT-07.SPEC-005 reports as captured, which must equal the amount FEAT-07.SPEC-003 locked on the Booking -- this automation does not recompute or adjust the amount.
- This automation is the sole writer of the Booking's Pending Payment -> Confirmed transition; no other feature ever performs this specific transition (per the Entity-Lifecycle Coverage Matrix).
- Confirmation is instant from the client's perspective: the Booking flips to Confirmed the moment capture is reported, not on a delay or batch cycle (product-features.md, Primary Flows: "the booking flips from pending to confirmed instantly").

## Edge Cases

- **Duplicate capture event for the same Booking** -- The second (and any subsequent) delivery finds an existing Deposit Transaction and takes no action; the Booking remains Confirmed with its original capture timestamp. No duplicate confirmation message fires.
- **Capture event arrives for a Booking no longer Pending Payment** -- If the Booking's state changed for a reason other than this automation (for example, an expired-hold automation already moved it out of Pending Payment before the capture event arrived), no Deposit Transaction is created against it; the outcome is escalated to FEAT-07.SPEC-004 as a payment succeeded against an ineligible booking, which that spec's correctness guarantee resolves rather than silently dropping the money.
- **Confirmation-step processing fails after the charge succeeded** -- The create-and-confirm step either fully commits or does not commit at all; a partial state (Deposit Transaction created but Booking still Pending Payment, or the reverse) never exists. If the step has not yet committed, it is retried automatically; Riley's screen shows the safe-retry behavior FEAT-07.SPEC-004 defines rather than a false failure.
- **Concurrent trigger firing (two capture events for two different Bookings at effectively the same time)** -- Each runs independently against its own Booking and Deposit Transaction; there is no shared state between two different Bookings' captures, so neither run affects the other.
- **Trigger fires while a previous run for the same Booking is still in flight** -- Cannot occur under normal operation, because FEAT-07.SPEC-004's one-charge-per-booking guarantee ensures a second charge attempt for the same Pending Payment Booking is never authorized while the first is being captured; if a second capture event nonetheless arrives before the first run has finished committing, it is treated exactly as the duplicate-capture-event case once the first run's Deposit Transaction becomes visible.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Triggered by (inbound) | A reported successful capture fires this automation |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | References (inbound) | Confirms the captured amount matches the locked computation and that the eligibility precondition held |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Governs the atomicity guarantee this automation implements and the resolution when a capture cannot be applied |
| FEAT-07.SPEC-001 (Deposit Payment) | Affects (outbound) | The Confirmed transition is what moves that screen into its Success state |
| FEAT-05.SPEC-005 (Booking Confirmation) | Affects (outbound) | The confirmed Booking is what this screen displays after hand-off |
| FEAT-08.SPEC-001 (Booking Confirmation Message), FEAT-08.SPEC-005 (Pro Booking Activity Notification), FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) -- within FEAT-08 (Automated Booking Messaging) | Affects (outbound) | The confirmed Booking feeds that feature's own confirmation-message Notification spec |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | deposit_payment_succeeded is written to the append-only activity record |
| FEAT-25.SPEC-004 -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | The Pending Payment -> Confirmed transition fires the booking-count and per-service aggregate increment (step 9) |
| FEAT-04.SPEC-005 -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | The Confirmed transition fires the write of the booking to the Pro's connected calendar (step 10) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on Talia's schedule |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on the Pro's booking management surface |

## Analytics and Success Signals

- **deposit_capture_confirmed** (booking reference, deposit amount, currency) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_capture_duplicate_ignored** (booking reference) -- N/A -- no Stage 2 metric measures duplicate-delivery frequency directly; retained so the one-charge-per-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric.
- **deposit_capture_not_applied** (booking reference, reason: booking_no_longer_eligible) -- supports success-metrics.md: "Deposit Capture Rate" (this is exactly the "ambiguous or lost state" the metric's 100% target rules out, so its occurrence must be visible)

## Acceptance Criteria

**FEAT-07.SPEC-002-AC-01:** Given Riley's card is authorized and captured for a Booking that is Pending Payment, when FEAT-07.SPEC-005 reports the capture, then a Deposit Transaction is created with status Captured and the Booking transitions to Confirmed in the same step.

**FEAT-07.SPEC-002-AC-02:** Given a Deposit Transaction was just created for a Booking, when the same capture event is delivered a second time, then no second Deposit Transaction is created and the Booking's Confirmed state is unchanged.

**FEAT-07.SPEC-002-AC-03:** Given a capture event arrives for a Booking that is no longer Pending Payment for a reason unrelated to this automation, when the automation checks the Booking's state, then no Deposit Transaction is created and the outcome is escalated to FEAT-07.SPEC-004.

**FEAT-07.SPEC-002-AC-04:** Given the Deposit Transaction is created and the Booking transitions to Confirmed, when the transition completes, then FEAT-07.SPEC-001 shows its Success state and Riley is handed off to FEAT-05.SPEC-005.

**FEAT-07.SPEC-002-AC-05:** Given a Booking has just been confirmed by this automation, when the confirmation completes, then the confirmed Booking is handed to FEAT-08 for the client's confirmation message, and this automation itself sends no message.

**FEAT-07.SPEC-002-AC-06:** Given a Booking has just been confirmed by this automation, when the transition completes, then a deposit_payment_succeeded event is written to the append-only activity record (FEAT-16).

**FEAT-07.SPEC-002-AC-07:** Given Talia is viewing her schedule (FEAT-12) at the moment a client's deposit is captured, when the automation confirms the Booking, then the booking's paid/unpaid status updates to paid without Talia needing to refresh.

**FEAT-07.SPEC-002-AC-08:** Given the create-and-confirm step is interrupted by a processing error after the charge succeeded, when the automation retries, then the Deposit Transaction and the Confirmed transition either both persist or neither does -- Riley is never shown a state where one exists without the other.

**FEAT-07.SPEC-002-AC-09:** Given two different clients' captures are reported at effectively the same time, when both automations run, then each creates its own Deposit Transaction and confirms its own Booking independently, with no interference between the two runs.

**FEAT-07.SPEC-002-AC-10:** Given a captured amount reported by FEAT-07.SPEC-005 for a Booking, when this automation writes the Deposit Transaction, then the recorded amount and currency exactly match the deposit_amount FEAT-07.SPEC-003 locked on that Booking.

**FEAT-07.SPEC-002-AC-11:** Given a Deposit Transaction has been created for a Booking, when any later capture event for that same Booking is evaluated, then it is recognized as a duplicate and produces no new Deposit Transaction, satisfying the one-charge-per-booking guarantee.

**FEAT-07.SPEC-002-AC-12:** Given a Booking is confirmed by this automation, when FEAT-30 (Pro Booking Management) next loads that booking, then its paid/unpaid status reflects Confirmed/paid.

**FEAT-07.SPEC-002-AC-13:** Given a Booking has just been confirmed by this automation, when the transition completes, then FEAT-25.SPEC-004 is fired with the Booking's Pro Account reference, service reference, and start_time, and a failure of that aggregate update leaves the Deposit Transaction and Confirmed state unchanged.

**FEAT-07.SPEC-002-AC-14:** Given a Booking has just been confirmed by this automation, when the transition completes, then FEAT-04.SPEC-005 is fired with the Booking reference so the booking is written to the Pro's connected calendar, and a failure of that write leaves the Deposit Transaction and Confirmed state unchanged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (confirmed, duplicate ignored, not applied, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 4 | 4 |

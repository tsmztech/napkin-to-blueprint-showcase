---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-007
spec_name: Pro-Created Deposit Request Hold & Expiration
spec_slug: pro-created-deposit-request-hold-expiration
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 12
---

# Automation Spec: Pro-Created Deposit Request Hold & Expiration

## Overview

**Name:** Pro-Created Deposit Request Hold & Expiration
**ID:** FEAT-03.SPEC-007
**Type:** Automation
**Purpose:** Reserves a slot the instant the Pro books a client in with a deposit request through FEAT-30, holding it up to 24 hours (platform parameter: `deposit-request-hold-max-hours`) or until 2 hours before the appointment (platform parameter: `deposit-request-hold-appointment-cutoff-hours`), whichever comes first, and expires the booking with a Pro notification if the deposit is never paid.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Creating a Slot Hold the instant the Pro books a client in and requests a deposit through FEAT-30
- Setting the hold's expiry to whichever comes first: platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment
- Re-validating the candidate slot against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception applied)
- Resolving a contested slot per FEAT-03.SPEC-005
- Expiring the hold and marking the associated Booking Expired (unpaid) -- this spec is the sole writer of that Booking transition -- and triggering the Pro notification, if the deposit is never paid

**Non-Goals:**
- The checkout hold class created when a client pays through the public booking flow -- owned by FEAT-03.SPEC-002/FEAT-03.SPEC-003, a distinct creation path and a distinct (fixed, few-minute) timeout, per the Entity-Lifecycle Coverage Matrix's "two creation paths, one Slot Hold shape" note
- Generating or sending the deposit-request link itself, or capturing the client's payment against it -- owned by FEAT-30 (Pro Booking Management) and FEAT-07 (Deposit Payment at Booking); this spec only reserves the time slot the request is for
- Composing or delivering the Pro's expiry notice content -- FEAT-08.SPEC-006 (Pro Attention Alert) is the content owner, with FEAT-30.SPEC-013 referencing it for dashboard surfacing; this spec is only the trigger and does not define their wording
- Writing the Booking's Expired (unpaid) state from any other spec -- FEAT-30.SPEC-010 (and FEAT-30.SPEC-013) rely on this spec's transition and never write it themselves (XBR-02 authority: FEAT-03)
- Recurring-occurrence release timing -- excluded per this Brief's Non-Goals: XBR-02 assigns an unpaid recurring occurrence's release at its cancellation cut-off to FEAT-21; this spec supplies only the generic hold/release mechanism FEAT-21 invokes

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro books a client in and creates a deposit request | FEAT-30 (Pro Booking Management) | Fires the instant Talia, at the chair or from her dashboard, books a client into a chosen slot and chooses to request a deposit rather than collect it in person | Service ID, chosen start time and duration, the Client reference, the appointment time the 2-hour cutoff is measured against |
| A Pro-created deposit-request hold reaches its expiry | (self -- schedule-based, derived from the hold's own expiry timestamp) | Fires the moment the hold's expiry timestamp passes with the hold still Active (deposit not paid) | The hold's service, start time, duration, owning Booking reference |

## Processing Logic

1. **Hold creation path:** Receive the candidate Service ID, start time, duration, and Client reference from FEAT-30's booking-in step.
2. Re-validate the candidate slot against FEAT-03.SPEC-004's rules, applying the Pro-only exception to minimum notice and booking horizon (the fit rule still applies unexempted).
3. Check for any existing active Slot Hold, Booking, Time Block, Recurring Series occurrence, or calendar busy period already covering the candidate time.
4. If no conflict exists, create a new Slot Hold: service, start time, duration, the owning Booking (created in Pending Payment state by FEAT-30), hold-created timestamp, and an expiry timestamp computed as the earlier of (creation time + platform parameter: `deposit-request-hold-max-hours`) and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`), state Active.
5. If a conflict exists, apply the contention resolution rule (FEAT-03.SPEC-005).
6. The created hold immediately excludes the candidate time from the next slot-list computation (FEAT-03.SPEC-001) for every client.
7. **Expiration path:** Identify each Active Pro-created deposit-request hold whose computed expiry timestamp has passed.
8. Confirm the hold has not already transitioned to Consumed by a completed deposit payment (FEAT-07) -- if it has, take no action.
9. Transition the hold's state to Expired and delete the hold record.
10. Mark the associated Booking's state as Expired (unpaid). This spec is the sole writer of the Booking -> Expired (unpaid) transition; FEAT-30.SPEC-010 does not repeat it and instead reads the resulting state, and no other automation may set it.
11. Trigger the Pro notification that the deposit was never paid: content is owned by FEAT-08.SPEC-006 (Pro Attention Alert, in-app and message) and surfaced on the dashboard via FEAT-30.SPEC-013; this spec supplies only the trigger and the Booking reference.
12. The freed time reappears the next time FEAT-03.SPEC-001 computes the open-slot list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deposit-request hold created | Candidate slot passes re-validation (with Pro exception) and no conflict exists | New Slot Hold created (Active); Booking created in Pending Payment state | Talia sees the booking on her dashboard as awaiting deposit payment; the client receives the deposit-request link (FEAT-30, FEAT-08) | FEAT-30, FEAT-03.SPEC-001 |
| Slot contested at creation | Another hold, Booking, Time Block, or calendar event already occupies the candidate time | None | Talia sees the slot is no longer available and is returned to a refreshed live view, per FEAT-03.SPEC-005 | FEAT-03.SPEC-005, FEAT-30 |
| Deposit paid before expiry | Client completes the deposit payment while the hold is still Active | Hold transitioned to Consumed; Booking transitioned to Confirmed (FEAT-07) | Talia and the client both see the booking confirmed | FEAT-07, FEAT-30 |
| Hold expired -- deposit never paid | The computed expiry timestamp passes with the hold still Active | Hold transitioned to Expired then deleted; Booking transitioned to Expired (unpaid), written only by this spec | Talia is notified the deposit was never paid, with content owned by FEAT-08.SPEC-006 and surfaced via FEAT-30.SPEC-013 (dashboard); the client receives no further reminder for this booking | FEAT-30.SPEC-010, FEAT-30.SPEC-013, FEAT-08.SPEC-006, FEAT-03.SPEC-001 |
| Hold creation failure | The hold cannot be written (e.g., a processing error) | None | Talia sees a retry prompt on FEAT-30; the deposit-request link is not sent until a confirmed hold exists | FEAT-30 |

## Data Model

**Reads:** Service (duration, buffer_override), Availability Rule (for re-validation, notice/horizon exempted for Pro requests), Booking, Time Block, Recurring Series, existing Slot Hold records, Calendar Connection busy periods (via FEAT-03.SPEC-006).
**Creates:** Slot Hold -- service, start time, duration, owning Booking reference, hold-created timestamp, computed expiry timestamp, state Active.
**Updates:** Slot Hold -- state transitioned to Expired on timeout, or Consumed on payment completion (by FEAT-07). Booking -- state transitioned to Expired on hold expiration.
**Deletes:** Slot Hold -- the expired record is removed, per the Entity-Lifecycle Coverage Matrix (no restore path; no retention).

## Business Rules

- The Pro-created deposit-request hold's expiry is computed as the earlier of two limits: platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment (XBR-02) -- whichever boundary is reached first governs.
- The Pro-only exception to minimum notice and booking horizon applies to this hold's creation (FEAT-03.SPEC-004), since the triggering action is always a Pro-side booking through FEAT-30; the fit rule (duration+buffer) is never exempted.
- The first committed action wins a race between an expiring deposit-request hold and a completing deposit payment (XBR-01, FEAT-03.SPEC-005): if payment completes before this automation processes the expiry, the hold is Consumed and the Booking Confirmed, not Expired.
- An expired deposit-request hold's associated Booking is marked Expired (unpaid), never silently deleted -- this preserves the record that a booking was attempted and lapsed, distinct from the transient hold itself which is deleted (Entity-Lifecycle Coverage Matrix).
- This spec is the sole writer of Booking -> Expired (unpaid) for a Pro-created deposit request (XBR-02 authority: FEAT-03); FEAT-30.SPEC-010 relies on this transition and does not perform it.
- The Pro is always notified on expiration through two channels (FEAT-30 dashboard, FEAT-08 message) -- never silently, since this is money the Pro was counting on that never arrived. FEAT-08.SPEC-006 owns the notice content; this spec only triggers it.
- The hold-window values (platform parameter: `deposit-request-hold-max-hours`, platform parameter: `deposit-request-hold-appointment-cutoff-hours`) and the Pro-only notice/horizon exception are stated by FEAT-30.SPEC-006 (Pro Booking Action Rules); this spec consumes them and computes and enforces the expiry.

## Edge Cases

- **Deposit payment completes in the same instant the hold's computed expiry elapses** -- The completed-payment transition (to Consumed, Booking Confirmed) takes precedence; this automation detects the already-Consumed state before processing the expiry and takes no further action.
- **The appointment is scheduled less than platform parameter: `deposit-request-hold-appointment-cutoff-hours` away at the moment of creation** -- The 2-hour-before-appointment limit is already closer than the 24-hour cap, so the hold's expiry is set to that earlier boundary immediately; if the appointment is itself less than the cutoff away from the current moment, the hold's effective window is correspondingly short, and Talia is not blocked from creating it (the Pro-only notice exception applies to hold creation, not to how soon the resulting hold itself may then expire).
- **Talia cancels the booking herself before the deposit is paid or the hold expires** -- The cancellation (via FEAT-30) transitions the Booking and deletes the hold directly, outside this automation's own expiration path; no expiration notification fires for a Pro-initiated cancellation.
- **Concurrent trigger firing (Talia books two different clients into two different slots at effectively the same time)** -- Each hold-creation attempt is processed independently against its own distinct candidate slot; no interference occurs since the slots do not overlap.
- **Trigger fires while a previous hold-creation attempt for the same booking-in action is still in flight** -- FEAT-30's booking-in control is disabled during submission, preventing a duplicate hold-creation request for the same client and slot.
- **A second Pro-created deposit request is attempted for the same slot after the first hold already exists** -- The second attempt is refused per FEAT-03.SPEC-005, since the first hold committed first; Talia is shown the slot is already held.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The created hold immediately excludes the slot from the next computation; an expired hold's slot reappears |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Candidate slot is re-validated at hold-creation time, with the Pro-only notice/horizon exception applied |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when the candidate slot is contested |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) / Affects (outbound) | The booking-in-with-deposit-request action triggers hold creation; this spec's expiration path is the sole writer of Booking -> Expired (unpaid), and FEAT-30.SPEC-010 relies on that transition and reflects it on the Pro's dashboard |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) -- within FEAT-30 | References (inbound) | States the hold-window values and the Pro-only notice/horizon exception this spec applies when creating and expiring the hold |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) -- within FEAT-30 | Affects (outbound) | Surfaces the expiry notice on the Pro's dashboard; references FEAT-08.SPEC-006 and defines no duplicate content |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Content owner of the Pro's unpaid-deposit expiry notice; this spec is its trigger when a hold expires unpaid |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) / Triggered by (inbound, on payment completion) | Converts the hold to Consumed and the Booking to Confirmed when the deposit is paid |
| FEAT-08 (Automated Booking Messaging) | Triggers (outbound) | An expired, unpaid deposit-request hold triggers the Pro notification message (content per FEAT-08.SPEC-006) |

## Analytics and Success Signals

- **slot_held** (service_id, hold_type: pro_deposit_request, start_time) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_expired** (service_id, hold_type: pro_deposit_request) -- supports success-metrics.md: "Pro Change Correctness" (the target's "at least 70% of deposit requests the pro sends when rebooking at the chair are paid before the hold expires" is directly measured by the paid-vs-expired split of this event alongside the Consumed outcome)
- **deposit_request_hold_creation_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **deposit_request_hold_contested** (service_id, outcome: won / lost) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given Talia books a client in at the chair and requests a deposit, when the hold-creation step runs, then a Slot Hold is created instantly, excluding the slot from every client's computed list.

**FEAT-03.SPEC-007-AC-02:** Given Talia creates a deposit-request hold for an appointment more than 24 hours away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-max-hours` from creation, since that boundary is reached first.

**FEAT-03.SPEC-007-AC-03:** Given Talia creates a deposit-request hold for an appointment less than platform parameter: `deposit-request-hold-max-hours` away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, since that boundary is reached first.

**FEAT-03.SPEC-007-AC-04:** Given Talia books a client in for a time that would be inside her own minimum_booking_notice for a client, when the hold-creation step re-validates the candidate, then the candidate is not excluded on notice grounds, per the Pro-only exception.

**FEAT-03.SPEC-007-AC-05:** Given a deposit-request hold's computed expiry passes with the deposit never paid, when the expiration automation runs, then the hold is expired and deleted, the associated Booking is marked Expired (unpaid) by this automation alone (no other spec writes that transition), and Talia is notified via her dashboard (FEAT-30.SPEC-013) and a message whose content is owned by FEAT-08.SPEC-006.

**FEAT-03.SPEC-007-AC-06:** Given the client completes the deposit payment before the hold's expiry, when payment completes, then the hold is transitioned to Consumed and the Booking to Confirmed, and no expiration notification fires.

**FEAT-03.SPEC-007-AC-07:** Given a deposit-request hold's expiry and a completing payment occur at effectively the same moment, when both are evaluated, then the completed payment takes precedence and the hold is Consumed, not Expired.

**FEAT-03.SPEC-007-AC-08:** Given Talia attempts to create a second deposit-request hold on a slot already held by a first deposit request, when the second attempt is evaluated, then it is refused per FEAT-03.SPEC-005 and Talia sees the slot is already held.

**FEAT-03.SPEC-007-AC-09:** Given Talia cancels a booking herself before its deposit-request hold expires, when the cancellation completes, then the hold is deleted directly by that cancellation, and no expiration notification fires.

**FEAT-03.SPEC-007-AC-10:** Given a deposit-request hold cannot be created due to a processing error, when Talia attempts to book a client in with a deposit request, then Talia sees a retry prompt and no deposit-request link is sent.

**FEAT-03.SPEC-007-AC-11:** Given a deposit-request hold expires, when Riley next requests the same service's slot list, then the freed time appears as open.

**FEAT-03.SPEC-007-AC-12:** Given Talia books two different clients into two different, non-overlapping slots at effectively the same time, when both hold-creation attempts run, then each succeeds independently with no interference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-006
spec_name: Slot Hold & Re-Validation at Checkout
spec_slug: slot-hold-re-validation-at-checkout
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Slot Hold & Re-Validation at Checkout

## Overview

**Name:** Slot Hold & Re-Validation at Checkout
**ID:** FEAT-05.SPEC-006
**Type:** Automation
**Purpose:** Creates the client's Booking in Pending Payment state and places a checkout hold on the chosen slot when the client advances into the deposit payment step (FEAT-05.SPEC-004 -> FEAT-07.SPEC-001), aligned to FEAT-03.SPEC-002; re-validates the hold immediately before charging, and resolves expiry or contention outcomes by returning the client to a fresh slot list.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Creating the Booking record in Pending Payment state the instant the client advances into the deposit payment step, in step with placing a checkout hold on the chosen slot (via FEAT-03.SPEC-002) -- not at slot pick
- Re-validating the held slot immediately before the deposit charge is attempted
- Transitioning the Booking to Expired (unpaid) when the underlying hold expires unpaid
- Resolving a contested slot in favor of the first client to complete payment, per XBR-01

**Non-Goals:**
- Computing slot availability or owning the Slot Hold entity itself, its timeout, or its contention resolution mechanics -- entirely owned by FEAT-03 (Real-Time Slot Availability Engine, specifically FEAT-03.SPEC-002, FEAT-03.SPEC-003, and FEAT-03.SPEC-005); this spec triggers and reacts to those mechanics but does not re-implement them
- Processing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only ensures the slot is still valid immediately before that hand-off
- Placing or expiring the Pro-created deposit-request hold, or holds created through a client reschedule (FEAT-10) or a Pro-side booking (FEAT-30) -- this spec covers only the client-initiated new-booking path through FEAT-05 (Entity-Lifecycle Coverage Matrix's note that Booking is also created by other features via other paths)
- Transitioning the Booking from Pending Payment to Confirmed -- owned by FEAT-07 once the deposit succeeds

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client advances into the deposit payment step | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Fires when the client taps Acknowledge & continue, after policy acknowledgment passes its own integrity check (FEAT-05.SPEC-009) and the availability gate (FEAT-05.SPEC-008) re-check; timing per FEAT-03.SPEC-002's trigger ("advances into the deposit payment step") | Service ID, chosen start time and duration, the client's entered details and Client reference, the acknowledged policy version |
| Client submits payment | FEAT-07.SPEC-001 (Deposit Payment) | Fires when the client taps Pay on the payment screen, before the charge is attempted (the pre-charge hold-still-active check) | The Booking's held slot reference |
| Checkout hold expires unpaid | FEAT-03.SPEC-003 (Slot Hold Expiration) | Fires when the checkout Slot Hold owning this Booking reaches its timeout without a completed payment | The expired hold's owning Booking reference |

## Processing Logic

1. On an advance-into-payment trigger from FEAT-05.SPEC-004: if this client's in-progress checkout already has an Active hold and Pending Payment Booking for the same slot (the client came back from the payment screen), reuse them and skip to step 4; otherwise request a checkout Slot Hold from FEAT-03.SPEC-002 for the chosen service, start time, and duration.
2. If the hold is created successfully, create the Booking record in Pending Payment state, fixing service, start_time, duration, client reference, price_agreed and deposit_amount (computed from the Service's current rule), the acknowledged policy version, wording and timestamp (from FEAT-05.SPEC-009), and source: "client link".
3. If the hold creation is refused (the slot no longer validates, or it is contested and lost per FEAT-03.SPEC-005), take no Booking-creation action and signal the triggering screen (FEAT-05.SPEC-004) with the specific reason (no-longer-valid or contested); that screen returns the client to FEAT-05.SPEC-002.
4. Signal FEAT-05.SPEC-004 that the hold is placed so the client is navigated to FEAT-07.SPEC-001 (Deposit Payment).
5. On a payment-submission trigger from FEAT-07.SPEC-001: re-request validation of the Booking's held slot from FEAT-03 (confirming the hold is still Active and has not expired or been lost).
6. If the hold is still valid, allow FEAT-07 to collect and capture the deposit against this Booking; if it is no longer valid (expired or lost in the moment between reaching the payment screen and tapping Pay), take no payment hand-off and signal FEAT-07.SPEC-001 with the specific reason (its Hold Expired state).
7. On a hold-expiration trigger from FEAT-03.SPEC-003: confirm the owning Booking is still Pending Payment (not already Confirmed by a payment that completed in the same instant); if still Pending Payment, transition the Booking to Expired (unpaid).
8. Signal the client, if still on a screen within the flow (the payment screen, FEAT-07.SPEC-001, or FEAT-05.SPEC-004 after a back navigation), that their held time expired, and return them to a refreshed slot list (FEAT-05.SPEC-002).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold placed, Booking created | Slot passes re-validation and hold creation succeeds when the client advances into the payment step | New Booking created (Pending Payment) | Client advances to FEAT-07.SPEC-001 with the slot reserved | FEAT-05.SPEC-004, FEAT-07.SPEC-001 |
| Existing hold reused | The client returns to FEAT-05.SPEC-004 from the payment screen and continues again while the hold is still Active | None -- no second hold or Booking | Client advances to FEAT-07.SPEC-001 again | FEAT-05.SPEC-004, FEAT-07.SPEC-001 |
| Slot no longer valid at continue time | FEAT-03.SPEC-002 rejects the candidate slot as no longer meeting timing rules | No Booking created | Client sees "That time is no longer available." and a refreshed slot list | FEAT-05.SPEC-004, FEAT-05.SPEC-002 |
| Slot contested at continue time | Another client's hold-creation attempt for the same slot commits first | No Booking created for the losing client | Client sees "That time was just taken." and a refreshed slot list | FEAT-05.SPEC-004, FEAT-05.SPEC-002, FEAT-03.SPEC-005 |
| Slot re-validated, payment proceeds | Hold is still Active at payment-submission time | None yet -- hand-off to FEAT-07 begins | Client sees the payment step proceed normally | FEAT-07.SPEC-001, FEAT-07 |
| Slot lost before payment | Hold has expired or was lost to contention by the time payment is submitted | Booking already transitioned to Expired (unpaid) by the hold-expiration path, or no Booking to charge | Client sees "Your held time expired. Pick a new time to continue." on the payment screen and returns to FEAT-05.SPEC-002 | FEAT-07.SPEC-001, FEAT-05.SPEC-002 |
| Booking expired unpaid | The underlying checkout hold expires with payment never completed | Booking transitioned from Pending Payment to Expired (unpaid); the slot is released | Client, if still present, sees the expiry message and a refreshed list; if already left, nothing further happens | FEAT-05.SPEC-002, FEAT-03.SPEC-001 |
| Race won by payment | Payment completes at effectively the same moment the hold's timeout is reached | Booking transitions to Confirmed via FEAT-07 instead of Expired (unpaid) | Client sees their booking confirmed, not an expiry message | FEAT-07, FEAT-05.SPEC-005 |

## Data Model

**Reads:** Service -- price, deposit_rule, duration, buffer_override (to compute price_agreed and deposit_amount). Slot Hold (via FEAT-03) -- state, expiry timestamp. The in-progress checkout details (Client reference, acknowledged policy version, wording, timestamp from FEAT-05.SPEC-009).
**Creates:** Booking -- service, start_time, duration, client reference, price_agreed, deposit_amount, policy_version and acknowledgment fields (as captured by FEAT-05.SPEC-009), state: Pending Payment, source: "client link".
**Updates:** Booking -- state transitioned to Expired (unpaid) on unpaid hold expiry.
**Deletes:** None -- an Expired (unpaid) Booking is retained as history, per SC-22; only the underlying Slot Hold (owned by FEAT-03) is deleted on expiry.

## Business Rules

- A time is offered, held, or booked only if it passes the live slot check owned by FEAT-03; this automation never assumes a slot computed a moment earlier is still valid without re-validating it (XBR-01).
- The hold and the Pending Payment Booking begin only when the client advances into the deposit payment step, exactly as FEAT-03.SPEC-002 specifies (XBR-02 authority); picking a slot on FEAT-05.SPEC-002 places no hold and creates no Booking.
- The checkout hold's timeout is fixed and short: platform parameter: `checkout-hold-timeout-minutes` (same marker as defined in FEAT-03.SPEC-002 -- reused verbatim).
- The first client to complete payment wins a contested slot; the other client sees a plain "just taken" message, never a payment error (XBR-01).
- The deposit amount fixed on Booking creation (price_agreed, deposit_amount) is computed once from the Service's rule at that moment and never recalculated afterward, even if the Pro edits the service before payment completes (XBR-04, XBR-05).
- An Expired (unpaid) Booking is never charged and is retained as history rather than deleted, consistent with the product-wide no-deletion policy for bookings (SC-22).

## Edge Cases

- **Client abandons the flow before advancing into the payment step** -- No hold or Booking exists, so nothing needs to expire or be cleaned up.
- **Client abandons the flow after a hold and Booking are created but never reaches payment** -- The Booking remains Pending Payment until the hold's timeout elapses, then both the hold (via FEAT-03.SPEC-003) and this spec's own expiration path resolve to Expired (unpaid); no separate abandonment signal is needed.
- **Payment completes in the same instant the hold's timeout elapses** -- The completed-payment transition to Confirmed (FEAT-07) takes precedence; Step 7 detects the Booking is no longer Pending Payment and takes no expiration action.
- **Client's device is offline when their hold expires** -- The Booking still expires server-side on schedule; the client sees the expiry message on their next successful interaction, never a silently-accepted late payment against an expired hold.
- **The chosen service is archived between hold creation and payment submission** -- Step 5's re-validation surfaces this as a slot-no-longer-valid condition; the client is refused with a refresh back to the service list (FEAT-05.SPEC-004's Business Rules), not charged; before the hold is placed, the same archived service is caught on Acknowledge & continue and nothing is held.
- **Concurrent trigger firing (two clients advance into payment for the same slot at effectively the same time)** -- Exactly one hold-creation-and-Booking-creation attempt succeeds per FEAT-03.SPEC-005's first-committed-wins rule; the other client's attempt produces no Booking at all, so no orphaned Pending Payment record is ever created for the losing attempt.
- **Trigger fires while a previous run is in flight for the same client** -- The triggering screen (FEAT-05.SPEC-004 or FEAT-07.SPEC-001) debounces its own submit control while an attempt is in progress, so a second concurrent run for the same client's same Booking cannot start; a second run for a different Booking proceeds independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Triggered by (inbound) / Affects (outbound) | Acknowledge & continue triggers hold-and-Booking creation; the outcome (hold placed, or slot lost) is reported back to this screen |
| FEAT-05.SPEC-002 (Slot Selection) | Affects (outbound) | An expired hold or a lost slot returns the client here with a refreshed list and message; this screen no longer triggers anything on slot pick |
| FEAT-07.SPEC-001 (Deposit Payment) | Triggered by (inbound) / Affects (outbound) | The Pay tap triggers pre-payment re-validation; a lost or expired hold is reported back to the payment screen |
| FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Triggers (outbound) | This spec requests hold creation from FEAT-03's own automation |
| FEAT-03.SPEC-003 (Slot Hold Expiration) | Triggered by (inbound) | This spec's Booking-expiration step fires in response to FEAT-03's own hold-expiration event |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when a candidate slot is contested |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) | Hands off to deposit collection once the slot is re-validated at payment time |

## Analytics and Success Signals

- **checkout_booking_created** (service ID, source: client link) -- supports success-metrics.md: "Booking Completion Speed"; the same event, tallied by its source property against a Pro's total bookings over time, is the basis for supports success-metrics.md: "DM-to-Link Migration"
- **checkout_slot_rejected** (reason: no_longer_valid / contested) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_slot_revalidated_at_payment** (result: valid / expired / contested) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_booking_expired_unpaid** (service ID) -- supports success-metrics.md: "Deposit Capture Rate" (an expired hold is a clean, non-ambiguous non-capture outcome the metric expects, not a lost or stuck state)

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Riley has acknowledged the policy on FEAT-05.SPEC-004, when she taps Acknowledge & continue, then a checkout Slot Hold is placed and a Booking is created in Pending Payment state, and Riley advances to FEAT-07.SPEC-001; picking the slot earlier on FEAT-05.SPEC-002 created neither.

**FEAT-05.SPEC-006-AC-02:** Given Riley's chosen slot no longer passes the live validation rules at the instant she taps Acknowledge & continue, when the hold-creation request runs, then no Booking is created and Riley sees "That time is no longer available." with a refreshed slot list.

**FEAT-05.SPEC-006-AC-03:** Given Riley and another client advance into payment for the same slot at effectively the same time, when hold creation resolves, then exactly one of them ends up with a Pending Payment Booking, and the other sees "That time was just taken."

**FEAT-05.SPEC-006-AC-04:** Given Riley taps Pay on FEAT-07.SPEC-001 with her hold still Active, when the pre-payment re-validation runs, then the slot is confirmed still valid and the hand-off to FEAT-07 proceeds.

**FEAT-05.SPEC-006-AC-05:** Given Riley's checkout hold has expired by the time she taps Pay, when the pre-payment re-validation runs, then no payment hand-off occurs and Riley sees the payment screen's hold-expired message and is returned to FEAT-05.SPEC-002 with a refreshed list.

**FEAT-05.SPEC-006-AC-06:** Given Riley's checkout hold reaches its fixed timeout while she is still mid-flow with no payment completed, when the expiration event fires, then her Booking transitions from Pending Payment to Expired (unpaid) and the slot is released.

**FEAT-05.SPEC-006-AC-07:** Given Riley's payment completes at effectively the same moment her hold's timeout is reached, when both processes evaluate, then her Booking transitions to Confirmed and the expiration path takes no action against it.

**FEAT-05.SPEC-006-AC-08:** Given the chosen service is archived by the Pro between Riley advancing into payment and her payment submission, when the pre-payment re-validation runs, then Riley is refused with a refresh back to the service list, never charged.

**FEAT-05.SPEC-006-AC-09:** Given Riley abandons the flow after a Booking is created but never pays, when the hold's fixed timeout elapses, then her Booking is expired and no charge is ever attempted against it.

**FEAT-05.SPEC-006-AC-10:** Given two clients' checkout attempts for different slots resolve at the same moment, when both are processed, then each is handled independently with no interference between them.

**FEAT-05.SPEC-006-AC-11:** Given Riley's Booking is created in Pending Payment state, when the price_agreed and deposit_amount are set, then they are computed once from the Service's rule at that moment and are never recalculated even if the Pro edits the service before payment completes.

**FEAT-05.SPEC-006-AC-12:** Given Riley returns to FEAT-05.SPEC-004 from FEAT-07.SPEC-001 while her hold is still Active and continues again, when the trigger fires, then the existing hold and Pending Payment Booking are reused and no second hold is placed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 8 | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |

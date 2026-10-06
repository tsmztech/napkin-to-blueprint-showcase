---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-002
spec_name: Slot Hold Creation & Checkout Reservation
spec_slug: slot-hold-creation-checkout-reservation
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 10
---

# Automation Spec: Slot Hold Creation & Checkout Reservation

## Overview

**Name:** Slot Hold Creation & Checkout Reservation
**ID:** FEAT-03.SPEC-002
**Type:** Automation
**Purpose:** Creates a time-limited Slot Hold the instant a client begins paying for a chosen slot, instantly excluding it from every other client's computed availability so no second client can grab it mid-checkout.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Creating a Slot Hold at the moment a client begins the payment step of checkout
- Re-validating the candidate slot against FEAT-03.SPEC-004's rules at the instant of hold creation (a slot computed a moment earlier must still pass)
- Resolving a contested slot per FEAT-03.SPEC-005 when two clients attempt to hold the same slot concurrently
- Handing the created hold's identity to the payment step so it can be converted to a Booking on payment completion

**Non-Goals:**
- Processing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reserves the time slot the payment is for
- Expiring or releasing the hold -- owned by FEAT-03.SPEC-003, which this spec's created hold is subject to
- The Pro-created deposit-request hold class (longer-lived, up to platform parameter: `deposit-request-hold-max-hours`) -- owned by FEAT-03.SPEC-007, a distinct creation path for a distinct trigger (a Pro booking a client in), per the Entity-Lifecycle Coverage Matrix's two-creation-paths note
- Converting a hold into a confirmed Booking -- a cross-feature outcome owned by FEAT-07 when payment completes (Booking creation is FEAT-07's write, not this automation's)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client begins the payment step for a chosen slot | FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) -- within FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) | Fires the instant a client, having selected a service and slot, advances into the deposit payment step | Service ID, chosen start time and duration, the in-progress Booking's Client reference |
| Client begins the payment step for a reschedule | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client, rescheduling an existing booking, advances to confirm the new time (no new deposit charge if within policy, but the new time must still be held while the change commits) | Service ID, new start time and duration, the existing Booking reference |

## Processing Logic

1. Receive the candidate Service ID, start time, and duration from the triggering checkout step.
2. Re-validate the candidate slot against FEAT-03.SPEC-004's rules (duration+buffer fit, minimum notice, booking horizon) exactly as at computation time -- a slot that no longer passes is rejected before any hold is attempted.
3. Check for any existing active Slot Hold (of either class) already covering the candidate start time.
4. If no conflicting hold, Booking, Time Block, Recurring Series occurrence, or calendar busy period covers the candidate time, create a new Slot Hold: service, start time, duration, the owning in-progress checkout (or reschedule), a hold-created timestamp, an expiry timestamp set to the fixed checkout-hold timeout, and state Active.
5. If a conflicting hold or occupancy is found, apply the contention resolution rule (FEAT-03.SPEC-005) to determine the outcome for this attempt.
6. Return the created hold's identity to the triggering checkout step so the payment step can proceed against it.
7. The created hold immediately excludes the candidate time from the next slot-list computation (FEAT-03.SPEC-001) for every other client.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold created | Candidate slot passes re-validation and no conflict exists | New Slot Hold created (Active) | Client proceeds to the deposit payment step with the slot reserved | FEAT-05, FEAT-07, FEAT-03.SPEC-001 |
| Slot no longer valid | Re-validation against FEAT-03.SPEC-004 fails (e.g., minimum notice now violated) | None | Client sees a plain message that the slot is no longer available and is returned to a refreshed live slot list | FEAT-05, FEAT-03.SPEC-001 |
| Slot contested | Another hold, Booking, or calendar event already occupies the candidate time | None for the losing attempt | Client sees the plain "just taken" message per FEAT-03.SPEC-005 and a refreshed live list | FEAT-03.SPEC-005, FEAT-05 |
| Hold creation failure | The hold cannot be written (e.g., a processing error) | None | Client sees a retry prompt; no payment step is entered without a confirmed hold | FEAT-05 |

## Data Model

**Reads:** Service (duration, buffer_override), Availability Rule (for re-validation), Booking, Time Block, Recurring Series, existing Slot Hold records, Calendar Connection busy periods (via FEAT-03.SPEC-006).
**Creates:** Slot Hold -- service, start time, duration, owning checkout/deposit-request reference, hold-created timestamp, expiry timestamp (fixed checkout-hold timeout), state Active.
**Updates:** None -- this spec only creates; expiration and conversion are owned elsewhere (FEAT-03.SPEC-003, FEAT-07).
**Deletes:** None.

## Business Rules

- A Slot Hold is created only after the candidate slot re-passes every rule in FEAT-03.SPEC-004 -- a slot computed a moment earlier is never assumed still valid (XBR-01).
- The checkout hold's expiry is a fixed, short timeout: platform parameter: `checkout-hold-timeout-minutes`.
- The first client to successfully create a hold on a given slot wins it; a second, concurrent attempt on the same slot is resolved per FEAT-03.SPEC-005's first-committed-wins rule (XBR-01, XBR-02).
- A reschedule's new-time hold follows the same creation rule as a new booking's hold -- the Brief's Shared Validation note that SPEC-002 re-validates rather than re-deriving FEAT-03.SPEC-004's rules.
- One hold exists per contested slot at a time; no two Active holds can cover the same overlapping time for the same Pro Account.

## Edge Cases

- **Client abandons checkout after a hold is created but before payment starts** -- The hold remains Active until its timeout, then expires per FEAT-03.SPEC-003; no separate abandonment signal is needed.
- **Candidate slot re-validation fails because minimum notice was crossed while the client was choosing** -- The client sees a plain "this time is no longer available" message and a refreshed live list, never a payment error.
- **Concurrent trigger firing (two clients begin checkout for the same slot at effectively the same time)** -- Exactly one hold-creation attempt succeeds; the other is refused per FEAT-03.SPEC-005, with no partial or duplicate hold ever existing.
- **Trigger fires while a previous hold-creation attempt for the same client is still in flight** -- The client's checkout step disables further submission until the in-flight attempt resolves, preventing a duplicate hold request from the same client.
- **Reschedule hold contends with the booking's own original time** -- The booking's own currently-held original time is never treated as a conflict against its own reschedule attempt; only the new candidate time is checked for conflicts.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The created hold immediately excludes the slot from the next computation |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Candidate slot is re-validated against these rules at hold-creation time |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when the candidate slot is contested |
| FEAT-03.SPEC-003 (Slot Hold Expiration) | Affects (outbound) | The created hold is subject to this spec's timeout and release logic |
| FEAT-05 (Public Booking Page & Booking Flow) | Triggered by (inbound) / Affects (outbound) | Checkout's payment step triggers hold creation and receives the outcome |
| FEAT-07 (Deposit Payment at Booking) | Triggered by (inbound) / Affects (outbound) | Payment step begins against the created hold; converts it to a Booking on success |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A reschedule's new-time confirmation also creates a hold through this spec |

## Analytics and Success Signals

- **slot_held** (service_id, hold_type: checkout, start_time) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_creation_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_contested_at_checkout** (service_id, outcome: won / lost) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_started** (service_id) -- supports success-metrics.md: "Booking Completion Speed" (marks the start of the timed checkout window this metric measures)

## Acceptance Criteria

**FEAT-03.SPEC-002-AC-01:** Given Riley selects a genuinely free slot and taps to begin payment, when the checkout step advances, then a Slot Hold is created instantly and Riley proceeds to the deposit payment step.

**FEAT-03.SPEC-002-AC-02:** Given a Slot Hold has just been created for Riley's chosen slot, when a second client (a different Riley) requests the same service's slot list moments later, then the held slot does not appear as open.

**FEAT-03.SPEC-002-AC-03:** Given Riley's chosen slot no longer passes FEAT-03.SPEC-004's minimum-notice rule by the time checkout begins, when the hold-creation step re-validates it, then Riley sees a plain "this time is no longer available" message and a refreshed live list, never a payment error.

**FEAT-03.SPEC-002-AC-04:** Given two clients begin checkout for the same slot at effectively the same time, when hold creation runs for both, then exactly one succeeds and the other sees the plain "just taken" message per FEAT-03.SPEC-005.

**FEAT-03.SPEC-002-AC-05:** Given Riley abandons checkout after a hold is created but never reaches payment, when the checkout-hold timeout elapses, then the hold expires per FEAT-03.SPEC-003 and the slot reappears.

**FEAT-03.SPEC-002-AC-06:** Given a hold cannot be created due to a processing error, when Riley attempts to begin checkout, then Riley sees a retry prompt and is not advanced into the payment step.

**FEAT-03.SPEC-002-AC-07:** Given Riley is rescheduling an existing booking and picks a new free time, when Riley confirms the new time, then a Slot Hold is created on the new time using the same validation as a new booking, without treating the booking's own current time as a conflict.

**FEAT-03.SPEC-002-AC-08:** Given a Slot Hold already exists on a candidate slot (checkout or Pro-created), when another client attempts to hold the same slot, then the attempt is refused per FEAT-03.SPEC-005 rather than creating a second, overlapping hold.

**FEAT-03.SPEC-002-AC-09:** Given Riley's checkout attempt is still in flight after tapping to begin payment, when Riley taps the same control again before the first attempt resolves, then no duplicate hold-creation request is submitted.

**FEAT-03.SPEC-002-AC-10:** Given a Slot Hold is successfully created for Riley, when Riley completes the deposit payment (FEAT-07), then the hold is converted to a confirmed Booking rather than expiring.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

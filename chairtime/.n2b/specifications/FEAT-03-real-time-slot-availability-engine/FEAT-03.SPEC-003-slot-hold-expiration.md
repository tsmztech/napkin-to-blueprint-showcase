---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-003
spec_name: Slot Hold Expiration
spec_slug: slot-hold-expiration
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 8
---

# Automation Spec: Slot Hold Expiration

## Overview

**Name:** Slot Hold Expiration
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** Automatically expires and deletes a checkout Slot Hold that reaches its fixed timeout without completed payment, returning the slot to public availability.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Detecting when an Active checkout Slot Hold (created by FEAT-03.SPEC-002) reaches its fixed timeout without a completed payment
- Transitioning that hold to Expired and deleting it, per the Entity-Lifecycle Coverage Matrix's Delete/Archive row
- Notifying the client, in-flow, that their held time has expired
- Applying the contention resolution rule (FEAT-03.SPEC-005) when a client's payment and the timeout race

**Non-Goals:**
- Expiring the Pro-created deposit-request hold class -- owned by FEAT-03.SPEC-007, whose longer, variable expiry (up to platform parameter: `deposit-request-hold-max-hours` or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment) is a distinct rule from this spec's fixed few-minute checkout timeout
- Creating the hold in the first place -- owned by FEAT-03.SPEC-002
- Converting a hold to a confirmed Booking -- a cross-feature outcome owned by FEAT-07 when payment completes before the timeout is reached
- Historical retention of expired holds -- excluded per the Entity-Lifecycle Coverage Matrix's explicit non-goal: "this is a transient computation artifact with a lifetime of minutes... never a historical record"

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A checkout Slot Hold's expiry timestamp is reached | FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Fires the moment the hold's expiry timestamp passes with the hold still Active (payment not completed) | The hold's service, start time, duration, owning checkout reference |

## Processing Logic

1. Identify each Active checkout Slot Hold whose expiry timestamp has passed.
2. Confirm the hold has not already transitioned to Consumed by a completed payment (FEAT-07) -- if it has, take no action (the race is resolved in payment's favor per Step 4 below).
3. Transition the hold's state to Expired.
4. Delete the hold record, per the Entity-Lifecycle Coverage Matrix (no restore path; a released hold simply becomes an ordinary open slot again).
5. Signal the owning checkout step (if the client is still on the payment screen) that the hold has expired.
6. The freed time reappears the next time FEAT-03.SPEC-001 computes the open-slot list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold expired and released | Timeout reached with no completed payment | Slot Hold transitioned to Expired then deleted | Client, if still on the payment screen, sees a plain message that their held time expired and is returned to a refreshed live slot list (never a payment error) | FEAT-05, FEAT-07, FEAT-03.SPEC-001 |
| Race won by payment | Payment completes at effectively the same moment the timeout is reached | Hold instead transitioned to Consumed by FEAT-07; this automation takes no further action | Client sees their booking confirmed, not an expiry message | FEAT-07 |
| Expiration failure | The expiration processing itself cannot complete (e.g., a processing error) | None -- the hold remains Active until retried | No client-facing message; this is a non-blocking internal failure that is retried | FEAT-03.SPEC-001 (a lingering stale hold would otherwise incorrectly withhold the slot) |

## Data Model

**Reads:** Slot Hold (state, expiry timestamp, owning checkout reference).
**Creates:** None.
**Updates:** Slot Hold -- state transitioned to Expired.
**Deletes:** Slot Hold -- the expired record is removed, per the Entity-Lifecycle Coverage Matrix.

## Business Rules

- The checkout hold's timeout is fixed and short: platform parameter: `checkout-hold-timeout-minutes` (same marker as defined in FEAT-03.SPEC-002 -- reused verbatim).
- The first committed action wins a race between an expiring hold and a completing payment (XBR-01, FEAT-03.SPEC-005): if payment completes before this automation processes the expiry, the hold is Consumed, not Expired.
- An expired hold is deleted, not archived -- there is no restore path and no retention requirement, since it is a transient computation artifact (Entity-Lifecycle Coverage Matrix).
- Expiration is a background process; it is never blocked by, or blocking to, any client's screen state.

## Edge Cases

- **Payment completes in the same instant the hold's timeout elapses** -- The completed-payment transition (to Consumed) takes precedence; this automation detects the already-Consumed state in Step 2 and takes no action, so the client is never shown an expiry message for a booking that actually succeeded.
- **Client's device is offline when their hold expires** -- The hold still expires server-side on schedule; the client sees the expiry message on their next successful interaction (e.g., attempting to submit payment), never a silently-accepted payment against an expired hold.
- **Concurrent trigger firing (two holds for different clients expire at the same moment)** -- Each hold's expiration is processed independently; there is no shared state between unrelated holds, so no contention exists between them.
- **Trigger fires while a previous expiration run for the same hold is still in flight** -- Expiration is idempotent: a hold already transitioned to Expired and deleted is a no-op if the expiration logic is invoked again for it.
- **Expiration processing itself fails** -- The hold remains Active and is retried; it is never left in an ambiguous state, and FEAT-03.SPEC-001's computation continues to correctly treat it as held until the retry succeeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Triggered by (inbound) | The hold this spec expires was created there, with the expiry timestamp it set |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The freed slot reappears in the next computation |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the race outcome between an expiring hold and a completing payment |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout), FEAT-05.SPEC-002 (Slot Selection) -- within FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Expired hold returns the client to the live slot list with an explanatory message |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) | Expired hold returns the client to the live slot list with an explanatory message; a completed payment instead converts the hold |

## Analytics and Success Signals

- **slot_hold_expired** (service_id, hold_type: checkout) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_expired_race_lost_to_payment** (service_id) -- N/A -- no Stage 2 metric measures this internal race outcome specifically; retained so the correctness of the first-committed-wins rule is observable
- **slot_hold_expiration_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence" (a failed expiration must never leave a stale hold silently blocking a slot -- this event measures how often the retry path is exercised)

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Riley's Slot Hold has reached its fixed checkout timeout without completed payment, when the expiration automation runs, then the hold is expired and deleted, and the slot reappears in the next computed list.

**FEAT-03.SPEC-003-AC-02:** Given Riley is still on the payment screen when the hold expires, when expiration completes, then Riley sees a plain message that the held time expired and is returned to a refreshed live slot list, never a payment error.

**FEAT-03.SPEC-003-AC-03:** Given Riley's payment completes at effectively the same moment the hold's timeout is reached, when both processes evaluate, then the hold is transitioned to Consumed by the completed payment, and the expiration automation takes no action.

**FEAT-03.SPEC-003-AC-04:** Given Riley's device loses connection just before the hold expires, when Riley next interacts with the payment screen after reconnecting, then Riley sees the expiry message rather than an ambiguous or silently-accepted payment attempt.

**FEAT-03.SPEC-003-AC-05:** Given two different clients' holds expire at the same moment, when the expiration automation processes both, then each is expired independently with no interference between them.

**FEAT-03.SPEC-003-AC-06:** Given a hold has already been expired and deleted, when the expiration logic is invoked again for the same hold, then nothing changes (no error, no duplicate action).

**FEAT-03.SPEC-003-AC-07:** Given the expiration processing itself encounters a processing error, when the hold's timeout has passed, then the hold remains Active and is retried, and FEAT-03.SPEC-001 continues to correctly exclude it as held until the retry succeeds.

**FEAT-03.SPEC-003-AC-08:** Given a checkout Slot Hold expires, when Riley next requests the same service's slot list, then the freed time appears as open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

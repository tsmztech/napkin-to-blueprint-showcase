---
document_type: spec
spec_type: automation
spec_id: FEAT-30.SPEC-010
spec_name: Pro-Created Booking & Deposit Request Hold
spec_slug: pro-created-booking-deposit-request-hold
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Pro-Created Booking & Deposit Request Hold

## Overview

**Name:** Pro-Created Booking & Deposit Request Hold
**ID:** FEAT-30.SPEC-010
**Type:** Automation
**Purpose:** Creates the pending Booking from a Pro-entered service, time, and client, invokes the slot hold that reserves the time, confirms the booking on payment, and reflects the hold's expiry if the deposit is never paid.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Creating the Booking record in Pending Payment state, with source = "Pro booked-in", from the service, time, and client Talia selects on FEAT-30.SPEC-004
- Invoking the slot-hold mechanism that reserves the candidate time for this booking
- Issuing the deposit request (link or on-screen code) once the Booking and its hold are created
- Confirming the Booking (Pending Payment -> Confirmed) when the client pays, exactly as a client-initiated booking confirms
- Relying on the Booking's transition to Expired (unpaid) when its hold lapses unpaid: FEAT-03.SPEC-007 is the sole writer of that transition (XBR-02), and this automation only reads the resulting state
- Triggering the personal-calendar write when the Booking is confirmed (FEAT-04.SPEC-005)

**Non-Goals:**
- Computing the hold's expiry timestamp, re-validating the candidate slot against live availability, or actually expiring the hold record -- owned entirely by FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration), which this automation triggers and whose outcomes it reflects onto the Booking it created; this automation never duplicates that spec's timing mechanics
- Selecting the service, time, and client -- owned by FEAT-30.SPEC-004 (Book Client In), the triggering screen
- Capturing the client's deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this automation only reacts to that capability's confirmation to transition the Booking
- Writing the Booking -> Expired (unpaid) transition -- owned solely by FEAT-03.SPEC-007 (XBR-02 authority: FEAT-03); this automation never performs, repeats, or races that write
- Composing or delivering the deposit-request content -- owned by FEAT-30.SPEC-013 (Deposit Request & Expiry Notice); this automation triggers it but does not define its content
- Composing or delivering the Pro's expiry notice -- FEAT-03.SPEC-007 is the trigger and FEAT-08.SPEC-006 (Pro Attention Alert) is the content owner

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia saves a new Pro-created booking | FEAT-30.SPEC-004 (Book Client In) | Fires when Talia confirms the service, time, and existing-or-new client for booking someone in | Service ID, chosen start time and duration, Client reference (existing or newly entered) |
| FEAT-03.SPEC-007 marks the Booking Expired (unpaid) | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Fires after that spec's expiration path has determined the deposit was never paid in time and has itself written Booking.state = Expired (unpaid) | The owning Booking reference (already Expired (unpaid)) |
| The client pays the deposit request | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires when that spec confirms the deposit payment against this Booking's hold | The owning Booking reference, payment outcome |

## Processing Logic

1. Receive the candidate Service ID, start time, duration, and Client reference from FEAT-30.SPEC-004's save action.
2. Create the Booking record: service, start_time, duration, client, price_agreed and deposit_amount (computed from the Service's rule, per XBR-05), policy_version (the Pro's current Cancellation Policy), state = Pending Payment, source = "Pro booked-in".
3. Invoke FEAT-03.SPEC-007 to create the owning slot hold against this new Booking, applying the Pro-only notice/horizon exception (FEAT-30.SPEC-006) to the candidate slot's re-validation.
4. If FEAT-03.SPEC-007 reports the candidate slot is contested (already held, booked, blocked, or busy), do not create the Booking; return the "just taken" outcome to FEAT-30.SPEC-004 for its inline recovery message.
5. If the hold is created successfully, issue the deposit request through FEAT-30.SPEC-013, by the delivery choice Talia selected on FEAT-30.SPEC-004 (link by text/email, or an on-screen code).
6. **On payment path:** When FEAT-07 confirms the deposit payment against this Booking's hold, transition Booking.state from Pending Payment to Confirmed, exactly as a client-initiated booking confirms.
7. **On expiry path:** When FEAT-03.SPEC-007 reports it has marked this Booking Expired (unpaid), take no state-writing step: FEAT-03.SPEC-007 is the sole writer of that transition. Read the resulting state so FEAT-30.SPEC-004's confirmation and FEAT-12 show the booking as expired. The Pro's expiry notice is triggered by FEAT-03.SPEC-007 with its content owned by FEAT-08.SPEC-006; this automation sends nothing.
8. **After the payment path (step 6):** Trigger FEAT-04.SPEC-005 (Booking-to-Calendar Sync) for the newly Confirmed Booking so it is written to Talia's connected personal calendar (XBR-13).
9. Return the created Booking's outcome to FEAT-30.SPEC-004 for its confirmation feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Booking created, hold placed, deposit request issued | The candidate slot passes re-validation (with the Pro-only exception) and FEAT-03.SPEC-007 places the hold | New Booking created in Pending Payment, source "Pro booked-in" | Talia sees the booking on her dashboard awaiting payment; the client receives the deposit request (FEAT-30.SPEC-013) | FEAT-30.SPEC-004, FEAT-03.SPEC-007, FEAT-30.SPEC-013 |
| Candidate slot contested at creation | FEAT-03.SPEC-007 reports the candidate time is already held, booked, blocked, or busy | No Booking created | Talia sees the plain "just taken" message on FEAT-30.SPEC-004 and re-selects a time | FEAT-30.SPEC-004, FEAT-03.SPEC-007 |
| Deposit paid before expiry | The client completes payment while the hold is still Active | Booking.state -> Confirmed; calendar write triggered | Talia and the client both see the booking confirmed | FEAT-07, FEAT-12, FEAT-04.SPEC-005 |
| Hold expired -- deposit never paid | FEAT-03.SPEC-007's expiration path has lapsed the hold unpaid and written Booking.state = Expired (unpaid) | None by this automation -- the state is written solely by FEAT-03.SPEC-007; this automation reads it | Talia is notified via her dashboard and a message (triggered by FEAT-03.SPEC-007, content owned by FEAT-08.SPEC-006); the client receives no further reminder for this booking | FEAT-03.SPEC-007, FEAT-08.SPEC-006 |
| Booking creation failure | The Booking record cannot be written (e.g., a processing error) | No Booking created; no hold placed | Talia sees a retry prompt on FEAT-30.SPEC-004; no deposit-request link or code is issued | FEAT-30.SPEC-004 |

## Data Model

**Reads:** Service (price, duration, deposit_rule); Cancellation Policy (current version); Client (existing lookup) or none (new client, created inline by FEAT-30.SPEC-004).
**Creates:** Booking -- service, start_time, duration, client, price_agreed, deposit_amount, policy_version, state (Pending Payment), source ("Pro booked-in").
**Updates:** Booking.state -- Pending Payment -> Confirmed (on payment) only. The -> Expired (unpaid) transition on unpaid hold expiry is written solely by FEAT-03.SPEC-007 and is only read here.
**Deletes:** None -- an expired Pro-created booking is retained as history (SC-22), never deleted; only its owning slot hold (a distinct, transient record owned by FEAT-03.SPEC-007) is deleted on expiry.

## Business Rules

- XBR-05: the deposit amount is computed once, exactly, from the Service's rule in the Pro's account currency, and cannot be altered by Talia, identical to any client-initiated booking.
- XBR-02: the slot hold this automation invokes reserves the candidate time for up to platform parameter: `deposit-request-hold-max-hours` or until platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, whichever comes first -- the value is stated by FEAT-30.SPEC-006 and enforced entirely by FEAT-03.SPEC-007; this automation never computes or enforces the expiry itself.
- The Pro-only notice/horizon exception (FEAT-30.SPEC-006) applies to the candidate slot's re-validation, since the triggering action is always a Pro-side booking; the duration+buffer fit rule is never exempted.
- A Booking this automation creates is never confirmed by anything other than FEAT-07's payment confirmation -- there is no "mark paid manually" path, consistent with the product's correctness-over-convenience stance (SC-21).
- FEAT-03.SPEC-007 is the sole writer of Booking -> Expired (unpaid) (XBR-02); this automation never sets that state, so two writers can never race on the same Booking.
- An expired Pro-created booking is marked Expired (unpaid) (by FEAT-03.SPEC-007), never silently deleted -- the record that a booking was attempted and lapsed is preserved, distinct from the transient slot hold itself, which FEAT-03.SPEC-007 deletes.

## Edge Cases

- **The candidate slot is contested by a client-side booking that completes payment first** -- FEAT-03.SPEC-007's contention resolution applies (first committed wins, per XBR-01); no Booking is created for Talia's attempt, and she sees the "just taken" message and re-selects a time.
- **Talia cancels the Pro-created booking herself before the deposit is paid or the hold expires** -- The cancellation (via FEAT-30.SPEC-007, the single-cancellation commit) transitions the Booking and, through FEAT-03.SPEC-007, deletes the hold directly; no expiration notification fires for a Pro-initiated cancellation.
- **The appointment is scheduled less than platform parameter: `deposit-request-hold-appointment-cutoff-hours` away at the moment of creation** -- The Pro-only notice exception permits creating the booking itself; the resulting hold's own window is correspondingly short, per FEAT-03.SPEC-007's own handling of this case.
- **The client pays and the hold expires at effectively the same instant** -- FEAT-03.SPEC-007's own precedence rule applies: the completed payment takes priority, and this automation confirms the Booking rather than expiring it.
- **Concurrent trigger firing (Talia books two different clients into two different, non-overlapping slots at effectively the same time)** -- Each booking-creation attempt is processed independently against its own distinct candidate slot; no interference occurs.
- **Trigger fires while a previous booking-creation attempt for the same booking-in action is still in flight** -- FEAT-30.SPEC-004's save control is disabled during submission, preventing a duplicate Booking-and-hold creation request for the same client and slot.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-004 (Book Client In) | Triggered by (inbound) / Affects (outbound) | Save triggers Booking creation; the outcome (created, contested, or failed) is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | States the notice/horizon exception and the hold-window values this automation invokes |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggers (outbound) / Triggered by (inbound) | This automation invokes hold creation on save; that spec is the sole writer of Booking -> Expired (unpaid) and this automation only relies on the result |
| FEAT-07 (Deposit Payment at Booking) | Triggered by (inbound) | Payment confirmation against this Booking's hold transitions it to Confirmed |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | Triggers (outbound) | A created hold issues the deposit request |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Owns the Pro's expiry-notice content, triggered by FEAT-03.SPEC-007 |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | A Pro-created Booking confirmed by payment is written to Talia's personal calendar |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Talia sees the booking's awaiting-payment, confirmed, or expired status on her schedule |

## Analytics and Success Signals

- **pro_booking_created** (service_id) -- supports success-metrics.md: "Pro Change Correctness"
- **deposit_request_paid** (time_to_pay) -- supports success-metrics.md: "Pro Change Correctness" (the target's "at least 70% of deposit requests the pro sends when rebooking at the chair are paid before the hold expires" is measured against deposit_request_expired, emitted below when this automation reads the Expired (unpaid) state FEAT-03.SPEC-007 writes)
- **deposit_request_expired** () -- supports success-metrics.md: "Pro Change Correctness"
- **pro_booking_creation_failed** (reason category) -- N/A -- no Stage 2 metric measures booking-creation failures directly; retained so a failed save is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-010-AC-01:** Given Talia saves a new booking for an existing client at a genuinely free time, when this automation runs, then a Booking is created in Pending Payment with source "Pro booked-in", and a slot hold is placed via FEAT-03.SPEC-007.

**FEAT-30.SPEC-010-AC-02:** Given the candidate slot is contested by another booking that lands first, when the hold-creation attempt runs, then no Booking is created and Talia sees the "just taken" message on FEAT-30.SPEC-004.

**FEAT-30.SPEC-010-AC-03:** Given a Booking and its hold are created successfully, when the save completes, then the deposit request is issued through FEAT-30.SPEC-013 by Talia's chosen delivery method.

**FEAT-30.SPEC-010-AC-04:** Given the client completes the deposit payment while the hold is Active, when FEAT-07 confirms the payment, then this Booking's state transitions from Pending Payment to Confirmed.

**FEAT-30.SPEC-010-AC-05:** Given the hold's computed expiry passes with the deposit never paid, when FEAT-03.SPEC-007's expiration path fires, then FEAT-03.SPEC-007 (and no other spec) sets this Booking's state to Expired (unpaid), this automation performs no state write, and Talia is notified via her dashboard and a message with content owned by FEAT-08.SPEC-006.

**FEAT-30.SPEC-010-AC-06:** Given Talia books a client in for a time inside her own minimum_booking_notice, when the candidate is re-validated, then it is not excluded on notice grounds, per the Pro-only exception (FEAT-30.SPEC-006).

**FEAT-30.SPEC-010-AC-07:** Given Talia cancels a Pro-created booking herself before its hold expires, when the cancellation completes via FEAT-30.SPEC-007, then the hold is deleted directly and no expiration notification fires.

**FEAT-30.SPEC-010-AC-08:** Given the deposit payment and the hold's expiry occur at effectively the same moment, when both are evaluated, then the completed payment takes precedence and the Booking is Confirmed, not Expired.

**FEAT-30.SPEC-010-AC-09:** Given the Booking record cannot be created due to a processing error, when Talia attempts to save the booking-in flow, then she sees a retry prompt and no deposit-request link or code is issued.

**FEAT-30.SPEC-010-AC-10:** Given Talia books two different clients into two different, non-overlapping slots at effectively the same time, when both booking-creation attempts run, then each succeeds independently with no interference.

**FEAT-30.SPEC-010-AC-11:** Given a booking-creation attempt is already in flight for a booking-in action, when Talia's save control is tapped again before it resolves, then no duplicate Booking-and-hold is created, since the control is disabled during submission.

**FEAT-30.SPEC-010-AC-12:** Given a Pro-created booking expires unpaid, when Riley next requests the same service's slot list, then the freed time appears as open, per FEAT-03.SPEC-007.

**FEAT-30.SPEC-010-AC-13:** Given a Pro-created booking's deposit is computed from the Service's rule, when the Booking is created, then the deposit_amount matches exactly what FEAT-07 would compute for a client-initiated booking of the same service.

**FEAT-30.SPEC-010-AC-14:** Given a Pro-created booking's deposit is paid and the Booking transitions to Confirmed, when this automation completes the payment path, then FEAT-04.SPEC-005 is triggered once for that Booking so it appears on Talia's connected personal calendar.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

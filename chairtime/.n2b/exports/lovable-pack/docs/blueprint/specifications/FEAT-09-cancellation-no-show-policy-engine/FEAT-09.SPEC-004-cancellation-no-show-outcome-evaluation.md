---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-004
spec_name: Cancellation & No-Show Outcome Evaluation
spec_slug: cancellation-no-show-outcome-evaluation
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Automation Spec: Cancellation & No-Show Outcome Evaluation

## Overview

**Name:** Cancellation & No-Show Outcome Evaluation
**ID:** FEAT-09.SPEC-004
**Type:** Automation
**Purpose:** On every cancellation, reschedule, or no-show marking, evaluates the booking's bound policy version against FEAT-09.SPEC-003's rule set and writes the resulting refund-due or forfeiture-due outcome to the Deposit Transaction, instantaneously and with no user-visible loading state.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Evaluating every cancellation, reschedule, or no-show-marking event against FEAT-09.SPEC-003's rule table
- Writing the determined outcome (outcome_reason and its timestamp) to the Deposit Transaction
- Handing a refund-due outcome to FEAT-09.SPEC-005 for execution
- Handing a forfeiture-due outcome to FEAT-11 for application (this automation flags it; it does not itself set the terminal Forfeited status)
- Writing the forfeiture-due determination on the original deposit at the moment FEAT-10.SPEC-004 commits a late reschedule -- the same commit that flags the new booking as requiring its own fresh deposit (that flag itself is set by FEAT-10.SPEC-004, not by this automation)

**Non-Goals:**
- Defining which outcome applies to which trigger -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this automation applies that spec's table, it does not define it
- Computing the booking's cutoff time or reading the bound policy version's wording -- owned by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this automation consumes that computation
- Executing the actual refund against the payment-processing capability -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund); this automation only determines that a refund is due and hands it off
- Setting the Deposit Transaction's terminal Forfeited status or the Booking's No-Show state -- excluded per the Entity-Lifecycle Coverage Matrix: this automation initiates (flags) the forfeiture outcome only; the transition itself is applied by FEAT-11, per the Key Capability's own wording

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client cancels a booking | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client-initiated cancellation is recorded against a Confirmed or Awaiting Outcome booking | Booking reference, cancellation timestamp, initiator = Client, the booking's bound policy_version and start_time, the linked Deposit Transaction's current status |
| Client reschedules a booking | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client-initiated reschedule to a new confirmed time is recorded | Booking reference, reschedule timestamp, initiator = Client, the booking's bound policy_version and start_time, the new appointment's time, the linked Deposit Transaction's current status |
| Booking marked no-show | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Fires when the Pro's no-show marking is recorded against an Awaiting Outcome booking | Booking reference, marking timestamp, initiator = Pro, the linked Deposit Transaction's current status |
| Pro cancels a booking | FEAT-30 (Pro Booking Management) | Fires when a Pro-initiated cancellation is recorded against a Confirmed or Awaiting Outcome booking | Booking reference, cancellation timestamp, initiator = Pro, the linked Deposit Transaction's current status |
| Pro reschedules a booking | FEAT-30 (Pro Booking Management) | Fires when a Pro-initiated reschedule to a new confirmed time is recorded | Booking reference, reschedule timestamp, initiator = Pro, the new appointment's time, the linked Deposit Transaction's current status |

## Processing Logic

1. Receive the triggering event's data: the Booking reference, the action type (cancel or reschedule), the initiator (Client or Pro), and the event timestamp.
2. Read the Deposit Transaction linked to the Booking. If its status is not Captured (already Refunded, Forfeited, Refund in Progress, or Disputed), stop and route to the already-resolved outcome below -- no second determination is ever written.
3. For a cancellation or a reschedule (not a no-show marking), read the Booking's bound policy_version and compute its cutoff time via FEAT-09.SPEC-002 (start_time minus the bound version's window_hours).
4. Compare the initiator, action type, and (where timing-dependent) the event timestamp against the cutoff, applying FEAT-09.SPEC-003's rule table in its stated precedence order (no-show and Pro-initiated rules override timing comparisons).
5. Determine the outcome: full refund due, deposit-kept (forfeiture due), or deposit-carries-over (reschedule outside the window, no outcome change).
6. Write the determined outcome_reason and its timestamp to the Deposit Transaction (Captured status is not changed by this step -- see Outcome Definitions for what changes next).
7. If the outcome is a refund due, hand the Deposit Transaction to FEAT-09.SPEC-005 to execute the refund. Separately, for a client-initiated or automatic cancellation (not a reschedule or no-show marking), if the Booking has a Balance Payment in Succeeded state, also hand the Booking and Balance Payment references to FEAT-09.SPEC-005, which invokes FEAT-22.SPEC-005 to refund the paid balance in full whatever the deposit outcome (XBR-23; a balance is never forfeited). A Pro-initiated cancellation's balance refund is invoked by FEAT-30.SPEC-007 directly, so this automation does not duplicate it.
8. If the outcome is a forfeiture due (deposit kept, from a cancellation, no-show, or a late reschedule), hand the forfeiture flag to FEAT-11 for application to the terminal Forfeited state.
9. If the outcome is the late-reschedule compound case (Rule 6), this evaluation runs as part of the same commit sequence FEAT-10.SPEC-004 already executed the moment Riley tapped "Confirm Reschedule" -- the single confirmation gate for the whole compound outcome (product-features.md, FEAT-10 Alternate Flow: she "sees plainly" both halves "before confirming" and "can back out with nothing changed" if she declines that one tap). The new Booking's fresh-deposit requirement was already flagged by FEAT-10.SPEC-004 at that same commit, not by this step; this automation's own role here is solely to write the forfeiture-due outcome_reason to the *original* Deposit Transaction, finalizing the half of the compound outcome Riley was shown before she confirmed. If Riley instead backs out before that tap (via "Choose a different time" or "Cancel instead" on FEAT-10.SPEC-003), FEAT-10.SPEC-004 never commits anything and this automation never fires for that attempt.
10. If the outcome is deposit-carries-over (reschedule outside the window), no Deposit Transaction change is made beyond re-associating it with the new appointment time on the same, continuing Booking record (FEAT-10.SPEC-004 updates that record in place; no new Booking is created for this outcome) -- no new charge, no outcome_reason change beyond noting the carry-over.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Refund due | Client cancels outside the window, or Pro cancels (any timing) | Deposit Transaction.outcome_reason set to the refund determination and timestamped | No separate loading state; the outcome is included in the relevant confirmation message (FEAT-08), not a standalone one | FEAT-09.SPEC-005 (executes the refund) |
| Forfeiture due (client cancel, inside window) | Client cancels inside the window | Deposit Transaction.outcome_reason set to the forfeiture determination and timestamped | Riley is shown, before confirming the cancellation, exactly what will happen to her deposit (FEAT-10); Talia sees the kept-deposit outcome on her dashboard once resolved | FEAT-11 (applies the terminal Forfeited state) |
| Forfeiture due (no-show) | Booking marked no-show | Deposit Transaction.outcome_reason set to the forfeiture determination and timestamped | Talia sees the kept-deposit outcome on the no-show prompt; Riley's deposit status reflects the outcome in her own view | FEAT-11 (applies the terminal Forfeited state) |
| Deposit carries over (reschedule, outside window) | Client or Pro reschedules outside the window | Deposit Transaction re-associated with the new appointment's Booking record; no status or outcome_reason change | Riley sees no new charge and no change to her deposit status; the appointment time updates | FEAT-10 (or FEAT-30 for a Pro-made reschedule) |
| Late-reschedule compound outcome | Client reschedules inside the window | Original Deposit Transaction's outcome_reason set to the forfeiture determination (as a late cancellation), written as part of the single atomic commit FEAT-10.SPEC-004 performs the moment Riley taps "Confirm Reschedule"; the new Booking's fresh-deposit flag is set by that same FEAT-10.SPEC-004 commit, not by this automation | Riley is shown, before that one confirm tap, that her original deposit will be kept and the new time will need its own deposit; declining before the tap (choosing a different time, or cancelling instead) leaves everything -- including this outcome -- untouched, since nothing has been evaluated yet | FEAT-11 (applies Forfeited to the original), FEAT-10 (shows the preview and performs the commit), FEAT-07 (collects the new deposit after the commit) |
| Already resolved -- no second determination | The linked Deposit Transaction's status is already Refunded, Forfeited, Refund in Progress, or Disputed when a trigger fires | None | No outcome-specific feedback is produced by this automation; the triggering spec (FEAT-10, FEAT-11, or FEAT-30) surfaces its own already-resolved messaging, since this booking-state conflict is that spec's concern, not this automation's | FEAT-10, FEAT-11, FEAT-30 (whichever triggered) |
| Evaluation failure (processing error) | The evaluation step itself cannot complete (e.g., the policy version or cutoff cannot be read) | No outcome_reason is written -- the Deposit Transaction remains Captured, unresolved | The triggering action (cancellation, reschedule, or no-show mark) is not blocked from recording; the outcome is retried automatically and, if retries are exhausted, flagged on Talia's dashboard as needing attention, never silently dropped | FEAT-12 (attention list) |

## Data Model

**Reads:** Booking -- start_time, policy_version, cancellation/reschedule/no-show timestamp, and initiator. Cancellation Policy -- the bound version's window_hours, via FEAT-09.SPEC-002. Deposit Transaction -- current status (must be Captured to proceed).
**Creates:** None.
**Updates:** Deposit Transaction -- outcome_reason and an outcome timestamp for a refund-due or forfeiture-due determination; for the deposit-carries-over outcome (reschedule outside the window), the same Deposit Transaction record is re-associated with the new appointment time on the same, continuing Booking record -- per FEAT-10.SPEC-004's resolution of the Entity-Lifecycle Coverage Matrix, an outside-window reschedule updates the existing Booking in place and never creates a new record, so this re-association carries no status or outcome_reason change of its own. This automation never itself sets status to Refunded, Refund in Progress, or Forfeited; those transitions belong to FEAT-09.SPEC-005 (refund path) and FEAT-11 (forfeiture path) respectively.
**Deletes:** None -- consistent with the dependency map's Deposit Transaction lifecycle, which has no delete path (SC-22).

## Business Rules

- XBR-09: this automation applies FEAT-09.SPEC-003's rule table exactly, with no independent interpretation of timing or intent.
- XBR-08: the cutoff and bound version used for every comparison are always the ones the specific booking acknowledged at booking time (FEAT-09.SPEC-002), never the current policy.
- Evaluation is instantaneous from the point of view of both parties -- neither Talia nor Riley ever sees a loading state for this step (product-features.md, States field); only the refund itself, when it cannot complete immediately, surfaces an "in progress" state (FEAT-09.SPEC-005/FEAT-09.SPEC-006).
- A deposit can be evaluated to a terminal outcome only once (dependency map, Deposit Transaction Contention): once outcome_reason is set and handed to FEAT-09.SPEC-005 or FEAT-11, this automation never re-fires for the same triggering event.
- The forfeiture flag this automation raises is an initiation, not the terminal state: FEAT-11 (not this automation) performs the actual Captured -> Forfeited transition, per the Entity-Lifecycle Coverage Matrix.
- **Single confirmation gate for the late-reschedule compound outcome:** the "Confirm Reschedule" tap on FEAT-10.SPEC-003 is the one point of commitment for the whole compound outcome (product-features.md, FEAT-10 Alternate Flow). Backing out before that tap -- via "Choose a different time" or "Cancel instead" -- means FEAT-10.SPEC-004 never commits and this automation never fires; nothing about the original booking or its Deposit Transaction changes. Once that tap succeeds, FEAT-10.SPEC-004's atomic write (original Booking -> Rescheduled, new Booking created and flagged for a fresh deposit) and this automation's forfeiture-due determination on the original Deposit Transaction happen as one committed sequence -- there is no second confirmation gate, and what later happens to the new Booking's own (unpaid) deposit never reopens or reverses that already-written determination.

## Edge Cases

- **A cancellation and a Pro-initiated reschedule are recorded for the same booking at effectively the same time (concurrent trigger firing)** -- Per the dependency map's Booking Contention rule (reject-with-refresh, first committed state transition wins), only one of the two triggering actions can have actually recorded against the Booking in the first place; this automation only ever receives the one event whose triggering action won that race, so no two evaluations ever run against the same booking concurrently.
- **A second trigger fires while this automation's evaluation for the same booking is still in flight (trigger fires while a previous run is in flight)** -- The Deposit Transaction's status check (step 2) means the second run finds the first run's outcome_reason already being written or written; the second run's own triggering action would only have been possible if the first action's Booking-state transition had not yet committed (per Booking Contention, reject-with-refresh), so a second determination against the same still-Captured transaction from a genuinely distinct action cannot occur -- the two triggers described above (cancel vs. Pro reschedule) are the same scenario, not a separate one.
- **The cutoff computation itself fails (the bound policy version cannot be read)** -- Routed to the Evaluation failure outcome: the triggering action still records, the evaluation retries automatically, and Talia's dashboard is flagged if retries are exhausted.
- **A client reschedules to a time, then reschedules again before the first new time arrives** -- Each reschedule is its own triggering event, evaluated independently against its own appointment's freshly computed cutoff (per FEAT-09.SPEC-003's edge case for repeated reschedules).
- **A no-show marking arrives for a booking whose Deposit Transaction was already set to Refund in Progress by an earlier automatic determination** -- Not possible under normal use: XBR-12 requires the appointment's start time to have passed before a no-show mark, and a booking already resolved to a refund outcome would already be in a terminal or in-progress state that FEAT-11's own eligibility check (FEAT-11.SPEC-004) refuses to re-open; if it is somehow attempted, this automation's status check at step 2 still refuses a second determination.
- **Riley backs out of a late reschedule before tapping "Confirm Reschedule" (via "Choose a different time" or "Cancel instead" on FEAT-10.SPEC-003)** -- There is nothing for this automation to evaluate: FEAT-10.SPEC-004 never commits the compound write, so this automation never fires for that attempt. The original Booking and its Deposit Transaction remain exactly as they were, consistent with the single confirmation gate covering the whole compound outcome (product-features.md, FEAT-10 Alternate Flow: "can back out with nothing changed").
- **Riley taps "Confirm Reschedule" on a late reschedule, but the new Booking that commit creates is never actually paid** -- The original Deposit Transaction's forfeiture-due outcome was already written by this automation as part of the same atomic commit FEAT-10.SPEC-004 performed at the "Confirm Reschedule" tap -- that tap, not the new deposit's payment, was the single confirmation gate for the whole compound outcome. The new Booking's later non-payment and expiration (governed by FEAT-10.SPEC-004/FEAT-03's ordinary hold-and-expiration handling, XBR-02) is a separate, subsequent fact about a different Booking record; it never reopens or reverses this automation's already-written determination on the original.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (outbound) | Reads the bound policy version and computed cutoff for every timing-dependent determination |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (outbound) | Applies the rule table this spec defines |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) | Triggers (outbound) | A refund-due outcome hands off to this spec to execute; a client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment also hands off the balance refund trigger, which that spec passes to FEAT-22.SPEC-005 (XBR-23) |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | FEAT-10.SPEC-004's committed cancellation or reschedule fires this automation; the outcome preview Riley sees before confirming (FEAT-10.SPEC-001/FEAT-10.SPEC-003) is read directly from FEAT-09.SPEC-003's rule table, not from this automation, which runs only after that commit |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) / Affects (outbound) | A recorded no-show marking fires this automation; the forfeiture flag this automation raises is applied by FEAT-11 |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A recorded Pro-initiated cancellation or reschedule fires this automation |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | An evaluation failure that exhausts retries is flagged on the attention list |

## Analytics and Success Signals

- **cancellation_within_window_flagged** (booking reference, days/hours before appointment) -- supports success-metrics.md: "Policy Clarity at Booking"
- **deposit_refund_triggered** (trigger type: client-cancel-outside / pro-cancel / pro-reschedule / client-reschedule-outside) -- supports success-metrics.md: "Automatic Refund Correctness"
- **late_reschedule_treated_as_cancellation** (booking reference) -- supports success-metrics.md: "Policy Clarity at Booking" (measures how often the compound outcome fires, which the disclosure-before-confirming step exists specifically to make unsurprising)
- **deposit_outcome_evaluation_failed** (trigger type, retry count) -- N/A -- no Stage 2 metric measures evaluation-failure frequency directly; retained so the "never silently dropped" guarantee (product-features.md, Error state) is observable

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given Riley cancels her booking outside the computed cutoff, when the cancellation is recorded, then this automation writes a refund-due outcome to the Deposit Transaction and hands it to FEAT-09.SPEC-005, with no loading state shown to Riley.

**FEAT-09.SPEC-004-AC-02:** Given Riley cancels her booking inside the computed cutoff, when the cancellation is recorded, then this automation writes a forfeiture-due outcome and hands it to FEAT-11, having already shown Riley the outcome before she confirmed (FEAT-10).

**FEAT-09.SPEC-004-AC-03:** Given Talia marks Riley's booking as no-show, when the marking is recorded, then this automation writes a forfeiture-due outcome regardless of how the marking time compares to the cutoff, and hands it to FEAT-11.

**FEAT-09.SPEC-004-AC-04:** Given Talia cancels Riley's booking, when the cancellation is recorded, then this automation writes a refund-due outcome and hands it to FEAT-09.SPEC-005, regardless of timing.

**FEAT-09.SPEC-004-AC-05:** Given Riley reschedules her booking outside the computed cutoff, when the reschedule is recorded, then this automation re-associates the existing Deposit Transaction with the new appointment with no outcome change and no new charge.

**FEAT-09.SPEC-004-AC-06:** Given Riley taps "Confirm Reschedule" on a chosen time inside the computed cutoff and FEAT-10.SPEC-004's commit succeeds, when the commit is recorded, then this automation writes a forfeiture-due outcome on the original deposit as part of that same commit -- the new appointment's fresh-deposit requirement was already flagged by FEAT-10.SPEC-004, and Riley had already seen both halves of the outcome before that one confirm tap.

**FEAT-09.SPEC-004-AC-07:** Given Talia reschedules Riley's booking, when the reschedule is recorded, then this automation re-associates the existing Deposit Transaction with the new time with no outcome change, regardless of timing.

**FEAT-09.SPEC-004-AC-08:** Given a booking's Deposit Transaction is already Refunded, when a second triggering event somehow arrives for it, then this automation writes no second outcome and produces no duplicate refund or forfeiture flag.

**FEAT-09.SPEC-004-AC-09:** Given the cutoff computation for a triggering event cannot complete due to a processing error, when evaluation is attempted, then the triggering action itself still records, the evaluation is retried automatically, and Talia's dashboard is flagged if retries are exhausted.

**FEAT-09.SPEC-004-AC-10:** Given two actions that could both apply to the same booking are attempted at effectively the same time, when the Booking's own contention rule resolves which one committed, then this automation evaluates only the one event whose action actually recorded, never both.

**FEAT-09.SPEC-004-AC-11:** Given Riley reschedules twice before her first new appointment time arrives, when each reschedule is recorded, then this automation evaluates each one independently against its own freshly computed cutoff.

**FEAT-09.SPEC-004-AC-12:** Given Riley taps "Confirm Reschedule" on the inside-window outcome and FEAT-10.SPEC-004's commit succeeds, when the new Booking that commit creates is later left unpaid and expires per FEAT-03's hold rules, then the original deposit's forfeiture-due outcome this automation wrote at that same commit still stands -- it is not reversed by the new Booking's later expiration, since the "Confirm Reschedule" tap itself, not the new deposit's payment, was the single confirmation gate for the whole compound outcome.

**FEAT-09.SPEC-004-AC-13:** Given a refund-due outcome is written for Riley's cancellation, when FEAT-09.SPEC-005 executes it, then the outcome the client eventually sees is included in her cancellation confirmation message (FEAT-08), never a separate standalone notice from this automation.

**FEAT-09.SPEC-004-AC-14:** Given a forfeiture-due outcome is written from a no-show marking, when FEAT-11 applies the terminal Forfeited state, then this automation's own record shows only the outcome_reason and timestamp it wrote -- the status transition itself is FEAT-11's action, not this automation's.

**FEAT-09.SPEC-004-AC-15:** Given Riley cancels a booking and its Balance Payment is in Succeeded state, when the cancellation is recorded, then this automation hands the balance refund trigger to FEAT-09.SPEC-005 for FEAT-22.SPEC-005 to refund in full, regardless of whether the deposit outcome is refund-due or forfeiture-due.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (client cancel, client reschedule, no-show mark, Pro cancel, Pro reschedule) | 5 |
| Outcome Paths | 7 (refund due, forfeiture due x2 trigger flavors, carries over, late-reschedule compound, already resolved, evaluation failure) | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |

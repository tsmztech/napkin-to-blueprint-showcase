---
document_type: spec
spec_type: automation
spec_id: FEAT-30.SPEC-009
spec_name: Goodwill Refund Commit
spec_slug: goodwill-refund-commit
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Goodwill Refund Commit

## Overview

**Name:** Goodwill Refund Commit
**ID:** FEAT-30.SPEC-009
**Type:** Automation
**Purpose:** Processes a confirmed goodwill refund against a booking's deposit -- a standalone Pro override, independent of the cancellation window and available until the booking completes.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Determining that a confirmed goodwill refund is due, governed by FEAT-30.SPEC-006's once-only and until-completion limits
- Requesting the refund through FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution)
- Writing the activity event and triggering the client notice this action causes
- Never changing Booking.state -- only Deposit Transaction.status

**Non-Goals:**
- Deciding whether a goodwill refund is warranted -- excluded per scope-boundaries.md (SC-17): the goodwill decision is entirely Talia's own judgment, exercised through FEAT-30.SPEC-003; this automation only processes a decision Talia has already confirmed
- Executing the refund request against the payment-processing capability -- owned by FEAT-30.SPEC-011; this automation determines the refund is due and hands off the request
- Evaluating a client cancellation's or no-show's own deposit outcome -- owned by FEAT-09; a goodwill refund is not a Pro cancellation and is never evaluated by FEAT-09's outcome-evaluation automation, per the Brief's own Analyst-Discovered rationale for this spec
- Marking or undoing a no-show mark -- owned by FEAT-11; a goodwill refund can follow either a no-show mark or an inside-window client cancellation, but this automation never itself marks or unmarks a no-show

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms a goodwill refund | FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Fires when Talia taps confirm, reached from a no-show prompt (FEAT-11), a dispute timeline (FEAT-16), or a booking row (FEAT-12) | Booking reference, optional private Pro reason |

## Processing Logic

1. Receive the Booking reference and any optional private Pro reason from FEAT-30.SPEC-003's confirm action.
2. Re-check eligibility against FEAT-30.SPEC-006 (ownership, Booking.state has not reached Completed, and Deposit Transaction.status is Captured or Forfeited) immediately before the write.
3. If eligibility fails, stop and return the specific denial reason to FEAT-30.SPEC-003 without changing any data.
4. If eligibility passes, request the full refund through FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution), carrying the Deposit Transaction reference and the Pro's payout account reference.
5. On the execution's immediate confirmation, set Deposit Transaction.status to Refunded and record the outcome_reason as goodwill and the refund timestamp; on a not-yet-completable report, set Deposit Transaction.status to Refund in Progress (FEAT-30.SPEC-011 owns the resulting retry).
6. Write an append-only activity event recording the goodwill refund, its timestamp, the actor (Talia), and any optional private reason (FEAT-16, XBR-21).
7. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) with the goodwill-refund outcome (or the "in progress" variant, if not yet complete).
8. Return the committed outcome to FEAT-30.SPEC-003 for its success feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Goodwill refund completed immediately | Eligibility passes and FEAT-30.SPEC-011 confirms the refund on the first attempt | Deposit Transaction.status -> Refunded; outcome_reason set to goodwill | Talia sees the refund confirmed on FEAT-30.SPEC-003; Riley receives the goodwill-refund notice | FEAT-30.SPEC-003, FEAT-30.SPEC-011, FEAT-16, FEAT-30.SPEC-012 |
| Goodwill refund entered in progress | Eligibility passes but FEAT-30.SPEC-011 reports it cannot complete immediately | Deposit Transaction.status -> Refund in Progress | Talia sees the refund confirmed as "in progress, will complete automatically" on FEAT-30.SPEC-003 and her dashboard attention flag; Riley receives the "in progress" client notice | FEAT-30.SPEC-003, FEAT-30.SPEC-011, FEAT-12, FEAT-30.SPEC-012 |
| Commit rejected -- deposit no longer refundable | Deposit Transaction.status is already Refunded, Refund in Progress, or Disputed at write time | No data changes | Talia sees "This booking's deposit has already been resolved and cannot be refunded again." on FEAT-30.SPEC-003 | FEAT-30.SPEC-003, FEAT-30.SPEC-006 |
| Commit rejected -- booking already completed | Booking.state has reached Completed at write time | No data changes | Talia sees "A goodwill refund is no longer available once a booking is completed." | FEAT-30.SPEC-003, FEAT-30.SPEC-006 |
| Write failure (processing error) | The commit cannot be written for a reason other than an eligibility conflict | No data changes | Talia sees a retry prompt on FEAT-30.SPEC-003; the deposit remains at its prior status | FEAT-30.SPEC-003 |

## Data Model

**Reads:** Booking (state, owning Pro Account); Deposit Transaction (status).
**Creates:** Activity Event (FEAT-16) -- one per committed goodwill refund.
**Updates:** Deposit Transaction -- status (Refunded or Refund in Progress), outcome_reason, refund timestamp.
**Deletes:** None.

## Business Rules

- SC-17: this automation never adjudicates whether a goodwill refund is warranted -- that judgment is made entirely by Talia through FEAT-30.SPEC-003 before this automation ever runs.
- XBR-10: the refund this automation triggers is always full, happens at most once per deposit, and a not-yet-completable attempt is retried automatically and never dropped, per FEAT-30.SPEC-011's execution and retry behavior.
- A goodwill refund never changes Booking.state -- only Deposit Transaction.status, per the Entity-Lifecycle Coverage Matrix; the booking's own cancellation/no-show/completion history is untouched by this action.
- A goodwill refund is available whenever Deposit Transaction.status is Captured or Forfeited and Booking.state has not reached Completed (FEAT-30.SPEC-006) -- independent of the cancellation policy window, since it is a Pro override rather than a window-based outcome.
- FEAT-09's outcome-evaluation automation never fires for a goodwill refund -- it is a standalone Pro-initiated path with its own commit, distinct from the automatic outside-window or Pro-cancellation refund paths FEAT-09 owns.

## Edge Cases

- **A client-side outside-window cancellation refunds this same deposit automatically (FEAT-09) in the instant before Talia confirms goodwill** -- The eligibility re-check finds Deposit Transaction.status already Refunded and denies with "This booking's deposit has already been resolved and cannot be refunded again."; no duplicate refund is requested.
- **The booking is marked Completed by the Auto-Completion Sweep in the instant before Talia confirms goodwill** -- The eligibility re-check finds Booking.state already Completed and denies with the completed-state message; the deposit is left exactly as it was.
- **Talia issues a goodwill refund reached from a no-show prompt (FEAT-11), before the no-show mark itself is confirmed** -- Talia chose goodwill instead of marking a no-show; the booking's Deposit Transaction is at Captured (never having been Forfeited), and this automation processes it as a standard Captured-status goodwill refund with no interaction with FEAT-11's own marking flow.
- **Talia issues a goodwill refund on a deposit already Forfeited from a no-show mark** -- Eligible per FEAT-30.SPEC-006 (Forfeited is a refundable status); the refund converts the kept deposit to Refunded, and per FEAT-11.SPEC-004's own edge case, this also closes that no-show mark's 24-hour undo window.
- **Concurrent trigger firing (Talia confirms goodwill refunds on two different bookings at effectively the same time)** -- Each commit processes independently against its own distinct Deposit Transaction; no interference occurs.
- **Trigger fires while a previous goodwill commit for the same booking is still in flight** -- FEAT-30.SPEC-003's confirm control is disabled during submission, preventing a duplicate commit request for the same booking's deposit.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; the outcome or denial is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Once-only and until-completion eligibility re-checked before the write |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | Triggers (outbound) | The determined-due refund is requested through this integration |
| FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | A successful commit writes an append-only activity event |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | A completed or in-progress goodwill refund triggers the client's notice |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | References (inbound) | One of this action's entry points; a goodwill refund can be chosen instead of marking a no-show |
| FEAT-16 (Booking & Payment Activity Record) | References (inbound) | Another of this action's entry points, from a no-show dispute timeline |

## Analytics and Success Signals

- **goodwill_refund_issued** (source: no_show_prompt / dispute_timeline / booking_row; outcome: completed / in_progress) -- supports success-metrics.md: "Pro Change Correctness"
- **goodwill_refund_completed** () -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_refund_rejected** (reason: not_refundable / already_completed) -- N/A -- no Stage 2 metric measures rejected goodwill attempts directly; retained so a denied action is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-009-AC-01:** Given Talia confirms a goodwill refund on a booking whose deposit is Captured and not yet Completed, when this commit runs and the execution confirms immediately, then Deposit Transaction.status is set to Refunded and Riley receives the goodwill-refund notice.

**FEAT-30.SPEC-009-AC-02:** Given the refund execution reports it cannot complete immediately, when this commit processes that outcome, then Deposit Transaction.status is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley sees "in progress."

**FEAT-30.SPEC-009-AC-03:** Given a booking's Deposit Transaction is already Refunded through an automatic client cancellation, when Talia confirms a goodwill refund on it, then the commit is rejected with "This booking's deposit has already been resolved and cannot be refunded again."

**FEAT-30.SPEC-009-AC-04:** Given a booking has reached Completed, when Talia confirms a goodwill refund on it, then the commit is rejected with "A goodwill refund is no longer available once a booking is completed."

**FEAT-30.SPEC-009-AC-05:** Given a successful goodwill refund commit, when it completes, then exactly one append-only activity event is written recording the action and Talia as the actor.

**FEAT-30.SPEC-009-AC-06:** Given Talia reaches the goodwill refund screen from a no-show prompt and confirms before marking the no-show, when this commit runs, then it processes the Captured-status deposit normally with no interaction with the no-show marking flow.

**FEAT-30.SPEC-009-AC-07:** Given a booking's deposit is already Forfeited from a no-show mark, when Talia confirms a goodwill refund on it, then the commit succeeds, converting the deposit to Refunded.

**FEAT-30.SPEC-009-AC-08:** Given a goodwill refund converts a Forfeited deposit to Refunded, when Talia later attempts to undo the original no-show mark, then the undo is denied per FEAT-11.SPEC-004, since the deposit is no longer Forfeited.

**FEAT-30.SPEC-009-AC-09:** Given the commit cannot be written due to a processing error, when the failure occurs, then Talia sees a retry prompt and the deposit remains at its prior status.

**FEAT-30.SPEC-009-AC-10:** Given Talia confirms goodwill refunds on two different bookings at effectively the same time, when both commits run, then each succeeds independently with no interference.

**FEAT-30.SPEC-009-AC-11:** Given a goodwill commit is already in flight for a booking, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-009-AC-12:** Given Talia reaches this action from a no-show dispute timeline (FEAT-16) rather than a no-show prompt, when she confirms the refund, then this commit processes it identically regardless of entry point.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-002
spec_name: No-Show Marking & Deposit Forfeiture
spec_slug: no-show-marking-deposit-forfeiture
parent_feature: FEAT-11
parent_feature_name: No-Show Marking & Deposit Forfeiture
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: No-Show Marking & Deposit Forfeiture

## Overview

**Name:** No-Show Marking & Deposit Forfeiture
**ID:** FEAT-11.SPEC-002
**Type:** Automation
**Purpose:** On Talia's confirmed mark, atomically transitions the Booking to No-Show and its Deposit Transaction to Forfeited in one step, deriving the forfeiture outcome from the booking's acknowledged cancellation policy version, with no separate invoicing step and no manual chasing.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- Re-validating marking eligibility immediately before writing (window and ownership, per FEAT-11.SPEC-004)
- Atomically transitioning the Booking (Awaiting Outcome -> No-Show) and its Deposit Transaction (Captured -> Forfeited) as one committed step
- Deriving the forfeiture outcome from the Cancellation Policy version the booking's client acknowledged
- Retrying a failed write and flagging it to Talia rather than silently dropping it, since money is at stake
- Feeding the resulting event to the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004)
- Notifying the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005) that the booking is now marked no-show, so it can record its explicit no-action decision for the Pro's connected calendar

**Non-Goals:**
- Deciding the marking window or ownership eligibility itself -- owned by FEAT-11.SPEC-004 (Logic/Rule); this automation only re-checks the outcome that rule spec defines immediately before writing
- Reversing a no-show mark -- owned by FEAT-11.SPEC-003 (No-Show Mark Undo), a distinct automation with its own trigger, window, and outcome set
- Charging or re-charging the client's card -- excluded per SC-13: no card is kept on file for later charges, and the deposit was already captured at booking by FEAT-07, so this automation moves no money, it only changes the deposit's disposition
- Writing the append-only activity record entry itself -- owned by FEAT-16 (XBR-21); this automation only emits the event FEAT-16 consumes
- Changing anything on the Pro's personal calendar -- owned by FEAT-04.SPEC-005 (Booking-to-Calendar Sync), which receives the no-show event and decides no calendar action is needed because the appointment already occurred; this automation only emits the event
- Notifying the Client -- excluded per the feature's Communications field, which keeps the outcome visible only in the Client's own booking history rather than triggering a separate notice

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "Mark no-show" | FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | The prompt's re-validation via FEAT-11.SPEC-004 has already passed at the moment the tap registers | Booking identity, start_time, state, policy_version, and the Deposit Transaction identity and current status |

## Processing Logic

1. Receive the confirmed mark request from FEAT-11.SPEC-001, carrying the Booking's identity.
2. Re-check the marking window and Pro-ownership eligibility per FEAT-11.SPEC-004: the Booking's start_time has passed, the current time is before the auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12), and the requesting Pro owns this booking.
3. If eligibility fails at this re-check (the window closed or the state changed between prompt open and confirm), stop and return the exact denied reason to FEAT-11.SPEC-001 without writing anything.
4. Read the Deposit Transaction linked to this Booking and confirm its current status is Captured. If it is any other status (already Forfeited, Refunded, Disputed, or in a refund cycle), stop and report the current status back to FEAT-11.SPEC-001 -- a deposit is refunded or forfeited only once (dependency map's Contention rule for Deposit Transaction).
5. Read the Booking's acknowledged policy_version to confirm the no-show outcome it defines is "deposit kept" (XBR-09's binary rule: inside the window or a no-show always keeps the deposit).
6. Atomically transition the Booking's state from Awaiting Outcome to No-Show and the Deposit Transaction's status from Captured to Forfeited, recording the outcome_reason as this no-show mark and the outcome timestamp as the current time. Both writes commit together or neither does -- there is no state where one succeeds and the other does not.
7. On successful commit, emit the no-show marked and deposit forfeited events for the activity record (FEAT-16.SPEC-002), revenue insights (FEAT-25.SPEC-004), and the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005, which confirms no calendar action is needed for a no-show and takes none), and return the success outcome to FEAT-11.SPEC-001.
8. If the atomic write cannot complete (connectivity or processing error), retry automatically; if it still cannot complete, flag the failure back to FEAT-11.SPEC-001 for Talia to see, leaving the Booking and Deposit Transaction in their prior, consistent state (Awaiting Outcome / Captured) -- never a partial transition.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Marked successfully | Eligibility re-check passes, Deposit Transaction was Captured, atomic write commits | Booking: Awaiting Outcome -> No-Show. Deposit Transaction: Captured -> Forfeited, outcome_reason and timestamp set | Prompt switches in place to the Undo state showing the deposit as kept | FEAT-11.SPEC-001 (result), FEAT-16.SPEC-002 (activity record), FEAT-25.SPEC-004 (revenue aggregates), FEAT-04.SPEC-005 (calendar no-action confirmation) |
| Marking window closed since prompt opened | Re-check finds the current time now past the auto-completion boundary, or the booking's state changed | None | Prompt shows the exact denied message from FEAT-11.SPEC-004 ("This booking has already auto-completed and can no longer be marked as a no-show." or the current-state message) and offers only Close | FEAT-11.SPEC-001 |
| Deposit Transaction not in a forfeitable state | Deposit Transaction status is not Captured at the moment of the check (already Forfeited, Refunded, Disputed, or Refund in Progress) | None | Prompt shows the current deposit status and does not attempt the mark | FEAT-11.SPEC-001 |
| Write failure, resolved on retry | Atomic write fails once, succeeds on automatic retry | Same as "Marked successfully," delayed by the retry interval | Prompt's loading state extends briefly through the retry, then resolves to the success view | FEAT-11.SPEC-001, FEAT-16.SPEC-002, FEAT-25.SPEC-004, FEAT-04.SPEC-005 |
| Write failure, unresolved | Atomic write fails and remains unresolved after automatic retry | None -- Booking and Deposit Transaction remain in their prior state | Prompt shows an error state flagging the failure to Talia, with the booking's true (unmarked) state re-read and displayed | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Booking -- start_time, state, policy_version, owning Pro Account. Deposit Transaction -- status. Cancellation Policy -- the version referenced by the Booking's policy_version, specifically its inside_window_outcome / no-show outcome (binary: deposit kept).
**Creates:** None.
**Updates:** Booking -- state (Awaiting Outcome -> No-Show). Deposit Transaction -- status (Captured -> Forfeited), outcome_reason, outcome timestamp.
**Deletes:** None -- financial and booking records are retained for the life of the account (SC-22); this automation never removes a record.

## Business Rules

- XBR-08: the forfeiture outcome is derived from the policy version the booking's client acknowledged at booking time -- a later edit to the Pro's cancellation policy never changes an existing booking's outcome.
- XBR-09: deposit outcomes are binary -- a no-show always keeps the full deposit already captured, never a percentage or schedule (SC-18).
- XBR-12: a booking can be marked no-show only after its start_time has passed and before its 7-day auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12); this automation re-enforces that window at write time even though FEAT-11.SPEC-001 already checked it, since time may have advanced between prompt open and confirm.
- The Booking and Deposit Transaction transitions are always written together, atomically -- this automation never leaves the Booking marked No-Show with the Deposit Transaction still Captured, or vice versa.
- A deposit can be forfeited only once overall (dependency map's Contention rule for Deposit Transaction) -- this automation refuses to act on a Deposit Transaction that is not currently Captured.
- This automation moves no money: the deposit was already captured by FEAT-07 through payment processing at booking time (ASMP-31); marking a no-show only changes the deposit's recorded disposition.

## Edge Cases

- **The booking's start_time has not yet passed when the confirm arrives** -- Cannot occur through the normal path: FEAT-11.SPEC-001 only offers this prompt from a past-due booking row. If reached anyway (e.g., a stale client-side state), the re-check in step 2 rejects it with the standard "not yet eligible" denial from FEAT-11.SPEC-004.
- **The Cancellation Policy version referenced by the booking has since been superseded by a newer edit** -- The booking's own acknowledged policy_version is read, never the Pro's current policy (XBR-08); a newer version never applies retroactively to this booking.
- **A card-issuer dispute notice arrives for this Deposit Transaction between prompt open and confirm** -- The Deposit Transaction's Disputed overlay does not itself change its underlying status from Captured (per the dependency map: "a Disputed overlay never erases the underlying outcome"), so the forfeiture proceeds normally if still otherwise eligible; the Disputed overlay and the Forfeited outcome coexist and both remain visible to Support and Talia through FEAT-16.
- **Concurrent trigger firing (Talia taps "Mark no-show" from two devices signed into the same account at effectively the same time)** -- The first commit to reach the atomic write wins; the second re-check (step 4) finds the Deposit Transaction already Forfeited and reports the "not in a forfeitable state" outcome rather than attempting a second forfeiture, per the dependency map's reject-with-refresh Contention resolution.
- **Trigger fires while a previous run is in flight for the same booking** -- The prompt's Confirming state (FEAT-11.SPEC-001) disables the action button while the first run is in progress, so a second run for the same booking cannot start from the same session; a run from a different session for the same booking is handled by the concurrent-trigger-firing case above.
- **The write partially applies before a processing error interrupts it** -- The atomic commit guarantees both the Booking and Deposit Transaction transitions succeed together or neither is retained; a retry after an interrupted attempt re-runs the full atomic write from the last confirmed state, never resuming from a half-applied one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | Triggered by (inbound) | Talia's "Mark no-show" confirm fires this automation |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Supplies the marking-window and ownership eligibility this automation re-checks before writing |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | References (outbound) | The counterpart automation that can later reverse this transition within the grace window |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (inbound) | Supplies the acknowledged policy version's no-show outcome this automation reads |
| FEAT-07 (Deposit Payment at Booking) | References (inbound) | Created and captured the Deposit Transaction this automation forfeits |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Consumes the no-show marked and deposit forfeited events for the append-only record |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Consumes the forfeited deposit for the no-show rate and revenue aggregates |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Affects (outbound) | Receives the booking-marked-no-show event and confirms no calendar action is needed (the appointment already occurred); this automation takes no calendar action itself |

## Analytics and Success Signals

- **no_show_marked** (time_since_start_time, retry_occurred: true/false) -- supports success-metrics.md: "No-Show Recovery Rate"
- **no_show_forfeiture_write_failed** (reason: connectivity / processing_error) -- supports success-metrics.md: "No-Show Recovery Rate" (the target is zero instances requiring manual chasing, so every unresolved write failure is exactly the gap this metric must surface)
- **no_show_marking_denied** (reason: window_closed / not_owner / deposit_not_forfeitable) -- supports success-metrics.md: "No-Show Recovery Rate" (denials on re-check are the cases the automatic path could not complete cleanly on the first attempt)

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Talia confirms "Mark no-show" on a booking that is past its start_time, before its auto-completion boundary, and owned by her, when the automation runs, then the Booking transitions to No-Show and the Deposit Transaction transitions to Forfeited in one atomic write.

**FEAT-11.SPEC-002-AC-02:** Given the atomic write in FEAT-11.SPEC-002-AC-01 commits successfully, when it completes, then the prompt (FEAT-11.SPEC-001) switches in place to the Undo state showing the deposit as kept, with no separate invoicing step.

**FEAT-11.SPEC-002-AC-03:** Given Talia confirms "Mark no-show" but the booking's auto-completion boundary (platform parameter: `booking-auto-completion-window-days`) has passed since the prompt opened, when the automation re-checks eligibility, then no write occurs and the exact denied message from FEAT-11.SPEC-004 is returned.

**FEAT-11.SPEC-002-AC-04:** Given Talia confirms "Mark no-show" on a booking whose Deposit Transaction is already Forfeited (e.g., marked from another session moments earlier), when the automation reads the Deposit Transaction's status, then no second forfeiture is attempted and the current status is reported back.

**FEAT-11.SPEC-002-AC-05:** Given the atomic write fails once due to a connectivity error, when the automation retries automatically, then the Booking and Deposit Transaction transition successfully on retry and Talia sees the success outcome after a brief extended loading state.

**FEAT-11.SPEC-002-AC-06:** Given the atomic write fails and remains unresolved after automatic retry, when the failure is reported, then the Booking and Deposit Transaction remain in their prior state (Awaiting Outcome / Captured) and Talia sees an error state flagging the failure rather than a false success.

**FEAT-11.SPEC-002-AC-07:** Given a booking's acknowledged policy_version defines the no-show outcome as deposit kept, when this automation derives the forfeiture outcome, then it applies that acknowledged version's outcome even if the Pro's current cancellation policy has since been edited to a newer version.

**FEAT-11.SPEC-002-AC-08:** Given this Deposit Transaction has an open card-issuer dispute (Disputed overlay) at the moment Talia confirms the mark, when the automation runs and the deposit is otherwise still Captured, then the forfeiture proceeds and the Disputed overlay remains visible alongside the new Forfeited outcome.

**FEAT-11.SPEC-002-AC-09:** Given Talia confirms "Mark no-show" from two devices for the same booking at effectively the same time, when both requests reach the automation, then only the first commit succeeds and the second is rejected with the current (already Forfeited) status.

**FEAT-11.SPEC-002-AC-10:** Given this automation's write is in flight for a booking, when a second "Mark no-show" confirm for the same booking arrives from the same session before the first completes, then the prompt's disabled Confirming state (FEAT-11.SPEC-001) prevents a second run from starting.

**FEAT-11.SPEC-002-AC-11:** Given this automation successfully marks a booking as a no-show, when the transition commits, then the no_show_marked event is emitted and the activity record (FEAT-16.SPEC-002), revenue insights (FEAT-25.SPEC-004), and the Booking-to-Calendar Sync automation (FEAT-04.SPEC-005, which takes no calendar action for a no-show) receive the resulting no-show and forfeiture data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (marked, window closed, deposit not forfeitable, retry-resolved failure, unresolved failure) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-003
spec_name: No-Show Mark Undo
spec_slug: no-show-mark-undo
parent_feature: FEAT-11
parent_feature_name: No-Show Marking & Deposit Forfeiture
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: No-Show Mark Undo

## Overview

**Name:** No-Show Mark Undo
**ID:** FEAT-11.SPEC-003
**Type:** Automation
**Purpose:** Within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`), atomically reverses a no-show mark -- restoring the Booking to Completed and the Deposit Transaction to its prior Captured status -- when Talia confirms the mistake was hers.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- Re-validating the undo grace window and ownership immediately before writing (per FEAT-11.SPEC-004)
- Atomically transitioning the Booking (No-Show -> Completed) and its Deposit Transaction (Forfeited -> Captured) as one committed step
- Retrying a failed undo write and flagging it to Talia rather than silently dropping it, since money is at stake
- Feeding the resulting reversal event to the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004), which reverses only the saved-from-no-shows aggregate contribution

**Non-Goals:**
- Deciding the undo grace window or ownership eligibility itself -- owned by FEAT-11.SPEC-004 (Logic/Rule); this automation only re-checks the outcome that rule spec defines immediately before writing
- Marking a booking as a no-show in the first place -- owned by FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture), the forward automation this one reverses
- Issuing a goodwill refund -- excluded per this feature's Key Capabilities; a goodwill refund is a distinct, separate action available only through Pro Booking Management (FEAT-30) and is never a side effect of this undo
- Undoing an undo (re-marking after reversal is a fresh "Mark no-show" action) -- excluded because once reversed, the Booking returns to Completed and the standard marking window and eligibility in FEAT-11.SPEC-004 govern any subsequent mark attempt from first principles, not a special "re-undo" path
- Extending or restarting the grace window on a failed or retried undo attempt -- excluded per FEAT-11.SPEC-004: the window is measured from the original Forfeited outcome timestamp and is never reset by an intervening attempt

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "Undo no-show" | FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | The prompt's re-validation via FEAT-11.SPEC-004 has already passed at the moment the tap registers, and the Booking is currently No-Show | Booking identity, state, and the Deposit Transaction identity, current status, and Forfeited outcome timestamp |

## Processing Logic

1. Receive the confirmed undo request from FEAT-11.SPEC-001, carrying the Booking's identity.
2. Re-check ownership and the undo grace window per FEAT-11.SPEC-004: the requesting Pro owns this booking, and the current time is within the fixed grace window (platform parameter: `no-show-undo-grace-window-hours`) measured from the Deposit Transaction's Forfeited outcome timestamp.
3. If eligibility fails at this re-check (the window elapsed between prompt open and confirm), stop and return the exact denied reason ("This no-show mark can no longer be undone.") to FEAT-11.SPEC-001 without writing anything.
4. Read the Booking's current state and confirm it is still No-Show, and read the Deposit Transaction's current status and confirm it is still Forfeited. If either has already changed (e.g., a goodwill refund was issued through FEAT-30 in the meantime), stop and report the current state back to FEAT-11.SPEC-001 -- this automation only reverses its own counterpart transition, never any other outcome.
5. Atomically transition the Booking's state from No-Show back to Completed and the Deposit Transaction's status from Forfeited back to Captured, clearing the no-show outcome_reason and timestamp back to the state they held immediately before the original mark. Both writes commit together or neither does.
6. On successful commit, emit the no-show mark undone event for the activity record (FEAT-16.SPEC-002) and revenue insights (FEAT-25.SPEC-004), and return the success outcome to FEAT-11.SPEC-001.
7. If the atomic write cannot complete (connectivity or processing error), retry automatically; if it still cannot complete, flag the failure back to FEAT-11.SPEC-001 for Talia to see, leaving the Booking and Deposit Transaction in their prior, consistent state (No-Show / Forfeited) -- never a partial reversal.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Undo successful | Re-check passes (window open, Pro owns booking), Booking still No-Show, Deposit Transaction still Forfeited, atomic write commits | Booking: No-Show -> Completed. Deposit Transaction: Forfeited -> Captured, outcome_reason/timestamp cleared | Prompt switches in place to the Mark state, reflecting the booking as Completed and the deposit as restored | FEAT-11.SPEC-001 (result), FEAT-16.SPEC-002 (activity record), FEAT-25.SPEC-004 (aggregate reversal) |
| Grace window elapsed since prompt opened | Re-check finds the current time now past the grace-window boundary | None | Prompt shows "This no-show mark can no longer be undone." and offers only Close | FEAT-11.SPEC-001 |
| Booking or Deposit Transaction state already changed | Booking is no longer No-Show, or Deposit Transaction is no longer Forfeited (e.g., a goodwill refund already issued) | None | Prompt shows the current state and does not attempt the undo | FEAT-11.SPEC-001 |
| Write failure, resolved on retry | Atomic write fails once, succeeds on automatic retry | Same as "Undo successful," delayed by the retry interval | Prompt's loading state extends briefly through the retry, then resolves to the success view | FEAT-11.SPEC-001, FEAT-16.SPEC-002, FEAT-25.SPEC-004 |
| Write failure, unresolved | Atomic write fails and remains unresolved after automatic retry | None -- Booking and Deposit Transaction remain in their prior state | Prompt shows an error state flagging the failure to Talia, with the booking's true (still-marked) state re-read and displayed | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Booking -- state, owning Pro Account. Deposit Transaction -- status, outcome_reason/timestamps.
**Creates:** None.
**Updates:** Booking -- state (No-Show -> Completed). Deposit Transaction -- status (Forfeited -> Captured), outcome_reason and timestamp cleared back to their pre-mark values.
**Deletes:** None -- financial and booking records are retained for the life of the account (SC-22); this automation never removes a record, it only reverses a transition.

## Business Rules

- XBR-12: an undo is available only within a fixed 24-hour window (platform parameter: `no-show-undo-grace-window-hours`) measured from the original Forfeited outcome timestamp, never extended by a failed or retried attempt.
- The Booking and Deposit Transaction transitions are always reversed together, atomically -- this automation never leaves the Booking Completed with the Deposit Transaction still Forfeited, or vice versa.
- This automation reverses only its own counterpart forward transition (FEAT-11.SPEC-002's mark); it never acts on a Booking or Deposit Transaction whose state changed through a different path (cancellation, goodwill refund, dispute) in the meantime.
- This automation moves no money in either direction: the deposit remains the same captured funds throughout, and reversing the mark only restores its recorded disposition -- no re-authorization or new charge occurs.
- Undoing does not reopen the original marking window: once reversed, the Booking is Completed, and any later no-show determination for this same booking is a fresh FEAT-11.SPEC-002 mark attempt, governed by FEAT-11.SPEC-004 from first principles (not a special re-undo case).

## Edge Cases

- **A goodwill refund is issued through FEAT-30 between the mark and the undo attempt** -- The Deposit Transaction is no longer Forfeited (it has moved to a refund outcome), so step 4's re-check finds the state already changed and reports it; this automation never overwrites a refund outcome back to Captured.
- **The undo grace window elapses by seconds while the confirm request is in flight** -- The re-check in step 2 uses the time at the moment the automation evaluates it, not the time the prompt was opened; a confirm that arrives after the boundary is denied even if the prompt showed time remaining moments earlier.
- **Concurrent trigger firing (Talia taps "Undo no-show" from two devices at effectively the same time)** -- The first commit to reach the atomic write wins; the second re-check (step 4) finds the Booking already Completed and the Deposit Transaction already Captured, and reports the current state rather than attempting a second reversal, per the dependency map's reject-with-refresh Contention resolution.
- **Trigger fires while a previous run is in flight for the same booking** -- The prompt's Confirming state (FEAT-11.SPEC-001) disables the action button while the first run is in progress, so a second run for the same booking cannot start from the same session; a run from a different session is handled by the concurrent-trigger-firing case above.
- **The write partially applies before a processing error interrupts it** -- The atomic commit guarantees both the Booking and Deposit Transaction reversal succeed together or neither is retained; a retry after an interrupted attempt re-runs the full atomic reversal from the last confirmed (No-Show / Forfeited) state, never resuming from a half-applied one.
- **Talia undoes, then immediately wants to re-mark the same booking as a no-show** -- Because undo does not reopen the original window (per Business Rules above), this is simply a fresh FEAT-11.SPEC-002 attempt; it succeeds only if the booking is still within the marking window bounds FEAT-11.SPEC-004 defines at that later moment.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | Triggered by (inbound) | Talia's "Undo no-show" confirm fires this automation |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Supplies the undo grace-window and ownership eligibility this automation re-checks before writing |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | References (inbound) | The forward automation whose transition this one reverses |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Consumes the no-show mark undone event for the append-only record |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | Consumes the no-show mark undone event to reverse only the saved-from-no-shows aggregate contribution; the booking count is untouched |

## Analytics and Success Signals

- **no_show_mark_undone** (time_since_marked, retry_occurred: true/false) -- supports success-metrics.md: "No-Show Recovery Rate" (a clean, complete reversal keeps the automatic outcome trustworthy when Talia catches her own mistake, rather than requiring a manual correction)
- **no_show_undo_write_failed** (reason: connectivity / processing_error) -- supports success-metrics.md: "No-Show Recovery Rate"
- **no_show_undo_denied** (reason: window_elapsed / not_owner / state_already_changed) -- supports success-metrics.md: "No-Show Recovery Rate"

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given Talia confirms "Undo no-show" on a booking marked no-show 3 hours ago, within the 24-hour grace window and owned by her, when the automation runs, then the Booking transitions back to Completed and the Deposit Transaction transitions back to Captured in one atomic write.

**FEAT-11.SPEC-003-AC-02:** Given the atomic write in FEAT-11.SPEC-003-AC-01 commits successfully, when it completes, then the prompt (FEAT-11.SPEC-001) switches in place to the Mark state, reflecting the booking as Completed with the deposit restored.

**FEAT-11.SPEC-003-AC-03:** Given Talia confirms "Undo no-show" but the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) has elapsed since the prompt opened, when the automation re-checks eligibility, then no write occurs and "This no-show mark can no longer be undone." is returned.

**FEAT-11.SPEC-003-AC-04:** Given a goodwill refund has already been issued for this booking's Deposit Transaction through FEAT-30 since it was marked no-show, when Talia confirms "Undo no-show," then the automation finds the Deposit Transaction is no longer Forfeited, makes no write, and reports the current state.

**FEAT-11.SPEC-003-AC-05:** Given the atomic reversal write fails once due to a connectivity error, when the automation retries automatically, then the Booking and Deposit Transaction transition back successfully on retry and Talia sees the success outcome after a brief extended loading state.

**FEAT-11.SPEC-003-AC-06:** Given the atomic reversal write fails and remains unresolved after automatic retry, when the failure is reported, then the Booking and Deposit Transaction remain in their prior state (No-Show / Forfeited) and Talia sees an error state flagging the failure rather than a false success.

**FEAT-11.SPEC-003-AC-07:** Given the undo grace window has exactly 5 seconds remaining when Talia's confirm request reaches the automation, when the automation evaluates the current time against the boundary, then the outcome is governed by the time of evaluation, not the time the prompt displayed.

**FEAT-11.SPEC-003-AC-08:** Given Talia confirms "Undo no-show" from two devices for the same booking at effectively the same time, when both requests reach the automation, then only the first commit succeeds and the second is rejected with the current (already Completed / Captured) state.

**FEAT-11.SPEC-003-AC-09:** Given this automation's write is in flight for a booking, when a second "Undo no-show" confirm for the same booking arrives from the same session before the first completes, then the prompt's disabled Confirming state (FEAT-11.SPEC-001) prevents a second run from starting.

**FEAT-11.SPEC-003-AC-10:** Given this automation successfully undoes a no-show mark, when the reversal commits, then the no_show_mark_undone event is emitted the activity record (FEAT-16.SPEC-002) receives the resulting reversal data, and revenue insights (FEAT-25.SPEC-004) reverses only the saved-from-no-shows contribution.

**FEAT-11.SPEC-003-AC-11:** Given Talia successfully undoes a no-show mark and the booking returns to Completed, when she later reconsiders and taps "no-show" on the same booking row again, then FEAT-11.SPEC-002 evaluates the attempt as a fresh mark against FEAT-11.SPEC-004's window rules, not as a special re-undo case.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (undo successful, window elapsed, state already changed, retry-resolved failure, unresolved failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-27.SPEC-011
spec_name: Automatic Pause Resume
spec_slug: automatic-pause-resume
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Automatic Pause Resume

## Overview

**Name:** Automatic Pause Resume
**ID:** FEAT-27.SPEC-011
**Type:** Automation
**Purpose:** Resumes bookings automatically when a Pro-chosen pause reaches its end date, without disturbing a still-active system-imposed pause.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Detecting that a Pro-chosen pause's end date has been reached
- Clearing the Pro-chosen pause source and its message/end date
- Leaving a system-imposed pause untouched, per the precedence rule

**Non-Goals:**
- The precedence logic itself (what "resumed" means when a system-imposed pause coexists) -- owned by FEAT-27.SPEC-009 (Pause State Precedence Rule); this automation only executes the clearing action that rule defines
- Setting or lifting the system-imposed pause -- owned by FEAT-18 (Pro Subscription Billing & Account Management); this automation never reads or writes that source's own trigger conditions, only whether it is currently present
- Notifying Talia that her pause has resumed -- product-features.md's Communications field for this feature states "changes to settings send no client messages," and no Pro-facing resume notification is named in Stage 2; the resumed state is visible the next time she opens FEAT-27.SPEC-004 or FEAT-12

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pause end date reached | Schedule-based (system clock, evaluated against the Pro Account's pause_end_date) | Fires once, at the start of the day following the Pro-chosen pause's pause_end_date, in the Pro's own timezone (XBR-25) | The Pro Account reference, the Pro-chosen pause's message and end date, and the current system-imposed pause flag |
| Pause end date cleared or changed before it is reached | FEAT-27.SPEC-004 (Pause Bookings) | The Pro edits or removes the end date, or turns the pause off manually, before this automation's scheduled fire | The updated pause state, which supersedes the originally-scheduled fire |

## Processing Logic

1. For every Pro Account with an active Pro-chosen pause carrying a pause_end_date, evaluate whether that date has been reached, using the Pro's own account timezone (XBR-25).
2. If the date has been reached: clear the Pro-chosen pause flag, and clear pause_message and pause_end_date (per FEAT-27.SPEC-009's Cross-Field Rules, these are re-enterable next time the Pro turns the pause on again).
3. Check the current system-imposed pause flag (set independently by FEAT-18).
4. If the system-imposed pause flag is false: the account's overall status becomes Active -- bookings resume.
5. If the system-imposed pause flag is true: the account's overall status remains Paused, per FEAT-27.SPEC-009's precedence rule; only the Pro-chosen source has cleared.
6. If the Pro manually turned off the pause or changed its end date before this evaluation runs, this automation takes no action for that account on this cycle -- the manual action already superseded the scheduled one.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Fully resumed | Pause end date reached and no system-imposed pause is active | Pro-chosen pause flag, pause_message, and pause_end_date cleared; overall status becomes Active | Booking page shows available times again immediately; FEAT-27.SPEC-004 shows "Taking bookings" on next view | FEAT-27.SPEC-004 (Pause Bookings), FEAT-05.SPEC-008 (Booking Page Availability Gate) |
| Partially resumed (system-imposed pause remains) | Pause end date reached but a system-imposed pause is active | Pro-chosen pause flag, pause_message, and pause_end_date cleared; overall status stays Paused | FEAT-27.SPEC-004 shows "Paused -- billing needs attention" on next view, no longer "Paused by you and by billing" | FEAT-27.SPEC-004, FEAT-05.SPEC-008 |
| No-op (already superseded) | The Pro manually cleared or changed the pause before this evaluation ran | None -- the manual action already stands | None from this automation | FEAT-27.SPEC-004 |
| No-op (no pause due) | No Pro Account has a pause_end_date reached on this evaluation cycle | None | None | -- |
| Automation failure | A processing error prevents the scheduled evaluation from completing for one or more accounts | No change for the affected accounts on this cycle; they remain paused | None -- a pause continuing slightly past its chosen date due to a processing hiccup is corrected on the next scheduled evaluation, and is never worse than resuming a booking Talia did not intend to take | -- |

## Data Model

**Reads:** Pro Account -- status (Pro-chosen pause flag, system-imposed pause flag), pause_message, pause_end_date.
**Creates:** None.
**Updates:** Pro Account -- status (Pro-chosen pause flag cleared), pause_message (cleared), pause_end_date (cleared).
**Deletes:** None.

## Business Rules

- FEAT-27.SPEC-009 governs the precedence outcome this automation executes: clearing the Pro-chosen source never implies clearing a system-imposed one.
- XBR-14: while paused (from either source, before or after this automation runs), existing bookings and their reminders, refunds, and client self-service remain unaffected -- this automation touches only the pause state, never any Booking record.
- The evaluation uses the Pro's own account timezone (XBR-25) to determine when the end date is reached, so "today" for this purpose always matches what the Pro sees on her own settings screen.
- A manual pause change by the Pro always takes precedence over this automation's scheduled action for the same cycle -- the automation never overwrites a state the Pro has already changed.

## Edge Cases

- **Talia's pause_end_date is today, and she also manually turns off the pause a few minutes before this automation's scheduled evaluation** -- No-op: her manual action already cleared the Pro-chosen pause; this automation finds nothing left to do for her account on this cycle.
- **Talia's pause_end_date is today, and a system-imposed pause begins on the very same day (subscription lapses past its grace period)** -- Whichever is evaluated at the automation's fire time is authoritative: if the system-imposed pause is already flagged by then, this automation clears the Pro-chosen source but the account stays Paused; if it is flagged moments after this automation runs, the account briefly shows Active before the system-imposed pause takes effect on its own trigger.
- **Concurrent trigger firing (this automation's scheduled evaluation and a manual pause-toggle save from FEAT-27.SPEC-004 happen at effectively the same time)** -- Whichever commits first wins for that account; the second sees the already-updated state and either finds nothing to do (if the automation already cleared it) or overwrites a redundant clear (if the Pro's manual action already cleared it) -- either order yields the same final state, since both actions clear the identical fields.
- **Trigger fires while a previous run is in flight (the scheduled evaluation is still processing a large batch when its next scheduled cycle would fire)** -- The next scheduled cycle is skipped until the in-flight run completes, since re-running against partially-updated accounts could double-process the same pause; accounts not yet reached by the in-flight run are caught by the following cycle, and no pause resumes later than at most one evaluation cycle after its due date.
- **A Pro Account has a pause_end_date but the Pro Account itself was closed (FEAT-29) before the date is reached** -- This automation still runs its evaluation against the record, but FEAT-05.SPEC-008 already shows the account's closed-state message to any visitor regardless of this automation's outcome, so the resume has no visible effect for a closed account.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-004 (Pause Bookings) | Triggered by (inbound) / Affects (outbound) | A saved pause_end_date arms this automation; its outcome updates what the screen shows on next view |
| FEAT-27.SPEC-009 (Pause State Precedence Rule) | References (inbound) | Defines the precedence outcome this automation executes |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Supplies the system-imposed pause flag this automation checks before fully resuming |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Reflects the resumed (or still-paused) state on the public booking page immediately |

## Analytics and Success Signals

- **bookings_resumed** (trigger: automatic, still_paused_by_system: yes/no) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so automatic-resume activity, and how often a system-imposed pause outlives the Pro's own, is observable
- **automatic_resume_superseded** (reason: manual_change_first) -- N/A -- no connected success-metrics.md metric; retained so the frequency of manual pre-emption is observable

## Acceptance Criteria

**FEAT-27.SPEC-011-AC-01:** Given Talia has a Pro-chosen pause with an end date of today and no system-imposed pause, when this automation's scheduled evaluation runs, then the pause clears fully, her account becomes Active, and the booking page shows available times again.

**FEAT-27.SPEC-011-AC-02:** Given Talia has a Pro-chosen pause with an end date of today and a system-imposed pause is also active, when this automation runs, then the Pro-chosen source clears but her account stays Paused, showing "Paused -- billing needs attention" on next view.

**FEAT-27.SPEC-011-AC-03:** Given Talia manually turns off her pause a few minutes before this automation's scheduled evaluation, when the automation runs, then it finds nothing to do for her account.

**FEAT-27.SPEC-011-AC-04:** Given Talia's pause_end_date has not yet been reached, when this automation's scheduled evaluation runs, then her account is left unchanged.

**FEAT-27.SPEC-011-AC-05:** Given the scheduled evaluation encounters a processing error for a batch of accounts, when the run fails, then those accounts remain paused unchanged, and the next scheduled evaluation re-evaluates them.

**FEAT-27.SPEC-011-AC-06:** Given a system-imposed pause is lifted by FEAT-18 after this automation already cleared Talia's Pro-chosen pause on an earlier cycle, when the lift is confirmed, then her account becomes Active without any further action from this automation.

**FEAT-27.SPEC-011-AC-07:** Given this automation's scheduled evaluation and a manual pause-toggle save happen at effectively the same time for the same account, when both are processed, then the final state is the pause cleared exactly once, with no duplicate or conflicting write.

**FEAT-27.SPEC-011-AC-08:** Given Talia's pause_end_date evaluation uses her account's own timezone, when her local date reaches the chosen end date, then the automation fires for her account at that local boundary, not at a different pro's timezone boundary.

**FEAT-27.SPEC-011-AC-09:** Given a Pro Account with a due pause_end_date was closed before this automation's evaluation runs, when the automation processes it, then no visible change results, since FEAT-05.SPEC-008 already shows the closed-account message to any visitor regardless.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (end date reached, superseded by manual change) | 2 |
| Outcome Paths | 5 (fully resumed, partially resumed, no-op superseded, no-op none due, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

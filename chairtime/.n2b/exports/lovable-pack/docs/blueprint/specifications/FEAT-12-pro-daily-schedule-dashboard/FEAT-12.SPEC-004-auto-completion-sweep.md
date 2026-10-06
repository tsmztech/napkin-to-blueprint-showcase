---
document_type: spec
spec_type: automation
spec_id: FEAT-12.SPEC-004
spec_name: Auto-Completion Sweep
spec_slug: auto-completion-sweep
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Auto-Completion Sweep

## Overview

**Name:** Auto-Completion Sweep
**ID:** FEAT-12.SPEC-004
**Type:** Automation
**Purpose:** Automatically marks a Booking Completed 7 days (platform parameter: `booking-auto-completion-window-days`) after its appointment time if the Pro never marked it Completed or No-Show.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Periodically scanning for bookings eligible for automatic completion
- Applying the completion eligibility rules from FEAT-12.SPEC-006 to each candidate
- Transitioning eligible bookings to Completed and recording the balance as settled in person
- Emitting the resulting outcome so it is reflected on the schedule and past-bookings screens

**Non-Goals:**
- Defining the eligibility window and completion mechanics themselves -- owned by FEAT-12.SPEC-006 (Booking Completion Rules); this automation only applies that spec's rules on a schedule
- Marking a booking No-Show -- excluded per the dependency map's Entity-Lifecycle Coverage Matrix: No-Show is a distinct, Pro-initiated transition owned by FEAT-11, never an automatic outcome of this sweep
- Notifying the client that their booking was completed -- product-features.md's Communications field for this feature is N/A (this is a viewing surface); no client-facing message is defined for auto-completion, and none is added here without a Stage 2 source
- Taking or recording an in-app balance payment -- excluded per scope-boundaries.md SC-16; this automation only marks the balance as settled in person, never processes a payment

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Scheduled sweep pass | system (schedule-based; no user-facing trigger spec) | Runs on a recurring schedule frequent enough that no eligible booking waits more than a small fraction of a day past its 7-day mark before being swept | For each candidate Booking: state, start_time, price_agreed, deposit_amount, and the Pro Account it belongs to |

## Processing Logic

1. On each scheduled pass, read every Booking currently in `Confirmed` or `Awaiting Outcome` state across all Pro Accounts.
2. For each such Booking, evaluate whether the current time is at least 7 days after its `start_time`.
3. For every Booking that meets the window condition, apply the eligibility check defined in FEAT-12.SPEC-006 (state must still be `Confirmed` or `Awaiting Outcome` at the moment of transition, to guard against a race with a Pro action taken between step 1's read and this step).
4. For each Booking that still passes the check, transition its `state` to `Completed` and record the balance as settled in person (per FEAT-12.SPEC-006's completion mechanics -- no in-app balance charge is created).
5. For any Booking that no longer passes the check (because the Pro or another automation already moved it out of `Confirmed`/`Awaiting Outcome` since step 1), skip it without effect.
6. Log the sweep pass's outcome counts (bookings evaluated, bookings completed, bookings skipped) for operational visibility; this logging is internal and not itself a user-facing feature.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Booking auto-completed | Booking was `Confirmed` or `Awaiting Outcome`, start_time was at least 7 days ago, and it still passed the eligibility check at transition time | Booking.state -> Completed; balance recorded as settled in person | The booking now shows a Completed status the next time the Pro opens FEAT-12.SPEC-001 or FEAT-12.SPEC-003 -- no interrupting notification, since this is a background sweep, not a screen the Pro is actively watching | FEAT-12.SPEC-001, FEAT-12.SPEC-003 (both display the updated state); FEAT-25.SPEC-004 (aggregates the completed outcome) |
| No action needed (not yet eligible) | Booking is `Confirmed`/`Awaiting Outcome` but start_time is less than 7 days in the past | None | None -- silent, re-evaluated on the next pass | -- |
| No action needed (already resolved) | Booking already left `Confirmed`/`Awaiting Outcome` before this pass reached it (Pro marked it, cancelled it, or a prior sweep pass already completed it) | None | None -- silent | -- |
| Sweep pass failure | The scheduled pass itself cannot run to completion (e.g., an internal processing error interrupts the scan) | No partial state changes are left inconsistent -- any Booking not reached by a failed pass is picked up cleanly on the next scheduled pass | None directly; no booking is left in an ambiguous state, since the sweep only ever moves a Booking forward to Completed in a single step | FEAT-12.SPEC-001, FEAT-12.SPEC-003 (unaffected until the next successful pass catches up) |

## Data Model

**Reads:** Booking -- `state`, `start_time`, `price_agreed`, `deposit_amount`, and the owning Pro Account reference, across all Pro Accounts, for every Booking currently `Confirmed` or `Awaiting Outcome`.
**Creates:** None.
**Updates:** Booking -- `state` (to `Completed`) for each eligible Booking found by this pass.
**Deletes:** None.

## Business Rules

- The completion window is 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time`, per XBR-12 and FEAT-12.SPEC-006.
- This sweep never marks a Booking No-Show -- No-Show is exclusively a Pro-initiated action owned by FEAT-11.
- This sweep is non-destructive: it only ever transitions a Booking forward from `Confirmed`/`Awaiting Outcome` to `Completed`; it never reverts, cancels, or reschedules a Booking.
- The sweep runs across every Pro Account uniformly -- there is no per-Pro configuration of the completion window (it is one value for every Pro (platform parameter: `booking-auto-completion-window-days`), not a per-account setting).
- A Booking already moved out of `Confirmed`/`Awaiting Outcome` by the time this sweep reaches it (by the Pro, by a client cancellation, or by a prior sweep pass) is left untouched, consistent with the Booking entity's reject-with-refresh contention resolution: the first committed transition wins.

## Edge Cases

- **Booking passes its 7-day mark while a client-initiated cancellation is being processed at the same instant** -- Concurrent trigger firing: whichever transition (the sweep's completion, or the cancellation) commits first wins; the other finds the Booking already out of `Confirmed`/`Awaiting Outcome` at its eligibility check and skips it without effect, per FEAT-12.SPEC-006's reject-with-refresh resolution.
- **The Pro marks a booking completed manually a moment before a scheduled sweep pass reaches it** -- The sweep's eligibility check (step 3) re-verifies state at transition time and finds the Booking already `Completed`; it is skipped without effect and without any error.
- **A sweep pass is still processing a large batch when the next scheduled pass would normally start** -- The next pass does not start a second concurrent scan while one is in flight; it waits for the current pass to finish, so no Booking is evaluated by two overlapping passes at once.
- **A Booking's start_time falls in a time zone whose day boundary is ambiguous relative to the sweep's own scheduling clock** -- The comparison is always against the Booking's own `start_time` (stored and interpreted in the Pro's account timezone, per XBR-25), not the sweep's own clock's local day boundary; the 7-day window is computed from that same instant, so timezone handling introduces no separate ambiguity.
- **Pro Account is Paused (subscription lapse or Pro-chosen pause) when a sweep pass reaches one of its bookings** -- The pause affects new bookings and deposits only (XBR-14); it does not exempt existing bookings from this sweep, so eligible bookings on a paused account are still completed on schedule.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-006 (Booking Completion Rules) | References (outbound) | This automation applies SPEC-006's eligibility window and transition mechanics on every pass |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Affects (outbound) | Displays the resulting Completed state on the Pro's next visit |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Affects (outbound) | Displays the resulting Completed state once the booking is in the past |
| FEAT-25.SPEC-004 (Insights Aggregates) | Triggers (outbound) | Each booking auto-completed by this sweep is a booking-outcome event that FEAT-25.SPEC-004 picks up when refreshing its insights aggregates |

## Analytics and Success Signals

- **booking_auto_completed** (days_since_start_time) -- N/A -- reason: this is a background, automatic outcome with no Pro-facing speed or correctness dimension of its own to measure; it supports operational visibility (sweep pass outcome counts, per Processing Logic step 6) rather than any Stage 2 success metric. The Pro-facing signals for this feature's contribution to Daily Dashboard Glance Speed and Pro Change Correctness are emitted by FEAT-12.SPEC-001 (dashboard_viewed, quick_action_taken, booking_marked_completed) and FEAT-12.SPEC-002/FEAT-12.SPEC-005 (attention_item_resolved), not by this automation.
- **auto_completion_sweep_pass_summary** (bookings_evaluated, bookings_completed, bookings_skipped) -- N/A -- reason: an internal operational log for the sweep's own health, not a product success signal tied to a persona-facing outcome.

## Acceptance Criteria

**FEAT-12.SPEC-004-AC-01:** Given a Confirmed booking whose start_time was 7 days ago and Talia never marked it completed or no-show, when the scheduled sweep pass runs, then the booking's state transitions to Completed.

**FEAT-12.SPEC-004-AC-02:** Given a Confirmed booking whose start_time was only 2 days ago, when the sweep pass runs, then the booking is left unchanged.

**FEAT-12.SPEC-004-AC-03:** Given a booking already marked Completed by Talia before the sweep pass reaches it, when the sweep evaluates it, then it is skipped without effect.

**FEAT-12.SPEC-004-AC-04:** Given a booking already marked No-Show by Talia, when the sweep pass runs, then the booking is left unchanged, since this sweep never marks or overrides a No-Show.

**FEAT-12.SPEC-004-AC-05:** Given a booking is cancelled by the Client at effectively the same moment the sweep would complete it, when the cancellation commits first, then the sweep's eligibility check finds the booking already out of Confirmed/Awaiting Outcome and skips it.

**FEAT-12.SPEC-004-AC-06:** Given the sweep transitions a booking to Completed, when Talia next opens FEAT-12.SPEC-001, then the booking shows as Completed with its balance recorded as settled in person.

**FEAT-12.SPEC-004-AC-07:** Given a sweep pass is interrupted by a processing error partway through, when the failure occurs, then no booking is left in a partially-updated state, and the next scheduled pass picks up every still-eligible booking cleanly.

**FEAT-12.SPEC-004-AC-08:** Given a scheduled sweep pass is still running when the next pass would normally start, when the next scheduled time arrives, then the new pass does not start until the current one finishes.

**FEAT-12.SPEC-004-AC-09:** Given a Pro Account is Paused due to a subscription lapse, when an eligible booking on that account reaches its 7-day mark, then the sweep still completes it, since the pause affects only new bookings and deposits (XBR-14).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

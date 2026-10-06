---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-004
spec_name: Confirmed Booking Conflict Flagging
spec_slug: confirmed-booking-conflict-flagging
parent_feature: FEAT-02
parent_feature_name: Availability & Working Hours Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 11
---

# Automation Spec: Confirmed Booking Conflict Flagging

## Overview

**Name:** Confirmed Booking Conflict Flagging
**ID:** FEAT-02.SPEC-004
**Type:** Automation
**Purpose:** System checks every existing confirmed booking against a newly saved Availability Rule version and flags any that now fall outside working hours, without ever cancelling them.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Comparing every upcoming confirmed booking's start time and duration (plus applicable buffer) against a newly saved Availability Rule version's weekly windows
- Producing the set of bookings that no longer fit, for Pro Booking Management (FEAT-30) to surface as attention items
- Never modifying, cancelling, or rescheduling any Booking record as part of this check

**Non-Goals:**
- Cancelling, rescheduling, or otherwise resolving a flagged booking -- excluded per XBR-11: setup changes never silently cancel a confirmed booking; resolution happens only through the Pro's explicit choice in Pro Booking Management (FEAT-30)
- Checking bookings against Time Blocks, Calendar Connection busy time, or Recurring Series -- this spec checks only the newly saved Availability Rule's weekly windows and buffer; the full slot-fitness computation (including those other inputs) belongs to FEAT-03 (Real-Time Slot Availability Engine) and applies only to new bookings, not to re-checking existing ones
- Notifying the client whose booking is flagged -- the Brief's Side-Effect Inventory and XBR-11 route flagged bookings to the Pro's own attention only; any client-facing communication about a resulting change is triggered separately, by whatever action the Pro takes in Pro Booking Management (FEAT-08 messaging)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new Availability Rule version is written | FEAT-02.SPEC-003 (Availability Rule Versioning) | Fires every time FEAT-02.SPEC-003 successfully commits a new version, whether triggered from FEAT-02.SPEC-001 or FEAT-02.SPEC-002 | The newly written Availability Rule version's full field set (weekly_windows, default_buffer, minimum_booking_notice, booking_horizon) and, when relevant, the changed Service.buffer_override |

## Processing Logic

1. Receive the newly written Availability Rule version from FEAT-02.SPEC-003.
2. Read every Booking for this Pro Account currently in the Confirmed state with a start_time in the future.
3. For each such booking, read its service (to determine the applicable buffer: the service's buffer_override if one is set on the new version, otherwise the new version's default_buffer) and its start_time and duration.
4. For each booking, evaluate whether its start_time through start_time + duration + applicable buffer falls entirely within one of the new version's weekly_windows for that day of week, interpreted in the Pro's account timezone.
5. Classify each evaluated booking as either fitting (no action) or no longer fitting (flagged).
6. For every booking classified as no longer fitting, hand off its reference to Pro Booking Management (FEAT-30) so it appears on that feature's attention list; the Booking record itself is left completely unchanged -- no state transition, no cancellation.
7. If zero bookings are flagged, the run completes silently with no Pro-visible output from this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| No conflicts found | Every confirmed future booking still fits the new version | None | None -- the run is silent; the triggering screen's own success confirmation (FEAT-02.SPEC-001 / FEAT-02.SPEC-002) is the only feedback the Pro sees | -- |
| Conflicts found | One or more confirmed future bookings no longer fit the new version | None on the Booking record itself; the flagged reference is handed to Pro Booking Management (FEAT-30) for display | An attention item appears on Pro Booking Management (FEAT-30) per flagged booking; the triggering screen (FEAT-02.SPEC-001 / FEAT-02.SPEC-002) still shows its own ordinary success state, never an error | FEAT-30 (Pro Booking Management) |
| Check failure | The comparison cannot complete (e.g., a processing failure reading bookings) | None | The triggering screen's own save still reports success (versioning already committed); a non-blocking notice appears on the Pro's dashboard: "Some upcoming bookings could not be checked against your new hours -- we'll check again shortly," and the check is retried automatically in the background until it completes | FEAT-02.SPEC-001, FEAT-02.SPEC-002 (unaffected -- their own save already succeeded), Pro Daily Schedule Dashboard (FEAT-12, notice display) |

## Data Model

**Reads:** Booking -- state (Confirmed only), start_time, duration, service reference, for every booking belonging to this Pro Account with a future start_time. Availability Rule -- the newly written version's weekly_windows and default_buffer. Service -- buffer_override, per booking's service, when present.
**Creates:** None.
**Updates:** None -- this automation never writes to the Booking entity, per the dependency map ("it never writes to Booking records itself") and XBR-11.
**Deletes:** None.

## Business Rules

- A confirmed booking is never cancelled, rescheduled, or otherwise modified by this automation -- flagging is purely informational and surfaced through Pro Booking Management (XBR-11).
- Only bookings in the Confirmed state with a future start_time are evaluated; past bookings and bookings already in another state (Completed, No-Show, Cancelled, Rescheduled, Awaiting Outcome) are not re-checked, since a rule change has no bearing on an appointment that has already been kept, missed, or resolved.
- The buffer applied per booking is the service's buffer_override if the new version carries one for that service, otherwise the new version's default_buffer -- the same precedence FEAT-03 applies when computing live slots.
- This check runs against the newly written version only -- it never re-evaluates bookings against any prior, superseded version.
- Resolution of a flagged booking (cancel, reschedule, or keep as an exception) is entirely the Pro's explicit choice, made through Pro Booking Management, never automated here (XBR-11).

## Edge Cases

- **No confirmed future bookings exist for this Pro Account** -- The run completes immediately with zero comparisons and no flags.
- **A booking's service has no buffer_override and the new version's default_buffer changed** -- The booking is evaluated using the new default_buffer; a booking that fit under the old default may newly fail to fit and gets flagged.
- **A booking spans a boundary exactly (start_time + duration + buffer ends exactly at a window's end)** -- Treated as fitting; the fit test is inclusive of the exact boundary, consistent with FEAT-03's own "fits entirely within an open working window" rule.
- **The same booking is flagged by two different rule-version checks in quick succession** (e.g., the Pro edits hours twice within a short span) -- Each check evaluates the booking against its own triggering version independently; the more recent check's flag (or lack of one) is what Pro Booking Management displays, since it reflects the currently-current version.
- **A flagged booking is resolved by the Pro (via FEAT-30) before this automation's next run** -- The next run for a subsequent version change re-evaluates the booking fresh, based on its state at that time; a booking the Pro has since cancelled or rescheduled is simply no longer in the Confirmed-with-future-start_time set this automation reads.
- **Concurrent trigger firing (two Availability Rule versions are written in close succession, e.g., a rapid edit-then-re-edit)** -- Each version triggers its own independent conflict check; if the second version's check completes after the first's, its flag set supersedes the first's for display purposes, since it reflects the latest-effective rule.
- **Trigger fires while a previous run is still in flight** -- A run started for an earlier version is allowed to complete; a run started for a newer version proceeds independently and its results, once ready, are what Pro Booking Management shows, since they reflect the currently-current version. Neither run blocks the other, and neither blocks the triggering screen's own save confirmation, which has already been shown.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggered by (inbound) | Fires on every successfully written new Availability Rule version |
| FEAT-30 (Pro Booking Management) | Affects (outbound) | Flagged bookings are surfaced there as attention items for the Pro's explicit resolution |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A check-failure notice is shown there as a non-blocking attention item |
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | References (inbound) | The triggering screen's own success feedback is unaffected by this automation's outcome |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | References (inbound) | The triggering screen's own success feedback is unaffected by this automation's outcome |

## Analytics and Success Signals

- **booking_conflict_check_completed** (result: no_conflicts / conflicts_found; conflict_count) -- supports success-metrics.md: "Availability Setup Accuracy" (a rising rate of conflicts found indicates the Pro's edits are surprising her own existing bookings, the exact gap this metric measures)
- **booking_conflict_flagged** (booking reference, reason: outside new working windows) -- supports success-metrics.md: "Availability Setup Accuracy"
- **booking_conflict_check_failed** (retry attempt number) -- N/A -- no success-metrics.md metric measures the reliability of this internal check itself; recorded here as an explicit gap rather than silently dropped, consistent with pipeline-rules.md's output-completeness constraint

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Talia has three confirmed future bookings that all still fit her newly saved hours, when FEAT-02.SPEC-003 triggers this check, then no attention items appear on Pro Booking Management and Talia's save shows only its ordinary success confirmation.

**FEAT-02.SPEC-004-AC-02:** Given Talia narrows her Tuesday working window such that a confirmed Tuesday booking no longer fits, when this check runs against the new version, then that booking is flagged and appears as an attention item on Pro Booking Management, while the booking itself remains Confirmed and unchanged.

**FEAT-02.SPEC-004-AC-03:** Given a flagged booking, when Talia views it on Pro Booking Management, then she sees the attention item and can choose to cancel, reschedule, or keep it as an exception -- this automation itself takes no such action.

**FEAT-02.SPEC-004-AC-04:** Given Talia has a confirmed booking already in the Completed state, when a new Availability Rule version is saved, then that booking is not evaluated or flagged by this check.

**FEAT-02.SPEC-004-AC-05:** Given Talia sets a per-service buffer override that increases the buffer a service's confirmed booking needs, when this check runs, then the booking is evaluated with the new override applied and is flagged if it no longer fits.

**FEAT-02.SPEC-004-AC-06:** Given a confirmed booking whose end time plus buffer lands exactly at the edge of a working window, when this check runs, then the booking is treated as fitting.

**FEAT-02.SPEC-004-AC-07:** Given this check cannot complete due to a processing failure, then Talia's triggering save still shows its own success confirmation, and a non-blocking notice appears on her dashboard that some bookings could not yet be checked, with an automatic retry.

**FEAT-02.SPEC-004-AC-08:** Given Talia has zero confirmed future bookings, when a new Availability Rule version is saved, then this check completes immediately with no flags.

**FEAT-02.SPEC-004-AC-09:** Given Talia edits her hours twice in quick succession, when both versions trigger their own conflict checks, then the flag set shown on Pro Booking Management reflects the check against the most recently saved version.

**FEAT-02.SPEC-004-AC-10:** Given a booking was flagged and Talia resolves it through Pro Booking Management before her next hours edit, when she saves another new version, then that booking is re-evaluated fresh based on its current state, not its prior flag.

**FEAT-02.SPEC-004-AC-11:** Given this automation runs for a Pro Account whose Availability Rule was just created for the first time, when the check runs, then it finds no confirmed bookings to flag, since none could exist before the account's first rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (new Availability Rule version written) | 1 |
| Outcome Paths | 3 (no conflicts, conflicts found, check failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

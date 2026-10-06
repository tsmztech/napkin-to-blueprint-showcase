---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-003
spec_name: Recurring Series Setup & Generation Limits
spec_slug: recurring-series-setup-generation-limits
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 6
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Recurring Series Setup & Generation Limits

## Overview

**Name:** Recurring Series Setup & Generation Limits
**ID:** FEAT-21.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the valid interval range for a Recurring Series, the booking-horizon ceiling that governs how far ahead occurrences may ever be generated, and who may create a series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments
**Governed Entity:** Recurring Series

## Scope and Non-Goals

**In Scope:**
- The interval field's valid range (every 1 to 12 weeks) and its error message
- The booking-horizon ceiling that bounds how far ahead FEAT-21.SPEC-004 may ever generate an occurrence
- Authorization for creating a Recurring Series
- Default values and derivations for the originating service and time fields

**Non-Goals:**
- Authorization for viewing, cancelling, or managing an existing series -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) for the cancellation actions, and by FEAT-21.SPEC-002 for the view action's own screen-level access rules.
- The per-occurrence slot validation performed at each generation attempt (duration, buffer, conflicts) -- owned by FEAT-03.SPEC-004 (Slot Validation & Timing Rules); this spec only supplies the horizon ceiling that FEAT-03's own check is bounded by.
- A dedicated pause action for a series -- excluded per this Brief's Non-Goals: no Key Capability, Primary Flow, Alternate, or States-field line describes a pause interaction, so only Active and Ended states are modeled.
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature's Key Capabilities name only setup, group viewing/management, and cancellation; a client who wants a different cadence cancels and sets up a fresh series through FEAT-21.SPEC-001.

## Governed Entity

**Entity:** Recurring Series
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| interval | number (weeks) | How often the series repeats, in whole weeks |
| originating_service | reference (Service) | The Service the series repeats, captured from the booking the series was created from |
| originating_time | derived (time-of-day, day-of-week) | The time-of-day and day-of-week pattern the series repeats, captured from the originating Booking |
| state | enum (Active, Ended) | The series' lifecycle state |
| generated_occurrences | derived list (Booking references) | The occurrences this series has generated within the booking horizon |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Set Up Recurring Series | On submit, when Riley chooses an interval |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | On every generation attempt, to bound how far ahead an occurrence may ever be created |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | On submit, when Talia sets up a series for a client at the chair, subject to the same setup validation as FEAT-21.SPEC-001 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| interval | Required; whole number of weeks, between 1 and 12 inclusive | Always | On submit | "Choose a repeat interval between 1 and 12 weeks." | Yes |
| originating_service | No validation beyond data type -- auto-populated from the originating Booking's Service at setup time; not independently entered | Always | -- | -- | -- |
| originating_time | No validation beyond data type -- auto-populated from the originating Booking's start_time (time-of-day and day-of-week) at setup time; not independently entered | Always | -- | -- | -- |
| state | No validation beyond data type on this spec's side; the Active -> Ended transition is governed by FEAT-21.SPEC-006, not by this spec | Always | -- | -- | -- |
| generated_occurrences | No validation beyond data type on this spec's side; each occurrence's own creation is governed by FEAT-21.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Generation horizon ceiling | interval (this spec), Availability Rule.booking_horizon (external, FEAT-02) | An occurrence is never generated further ahead than the Pro's currently configured booking_horizon, regardless of the series' interval; a series with a short interval simply accumulates more generated occurrences within the same horizon window than a series with a long interval | N/A -- this is a generation-time boundary enforced silently by FEAT-21.SPEC-004, not a form-submission error the client or Pro ever sees at setup time |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Recurring Series (from own booking) | The Client (Riley) | Only from a Booking that is her own, currently active (not cancelled or expired), and has not already originated a series | Create controls (FEAT-21.SPEC-001's offer) are not shown for a booking that is cancelled, expired, or has already originated a series; a direct attempt shows the withdrawn-offer message defined in FEAT-21.SPEC-001's Edge Cases |
| Create Recurring Series (for a client at the chair) | The Pro (Talia) | Always, for any client and booking on her own schedule, through FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail | -- |
| Create Recurring Series | Platform Operator (Support) | Never | The create action is not shown anywhere in Support's read-only view (FEAT-19); Support has no path to this action at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| originating_service | Set to the Service of the Booking the series is created from | On create only | No -- Riley and Talia both create a series from an existing booking, not by choosing a service independently |
| originating_time | Set to the time-of-day and day-of-week of the Booking the series is created from | On create only | No |
| state | Set to Active | On create only | No |
| generated_occurrences | Starts empty; populated by FEAT-21.SPEC-004 as occurrences are generated | On create (empty), then continuously by FEAT-21.SPEC-004 | No -- occurrences are system-generated, never manually added |

## Business Rules

- The interval range (1-12 weeks) and the booking-horizon ceiling are referenced, not duplicated, by FEAT-21.SPEC-001 at setup time and by FEAT-21.SPEC-004 at every later generation attempt, per this Brief's Shared Validation section.
- XBR-03: minimum booking notice and booking horizon limit every client-facing booking path, including occurrence generation; the Pro alone (through FEAT-21.SPEC-010, reached from FEAT-30) may set up or manage a series outside the ordinary notice window, consistent with her general booking-in exception.
- The booking-horizon ceiling is the Pro's currently configured Availability Rule.booking_horizon (FEAT-02) at the moment each generation attempt runs -- it is never a value this spec fixes independently, so a Pro who later widens or narrows her horizon immediately changes how far ahead future generation attempts may reach, without any action on the series itself.
- A series' interval, once set at creation, cannot be changed by any action this spec authorizes; the only way to change cadence is to cancel and create a fresh series (FEAT-21.SPEC-001).

## Edge Cases

- **Interval entered as exactly 1 week** -- Passes validation; the series generates as frequently as the booking horizon allows.
- **Interval entered as exactly 12 weeks** -- Passes validation, the upper boundary.
- **Interval entered as 0 or a negative number** -- Rejected with the standard error message; 0 and negative values are both out of range, not separately messaged.
- **Interval entered as a non-whole number (e.g., 2.5)** -- Rejected with the standard error message; only whole numbers of weeks are valid.
- **The Pro narrows her booking_horizon after a series already has generated occurrences** -- Already-generated occurrences are untouched (XBR-11: setup changes never silently cancel a confirmed booking); FEAT-21.SPEC-004 simply generates no further occurrences until the horizon rolls forward enough to reach the series' next due date again.
- **The Pro widens her booking_horizon while a series is Active** -- FEAT-21.SPEC-004 becomes able to generate further-ahead occurrences on its next run, up to the new, wider ceiling; no action on the series itself is required.
- **Riley attempts to create a second series from the same booking after the first has already been created and later cancelled** -- Denied per the Authorization Rules condition: a booking that has already originated a series (regardless of that series' current state) cannot originate a second one; Riley must build a new series from a different, later booking instead.

## Acceptance Criteria

**FEAT-21.SPEC-003-AC-01:** Given Riley chooses an interval of 3 weeks, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-02:** Given Riley chooses an interval of 1 week, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-03:** Given Riley chooses an interval of 12 weeks, when validation runs, then it passes.

**FEAT-21.SPEC-003-AC-04:** Given Riley chooses an interval of 0 weeks, when validation runs, then she sees "Choose a repeat interval between 1 and 12 weeks." and setup is blocked.

**FEAT-21.SPEC-003-AC-05:** Given Riley chooses an interval of 13 weeks, when validation runs, then she sees the same error message and setup is blocked.

**FEAT-21.SPEC-003-AC-06:** Given Riley chooses an interval of 2.5 weeks, when validation runs, then she sees the same error message and setup is blocked.

**FEAT-21.SPEC-003-AC-07:** Given a series is created, when its originating_service and originating_time are set, then they exactly match the Service and start_time of the Booking it was created from, with no independent input from Riley.

**FEAT-21.SPEC-003-AC-08:** Given Riley's booking is her own, active, and has not already originated a series, when she opens FEAT-21.SPEC-001, then the create offer is shown.

**FEAT-21.SPEC-003-AC-09:** Given Riley's booking has already originated a series, when she navigates back to FEAT-21.SPEC-001 for that same booking, then the create offer is not shown again, per FEAT-21.SPEC-001's own edge case.

**FEAT-21.SPEC-003-AC-10:** Given Talia (the Pro) sets up a series for a client at the chair through FEAT-21.SPEC-010, when she submits an interval within 1-12 weeks, then the same validation passes and the series is created.

**FEAT-21.SPEC-003-AC-11:** Given Platform Operator (Support) views a Pro's account, when they look for a create-series action, then none is shown anywhere in their read-only view.

**FEAT-21.SPEC-003-AC-12:** Given a series' next due occurrence falls further ahead than the Pro's currently configured booking_horizon, when FEAT-21.SPEC-004 evaluates generation, then no occurrence is generated until the horizon rolls forward far enough to reach it.

**FEAT-21.SPEC-003-AC-13:** Given the Pro narrows her booking_horizon after a series already has generated occurrences, when the change takes effect, then the already-generated occurrences are untouched and only future generation attempts are affected.

**FEAT-21.SPEC-003-AC-14:** Given the Pro widens her booking_horizon while a series is Active, when FEAT-21.SPEC-004 next runs, then it can generate occurrences up to the new, wider ceiling.

**FEAT-21.SPEC-003-AC-15:** Given a booking has already originated one series, when a second attempt is made to create a series from that same booking, then it is denied per the Authorization Rules condition.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

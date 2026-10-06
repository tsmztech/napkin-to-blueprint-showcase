---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-005
spec_name: Recurring Time Block Occurrence Generation
spec_slug: recurring-time-block-occurrence-generation
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Recurring Time Block Occurrence Generation

## Overview

**Name:** Recurring Time Block Occurrence Generation
**ID:** FEAT-17.SPEC-005
**Type:** Automation
**Purpose:** Generates and maintains the future dated occurrences of a recurring block pattern (e.g., every Sunday), running each new occurrence through the same conflict detection as a single-date block.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Turning a recurrence pattern (day-of-week, start/end time-of-day, label) into individual dated Time Block occurrence records
- Keeping the generated horizon current as time passes (rolling generation)
- Regenerating not-yet-elapsed future occurrences when Talia edits the pattern's span, day-of-week, or label
- Running every newly generated occurrence through the same conflict scope as FEAT-17.SPEC-004

**Non-Goals:**
- Capturing the recurrence pattern itself -- owned by FEAT-17.SPEC-001, which this automation only reads
- Retiring an occurrence once its own end time passes -- owned by FEAT-17.SPEC-007
- Deriving a recurring pattern automatically from the Pro's personal calendar -- excluded per the Brief's own Non-Goals: calendar busy time is FEAT-04's distinct responsibility and is never converted into a Time Block
- Cross-Pro or shared recurrence patterns -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia saves a new recurring pattern | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires when the Repeats toggle is on and the save commits (via FEAT-17.SPEC-004, zero conflicts on the first occurrence) | Day-of-week, start/end time-of-day, label, first occurrence date |
| Talia edits an existing recurring pattern's span, day-of-week, or label | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires when the edit to a pattern-defining block commits (via FEAT-17.SPEC-004) | Updated day-of-week, start/end time-of-day, label |
| The Pro Account's booking_horizon setting changes | FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Fires when the horizon is extended, since a longer horizon means further-out occurrences must now be generated | Current booking_horizon value |
| Rolling generation check | System (time-based) | Fires periodically to keep each active pattern's generated occurrences current out to the Pro Account's current booking_horizon | Each active pattern's definition; current date |

## Processing Logic

1. Read the recurrence pattern's definition: day-of-week, start/end time-of-day, label, and the Pro Account's current booking_horizon (Availability Rule).
2. Determine every future date, out to the booking_horizon from today, that matches the pattern's day-of-week and does not already have a generated occurrence.
3. For each such date, construct a candidate occurrence (that date's start/end from the pattern's time-of-day, carrying the pattern's label).
4. Run each candidate occurrence through the same validation and conflict scope as FEAT-17.SPEC-004 (end-after-start already guaranteed by the pattern; conflicting-booking check against confirmed Bookings on that date).
5. If a candidate has zero conflicts, commit it as a Time Block occurrence record referencing the parent pattern.
6. If a candidate conflicts with one or more confirmed bookings, hold that single occurrence pending and hand it to FEAT-17.SPEC-003 for Talia's review, exactly as a single-date save would; generation continues independently for the pattern's other candidate dates.
7. When the pattern itself is edited (span, day-of-week, or label changed), delete every not-yet-elapsed generated occurrence that has not already had a conflict resolved by Talia, and repeat steps 2--6 under the new definition; occurrences whose own end time has already passed are never touched (they carry no retention requirement of their own, but they are also simply historical and out of scope for regeneration).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Occurrences generated, no conflicts | Every candidate date's occurrence overlaps zero confirmed bookings | New Time Block occurrence records created out to the current booking_horizon | New occurrences appear on FEAT-17.SPEC-002 and the Pro's schedule (FEAT-12) on next view | FEAT-17.SPEC-002, FEAT-12 |
| One or more occurrences conflict | A candidate date's occurrence overlaps a confirmed booking | Non-conflicting candidates commit; the conflicting one is held pending | Talia is routed to FEAT-17.SPEC-003 for the conflicting occurrence | FEAT-17.SPEC-003 |
| Pattern edited, occurrences regenerated | Talia changes span, day-of-week, or label on an existing pattern | Not-yet-elapsed, not-yet-conflict-resolved future occurrences are deleted and regenerated under the new definition | Updated occurrences reflect the new definition on next view of FEAT-17.SPEC-002 | FEAT-17.SPEC-002, FEAT-12 |
| Horizon extended, more occurrences generated | The Pro Account's booking_horizon increases | Additional future occurrences generated up to the new horizon | New further-out occurrences appear on next view | FEAT-17.SPEC-002 |
| No action needed | Every date within the current horizon already has a generated occurrence | None | None -- silent, logged internally | -- |
| Generation failure | A processing error prevents an occurrence from being created for one or more candidate dates | No occurrence created for the failed date(s); other candidate dates are unaffected | No blocking user feedback -- the gap is not user-visible until it would matter; surfaced to the Pro only indirectly if a client later books a slot the Pro expected blocked, which is out of this automation's scope to detect | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Time Block (the parent pattern's definition: day-of-week, start/end time-of-day, label; existing generated occurrences to avoid duplicate generation); Availability Rule -- booking_horizon field, per Pro Account; Booking -- state, start_time, duration, for the conflict check on each candidate date.
**Creates:** Time Block occurrence records -- one per generated future date, referencing the parent pattern.
**Updates:** None to the parent pattern record itself; regeneration deletes and recreates affected future occurrence records.
**Deletes:** Not-yet-elapsed, not-yet-conflict-resolved future occurrence records, when the parent pattern is edited (step 7).

## Business Rules

- Occurrences are generated only out to the Pro Account's current booking_horizon (Availability Rule) -- matching the range within which any slot can be booked at all, so a block is never generated for a date no client could book against anyway.
- Each generated occurrence is checked against the same conflicting-booking definition as a single-date block (FEAT-17.SPEC-008) -- recurrence introduces no separate conflict rule.
- An occurrence whose conflict Talia has already resolved (cancel, reschedule, or keep-as-exception) is never silently deleted or regenerated by a later pattern edit; only not-yet-resolved future occurrences are replaced.
- Regeneration never touches an occurrence whose own end time has already passed -- past occurrences are historical, per the entity's own no-retention lifecycle.

## Edge Cases

- **A candidate date falls on a day the Pro Account's booking_horizon does not yet reach** -- Not generated in this run; generated automatically once the rolling generation check runs again and the horizon (relative to today) reaches that date.
- **Talia's booking_horizon shrinks (a narrower horizon is set)** -- Already-generated occurrences beyond the new, shorter horizon are left in place (a Pro-set horizon change never deletes an already-committed block); no new occurrences are generated beyond the new horizon going forward.
- **A pattern is edited while one of its future occurrences has an unresolved conflict on FEAT-17.SPEC-003** -- That occurrence is left untouched by the regeneration (its resolution takes priority); it is included in step 7's exclusion because it has not yet had a conflict resolved by Talia at the moment of the edit, so it is preserved rather than deleted out from under an in-progress review.
- **Concurrent trigger firing (a pattern edit and the rolling generation check for the same pattern run at effectively the same time)** -- The pattern edit's regeneration (steps 2--7) is authoritative and its result is what persists; the rolling check's own run against the same pattern, if it started first, has its results superseded once the edit's regeneration commits.
- **Trigger fires while a previous generation run for the same pattern is still in flight** -- A second run for the same pattern is not started until the first completes; a run for a different pattern proceeds independently.
- **A pattern's every candidate date within the horizon already conflicts with the same recurring confirmed booking** -- Each conflicting date is routed to FEAT-17.SPEC-003 independently as its own occurrence; Talia resolves each one separately, since the Brief provides no bulk-recurring-conflict resolution.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Triggered by (inbound) | A saved or edited recurrence pattern triggers generation or regeneration |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Affects (outbound) | A conflicting generated occurrence routes here |
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | References (inbound) | Shares the same conflict-check logic, applied per candidate occurrence |
| FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Affects (outbound) | Generated occurrences are later retired there as their own end time passes |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Triggered by (inbound) | A booking_horizon change re-triggers generation |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Each committed occurrence is read live by the next slot computation |

## Analytics and Success Signals

- **time_block_added** (source: recurring occurrence generation) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **time_block_conflict_flagged** (source: recurring occurrence generation) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-005-AC-01:** Given Talia saves a new "every Sunday" pattern with a booking_horizon of several weeks, when this automation runs, then it generates one occurrence for each future Sunday within the horizon with zero conflicts.

**FEAT-17.SPEC-005-AC-02:** Given one candidate Sunday's occurrence conflicts with a confirmed booking, when generation processes that date, then Talia is routed to FEAT-17.SPEC-003 for that occurrence while the other Sundays' occurrences commit normally.

**FEAT-17.SPEC-005-AC-03:** Given Talia edits her existing pattern's end time, when the edit commits, then every not-yet-elapsed, not-yet-conflict-resolved future occurrence is regenerated with the new end time.

**FEAT-17.SPEC-005-AC-04:** Given an occurrence's own end time has already passed, when Talia edits the pattern, then that already-elapsed occurrence is left completely unchanged.

**FEAT-17.SPEC-005-AC-05:** Given Talia extends her booking_horizon, when the rolling generation check next runs, then additional future occurrences are generated out to the new horizon.

**FEAT-17.SPEC-005-AC-06:** Given Talia narrows her booking_horizon, when the change takes effect, then already-generated occurrences beyond the new horizon remain in place.

**FEAT-17.SPEC-005-AC-07:** Given a future occurrence has an unresolved conflict currently open on FEAT-17.SPEC-003, when Talia edits the parent pattern at that moment, then that specific occurrence is excluded from regeneration and left untouched.

**FEAT-17.SPEC-005-AC-08:** Given a generation run for a pattern is still in flight, when the rolling generation check fires again for the same pattern, then no second run starts until the first completes.

**FEAT-17.SPEC-005-AC-09:** Given every date within the horizon already has a generated occurrence, when the rolling generation check runs, then no new occurrences are created and nothing is shown to Talia.

**FEAT-17.SPEC-005-AC-10:** Given a processing error prevents one candidate date's occurrence from being created, when the run completes, then the other candidate dates' occurrences are unaffected and commit normally.

**FEAT-17.SPEC-005-AC-11:** Given a generated occurrence commits with zero conflicts, when the next slot computation runs, then FEAT-03.SPEC-001 reads it and removes that date's time from the bookable list.

**FEAT-17.SPEC-005-AC-12:** Given a pattern edit's regeneration and the rolling generation check for the same pattern run at effectively the same time, when both complete, then the pattern edit's result is what persists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 6 (generated, conflict, regenerated, horizon-extended, no-action, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

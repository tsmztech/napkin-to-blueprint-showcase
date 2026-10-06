---
document_type: spec
spec_type: automation
spec_id: FEAT-21.SPEC-004
spec_name: Occurrence Generation & Conflict Handling
spec_slug: occurrence-generation-conflict-handling
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Occurrence Generation & Conflict Handling

## Overview

**Name:** Occurrence Generation & Conflict Handling
**ID:** FEAT-21.SPEC-004
**Type:** Automation
**Purpose:** Generates each occurrence's Booking within the Pro's booking horizon as an Active series' due date arrives, subject to the same slot validation as any booking, and hands an occurrence whose usual time is no longer available to a client pick-a-new-time flow without breaking the rest of the series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Generating the next occurrence's Booking for every Active series as the booking horizon rolls forward to reach its due date
- Re-validating each candidate occurrence against the same slot check as any booking (FEAT-03.SPEC-004, XBR-01)
- Handing an occurrence whose usual time is no longer available to a client pick-a-new-time flow for that occurrence only
- Retrying a failed generation attempt and flagging a persistently failing one to the Pro

**Non-Goals:**
- Deciding whether a candidate interval or horizon is valid in the first place -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this spec applies that ceiling, it does not define it.
- Requesting or releasing an occurrence's deposit -- owned by FEAT-21.SPEC-005 (Occurrence Deposit Request & Release), which begins once this spec has created the occurrence's Booking.
- Ending a series or cancelling an occurrence -- owned by FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules); this spec only ever creates or time-shifts an occurrence, it never cancels one.
- The actual reservation and slot-hold mechanics themselves -- owned by FEAT-03 (Real-Time Slot Availability Engine); this spec triggers FEAT-03's ordinary validation and relies on the created Booking's own state to occupy the slot, per the mutual FEAT-03/FEAT-21 dependency this Brief's dependency map records.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Recurring Series is newly created | FEAT-21.SPEC-001 (Set Up Recurring Series) / FEAT-21.SPEC-010 (Pro Recurring Series Management) | Fires once, immediately after series creation, to generate the series' first occurrence(s) within the current booking horizon | Recurring Series (interval, originating_service, originating_time), Availability Rule (booking_horizon) |
| An Active series' booking horizon rolls forward | Schedule-based (daily evaluation against each Active series' next due date and the Pro's current booking_horizon) | Fires whenever a series' next due occurrence date now falls within the Pro's currently configured booking_horizon and has not yet been generated | Recurring Series (interval, originating_service, originating_time, generated_occurrences), Availability Rule (booking_horizon) |
| The client picks a new time for an occurrence flagged as needing one | FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) | Fires when the client submits a replacement time for an occurrence this spec previously flagged as conflicted | The flagged occurrence's Booking reference, the client's chosen replacement time |

## Processing Logic

1. On the initial-creation trigger, or on each scheduled horizon-rollforward run, identify every Active Recurring Series whose next due occurrence date (the last generated occurrence's date plus the series' interval, or the originating Booking's date plus the interval if none has been generated yet) now falls within the Pro's currently configured booking_horizon (FEAT-21.SPEC-003).
2. For each such series, construct the candidate occurrence: the series' originating_service, and a start time on the due date at the series' originating time-of-day.
3. Confirm the originating_service is still Active (not Archived). If it has been archived, take the Service-archived path (Step 9) instead of continuing.
4. Re-validate the candidate slot against FEAT-03.SPEC-004's rules exactly as any other booking (full duration plus buffer inside an open window, no conflicting booking, block, other recurring occurrence, or calendar busy time), applying no Pro-only exception -- occurrence generation follows the ordinary client-facing notice and horizon rules (XBR-01, XBR-03), since it stands in for a booking the client would otherwise have made themselves.
5. If the candidate slot passes validation, create a Booking in Pending Payment state: service, start_time, duration, and price agreed from the current Service definition at generation time; client set to the series' client; source set to "recurring occurrence"; a reference to the originating Recurring Series.
6. Add the new Booking to the series' generated_occurrences list.
7. Trigger FEAT-21.SPEC-007 (Occurrence Generated Notification) for the newly created Booking.
8. Hand off the Booking to FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) to begin its deposit-lifecycle monitoring.
9. **Service-archived path:** if the originating_service is Archived, generate no occurrence for this due date and take no further generation action for this series until the Pro either restores an equivalent active service or the client sets up a fresh series against a currently active service (FEAT-21.SPEC-001); flag the gap to the Pro on her dashboard (FEAT-12), since a standing client is due and nothing was booked for them.
10. **Conflict path:** if the candidate slot fails validation (the Pro changed her hours, added a block, or another commitment now occupies the time), still create the occurrence's Booking in Pending Payment state as in Step 5, but leave its start_time unset pending a replacement, and mark it as needing a new time.
11. Trigger FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) for the conflicted occurrence, asking the client to pick a new time for that occurrence only.
12. **Replacement-time path:** when the client submits a replacement time (via the flow FEAT-21.SPEC-008 links to), re-validate that specific candidate time against FEAT-03.SPEC-004 exactly as in Step 4. If it passes, set the occurrence Booking's start_time to the chosen time and proceed as a normally generated occurrence (Steps 6-8). If it fails, the client is shown the refreshed unavailable-time experience and asked to choose again -- the rest of the series is never affected by how many attempts this takes.
13. **Failure path:** if the generation attempt itself cannot complete (a processing error unrelated to slot validation), retry up to platform parameter: `occurrence-generation-retry-count` times. If it keeps failing after those retries, flag the gap on the Pro's dashboard (FEAT-12) as a missed occurrence needing her attention, rather than silently skipping the appointment.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Occurrence generated normally | Candidate slot passes validation | New Booking created (Pending Payment, source "recurring occurrence"); added to the series' generated_occurrences | Client receives the generated notification (FEAT-21.SPEC-007); occurrence appears Upcoming on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-005, FEAT-21.SPEC-007 |
| Occurrence needs a new time | Candidate slot fails validation (usual time no longer available) | New Booking created (Pending Payment, start_time unset, flagged needing new time) | Client receives the advance-notice notification (FEAT-21.SPEC-008) and is asked to pick a new time; occurrence appears "Needs new time" on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-008 |
| Replacement time accepted | Client's chosen replacement time passes validation | Booking's start_time set; occurrence proceeds as normally generated | Client receives the generated notification (FEAT-21.SPEC-007) for the now-scheduled occurrence | FEAT-21.SPEC-002, FEAT-21.SPEC-005, FEAT-21.SPEC-007 |
| Replacement time rejected | Client's chosen replacement time also fails validation | None | Client sees the refreshed unavailable-time experience and is asked to choose again | FEAT-21.SPEC-008 |
| Generation skipped -- horizon not yet reached | Series' next due date is still beyond the Pro's current booking_horizon | None | None -- this is silent, expected behavior each time the automation runs | -- |
| Generation skipped -- originating service archived | The series' originating_service is Archived at generation time | None (no Booking created for this due date) | No client-facing feedback; the Pro sees the gap flagged on her dashboard (FEAT-12) | FEAT-12 |
| Generation failure (after retries exhausted) | The generation attempt fails for reasons unrelated to slot validation, and retries are exhausted | None | No client-facing feedback; the Pro sees the gap flagged on her dashboard (FEAT-12) as a missed occurrence | FEAT-12 |

## Data Model

**Reads:** Recurring Series -- interval, originating_service, originating_time, state, generated_occurrences; Availability Rule -- booking_horizon (via FEAT-03); Service -- status, price, duration (at generation time); existing Slot Hold, Booking, Time Block, and Calendar Connection busy-time records (via FEAT-03.SPEC-004/FEAT-03.SPEC-006, for slot validation).
**Creates:** Booking -- service, start_time (or unset, if conflicted), duration, price_agreed, deposit_amount, client, source ("recurring occurrence"), Recurring Series reference, state Pending Payment.
**Updates:** Recurring Series -- generated_occurrences list (appended). Booking -- start_time (on replacement-time acceptance).
**Deletes:** None.

## Business Rules

- XBR-01: every generated occurrence is subject to the same live slot check as any other booking; the first commitment to a contested time wins, exactly as it would for a client-facing booking.
- XBR-02: a generated occurrence's slot is treated the same as any Pending Payment Booking for availability purposes -- it is never a separate reservation type, per the mutual FEAT-03/FEAT-21 dependency.
- XBR-03: occurrence generation follows the ordinary client-facing minimum-notice and booking-horizon rules; unlike a Pro-created deposit-request booking (FEAT-03.SPEC-007), no Pro-only exception applies here, since generation stands in for what the client would otherwise book herself.
- The booking-horizon ceiling this spec applies is always the Pro's currently configured Availability Rule.booking_horizon at the moment each generation attempt runs (FEAT-21.SPEC-003) -- never a value cached from series creation time.
- A conflicted occurrence's replacement-time flow never affects the series' other occurrences, its interval, or its state -- only that one occurrence's start_time is at stake (this Brief's Entity-Lifecycle Coverage Matrix, Update row).
- A repeatedly failing occurrence generation is retried (platform parameter: `occurrence-generation-retry-count`) and, if it keeps failing, surfaces to the Pro as a flagged gap on her dashboard rather than a silently missed appointment.

## Edge Cases

- **A series' due date arrives on a day the Pro has fully blocked with a Time Block** -- The candidate slot fails validation for the same reason any client-facing booking would; the occurrence takes the conflict path and the client is asked to pick a new time.
- **The originating service's price or duration has changed since the series was created** -- The generated occurrence uses the Service's current price and duration at generation time (Step 5), not the price agreed at the original booking, since each occurrence is a fresh booking in its own right, consistent with XBR-04 applying prospectively to each new booking rather than retroactively to a past one.
- **The originating service is archived and later a new, similarly named service is added** -- Generation for the existing series does not resume automatically against the new service, since the series' originating_service reference points to the specific archived Service, not a name; the client is left without occurrences until she sets up a fresh series (FEAT-21.SPEC-001) against the new service.
- **Two occurrences from different series happen to compete for the same slot on the same generation run** -- The first one processed in the run commits the slot; the second fails validation and takes the conflict path, per the same first-commit-wins rule that governs any two competing bookings (XBR-01).
- **Concurrent trigger firing (the initial-creation trigger and a scheduled horizon-rollforward run overlap for the same series)** -- A second generation attempt for a series already holding a due, ungenerated occurrence is a no-op: the series' generated_occurrences list is checked before creating a new Booking, preventing a duplicate occurrence for the same due date.
- **Trigger fires while a previous generation run is still in flight for the same series** -- Generation for a given series processes its due dates sequentially; a new trigger for that series queues behind the in-flight run rather than running concurrently against the same generated_occurrences list.
- **A client abandons the replacement-time flow without ever picking a new time** -- The occurrence's Booking remains in Pending Payment with start_time unset and status "Needs new time" indefinitely on FEAT-21.SPEC-002; it does not block the series from generating its next due occurrence, since the series' due-date calculation advances from the last successfully time-set occurrence, not from every attempted one -- this spec never auto-cancels an abandoned replacement flow, leaving that as the client's own choice via FEAT-21.SPEC-002's cancel action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Triggered by (inbound) | Series creation fires the first generation run |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Triggered by (inbound) / Affects (outbound) | A Pro's series set-up fires the first generation run; generated and conflicted occurrences appear there |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Supplies the booking-horizon ceiling this spec applies at every generation attempt |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Every candidate occurrence, and every replacement time, is validated against this spec's rules |
| FEAT-21.SPEC-002 (My Recurring Series) | Affects (outbound) | Newly generated and conflicted occurrences appear here |
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Affects (outbound) | A successfully generated occurrence's Booking is handed off here for deposit-lifecycle monitoring |
| FEAT-21.SPEC-007 (Occurrence Generated Notification) | Triggers (outbound) | A successful generation (initial or after a replacement time is accepted) fires this notification |
| FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice) | Triggers (outbound) / Triggered by (inbound) | A conflicted occurrence triggers this notification; the client's submitted replacement time re-enters this automation |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A persistently failing generation, or an archived-service gap, is flagged here |

## Analytics and Success Signals

- **recurring_occurrence_generated** (series reference, generation_outcome: normal / conflicted) -- supports success-metrics.md: "Zero Double-Booking Confidence" (every generated occurrence passes the same live slot check any booking does, so this event's outcome split is direct evidence the engine never double-books a standing appointment)
- **recurring_occurrence_conflict_flagged** (series reference) -- N/A -- no Stage 2 metric measures occurrence conflict frequency directly; retained so how often the Pro's own hours changes disrupt standing clients is observable.
- **recurring_occurrence_replacement_time_set** (series reference, attempts_needed) -- N/A -- no Stage 2 metric measures the replacement-time flow directly; retained to observe how often and how easily a conflicted occurrence resolves.
- **recurring_occurrence_generation_failed** (series reference, reason: service_archived / processing_error) -- N/A -- no Stage 2 metric measures generation failure directly; retained so a silently-missed standing appointment never goes unobserved, consistent with this Brief's Error state commitment.

## Acceptance Criteria

**FEAT-21.SPEC-004-AC-01:** Given Riley's series is newly created with an interval of 3 weeks, when the initial-creation trigger fires, then the first occurrence due within the Pro's current booking_horizon is generated as a Booking in Pending Payment state.

**FEAT-21.SPEC-004-AC-02:** Given an Active series' next due date now falls within the Pro's booking_horizon, when the scheduled horizon-rollforward run evaluates it, then the next occurrence is generated.

**FEAT-21.SPEC-004-AC-03:** Given a series' next due date is still beyond the Pro's booking_horizon, when the scheduled run evaluates it, then no occurrence is generated and nothing is shown to Riley.

**FEAT-21.SPEC-004-AC-04:** Given Talia has added a Time Block over an occurrence's usual due time, when generation is attempted, then the candidate slot fails validation, the occurrence is created flagged "Needs new time," and FEAT-21.SPEC-008 is triggered.

**FEAT-21.SPEC-004-AC-05:** Given a conflicted occurrence flagged "Needs new time," when Riley submits a replacement time that passes validation, then the occurrence's start_time is set, it proceeds as a normally generated occurrence, and FEAT-21.SPEC-007 fires for it.

**FEAT-21.SPEC-004-AC-06:** Given a conflicted occurrence, when Riley submits a replacement time that also fails validation, then she sees the refreshed unavailable-time experience and is asked to choose again, with the rest of her series unaffected.

**FEAT-21.SPEC-004-AC-07:** Given the series' originating service has been archived by the time an occurrence is due, when generation is attempted, then no occurrence is created for that due date and the gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-004-AC-08:** Given an occurrence's originating service's price has changed since the series was created, when the occurrence is generated, then it uses the Service's current price and duration, not the originally agreed ones.

**FEAT-21.SPEC-004-AC-09:** Given a generation attempt fails due to a processing error, when it is retried up to platform parameter: `occurrence-generation-retry-count` times and still fails, then the gap is flagged on Talia's dashboard as a missed occurrence.

**FEAT-21.SPEC-004-AC-10:** Given two different series each have a due occurrence competing for the same slot in the same generation run, when both are processed, then the first one processed commits the slot and the second takes the conflict path.

**FEAT-21.SPEC-004-AC-11:** Given a series already holds a due, ungenerated occurrence, when a second generation trigger fires for that same series before the first completes, then no duplicate occurrence is created.

**FEAT-21.SPEC-004-AC-12:** Given a generation run is already in flight for a series, when another trigger fires for that same series, then the new attempt queues behind the in-flight run rather than running concurrently.

**FEAT-21.SPEC-004-AC-13:** Given Riley abandons a conflicted occurrence's replacement-time flow, when the series' next due date is later evaluated, then the series still generates its next occurrence normally, unaffected by the abandoned one.

**FEAT-21.SPEC-004-AC-14:** Given a Booking is successfully generated for an occurrence, when the generation completes, then FEAT-21.SPEC-005 begins its deposit-lifecycle monitoring for that Booking.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |

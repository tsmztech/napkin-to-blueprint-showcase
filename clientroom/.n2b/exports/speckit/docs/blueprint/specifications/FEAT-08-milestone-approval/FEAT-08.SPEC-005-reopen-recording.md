---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-005
spec_name: Reopen Recording
spec_slug: reopen-recording
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Reopen Recording

## Overview

**Name:** Reopen Recording
**ID:** FEAT-08.SPEC-005
**Type:** Automation
**Purpose:** Writes Nadia's reopen of an approved milestone as a distinct, logged event, resetting the milestone to Reopened status while leaving the original approval record's `approved_at` and `approved_by` fields untouched.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Re-validating Nadia's authorization and the milestone's eligibility (status = Approved) at the moment of the Reopen attempt
- Atomically writing `status` -> "Reopened" without altering `approved_at` or `approved_by`
- Firing the Activity Log entry for the reopen event, which is how the event stays non-silent
- Making the milestone eligible again for Nadia's edits (FEAT-04) once reopened

**Non-Goals:**
- Rendering the Reopen control and the confirmation the freelancer sees before committing to reopen -- owned by FEAT-08.SPEC-002 (Milestone Reopen Screen), which this automation reports its outcome back to.
- Defining who may reopen and under what milestone state -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this automation enforces rather than restates.
- Sending any notification about the reopen -- the Brief's Communications field names no separate email for reopen; the append-only Activity Log entry this automation fires is how the event stays "non-silent" (Key Capabilities), not a notification.
- Reversing or cancelling an invoice that was already issued by the approval being reopened -- excluded per the Brief's Non-Goals: correcting or cancelling that invoice is owned entirely by FEAT-09 via a credit note or new invoice (XBR-04); this automation never touches the Invoice entity.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms reopen | FEAT-08.SPEC-002 (Milestone Reopen Screen) | Fires when Nadia confirms the reopen action on an approved milestone | The milestone reference, Nadia's authenticated identity, the current timestamp |

## Processing Logic

1. Receive the reopen request from FEAT-08.SPEC-002, carrying the milestone reference and Nadia's authenticated identity.
2. Confirm connectivity and an authenticated session are present; if either is missing, stop and return the connectivity/session failure outcome without touching the milestone record.
3. Re-check authorization per FEAT-08.SPEC-006: confirm the requesting identity is Nadia (the Freelancer) for this milestone's own account.
4. Read the milestone's current, live status.
5. If the live status is not "Approved," stop and return the already-changed outcome -- there is nothing to reopen.
6. If the live status is "Approved," write `status` -> "Reopened," leaving `approved_at` and `approved_by` exactly as they are.
7. Confirm the write committed successfully before reporting success -- an unconfirmed write is treated as a failure.
8. On a confirmed successful write, fire the append-only Activity Log entry to FEAT-13 (event type "milestone reopened," actor Nadia, the timestamp from step 6, the Milestone as the affected record) per XBR-05.
9. Return the outcome (reopened, already-changed, or failure) to FEAT-08.SPEC-002 for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Reopen recorded | Live status is "Approved" at the moment of the confirmed request | Milestone `status` -> "Reopened"; `approved_at` and `approved_by` unchanged | FEAT-08.SPEC-002 shows the milestone as Reopened; FEAT-08.SPEC-001 now shows the Approve control again instead of the "Approved on {date}" marker; FEAT-04.SPEC-001 shows the milestone as editable again | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-04.SPEC-001, FEAT-13 |
| Already changed (status is no longer Approved) | The milestone's status changed (e.g., a fresh approval cycle, or an earlier reopen already recorded) since Nadia's screen last loaded | None | FEAT-08.SPEC-002 shows a refresh: "This milestone's status has changed. Here's the current state." and reloads to the live status | FEAT-08.SPEC-002 |
| Connectivity/session failure | Connectivity or an authenticated session is not present at the moment of the attempt | None | FEAT-08.SPEC-002 shows a connectivity error; Nadia's confirmation never appears to succeed | FEAT-08.SPEC-002 |
| Write failure (system error after checks pass) | Eligibility checks pass but the write cannot be confirmed as committed | None -- the milestone is left in its pre-attempt "Approved" state | FEAT-08.SPEC-002 shows an error banner with a Retry option; retrying re-runs this automation from step 3 | FEAT-08.SPEC-002 |

## Data Model

**Reads:** Milestone -- `status`. Client Contact / Freelancer Account -- identity, to confirm Nadia is the requester.
**Creates:** None (the reopen event itself is captured as an Activity Log entry by FEAT-13, not as a new record on the Milestone entity).
**Updates:** Milestone -- `status` only, written on a successful outcome.
**Deletes:** None.

## Business Rules

- Reopen never rewrites approval history (FEAT-08.SPEC-006): this automation writes `status` only; `approved_at` and `approved_by` are never cleared, edited, or reset by a reopen.
- Reopen is Nadia-only (Key Capabilities: "Reopen (freelancer only)"): no client contact role can trigger this automation under any condition.
- Reopen requires the milestone to currently be "Approved" (FEAT-08.SPEC-006): there is no reopen of a milestone that was never approved, and no double-reopen of one already Reopened.
- A reopen is itself a logged, non-silent event (XBR-05, Key Capabilities): the Activity Log entry in step 8 is the mechanism that satisfies "never a silent edit" -- there is no separate reopen confirmation sent to any client contact, since reopening is a freelancer-side correction, not a client-facing event.
- Once reopened, the milestone becomes eligible again for Nadia's edits in FEAT-04 (feature-dependency-map.md, Cross-Feature Touchpoints): her edit attempts on the still-Approved milestone were refused until this automation's write commits.

## Edge Cases

- **Nadia confirms reopen twice in rapid succession (double-submit)** -- The first confirmation's write begins moving `status` out of "Approved" immediately; the second is evaluated against the now-"Reopened" state in step 5 and returns the already-changed outcome, never recording a second reopen event.
- **The milestone is re-approved (a new approval cycle) between Nadia's screen load and her reopen confirmation** -- Refused as already-changed only if status moved away from "Approved" in the interim; if it is still "Approved" (just a fresh approval cycle with new `approved_at`/`approved_by`), the reopen proceeds normally against the current approval.
- **Connectivity drops between Nadia's confirmation and the write's confirmation** -- Treated as a write failure; the milestone remains "Approved" and Nadia sees the connectivity/error state rather than an ambiguous "maybe reopened" state.
- **Concurrent trigger firing -- Nadia reopens the same milestone from two of her own sessions at effectively the same time** -- Whichever request's write commits first wins; the second request's check in step 5 then finds the milestone already "Reopened" and returns the already-changed outcome. Exactly one reopen event is ever recorded.
- **A reopen request arrives while a previous reopen run for the same milestone is still in flight** -- The second request's read of live status either observes the pre-write "Approved" state (and, if it also attempts to write, is subject to the same single-writer commit guarantee so only one write ultimately commits) or observes the already-"Reopened" state (and returns already-changed). No interleaving produces two reopen events for one approval cycle.
- **The Activity Log write (step 8) fails after the Milestone write (step 6-7) already succeeded** -- Non-blocking: the reopen itself stands (the milestone shows "Reopened," FEAT-04 edits become available); the audit-trail write is retried by FEAT-13's own handling, since the reopen's non-silence depends on that entry eventually landing, but a transient logging failure must never unwind an already-committed status change.
- **Nadia reopens a milestone and immediately attempts to edit it before the reopen's write is confirmed** -- FEAT-04.SPEC-003's edit-lock rule still reads "Approved" until this automation's write commits; her edit is refused until the reopen is confirmed, after which the same edit attempt succeeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Milestone Reopen Screen) | Triggered by (inbound) | Nadia's confirmed reopen action fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Defines the Reopen authorization and eligibility rules this automation enforces |
| FEAT-08.SPEC-002 (Milestone Reopen Screen) | Affects (outbound) | Returns the reopened, already-changed, connectivity, or failure outcome for display |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Affects (outbound) | A successful reopen returns the milestone to an unapproved state, re-enabling the Approve control there |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | Affects (outbound) | A successful reopen lifts the edit-lock, making Nadia's edits on this milestone allowed again |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the reopen event (XBR-05) |

## Analytics and Success Signals

- **milestone_reopened_by_freelancer** (milestone reference, time since original approval) -- N/A -- no Stage 2 metric measures reopen frequency directly; retained because the Brief's Key Capabilities treat "a logged, non-silent way to reopen" as a first-class product guarantee, and this event is the only signal that the guarantee is being exercised as intended rather than never used or, at the other extreme, used so often it signals an approval-quality problem worth the founder's attention.
- **milestone_reopen_refused_already_changed** (milestone reference) -- N/A -- no Stage 2 metric measures refused reopen attempts; retained for the same observability reason as the analogous refused-approval event in FEAT-08.SPEC-003.

## Acceptance Criteria

**FEAT-08.SPEC-005-AC-01:** Given Nadia confirms reopen on a milestone whose live status is "Approved," when the write commits, then `status` becomes "Reopened" and `approved_at`/`approved_by` remain unchanged.

**FEAT-08.SPEC-005-AC-02:** Given a successful reopen write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone reopened," Nadia as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-005-AC-03:** Given a successful reopen, when Owen next opens FEAT-08.SPEC-001 for that milestone, then he sees the Approve control again instead of "Approved on {date}."

**FEAT-08.SPEC-005-AC-04:** Given a successful reopen, when Nadia opens FEAT-04.SPEC-001 for that milestone, then its edit controls are enabled again.

**FEAT-08.SPEC-005-AC-05:** Given the milestone's status is no longer "Approved" by the time Nadia's confirmed request is processed, when the check in step 5 runs, then the attempt returns the already-changed outcome with no write.

**FEAT-08.SPEC-005-AC-06:** Given Nadia has no connectivity at the moment she confirms reopen, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-005-AC-07:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Nadia sees the resulting error, then she can retry, and the retry re-evaluates eligibility fresh.

**FEAT-08.SPEC-005-AC-08:** Given Nadia confirms reopen twice in rapid succession, when the first confirmation's write is already committing, then the second is refused as already-changed and only one reopen event is ever recorded.

**FEAT-08.SPEC-005-AC-09:** Given Nadia reopens the same milestone from two of her own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other returns the already-changed outcome.

**FEAT-08.SPEC-005-AC-10:** Given the Milestone write for a reopen succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the reopen itself still stands, and the audit-trail write is retried by FEAT-13's own handling.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (reopened, already-changed, connectivity failure, write failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

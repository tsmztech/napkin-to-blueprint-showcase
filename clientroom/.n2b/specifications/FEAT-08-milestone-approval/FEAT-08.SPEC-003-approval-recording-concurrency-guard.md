---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-003
spec_name: Approval Recording & Concurrency Guard
spec_slug: approval-recording-concurrency-guard
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Approval Recording & Concurrency Guard

## Overview

**Name:** Approval Recording & Concurrency Guard
**ID:** FEAT-08.SPEC-003
**Type:** Automation
**Purpose:** Records a milestone's approval exactly once, atomically writing the immutable timestamp and approving contact's identity, and refuses the attempt -- showing the refreshed milestone instead -- if the milestone changed underneath the approving contact since it was shown.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Re-validating Owen's authorization and the milestone's eligibility at the moment of the Approve attempt, not just at screen load
- Atomically writing `status` (-> Approved), `approved_at`, and `approved_by` exactly once
- Detecting and refusing an approval attempt made against a stale milestone view (reject-with-refresh)
- Firing the downstream next-invoice trigger and confirmation notification on a successful write
- Firing the Activity Log entry for the approval event

**Non-Goals:**
- Rendering the Approve control, the loading/offline/error states the client sees, and the client-visible retry affordance -- owned by FEAT-08.SPEC-001 (Milestone Review & Approval Screen), which this automation reports its outcome back to.
- Defining who may approve, under what milestone state, and what "already approved" or "stale" means -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this automation enforces rather than restates.
- Generating the next invoice itself -- owned by FEAT-08.SPEC-004 (Next-Invoice Trigger), which this automation only fires on success; invoice numbering, content, and sending belong entirely to FEAT-09.
- Recording a reopen or any reversal of an approval -- owned by FEAT-08.SPEC-005 (Reopen Recording); this automation has no path back out of "Approved" once it writes.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Owen taps Approve | FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Fires whenever the Approve control is actionable and Owen taps it, whether or not the underlying milestone has changed since the screen loaded | The milestone reference and the version/state Owen's screen last loaded (for the staleness check), Owen's authenticated Client Contact identity, the current timestamp |

## Processing Logic

1. Receive the Approve request from FEAT-08.SPEC-001, carrying the milestone reference, the milestone state as last loaded by Owen's screen, and Owen's authenticated Client Contact identity.
2. Confirm connectivity and an authenticated session are present; if either is missing, stop and return the connectivity/session failure outcome without touching the milestone record.
3. Re-check authorization per FEAT-08.SPEC-006: confirm the requesting identity is a Primary Contact (Owen's role) for the milestone's own client company. If this check fails, stop and return the unauthorized outcome.
4. Read the milestone's current, live state (not the state Owen's screen last loaded).
5. Compare the live state against the state Owen's screen last loaded, specifically: `status`, and whether the current Deliverable and its price/eligibility fields differ from what was shown.
6. If the live state differs from what was shown (the milestone was re-priced, its deliverable was removed, or `status` is no longer "Deliverable Uploaded" or "Reopened"), stop and return the stale-attempt outcome -- no write occurs.
7. If the live state matches, atomically write, in a single operation: `status` -> "Approved", `approved_at` -> the current timestamp, `approved_by` -> Owen's identity.
8. Confirm the write committed successfully before reporting success -- a write that cannot be confirmed as committed is treated as a failure, never as an assumed success.
9. On a confirmed successful write, fire FEAT-08.SPEC-004 (Next-Invoice Trigger) and FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification).
10. On a confirmed successful write, fire the append-only Activity Log entry to FEAT-13 (event type "milestone approved," actor Owen, the timestamp from step 7, the Milestone as the affected record) per XBR-05.
11. Return the outcome (success, stale, unauthorized, or failure) to FEAT-08.SPEC-001 for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Approval recorded | Live state matches what Owen's screen loaded; write commits and is confirmed | Milestone `status`, `approved_at`, `approved_by` set atomically | FEAT-08.SPEC-001 shows "Approved on {date}" in place of the Approve control | FEAT-08.SPEC-001, FEAT-08.SPEC-004, FEAT-08.SPEC-007, FEAT-13 |
| Stale attempt refused | Milestone was re-priced, its deliverable removed, or its status changed since Owen's screen loaded | None -- no partial write | FEAT-08.SPEC-001 shows a refresh dialog: "This milestone has changed since you opened it. Here's the current version." and reloads to the live state; Owen's tap is not recorded as an approval | FEAT-08.SPEC-001 |
| Unauthorized | Requesting identity is not a Primary Contact for the milestone's own client company, or the milestone is already "Approved" at the time of the attempt | None | Treated identically to the stale-attempt outcome from Owen's perspective when caused by a race (already approved by the time this request lands); if reached through any other path, no control was ever shown to begin with (FEAT-08.SPEC-006) | FEAT-08.SPEC-001 |
| Connectivity/session failure | Connectivity or an authenticated session is not present at the moment of the attempt | None | FEAT-08.SPEC-001 shows the offline "reconnect to approve" state; Owen's tap never appears to succeed | FEAT-08.SPEC-001 |
| Write failure (system error after checks pass) | All eligibility checks pass, but the write itself cannot be confirmed as committed | None -- the milestone is left in its pre-attempt state, never partially written | FEAT-08.SPEC-001 shows an error banner with a Retry option; retrying re-runs this automation from step 3, so a retried attempt is never double-recorded | FEAT-08.SPEC-001 |

## Data Model

**Reads:** Milestone -- `status`, `price`, `no_separate_charge`, current Deliverable reference and its status, for the staleness comparison. Client Contact -- role and client-company reference, to establish Owen's identity and authorization.
**Creates:** None.
**Updates:** Milestone -- `status`, `approved_at`, `approved_by`, written together in one atomic operation on a successful outcome only.
**Deletes:** None.

## Business Rules

- Exactly-once and immutability (FEAT-08.SPEC-006): a confirmed write is never issued a second time against the same pre-approval state; `approved_at` and `approved_by` are never altered by any subsequent action other than a future reopen-then-approve cycle.
- Reject-with-refresh, never last-write-wins or merge, on the Milestone entity (feature-dependency-map.md, Milestone Contention note): the comparison in step 5 is authoritative, and any mismatch refuses the write rather than attempting to reconcile it.
- Approval requires connectivity (FEAT-08.SPEC-006): this automation never queues an approval for later delivery -- a request received without a confirmed connection and session is treated as a connectivity failure, not a pending approval.
- A failed approval action is retried by Owen without double-recording (product-features.md, States: Error): because the write is a single atomic, idempotent-by-precondition operation, retrying after a write failure re-evaluates eligibility fresh and cannot produce two approval records for the same cycle.
- The next-invoice trigger and the confirmation notification fire only after the write is confirmed committed (step 8) -- never optimistically before commit, so a failed or stale attempt can never issue an invoice or send a confirmation.

## Edge Cases

- **Owen retries after a write-failure error banner** -- The retry re-runs the full check from step 3; if the milestone is still eligible, the retry succeeds and records exactly one approval; if another approval was recorded in the meantime, the retry is refused as a stale attempt.
- **The milestone's deliverable is removed by Nadia between screen load and Owen's tap** -- Refused as a stale attempt per step 6; Owen sees the refreshed milestone, which has no reviewable deliverable and no Approve control.
- **The milestone is re-priced by Nadia between screen load and Owen's tap, with no status change** -- Still refused as a stale attempt: any live-state mismatch from what Owen was shown refuses the write, not only a status change, so his approval is never recorded against pricing he did not actually see.
- **Connectivity drops between Owen's tap and the write's confirmation** -- Treated as a write failure (step 8): the automation does not assume success without confirmation, and Owen sees the error/offline state rather than a false "Approved" confirmation.
- **Concurrent trigger firing -- Owen taps Approve on two of his own sessions at effectively the same time** -- Both requests reach step 4-6; whichever commits first wins, moving `status` out of the eligible range. The second request's comparison in step 6 then finds a mismatch (status is already "Approved") and is refused as a stale attempt, showing the now-Approved milestone. Exactly one approval is ever recorded.
- **A second Approve request arrives while the first is still mid-write (trigger fires while a previous run is in flight)** -- The second request's read of live state (step 4) either observes the pre-write state (and, if it then wins the race to write, is caught by the same single-writer commit guarantee so only one write ultimately commits) or observes the already-committed state (and is refused as stale). No interleaving of the two requests can produce two committed approvals or a partially-written record.
- **The Activity Log write (step 10) fails after the Milestone write (step 7-8) already succeeded** -- Non-blocking: the approval itself stands (Owen sees "Approved on {date}," the next invoice still fires); the audit-trail write is retried by FEAT-13's own handling, per the pattern used elsewhere in this product for a downstream logging failure that must never unwind an already-committed evidentiary record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Triggered by (inbound) | Owen's Approve tap fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Defines the authorization, eligibility, exactly-once, immutability, and concurrency rules this automation enforces |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Affects (outbound) | Returns the success, stale, unauthorized, connectivity, or failure outcome for display |
| FEAT-08.SPEC-004 (Next-Invoice Trigger) | Affects (outbound) | Fired immediately after a confirmed successful approval write |
| FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification) | Affects (outbound) | Fired immediately after a confirmed successful approval write |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (inbound) | This automation's staleness check reads the same Milestone state that FEAT-04.SPEC-003's edit-lock rule protects |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the approval event (XBR-05) |

## Analytics and Success Signals

- **milestone_approval_recorded** (time from deliverable-ready to approval, milestone reference) -- supports success-metrics.md: "Milestone Approval Turnaround"
- **milestone_approval_refused_stale** (reason: re-priced / deliverable-removed / already-approved) -- N/A -- no Stage 2 metric measures refused attempts directly; retained so a client-visible refusal remains observable rather than a silent dead end, since a repeatedly refused approval would otherwise look like a slow turnaround with no diagnosable cause.
- **milestone_approval_write_failed** (retry_count) -- N/A -- no Stage 2 metric covers infrastructure write failures; retained to distinguish a genuinely slow approval from one blocked by a technical failure, which "Milestone Approval Turnaround" alone cannot tell apart.

## Acceptance Criteria

**FEAT-08.SPEC-003-AC-01:** Given Owen taps Approve on a milestone whose live state matches what his screen loaded, when the write commits, then `status`, `approved_at`, and `approved_by` are all set together and FEAT-08.SPEC-001 shows "Approved on {date}."

**FEAT-08.SPEC-003-AC-02:** Given a successful approval write, when it commits, then FEAT-08.SPEC-004 (Next-Invoice Trigger) and FEAT-08.SPEC-007 (Confirmation Notification) both fire.

**FEAT-08.SPEC-003-AC-03:** Given a successful approval write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone approved," Owen as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-003-AC-04:** Given Nadia removed the milestone's deliverable after Owen's screen loaded, when Owen taps Approve, then the attempt is refused as stale and Owen is shown the refreshed milestone with no write to `status`, `approved_at`, or `approved_by`.

**FEAT-08.SPEC-003-AC-05:** Given Nadia re-priced the milestone after Owen's screen loaded, with no status change, when Owen taps Approve, then the attempt is still refused as stale.

**FEAT-08.SPEC-003-AC-06:** Given the milestone is already "Approved" by the time Owen's request reaches this automation, when the comparison in step 6 runs, then the attempt is refused and Owen is shown the current "Approved on {date}" state.

**FEAT-08.SPEC-003-AC-07:** Given Owen has no connectivity at the moment he taps Approve, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-003-AC-08:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Owen sees the resulting error, then he can retry, and the retry re-evaluates eligibility fresh rather than assuming the prior attempt partially succeeded.

**FEAT-08.SPEC-003-AC-09:** Given Owen taps Approve twice in rapid succession from the same session, when the first tap's write is already committing, then the second tap's request is refused as stale, showing the now-Approved milestone, and only one approval is ever recorded.

**FEAT-08.SPEC-003-AC-10:** Given Owen approves the same milestone from two of his own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other is refused as stale.

**FEAT-08.SPEC-003-AC-11:** Given a second Approve request arrives while a first request's write is still in flight, when both are processed, then no interleaving produces two committed approvals or a partially written record.

**FEAT-08.SPEC-003-AC-12:** Given the Milestone write for an approval succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the approval itself still stands (Owen still sees "Approved on {date}," the invoice trigger still fires), and the audit-trail write is retried by FEAT-13's own handling.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (recorded, stale, unauthorized, connectivity failure, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

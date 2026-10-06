---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-009
spec_name: Discard Draft
spec_slug: discard-draft
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 6
---

# Automation Spec: Discard Draft

## Overview

**Name:** Discard Draft
**ID:** FEAT-02.SPEC-009
**Type:** Automation
**Purpose:** Permanently removes an unsent Draft proposal that Nadia no longer wants to keep.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Hard-deleting a Draft-status Proposal record
- Refusing to delete a proposal that is not in Draft status

**Non-Goals:**
- Soft-deleting or archiving a Sent, Voided, or Accepted proposal -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: those statuses are evidentiary (XBR-04) and are retained for the life of the account, removed only by FEAT-24 account deletion.
- Providing a restore or undo path after discard -- a discarded draft was never sent and carries no evidentiary value, so no retention or recovery window applies; the confirmation dialog (FEAT-02.SPEC-001, FEAT-02.SPEC-003) is the only safeguard before permanent removal.
- Cascading deletes to other entities -- nothing references an unsent Draft (dependency map: Entity-Lifecycle Coverage Matrix), so no cascade logic exists in this automation.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms "Discard Draft" | FEAT-02.SPEC-001 (Proposal Draft Editor) | The proposal is currently in Draft status | Proposal id, owning project reference |
| Nadia confirms "Discard" | FEAT-02.SPEC-003 (Proposal Detail) | The proposal is currently in Draft status | Proposal id, owning project reference |

## Processing Logic

1. Read the Proposal's current status; if it is not Draft, stop and report a state-mismatch failure (see Edge Cases).
2. Permanently delete the Proposal record.
3. Write an Activity Log Entry recording the discard (XBR-05), owned by FEAT-13.
4. Signal the triggering screen to show the project's empty proposal state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Discard succeeds | Proposal is currently Draft | Proposal record permanently removed | Triggering screen navigates to FEAT-02.SPEC-003 showing the empty state | FEAT-02.SPEC-001, FEAT-02.SPEC-003, FEAT-13 |
| Blocked -- state mismatch | The proposal is no longer Draft (e.g., sent from another session in the moment before this ran) | None | Triggering screen shows: "This proposal was just sent and can no longer be discarded." and reloads to show the current (Sent) state | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |
| Failure | Processing error (e.g., connectivity lost mid-operation) | None | Error banner: "Could not discard this draft. Try again." with a Retry option; the Draft is preserved | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |

## Data Model

**Reads:** Proposal -- current status.
**Creates:** Activity Log Entry (via FEAT-13) recording the discard.
**Updates:** None.
**Deletes:** Proposal record -- permanent, hard delete, Draft-status records only.

## Business Rules

- Discard is available only against a Draft-status proposal, per FEAT-02.SPEC-010 -- Sent, Voided, and Accepted proposals can never be discarded through this automation.
- The delete is permanent and immediate on confirmation -- there is no soft-delete state, retention window, or restore path for a discarded Draft, since it was never sent and carries no evidentiary value.
- Discarding a Draft does not affect any other proposal for the same project, since a project has at most one active proposal at a time -- after discard, the project simply has none.

## Edge Cases

- **The proposal was sent from another session in the instant before this automation's status check runs** -- Reported as the state-mismatch outcome; the now-Sent proposal is never deleted, and the triggering screen reloads to show it as Sent.
- **Concurrent trigger firing (Discard confirmed from two open sessions for the same Draft at effectively the same time)** -- Only the first to pass the Draft-status check performs the delete; the second finds the proposal already gone and receives the state-mismatch outcome (surfaced as the empty-state redirect, since a deleted record and a "someone else already discarded it" outcome are indistinguishable to the user and equally resolved by returning to the empty state).
- **A trigger fires while a previous discard for the same proposal is still in flight** -- The triggering screen's Discard control is disabled during the operation (FEAT-02.SPEC-001, FEAT-02.SPEC-003), preventing a second discard attempt from the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **Nadia discards the only Draft that was pre-filled from a copy (FEAT-02.SPEC-008)** -- The discard proceeds normally; the source proposal that was copied from is entirely unaffected, since copied_from is a one-time provenance reference, not a live dependency.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | "Discard Draft" action, after confirmation, triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Triggered by (inbound) / Affects (outbound) | "Discard" action triggers this automation; shows the resulting empty state |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Draft-only eligibility rule for the discard action |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the discard event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_draft_discarded** (project id, minutes since draft creation) -- supports success-metrics.md: "Proposal Send Speed" (a discarded draft that never sent is a data point on the drafting funnel, distinguishing abandoned attempts from the sent-proposal timing this metric targets)

## Acceptance Criteria

**FEAT-02.SPEC-009-AC-01:** Given a project has a Draft proposal, when Nadia confirms "Discard Draft" from FEAT-02.SPEC-001, then the Proposal record is permanently deleted and the screen navigates to the empty state.

**FEAT-02.SPEC-009-AC-02:** Given a project has a Draft proposal, when Nadia confirms "Discard" from FEAT-02.SPEC-003, then the Proposal record is permanently deleted and the empty state is shown.

**FEAT-02.SPEC-009-AC-03:** Given the proposal was sent from another session in the moment before this automation's status check runs, when the discard is attempted, then it is refused with "This proposal was just sent and can no longer be discarded." and the proposal is not deleted.

**FEAT-02.SPEC-009-AC-04:** Given a processing failure occurs during discard, then the error banner "Could not discard this draft. Try again." appears and the Draft is preserved.

**FEAT-02.SPEC-009-AC-05:** Given two sessions confirm Discard for the same Draft at effectively the same time, then only the first deletes the record and the second is redirected to the empty state without error.

**FEAT-02.SPEC-009-AC-06:** Given a Draft that was created from a copy (FEAT-02.SPEC-008) is discarded, then the source proposal it was copied from is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 (success, state mismatch, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

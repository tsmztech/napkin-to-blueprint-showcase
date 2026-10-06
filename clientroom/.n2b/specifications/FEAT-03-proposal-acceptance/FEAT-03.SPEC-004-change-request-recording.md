---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-004
spec_name: Change-Request Recording
spec_slug: change-request-recording
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Change-Request Recording

## Overview

**Name:** Change-Request Recording
**ID:** FEAT-03.SPEC-004
**Type:** Automation
**Purpose:** Writes Owen's change-request note as a Comment on the proposal and notifies Nadia immediately.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- Validating the note's length (1--2,000 characters) at write time
- Re-checking that the proposal is still eligible to receive a change request (not Voided, not Accepted) at the moment of write
- Writing the Comment record with `target: Proposal`, `text`, `author`, `posted_at`
- Firing the change-request notification (FEAT-03.SPEC-007) and the audit-trail entry (FEAT-13)

**Non-Goals:**
- Altering the proposal itself -- excluded per XBR-26: a request-changes note never changes the Proposal's `scope_description`, `price`, or `status`; it only creates a Comment referencing the proposal.
- Composing or sending the notification email's content -- owned by FEAT-03.SPEC-007 (Change-Request Notification), which this automation only triggers.
- General comment lifecycle (editing within a grace window, retraction) -- owned entirely by FEAT-07 (Deliverable Review & Feedback), per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix; this automation only creates the change-request variant of a Comment, once.
- Nadia's revision of the proposal in response to the note -- owned by FEAT-02 (Proposal Creation & Sending); this automation's responsibility ends once the note is recorded and Nadia is notified.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Owen taps Send | FEAT-03.SPEC-002 (Request Changes) | Fires after the screen's own length check passes (1--2,000 characters, non-empty) | Proposal reference, note text, Owen's Client Contact identity, current timestamp |

## Processing Logic

1. Receive the proposal reference, note text, and Owen's Client Contact identity from FEAT-03.SPEC-002.
2. Re-validate the note length at write time: must be 1--2,000 characters. This re-check is authoritative over the screen-time check, in case the text was altered in transit or the screen check was bypassed.
3. If the length re-check fails, stop and return the "invalid note" outcome (no write performed) -- this should not normally occur since the screen already checked, but guards against a stale or tampered submission.
4. Re-check the proposal's current `status`: it must not be `Voided` and must not be `Accepted` (a change request against an already-decided proposal has nothing left to request changes on).
5. If the proposal is `Voided`, stop and return the "voided -- redirect" outcome (no write performed).
6. If the proposal is `Accepted`, stop and return the "already accepted" outcome (no write performed).
7. If both checks pass, write a new Comment record: `target: Proposal` (the proposal reference), `text` (the note), `author` (Owen's Client Contact reference), `posted_at` (current timestamp), `status: Posted`. This is a create-only write -- no existing Comment is modified.
8. Fire the audit-trail entry to FEAT-13 (XBR-05) with event type "change request submitted," Owen as actor, the timestamp from step 7, and the Proposal as the affected record.
9. Fire FEAT-03.SPEC-007 (Change-Request Notification) to notify Nadia immediately.
10. Return the success outcome to FEAT-03.SPEC-002, which shows the confirmation and navigates back to FEAT-03.SPEC-001.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Note recorded | Both re-checks pass | New Comment created (target: Proposal, text, author, posted_at) | FEAT-03.SPEC-002 shows "Your note has been sent to Nadia" and returns to FEAT-03.SPEC-001 | FEAT-03.SPEC-002, FEAT-03.SPEC-007, FEAT-13 |
| Invalid note (length) | Write-time length re-check fails | None | FEAT-03.SPEC-002 shows the field-level length error and does not navigate away | FEAT-03.SPEC-002 |
| Voided -- redirect | Write-time status re-check finds `Voided` | None | FEAT-03.SPEC-002 shows a message directing Owen to the current proposal version; note text preserved for re-submission | FEAT-03.SPEC-002 |
| Already accepted | Write-time status re-check finds `Accepted` | None | FEAT-03.SPEC-002 shows a message that the proposal has already been accepted, with navigation back to FEAT-03.SPEC-001 (now showing Accepted) | FEAT-03.SPEC-002, FEAT-03.SPEC-001 |
| Write failure (connectivity or processing error) | The write itself does not complete | None | FEAT-03.SPEC-002 shows an inline error with a retry option; note text preserved | FEAT-03.SPEC-002 |

## Data Model

**Reads:** Proposal -- `status` (to confirm eligibility at write time). Client Contact -- Owen's identity and `role`.
**Creates:** Comment -- `target: Proposal`, `text` (1--2,000 characters), `author` (Owen's Client Contact reference), `posted_at`, `status: Posted`.
**Updates:** None -- the proposal itself is never modified by this automation (XBR-26).
**Deletes:** None.

## Business Rules

- XBR-26: A request-changes note from the Primary contact is recorded as a comment on the proposal, notifies the freelancer immediately, and never alters the proposal itself.
- XBR-05: The change-request event writes an append-only Activity Log Entry with actor and timestamp.
- Note length (1--2,000 characters) is the single validation rule this automation enforces at write time, mirroring FEAT-03.SPEC-005 and FEAT-03.SPEC-002's screen-level check.
- A change request can only be submitted against a proposal that is neither Voided nor Accepted -- consistent with FEAT-03.SPEC-005's eligibility rules for actions on a proposal.
- The Comment created here follows the dependency map's Comment Contention note ("None -- each comment is written and retracted only by its own author"): this automation only ever creates a new Comment, never modifies an existing one, so no concurrent-write conflict on the Comment entity itself can arise from this automation.

## Edge Cases

- **Owen submits a note exceeding 2,000 characters via a route that bypassed the screen's own live check (e.g., a stale form state)** -- The write-time re-check in step 2 catches this and returns "invalid note"; no Comment is created.
- **The proposal is voided by Nadia between Owen tapping Send on FEAT-03.SPEC-002 and this automation's write-time check** -- The re-check in step 4 is authoritative: it finds `Voided` and returns "voided -- redirect," with no Comment created.
- **The proposal is accepted (by Owen through another session, or by another Primary contact) between Send and this automation's write-time check** -- The re-check in step 4 finds `Accepted` and returns "already accepted," with no Comment created.
- **Two change-request notes are submitted in quick succession by Owen (e.g., a double-tap on Send)** -- Concurrent trigger firing: FEAT-03.SPEC-002 debounces the Send button while a send is in flight, so a second automation run for the same submission cannot start until the first completes; if both nonetheless reached this automation, each independently creates its own Comment (append-only, no conflict), consistent with the dependency map's Comment Contention note that many authors' comments on the same thread need no resolution beyond ordering by posted time.
- **A change-request submission is still in flight (processing) when Owen navigates back to FEAT-03.SPEC-001 and returns to FEAT-03.SPEC-002 to submit another note before the first completes** -- Trigger fires while a previous run is in flight: each submission is processed independently against its own note text; the first run's write, once it completes, does not block or invalidate the second's eligibility re-check, since the proposal's eligibility for a change request does not change as a result of a Comment being created (Comments do not affect Proposal `status`).
- **The audit-trail trigger to FEAT-13 fails to fire after the Comment write has already succeeded** -- Non-blocking: the Comment record and the notification to Nadia are unaffected; the audit-trail write is retried by FEAT-13's own handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-002 (Request Changes) | Triggered by (inbound) | Send button tap, after screen-level length validation passes |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (inbound) | Note length and eligibility rules this automation re-checks and enforces at write time |
| FEAT-03.SPEC-007 (Change-Request Notification) | Triggers (outbound) | Fires the notification email to Nadia once the note is written |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the change-request event (XBR-05) |
| FEAT-02 (Proposal Creation & Sending) | References (outbound) | Nadia later opens the recorded note when she revises and re-sends the proposal |

## Analytics and Success Signals

- **proposal_changes_requested** (proposal reference, note character count) -- N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute this signal to
- **change_request_write_failed** (failure reason: invalid-note / voided / already-accepted / connectivity) -- N/A -- no connected success metric measures write failures; recorded as diagnostic-only exhaust from this automation's write path

## Acceptance Criteria

**FEAT-03.SPEC-004-AC-01:** Given Owen submits a valid note (1--2,000 characters) on a proposal in Sent status, when this automation runs, then a Comment is created with `target: Proposal`, the note text, Owen as author, and the current timestamp.

**FEAT-03.SPEC-004-AC-02:** Given the Comment write succeeds, when this automation completes, then it fires FEAT-03.SPEC-007 to notify Nadia and fires the FEAT-13 audit-trail entry for the change-request event.

**FEAT-03.SPEC-004-AC-03:** Given a note somehow reaches this automation exceeding 2,000 characters, when the write-time length re-check runs, then no Comment is created and the "invalid note" outcome is returned.

**FEAT-03.SPEC-004-AC-04:** Given Owen submits a note on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no Comment is recorded and FEAT-03.SPEC-002 directs Owen to the current version.

**FEAT-03.SPEC-004-AC-05:** Given Owen submits a note on a proposal that is accepted (by himself in another session, or by another Primary contact) before this automation's write-time re-check runs, when the re-check finds `status: Accepted`, then no Comment is recorded and FEAT-03.SPEC-002 shows that the proposal has already been accepted.

**FEAT-03.SPEC-004-AC-06:** Given Owen submits a note and connectivity drops before the write completes, when the failure occurs, then no Comment is created and FEAT-03.SPEC-002 shows a retry option with the note text preserved.

**FEAT-03.SPEC-004-AC-07:** Given Owen submits two change-request notes on the same proposal in quick succession, when both reach this automation, then each is recorded as its own independent Comment with no conflict, ordered by posted time.

**FEAT-03.SPEC-004-AC-08:** Given a change-request note is successfully recorded, when the analytics signal is emitted, then a proposal_changes_requested event carries the proposal reference and the note's character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (recorded, invalid note, voided-redirect, already accepted, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 (invalid-note-race, void-race, accept-race, double-submit, run-in-flight, audit-trigger-failure) | 6 |

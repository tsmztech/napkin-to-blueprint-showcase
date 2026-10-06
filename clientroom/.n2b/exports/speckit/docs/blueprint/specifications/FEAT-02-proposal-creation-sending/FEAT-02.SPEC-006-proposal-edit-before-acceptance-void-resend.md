---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-006
spec_name: Proposal Edit-Before-Acceptance Void & Resend
spec_slug: proposal-edit-before-acceptance-void-resend
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Proposal Edit-Before-Acceptance Void & Resend

## Overview

**Name:** Proposal Edit-Before-Acceptance Void & Resend
**ID:** FEAT-02.SPEC-006
**Type:** Automation
**Purpose:** When Nadia edits a Sent-but-unaccepted proposal, voids the prior version and creates and sends the new one, so the client is never shown an outdated price (XBR-06).
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-validating the edited content and send eligibility
- Voiding the prior Sent version
- Creating and sending the new version with the edited content
- Preserving the void-then-send sequence as a single atomic outcome from the user's perspective

**Non-Goals:**
- Editing an unsent Draft -- an unsent Draft is simply updated in place by FEAT-02.SPEC-001; this automation only applies once a proposal has been Sent.
- Handling the case where the proposal has already been accepted -- an accepted proposal is immutable (XBR-04); this automation refuses to run against an Accepted proposal rather than voiding accepted, evidentiary content.
- Presenting a version-history or diff view of the voided version -- excluded per the Feature Breakdown Brief's Non-Goals; the voided version is retained only as immutable evidence, never surfaced for browsing.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps "Save & Resend" on the editor in edit mode | FEAT-02.SPEC-001 (Proposal Draft Editor) | The target proposal is currently in Sent status (not yet Accepted or already Voided) | Existing Sent proposal id, edited scope_description, edited price, currency (read-only, unchanged), payment_schedule_reference |

## Processing Logic

1. Read the target Proposal's current status; if it is not Sent, stop and report a state-mismatch failure (see Edge Cases -- covers both the already-Accepted and already-Voided cases).
2. Re-run the required-field and positive-price checks on the edited content, per FEAT-02.SPEC-010.
3. Re-check that the project's owning Client still has at least one Primary contact (XBR-07).
4. If all checks pass, mark the existing Sent proposal's status as Voided (an immutable, permanent state -- XBR-04).
5. Create a new Proposal record for the same project with the edited scope_description, price, and currency, and the project's current payment_schedule_reference.
6. Transition the new Proposal directly to Sent, recording its own sent_at.
7. Record the new Proposal's predecessor reference so the void-and-resend relationship is traceable (distinct from copied_from, which records only freelancer-initiated reuse via FEAT-02.SPEC-008).
8. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's Primary contact(s) with the new version's content.
9. Write an Activity Log Entry recording both the void and the new send as one logged event (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Void & resend succeeds | All eligibility checks pass and the target proposal is Sent | Prior proposal -> Voided; new Proposal created and set to Sent with its own sent_at | Editor navigates to FEAT-02.SPEC-003 showing the new Sent version; confirmation "Proposal updated and resent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- already accepted | The target proposal's status is Accepted | None | Editor shows: "This proposal has already been accepted and can no longer be edited." with a "View Current Status" link to FEAT-02.SPEC-003 | FEAT-02.SPEC-001 |
| Blocked -- already voided or superseded | The target proposal's status is already Voided (a second concurrent edit lost the race) | None | Editor shows: "This proposal was already edited and resent. Review the current version." and reloads the current (new) version's content | FEAT-02.SPEC-001, FEAT-02.SPEC-003 |
| Blocked -- invalid fields or no Primary contact | Scope description empty, price missing/non-positive, or the client has no Primary contact | None | Editor (or, if applicable, the send-eligibility banner) shows the specific error per FEAT-02.SPEC-010 | FEAT-02.SPEC-001 |
| Failure | Processing error after eligibility passed (e.g., connectivity lost mid-operation) | No partial state -- the prior Sent proposal remains Sent and unvoided; no new version is created | Editor shows an error banner with Retry; the edited content is preserved locally for retry | FEAT-02.SPEC-001 |

## Data Model

**Reads:** Proposal -- current status, scope_description, price, currency of the target version. Client -- Primary contact roster. Payment Schedule -- current structure for the new version's reference.
**Creates:** A new Proposal record (status Sent, its own sent_at, payment_schedule_reference locked at this moment) with a predecessor reference to the voided version. Activity Log Entry (via FEAT-13) recording the void-and-resend.
**Updates:** The prior Proposal record -- status set to Voided (terminal, immutable thereafter).
**Deletes:** None -- the voided version is retained as evidence (XBR-04), never deleted.

## Business Rules

- A Sent-but-unaccepted proposal is never updated in place -- every edit after send produces a new version through void-and-resend (XBR-06).
- A voided proposal cannot be accepted; if Owen attempts to accept the version that was just voided, FEAT-03 refuses the acceptance and shows him the current version (joint contention rule, dependency map).
- An edit attempt against an already-Accepted proposal is always refused -- accepted proposals are immutable (XBR-04) -- regardless of how the edit was initiated.
- A project retains at most one active (non-voided) proposal at any time -- the new version becomes the sole active proposal for the project the moment it is created.
- The void and the new send are treated as one logged event in the Activity Log Entry, not two independent entries, so the trail reads as a single edit-and-resend action.

## Edge Cases

- **Owen accepts the proposal in the moment between Nadia tapping "Save & Resend" and this automation's status check completing** -- The status check reads Accepted and the automation reports the "already accepted" blocked outcome; Owen's acceptance stands as recorded by FEAT-03, and nothing is voided.
- **Concurrent trigger firing (Nadia edits the same Sent proposal from two open sessions and both trigger Save & Resend at effectively the same time)** -- Only the first to pass the Sent-status check proceeds; it voids the original and creates the new Sent version. The second finds the proposal already Voided and receives the "already voided or superseded" blocked outcome, with its editor reloading the new current version's content -- the second session's edits are not silently merged or lost, but the user must re-apply them against the new version if still wanted.
- **A trigger fires while a previous void-and-resend run for the same proposal is still in flight** -- The triggering editor's "Save & Resend" button is disabled during the operation (FEAT-02.SPEC-001), preventing a second run for the same session; a second session racing in is covered by the concurrent-trigger-firing case above.
- **The client's Primary contact changes between the original send and this edit** -- The new version is sent to the client's current Primary contact(s) at the moment this automation runs, not to whoever received the original Sent version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | "Save & Resend" in edit mode triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Affects (outbound) | Shows the new Sent version, or the blocked-outcome redirect |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Field validation, send eligibility, and the post-acceptance immutability rule |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on success to deliver the new version's link |
| FEAT-03 (Proposal Acceptance) | References (inbound/outbound) | Joint owner of the accept-vs-void contention resolution; the request-changes flow (XBR-26) that often precedes this automation |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the void-and-resend event to the trail (XBR-05) |

## Analytics and Success Signals

- **proposal_edited_before_acceptance** (project id, days between original send and this edit) -- supports success-metrics.md: "Proposal Send Speed" (a fast edit-and-resend keeps the overall proposal-to-acceptance loop quick)
- **proposal_void_resend_blocked_already_accepted** (-- ) -- N/A -- no Stage 2 metric measures this specific race outcome; retained so the accept-vs-edit contention's real-world frequency is observable
- **proposal_void_resend_blocked_already_voided** (-- ) -- N/A -- no Stage 2 metric measures concurrent-edit collisions; retained for operational visibility

## Acceptance Criteria

**FEAT-02.SPEC-006-AC-01:** Given Nadia edits a Sent-but-unaccepted proposal's price and taps "Save & Resend", when validation passes, then the prior version is set to Voided, a new Proposal is created and Sent with the edited content, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-006-AC-02:** Given the target proposal's status is Accepted at the moment of the check, when this automation runs, then it is refused with "This proposal has already been accepted and can no longer be edited." and no void occurs.

**FEAT-02.SPEC-006-AC-03:** Given the target proposal was already voided by a concurrent edit, when this automation runs for the second session, then it is refused with "This proposal was already edited and resent. Review the current version." and the second session's editor reloads the new current version.

**FEAT-02.SPEC-006-AC-04:** Given the edited scope description is empty, when Save & Resend is triggered, then the void does not occur and the field-level error is shown per FEAT-02.SPEC-010.

**FEAT-02.SPEC-006-AC-05:** Given the client has no Primary contact at the moment of this edit, when Save & Resend is triggered, then the void does not occur and the Primary-contact eligibility error is shown.

**FEAT-02.SPEC-006-AC-06:** Given Owen accepts the proposal in the instant before this automation's status check runs, when the automation proceeds, then it reads Accepted and refuses with the already-accepted outcome, leaving Owen's acceptance intact.

**FEAT-02.SPEC-006-AC-07:** Given two sessions trigger Save & Resend on the same Sent proposal at effectively the same time, then only the first succeeds and the second receives the already-voided-or-superseded outcome.

**FEAT-02.SPEC-006-AC-08:** Given a processing failure occurs after eligibility passes but before the transition completes, then the prior proposal remains Sent and unvoided, and no new version is created.

**FEAT-02.SPEC-006-AC-09:** Given void-and-resend succeeds, then a single Activity Log Entry records both the void and the new send as one event (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (success, already accepted, already voided, invalid/blocked, failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 4 | 4 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-005
spec_name: Proposal Send
spec_slug: proposal-send
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Proposal Send

## Overview

**Name:** Proposal Send
**ID:** FEAT-02.SPEC-005
**Type:** Automation
**Purpose:** Validates and transitions a Draft proposal to Sent, recording the send timestamp and locking the payment schedule reference so the client always sees the schedule as it stood at send.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Re-validating send eligibility at the moment of send (required fields, positive price, client Primary-contact requirement)
- Transitioning the Proposal from Draft to Sent
- Recording sent_at and locking the payment_schedule_reference
- Triggering the Proposal Sent email

**Non-Goals:**
- Editing or discarding the proposal -- owned by FEAT-02.SPEC-001, FEAT-02.SPEC-006, and FEAT-02.SPEC-009; this automation only performs the Draft-to-Sent transition.
- Composing or delivering the email itself -- owned by FEAT-02.SPEC-011 (Proposal Sent/Resent Email); this automation only triggers it on success.
- Re-sending an already-Sent proposal's link -- owned by FEAT-02.SPEC-007 (Proposal Resend); this automation only handles the first Draft-to-Sent transition for a given version.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps Send on the Preview screen | FEAT-02.SPEC-002 (Proposal Preview) | The proposal is currently in Draft status | Proposal id, scope_description, price, currency, payment_schedule_reference, owning project and client |

## Processing Logic

1. Read the Proposal's current record by id. If no record is found, the Draft was discarded (FEAT-02.SPEC-009) since Preview was opened -- stop and report the discarded-draft outcome (see Edge Cases). If a record is found but its status is not Draft, stop and report the state-mismatch failure -- these are reported as distinct outcomes so the message Nadia sees always matches what actually happened, never assuming "already sent" when the Draft was instead discarded.
2. Re-run the required-field and positive-price checks defined in FEAT-02.SPEC-010.
3. Check that the project's owning Client has at least one Primary contact (XBR-07); if not, stop and report the Primary-contact failure.
4. Check that the project has no other active (non-voided, non-Draft-being-sent) proposal -- confirming the one-active-proposal-per-project rule still holds at the moment of send.
5. If all checks pass, transition the Proposal's status from Draft to Sent.
6. Record sent_at as the current date and time.
7. Lock the proposal's payment_schedule_reference to the project's Payment Schedule as it currently stands, so later schedule adjustments do not retroactively change what this sent version references (consistent with the Payment Schedule's own non-retroactive adjustment rule).
8. Trigger FEAT-02.SPEC-011 (Proposal Sent/Resent Email) to the client's Primary contact(s).
9. Write an Activity Log Entry recording the send (XBR-05), owned by FEAT-13.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Send succeeds | All eligibility checks pass | Proposal status -> Sent; sent_at set; payment_schedule_reference locked | Preview navigates to FEAT-02.SPEC-003 showing Sent status; confirmation "Proposal sent to {Primary Contact name}." | FEAT-02.SPEC-003, FEAT-02.SPEC-011, FEAT-13 |
| Blocked -- no Primary contact | Client has zero Primary contacts (XBR-07) | None | Preview shows the Send Blocked banner "This client has no Primary contact yet." with a link into FEAT-18 | FEAT-02.SPEC-002 |
| Blocked -- invalid fields | Scope description empty, or price missing/non-positive | None | Preview (or, if reached directly, the editor) shows the specific field error per FEAT-02.SPEC-010 | FEAT-02.SPEC-001, FEAT-02.SPEC-002 |
| Blocked -- state mismatch | The proposal record still exists but is no longer in Draft status (e.g., already sent from another session) | None | Preview shows: "This proposal has already been sent. Viewing the current version." and navigates to FEAT-02.SPEC-003 | FEAT-02.SPEC-002, FEAT-02.SPEC-003 |
| Blocked -- draft discarded | The Draft's record no longer exists because it was discarded (FEAT-02.SPEC-009) concurrently, before this Send call's status check | None | Preview shows: "This draft no longer exists -- it was discarded." and navigates to FEAT-02.SPEC-003 showing the empty state | FEAT-02.SPEC-002, FEAT-02.SPEC-003, FEAT-02.SPEC-009 |
| Failure | Processing error after eligibility passed (e.g., connectivity lost mid-operation) | No partial state -- the proposal remains Draft; nothing is sent | Preview shows an error banner with Retry; the Draft is unaffected | FEAT-02.SPEC-002 |

## Data Model

**Reads:** Proposal -- status, scope_description, price, currency, payment_schedule_reference. Client -- Client Contact roster (for the Primary-contact check). Payment Schedule -- current structure, to lock into the reference.
**Creates:** Activity Log Entry (via FEAT-13) recording the send event.
**Updates:** Proposal -- status (Draft -> Sent), sent_at, payment_schedule_reference (locked to the current schedule).
**Deletes:** None.

## Business Rules

- Send eligibility is fully re-checked at send time, not only when the Draft was last saved -- conditions (client Primary contact, field validity, one-active-proposal cap) can change between drafting and sending.
- A proposal cannot be sent until the client has at least one Primary contact (XBR-07).
- The payment_schedule_reference is locked at the moment of send; later, non-retroactive Payment Schedule adjustments (FEAT-04) never change what an already-sent proposal references.
- Sending is the only path that transitions a Proposal from Draft to Sent -- FEAT-02.SPEC-006 (Void & Resend) creates a new Sent version through its own path rather than reusing this automation on an existing Draft.

## Edge Cases

- **The proposal is no longer in Draft status when Send is invoked because it was already sent from another session** -- Reported as the state-mismatch outcome ("This proposal has already been sent."); the Preview screen redirects to FEAT-02.SPEC-003 to show the current Sent version.
- **The proposal's Draft record no longer exists when Send is invoked because it was discarded concurrently (FEAT-02.SPEC-009)** -- Reported as the distinct discarded-draft outcome ("This draft no longer exists -- it was discarded."), never the state-mismatch message, since "already been sent" would misinform Nadia about what actually happened. The Preview screen redirects to FEAT-02.SPEC-003's empty state.
- **The client's only Primary contact is removed between Preview load and Send tap** -- Blocked with the Primary-contact outcome; no partial Sent state is created.
- **Concurrent trigger firing (Send tapped from two open Preview sessions for the same Draft at effectively the same time)** -- Only the first to pass the Draft-status check transitions the proposal to Sent; the second finds the proposal already Sent and receives the state-mismatch outcome, redirected to view the current (now Sent) version. No duplicate Sent version and no duplicate email are produced.
- **A trigger fires while a previous send for the same proposal is still in flight** -- The triggering Preview screen's Send button is disabled during the operation (FEAT-02.SPEC-002), so a second send for the same Draft cannot be initiated from the same session; a second session is covered by the concurrent-trigger-firing case above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Proposal Preview) | Triggered by (inbound) | Send tap, after screen-level eligibility passes, triggers this automation |
| FEAT-02.SPEC-003 (Proposal Detail) | Affects (outbound) | Shows the resulting Sent status, or the state-mismatch redirect |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | Field validation and send-eligibility rules re-checked here |
| FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Affects (outbound) | Triggered on send success to deliver the proposal link |
| FEAT-18 (Client Contact Management & Roles) | References (outbound) | Primary-contact requirement checked against this feature's contact roster |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the send event to the trail (XBR-05) |
| FEAT-02.SPEC-009 (Discard Draft) | References (inbound) | A concurrent discard produces the distinct discarded-draft outcome rather than the state-mismatch outcome |

## Analytics and Success Signals

- **proposal_sent** (project id, price, currency, time from draft creation to send in minutes) -- supports success-metrics.md: "Proposal Send Speed"
- **proposal_send_blocked_no_primary_contact** (client id) -- N/A -- no Stage 2 metric measures this specific block reason; retained so the XBR-07 gate's real-world frequency is observable
- **proposal_send_state_mismatch** (-- ) -- N/A -- no Stage 2 metric measures concurrent-send collisions; retained for operational visibility into how often the race occurs
- **proposal_send_draft_discarded** (-- ) -- N/A -- no Stage 2 metric measures this specific race; retained for operational visibility into how often a concurrent discard is the cause of a blocked send, as distinct from an already-sent collision

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Nadia's Draft proposal has valid scope, a positive price, and the client has a Primary contact, when Send is triggered from FEAT-02.SPEC-002, then the proposal transitions to Sent, sent_at is recorded, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-005-AC-02:** Given the client has no Primary contact, when Send is triggered, then the proposal remains Draft and the Blocked -- no Primary contact outcome is returned.

**FEAT-02.SPEC-005-AC-03:** Given the proposal's scope description is empty, when Send is triggered, then the proposal remains Draft and the Blocked -- invalid fields outcome is returned.

**FEAT-02.SPEC-005-AC-04:** Given the proposal was already sent from another session before this Send call reaches the status check, when this automation runs, then it returns the state-mismatch outcome ("This proposal has already been sent.") and no second Sent version or email is created.

**FEAT-02.SPEC-005-AC-05:** Given the Draft was discarded (FEAT-02.SPEC-009) from another session before this Send call reaches the status check, when this automation runs, then it returns the distinct discarded-draft outcome ("This draft no longer exists -- it was discarded."), not the state-mismatch message, and Preview navigates to FEAT-02.SPEC-003's empty state.

**FEAT-02.SPEC-005-AC-06:** Given Send succeeds, then the proposal's payment_schedule_reference is locked to the Payment Schedule as it stood at that moment.

**FEAT-02.SPEC-005-AC-07:** Given a processing failure occurs after eligibility checks pass but before the transition completes, then the proposal remains Draft and no email is sent.

**FEAT-02.SPEC-005-AC-08:** Given two Preview sessions for the same Draft trigger Send at effectively the same time, then only the first transitions the proposal to Sent and the second receives the state-mismatch outcome.

**FEAT-02.SPEC-005-AC-09:** Given Send succeeds, then an Activity Log Entry recording the send is written (FEAT-13).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (success, no primary contact, invalid fields, state mismatch, draft discarded, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

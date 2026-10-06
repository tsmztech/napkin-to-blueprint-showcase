---
document_type: spec
spec_type: automation
spec_id: FEAT-31.SPEC-003
spec_name: Support Session Open & Read-Only Enforcement
spec_slug: support-session-open-read-only-enforcement
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Support Session Open & Read-Only Enforcement

## Overview

**Name:** Support Session Open & Read-Only Enforcement
**ID:** FEAT-31.SPEC-003
**Type:** Automation
**Purpose:** Opens the Support Access Session record the instant Dana starts a session, and enforces that every edit, send, approve, pay, file-download, and export-generation control is unavailable for the session's duration.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Authorization check at the moment Dana selects a queued request (FEAT-31.SPEC-005: Dana-only, one-account-at-a-time)
- Setting `operator` and `opened_at` on the selected Support Access Session record
- Loading the named freelancer's account data read-only across FEAT-01 through FEAT-25
- Applying the read-only enforcement to every control on every mirrored screen for the session's duration
- Triggering the session-opened notice (FEAT-31.SPEC-007) and the activity trail entry (FEAT-13)

**Non-Goals:**
- Closing the session -- handled exclusively by FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity); this automation has no closing logic of its own, consistent with the Brief's Non-Goals excluding any manual close path.
- Defining the exact set of excluded actions and their denied messages -- owned by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules); this automation applies those rules but is not their source of truth.
- Rendering the mirrored screens' own content -- each of FEAT-01 through FEAT-25 owns its own screen; this automation only enforces the read-only constraint across whichever one Dana is viewing.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Dana selects a queued request and taps "Open Session" | FEAT-31.SPEC-002 (Operator Support Session Console) | Fires when authorization (FEAT-31.SPEC-005) passes -- Dana has no other session currently open | The selected Support Access Session's `freelancer_account` and `request_text`; Dana's operator identity |

## Processing Logic

1. Verify authorization per FEAT-31.SPEC-005: confirm Dana has no other Support Access Session currently open (one-account-at-a-time) and that she is the sole role permitted to open a session.
2. If authorization fails, stop -- no record is changed (see Outcome Definitions).
3. Load the named freelancer's account data read-only, spanning FEAT-01 through FEAT-25, exactly as the freelancer would see it herself.
4. If the account's data cannot be loaded read-only, stop -- `operator`/`opened_at` stay unset (see Outcome Definitions) and nothing about the account changes.
5. Set `operator` to Dana's identity and `opened_at` to the current timestamp on the selected Support Access Session record. The mirrored view is not yet shown to Dana; the row stays in its Opening state.
6. Trigger FEAT-13.SPEC-003 (Activity Entry Recording) to write the session-opened trail entry, with Dana as actor and `opened_at` as the timestamp. The trail entry is a precondition for the session becoming visible to Dana. If the write fails, go to step 9 (Trail failure).
7. Trigger FEAT-31.SPEC-007 (Support Session Opened Notice) to queue the email to Nadia. Accepting the notice into SPEC-007's delivery queue is a precondition for the session becoming visible to Dana (delivery retries after that point belong to FEAT-31.SPEC-007). If the trigger is not accepted, go to step 9 (Notice failure).
8. Only after steps 6 and 7 both succeed: apply the read-only enforcement continuously for the session's duration (every edit, send, approve, pay, file-download, and export-generation control disabled or not shown on every mirrored screen Dana views, per FEAT-31.SPEC-005's exact excluded-action list) and display the permanent "Read-only support session -- {freelancer_account name}" banner on every mirrored screen (FEAT-31.SPEC-002).
9. On a step 6 or step 7 failure, roll back: unset `operator` and `opened_at` together so the request returns to the pending state and stays in the queue, and never show the mirrored view. If a session-opened trail entry had already been written (notice failure), write a second FEAT-13.SPEC-003 entry recording "Support session did not start" with Dana as actor, so Nadia's trail never shows an open session that Dana never saw. Dana sees the partial-failure message and a Retry control (FEAT-31.SPEC-002).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session opened | Authorization passes, the account loads read-only, the trail entry is written, and the notice trigger is accepted | `operator` and `opened_at` set on the Support Access Session record; session-opened trail entry written; notice queued | Mirrored view with the permanent banner appears | FEAT-31.SPEC-002 (renders it), FEAT-31.SPEC-007 (notice sent), FEAT-13.SPEC-003 (trail entry written) |
| Authorization denied | Dana already has another session open, or a role other than Dana attempts to trigger this automation | None | The exact denied message from FEAT-31.SPEC-005: "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it." Dana can Resume the open session or wait for it to close automatically; no manual close exists | FEAT-31.SPEC-002 |
| Partial failure (trail entry) | The account loaded and `operator`/`opened_at` were set, but the FEAT-13.SPEC-003 session-opened trail entry could not be written | Rolled back: `operator` and `opened_at` unset together; no trail entry exists; no notice triggered | The mirrored view never appears. Dana sees inline on the queue row "The session could not be started right now. Nothing was opened." with a "Retry" button; the request stays in the queue. Retry re-runs the whole automation from step 1 | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| Partial failure (notice) | The trail entry was written but FEAT-31.SPEC-007 did not accept the notice trigger | Rolled back: `operator` and `opened_at` unset together; the written trail entry is followed by a "Support session did not start" entry | Same message and "Retry" button as the trail-entry failure; the mirrored view never appears | FEAT-31.SPEC-002, FEAT-31.SPEC-007, FEAT-13.SPEC-003 |
| Load failure | The named freelancer's account data cannot be loaded read-only (e.g., the account is mid-deletion via FEAT-24) | None -- `operator`/`opened_at` remain unset | Dana sees the specific reason inline on the queue row (e.g., "This account's data could not be loaded right now."); nothing about the account changes | FEAT-31.SPEC-002 |

## Data Model

**Reads:** Support Access Session -- the selected pending record's `freelancer_account` and `request_text`. Freelancer Account and every entity it owns across FEAT-01 through FEAT-25 -- read-only, for the session's duration.
**Creates:** None (FEAT-31.SPEC-001 already created the Support Access Session record).
**Updates:** Support Access Session -- `operator` (Dana's identity) and `opened_at` (current timestamp).
**Deletes:** None.

## Business Rules

- XBR-29: sessions are read-only in every feature, cover one account at a time, end after inactivity, exclude file downloads and data/accounting exports, are always announced to the freelancer by email, and are always listed in her trail.
- Authorization and the exact excluded-action list are owned by FEAT-31.SPEC-005; this automation enforces them but does not restate or duplicate them.
- The read-only enforcement is unconditional for the entire session -- there is no partial-edit window, no grace period, and no escalation path to write access.
- XBR-29's "always announced" and "always listed in her trail" are guaranteed by ordering: a session becomes visible to Dana only after its trail entry is written and its notice trigger is accepted, so a session Dana can see is always both logged and announced; a session that cannot be logged or announced never opens.
- Only Dana may trigger this automation; no other role has a control that reaches it.

## Edge Cases

- **The freelancer account is deleted (FEAT-24) between Dana selecting the request and this automation loading it** -- Load failure outcome: nothing changes, Dana sees the reason. The request itself is removed from the queue as part of that account's full deletion.
- **Dana's authorization check passes but the account load then fails** -- No partial state: `operator` and `opened_at` are set together with a successful load, never independently of it; a load failure leaves both unset.
- **Concurrent trigger firing (Dana attempts to open two different requests from two browser tabs at effectively the same time)** -- Each triggering attempt is evaluated independently against the one-account-at-a-time rule at the moment it fires; whichever attempt's authorization check completes first opens successfully, and the second is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it.", even though both taps were near-simultaneous.
- **The trail entry or notice trigger fails after `operator`/`opened_at` are set** -- Rolled back per Processing Logic step 9; the request returns to the pending queue, Dana retries, and no session is ever visible without both its trail entry and its accepted notice. Concurrent "Open Session" attempts during the rolled-back window are evaluated against the one-account-at-a-time rule only after the rollback completes.
- **Trigger fires while a previous run is in flight** -- The "Open Session" control on the queue is disabled for the row being opened while this automation is processing (FEAT-31.SPEC-002's loading state), so a second run for the same request cannot start until the first completes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-002 (Operator Support Session Console) | Triggered by (inbound) | "Open Session" fires this automation |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Authorization check and the excluded-action list this automation enforces |
| FEAT-31.SPEC-007 (Support Session Opened Notice) | Triggers (outbound) | Fires on successful open |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Triggers (outbound) | Writes the session-opened trail entry |

## Analytics and Success Signals

- **support_session_opened** (no properties beyond the event itself) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable
- **support_session_open_denied** (reason: already_open) -- N/A -- same reason as above
- **support_session_open_load_failed** (reason: account_load / trail_entry / notice_trigger) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-003-AC-01:** Given Dana selects a queued request with no other session currently open, when she taps "Open Session," then `operator` and `opened_at` are set on the record, the mirrored read-only view appears with the permanent banner, the opened notice (FEAT-31.SPEC-007) is triggered, and the trail entry is written.

**FEAT-31.SPEC-003-AC-02:** Given Dana already has a session open on one freelancer account, when she attempts to open a second, then authorization fails, no record changes, and she sees "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-03:** Given the selected request's freelancer account cannot be loaded read-only, when this automation attempts to open the session, then `operator` and `opened_at` remain unset and Dana sees the specific reason inline.

**FEAT-31.SPEC-003-AC-04:** Given a session has opened successfully, when Dana views any mirrored screen, then every edit, send, approve, pay, file-download, and export-generation control is disabled or not shown.

**FEAT-31.SPEC-003-AC-05:** Given Dana attempts to open two different requests from two browser tabs at effectively the same time, when both "Open Session" taps fire, then only the first to complete its authorization check succeeds and the second is refused with "Your support session on {freelancer_account name} is still open and ends automatically after inactivity. Tap Resume to return to it."

**FEAT-31.SPEC-003-AC-06:** Given this automation is still processing a prior "Open Session" tap for a request, when Dana taps the same request's "Open Session" control again, then no second run starts because the control is disabled while the first is in flight.

**FEAT-31.SPEC-003-AC-07:** Given the account loads read-only and `operator`/`opened_at` are set but the FEAT-13.SPEC-003 session-opened trail entry cannot be written, when this automation handles the failure, then `operator` and `opened_at` are unset together, no notice is triggered, the mirrored view never appears, and Dana sees "The session could not be started right now. Nothing was opened." with a "Retry" button while the request stays in the queue.

**FEAT-31.SPEC-003-AC-08:** Given the trail entry was written but FEAT-31.SPEC-007 does not accept the notice trigger, when this automation handles the failure, then `operator` and `opened_at` are unset together, a "Support session did not start" trail entry follows the earlier entry, the mirrored view never appears, and Dana sees the same message with "Retry."

**FEAT-31.SPEC-003-AC-09:** Given a prior attempt was rolled back by AC-07 or AC-08, when Dana taps "Retry" and the trail entry and notice trigger both succeed, then the session opens with the permanent banner and Nadia's trail shows the session-opened entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (opened, denied, load failure, partial failure trail entry, partial failure notice) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

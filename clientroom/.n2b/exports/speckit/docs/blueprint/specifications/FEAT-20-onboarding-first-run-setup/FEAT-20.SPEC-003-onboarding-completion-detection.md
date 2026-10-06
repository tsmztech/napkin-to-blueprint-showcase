---
document_type: spec
spec_type: automation
spec_id: FEAT-20.SPEC-003
spec_name: Onboarding Completion Detection
spec_slug: onboarding-completion-detection
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Onboarding Completion Detection

## Overview

**Name:** Onboarding Completion Detection
**ID:** FEAT-20.SPEC-003
**Type:** Automation
**Purpose:** Detects when a first client, project, and drafted proposal all exist for the Freelancer Account, marks onboarding complete, and signals FEAT-20.SPEC-002 to render the Ready state. This automation only signals; FEAT-20.SPEC-002 owns the Ready state and the route into the dashboard.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Re-checking onboarding's exit criteria (a first Client, a first Project under it, and a drafted Proposal for that project) each time a guided step reports completion, each time the onboarding shell opens while onboarding is In Progress, and each time Nadia taps "Check again"
- Reporting a check that could not finish, so the shell can offer a retry
- When completing with the "How did you hear" question unresolved, auto-resolving it as "unknown" and firing the FEAT-20.SPEC-004 hand-off
- Marking onboarding complete on the Freelancer Account's onboarding-progress state exactly once
- Emitting `onboarding_completed` and signaling FEAT-20.SPEC-002 to render the Ready state
- Detecting completion regardless of whether the underlying records were created through the guided sequence's own buttons or by navigating directly to each feature's screen

**Non-Goals:**
- Defining which steps are mandatory for exit -- owned by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules); this automation only evaluates the exit criteria that spec defines, it does not define them
- Creating the Client, Project, or Proposal records themselves -- owned by FEAT-01 and FEAT-02 respectively; this automation only reads their existence and state
- Routing, navigation, and the Ready state itself -- the Ready state and the "Go to your dashboard" route belong to FEAT-20.SPEC-002, and the dashboard's content to FEAT-12 (Freelancer Financial Dashboard); this automation never navigates and only reports an outcome
- Re-evaluating completion after onboarding has already been marked complete once -- excluded per the product's Entity-Lifecycle Coverage Matrix, which treats onboarding as a one-time, per-account setup sequence with no independent re-run

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A guided step reports completion | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires every time Nadia returns from a step's own screen having completed it, and onboarding is not already marked complete | Freelancer Account reference, the step just completed |
| The onboarding shell opens while onboarding is In Progress | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires on every open of the shell after its progress read succeeds, while onboarding is not already marked complete (covers a completion check that failed earlier, or completion that happened in another tab) | Freelancer Account reference |
| Nadia taps "Check again" on the completion-check notice | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires only when a previous check reported a read failure and onboarding is not already marked complete | Freelancer Account reference |
| A first client, project, or proposal draft is created outside the guided sequence's own navigation | FEAT-01.SPEC-002 (Create Project), FEAT-02.SPEC-001 (Proposal Draft Editor) | Fires when Nadia reaches a mandatory milestone (project creation, proposal draft save) by navigating directly rather than through FEAT-20.SPEC-002's buttons, and onboarding is not already marked complete | Freelancer Account reference, the record just created |

## Processing Logic

1. Confirm the Freelancer Account exists and onboarding is not already marked complete on its onboarding-progress state (per the Non-Goals above, a completed account never re-enters this check).
2. Read whether at least one Client exists under the Freelancer Account.
3. Read whether at least one Project exists under that Client (or any Client, if more than one now exists).
4. Read whether a Proposal in Draft or later status exists for that Project.
5. Evaluate the exit criteria defined by FEAT-20.SPEC-005: all three of Client, Project, and drafted Proposal must exist. Branding, payment connection, and full billing setup are never part of this evaluation, regardless of their skip state.
6. If all three exist, mark the Freelancer Account's onboarding-progress state complete, record the completion timestamp, and emit `onboarding_completed`. If onboarding_how_did_you_hear_resolved is still false, trigger FEAT-20.SPEC-004 with "unknown" (FEAT-20.SPEC-005 Rule R-09); this spec never writes onboarding_how_did_you_hear_resolved, FEAT-20.SPEC-004 sets it after its hand-off attempt.
7. If any of the three is missing, take no action -- onboarding-progress state remains unchanged, and FEAT-20.SPEC-002 continues to show the current in-progress step.
7a. If the read in steps 2-4 cannot complete, take no data action and treat the outcome as "check could not finish".
8. Signal FEAT-20.SPEC-002 with the outcome (complete, not yet, or check could not finish) so the shell can render the Ready state, remain on its current step, or show the completion-check notice with "Check again".

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Exit criteria not yet met | One or more of Client, Project, drafted Proposal is missing | None | None directly -- FEAT-20.SPEC-002 continues showing the current step | FEAT-20.SPEC-002 |
| Onboarding marked complete | Client, Project, and a drafted Proposal all exist, and onboarding was not already complete | Freelancer Account's onboarding-progress state set to complete with a completion timestamp; FEAT-20.SPEC-004 is triggered (and it, not this spec, sets onboarding_how_did_you_hear_resolved) if the question was still unresolved | FEAT-20.SPEC-002 renders the Ready state (Nadia leaves it by tapping "Go to your dashboard"; no automatic routing) | FEAT-20.SPEC-002, FEAT-20.SPEC-004 (only when the question was unresolved) |
| Already complete (re-trigger ignored) | This automation fires again after onboarding is already marked complete | None | None -- silent no-action, since re-checking a settled outcome has no visible effect | -- |
| Read failure | The check for Client, Project, or Proposal existence cannot complete (e.g., a transient failure reading a shared entity) | None | FEAT-20.SPEC-002 shows the notice "We couldn't confirm your setup just now." with a "Check again" button. The check re-runs when Nadia taps "Check again", on every later shell open, and on every later step completion or record save -- so a failure on the final step never strands her | FEAT-20.SPEC-002 |

## Data Model

**Reads:** Freelancer Account -- to confirm the account exists and to read its onboarding-progress state. Client, Project (owned and created by FEAT-01) -- to confirm a first client and project exist. Proposal (owned and created by FEAT-02) -- to confirm a drafted proposal exists for the new project.
**Creates:** None.
**Updates:** Freelancer Account -- its onboarding-progress state only: onboarding_status set to complete with a completion timestamp. onboarding_how_did_you_hear_resolved is read but never written here (FEAT-20.SPEC-004 is its only writer). No other Freelancer Account field is touched.
**Deletes:** None.

## Business Rules

- The exit criteria are exactly the three named in FEAT-20.SPEC-005: a first Client, a first Project, and a drafted Proposal -- branding, payment connection, and full billing setup are optional and never gate completion (product-features.md, Validation & Limits).
- Completion is detected regardless of navigation path: whether Nadia used FEAT-20.SPEC-002's own step buttons or navigated directly into FEAT-01 or FEAT-02's screens, the same three records satisfy the same check.
- A check that cannot finish is never silent-and-final: it is reported to FEAT-20.SPEC-002 and re-runs on the next shell open, the next step completion or record save, or a "Check again" tap.
- Completion is recorded exactly once; once marked complete, this automation takes no further action on subsequent triggers for the same account.
- A Proposal in any status of Draft or later (Sent, Voided, Accepted) satisfies the "drafted proposal" criterion -- a proposal that has since been sent or accepted still means a draft was reached.

## Edge Cases

- **Nadia creates a project and drafts a proposal for it, then that project is archived before onboarding's check runs again** -- Archiving a project (FEAT-01) does not remove its historical existence; the exit criteria evaluate whether the records were ever created, not whether they remain active, so completion is still detected.
- **Nadia adds a client but no project yet** -- Exit criteria are not met; onboarding remains in progress and no completion is recorded.
- **The Proposal is later voided and re-sent (FEAT-02.SPEC-006)** -- The original draft already satisfied the exit criterion before the void; a Proposal's later lifecycle changes never retroactively un-complete onboarding, since completion is a one-time, never-reversed transition.
- **Concurrent trigger firing (Nadia completes the client/project step and, in a second open tab, drafts the proposal at effectively the same moment)** -- Both triggers independently read the current record state; whichever evaluation runs second sees the results of the first and correctly finds all three records present, marking completion once. The one-time completion rule (Business Rules) prevents a duplicate `onboarding_completed` emission even if both evaluations reach the completion condition before either writes.
- **Trigger fires while a previous run is in flight** -- A second evaluation for the same account is not started while the account's onboarding-progress state is mid-update from a prior evaluation; it re-reads the (by-then-updated) state instead, so it correctly finds onboarding already complete and takes no action.
- **The Freelancer Account is deleted (FEAT-24) between a step completing and this automation evaluating** -- The automation's read of the account fails; per the Read Failure outcome, no action is taken and there is no shell left to signal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Triggered by (inbound) / Affects (outbound) | Every step completion, every shell open while In Progress, and every "Check again" tap triggers this check; the outcome tells the shell whether to advance, show Ready, or show the completion-check notice |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggers (outbound) | Completion with the "How did you hear" question unresolved auto-resolves it as "unknown" and fires the hand-off once |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Defines which three records constitute the exit criteria this automation evaluates |
| FEAT-01.SPEC-002 (Create Project) | Triggered by (inbound) | A project created by navigating directly (outside the guided sequence's buttons) also re-triggers this check |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Triggered by (inbound) | A proposal drafted by navigating directly also re-triggers this check |
| FEAT-12 (Freelancer Financial Dashboard) | References (outbound, indirect) | This automation does not route to or write to FEAT-12; the Ready state (FEAT-20.SPEC-002) it triggers is what offers Nadia the "Go to your dashboard" button |

## Analytics and Success Signals

- **onboarding_completed** (time from account creation to completion, whether branding was set, whether payments were connected) -- supports success-metrics.md: "First-Session Activation"
- **onboarding_exit_criteria_check_no_action** (which of the three criteria is still missing) -- N/A -- no Stage 2 metric measures individual in-progress checks; retained only as an internal no-action outcome, not a candidate for a dedicated signal beyond the completion event above.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given Nadia has added a first client and project but has not yet drafted a proposal, when this automation evaluates the exit criteria, then onboarding is not marked complete and FEAT-20.SPEC-002 remains on its current step.

**FEAT-20.SPEC-003-AC-02:** Given Nadia has a first client, project, and drafted proposal, when this automation evaluates the exit criteria after the proposal draft is saved, then onboarding is marked complete, `onboarding_completed` is emitted, and FEAT-20.SPEC-002 shows the Ready state.

**FEAT-20.SPEC-003-AC-03:** Given onboarding is already marked complete, when this automation is triggered again (e.g., by an unrelated later action), then no action is taken and no duplicate `onboarding_completed` is emitted.

**FEAT-20.SPEC-003-AC-04:** Given Nadia skipped branding and payment connection entirely, when she completes the client, project, and proposal draft, then onboarding is marked complete regardless of the skipped steps.

**FEAT-20.SPEC-003-AC-05:** Given Nadia creates her first project by navigating directly to FEAT-01.SPEC-002 rather than through FEAT-20.SPEC-002's own button, when the project is created, then this automation still re-evaluates the exit criteria from that trigger.

**FEAT-20.SPEC-003-AC-06:** Given Nadia's drafted proposal is later voided and re-sent, when this automation is asked to re-evaluate at any later point, then onboarding remains complete -- the earlier draft already satisfied the criterion and completion is never reversed.

**FEAT-20.SPEC-003-AC-07:** Given Nadia completes her client/project step in one browser tab and drafts her proposal in another at effectively the same moment, when both triggers fire, then onboarding is marked complete exactly once, with no duplicate `onboarding_completed` event.

**FEAT-20.SPEC-003-AC-08:** Given a prior evaluation for Nadia's account is still updating onboarding-progress state, when a new trigger fires for the same account before that update finishes, then the new evaluation reads the just-updated state and correctly takes no further action.

**FEAT-20.SPEC-003-AC-09:** Given Nadia's Freelancer Account is deleted between a step completing and this automation's evaluation, when the automation attempts to read the account, then it takes no action and nothing is signaled to any shell.

**FEAT-20.SPEC-003-AC-10:** Given Nadia has a first client and project, when a Proposal for that project exists in any status of Draft or later, then the "drafted proposal" exit criterion is satisfied.

**FEAT-20.SPEC-003-AC-11:** Given the check for Nadia's final mandatory step could not read the proposal, when the failure occurs, then FEAT-20.SPEC-002 shows "We couldn't confirm your setup just now." with "Check again", and tapping it re-runs this automation and, once it succeeds, onboarding is marked complete and the Ready state is shown.

**FEAT-20.SPEC-003-AC-12:** Given a previous check failed and Nadia closes and reopens the shell, when the shell opens while onboarding is In Progress, then this automation re-runs on open and marks onboarding complete if all three records exist.

**FEAT-20.SPEC-003-AC-13:** Given onboarding completes, when the outcome is signalled, then FEAT-20.SPEC-002 renders the Ready state and this automation performs no navigation of its own.

**FEAT-20.SPEC-003-AC-14:** Given onboarding completes while onboarding_how_did_you_hear_resolved is false, when completion is recorded, then FEAT-20.SPEC-004 is triggered once with "unknown" and this spec does not write the field (FEAT-20.SPEC-004 sets it true after its hand-off attempt).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

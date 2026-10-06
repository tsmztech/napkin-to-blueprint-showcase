---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-004
spec_name: Project Cancellation Recording
spec_slug: project-cancellation-recording
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Project Cancellation Recording

## Overview

**Name:** Project Cancellation Recording
**ID:** FEAT-25.SPEC-004
**Type:** Automation
**Purpose:** Persists Nadia's cancellation of a project, sets `cancelled_at`, derives the Cancelled stage, and preserves every existing proposal, milestone, deliverable, and invoice record unchanged.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Re-validating the project's state against a live re-check at the moment of commit (not the state the screen loaded with)
- Setting the Project's `cancelled_at` timestamp and the optional freelancer-entered reason
- Triggering FEAT-01.SPEC-011 (Project Stage Derivation) so the Cancelled stage takes precedence over any milestone- or invoice-driven "In Progress" condition
- Notifying Owen once the cancellation is recorded (via FEAT-25.SPEC-007)

**Non-Goals:**
- Deleting or archiving any project record -- excluded per the dependency map (Entity: Project, Lifecycle): this automation preserves the Proposal, Payment Schedule, Milestones, Deliverables, Invoices, and Activity Log Entries exactly as they were, and touches none of their fields
- Collecting the reason from Nadia -- owned entirely by FEAT-25.SPEC-002 (Mark Project Cancelled Screen); this automation begins where that screen's submission ends
- Computing the stage label itself -- owned entirely by FEAT-01.SPEC-011 (Project Stage Derivation); this automation only sets `cancelled_at`, which that spec reads as one of its precedence-ordered inputs
- Acting on an unaccepted proposal -- excluded per XBR-25: a proposal left open when its project is cancelled stays exactly as it was; revising or voiding it is Nadia's own separate action through FEAT-02, never triggered by this automation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Cancellation submitted | FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Fires when Nadia confirms cancellation a second time on the confirming dialog, on a project whose derived stage is none of Cancelled, Archived, or Complete when the screen loaded | Project reference, optional reason |

## Processing Logic

1. Receive the project reference and optional reason from the triggering screen.
2. Re-read the Project's current derived `stage` at the moment of commit (not the value the screen loaded with), by invoking FEAT-01.SPEC-011.
3. Confirm the current derived stage is none of Cancelled, Archived, or Complete (Complete meaning the Project's `completed_at` is set, per FEAT-01.SPEC-011). If it is any of the three, stop and report a stale-state outcome. Because FEAT-25.SPEC-002 offers cancellation only on projects in none of those stages, reaching one of them here always means the project changed since the screen loaded, so the stale-state message applies; a project in any other stage proceeds to plain success.
4. Set the Project's `cancelled_at` to the current timestamp and store the optional reason.
5. Signal FEAT-01.SPEC-011 to recompute the derived `stage`, which resolves to Cancelled once `cancelled_at` is set (taking precedence over any milestone- or invoice-driven "In Progress" condition, per that spec's precedence order).
6. Leave the project's Proposal, Payment Schedule, Milestones, Deliverables, Invoices, and Activity Log Entries entirely untouched.
7. Trigger FEAT-25.SPEC-007 (Refund & Cancellation Notification) to email Owen.
8. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
9. Return the new stage to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation recorded | The project's derived stage at commit is none of Cancelled, Archived, or Complete | `cancelled_at` set; reason stored; stage recomputes to Cancelled | Toast "Project cancelled"; FEAT-01.SPEC-005 shows the Cancelled stage badge | FEAT-25.SPEC-002, FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-25.SPEC-007, FEAT-13 |
| Stale-state rejection | The project's derived stage at commit is Cancelled, Archived, or Complete (changed since the screen loaded) | None -- no write occurs | FEAT-25.SPEC-002 shows "This project's state changed since you opened this page. Refresh to see the latest state." | FEAT-25.SPEC-002 |
| Automation failure | Persisting `cancelled_at` fails after the stale-state check passes (e.g., a transient write failure) | No partial write | FEAT-25.SPEC-002 shows "Couldn't cancel this project. Try again." with a Retry button; the project's prior stage remains authoritative | FEAT-25.SPEC-002 |

## Data Model

**Reads:** Project -- derived `stage` and `completed_at` (re-checked at commit, via FEAT-01.SPEC-011).
**Creates:** None.
**Updates:** Project -- `cancelled_at` (set), reason (stored alongside the cancellation).
**Deletes:** None -- no record on the project or any of its related entities is removed (XBR-25).

## Business Rules

- XBR-25: Marking a project cancelled preserves its full history; an unaccepted proposal stays open until Nadia revises and re-sends it or cancels the project -- this automation is the "cancels the project" half of that rule and never itself acts on the proposal.
- Per the dependency map's Contention note for Project: system-driven stage changes (acceptance, approval) never overwrite a freelancer's explicit Cancelled transition; this automation's commit-time check (step 3) exists to keep that guarantee true even under a race, and once `cancelled_at` is set, FEAT-01.SPEC-011's precedence order keeps Cancelled from being overwritten going forward.
- A Complete project cannot be cancelled: Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order), and this automation refuses a project whose derived stage is Cancelled, Archived, or Complete with the stale-state outcome (FEAT-25.SPEC-006).
- Validation at commit is authoritative over validation at form load -- the screen's own confirming dialog (FEAT-25.SPEC-002) is a first pass for user confirmation; this automation's step 2-3 re-check is what actually gates the write.
- Cancellation carries no financial reconciliation step of its own -- open, unpaid invoices on a cancelled project are untouched by this automation and are refunded separately, if needed, through FEAT-25.SPEC-003.

## Edge Cases

- **Two cancellation confirmations for the same project arrive from two open sessions of Nadia's at effectively the same time** -- The first to commit sets `cancelled_at`; the second re-checks at step 3, finds the stage already Cancelled, and is rejected with the stale-state outcome. Only one cancellation is ever recorded per project.
- **A trigger fires while a previous run for the same project is still in flight** -- FEAT-25.SPEC-002 disables Confirm Cancellation during submission, so a second run for the same project cannot start from the same screen instance; a second session's independent submission is handled by the concurrent-firing case above.
- **A milestone approval or proposal acceptance commits for this same project at effectively the same moment as this automation's own commit** -- Both events are recorded (the milestone is still marked Approved, the proposal still Accepted); FEAT-01.SPEC-011's precedence order resolves the displayed stage to Cancelled regardless of which event committed first, per the dependency map's Contention note.
- **Nadia marks the project Complete via FEAT-01.SPEC-006 at effectively the same moment this automation commits the cancellation** -- Whichever commits first sets its own field (`completed_at` or `cancelled_at`); the second commit's stale-state check (step 3 for this automation, which tests Complete via `completed_at`; FEAT-01.SPEC-006's own check for the reverse order) rejects the second attempt with its own screen's refresh message, so a project never carries both an explicit Complete and an explicit Cancelled transition from the same race.
- **Persisting `cancelled_at` fails after the stale-state check passes** -- No partial write occurs; the project's stage remains whatever it was before this automation ran, until a successful retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Triggered by (inbound) | Fires on a confirmed cancellation submission |
| FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | Affects (outbound) | Returns the new stage or a rejection outcome to the screen |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Displays the resulting Cancelled stage badge |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Triggers (outbound) / References (inbound) | This automation sets `cancelled_at`, which that spec's precedence-ordered formula reads to resolve the displayed stage |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Supplies the reject-with-refresh concurrency behavior this automation enforces at commit |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | Triggers (outbound) | A recorded cancellation fires Owen's notification email |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded cancellation writes the append-only trail entry (XBR-05) |

## Analytics and Success Signals

- **project_marked_cancelled** (project reference, reason provided: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so cancellation activity is observable rather than invisible.
- **project_cancellation_rejected** (reason: stale_state / automation_failure) -- N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation friction and concurrency collisions are observable rather than silent.

## Acceptance Criteria

**FEAT-25.SPEC-004-AC-01:** Given Nadia confirms cancellation on a project whose derived stage is none of Cancelled, Archived, or Complete, when this automation processes it, then `cancelled_at` is set and the project's derived stage resolves to Cancelled.

**FEAT-25.SPEC-004-AC-02:** Given a cancellation is recorded, when the Project Detail screen (FEAT-01.SPEC-005) is next opened, then it shows the Cancelled stage badge and every proposal, milestone, deliverable, and invoice record is unchanged.

**FEAT-25.SPEC-004-AC-03:** Given the project's derived stage at commit is Cancelled, Archived, or Complete (changed since the screen loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-002 shows the refresh message.

**FEAT-25.SPEC-004-AC-04:** Given a cancellation is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-004-AC-05:** Given persisting `cancelled_at` fails after the stale-state check passes, when the failure occurs, then FEAT-25.SPEC-002 shows "Couldn't cancel this project. Try again." and the project's prior stage remains displayed.

**FEAT-25.SPEC-004-AC-06:** Given two cancellation confirmations for the same project arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the stage already Cancelled and is rejected.

**FEAT-25.SPEC-004-AC-07:** Given a milestone is approved on this same project at effectively the same moment as this automation's commit, when both are processed, then the milestone remains Approved and the project's displayed stage still resolves to Cancelled.

**FEAT-25.SPEC-004-AC-08:** Given Nadia marks the same project Complete via FEAT-01.SPEC-006 at effectively the same moment this automation commits the cancellation, when both attempts race, then only the first to commit succeeds and the second is rejected with its own screen's refresh message; if Complete commits first, this automation's step 3 finds `completed_at` set, writes nothing, and FEAT-25.SPEC-002 shows "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-004-AC-09:** Given the project has an unaccepted proposal, when this automation records the cancellation, then the Proposal record itself is untouched -- it stays open exactly as it was (XBR-25).

**FEAT-25.SPEC-004-AC-10:** Given the project has open, unpaid invoices, when this automation records the cancellation, then those invoices are untouched.

**FEAT-25.SPEC-004-AC-11:** Given no reason accompanies the cancellation submission, when this automation commits, then the cancellation is still recorded with no reason stored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (recorded, stale-state, automation failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

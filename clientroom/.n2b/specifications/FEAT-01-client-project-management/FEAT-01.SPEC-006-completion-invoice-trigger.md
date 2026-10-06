---
document_type: spec
spec_type: automation
spec_id: FEAT-01.SPEC-006
spec_name: Completion Invoice Trigger
spec_slug: completion-invoice-trigger
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Completion Invoice Trigger

## Overview

**Name:** Completion Invoice Trigger
**ID:** FEAT-01.SPEC-006
**Type:** Automation
**Purpose:** Marking a project complete fires the on-completion invoice in Invoicing & Payments (FEAT-09) when the project's payment schedule includes one.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Detecting, at the moment a project is marked complete, whether its Payment Schedule includes an on-completion payment
- Handing off to Invoice Generation & Sending (FEAT-09) to create and send that invoice when applicable
- Setting the project's completed_at timestamp regardless of whether an invoice fires
- Recomputing the project's derived stage to "Complete" via FEAT-01.SPEC-011 once this automation resolves

**Non-Goals:**
- Generating the invoice content (amount, currency, tax line, numbering) -- owned entirely by FEAT-09; this automation only fires the trigger and hands off the schedule reference
- Deciding whether the freelancer is allowed to mark the project complete -- that decision is made on Project Detail (FEAT-01.SPEC-005), where the action originates
- Handling a project marked cancelled instead of complete -- owned by Refund & Cancelled Project Handling (FEAT-25), a distinct transition with its own trigger
- Retrying a failed invoice generation on a schedule -- excluded per scope-boundaries.md SC-11: the product ships fixed, sensible behavior rather than a configurable retry/automation builder; a failed hand-off surfaces to Nadia for a manual retry instead (see Edge Cases)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia marks a project complete | FEAT-01.SPEC-005 (Project Detail) | Fires when Nadia confirms Mark Complete on a project not already Complete, Cancelled, or Archived | Project reference, its Payment Schedule reference, current time |

## Processing Logic

1. Receive the project reference from the triggering Mark Complete confirmation on FEAT-01.SPEC-005.
2. Read the project's Payment Schedule (FEAT-04) as it stands at this exact moment.
3. Determine whether the schedule's structure includes an on-completion payment (a completion_amount is set).
4. If it does, hand off to Invoice Generation & Sending (FEAT-09) with the project reference and the schedule's completion_amount, requesting the on-completion invoice be created and sent (XBR-03).
5. Set the project's completed_at timestamp to the current time, regardless of whether step 4 ran.
6. Signal FEAT-01.SPEC-011 (Project Stage Derivation) to recompute the project's stage, which resolves to "Complete" now that completed_at is set.
7. Return the outcome to the triggering screen (FEAT-01.SPEC-005) so it can update its stage badge and confirmation feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Completed, invoice issued | Schedule includes an on-completion payment | completed_at set; stage recomputed to "Complete"; a new Invoice created and sent by FEAT-09 | Toast "Project marked complete" on FEAT-01.SPEC-005; the client sees the invoice arrive per FEAT-09's own notification | FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-09 |
| Completed, no invoice | Schedule has no on-completion payment | completed_at set; stage recomputed to "Complete"; no invoice created | Toast "Project marked complete" on FEAT-01.SPEC-005 | FEAT-01.SPEC-005, FEAT-01.SPEC-011 |
| Completion recorded, invoice hand-off failed | Schedule includes an on-completion payment but FEAT-09 cannot create the invoice | completed_at set; stage recomputed to "Complete"; no invoice created | Non-blocking warning on FEAT-01.SPEC-005: "Project marked complete, but the final invoice couldn't be issued. Retry from the invoices area." with a retry action into FEAT-09 | FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-09 |
| Automation failure before completion is recorded | The trigger itself fails before completed_at is set (e.g., the schedule cannot be read) | No data changes | Blocking error on FEAT-01.SPEC-005: "Couldn't mark this project complete. Try again." -- the project remains in its prior stage | FEAT-01.SPEC-005 |

## Data Model

**Reads:** Payment Schedule -- structure and completion_amount, as they stand at the moment of firing. Project -- identifier, current stage.
**Creates:** None directly -- the Invoice itself is created by FEAT-09, not by this automation.
**Updates:** Project -- completed_at (set once, never altered afterward); stage (recomputed via FEAT-01.SPEC-011).
**Deletes:** None.

## Business Rules

- XBR-03: marking a project complete generates the on-completion invoice when the schedule includes one; completed projects stay visible to the client until archived.
- The schedule is read as it stood at the moment of completion -- a schedule change saved afterward never retroactively creates or alters a completion invoice already issued, consistent with the dependency map's Contention note for Payment Schedule.
- completed_at, once set, is never altered by this automation; a later correction to the schedule or invoice is handled entirely within FEAT-04/FEAT-09, not by re-firing this trigger.
- System-driven stage changes never overwrite completed_at or a freelancer's explicit Complete transition (dependency map, Project Contention note).

## Edge Cases

- **Project has no Payment Schedule at all** -- Treated as "no on-completion payment"; completed_at is set and the stage recomputes to "Complete" with no invoice, since at least one payment trigger is required for any invoicing to exist per FEAT-04's validation, and its absence simply means nothing is owed at completion.
- **FEAT-09 is unable to create the invoice (e.g., a downstream validation failure)** -- Completion is not blocked or rolled back; completed_at and the stage change persist, and Nadia sees the non-blocking warning with a retry path, so a billing hiccup never traps the project in a stale "In Progress" state.
- **The project was already cancelled by FEAT-25 in another session before this trigger runs** -- The Mark Complete action itself is rejected upstream on FEAT-01.SPEC-005 with a refresh prompt before this automation ever fires; this automation never runs against a project already Cancelled.
- **Concurrent trigger firing (Mark Complete tapped from two open sessions of Nadia's at effectively the same time)** -- The triggering screen (FEAT-01.SPEC-005) rejects the second attempt with reject-with-refresh once the first has set completed_at, per the dependency map's Contention note for Project; this automation itself never runs twice for the same project.
- **Trigger fires while a previous run is in flight** -- Cannot occur: Mark Complete is disabled on FEAT-01.SPEC-005 while a prior completion attempt for the same project is processing, so a second run for the same project never starts before the first resolves.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Project Detail) | Triggered by (inbound) | Mark Complete confirmation fires this automation |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Affects (outbound) | Recomputes the project's stage after completed_at is set |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Supplies the Payment Schedule read at trigger time |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Owns creation and sending of the on-completion invoice |
| FEAT-25 (Refund & Cancelled Project Handling) | References (inbound) | A project it has cancelled cannot be completed by this automation |

## Analytics and Success Signals

- **completion_invoice_triggered** (had on-completion payment: yes/no) -- N/A -- no success-metrics.md metric tracks project-completion invoicing directly for this feature; the resulting invoice's own accuracy is measured under Invoice Generation & Sending's "Invoice Auto-Generation Accuracy" metric, not here.
- **completion_invoice_hand_off_failed** (reason: schedule unreadable / invoice service error) -- N/A -- no success-metrics.md metric in this feature's slice measures this failure path; it is tracked for reliability visibility only.

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Nadia marks a project complete and its payment schedule includes an on-completion payment, when the trigger fires, then the final invoice is created and sent via FEAT-09, completed_at is set, and the stage recomputes to "Complete."

**FEAT-01.SPEC-006-AC-02:** Given Nadia marks a project complete and its payment schedule has no on-completion payment, when the trigger fires, then completed_at is set and the stage recomputes to "Complete" with no invoice issued.

**FEAT-01.SPEC-006-AC-03:** Given a project has no Payment Schedule at all, when Nadia marks it complete, then it is treated as having no on-completion payment and completes with no invoice.

**FEAT-01.SPEC-006-AC-04:** Given the schedule includes an on-completion payment but FEAT-09 cannot create the invoice, when the trigger fires, then the project still completes and Nadia sees "Project marked complete, but the final invoice couldn't be issued. Retry from the invoices area."

**FEAT-01.SPEC-006-AC-05:** Given the automation fails before completed_at is set (e.g., the schedule cannot be read), when Nadia attempts to mark the project complete, then she sees "Couldn't mark this project complete. Try again." and the project remains in its prior stage.

**FEAT-01.SPEC-006-AC-06:** Given the payment schedule is later adjusted after a completion invoice has already been issued, then the already-issued invoice is unaffected, per the schedule-as-it-stood rule.

**FEAT-01.SPEC-006-AC-07:** Given Nadia has two sessions open on the same project and taps Mark Complete in both at effectively the same time, when the first completes, then the second is rejected with a refresh prompt rather than issuing a second completion invoice.

**FEAT-01.SPEC-006-AC-08:** Given a project was already cancelled by FEAT-25, when Nadia attempts Mark Complete, then the attempt is rejected upstream on FEAT-01.SPEC-005 and this automation never fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

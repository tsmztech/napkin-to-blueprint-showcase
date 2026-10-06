---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-004
spec_name: Next-Invoice Trigger
spec_slug: next-invoice-trigger
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Next-Invoice Trigger

## Overview

**Name:** Next-Invoice Trigger
**ID:** FEAT-08.SPEC-004
**Type:** Automation
**Purpose:** On a successful milestone approval, automatically hands off to invoice generation for the next invoice in the payment schedule, with no freelancer action, reading the schedule exactly as it stood at the moment of approval.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Firing immediately and only on a confirmed successful approval write
- Resolving which invoice, if any, the approved milestone's `payment_trigger` maps to in the Payment Schedule as it stood at the moment of approval
- Handing that resolved trigger off to invoice generation, with no freelancer action required
- Defining the no-action outcome for a milestone with no invoicing consequence

**Non-Goals:**
- Creating, numbering, formatting, or sending the invoice itself -- owned entirely by Invoice Generation & Sending (FEAT-09.SPEC-004, Automatic Invoice Generation), per XBR-02; this automation only fires the trigger and hands off the resolved schedule reference.
- Deciding whether the approval itself is valid or should be recorded -- owned by FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard); this automation only runs after that spec confirms a successful write.
- Correcting, cancelling, or crediting an invoice already issued by this trigger -- excluded per the Brief's Non-Goals: once this trigger fires, correcting the resulting invoice is FEAT-09's responsibility alone (credit note or new invoice, XBR-04); this feature has no invoice-editing capability of its own.
- Adjusting the Payment Schedule itself -- owned by FEAT-04 (Milestone & Payment Schedule Setup); this automation only reads the schedule as it stood at the moment of approval and never writes to it.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone approval successfully recorded | FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Fires immediately and only after FEAT-08.SPEC-003 confirms the approval write committed | The approved Milestone's reference, its `payment_trigger` value, and its Project reference (to resolve the owning Payment Schedule) |

## Processing Logic

1. Receive the confirmed-approval event from FEAT-08.SPEC-003, carrying the approved Milestone's reference and its `payment_trigger` value at the moment of approval.
2. Read the Project's Payment Schedule exactly as it currently stands (which, per the Payment Schedule Contention rule, is guaranteed to be the schedule as it stood at the moment of approval, since a later Nadia edit applies only to future triggers and never retroactively).
3. Determine whether the approved milestone's `payment_trigger` marks it as issuing an invoice on approval, per the schedule's structure.
4. If it does, hand off to FEAT-09.SPEC-004 (Automatic Invoice Generation) with the Milestone reference, its price (or the schedule's relevant amount), the Project, and the triggering event type "milestone approval."
5. If it does not (the milestone carries `no_separate_charge` or its `payment_trigger` is not set to invoice on approval), take no invoicing action and record the no-action outcome.
6. Return the outcome (handed off, or no action) for observability; this automation does not itself confirm invoice creation, numbering, or sending -- those confirmations belong to FEAT-09.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invoice generation triggered | The approved milestone's `payment_trigger` marks it as an invoice-on-approval milestone in the schedule as it stood at approval | None made directly by this automation -- FEAT-09.SPEC-004 owns the Invoice record itself | Nadia and Owen see the new invoice appear and its own notification arrive shortly after approval, through FEAT-09's own screens and notifications; no separate confirmation from this automation is shown | FEAT-09.SPEC-004 |
| No invoicing action (milestone carries no payment trigger) | The approved milestone's `payment_trigger` is not set to invoice on approval, or `no_separate_charge` is set | None | Nothing is shown -- approving a milestone with no payment trigger completes exactly like any other approval, with no invoice-related feedback of any kind | None -- silent no-action, consistent with the milestone carrying no charge |
| Hand-off failure (the automation cannot resolve or reach invoice generation) | A system error prevents resolving the schedule or reaching FEAT-09.SPEC-004 after the approval has already been recorded | None to the Milestone or Invoice; the approval itself is never rolled back | The approval stands ("Approved on {date}" remains visible to Owen); the missing invoice surfaces to Nadia as a delivery/processing warning on the project (consistent with XBR-30's pattern for a downstream failure that must never unwind an already-recorded evidentiary approval), and the hand-off is retried automatically | FEAT-08.SPEC-001 (approval display unaffected), FEAT-09 (retried hand-off) |

## Data Model

**Reads:** Milestone -- `payment_trigger`, `price`, `no_separate_charge`, Project reference. Payment Schedule -- `structure`, `deposit_amount`, `completion_amount`, as they stand at the moment of approval.
**Creates:** None -- Invoice creation belongs entirely to FEAT-09.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-02: Approving a milestone automatically generates and sends the next invoice in the payment schedule, with no freelancer action -- this automation is the FEAT-08 half of that rule; FEAT-09 owns invoice creation itself.
- The schedule used is the schedule as it stood at the moment of approval (feature-dependency-map.md, Payment Schedule Contention note): a schedule edit Nadia saves afterward is dated and applies only to later triggers, never retroactively to this one.
- This automation fires unconditionally on every confirmed successful approval -- it is never skipped, delayed, or made optional by a freelancer setting, per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior rather than a configurable workflow.
- A milestone with no payment trigger produces no invoice and no error -- silence is the correct behavior for a milestone that was never meant to bill separately.
- A hand-off failure never reverses or delays display of the already-recorded approval (XBR-04): the approval's immutability and evidentiary status do not depend on whether the downstream invoice succeeds.

## Edge Cases

- **The approved milestone carries `no_separate_charge`** -- No invoice is generated; the approval completes with no invoicing feedback of any kind.
- **The Payment Schedule has no payment trigger configured at all at the time of approval** -- No invoice is generated; this is a pre-existing condition surfaced earlier by FEAT-04.SPEC-003's non-blocking payment-trigger indicator, not a new failure introduced here.
- **Nadia edits the Payment Schedule at the exact moment Owen's approval is being recorded** -- Per the Payment Schedule Contention rule, this trigger resolves against the schedule as it stood at approval; Nadia's concurrent edit is dated and takes effect only for triggers that fire after her edit commits, never for this one.
- **Concurrent trigger firing -- two different milestones in the same project are approved by Owen at effectively the same time** -- Each approval is a separate, independently confirmed write (FEAT-08.SPEC-003's exactly-once guarantee is per-milestone); this automation runs once per approval and hands off two independent invoice-generation requests to FEAT-09, which processes them independently.
- **This trigger fires while a previous run for a different milestone is still in flight** -- Runs for different milestones proceed independently; there is no shared state between them beyond both reading the same Project's Payment Schedule, which is read-only from this automation's perspective and therefore never contended between the two runs.
- **FEAT-09.SPEC-004 is unreachable at the moment this automation attempts the hand-off** -- The approval remains recorded and visible; the hand-off is retried automatically, and if retries are exhausted the missing invoice surfaces to Nadia as a project-level warning rather than being silently lost.
- **The milestone is later reopened and re-approved** -- Each approval independently re-evaluates the schedule as it stands at that moment and, if the milestone's `payment_trigger` still applies, hands off again; this automation carries no memory of a prior hand-off from an earlier approval cycle on the same milestone, since a genuinely new approval is a new triggering event.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggered by (inbound) | A confirmed successful approval write fires this automation |
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | Affects (outbound) | Hands off the resolved trigger and milestone/schedule data for invoice creation, numbering, and sending (XBR-02) |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Reads the Payment Schedule and the milestone's `payment_trigger`, as they stood at the moment of approval |

## Analytics and Success Signals

- **milestone_invoice_auto_generated** (milestone reference, invoice trigger type) -- supports success-metrics.md: "Milestone Approval Turnaround" (a fired trigger with no delay is the visible half of a fast approval-to-billing loop; a stalled or failed hand-off would otherwise look identical to a slow approval from the freelancer's perspective)
- **milestone_invoice_trigger_skipped** (reason: no_payment_trigger / no_separate_charge) -- N/A -- no Stage 2 metric measures milestones with no invoicing consequence; retained so a milestone approval with no invoice is distinguishable from a hand-off failure.
- **milestone_invoice_handoff_failed** (retry_count) -- N/A -- no Stage 2 metric covers infrastructure hand-off failures directly; retained because "Invoice Auto-Generation Accuracy" (product-features.md, FEAT-09) measures correctness of invoices that were generated, not invoices that failed to generate at all -- this event is this feature's only signal of that failure mode.

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given Owen's approval of a milestone with `payment_trigger` set to invoice on approval is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then this automation hands off to FEAT-09.SPEC-004 immediately, with no action required from Nadia.

**FEAT-08.SPEC-004-AC-02:** Given the approved milestone carries `no_separate_charge`, when the approval is recorded, then no invoice hand-off occurs and no invoice-related feedback appears anywhere.

**FEAT-08.SPEC-004-AC-03:** Given the Payment Schedule has no configured payment trigger at all, when a milestone under it is approved, then no invoice hand-off occurs.

**FEAT-08.SPEC-004-AC-04:** Given Nadia edits the Payment Schedule at the same moment Owen's approval is being recorded, when this automation resolves the trigger, then it uses the schedule exactly as it stood at the moment of approval, not Nadia's concurrent edit.

**FEAT-08.SPEC-004-AC-05:** Given a schedule edit Nadia saved earlier is dated after the approval being processed, when this automation runs, then it is unaffected by that later edit, consistent with the non-retroactive rule.

**FEAT-08.SPEC-004-AC-06:** Given Owen approves two different milestones in the same project at effectively the same time, when both approvals are confirmed, then this automation fires once per approval and hands off two independent invoice-generation requests.

**FEAT-08.SPEC-004-AC-07:** Given FEAT-09.SPEC-004 is unreachable at the moment of hand-off, when the failure occurs, then the approval remains recorded and visible to Owen, the hand-off is retried automatically, and Nadia sees a project-level warning if retries are exhausted.

**FEAT-08.SPEC-004-AC-08:** Given a milestone is reopened and re-approved, when the second approval is confirmed, then this automation re-evaluates the schedule fresh and hands off again if the milestone's `payment_trigger` still applies, independent of any hand-off from the first approval cycle.

**FEAT-08.SPEC-004-AC-09:** Given a milestone's `payment_trigger` marks it to invoice on approval, when the automation completes its hand-off, then the milestone_invoice_auto_generated event is emitted with the milestone reference and trigger type.

**FEAT-08.SPEC-004-AC-10:** Given this automation runs for two milestones approved at effectively the same time, when both runs read the Project's Payment Schedule, then neither run's read is contended by the other, since the schedule is read-only from this automation's perspective.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (triggered, no action, hand-off failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-10.SPEC-005
spec_name: Record Off-Platform Payment
spec_slug: record-off-platform-payment
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Record Off-Platform Payment

## Overview

**Name:** Record Off-Platform Payment
**ID:** FEAT-10.SPEC-005
**Type:** Automation
**Purpose:** Validates and persists Nadia's manual payment record -- full invoice amount, not future-dated -- and sets the invoice to "Paid (recorded by freelancer)."
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- Validating Nadia's entered date and method against the full-payment-only and not-future-dated limits
- Checking the invoice is still eligible for a manual record at the moment of save (not already Paid by any path)
- Creating the Payment record with `status` = Recorded manually and setting Invoice `status` to Paid (recorded by freelancer)

**Non-Goals:**
- Collecting the date and method input -- owned by FEAT-10.SPEC-002 (Record Off-Platform Payment Screen); this automation begins only once Nadia taps Save.
- Processing a card or bank-transfer payment -- owned by FEAT-10.SPEC-003 and FEAT-10.SPEC-004; this automation never touches the payment-processing capability.
- Allowing a manual record to be edited or reversed after it is saved -- Payment and Invoice records are evidentiary and never silently altered (assumptions-constraints.md, ASMP-25); a correction is a decision the product definition does not make available in this feature.
- Recording anything other than a full-invoice-amount payment -- excluded per scope-boundaries.md (SC-17); no partial-amount manual record exists.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Save record tapped | FEAT-10.SPEC-002 (Record Off-Platform Payment Screen) | Fires when Nadia taps "Save record" with a date and method entered | Invoice reference, entered `paid_at` (date), entered `method` |

## Processing Logic

1. Receive the invoice reference, entered date, and entered method from FEAT-10.SPEC-002.
2. Validate the date is not in the future (FEAT-10.SPEC-006). If invalid, return the validation-failure outcome to the screen without creating a record.
3. Validate a method was selected (FEAT-10.SPEC-006). If invalid, return the validation-failure outcome without creating a record.
4. Read the Invoice's current `status`. If it is already Paid or Paid (recorded by freelancer), stop and return the refused outcome -- the invoice was resolved by another path since Nadia opened this screen (reject-with-refresh, XBR-20).
5. Create a Payment record: `invoice` = the referenced invoice, `amount` = the invoice's full `total` (never a partial or user-entered amount), `method` = the entered method, `paid_at` = the entered date, `status` = Recorded manually, `recorded_by` = Nadia.
6. Set Invoice `status` to Paid (recorded by freelancer).
7. Fire the reminder-stop trigger toward FEAT-11 (Automated Payment Reminders), since the invoice has reached a Paid-family status.
8. Return the success outcome, visible to Nadia immediately and to Owen the next time he views the invoice (FEAT-10.SPEC-001).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Manual record saved | Validation passes and the invoice is still eligible | Payment record created (Recorded manually); Invoice status set to Paid (recorded by freelancer); reminder-stop trigger fired | Nadia sees "Payment recorded" and returns to invoice detail showing the new status; Owen sees the same status on FEAT-10.SPEC-001 on next view | FEAT-10.SPEC-002, FEAT-10.SPEC-001, FEAT-11 |
| Validation failure (future date) | Entered date is later than today | None | FEAT-10.SPEC-002 shows "Payment date cannot be in the future." on the date field | FEAT-10.SPEC-002 |
| Validation failure (no method) | No method selected | None | FEAT-10.SPEC-002 shows a required-field error on the method selector | FEAT-10.SPEC-002 |
| Refused -- already paid online | Invoice status is already Paid at the moment of save | None | FEAT-10.SPEC-002 shows "This invoice was already paid online. Refresh to see the current status." | FEAT-10.SPEC-002, FEAT-10.SPEC-001 |
| Automation failure | Processing cannot complete (for example, the write cannot be committed) | No partial write -- the Payment record and the Invoice status change either both commit or neither does | FEAT-10.SPEC-002 shows "Could not save this record. Check your connection and try again." | FEAT-10.SPEC-002 |

## Data Model

**Reads:** Invoice -- `status`, `total` (for the recorded amount).
**Creates:** Payment -- `invoice`, `amount`, `method`, `paid_at`, `status` (Recorded manually), `recorded_by`.
**Updates:** Invoice -- `status` (set to Paid (recorded by freelancer)).
**Deletes:** None.

## Business Rules

- The recorded amount is always the invoice's full total -- there is no path to record a partial amount (scope-boundaries.md SC-17).
- The recorded date cannot be in the future (FEAT-10.SPEC-006).
- Processor-confirmed payment status is authoritative over a concurrent manual entry: if the invoice reached Paid through FEAT-10.SPEC-004 before this save commits, the save is refused rather than silently overwriting the processor's record (XBR-20).
- This automation runs synchronously with Nadia's Save action -- she sees the outcome (success, validation error, or refusal) before leaving the screen.
- Once saved, a manual record is permanent; this automation defines no update or delete path for a record it has created (ASMP-25).

## Edge Cases

- **Nadia submits the form while the invoice is simultaneously marked Paid by the payment-processing capability (concurrent trigger firing)** -- Whichever write reaches the Invoice record first wins; if FEAT-10.SPEC-004's processor-confirmed write commits first, this automation's step 4 check detects the already-Paid status and returns the refused outcome rather than overwriting it. If this automation's write commits first, FEAT-10.SPEC-004's own already-resolved guard (FEAT-10.SPEC-004, step 2) then treats a subsequent genuine processor Succeeded event as a discrepancy rather than silently overwriting Nadia's manual record.
- **Nadia triggers this automation twice in quick succession for the same invoice (a trigger fires while a previous run is in flight)** -- The second run's step 4 check sees the first run's already-committed Paid (recorded by freelancer) status and returns the refused outcome; only one Payment record is ever created for a given manual save.
- **The entered date is exactly today** -- Passes validation; "not in the future" is inclusive of the current date.
- **The invoice's due date has already passed (Overdue) when Nadia records it** -- Not applicable as a distinct case; Overdue is an Invoice-level flag owned by FEAT-11 and does not block or alter this automation's eligibility check, which looks only at whether the invoice has already reached a Paid-family status.
- **The write to create the Payment record succeeds but the Invoice status update fails** -- Not possible as a partial state: both writes are applied together or neither is, so a Payment record is never left orphaned against an invoice still showing Sent or Overdue.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Record Off-Platform Payment Screen) | Triggered by (inbound) | Save record initiates this automation |
| FEAT-10.SPEC-002 (Record Off-Platform Payment Screen) | Affects (outbound) | Success, validation-failure, and refused outcomes are shown there |
| FEAT-10.SPEC-006 (Payment Authorization & Validation Rules) | References (inbound) | Full-payment-only, not-future-dated, and processor-authoritative-over-manual rules |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Affects (outbound) | A successful record is what Owen's screen shows on next view |
| FEAT-11 (Automated Payment Reminders) | Affects (outbound) | Reminder-stop trigger fired on a successful manual record |

## Analytics and Success Signals

- **payment_recorded_manually** (method) -- supports success-metrics.md: "Time to Payment"
- **payment_recorded_manually_validation_failed** (reason: future_date / no_method) -- N/A -- no Stage 2 metric measures manual-record input errors; retained as a standard input-quality signal.
- **payment_recorded_manually_refused** (reason: already_paid_online) -- N/A -- no Stage 2 metric measures this concurrency outcome directly; retained so the frequency of the processor-authoritative rule being exercised is observable.

## Acceptance Criteria

**FEAT-10.SPEC-005-AC-01:** Given Nadia enters a valid date and method for an unpaid invoice, when she saves, then a Payment record is created with the invoice's full total, `status` Recorded manually, and `recorded_by` Nadia, and the Invoice status is set to Paid (recorded by freelancer).

**FEAT-10.SPEC-005-AC-02:** Given Nadia enters a future date, when she attempts to save, then no Payment record is created and FEAT-10.SPEC-002 shows "Payment date cannot be in the future."

**FEAT-10.SPEC-005-AC-03:** Given Nadia leaves the method unselected, when she attempts to save, then no Payment record is created and a required-field error is shown.

**FEAT-10.SPEC-005-AC-04:** Given the invoice was marked Paid by the payment-processing capability moments before Nadia's save commits, when the save is processed, then it is refused with "This invoice was already paid online. Refresh to see the current status." and no Payment record is created.

**FEAT-10.SPEC-005-AC-05:** Given Nadia's save succeeds, when the write completes, then the reminder-stop trigger fires toward FEAT-11 and Owen sees the new status on his next view of FEAT-10.SPEC-001.

**FEAT-10.SPEC-005-AC-06:** Given Nadia enters today's date, when she saves, then the date passes validation (the future-date rule is inclusive of today).

**FEAT-10.SPEC-005-AC-07:** Given this automation fails mid-write, when the failure occurs, then no Payment record is left orphaned against an invoice whose status was not also updated.

**FEAT-10.SPEC-005-AC-08:** Given Nadia's save and the payment-processing capability's confirmation arrive at effectively the same time, when both attempt to write, then only one status change is applied and the losing write is refused rather than silently overwritten.

**FEAT-10.SPEC-005-AC-09:** Given a second save attempt for the same invoice fires while the first is still in flight, when the second run checks the invoice's status, then it finds the first run's already-committed status and returns the refused outcome.

**FEAT-10.SPEC-005-AC-10:** Given Nadia has successfully recorded a manual payment, when she or anyone else looks for an edit or undo control on that record, then none exists -- the record is permanent.

**FEAT-10.SPEC-005-AC-11:** Given an invoice's due date has already passed when Nadia records it, when the automation processes the save, then the Overdue flag has no effect on the outcome.

**FEAT-10.SPEC-005-AC-12:** Given a manual record is saved successfully, when the write completes, then the payment_recorded_manually event fires, supporting the "Time to Payment" metric.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

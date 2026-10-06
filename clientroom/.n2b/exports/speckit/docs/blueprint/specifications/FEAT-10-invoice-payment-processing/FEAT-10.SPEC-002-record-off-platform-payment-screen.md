---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-002
spec_name: Record Off-Platform Payment Screen
spec_slug: record-off-platform-payment-screen
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Record Off-Platform Payment Screen

## Overview

**Name:** Record Off-Platform Payment Screen
**ID:** FEAT-10.SPEC-002
**Type:** Screen
**Purpose:** Nadia records that an invoice was paid outside the portal, entering the date and method, so the payment is logged and never silent.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- A form for Nadia to record an off-platform payment (date, method) against one of her own invoices
- Showing why the record is being made this way, and what will happen to the invoice's status on save
- Validation feedback for the full-payment-only and not-future-dated limits (governed by FEAT-10.SPEC-006)

**Non-Goals:**
- Entering a partial amount -- excluded per scope-boundaries.md (SC-17): a manually recorded payment must be the full invoice amount; this screen offers no amount field to change.
- In-portal card or bank-transfer payment -- owned by FEAT-10.SPEC-001 (Pay Invoice Screen); this screen exists only for money Owen already sent Nadia outside the portal.
- Validating and persisting the record -- owned by FEAT-10.SPEC-005 (Record Off-Platform Payment); this screen collects the input and displays the automation's outcome.
- Editing or reversing a manual record once saved -- Payment and Invoice records are evidentiary and never silently altered (assumptions-constraints.md, ASMP-25); a correction would require a distinct, logged event this feature does not define, consistent with the entity-lifecycle matrix showing no Update path for a manual record after it is created.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09 (Invoice Generation & Sending) -- invoice detail (freelancer side) | Nadia opens an unpaid invoice and chooses "Record a payment received elsewhere" | Invoice reference; invoice total and currency pre-filled for display |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Enters date and method, and saves the manual record | -- |
| Owen (Client Primary Contact) | No -- this is a freelancer-only workspace screen; Own-only Invoicing & Payments for Owen covers viewing and paying his own company's invoice (FEAT-10.SPEC-001), not recording on Nadia's behalf | No | -- (not a denial -- Owen has no equivalent action; he only sees the resulting status on FEAT-10.SPEC-001) |
| Priya (Client Reviewer Contact) | No | No | -- (not applicable -- Reviewer contacts have no Invoicing & Payments access at all, and this screen is not client-facing) |
| Dana (Support Operator) | View only, inside a logged support session (FEAT-31) -- sees that a manual record exists and its date/method, never a control to create one | No | Save control is not shown; any attempt to reach this exact freelancer screen is not possible from a support session, which presents Nadia's data through FEAT-31's own read-only interface |
| Unauthenticated | No | No | Redirected to freelancer sign-in; no invoice content shown first |
| Expired session | No | No | Freelancer session-expiry handling (owned by FEAT-21/settings sign-in) applies; entered but unsaved form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Record a payment" with the invoice number and client name, and a back arrow returning to the invoice detail (FEAT-09).

**Body:** A single-column form:
- Read-only summary line: "{total} {currency} for invoice {invoice_number}" -- confirms the amount that will be recorded, since no amount field is editable.
- Payment date (date input, required, defaults to today, cannot be a future date)
- Payment method (selection input, required: "Bank transfer," "Cash," "Cheque," "Other")
- Explanatory note, directly below the fields: "This records that {client_name} paid you outside Clientroom. The invoice will show as Paid (recorded by freelancer), and this entry is logged permanently."

**Footer:** "Save record" button (primary) and "Cancel" (returns to invoice detail without saving).

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; Save and Cancel stack with Save on top.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; Save and Cancel sit side by side in the footer.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the invoice detail (FEAT-09) without saving | Screen closes | Returns to invoice detail unchanged |
| Payment date input | Select a date | Captures the date | Field shows the selected date | Standard date-picker feedback |
| Payment date input | Select a future date | Triggers validation via FEAT-10.SPEC-006 | Error state on field | "Payment date cannot be in the future." shown below the field |
| Payment method selector | Select an option | Captures the method | Field shows the selected method | Selected method displayed |
| Save record button | Tap | 1. Validate date and method via FEAT-10.SPEC-006. 2. If valid, trigger FEAT-10.SPEC-005 (Record Off-Platform Payment). | Button shows a saving state; form fields disabled | Success: "Payment recorded" confirmation and navigation to invoice detail, now showing Paid (recorded by freelancer). Failure: inline error message (validation) or a refused-with-refresh message (invoice already paid online). |
| Save record button (while saving) | Tap | No action -- ignored while a save is already in flight | None | Button remains in its saving state |
| Cancel button | Tap | Navigate to invoice detail without saving | Screen closes | Returns to invoice detail unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> summary line (read-only, not a focus stop) -> Payment date input -> Payment method selector -> Save record button -> Cancel button.
- **Validation announcements:** When a field enters an error state, its message is announced to assistive technology and associated with the field.
- **Save feedback:** The "Payment recorded" confirmation is announced on success; on validation failure or a refused save, focus moves to the first field or message in error.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; the date input and method selector have no pointer-only equivalents.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Filling (default) | Date defaulted to today, method unselected, Save enabled | Screen first opens | Nadia changes the date, selects a method, or taps Save |
| Validation Error | Failing field(s) highlighted with error messages below them | Save attempted with an invalid date or no method selected | Nadia corrects the field(s) and re-triggers validation |
| Saving | Save button shows a saving indicator, fields disabled | Validation passes and Save is triggered | The record is created (success) or the save is refused |
| Refused (invoice already paid online) | Blocking message in place of the form: "This invoice was already paid online. Refresh to see the current status." with a "View invoice" action | FEAT-10.SPEC-005 refuses the save because the processor already marked the invoice Paid (processor-authoritative rule, XBR-20) | Nadia taps "View invoice," returning to invoice detail with the current status |
| Error | Error banner: "Could not save this record. Check your connection and try again." with a Retry button; entered date and method preserved | The save operation fails for a reason other than the refused-with-refresh case | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this payment record will be saved when you reconnect." at top; form remains editable; Save queues the record locally | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-006 (Payment Authorization & Validation Rules). See that spec for the full-payment-only and not-future-dated limits and the processor-authoritative-over-manual reject-with-refresh rule.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Invoice detail | FEAT-09 (Invoice Generation & Sending) |
| Successful save | Invoice detail (now showing Paid (recorded by freelancer)) | FEAT-09 (Invoice Generation & Sending) |
| Cancel tap | Invoice detail (unchanged) | FEAT-09 (Invoice Generation & Sending) |
| "View invoice" (refused save) | Invoice detail (current status) | FEAT-09 (Invoice Generation & Sending) |

## Data Model

**Creates:** None directly -- this screen collects input; FEAT-10.SPEC-005 creates the Payment record.
**Reads:** Invoice -- `invoice_number`, `total`, `currency`, `status` (to confirm it is still eligible for a manual record). Client -- `client_name` (for the explanatory note).
**Updates:** None directly -- FEAT-10.SPEC-005 updates Invoice `status` on a successful save.
**Deletes:** None.

## Business Rules

- Only Nadia may record an off-platform payment; the action is never offered to any client contact or to Dana, per FEAT-10.SPEC-006's Authorization Rules.
- The recorded amount is always the invoice's full total -- there is no amount field, since manual records follow the same full-payment-only rule as in-portal payments (scope-boundaries.md SC-17).
- The payment date cannot be in the future (FEAT-10.SPEC-006).
- If the invoice has already been marked Paid by the payment-processing capability before this save commits, the save is refused: processor-confirmed status is authoritative over a concurrent manual entry (XBR-20, FEAT-10.SPEC-006).
- Saving is a permanent, logged action -- it cannot be undone from this screen, per the record-immutability constraint (assumptions-constraints.md, ASMP-25).

## Edge Cases

- **Nadia navigates away with the date or method entered but unsaved** -- No confirmation dialog is required; no destructive loss occurs, since nothing has been recorded yet and the invoice remains in its prior state.
- **Nadia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress.
- **The invoice is marked Paid by the payment-processing capability at the same moment Nadia taps Save (concurrent-edit conflict)** -- The save is rejected with the Refused state message: "This invoice was already paid online. Refresh to see the current status." Resolution: reject-with-refresh, per the dependency map's Contention note for the Payment entity -- processor-confirmed status wins over a concurrent manual entry.
- **Network failure during save** -- Error banner: "Could not save this record. Check your connection and try again." with a Retry button. Entered date and method are preserved.
- **Nadia opens this screen for an invoice that is already "Paid (recorded by freelancer)"** -- Not applicable; the entry point (invoice detail) does not offer "Record a payment received elsewhere" once an invoice already carries any Paid-family status, so this screen is never reached for an already-recorded invoice.
- **Nadia selects "Other" as the method** -- No free-text elaboration is collected; the record simply carries `method` = the recorded off-platform method value "Other," consistent with the dependency map's Payment.method field.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-005 (Record Off-Platform Payment) | Triggers (outbound) | Save record initiates validation and persistence of the manual record |
| FEAT-10.SPEC-006 (Payment Authorization & Validation Rules) | References (inbound) | Full-payment-only, not-future-dated, and processor-authoritative-over-manual rules |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | References (outbound) | The resulting "Paid (recorded by freelancer)" status is what Owen sees on that screen |
| FEAT-09 (Invoice Generation & Sending) | Navigation (inbound/outbound) | Entry from and return to invoice detail |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| payment_recorded_manually_attempted | method | Nadia taps Save record | supports success-metrics.md: "Time to Payment" |
| payment_recorded_manually_refused | reason: already_paid_online | The save is refused because the processor already marked the invoice Paid | N/A -- no Stage 2 metric measures this concurrency outcome directly; retained so the frequency of the processor-authoritative rule being exercised is observable |

## Acceptance Criteria

**FEAT-10.SPEC-002-AC-01:** Given Nadia opens an unpaid invoice, when she chooses "Record a payment received elsewhere," then this screen shows the invoice total, a date field defaulted to today, and an unselected method field.

**FEAT-10.SPEC-002-AC-02:** Given Nadia selects a payment date and method and taps "Save record," when the save succeeds, then she sees "Payment recorded" and the invoice detail shows "Paid (recorded by freelancer)."

**FEAT-10.SPEC-002-AC-03:** Given Nadia selects a future date, when she attempts to save, then the date field shows "Payment date cannot be in the future." and the save does not proceed.

**FEAT-10.SPEC-002-AC-04:** Given Nadia leaves the method field unselected, when she taps Save, then a validation error is shown and the save does not proceed.

**FEAT-10.SPEC-002-AC-05:** Given the payment-processing capability marks this invoice Paid at the same moment Nadia taps Save, when the save is processed, then it is refused with "This invoice was already paid online. Refresh to see the current status."

**FEAT-10.SPEC-002-AC-06:** Given Nadia taps Save twice rapidly, when the second tap occurs, then it has no effect and only one Payment record is created.

**FEAT-10.SPEC-002-AC-07:** Given a network failure occurs during save, when the failure is detected, then Nadia sees "Could not save this record. Check your connection and try again." with her entered date and method preserved.

**FEAT-10.SPEC-002-AC-08:** Given Nadia loses connectivity while filling this form, when she taps Save, then the offline banner appears and the record is submitted automatically once connectivity returns.

**FEAT-10.SPEC-002-AC-09:** Given Owen (Client Primary Contact) has no path to this screen, when he views his own invoice, then no "Record a payment" action is ever shown to him.

**FEAT-10.SPEC-002-AC-10:** Given Dana (Support Operator) is inside a logged support session, when she views this invoice, then she sees that a manual record exists with its date and method, but no Save control is available to her.

**FEAT-10.SPEC-002-AC-11:** Given Nadia taps "Cancel" with fields filled in, when the tap is processed, then she returns to invoice detail and no record is created.

**FEAT-10.SPEC-002-AC-12:** Given Nadia selects "Other" as the method, when she saves successfully, then the Payment record's method is stored as "Other" with no additional free-text field required.

**FEAT-10.SPEC-002-AC-13:** Given Nadia's session expires while she has entered a date and method, when she re-authenticates, then her entered but unsaved data is restored on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (filling, validation error, saving, refused, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

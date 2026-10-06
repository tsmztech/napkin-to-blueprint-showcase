---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-003
spec_name: Manual Invoice & Credit Note Issuance
spec_slug: manual-invoice-credit-note-issuance
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Manual Invoice & Credit Note Issuance

## Overview

**Name:** Manual Invoice & Credit Note Issuance
**ID:** FEAT-09.SPEC-003
**Type:** Screen
**Purpose:** Lets Nadia issue an ad-hoc invoice outside the payment schedule, or a credit note correcting a previously sent invoice, with an adjustable due date.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- The form for a new ad-hoc invoice: description, amount, due date (defaulted, adjustable)
- The form for a credit note against a specific prior invoice, pre-selected when entered from FEAT-09.SPEC-002's correction action
- Submitting either form to FEAT-09.SPEC-005 for validation and recording
- Field-level and submit-time feedback while the submission is processed

**Non-Goals:**
- Validating and recording the submitted invoice or credit note -- owned entirely by FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording); this screen only collects and submits the input
- Automatic invoice generation from a payment-schedule trigger -- owned by FEAT-09.SPEC-004; this screen exists only for issuance outside that schedule
- Choosing which invoice to correct from a general search -- excluded per scope-boundaries.md's Deferral Notes (search is a v1-phase feature, FEAT-28); a credit note is always entered against one specific invoice, either pre-selected from FEAT-09.SPEC-002 or picked from this project's own invoice list (FEAT-09.SPEC-001)
- Recurring or calendar-scheduled billing -- excluded per scope-boundaries.md (SC-14): this screen issues one-off invoices only; an occasional retainer charge is issued here as a single ad-hoc invoice, not a recurring schedule

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-001 (Invoice List) | Nadia taps "New Invoice" | Project reference; form opens in "new ad-hoc invoice" mode with no invoice pre-selected |
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia taps "Issue Credit Note" | The specific invoice reference to correct; form opens in "credit note" mode with that invoice pre-selected and its amount shown for reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Submit an ad-hoc invoice or a credit note | -- |
| Owen (Client Primary Contact) | None -- issuance is a freelancer-only action per the Access Matrix | None | This screen has no entry point in Owen's portal; a direct link redirects him to his portal home |
| Priya (Client Reviewer Contact) | None | None | Same as Owen -- redirected to portal home |
| Dana (Support Operator) | None -- issuing an invoice is excluded from support access per BRIEF.md's "read-only, nothing more" and XBR-29 | None | This screen is never rendered inside a support session; Dana's read-only access stops at FEAT-09.SPEC-001 and FEAT-09.SPEC-002 |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Nadia is redirected to sign-in; entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title -- "New Invoice" or "Issue Credit Note" depending on mode -- with a back control and a "Submit" action (right-aligned).

**Body -- Ad-hoc invoice mode:**
- Description (text input, required) -- what the charge is for
- Amount (numeric input, required, positive, in the project's set currency)
- Due date (date input, pre-filled from Nadia's `default_payment_terms`, adjustable)

**Body -- Credit note mode:**
- Read-only reference block showing the invoice being corrected: its number, amount, and issue date
- Credit amount (numeric input, required, positive, capped at the original invoice's total)
- Reason (text input, required) -- shown to Owen on the resulting credit note
- Due date -- not applicable to a credit note; this field is not shown in credit-note mode

All fields use one consistent input treatment platform-wide. Required fields are marked with a visual indicator.

**Footer:** None -- Submit is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; Submit remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to entry point | Screen closes | Confirmation dialog if the form has unsaved input |
| Description / Reason input | Type | Captures text | Field shows entered text | Standard input focus state |
| Amount / Credit amount input | Blur (empty or non-positive) | Triggers validation via FEAT-09.SPEC-007 | Error state on field | "Enter an amount greater than zero." |
| Due date input (ad-hoc only) | Change | Sets the due date, overriding the default-terms pre-fill | Field shows chosen date | Standard input state |
| Submit | Tap | 1. Validate all fields per FEAT-09.SPEC-007. 2. If valid, submit to FEAT-09.SPEC-005 for recording. | Submit shows loading state | Success: navigates to FEAT-09.SPEC-002 for the newly recorded invoice/credit note. Failure: inline error messages, or FEAT-09.SPEC-005's specific rejection reason (e.g., credit amount exceeds the original invoice) |
| Submit (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order (ad-hoc mode):** Back -> Description -> Amount -> Due date -> Submit.
- **Focus order (credit-note mode):** Back -> reference block -> Credit amount -> Reason -> Submit.
- **Validation announcements:** A field entering an error state has its message announced to assistive technology and associated with the field.
- **Keyboard alternatives:** Every action is keyboard-reachable; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton over the due-date field (ad-hoc mode) or over the reference block (credit-note mode), while `default_payment_terms` (ad-hoc) or the original invoice's `invoice_number`/`total`/`issue_date` (credit-note) are fetched; other fields render immediately since they carry no fetched data | Screen first opens | The relevant fetch completes, populating the pre-fill or reference block |
| Empty (default) | Form fields empty except the due-date pre-fill (ad-hoc mode) or the reference block (credit-note mode); Submit enabled | Loading completes | User begins typing or taps Submit |
| Filling | Fields contain user input | User types or changes a field | Submit tapped or user navigates away |
| Submitting | Submit shows a loading indicator, fields disabled | Submit tapped and client-side validation passes | FEAT-09.SPEC-005 returns an outcome |
| Validation Error | Failed fields highlighted with error messages | Client-side validation fails, or FEAT-09.SPEC-005 rejects the submission | User corrects the field(s) |
| Error | Error banner "Couldn't submit this invoice. Check your connection and try again." with Retry, form data preserved | Submission fails for a reason other than validation (e.g., connectivity) | User taps Retry and the resubmission succeeds |
| Offline/Degraded | A "You're offline -- reconnect to submit" banner is shown; the form remains editable but Submit is disabled; the action never appears to succeed while offline | Connectivity is lost while the screen is open, or the screen is opened while offline | Connectivity returns and Submit becomes available again |

## Validation Rules

Validation governed by FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules). See that spec for the amount-positivity rule, the due-date derivation and override rule, and the credit-amount-cannot-exceed-original rule. This screen applies validation on field blur and on submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control (no unsaved changes) | Entry point (FEAT-09.SPEC-001 or FEAT-09.SPEC-002) | -- |
| Successful submission | FEAT-09.SPEC-002 (Invoice Detail) for the newly recorded invoice or credit note | -- |
| Cancel with unsaved changes | Entry point, after confirmation | -- |

## Data Model

**Creates:** None directly -- this screen submits input to FEAT-09.SPEC-005, which creates the Invoice record (ad-hoc invoice or credit note).
**Reads:** Invoice (credit-note mode only) -- `invoice_number`, `total`, `issue_date` of the invoice being corrected, for the read-only reference block. Freelancer Account -- `default_payment_terms`, to pre-fill the due date.
**Updates:** None directly.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-007 governs numbering, required content, amount validation, and due-date derivation -- this screen applies it rather than re-deriving it.
- FEAT-09.SPEC-008 governs correction rules: a credit note is always a new, linked Invoice record; this screen never edits the original invoice's fields.
- Submission is disabled while offline (per the States table), consistent with the Brief's Side-Effect Inventory: "the action never appears to succeed" without connectivity.
- Only Nadia can reach this screen, per the Access Matrix -- issuance is never exposed to any client contact or to Dana.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Submit twice rapidly** -- Second tap is ignored while the first submission is in progress.
- **Credit amount entered exceeds the original invoice's total** -- Blocked at submit time by FEAT-09.SPEC-007 with "A credit note cannot exceed the original invoice's total." Field-level error shown; submission does not proceed.
- **The invoice being corrected is corrected by another session between this screen opening and Submit** -- Rejected-with-refresh, consistent with the Invoice entity's Contention note: the submission is refused with "This invoice was already corrected. View the existing credit note." and the reference block updates to the current state.
- **Network failure during submission** -- Error banner: "Couldn't submit this invoice. Check your connection and try again." with Retry; entered data is preserved.
- **Nadia opens this screen offline** -- The form is viewable and editable, but Submit is disabled with the offline banner until connectivity returns, per the Brief's Side-Effect Inventory offline behavior for this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Invoice List) | Navigation (inbound) | "New Invoice" arrives here in ad-hoc mode |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound) | "Issue Credit Note" arrives here in credit-note mode |
| FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Triggers (outbound) | Submission hands off to this automation for validation and recording |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Validation rules applied to form fields |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs why a credit note is a new record, never an edit |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_invoice_form_opened | mode (ad_hoc / credit_note) | Screen opens | N/A -- no success-metrics.md metric tracks form opens directly; retained as the funnel entry for the manual_invoice_issued signal below |
| manual_invoice_submitted | mode, outcome (accepted / rejected) | Submit completes | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (a rejected ad-hoc submission -- e.g., a credit note exceeding the original total -- is the manual-path counterpart to the accuracy this metric measures on the automatic path) |
| manual_invoice_form_abandoned | mode, fields filled | User discards unsaved changes | N/A -- no connected metric measures abandonment; retained for product visibility into a form that goes unfinished |

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Nadia taps "New Invoice" from a project's invoice list, when the form opens, then it is in ad-hoc mode with an empty description and amount, and a due date pre-filled from her default payment terms.

**FEAT-09.SPEC-003-AC-02:** Given Nadia taps "Issue Credit Note" from an invoice's detail screen, when the form opens, then it is in credit-note mode with that invoice's number, total, and issue date shown read-only.

**FEAT-09.SPEC-003-AC-03:** Given Nadia enters a description and a positive amount and taps Submit in ad-hoc mode, when submission succeeds, then she lands on FEAT-09.SPEC-002 for the newly recorded invoice.

**FEAT-09.SPEC-003-AC-04:** Given Nadia leaves the amount field empty and taps Submit, when validation runs, then the amount field shows "Enter an amount greater than zero." and submission does not proceed.

**FEAT-09.SPEC-003-AC-05:** Given Nadia enters a credit amount greater than the original invoice's total, when she taps Submit, then she sees "A credit note cannot exceed the original invoice's total." and submission does not proceed.

**FEAT-09.SPEC-003-AC-06:** Given Nadia adjusts the pre-filled due date on an ad-hoc invoice, when she submits, then the invoice is recorded with her chosen due date, not the default.

**FEAT-09.SPEC-003-AC-07:** Given Nadia navigates away with unsaved input, when she taps the back control, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-09.SPEC-003-AC-08:** Given Nadia taps Submit twice in rapid succession, when the first submission is still in flight, then the second tap has no effect.

**FEAT-09.SPEC-003-AC-09:** Given the invoice Nadia is correcting was already corrected in another session, when she taps Submit, then she sees "This invoice was already corrected. View the existing credit note." and no second credit note is recorded.

**FEAT-09.SPEC-003-AC-10:** Given a network failure occurs during submission, when the failure is detected, then Nadia sees "Couldn't submit this invoice. Check your connection and try again." with her entered data preserved.

**FEAT-09.SPEC-003-AC-11:** Given Nadia opens this screen while offline, when she attempts to submit, then Submit is disabled and a "reconnect to submit" banner is shown; the action never appears to succeed.

**FEAT-09.SPEC-003-AC-12:** Given Owen attempts to reach this screen directly, when the link resolves, then he is redirected to his portal home with no form ever rendered.

**FEAT-09.SPEC-003-AC-13:** Given Dana is inside a logged support session, when she looks for an issuance entry point, then none exists anywhere in her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (loading, empty, filling, submitting, validation error, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

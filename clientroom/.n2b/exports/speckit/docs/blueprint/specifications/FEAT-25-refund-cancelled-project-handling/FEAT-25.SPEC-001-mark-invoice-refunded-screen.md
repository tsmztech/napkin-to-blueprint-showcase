---
document_type: spec
spec_type: screen
spec_id: FEAT-25.SPEC-001
spec_name: Mark Invoice Refunded Screen
spec_slug: mark-invoice-refunded-screen
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Screen Spec: Mark Invoice Refunded Screen

## Overview

**Name:** Mark Invoice Refunded Screen
**ID:** FEAT-25.SPEC-001
**Type:** Screen
**Purpose:** Nadia marks a paid invoice Refunded or Partially refunded, entering the refunded amount and an optional reason, from the invoice detail view.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Entering a refund against a single refund-eligible invoice: full amount or a partial amount, plus an optional freelancer-entered reason. Refund-eligible means exactly one of two combinations (FEAT-25.SPEC-006, Refund eligibility rule): Invoice `status` Paid with its Payment in Succeeded status (paid through the platform), or Invoice `status` Paid (recorded by freelancer) with its Payment in Recorded manually status (paid off-platform and recorded by Nadia via FEAT-10.SPEC-005). Recording a refund against an off-platform payment is the case the feature's "records that a refund was issued outside the platform" capability fits most directly.
- Showing the invoice's amount paid so Nadia can see the ceiling her refund amount cannot cross
- Submitting the refund for validation and recording (FEAT-25.SPEC-003)
- Reflecting the invoice's resulting status (Refunded, Partially refunded, or Disputed if a reversal was recorded in the meantime) once the invoice detail screen (FEAT-09.SPEC-002) is reopened

**Non-Goals:**
- Issuing the refund itself through a payment-processing capability -- excluded per scope-boundaries.md (SC-18): the platform never holds or moves funds, so this screen only records a refund Nadia has already issued through her own processor account; it initiates no money movement
- Marking a project cancelled -- handled by FEAT-25.SPEC-002 (Mark Project Cancelled Screen); this screen acts on a single invoice only
- Recording a second refund against an invoice already Refunded or Partially refunded -- this feature treats a refunded invoice's status as terminal for this action; a further correction is a new, logged event outside this screen's scope (FEAT-25.SPEC-006, XBR-20), not a repeat submission here
- Validating the refund amount or authorizing who may act -- both governed entirely by FEAT-25.SPEC-006 (Authorization and Validation Rules); this screen only surfaces the outcome
- Applying the refund to the Invoice and Payment records -- owned entirely by FEAT-25.SPEC-003 (Refund & Partial Refund Recording); this screen only submits the entry and displays the result

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia taps "Record a refund" on an invoice whose `status` is Paid (Payment Succeeded) or Paid (recorded by freelancer) (Payment Recorded manually); the control is offered for no other invoice status | Invoice reference, invoice's amount paid and currency |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Nadia opens the invoice from her reversal notification; the invoice is Disputed by then, so no "Record a refund" entry is offered and this screen is not reached | Invoice reference -- see Edge Cases: a Disputed invoice is never eligible for this screen's action |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Enter and submit a full or partial refund with an optional reason | -- |
| Owen (Client Primary Contact) | No | No | Not reachable from Owen's portal navigation; he sees the resulting Refunded/Partially refunded status on his own invoice through FEAT-09's client-facing invoice view (Own-only, view), never this entry screen |
| Priya (Client Reviewer Contact) | No | No | Not reachable; Priya has no billing visibility at all (per the Access Matrix, Invoicing & Payments is None for Reviewer contacts) |
| Dana (Support Operator) | No | No | Not reachable -- Dana never reaches this screen. Inside a logged support session (FEAT-31) she sees only the resulting Refunded/Partially refunded/Disputed status, read-only, on the invoice detail screen (FEAT-09.SPEC-002), where the "Record a refund" control is not rendered in her session; a direct link to this screen opens FEAT-09.SPEC-002 in her read-only session with no error message |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on the invoice she was trying to refund if the link carried the reference, otherwise on FEAT-09.SPEC-001 (Invoice List) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress amount or reason entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Record a Refund" with a back arrow (returns to FEAT-09.SPEC-002, Invoice Detail) and a "Mark Refunded" action button (right-aligned, disabled until the form is valid).

**Body, in order:**
- **Invoice summary (read-only):** Invoice number, client name, amount paid, and currency -- the same values shown on FEAT-09.SPEC-002, so Nadia never has to hold the paid amount in her head while entering a refund.
- **Refund type:** A two-option choice -- "Full amount" (the default) and "Partial amount."
- **Amount:** A currency-formatted number input, in the invoice's currency. Pre-filled with the full amount paid and disabled (read-only display) when "Full amount" is selected; empty and editable, required, when "Partial amount" is selected.
- **Reason (optional):** A multi-line text input for a freelancer-entered note about why the refund was issued.

**Footer:** None -- "Mark Refunded" is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the header action remains reachable at the top of the screen.
- **Medium size class and above:** Same single-column form, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-09.SPEC-002 (Invoice Detail) | Screen closes | Standard navigation transition |
| "Full amount" option | Select | Sets amount to the full paid total; disables direct editing of the amount field | Amount field shows the full paid amount, read-only | Field visibly locks with the full amount displayed |
| "Partial amount" option | Select | Clears the amount field and makes it editable | Amount field becomes empty and editable | Field visibly unlocks; focus moves to it |
| Amount input | Type (Partial amount only) | Captures the entered value | Field shows entered value | Standard input state |
| Amount input | Blur | Triggers validation via FEAT-25.SPEC-006 (amount required, greater than zero, not exceeding the amount paid) | Error state on field if invalid | Exact error message per FEAT-25.SPEC-006 below the field |
| Reason input | Type | Captures the optional note | Field shows entered text | Standard input state |
| "Mark Refunded" button | Tap | 1. Validate the form via FEAT-25.SPEC-006. 2. If valid, trigger FEAT-25.SPEC-003 (Refund & Partial Refund Recording). | Button shows a loading state during submission | Success: toast "Invoice marked {Refunded / Partially refunded}" and navigate to FEAT-09.SPEC-002 showing the new status. Failure: inline error per the Error state below. |
| "Mark Refunded" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Refund type options -> Amount input (when editable) -> Reason input -> Mark Refunded.
- **Validation announcements:** When the amount field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Submission feedback:** The success toast is announced on completion; on validation failure, focus moves to the amount field if it is the failing field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for the invoice summary and form | Screen opens | The invoice's amount paid and currency finish loading |
| Loaded (default) | Form shown with "Full amount" pre-selected and the amount field showing the full paid total, read-only; Reason empty; "Mark Refunded" enabled | Screen opens on a refund-eligible invoice (Paid with a Succeeded Payment, or Paid (recorded by freelancer) with a Recorded manually Payment) | Nadia changes the refund type, edits the reason, or taps Mark Refunded |
| Editing (partial) | Amount field empty and editable, Reason as entered | Nadia selects "Partial amount" | Nadia re-selects "Full amount", or submits |
| Validation Error | The amount field shows its error message below it; Mark Refunded remains enabled to allow retry | Amount validation fails on blur or submit | Nadia corrects the amount and it re-validates |
| Submitting | "Mark Refunded" button shows a loading spinner; all inputs disabled | Nadia taps Mark Refunded with a valid form | FEAT-25.SPEC-003 completes or fails |
| Error | Error banner at the top of the form: "Couldn't record this refund. Try again." with a Retry button; entered amount and reason are preserved | FEAT-25.SPEC-003 reports a failure | Nadia taps Retry and the submission succeeds |
| Offline/Degraded | Banner "You're offline -- this refund can't be recorded until you reconnect." at the top; the form remains editable but Mark Refunded is disabled | Connectivity lost while this screen is open | Connectivity restored -- Mark Refunded re-enables; nothing is queued, since a refund status change must persist immediately as part of the record (assumptions-constraints.md ASMP-27) |

## Validation Rules

Validation governed by FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules). See that spec for the refund-amount ceiling, the no-partial-payment corollary, and the refunded-cannot-be-repaid-without-correction rule. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-09.SPEC-002 (Invoice Detail) | FEAT-09 |
| Successful "Mark Refunded" submission | FEAT-09.SPEC-002 (Invoice Detail), showing the new status | FEAT-09 |
| Retry after Error state succeeds | FEAT-09.SPEC-002 (Invoice Detail), showing the new status | FEAT-09 |

## Data Model

**Creates:** None -- this screen creates no new record; the refund is applied to the existing Invoice and Payment by FEAT-25.SPEC-003.
**Reads:** Invoice -- invoice number, currency, `status` (must be Paid, or Paid (recorded by freelancer), to reach this screen). Payment -- `status` (must be Succeeded for a Paid invoice, or Recorded manually for a Paid (recorded by freelancer) invoice) and amount (the ceiling shown and validated against).
**Updates:** None directly -- submission hands the entered amount and reason to FEAT-25.SPEC-003, which performs the actual Invoice and Payment updates.
**Deletes:** None.

## Business Rules

- Only a refund-eligible invoice reaches this screen: Invoice `status` Paid with a Payment in Succeeded status, or Invoice `status` Paid (recorded by freelancer) with a Payment in Recorded manually status (FEAT-25.SPEC-006, Refund eligibility rule). FEAT-09.SPEC-002 shows the entry point on those two combinations only; an invoice that is Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected offers no path here.
- The screen behaves identically for both eligible combinations; a manually recorded payment is refunded exactly like a platform-processed one, since the screen only records a refund Nadia already issued outside the platform.
- The amount field's ceiling is the invoice's amount paid, per FEAT-25.SPEC-006's refund-amount limit (XBR-20).
- Submission triggers FEAT-25.SPEC-003, which is the sole owner of the Invoice status and Payment record changes; this screen never writes those fields itself.
- A recorded refund cannot be undone from this screen -- reversing course requires a new, logged correction outside this feature's scope (XBR-20), consistent with the product's record-immutability constraint (XBR-04).

## Edge Cases

- **Nadia navigates away with an entered amount or reason unsaved** -- No confirmation dialog is shown; unlike a multi-field creation form, a partially entered refund carries no risk of an accidental duplicate record, since nothing is written until Mark Refunded succeeds. The form simply discards the entry.
- **Nadia taps Mark Refunded twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **The invoice was refunded, partially refunded, or reported Disputed by a reversal in another session while this screen was open** -- Submission is rejected with "This invoice's status changed since you opened this page. Refresh to see the latest state." and a "Refresh" action reloads the invoice's current status; this is the concurrent-edit conflict behavior for this screen's update to the shared Invoice and Payment entities, per the dependency map's Contention notes (reject-with-refresh) and FEAT-25.SPEC-006.
- **A reversal notice (FEAT-25.SPEC-005) is applied to this exact invoice at the same moment Nadia submits a refund** -- Processor-confirmed reversal status is authoritative over a concurrent manual refund entry (dependency map, Invoice Contention: "processor-confirmed payment status is authoritative over a concurrent manual entry"); the refund submission is rejected with the same refresh message above, and the invoice reopens showing Disputed.
- **Network failure during submission** -- The Error state's banner and Retry button appear; the entered amount and reason are preserved, and the invoice's prior confirmed status (Paid, or Paid (recorded by freelancer)) remains authoritative until the retry succeeds.
- **Nadia enters an amount exactly equal to the amount paid while "Partial amount" is selected** -- Validation passes (the boundary is inclusive), and FEAT-25.SPEC-003 records it as a full refund (Refunded), not Partially refunded, since the resulting status reflects the amount, not which radio option was used to reach it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound/outbound) | Entry point via "Record a refund"; destination after submission or cancellation |
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Triggers (outbound) | Mark Refunded submits the entry for validation and recording |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Validation and authorization rules applied to the amount field and to who may reach this screen |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | References (inbound) | A reversal applied to this invoice takes precedence over a concurrent refund submission |
| FEAT-31 (Operator Support Access) | References (inbound) | Owns Dana's read-only support session, in which she sees the resulting invoice status on FEAT-09.SPEC-002; no navigation from FEAT-31 into this screen exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| refund_screen_opened | refund type available (full only / full and partial), payment origin (platform / recorded manually) | Screen loads on a refund-eligible invoice | N/A -- no success-metrics.md metric is connected to this feature; retained so refund entry is observable rather than invisible |
| refund_submitted | refund type (full / partial), amount, currency | Nadia taps Mark Refunded with a valid form | N/A -- no success-metrics.md metric is connected to this feature; the outcome is measured downstream by FEAT-25.SPEC-003's own signals |
| refund_submission_rejected | reason (validation_error / stale_state / network_failure) | Submission fails for any reason | N/A -- no success-metrics.md metric is connected to this feature; retained so refund friction is observable rather than silent |

## Acceptance Criteria

**FEAT-25.SPEC-001-AC-01:** Given Nadia is on the Invoice Detail screen (FEAT-09.SPEC-002) for an invoice with status Paid, when she taps "Record a refund", then she lands on this screen with "Full amount" pre-selected and the amount field showing the full paid total, read-only.

**FEAT-25.SPEC-001-AC-02:** Given Nadia is on this screen with "Full amount" selected, when she taps "Mark Refunded", then FEAT-25.SPEC-003 records the invoice as Refunded and she sees a toast "Invoice marked Refunded" before returning to FEAT-09.SPEC-002.

**FEAT-25.SPEC-001-AC-03:** Given Nadia selects "Partial amount", when the amount field becomes editable, then it starts empty and focus moves to it.

**FEAT-25.SPEC-001-AC-04:** Given Nadia enters an amount greater than the invoice's amount paid, when she blurs the field, then the field shows the exact error defined by FEAT-25.SPEC-006 and Mark Refunded does not proceed.

**FEAT-25.SPEC-001-AC-05:** Given Nadia enters a partial amount equal to the full amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Refunded, not Partially refunded.

**FEAT-25.SPEC-001-AC-06:** Given Nadia enters a partial amount less than the amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Partially refunded and she sees the toast "Invoice marked Partially refunded."

**FEAT-25.SPEC-001-AC-07:** Given Nadia enters an optional reason, when she submits, then the reason is recorded alongside the refund by FEAT-25.SPEC-003.

**FEAT-25.SPEC-001-AC-08:** Given Nadia leaves the reason field empty, when she submits, then the refund is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-001-AC-09:** Given Nadia taps Mark Refunded and the submission fails due to a network error, then the Error banner "Couldn't record this refund. Try again." appears with her entered amount and reason preserved.

**FEAT-25.SPEC-001-AC-10:** Given Nadia taps Mark Refunded twice rapidly, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-001-AC-11:** Given the invoice was already marked Refunded, Partially refunded, or Disputed in another session since Nadia opened this screen, when she taps Mark Refunded, then the submission is rejected with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-001-AC-12:** Given a payment reversal is applied to this exact invoice by FEAT-25.SPEC-005 at the same moment Nadia submits a refund, then the processor-confirmed reversal wins, the refund submission is rejected with the refresh message, and the invoice reopens showing Disputed.

**FEAT-25.SPEC-001-AC-13:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this refund can't be recorded until you reconnect." appears and Mark Refunded is disabled.

**FEAT-25.SPEC-001-AC-14:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-001-AC-15:** Given Dana (Support Operator) is in a logged support session viewing an invoice that Nadia has marked Refunded, Partially refunded, or Disputed, when she opens the invoice detail screen (FEAT-09.SPEC-002), then she sees the resulting status read-only, no "Record a refund" control is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-09.SPEC-002 in her read-only session with no error message.

**FEAT-25.SPEC-001-AC-16:** Given Nadia's session expires while she has an amount and reason entered, when she re-authenticates, then the entered amount and reason are restored on this screen.

**FEAT-25.SPEC-001-AC-17:** Given Nadia taps "Record a refund" on an invoice, when this screen opens, then a skeleton layout appears for the invoice summary and form until the amount paid and currency finish loading.

**FEAT-25.SPEC-001-AC-18:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia opens it on FEAT-09.SPEC-002, then "Record a refund" is offered, and when she submits a valid full refund, then FEAT-25.SPEC-003 records the invoice as Refunded and she sees the toast "Invoice marked Refunded."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 7 (loading, loaded, editing, validation error, submitting, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

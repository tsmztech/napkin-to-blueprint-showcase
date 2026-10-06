---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-001
spec_name: Pay Invoice Screen
spec_slug: pay-invoice-screen
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Pay Invoice Screen

## Overview

**Name:** Pay Invoice Screen
**ID:** FEAT-10.SPEC-001
**Type:** Screen
**Purpose:** Owen views a sent invoice and pays it by card or bank transfer, sees its pending, paid, or declined status, and retries a failed payment immediately.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- Displaying a sent invoice's amount, due date, and current payment status to Owen
- Collecting Owen's chosen payment method (card or bank transfer) and initiating payment
- Showing the current or most recent payment attempt's status (Pending, Failed, Succeeded) and an immediate retry path after a decline
- Showing the "online payment is temporarily unavailable" state when the freelancer's payment account is not ready (XBR-19)

**Non-Goals:**
- Recording a payment made outside the portal -- owned by FEAT-10.SPEC-002 (Record Off-Platform Payment Screen), Nadia's own screen; Owen never records a payment on her behalf.
- Submitting the payment to the payment-processing capability and receiving its outcome -- owned by FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing); this screen only initiates the request and displays the result.
- Applying the processor's confirmation to the Invoice and Payment records -- owned by FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync); this screen renders whatever status that automation has already applied.
- Partial payment or an amount other than the full invoice total -- excluded per scope-boundaries.md (SC-17): an invoice is either unpaid or paid in full; this screen offers no partial-amount entry.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09 (Invoice Generation & Sending) -- invoice email link | Owen opens the invoice email and follows the pay link | Invoice reference; magic-link sign-in context (FEAT-05) if not already signed in |
| FEAT-11.SPEC-004 (Overdue Reminder Email, FEAT-11 Automated Payment Reminders) -- reminder email link | Owen follows a day-3 or day-10 reminder's link | Invoice reference |
| FEAT-05 (Client Portal Access) -- portal home | Owen opens an invoice waiting on him from his portal's invoice list | Invoice reference |
| FEAT-10.SPEC-002 (Record Off-Platform Payment Screen), via FEAT-10.SPEC-005 | Owen reopens or refreshes this screen after Nadia records an off-platform payment on the same invoice | Invoice reference; the invoice now reflects "Paid (recorded by freelancer)" |
| FEAT-10.SPEC-007 (Payment Confirmation Notification) | Owen taps the confirmation email's "View invoice" CTA | Invoice reference; the invoice shows its current payment status |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen, scoped to his own company's invoice (Own-only) | Selects a payment method, pays, retries a failed payment, views current status | -- |
| Priya (Client Reviewer Contact) | No -- Invoicing & Payments is None for Reviewer contacts (Access Matrix) | No | The invoice link and any direct navigation to this screen shows: "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." She is offered a link back to her own portal home (FEAT-05). |
| Nadia (Freelancer) | No -- this is a client-portal screen (FEAT-05); Nadia's Full access to Invoicing & Payments is exercised through her own project view (FEAT-09), not this screen (XBR-09 isolation) | No | -- (not a denial -- she simply uses a different, freelancer-side view of the same invoice) |
| Dana (Support Operator) | No -- Dana's View-only access to Invoicing & Payments is exercised entirely inside her own logged support session interface (FEAT-31), never this client-portal screen | No | She has no path to open this exact screen; her equivalent read of payment status is presented within FEAT-31's session view, which never shows a Pay control |
| Unauthenticated | No | No | Redirected to FEAT-05's sign-in request screen with the message: "Sign in to view and pay this invoice." No invoice content renders before sign-in. |
| Expired session/link | No | No | Shows the expired-link explanation defined by FEAT-05 (XBR-28): "This link has expired. Request a fresh one to continue." with a one-tap way to request a new link; any in-progress payment method selection is discarded, since a session cannot be assumed safe to resume. |

## Layout and Content

**Header:** Invoice identifying strip showing the invoice number, the project and client names, the issue date, the due date, and a status badge (Sent, Payment pending, Paid, Paid (recorded by freelancer), Overdue, or Failed -- the last reflecting the most recent unsuccessful attempt rather than a distinct Invoice status).

**Body, above the fold:** Invoice total (amount, tax line, and total) in the invoice's set currency (FEAT-15). Below it, the Payment Panel:
- Payment method selector: two options, "Card" and "Bank transfer," shown only for the methods currently available on this invoice's pay link (`available_payment_methods`, derived from Payment Account Connection, FEAT-32.SPEC-002); a method not currently available is not shown as a disabled option -- it is simply absent.
- "Pay {total} {currency}" button, directly below the selector.
- Current attempt status line, shown when a payment attempt exists for this invoice: "Payment pending -- waiting on your bank transfer to confirm" (Pending), "Payment declined: {failure_reason}" with a "Try again" button in the same place as the original Pay button (Failed), or "Paid on {paid_at_formatted}" (Succeeded).

**Body, below the fold:** Invoice line items and tax breakdown (read-only, sourced from FEAT-09). A "Download a copy" link (owned by FEAT-09) for a printable copy of the invoice.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint (phone):** Single-column, full-width layout as described above; the Payment Panel remains directly below the invoice total, since Owen typically opens this screen from a phone link (user-persona.md, Behavioral Context).
- **Medium size class and above:** The invoice header, total, and Payment Panel form a single card capped at a consistent platform-wide content width and horizontally centered; line items and tax breakdown render in a two-column table rather than stacked rows. No structural change to the Payment Panel itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Payment method selector | Tap "Card" or "Bank transfer" | Sets the chosen method for the pending Pay action | Selected option highlighted | Selected method shown as chosen |
| Pay button | Tap | 1. Validate the invoice is still payable and the payment account is ready, per FEAT-10.SPEC-006. 2. If valid, submit the payment via FEAT-10.SPEC-003 with the chosen method. | Button enters a processing state; method selector disabled | "Processing your payment..." shown; on card success the status line updates to "Paid on {paid_at_formatted}" once FEAT-10.SPEC-004 confirms; on bank transfer the status line updates to "Payment pending" |
| Pay button (while processing) | Tap | No action -- ignored while a submission is already in flight | None | Button remains in its processing state |
| "Try again" button (shown after a decline) | Tap | Re-submits the payment via FEAT-10.SPEC-003 with a freshly chosen or the same method | Button enters processing state, same as Pay | Same feedback pattern as the original Pay attempt |
| "Download a copy" link | Tap | Navigates to the printable invoice copy owned by FEAT-09 | Screen unaffected (opens the copy in place or as a new view) | Printable copy renders |
| Status badge / status line | None (display-only) | -- | Updates automatically when FEAT-10.SPEC-004 applies a new status while this screen is open | Badge and status line reflect the latest applied status without requiring a manual refresh |

### Accessibility Notes

- **Focus order:** Invoice header -> invoice total -> payment method selector -> Pay button -> current attempt status line -> line items -> Download a copy link.
- **Dynamic announcements:** When the Pay button enters its processing state, "Processing your payment" is announced to assistive technology. When a status change is applied (Paid, Payment pending, Failed), the updated status line is announced as it changes, and on a decline, focus moves to the "Try again" button.
- **Colour independence:** The status badge and status line always carry text ("Paid," "Payment pending," "Payment declined") alongside any colour treatment -- colour is never the sole distinguishing signal, per the feature's stated Accessibility expectation.
- **Keyboard alternatives:** Every action on this screen (method selection, Pay, Try again, Download) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for the invoice header, total, and Payment Panel | Screen first opens | Invoice and current payment status finish loading |
| Payable (default) | Payment method selector and Pay button active for the available methods | Invoice status is Sent or Overdue and no attempt is currently Pending or Succeeded | Owen taps Pay, or the invoice's status changes |
| Pending | "Payment pending" status line replaces the Pay button; no Pay control shown | A bank-transfer payment attempt is submitted and awaiting processor confirmation | The processor confirms (Paid) or reports the transfer failed (returns to Payable with a notice) |
| Paid | "Paid on {paid_at_formatted}" status line; no Pay control shown | Invoice status is Paid, by any path (card, confirmed bank transfer, or Nadia's manual record) | Never exits -- Paid is a terminal state for this screen |
| Declined | Payment method selector re-enabled; "Payment declined: {failure_reason}" status line with a "Try again" button in place of Pay | The payment-processing capability reports a card payment failed (FEAT-10.SPEC-003 / FEAT-10.SPEC-004) | Owen retries successfully (moves to Pending or Paid), or navigates away |
| Payment account unavailable | Payment method selector and Pay button are replaced entirely with: "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." | Payment Account Connection status is Needs attention or Disconnected at the moment this screen loads or refreshes (XBR-19) | Payment Account Connection returns to Connected and the screen is reloaded or refreshed |
| Error | Error banner above the Payment Panel: "We couldn't process that. Check your connection and try again." Payment Panel remains usable. | The payment submission fails for a reason other than a processor decline (e.g., the request could not be sent) | Owen dismisses the banner and retries, or navigates away |
| Offline/Degraded | Banner "You're offline. Reconnect to pay this invoice." above the Payment Panel; the Pay button is disabled while offline; invoice content remains fully visible | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity returns -- banner clears and the Pay button re-enables; no payment attempt is queued for automatic submission, since a payment must never appear to succeed without a live connection to the payment-processing capability |

## Validation Rules

Validation governed by FEAT-10.SPEC-006 (Payment Authorization & Validation Rules). See that spec for the Own-only pay gate, the payment-account-readiness gate, and the reject-with-refresh behavior on an already-resolved invoice.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Download a copy" tap | Printable invoice copy | FEAT-09 (Invoice Generation & Sending) |
| Portal navigation (back to home) | Portal home | FEAT-05 (Client Portal Access) |
| Successful payment confirmed | This same screen, now in the Paid state | -- |

## Data Model

**Creates:** None directly -- this screen initiates a payment request (FEAT-10.SPEC-003 creates the Payment record).
**Reads:** Invoice -- `invoice_number`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, `issue_date`, `due_date`, `status`, `pay_link availability` (from FEAT-09.SPEC-009, derived from Payment Account Connection). Payment -- the current or most recent attempt's `method`, `status`, `paid_at` for this invoice. Payment Account Connection -- `status`, `available_payment_methods` (read via FEAT-32.SPEC-002).
**Updates:** None directly -- this screen triggers FEAT-10.SPEC-003, which creates and updates the Payment record; FEAT-10.SPEC-004 applies the resulting Invoice status.
**Deletes:** None.

## Business Rules

- Only the Own-only Client Primary Contact for this invoice's client may pay or retry, per FEAT-10.SPEC-006's Authorization Rules.
- An invoice already Paid (by any path) cannot be paid again: a submission against a stale Payable view is refused and this screen refreshes to the current status (reject-with-refresh, FEAT-10.SPEC-006; dependency map, Entity: Payment, Contention).
- Payment is only offered when the Payment Account Connection status is Connected; a Needs attention or Disconnected status replaces the Payment Panel with the unavailable message (XBR-19).
- Partial payments are never accepted -- an invoice is either unpaid or paid in full (scope-boundaries.md SC-17); this screen collects no amount input.
- Once submitted, a payment attempt cannot be resubmitted while it is in flight -- the Pay button is disabled during processing rather than resubmittable (Shared UI Patterns, feature-overview.md).

## Edge Cases

- **Owen taps Pay twice in rapid succession** -- The second tap is ignored while the first submission is in the processing state; no second Payment record is created.
- **Another browser tab or device already paid this invoice while this screen was open (concurrent-edit conflict)** -- The stale Pay attempt is rejected by FEAT-10.SPEC-003/FEAT-10.SPEC-006 with the invoice's refreshed status: the Payment Panel is replaced by "This invoice was already paid on {paid_at_formatted}." Resolution: reject-with-refresh, per the dependency map's Contention note for the Payment entity -- the first confirmed full payment wins.
- **The Payment Account Connection changes from Connected to Needs attention while Owen is viewing this screen but before he taps Pay** -- The next attempt to submit is refused with the same "Online payment is temporarily unavailable" message; the screen also refreshes into the Payment account unavailable state on the connection status change (XBR-19), rather than allowing a payment to be attempted against a connection that will reject it.
- **Owen navigates away mid-processing and returns** -- The current attempt's status (Pending, Failed, or Paid) is re-fetched and shown correctly; no duplicate submission occurs on return, since the original attempt continues independently of this screen being open.
- **A bank transfer that was Pending is later reported failed** -- The screen returns to the Payable state with a notice above the Payment Panel: "Your bank transfer could not be completed. You can try again." (FEAT-10.SPEC-004).
- **Owen opens the pay link before the invoice has fully finished generating** -- Not applicable; per FEAT-09, invoice generation is near-instant and the pay link is only sent once the invoice is Sent, so this screen never loads against a Generated-but-unsent invoice.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | Triggers (outbound) | Pay and Try again submit the chosen method's payment request |
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | References (inbound) | Applies the processor's outcome, which this screen's status line and badge reflect |
| FEAT-10.SPEC-006 (Payment Authorization & Validation Rules) | References (inbound) | Own-only pay gate, payment-account-readiness gate, and reject-with-refresh behavior |
| FEAT-10.SPEC-005 (Record Off-Platform Payment) | References (inbound) | A manual record by Nadia updates the status this screen shows on next load |
| FEAT-09 (Invoice Generation & Sending) | Navigation (inbound/outbound) | Entry from the invoice email/view; Download a copy navigates back into FEAT-09 |
| FEAT-11 (Automated Payment Reminders) | Navigation (inbound) | Entry from a reminder email's link |
| FEAT-05 (Client Portal Access) | Navigation (inbound) | Entry from portal home; gates sign-in |
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | References (inbound) | Payment Account Connection status and available payment methods |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| payment_initiated | method (card / bank_transfer), invoice reference | Owen taps Pay or Try again and the request is submitted | supports success-metrics.md: "Time to Payment" |
| payment_screen_viewed | invoice status at load | This screen finishes loading | supports success-metrics.md: "Time to Payment" |
| payment_retry_attempted | method, prior failure reason | Owen taps Try again after a decline | supports success-metrics.md: "Time to Payment" |
| payment_account_unavailable_shown | -- | The Payment Panel is replaced by the unavailable message | N/A -- no Stage 2 metric measures payment-account readiness incidents from the client's side; retained so the frequency of this degraded state is observable rather than invisible |

## Acceptance Criteria

**FEAT-10.SPEC-001-AC-01:** Given Owen opens a Sent invoice's pay link, when the screen loads, then he sees the invoice total, due date, and an active Pay button for the currently available payment methods.

**FEAT-10.SPEC-001-AC-02:** Given Owen selects "Card" and taps Pay, when the payment succeeds, then the status line updates to "Paid on {paid_at_formatted}" and no Pay control remains.

**FEAT-10.SPEC-001-AC-03:** Given Owen selects "Bank transfer" and taps Pay, when the request is submitted, then the status line shows "Payment pending -- waiting on your bank transfer to confirm" and no Pay control remains until the transfer resolves.

**FEAT-10.SPEC-001-AC-04:** Given Owen's card payment is declined, when the decline is reported, then he sees "Payment declined: {failure_reason}" with a "Try again" button in place of Pay.

**FEAT-10.SPEC-001-AC-05:** Given Owen sees the declined state, when he taps "Try again" and the retried payment succeeds, then the status line updates to Paid, with no separate support step required.

**FEAT-10.SPEC-001-AC-06:** Given the freelancer's Payment Account Connection status is Needs attention, when Owen opens this invoice's pay link, then he sees "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." instead of a working form.

**FEAT-10.SPEC-001-AC-07:** Given the invoice was already paid in a different session, when Owen taps Pay against his stale view, then the payment is refused and the screen refreshes to show "This invoice was already paid on {paid_at_formatted}."

**FEAT-10.SPEC-001-AC-08:** Given Owen taps Pay, when he taps it again before the first submission resolves, then the second tap has no effect and only one Payment record is created.

**FEAT-10.SPEC-001-AC-09:** Given Owen loses connectivity while viewing this screen, when he attempts to tap Pay, then the button is disabled and the banner "You're offline. Reconnect to pay this invoice." is shown.

**FEAT-10.SPEC-001-AC-10:** Given Priya (Client Reviewer Contact) attempts to open this invoice's pay link, when the screen is requested, then she sees "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." instead of the invoice.

**FEAT-10.SPEC-001-AC-11:** Given an unauthenticated visitor opens a pay link, when the screen is requested, then they are redirected to sign-in with "Sign in to view and pay this invoice." and no invoice content is shown first.

**FEAT-10.SPEC-001-AC-12:** Given Owen's sign-in link has expired, when he opens it, then he sees "This link has expired. Request a fresh one to continue." with a one-tap way to request a new link.

**FEAT-10.SPEC-001-AC-13:** Given a bank transfer Owen submitted is later reported failed, when the failure is applied, then the screen returns to the Payable state with "Your bank transfer could not be completed. You can try again."

**FEAT-10.SPEC-001-AC-14:** Given Nadia records an off-platform payment on this invoice while Owen has this screen open, when Owen refreshes or reopens the screen, then it shows "Paid (recorded by freelancer)" with no Pay control.

**FEAT-10.SPEC-001-AC-15:** Given Owen is viewing this screen, when he taps "Download a copy," then he navigates to the printable invoice copy owned by FEAT-09.

**FEAT-10.SPEC-001-AC-16:** Given the status badge or status line updates while Owen has this screen open (for example a bank transfer confirming), when the update is applied, then the change is announced to assistive technology without requiring a manual refresh.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (loading, payable, pending, paid, declined, unavailable, error, offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

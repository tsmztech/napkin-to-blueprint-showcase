---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-002
spec_name: Invoice Detail
spec_slug: invoice-detail
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Invoice Detail

## Overview

**Name:** Invoice Detail
**ID:** FEAT-09.SPEC-002
**Type:** Screen
**Purpose:** Shows one invoice's amount, status, due date, reminder history, and pay-link/download controls, with what each side may do differentiated by role.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Displaying one invoice's full content: amount, currency, tax line, status, issue and due dates, business/billing details, triggering event, and reminder history
- The pay-link status banner (ready / not yet available / temporarily unavailable), sourced from FEAT-09.SPEC-009
- Downloading a printable copy of the invoice, credit note, or receipt
- The entry point into issuing a credit note against this invoice, once it is Sent or later
- Owen's pay-link action, and Nadia's entry point to recording an off-platform payment or a refund (both owned by other features)

**Non-Goals:**
- Generating or sending the invoice -- owned entirely by FEAT-09.SPEC-004 (automatic) and FEAT-09.SPEC-005 (manual); this screen only displays the result
- The actual payment flow behind the pay link -- owned by Invoice Payment Processing (FEAT-10); this screen only surfaces the link and its status
- Recording an off-platform payment or a refund -- owned by FEAT-10 and FEAT-25 respectively; this screen only provides the entry points from the freelancer's side
- Editing a sent invoice's content -- excluded per FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules): once Sent or later, no edit control is ever shown; a correction is always a new credit note (FEAT-09.SPEC-003) or a new invoice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-001 (Invoice List) | Nadia or Owen taps an invoice row | Invoice reference |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Owen taps the invoice email's pay-link CTA, or Nadia taps her copy confirmation's CTA | Invoice reference |
| FEAT-12 (Freelancer Financial Dashboard) | Nadia opens a specific invoice from a client drill-down | Invoice reference |
| FEAT-31 (Operator Support Access) | Dana opens an invoice inside a logged support session | Invoice reference, read-only session context |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | The recipient taps the email's "View invoice" CTA | Invoice reference for the affected invoice |
| FEAT-25.SPEC-008 (Payment Reversal Notification) | Nadia taps the email's "View invoice" CTA | Invoice reference for the disputed invoice |
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps an invoice-related trail entry or its affected-record link | Invoice reference |
| FEAT-28.SPEC-001 (Global Search) | Nadia selects an Invoice search result | Invoice reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full content of any of her invoices | Download a copy; issue a credit note (via FEAT-09.SPEC-003) once Sent or later; record an off-platform payment (FEAT-10) or a refund (FEAT-25) from this screen's entry points | -- |
| Owen (Client Primary Contact) | Full content of his own company's invoices, own-only | Follow the pay link (when ready); download a copy | -- |
| Priya (Client Reviewer Contact) | None -- per FEAT-09.SPEC-006, invoice content is hidden entirely from Reviewer contacts | None | A direct link to this screen redirects to her portal home with no error message and no partial content ever rendered |
| Dana (Support Operator) | Full content, read-only, inside a logged support session (FEAT-31) | View only | Every action control (download, pay, credit note, off-platform payment, refund) is not rendered in Dana's session; she sees content only |
| Unauthenticated | No | No | Redirected to sign-in (FEAT-05.SPEC-001 for a client contact) |
| Expired session | No | No | Client contact: FEAT-05.SPEC-002's expired-link explanation with a one-tap fresh-link request; Nadia: redirected to sign-in |

## Layout and Content

**Header:** Invoice number and status badge, with a back control returning to the entry point (Invoice List or dashboard).

**Body, in order:**
- **Summary card** (the same visual pattern as FEAT-09.SPEC-001's list row, at full density): triggering event label, amount/currency/total with the tax line broken out, issue date, due date.
- **Business and billing details:** Nadia's business name/address/tax ID and the client's billing name/address, as printed on the invoice.
- **Pay-link status banner:** one of the three states defined by FEAT-09.SPEC-009 (ready to pay, online payment not yet available, temporarily unavailable), with Owen's "Pay now" action shown only in the ready state.
- **Reminder history:** a chronological list of Reminder Log entries (day 3, day 10, manual, pause state), visible to Nadia only.
- **Download control:** available to Nadia and Owen for this invoice, or its credit note/receipt counterpart if one exists.
- **Correction area** (Nadia only, once status is Sent or later): "Correct with a credit note" action, replacing any would-be edit control per the shared "not yet edited after sending" convention.
- **Off-platform actions area** (Nadia only): "Record a payment received elsewhere" (FEAT-10) and, once paid, "Record a refund" (FEAT-25).

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order listed above; the pay-link banner and its action remain visible without scrolling past the summary card.
- **Medium size class and above:** Summary card and business/billing details sit side by side; the remaining sections continue to stack full-width beneath them.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to entry point | Screen closes | Standard transition |
| "Pay now" (Owen, ready state only) | Tap | Navigate to FEAT-10 (pay invoice) | Screen transitions | Standard transition |
| Download control | Tap | Produces a printable copy of the invoice, credit note, or receipt | None -- read-only action | A file is offered for save; a brief "Preparing your copy..." indicator shows while it generates |
| "Correct with a credit note" (Nadia, Sent+ only) | Tap | Navigate to FEAT-09.SPEC-003 with this invoice pre-selected for correction | Screen transitions | Standard transition |
| "Record a payment received elsewhere" (Nadia) | Tap | Navigate to FEAT-10's manual-payment entry | Screen transitions | Standard transition |
| "Record a refund" (Nadia, Paid invoices only) | Tap | Navigate to FEAT-25's refund entry | Screen transitions | Standard transition |
| Reminder history entries | Display only | None | None | Read-only list; no interaction |

### Accessibility Notes

- **Focus order:** Back control -> status badge -> summary card -> business/billing details -> pay-link banner (and its action, if present) -> reminder history -> download control -> correction/off-platform actions.
- **Status announcements:** A live pay-link status change (e.g., from ready to temporarily unavailable, per FEAT-09.SPEC-009) while this screen is open is announced to assistive technology.
- **Keyboard alternatives:** Every action is keyboard-reachable; the download control has no pointer-only gesture equivalent.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton in place of the summary card and sections | Screen first opens | Data finishes loading |
| Empty | N/A -- this screen is never opened without a resolved invoice reference; it has no zero-data state. Every entry point (FEAT-09.SPEC-001, FEAT-09.SPEC-010, FEAT-12, FEAT-31) carries a specific invoice reference as context, so a load always resolves to either Populated or Error, never to an invoice-less screen | Not applicable | Not applicable |
| Populated | Full content as described above | Load completes successfully | Screen closes or the invoice's live status changes (re-renders in place) |
| Download in progress | "Preparing your copy..." indicator on the download control | Download tapped | Copy is ready, or generation fails |
| Error | Error banner "Couldn't load this invoice. Check your connection and try again." with Retry | Initial load fails | Retry succeeds |
| Offline/Degraded | The last-loaded invoice detail remains visible under a "You're offline -- this may be out of date" banner; every action requiring connectivity (pay, download, credit note, off-platform recording) is disabled with a "Reconnect to continue" note | Connectivity is lost while viewing, or the screen is opened while offline | Connectivity returns and the screen re-fetches the current state |

## Validation Rules

Not applicable -- this screen has no user input beyond navigation and action taps; all validation for the actions it launches into lives in the destination specs (FEAT-09.SPEC-003, FEAT-10, FEAT-25).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Pay now" | FEAT-10 (pay invoice) | FEAT-10 |
| "Correct with a credit note" | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | -- |
| "Record a payment received elsewhere" | FEAT-10 (manual payment entry) | FEAT-10 |
| "Record a refund" | FEAT-25 (refund entry) | FEAT-25 |

## Data Model

**Creates:** None.
**Reads:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, `due_date`, `status`, `pay_link availability`. Reminder Log -- `reminder_type`, `scheduled_for`, `sent_at`, `pause_state` (Nadia only). Payment Account Connection -- status, read via FEAT-09.SPEC-009 to derive the pay-link banner.
**Updates:** None directly -- this screen never writes to the Invoice; downstream actions (pay, credit note, off-platform payment, refund) write through their own owning specs.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-006 governs who reaches this screen and what each role may do; this screen's Access and Visibility table is consistent with it.
- FEAT-09.SPEC-008 governs the "not yet edited after sending" convention: no edit control is ever shown once `status` is Sent or later; only "Correct with a credit note" is offered.
- FEAT-09.SPEC-009 governs the exact wording and behavior of the pay-link status banner; this screen renders that spec's output without deciding it independently.
- The reminder history section reads the Reminder Log (owned by FEAT-11) and is visible to Nadia only, since Owen has no entitlement to see the freelancer's follow-up cadence, per the Access Matrix.

## Edge Cases

- **The invoice's status changes (e.g., to Paid, or to Overdue) while this screen is open** -- No live conflict, since this screen performs no writes of its own: the displayed status re-renders to the new value in place, consistent with the read-only nature of this view; no reload or user action is required.
- **Owen opens this screen for an invoice with no connected payment account** -- The pay-link banner shows the "online payment not yet available" wording per FEAT-09.SPEC-009, and no "Pay now" action is rendered.
- **Nadia taps "Correct with a credit note" on an invoice that another session has already corrected** -- Rejected-with-refresh: the screen reloads to show `status: Corrected` and the existing credit note link, rather than opening a second correction flow against an invoice already corrected.
- **The download control is tapped while offline** -- The action is disabled with the "Reconnect to continue" note from the Offline/Degraded state; no partial or corrupted copy is ever produced.
- **Nadia or Owen attempts a manual issuance action or opens this detail page while offline** -- Per the Brief's Side-Effect Inventory: a clear "reconnect" state is shown; the action never appears to succeed, and the last-known invoice detail is shown as possibly out of date if cached, consistent with the Offline/Degraded state above.
- **Dana opens this screen inside a support session and the invoice is later corrected by Nadia in a separate session** -- Dana's read-only view re-renders to the corrected state on next refresh; she never sees a control to act on either version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Invoice List) | Navigation (inbound) | Row selection arrives here |
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Navigation (outbound) | "Correct with a credit note" opens the correction flow |
| FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) | References (inbound) | Governs the Access and Visibility table |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs the no-edit-after-sending convention |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Governs the pay-link banner's content |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Navigation (inbound) | Email CTAs arrive here |
| FEAT-10 (Invoice Payment Processing) | Navigation (outbound) | "Pay now" and "Record a payment received elsewhere" |
| FEAT-25 (Refund & Cancelled Project Handling) | Navigation (outbound) | "Record a refund" |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support-session entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invoice_detail_viewed | invoice reference, viewer role, invoice status at view time | Screen finishes loading | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (viewing is the moment either party could notice, and report, an incorrect amount, currency, or tax line) |
| invoice_copy_downloaded | invoice reference, document type (invoice / credit note / receipt), viewer role | Download completes successfully | N/A -- no success-metrics.md metric measures copy downloads; retained since product-features.md's Signals field for FEAT-09 names `invoice_copy_downloaded` explicitly as a required signal |
| invoice_correction_entry_opened | invoice reference | Nadia taps "Correct with a credit note" | N/A -- no connected metric measures correction-flow entry; retained as the upstream signal feeding FEAT-09.SPEC-005's correction-recording events |

## Acceptance Criteria

**FEAT-09.SPEC-002-AC-01:** Given Nadia opens an invoice she issued, when the screen loads, then she sees its amount, currency, tax line, status, issue date, due date, and reminder history.

**FEAT-09.SPEC-002-AC-02:** Given Owen opens one of his company's invoices, when the screen loads, then he sees its content but no reminder history section.

**FEAT-09.SPEC-002-AC-03:** Given Priya attempts to open a direct link to an invoice, when the link resolves, then she is redirected to her portal home with no invoice content ever rendered.

**FEAT-09.SPEC-002-AC-04:** Given the invoice's Payment Account Connection is not yet connected, when Owen views the invoice, then the pay-link banner shows "online payment not yet available" wording and no "Pay now" action appears.

**FEAT-09.SPEC-002-AC-05:** Given Owen views an invoice with a ready, connected payment account, when he taps "Pay now", then he navigates to FEAT-10's pay flow.

**FEAT-09.SPEC-002-AC-06:** Given the invoice's status is Sent, when Nadia views the screen, then no edit control is shown anywhere, only "Correct with a credit note."

**FEAT-09.SPEC-002-AC-07:** Given Nadia taps "Correct with a credit note", when the tap registers, then she navigates to FEAT-09.SPEC-003 with this invoice pre-selected for correction.

**FEAT-09.SPEC-002-AC-08:** Given Nadia or Owen taps Download on a Sent invoice, when the copy generates, then a printable file is offered and the invoice_copy_downloaded event is emitted.

**FEAT-09.SPEC-002-AC-09:** Given Dana opens this screen inside a support session, when she looks for any action control, then none is rendered -- only read-only content.

**FEAT-09.SPEC-002-AC-10:** Given the invoice's status changes to Paid while Nadia has this screen open, when the change lands, then the status badge updates in place with no reload required.

**FEAT-09.SPEC-002-AC-11:** Given two of Nadia's sessions both view an invoice already corrected in one of them, when the second session attempts "Correct with a credit note" on the now-stale view, then it is rejected with a refresh showing the existing correction rather than opening a second correction flow.

**FEAT-09.SPEC-002-AC-12:** Given Owen loses connectivity while viewing an invoice, when the loss occurs, then the last-loaded content remains visible under an offline banner and every connectivity-requiring action is disabled with "Reconnect to continue."

**FEAT-09.SPEC-002-AC-13:** Given the initial load fails, when the failure occurs, then Nadia sees "Couldn't load this invoice. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-002-AC-14:** Given Nadia views a Paid invoice, when she looks for a refund entry point, then "Record a refund" is shown and navigates to FEAT-25.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (loading, empty (N/A), populated, download in progress, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

# FEAT-10 — Invoice Payment Processing

This chapter covers Invoice Payment Processing, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 7 specifications carrying 106 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-10.SPEC-001 | Pay Invoice Screen | screen | 16 |
| FEAT-10.SPEC-002 | Record Off-Platform Payment Screen | screen | 13 |
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | integration | 15 |
| FEAT-10.SPEC-004 | Payment Confirmation & Invoice Status Sync | automation | 14 |
| FEAT-10.SPEC-005 | Record Off-Platform Payment | automation | 12 |
| FEAT-10.SPEC-006 | Payment Authorization & Validation Rules | logic-rule | 23 |
| FEAT-10.SPEC-007 | Payment Confirmation Notification | notification | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Invoice Payment Processing

## Summary

**Feature:** Invoice Payment Processing
**ID:** FEAT-10
**Description:** The client pays an invoice by card or bank transfer directly from the portal; the payment lands in the freelancer's own processor account, and the invoice's status updates immediately on confirmed payment.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Experience narrative: "they pay it by card on the spot" — instant, in-portal payment is central to the "get paid faster" value proposition. MVP phase: the product's payment promise depends on it. [RESEARCH-INFORMED: competitors that route payments through their own processor stack a percentage fee on top of the subscription (HoneyBook 2.7%+10¢ card and 1.5% bank; Bonsai about 3%) and Bonsai users report payouts delayed up to 10 business days (G2 and Trustpilot reviews, MEDIUM–HIGH) — here payments land directly in the freelancer's own connected account (FEAT-32) with no platform fee] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Pay by card or bank transfer — client completes payment from the invoice view
- Immediate status update — the invoice reflects Paid the moment payment is confirmed
- Failure handling — a failed or declined payment can be retried immediately
- Record an off-platform payment — Nadia marks an invoice paid when the client paid outside the portal (for example a direct bank transfer), with date and method; the entry is logged, never silent
- Pending bank transfers — a bank-transfer payment shows "Payment pending" until the processor confirms it

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-10.SPEC-001 | Pay Invoice Screen | Screen | Owen (Client Primary Contact) | Owen views a sent invoice and pays it by card or bank transfer, sees pending/paid/declined status, and retries a failed payment immediately |
| FEAT-10.SPEC-002 | Record Off-Platform Payment Screen | Screen | Nadia (Freelancer) | Nadia records that an invoice was paid outside the portal, entering date and method, so the payment is logged and never silent |
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | Integration | Owen (Client Primary Contact), Nadia (Freelancer) | Submits Owen's card or bank-transfer payment to the payment-processing capability and receives back its outcome — succeeded, failed/declined, or pending confirmation |
| FEAT-10.SPEC-004 | Payment Confirmation & Invoice Status Sync | Automation | Owen (Client Primary Contact), Nadia (Freelancer) | Applies the processor's reported outcome to the Payment and Invoice records the instant it arrives — Paid, Payment pending, Failed/Unpaid — and refuses a second attempt on an already-paid invoice |
| FEAT-10.SPEC-005 | Record Off-Platform Payment | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | Validates and persists Nadia's manual payment record — full invoice amount, not future-dated — and sets the invoice to "Paid (recorded by freelancer)" |
| FEAT-10.SPEC-006 | Payment Authorization & Validation Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Governs who may pay, view, or manually record a payment, the full-payment-only and manual-record limits, the reject-with-refresh concurrency behavior, and the payment-account-readiness gate |
| FEAT-10.SPEC-007 | Payment Confirmation Notification | Notification | Owen (Client Primary Contact), Nadia (Freelancer) | Sends a confirmation email to Owen and Nadia the moment a card payment or a confirmed bank transfer succeeds; a pending bank transfer sends no receipt until then |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Pay by card or bank transfer — client completes payment from the invoice view | FEAT-10.SPEC-001, FEAT-10.SPEC-003 | The screen collects the method and initiates payment; the integration spec submits it to the payment-processing capability | Phase 2 (Explicit) |
| Immediate status update — the invoice reflects Paid the moment payment is confirmed | FEAT-10.SPEC-004 | The automation applies the processor's confirmation to the Invoice record the instant it is reported, for both Owen and Nadia | Phase 2 (Explicit) |
| Failure handling — a failed or declined payment can be retried immediately | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-004 | The screen shows the decline reason and keeps the pay control available for an immediate retry; the automation keeps the invoice correctly Unpaid rather than a false Paid | Phase 2 (Explicit) |
| Record an off-platform payment — Nadia marks an invoice paid when the client paid outside the portal, with date and method; the entry is logged, never silent | FEAT-10.SPEC-002, FEAT-10.SPEC-005 | The screen collects date and method; the automation validates and persists the record and updates the invoice status | Phase 2 (Explicit) |
| Pending bank transfers — a bank-transfer payment shows "Payment pending" until the processor confirms it | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-004, FEAT-10.SPEC-007 | The screen displays the pending state; the integration spec carries the pending and later confirmed/failed events; the automation applies each transition; the notification holds its receipt until confirmation | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-28) names the payment-processing capability this feature relies on for money movement; per the standalone-spec decision rule, any trigger-response that crosses the product boundary to an external capability belongs to an Integration spec, never inline in a screen |
| FEAT-10.SPEC-006 | Payment Authorization & Validation Rules | Phase 5 (Rule-Constraint Discovery) | The Access, Validation & Limits, and Contention fields together produce 5+ interacting rules (Own-only pay gate, no-partial-payment rule, full-amount/not-future-dated manual-record limits, reject-with-refresh concurrency, processor-authoritative-over-manual rule, payment-account-readiness gate, Dana's view-only/no-payment-action constraint) shared across SPEC-001, SPEC-002, SPEC-004, and SPEC-005 — past the inline-validation threshold |
| FEAT-10.SPEC-007 | Payment Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names a confirmation email to Owen and Nadia with a defined audience, trigger, and a delivery-timing rule for pending bank transfers — this carries delivery rules and cannot stay an inline toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Payment**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-10.SPEC-003, FEAT-10.SPEC-005 | SPEC-003 creates a Payment record as the payment-processing capability reports the outcome of a card or bank-transfer attempt; SPEC-005 creates a Payment record from Nadia's manual entry, marked Recorded manually | -- |
| Read (single) | FEAT-10.SPEC-001 | Pay Invoice Screen shows the current or most recent payment attempt's status (Pending, Failed, Succeeded) for the invoice being viewed | -- |
| Read (list) | N/A | An invoice has at most one successful Payment (no partial payments, scope-boundaries.md SC-17), so this feature exposes no payment history list; historical Payment records are read by Financial Dashboard (FEAT-12) and Support Access (FEAT-31), not by a screen this feature owns | -- |
| Update | FEAT-10.SPEC-003 | Transitions Payment status Initiated → Pending → Succeeded/Failed as the payment-processing capability reports each stage | SPEC-005's manual record is written directly to its terminal status and is not subsequently updated by this feature |
| Delete/Archive | N/A | Payment is never deleted or archived in-product; it is removed only on account deletion (FEAT-24), subject to legal financial-record retention (feature-dependency-map.md, Entity: Payment). This feature defines no delete/archive operation — recorded as an explicit non-goal below | -- |
| State Transition | FEAT-10.SPEC-003 (Initiated → Pending → Succeeded/Failed), FEAT-10.SPEC-005 (→ Recorded manually) | -- | -- |

**Entity: Invoice**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Invoices are created by Invoice Generation & Sending (FEAT-09); this feature never creates one | -- |
| Read (single) | FEAT-10.SPEC-001, FEAT-10.SPEC-002 | SPEC-001 loads the invoice Owen is paying; SPEC-002 loads the invoice Nadia is marking paid | -- |
| Read (list) | N/A | Invoice list browsing belongs to the portal and dashboard views (FEAT-05, FEAT-09, FEAT-12), not to this feature | -- |
| Update | FEAT-10.SPEC-004, FEAT-10.SPEC-005 | SPEC-004 writes Payment pending, Paid, or a return to Unpaid (failed bank transfer); SPEC-005 writes "Paid (recorded by freelancer)" | Invoice `status` field (feature-dependency-map.md, Entity: Invoice) |
| Delete/Archive | N/A | This feature never deletes or archives an invoice; deletion is owned entirely by account deletion (FEAT-24), subject to legal financial-record retention (XBR-33) — recorded as an explicit non-goal below | -- |
| State Transition | FEAT-10.SPEC-004 (→ Payment pending, → Paid, Payment pending → Unpaid on a failed transfer), FEAT-10.SPEC-005 (→ Paid (recorded by freelancer)) | Sent/Overdue → Paid transitions this feature owns; Generated → Sent is owned by FEAT-09, Overdue flagging by FEAT-11, Refunded/Disputed/Corrected by FEAT-25/FEAT-09 | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Payment Account Connection | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-006 | Gates pay-link availability and the currently available payment methods; when the connection Needs attention, the pay screen shows "online payment is temporarily unavailable" instead of a working form (XBR-19) |
| Client Contact | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-006 | Establishes Owen's identity and enforces the Own-only role gate on paying; supplies the `recorded_by` identity context for Nadia's manual entries |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Owen submits a card or bank-transfer payment on a payable invoice | Submit the payment to the payment-processing capability and await its outcome | Standalone Integration | SPEC-003 |
| The payment-processing capability reports a card payment Succeeded | Mark Payment Succeeded and Invoice Paid immediately, visible to both Owen and Nadia | Standalone Automation | SPEC-004 |
| The payment-processing capability reports a card payment Failed/Declined | Mark Payment Failed, keep the invoice correctly Unpaid, and show Owen the failure reason with an immediate retry path | Inline in SPEC-001 (Error state), driven by SPEC-004 | SPEC-001 / SPEC-004 |
| Owen submits a bank-transfer payment | Mark Payment Pending and the invoice "Payment pending"; the reminder schedule pauses | Standalone Automation; reminder pause itself is cross-feature | SPEC-004 (pause owned by FEAT-11) |
| The payment-processing capability confirms a pending bank transfer | Mark Payment Succeeded and Invoice Paid; release the delayed confirmation receipt | Standalone Automation + Standalone Notification | SPEC-004 / SPEC-007 |
| The payment-processing capability reports a pending bank transfer failed | Return the invoice to Unpaid with a clear notice to both Owen and Nadia | Standalone Automation | SPEC-004 |
| A card payment or a confirmed bank transfer succeeds | Send a confirmation email to Owen and Nadia | Standalone Notification | SPEC-007 |
| Owen attempts to pay an invoice that is already Paid (a concurrent second attempt) | Refuse the payment and show Owen the current, refreshed status instead of processing it again | Inline in SPEC-001 (reject-with-refresh outcome), governed by SPEC-006 | SPEC-001 / SPEC-006 |
| Nadia records an off-platform payment | Validate the amount equals the full invoice total and the date is not in the future, then create the Payment record and set the invoice to "Paid (recorded by freelancer)" | Standalone Automation | SPEC-005 |
| Nadia attempts to record a manual payment on an invoice the processor has already marked Paid | Refuse the manual entry; processor-confirmed status is authoritative over a concurrent manual entry | Inline in SPEC-002 (reject-with-refresh outcome), governed by SPEC-006 | SPEC-002 / SPEC-006 |
| A payment succeeds, fails, or is manually recorded | Write an append-only Activity Log Entry (XBR-05) | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| An invoice reaches Paid, by any path | Stop the automated reminder schedule for that invoice | Cross-feature -- owned by Automated Payment Reminders (FEAT-11) | FEAT-11 responsibility |
| A payment succeeds or is manually recorded | Reflect the paid amount in dashboard and export totals | Cross-feature -- owned by Financial Dashboard (FEAT-12) | FEAT-12 responsibility |
| Owen opens the pay link while the freelancer's payment account Needs attention | Show "online payment is temporarily unavailable" instead of a working pay form | Inline in SPEC-001 (Permission/Degraded state), governed by SPEC-006 | SPEC-001 / SPEC-006 |
| Owen opens the pay screen while offline | Show a clear "reconnect to pay" message; the action never appears to succeed without connectivity | Inline in SPEC-001 (Offline/Degraded state) | SPEC-001 |
| A reversal or chargeback is reported later on a paid invoice | Mark the invoice Disputed alongside its preserved Paid record and notify the freelancer | Cross-feature -- owned by Refund & Cancelled Project Handling (FEAT-25) | FEAT-25 responsibility |

## Shared Context

**Shared Entities:**
- Invoice -- read by SPEC-001 and SPEC-002 for display; updated by SPEC-004 (Payment pending, Paid, back to Unpaid on a failed transfer) and SPEC-005 (Paid (recorded by freelancer)). Fields touched: `status`, and the read-only `pay_link availability` derived from Payment Account Connection (FEAT-32).
- Payment -- created by SPEC-003 (processor-confirmed) and SPEC-005 (manual); updated by SPEC-003 as the processor reports each stage; read by SPEC-001 for the current attempt's display. Fields: `invoice`, `amount`, `method`, `paid_at`, `status`, `recorded_by`.
- Payment Account Connection -- read by SPEC-001, SPEC-003, and SPEC-006 to gate pay-link availability, available methods, and the "temporarily unavailable" state; never written by this feature (owned by FEAT-32).
- Client Contact -- read by SPEC-001, SPEC-003, and SPEC-006 to establish Owen's identity and the Own-only role gate on paying.

**Shared UI Patterns:**
- Payment status display (Unpaid / Payment pending / Paid / Paid (recorded by freelancer) / Failed) -- shown consistently by SPEC-001 (Owen's view) and SPEC-002 (Nadia's view) so both sides of the same invoice never disagree, and never conveyed by colour alone, per the Accessibility expectation in the feature's States field.
- In-progress, not-a-second-submission pattern -- SPEC-001's Loading state and SPEC-004's status sync rely on the same principle: once a payment attempt is underway, the pay control is disabled rather than resubmittable, and a second attempt on an already-resolved invoice is refused with the refreshed status.

**Shared Validation:**
- SPEC-006 defines the authorization, full-payment-only/manual-record limits, and concurrency rules. SPEC-001, SPEC-002, SPEC-004, and SPEC-005 all reference SPEC-006 rather than restating role gates or the reject-with-refresh behavior.

## Internal Dependency Map

```
SPEC-001 (Pay Invoice Screen) -> [Owen submits a card or bank-transfer payment] -> SPEC-006 (Payment Authorization & Validation Rules) -> [pass] -> SPEC-003 (Card & Bank-Transfer Payment Processing)
SPEC-003 (Card & Bank-Transfer Payment Processing) -> [processor reports an outcome] -> SPEC-004 (Payment Confirmation & Invoice Status Sync)
SPEC-004 (Payment Confirmation & Invoice Status Sync) -> [invoice updated to Paid] -> SPEC-001 (Pay Invoice Screen shows Paid)
SPEC-004 (Payment Confirmation & Invoice Status Sync) -> [payment succeeds] -> SPEC-007 (Payment Confirmation Notification)
SPEC-004 (Payment Confirmation & Invoice Status Sync) -> [a pending bank transfer fails] -> SPEC-001 (Pay Invoice Screen returns to Unpaid with a notice)
SPEC-002 (Record Off-Platform Payment Screen) -> [Nadia submits a manual record] -> SPEC-006 (Payment Authorization & Validation Rules) -> [pass] -> SPEC-005 (Record Off-Platform Payment)
SPEC-005 (Record Off-Platform Payment) -> [invoice set to Paid (recorded by freelancer)] -> SPEC-001 (Pay Invoice Screen reflects the new status for Owen)
SPEC-001 (Pay Invoice Screen) -> [checks payment-account readiness and access using] -> SPEC-006 (Payment Authorization & Validation Rules)
SPEC-002 (Record Off-Platform Payment Screen) -> [checks access and manual-record limits using] -> SPEC-006 (Payment Authorization & Validation Rules)
```

**Default Entry:** SPEC-001 (Pay Invoice Screen) -- the screen Owen reaches from an invoice email link, a reminder email's link, or the portal's invoice view, once an invoice has been sent.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-10.SPEC-001 | Inbound | FEAT-09 (Invoice Generation & Sending) | Owen reaches the pay screen from the invoice email or invoice view FEAT-09 sends | An invoice is generated and sent |
| FEAT-10.SPEC-001 | Inbound | FEAT-11 (Automated Payment Reminders) | Owen reaches the pay screen from a reminder email's link | A reminder is sent for an overdue invoice |
| FEAT-10.SPEC-001 | Inbound | FEAT-05 (Client Portal Access (Magic-Link Login)) | Owen opens an invoice awaiting payment from his portal home | Owen navigates to an unpaid invoice waiting on him |
| FEAT-10.SPEC-002 | Inbound | FEAT-09 (Invoice Generation & Sending) | Nadia reaches the manual-record action from the invoice detail view on her side | Nadia marks an invoice paid elsewhere |
| FEAT-10.SPEC-003 | Outbound | FEAT-32 (Payment Account Connection) | A payment lands directly in Nadia's own connected payment account, and processing depends on the connection's readiness and available methods | A payment is attempted |
| FEAT-10.SPEC-001, FEAT-10.SPEC-006 | Inbound | FEAT-32 (Payment Account Connection) | When the connection Needs attention, the pay screen shows "online payment is temporarily unavailable" instead of a working form (XBR-19) | The freelancer's payment account status changes to Needs attention |
| FEAT-10.SPEC-004 | Outbound | FEAT-11 (Automated Payment Reminders) | Invoice reaching Paid stops the reminder schedule; Payment pending pauses it | Invoice reaches Paid, or Payment pending is set |
| FEAT-10.SPEC-004, FEAT-10.SPEC-005 | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Dashboard and export totals derive only from this feature's Payment and Invoice status truth (XBR-22) | A payment succeeds or is manually recorded |
| FEAT-10.SPEC-004, FEAT-10.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Confirmed payments, failures, manual records, and status changes each write an append-only trail entry (XBR-05) | A payment succeeds, fails, or is manually recorded |
| FEAT-10.SPEC-003 | Inbound | FEAT-25 (Refund & Cancelled Project Handling) | A reversal or chargeback reported later on a paid invoice is owned entirely by FEAT-25 (Disputed status, freelancer notice); this feature's Payment/Invoice record is preserved, never deleted (XBR-20, XBR-21) | The payment-processing capability reports a reversal or chargeback |
| FEAT-10.SPEC-007 | Outbound | FEAT-14 (Notifications & Delivery) | The confirmation email is sent through the transactional email delivery capability, whose Integration spec is owned by FEAT-14; this feature carries no Integration spec of its own for that capability | A payment succeeds |
| FEAT-10.SPEC-001, FEAT-10.SPEC-002 | Inbound | FEAT-31 (Support Access) | Dana views payment status read-only inside a logged support session that FEAT-31 owns; she never reaches this feature's own screens and cannot pay or record anything | Dana opens a support session |

## Non-Functional Notes

**Data volumes / growth:** Each invoice receives at most one successful Payment (no partial payments, scope-boundaries.md SC-17); volume scales with ordinary invoice volume across a freelancer's active clients (a few thousand freelancers in year one, each with 3–15 active clients, per scope-boundaries.md SC-21), so this feature carries no growth concern beyond that baseline. This feature emits `payment_initiated`, `payment_succeeded`, `payment_failed`, `payment_retried`, `payment_pending`, and `payment_recorded_manually` signals (product-features.md, Signals field); SPEC-003 and SPEC-004 fire the processor-driven signals, and SPEC-005 fires `payment_recorded_manually`.

**Responsiveness:** The pay screen is a client-facing page and must become interactive within roughly 2 seconds on a typical mobile connection (assumptions-constraints.md, ASMP-21); payment processing shows a clear in-progress state rather than allowing a second submission while the result is awaited (product-features.md, States field).

**Data sensitivity / privacy:** Payment records capture payer identity and financial activity as personal data, but the product never captures or stores card numbers or bank credentials — that handling belongs entirely to the payment-processing capability (assumptions-constraints.md, ASMP-24; scope-boundaries.md, SC-10). Both Payment and Invoice records are evidentiary once created and never silently altered afterward (assumptions-constraints.md, ASMP-25).

**Compliance flags:** Payer identity and invoice billing details are GDPR-class personal data (assumptions-constraints.md, ASMP-24); Payment and Invoice records may be subject to legal financial-record retention even after account deletion (feature-dependency-map.md, Entity: Invoice and Entity: Payment, Data Sensitivity). The pay flow must remain usable with a screen reader and keyboard, and never rely on colour alone to distinguish paid from unpaid (assumptions-constraints.md, ASMP-27; product-features.md, States field).

## Non-Goals

- **Partial payments or instalments on a single invoice** -- Excluded per scope-boundaries.md (SC-17): an invoice is either unpaid or paid in full; instalments are expressed as separate milestone or deposit invoices through the payment schedule (FEAT-04) instead.
- **Issuing refunds or fighting chargebacks inside Clientroom** -- Excluded per scope-boundaries.md (SC-18): refunds are issued by the freelancer through her own payment-processing account; Clientroom only records the resulting refund, partial refund, or reversal, and that recording is owned by FEAT-25, not this feature.
- **The platform holding or moving client funds, or storing card data** -- Excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): every payment goes directly into the freelancer's own connected account, and the product never touches card numbers or bank credentials.
- **Automatic purge of Payment or Invoice records** -- Intentional lifecycle decision surfaced by the CRUD matrix: both records are retained for the life of the freelancer's account with no automatic purge, removed only on account deletion (FEAT-24) subject to legal financial-record retention (XBR-33; feature-dependency-map.md, Entity: Invoice and Entity: Payment, Data Sensitivity).
- **Choosing, configuring, or connecting payment methods within this feature** -- Owned entirely by Payment Account Connection (FEAT-32): this feature only consumes the connection's readiness status and its currently available payment methods; it defines no connection or configuration screen of its own.



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



# Integration Spec: Card & Bank-Transfer Payment Processing

## Overview

**Name:** Card & Bank-Transfer Payment Processing
**ID:** FEAT-10.SPEC-003
**Type:** Integration
**Purpose:** Submits Owen's card or bank-transfer payment to the payment-processing capability and receives back its outcome -- succeeded, failed/declined, or pending confirmation -- so the invoice's status can be applied.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- Submitting a card or bank-transfer payment request for an open invoice to the payment-processing capability
- Receiving and defining the product's response to every outcome the capability reports: succeeded, failed/declined, pending, confirmed after pending, and a pending transfer that ultimately fails
- User-facing behavior on the Pay Invoice Screen when the capability is slow, unavailable, or rejects a request
- Disclosure to Owen about what invoice and identity data is shared with the capability

**Non-Goals:**
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md's Ecosystem & Integrations section names the category ("an established processor") without mandating a specific vendor.
- Connecting, reconnecting, or disconnecting the freelancer's own processor account, or reporting its readiness status -- owned entirely by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this spec only depends on that connection being ready and consumes its currently available payment methods.
- Applying the reported outcome to the Invoice and Payment records -- owned by FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync); this spec defines only the request/response contract with the capability.
- Recording a payment reversal or chargeback reported after an invoice is already Paid -- owned entirely by FEAT-25 (Refund & Cancelled Project Handling); this spec's contract ends once a payment first reaches Succeeded, Failed, or Pending.

## Capability Category

**Category:** Payment processing (into each freelancer's own account)
**Dependency Source:** ASMP-28 -- Dependencies entry in assumptions-constraints.md naming the payment-processing capability
**External Touchpoint:** "Payment processing into each freelancer's own account -- connection, card and bank-transfer payment, status, pending transfers, reversals" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32)
**Vendor Mandate:** None -- BRIEF.md, Ecosystem & Integrations names only the category ("an established processor takes card and bank-transfer payments directly into each freelancer's own account"); vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Owen pays an open invoice by card or bank transfer directly from the portal | Pay by card or bank transfer | FEAT-10.SPEC-001 (Pay Invoice Screen) |
| A bank-transfer payment shows "Payment pending" until the processor confirms it | Pending bank transfers | FEAT-10.SPEC-001, FEAT-10.SPEC-004 |
| A failed or declined payment shows a reason and can be retried immediately | Failure handling | FEAT-10.SPEC-001 |
| Payment lands directly in the freelancer's own connected account with no platform fee | The product's "get paid faster" value proposition (BRIEF.md, Experience narrative) | FEAT-32.SPEC-002 (readiness), FEAT-10.SPEC-004 (status applied) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Invoice amount and currency | Invoice -- `total`, `currency` | Owen submits a payment on the Pay Invoice Screen | The capability must know what to charge |
| Invoice reference | Invoice -- `invoice_number` | Owen submits a payment | Ties the reported outcome back to the correct invoice |
| Payment method chosen | Payment -- `method` (card / bank transfer) | Owen submits a payment | Routes the request to the correct payment rail |
| Payer identity | Client Contact -- `name`, `email` (Owen's) | Owen submits a payment | Attaches a payer identity to the transaction for the capability's own records and for Owen's payment confirmation |
| Freelancer's processor account reference | Payment Account Connection -- `processor_account_reference` | Owen submits a payment | Directs the funds into the correct freelancer's own connected account |

Card numbers, bank account or routing numbers, and every other Client Contact or Invoice field beyond those listed above never leave the product (BRIEF.md, Constraints; scope-boundaries.md SC-10).

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Payment outcome (Succeeded / Failed / Pending) | The capability reports the result of a submitted card or bank-transfer request | Payment -- `status` |
| Paid timestamp | A card payment succeeds, or a pending bank transfer is confirmed | Payment -- `paid_at` |
| Failure reason (plain-language category) | A card payment is declined, or a pending bank transfer ultimately fails | Consumed by FEAT-10.SPEC-004 for the Pay Invoice Screen's failure message; not persisted as a standalone field on Payment beyond what `status` = Failed already records |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Card payment succeeded | The capability confirms a submitted card payment | Payment `status` set to Succeeded, `paid_at` recorded | Owen sees "Paid on {paid_at_formatted}" on the Pay Invoice Screen | FEAT-10.SPEC-004 (applies the Invoice status), FEAT-10.SPEC-001 (displays it) |
| Card payment failed/declined | The capability declines a submitted card payment | Payment `status` set to Failed | Owen sees "Payment declined: {failure_reason}" with an immediate "Try again" option | FEAT-10.SPEC-004, FEAT-10.SPEC-001 |
| Bank transfer submitted (pending) | Owen submits a bank-transfer payment | Payment `status` set to Pending | Owen sees "Payment pending -- waiting on your bank transfer to confirm" | FEAT-10.SPEC-004, FEAT-10.SPEC-001 |
| Pending bank transfer confirmed | The capability later confirms a Pending bank transfer | Payment `status` set to Succeeded, `paid_at` recorded | Owen and Nadia both see Paid; the delayed confirmation receipt is released | FEAT-10.SPEC-004, FEAT-10.SPEC-001, FEAT-10.SPEC-007 (Payment Confirmation Notification) |
| Pending bank transfer failed | The capability reports a Pending bank transfer could not be completed | Payment `status` set to Failed | Owen sees "Your bank transfer could not be completed. You can try again." and Nadia sees the same reverted status on her own invoice view | FEAT-10.SPEC-004, FEAT-10.SPEC-001 |

Handling for every event above is direct (a status field update and a screen reflecting it); the branching decision of what Invoice status and reminder-schedule effect each Payment status implies is defined in FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync), which names this spec as its external-event trigger source.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-10.SPEC-001 (Pay Invoice Screen) | The Pay button shows its processing state; after 10 seconds a note appears below it: "Still processing -- this is taking longer than usual." No second submission is offered while the first is outstanding. | The Pay button is disabled with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." -- the same message used when the Payment Account Connection itself needs attention (XBR-19), since from Owen's side the effect is identical: no way to pay in-portal right now. The rest of the invoice (amount, due date, download) remains fully usable. | Owen sees "Payment declined: {failure_reason}" with an immediate "Try again" option; the invoice remains correctly unpaid (Sent or Overdue) and no Payment record is left in an ambiguous state. |
| FEAT-10.SPEC-002 (Record Off-Platform Payment Screen) | N/A -- this screen never sends a request to the payment-processing capability; only FEAT-10.SPEC-005's manual-record path applies, which does not depend on this capability at all. | N/A -- same reason as above. | N/A -- same reason as above. |

## Consent and Disclosure

- **First payment disclosure** -- The first time Owen submits a payment on any invoice from this freelancer, a notice appears before the request is sent: "To process this payment, the invoice amount, invoice number, and your name and email are shared with an external payment-processing service." Options: "Continue" and "Cancel." Shown once per freelancer relationship; a "How payment data is shared" link on the Pay Invoice Screen reopens the same notice afterward.
- **Card/bank details entered on the capability's own step** -- If the payment flow hands off to a step the payment-processing capability itself presents (for entering a card number or bank details), that step states plainly that Clientroom never receives or stores those details -- only the capability's own outcome report returns to the product.
- **What is never shared** -- Card numbers, bank account or routing numbers, and every Client Contact or Invoice field beyond the amount, currency, invoice number, payer name, and payer email listed in Data Exchanged stay inside the product. This boundary is stated in the disclosure notice.

## Edge Cases

- **The same outcome event is delivered twice (for example, a duplicate "card payment succeeded" report)** -- The second delivery changes nothing: a Payment already Succeeded stays Succeeded with its original `paid_at`, and FEAT-10.SPEC-004 fires no duplicate confirmation.
- **Events arrive out of order (a "failed" report arrives after a later "succeeded" report for a retried attempt)** -- Each report is matched to its own distinct payment attempt (not the invoice as a whole); a failed report for an earlier, already-superseded attempt on the same invoice is recorded against that attempt and does not overwrite a subsequent Succeeded attempt.
- **An outcome event arrives for an invoice that has since reached Paid through a different path (Nadia's manual record)** -- Per XBR-20/XBR-22, processor-confirmed status is authoritative: if the processor's own event reports Succeeded, it is the genuine payment and the invoice's status is corrected to reflect the processor's record, consistent with "an invoice is paid once and in full only" -- this scenario is flagged to Nadia as a discrepancy notice rather than silently discarded, since two distinct payments would otherwise go unnoticed.
- **The capability goes down mid-submission, after the request was sent but before any outcome is confirmed** -- The Pay Invoice Screen shows the capability-down message and no Payment record is left in an ambiguous state; if the capability's own report never arrives, the attempt is treated as not completed and Owen can submit again once the capability recovers.
- **Owen submits a payment while the invoice's due date has just passed (Overdue)** -- Not applicable as a degradation case; the invoice's Overdue flag (FEAT-11) does not block payment eligibility, so the request proceeds normally regardless of due-date status.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Triggered by (inbound) | Pay and Try again initiate a submission to this capability |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Affects (outbound) | Degradation states and disclosure notice surface here |
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Triggers (outbound) | Every inbound event above fires this automation as its external-event trigger source |
| FEAT-10.SPEC-006 (Payment Authorization & Validation Rules) | References (inbound) | The Own-only pay gate and payment-account-readiness gate are checked before a request reaches this spec |
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | References (inbound) | Supplies the freelancer's processor account reference and readiness status this spec depends on |
| FEAT-10.SPEC-007 (Payment Confirmation Notification) | Triggers (outbound) | The card-succeeded and pending-transfer-confirmed events lead to this notification via FEAT-10.SPEC-004 |

## Analytics and Success Signals

- **payment_initiated** (method: card / bank_transfer, invoice reference) -- supports success-metrics.md: "Time to Payment"
- **payment_outcome_received** (outcome: succeeded / failed / pending, method) -- supports success-metrics.md: "Time to Payment"
- **payment_degradation_shown** (condition: slow / down / rejected, screen: FEAT-10.SPEC-001) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable rather than invisible.

## Acceptance Criteria

**FEAT-10.SPEC-003-AC-01:** Given Owen selects Card and taps Pay on an open invoice, when the request is submitted, then the invoice amount, currency, invoice number, and Owen's name and email are sent to the payment-processing capability, and no card number is ever received by the product.

**FEAT-10.SPEC-003-AC-02:** Given a submitted card payment, when the capability reports it succeeded, then the Payment record's status is set to Succeeded with a recorded `paid_at`.

**FEAT-10.SPEC-003-AC-03:** Given a submitted card payment, when the capability reports it declined, then the Payment record's status is set to Failed and Owen sees "Payment declined: {failure_reason}" with an immediate retry option.

**FEAT-10.SPEC-003-AC-04:** Given Owen selects Bank transfer and taps Pay, when the request is submitted, then the Payment record's status is set to Pending and Owen sees "Payment pending -- waiting on your bank transfer to confirm."

**FEAT-10.SPEC-003-AC-05:** Given a Pending bank-transfer Payment, when the capability later confirms it, then the Payment record's status is set to Succeeded with a recorded `paid_at`, and the delayed confirmation notification (FEAT-10.SPEC-007) is released.

**FEAT-10.SPEC-003-AC-06:** Given a Pending bank-transfer Payment, when the capability reports it ultimately failed, then the Payment record's status is set to Failed and both Owen and Nadia see the reverted status.

**FEAT-10.SPEC-003-AC-07:** Given the payment-processing capability is slow to respond, when 10 seconds have elapsed with no outcome, then Owen sees "Still processing -- this is taking longer than usual." below the processing Pay button.

**FEAT-10.SPEC-003-AC-08:** Given the payment-processing capability is unavailable, when Owen attempts to pay, then the Pay button is disabled with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." and the invoice is unchanged.

**FEAT-10.SPEC-003-AC-09:** Given Owen has never submitted a payment to this freelancer before, when he taps Pay for the first time, then the data-sharing disclosure notice appears with "Continue" and "Cancel," and no data leaves the product until he chooses "Continue."

**FEAT-10.SPEC-003-AC-10:** Given a card-payment-succeeded event is delivered twice for the same Payment, when the second delivery arrives, then nothing changes and no duplicate confirmation notification fires.

**FEAT-10.SPEC-003-AC-11:** Given a failed-outcome event for a superseded, earlier attempt arrives after a later attempt on the same invoice already succeeded, when the out-of-order event is processed, then the invoice's status remains Paid from the later, successful attempt.

**FEAT-10.SPEC-003-AC-12:** Given the capability goes down after Owen's request is sent but before any outcome is confirmed, when Owen returns to the screen, then no Payment record is left in an ambiguous state and he can submit again once the capability recovers.

**FEAT-10.SPEC-003-AC-13:** Given Owen's payment is submitted after the invoice's due date has passed, when the request reaches this capability, then it is processed identically to a payment submitted before the due date.

**FEAT-10.SPEC-003-AC-14:** Given the Record Off-Platform Payment Screen (FEAT-10.SPEC-002) never sends a request to this capability, when Nadia records a manual payment, then no degradation state from this spec ever applies to that screen.

**FEAT-10.SPEC-003-AC-15:** Given a payment succeeds, when the outcome is applied, then the payment_outcome_received event fires with outcome: succeeded, supporting the "Time to Payment" metric.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 5 | 5 |
| Degradation Paths | 3 (1 screen fully covered; 3 N/A cells on FEAT-10.SPEC-002 excluded) | 3 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Payment Confirmation & Invoice Status Sync

## Overview

**Name:** Payment Confirmation & Invoice Status Sync
**ID:** FEAT-10.SPEC-004
**Type:** Automation
**Purpose:** Applies the payment-processing capability's reported outcome to the Payment and Invoice records the instant it arrives -- Paid, Payment pending, or back to Unpaid -- visible to both Owen and Nadia, and refuses a second attempt on an already-paid invoice.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- Applying each of FEAT-10.SPEC-003's inbound events to the Payment record's `status` and `paid_at`, and to the Invoice record's `status`
- Pausing the invoice's reminder schedule when a bank transfer becomes Pending, and its cross-feature trigger to resume/stop when the outcome resolves
- Triggering the payment confirmation notification when a card payment or a confirmed bank transfer succeeds
- Concurrent-run and duplicate-event handling for the same invoice

**Non-Goals:**
- Submitting the payment request or receiving the raw outcome report from the payment-processing capability -- owned by FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing); this automation begins only once that spec's inbound event fires.
- The reject-with-refresh behavior when Owen attempts to pay an already-paid invoice -- that refusal happens inline on FEAT-10.SPEC-001 (Pay Invoice Screen), governed by FEAT-10.SPEC-006; this automation only ever runs against an outcome for a genuinely new, in-flight attempt.
- Recording a payment made outside the portal -- owned by FEAT-10.SPEC-005 (Record Off-Platform Payment); this automation processes only processor-confirmed outcomes.
- Actually stopping or pausing the Reminder Log's send schedule -- owned entirely by Automated Payment Reminders (FEAT-11); this automation only fires the pause/resume trigger, per the Brief's Side-Effect Inventory disposition ("reminder pause itself is cross-feature").

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Card payment succeeded | FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | Fires when the capability reports a submitted card payment succeeded | Payment reference, Invoice reference, `paid_at` |
| Card payment failed/declined | FEAT-10.SPEC-003 | Fires when the capability declines a submitted card payment | Payment reference, Invoice reference, failure reason |
| Bank transfer submitted (pending) | FEAT-10.SPEC-003 | Fires when Owen submits a bank-transfer payment and the capability acknowledges the attempt | Payment reference, Invoice reference |
| Pending bank transfer confirmed | FEAT-10.SPEC-003 | Fires when the capability later confirms a Pending bank-transfer Payment | Payment reference, Invoice reference, `paid_at` |
| Pending bank transfer failed | FEAT-10.SPEC-003 | Fires when the capability reports a Pending bank-transfer Payment could not be completed | Payment reference, Invoice reference |

## Processing Logic

1. Receive the reported event and its Payment/Invoice reference from FEAT-10.SPEC-003.
2. Read the Invoice's current `status`. If it is already Paid or Paid (recorded by freelancer), stop processing this event without changing the Invoice or Payment record further (see Edge Cases -- this guards against a race with a manual record or a duplicate delivery), except that a genuinely new Succeeded report against an invoice already Paid by a different path is flagged as a discrepancy for Nadia rather than silently discarded.
3. For a card-succeeded or bank-transfer-confirmed event: set Payment `status` to Succeeded and record `paid_at`; set Invoice `status` to Paid.
4. For a card-failed/declined event: set Payment `status` to Failed; leave Invoice `status` unchanged (it remains Sent or Overdue, whichever it already was) -- the invoice stays correctly unpaid rather than showing a false Paid.
5. For a bank-transfer-submitted event: set Payment `status` to Pending; set Invoice `status` to Payment pending; fire the reminder-pause trigger toward FEAT-11 (Automated Payment Reminders).
6. For a bank-transfer-failed event: set Payment `status` to Failed; return Invoice `status` to its pre-attempt state (Sent, or Overdue if the due date has since passed); fire the reminder-resume trigger toward FEAT-11.
7. When step 3 completes (a Succeeded outcome, by either path), trigger FEAT-10.SPEC-007 (Payment Confirmation Notification) once, immediately.
8. Regardless of outcome, the applied status is visible to both Owen (FEAT-10.SPEC-001) and Nadia (her own invoice view, FEAT-09) the instant this step completes -- neither side needs to refresh through a separate action for the change to be reflected on next view.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Card payment applied as Paid | Card-succeeded event processed | Payment Succeeded, `paid_at` set; Invoice Paid | Owen sees "Paid on {paid_at_formatted}"; Nadia sees the same status on her invoice view | FEAT-10.SPEC-001, FEAT-10.SPEC-007 |
| Card payment applied as Failed | Card-failed event processed | Payment Failed; Invoice unchanged (stays Sent/Overdue) | Owen sees "Payment declined: {failure_reason}" with immediate retry | FEAT-10.SPEC-001 |
| Bank transfer applied as Pending | Bank-transfer-submitted event processed | Payment Pending; Invoice Payment pending; reminder schedule paused | Owen and Nadia both see "Payment pending" | FEAT-10.SPEC-001, FEAT-11 |
| Bank transfer applied as Paid | Pending-confirmed event processed | Payment Succeeded, `paid_at` set; Invoice Paid | Owen and Nadia both see Paid; confirmation notification released | FEAT-10.SPEC-001, FEAT-10.SPEC-007, FEAT-11 |
| Bank transfer applied as Failed (reverted) | Pending-failed event processed | Payment Failed; Invoice reverted to Sent/Overdue; reminder schedule resumed | Owen and Nadia both see a clear notice that the transfer failed | FEAT-10.SPEC-001, FEAT-11 |
| No-op (already resolved) | Invoice already Paid or Paid (recorded by freelancer) when the event arrives | None, except a discrepancy flag for Nadia when the event itself reports a genuine new Succeeded outcome | No change visible to Owen; Nadia may see a discrepancy notice on her dashboard (FEAT-12) | FEAT-10.SPEC-001 (unchanged), FEAT-12 |
| Automation failure | Processing cannot complete (for example, the Invoice or Payment record cannot be reached) | No partial write -- either the full status change and its side effects are applied, or none are | Owen's screen shows no change until the automation retries and succeeds; no false Paid or false Pending state is ever shown | FEAT-10.SPEC-001 |

## Data Model

**Reads:** Invoice -- `status`. Payment -- the reference passed by the triggering event.
**Creates:** None -- the Payment record itself is created by FEAT-10.SPEC-003 at the moment the attempt is submitted; this automation only updates its `status` and `paid_at`.
**Updates:** Payment -- `status`, `paid_at`. Invoice -- `status`.
**Deletes:** None.

## Business Rules

- An invoice already Paid or Paid (recorded by freelancer) is never overwritten by this automation except to flag a genuine processor-reported discrepancy -- the first confirmed full payment wins (dependency map, Entity: Payment, Contention; XBR-20).
- A Payment pending state pauses the invoice's reminder schedule; a reverted-to-Failed outcome resumes it -- both are fired as cross-feature triggers toward FEAT-11, which owns the actual Reminder Log state (XBR-15).
- Every status change this automation applies is immediately visible to both Owen and Nadia -- neither side's screen depends on the other refreshing first (Shared UI Patterns, feature-overview.md: "both sides of the same invoice never disagree").
- This automation runs synchronously with the inbound event from FEAT-10.SPEC-003 -- there is no batching or delay between the capability's report and the applied status.
- A failed or declined card payment never changes the Invoice's status -- the invoice remains correctly unpaid so a false Paid state is never shown (feature-overview.md, Key Capabilities: Failure handling).

## Edge Cases

- **The same outcome event is delivered twice** -- The second delivery is a no-op per step 2's already-resolved guard: the Payment and Invoice stay at their already-applied values, and FEAT-10.SPEC-007 does not fire a second time for the same Succeeded outcome.
- **A card-failed event arrives for an attempt that a later attempt on the same invoice has already succeeded** -- The invoice remains Paid from the later, successful attempt; the failed event is recorded only against its own, earlier Payment attempt record and never reverts the invoice.
- **Concurrent trigger firing (a card-succeeded event and a card-failed event for two different attempts on the same invoice arrive at effectively the same time)** -- Only one attempt can be genuinely current per the payment-processing capability's own sequencing; this automation applies whichever event's underlying attempt the capability confirms as the completed one, and the invoice's final status reflects that attempt, not an arrival-order race.
- **A trigger fires while a previous run for the same invoice is still in flight** -- A second event for the same invoice is processed only after the in-flight run's write to the Invoice record completes, so the two runs apply in sequence rather than overwriting each other's partial state; runs for different invoices proceed independently and never queue behind each other.
- **A Succeeded event arrives for an invoice that Nadia has already manually recorded as "Paid (recorded by freelancer)"** -- Per XBR-20/XBR-22, the processor's confirmation is the genuine payment; this is treated as a discrepancy (two distinct payments received for one invoice) and flagged to Nadia rather than silently discarded, since it indicates the client paid both in-portal and through the outside channel Nadia recorded.
- **The automation itself fails mid-write** -- No partial state is left: either both the Payment and Invoice updates (and any reminder trigger) commit together, or neither does, so Owen and Nadia never see a Payment marked Succeeded against an Invoice still showing Sent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | Triggered by (inbound) | Every inbound event this automation handles originates from that spec |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Affects (outbound) | The applied status and any decline reason are what that screen displays |
| FEAT-10.SPEC-007 (Payment Confirmation Notification) | Triggers (outbound) | A Succeeded outcome (card or confirmed bank transfer) fires this notification |
| FEAT-11 (Automated Payment Reminders) | Affects (outbound) | Reminder pause/resume triggers on Payment pending and its resolution |
| FEAT-12 (Freelancer Financial Dashboard) | Affects (outbound) | Dashboard totals derive from the Invoice/Payment status this automation applies (XBR-22); the discrepancy flag surfaces there |

## Analytics and Success Signals

- **payment_succeeded** (method: card / bank_transfer) -- supports success-metrics.md: "Time to Payment"
- **payment_failed** (method, failure_reason category) -- supports success-metrics.md: "Time to Payment"
- **payment_pending** (method: bank_transfer) -- supports success-metrics.md: "Time to Payment"
- **payment_pending_resolved** (outcome: succeeded / failed) -- supports success-metrics.md: "Time to Payment"
- **payment_status_discrepancy_flagged** (invoice reference) -- N/A -- no Stage 2 metric measures this rare concurrency outcome; retained so a genuine double-payment is never silently invisible to product operators.

## Acceptance Criteria

**FEAT-10.SPEC-004-AC-01:** Given a card payment attempt in flight, when FEAT-10.SPEC-003 reports it succeeded, then Payment status is set to Succeeded with `paid_at` recorded, Invoice status is set to Paid, and FEAT-10.SPEC-007 fires immediately.

**FEAT-10.SPEC-004-AC-02:** Given a card payment attempt in flight, when FEAT-10.SPEC-003 reports it declined, then Payment status is set to Failed and Invoice status remains unchanged (Sent or Overdue).

**FEAT-10.SPEC-004-AC-03:** Given Owen submits a bank-transfer payment, when FEAT-10.SPEC-003 acknowledges the submission, then Payment status is set to Pending, Invoice status is set to Payment pending, and the reminder-pause trigger fires toward FEAT-11.

**FEAT-10.SPEC-004-AC-04:** Given a Pending bank-transfer Payment, when the capability confirms it, then Payment status is set to Succeeded with `paid_at` recorded, Invoice status is set to Paid, and FEAT-10.SPEC-007 fires.

**FEAT-10.SPEC-004-AC-05:** Given a Pending bank-transfer Payment, when the capability reports it failed, then Payment status is set to Failed, Invoice status reverts to Sent or Overdue, and the reminder-resume trigger fires toward FEAT-11.

**FEAT-10.SPEC-004-AC-06:** Given an invoice is already Paid, when a duplicate delivery of the same Succeeded event arrives, then nothing changes and FEAT-10.SPEC-007 does not fire again.

**FEAT-10.SPEC-004-AC-07:** Given an invoice already shows "Paid (recorded by freelancer)" from Nadia's manual entry, when the payment-processing capability reports a genuine Succeeded outcome for the same invoice, then a discrepancy is flagged to Nadia rather than silently discarded.

**FEAT-10.SPEC-004-AC-08:** Given a card-failed event for an earlier attempt arrives after a later attempt on the same invoice already succeeded, when the out-of-order event is processed, then the invoice remains Paid from the later, successful attempt.

**FEAT-10.SPEC-004-AC-09:** Given two events for the same invoice arrive at effectively the same time, when they are processed, then the invoice's final status reflects the attempt the payment-processing capability itself confirms as completed, not arrival order.

**FEAT-10.SPEC-004-AC-10:** Given a trigger fires for an invoice while a previous run for that same invoice is still in flight, when the second trigger is received, then it is processed only after the first run's write completes, and the two never overwrite each other's partial state.

**FEAT-10.SPEC-004-AC-11:** Given this automation fails mid-write, when the failure occurs, then no partial state is visible -- the Payment and Invoice updates and the reminder trigger either all commit together or none do.

**FEAT-10.SPEC-004-AC-12:** Given an outcome is applied to an invoice, when the write completes, then both Owen's and Nadia's views of that invoice reflect the same status without either side needing the other to act first.

**FEAT-10.SPEC-004-AC-13:** Given a card payment is declined, when the automation processes it, then the invoice is never shown as Paid -- it stays correctly unpaid.

**FEAT-10.SPEC-004-AC-14:** Given a bank-transfer payment is confirmed after being Pending, when the confirmation is applied, then the reminder schedule (already paused) is not separately re-paused, and the invoice reaches Paid exactly once.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 7 (5 named outcomes plus no-op and automation-failure) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



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



# Logic/Rule Spec: Payment Authorization & Validation Rules

## Overview

**Name:** Payment Authorization & Validation Rules
**ID:** FEAT-10.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may pay, view, or manually record a payment, the full-payment-only and manual-record limits, the reject-with-refresh concurrency behavior, and the payment-account-readiness gate.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing
**Governed Entity:** Payment

## Scope and Non-Goals

**In Scope:**
- Field validation rules for the Payment record (amount, method, paid_at)
- Cross-field rules tying a manual record's amount to the invoice total
- Authorization rules for every action on Payment (pay, retry, view, record manually), per role
- The payment-account-readiness gate and the reject-with-refresh concurrency rule
- Default values and derivations for Payment fields

**Non-Goals:**
- The Invoice entity's own broader lifecycle rules (Overdue flagging, reminder scheduling) -- owned by FEAT-09 and FEAT-11; this spec governs only the Payment-related actions and the Invoice `status` transitions those actions cause.
- Refund, reversal, or chargeback rules -- owned entirely by FEAT-25 (Refund & Cancelled Project Handling); this spec's authority ends once a Payment first reaches Succeeded, Failed, or Recorded manually.
- The mechanics of submitting a request to the payment-processing capability -- owned by FEAT-10.SPEC-003; this spec defines only who may initiate that submission and under what conditions.

## Governed Entity

**Entity:** Payment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice | text (reference) | The Invoice this payment is against |
| amount | number | The amount paid -- always the full invoice amount; no partial payments |
| method | enum | card \| bank transfer \| a recorded off-platform method (bank transfer, cash, cheque, other) |
| paid_at | date | The date/time the payment was made or confirmed |
| status | enum | Initiated \| Pending \| Succeeded \| Failed \| Reversed \| Recorded manually |
| recorded_by | text (reference) | Nadia's identity, present only for manually recorded payments |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-10.SPEC-001 | Pay Invoice Screen | On screen entry (Own-only gate, payment-account-readiness gate) and on Pay/Try again tap (reject-with-refresh) |
| FEAT-10.SPEC-002 | Record Off-Platform Payment Screen | On screen entry (Nadia-only gate) and on field blur/form submit (full-amount and not-future-dated validation) |
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | Before submitting a request (Own-only gate, readiness gate) |
| FEAT-10.SPEC-004 | Payment Confirmation & Invoice Status Sync | During processing (already-resolved guard against a stale in-flight outcome) |
| FEAT-10.SPEC-005 | Record Off-Platform Payment | During processing (full-amount, not-future-dated, and already-resolved checks) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| amount | Must equal the invoice's full `total` -- no partial amounts | Always | On save (FEAT-10.SPEC-003, FEAT-10.SPEC-005) | "This invoice must be paid in full." | Yes |
| method | Required; must be one of the valid values for the initiating path (card or bank transfer for FEAT-10.SPEC-003; bank transfer, cash, cheque, or other for FEAT-10.SPEC-005) | Always | On submit | "Choose a payment method." | Yes |
| paid_at (manual records only) | Cannot be a future date | Only for a manually recorded payment (FEAT-10.SPEC-005) | On blur and on submit (FEAT-10.SPEC-002) | "Payment date cannot be in the future." | Yes |
| paid_at (processor-confirmed records) | No validation beyond data type -- this value is set atomically by the payment-processing capability's own report, never entered by a user | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions are governed entirely by Business Rules below, never by a user-entered value | Always | -- | -- | -- |
| recorded_by | No validation beyond data type -- always Nadia for a manual record, always absent for a processor-confirmed one; not a user input | Always | -- | -- | -- |
| invoice | No validation beyond data type -- fixed by the entry context (the invoice being paid or recorded) and never user-selected on this screen | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Full-payment-only | amount, invoice | A Payment's `amount` must equal its `invoice`'s `total` at the moment of creation -- there is no field for entering a different amount, so this rule is enforced structurally rather than as a separate check the user can fail, except that FEAT-10.SPEC-005's automation re-verifies it before creating the record | "This invoice must be paid in full." |
| Manual record requires an eligible invoice | status, invoice.status | A Payment with `status` = Recorded manually may only be created while the referenced Invoice's `status` is not already Paid or Paid (recorded by freelancer) | "This invoice was already paid online. Refresh to see the current status." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Pay an invoice (card or bank transfer) | Owen (Client Primary Contact) | Only his own company's invoice (Own-only); only while the invoice's status is Sent or Overdue (not already in a Paid-family status); only while the Payment Account Connection status is Connected | If the invoice is already Paid: the Pay control is replaced by the current status and a direct attempt is refused with the refreshed status shown (reject-with-refresh). If the Payment Account Connection is not Connected: the Pay control is replaced entirely by "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." |
| Pay an invoice (card or bank transfer) | Nadia (Freelancer) | Never -- Nadia is not a payer or retry actor for her own invoices under any condition | No Pay control exists on any freelancer-side view of her own invoice; paying is exclusively a client-side action performed by the invoice's Client Primary Contact |
| Pay an invoice (card or bank transfer) | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| Pay an invoice (card or bank transfer) | Dana (Support Operator) | Never -- her access to Invoicing & Payments is View only, never Pay, regardless of session state | No Pay control is ever rendered for her, inside or outside a logged support session; a direct attempt has no control to trigger |
| Retry a failed or declined payment | Owen (Client Primary Contact) | Same conditions as Pay -- only his own company's invoice, only while not already Paid, only while the connection is ready | Same denied behavior as Pay |
| Retry a failed or declined payment | Nadia (Freelancer) | Never -- Nadia is not a payer or retry actor for her own invoices under any condition | No "Try again" control exists on any freelancer-side view of her own invoice; retrying is exclusively a client-side action performed by the invoice's Client Primary Contact |
| Retry a failed or declined payment | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| Retry a failed or declined payment | Dana (Support Operator) | Never -- her access to Invoicing & Payments is View only, never Pay or Retry, regardless of session state | No "Try again" control is ever rendered for her, inside or outside a logged support session; a direct attempt has no control to trigger |
| View payment status | Nadia (Freelancer) | Always, for any of her own invoices (Full) | -- |
| View payment status | Owen (Client Primary Contact) | Only his own company's invoice (Own-only) | Any invoice outside his own company's scope shows "You don't have access to invoices for this account." per XBR-09 |
| View payment status | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | The invoice detail and pay link are never shown to her; a direct link attempt shows "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| View payment status | Dana (Support Operator) | Only inside a logged, read-only support session (FEAT-31), and only status -- never a Pay or Record control | Outside a support session, no access exists at all; inside one, any Pay or Record action attempt has no control to trigger, since none is rendered for her |
| Record an off-platform payment | Nadia (Freelancer) | Only for her own invoices; only while the invoice's status is not already Paid or Paid (recorded by freelancer); amount always the full total; date never in the future | If the invoice is already Paid online: "This invoice was already paid online. Refresh to see the current status." |
| Record an off-platform payment | Owen (Client Primary Contact) | Never | The "Record a payment received elsewhere" action does not exist on Owen's side of the product; he has no screen offering it |
| Record an off-platform payment | Priya (Client Reviewer Contact) | Never | Same as Owen -- no such action exists for any client contact |
| Record an off-platform payment | Dana (Support Operator) | Never | The action is never shown inside a support session; her access to Invoicing & Payments is View only |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| amount | Derived from the referenced Invoice's `total` at the moment the Payment record is created | On create only | No |
| paid_at (manual record) | Defaults to today's date on the entry screen (FEAT-10.SPEC-002) | On create only | Yes -- Nadia may choose any non-future date |
| paid_at (processor-confirmed) | Set by the payment-processing capability's own report at the moment of confirmation | On create/update (FEAT-10.SPEC-004) | No |
| status | Initiated at submission, transitioning to Pending, Succeeded, or Failed as the capability reports each stage (FEAT-10.SPEC-004); set directly to Recorded manually for a manual entry (FEAT-10.SPEC-005) | On create and on update | No -- transitions are system-driven, never user-selected |
| recorded_by | Set to Nadia automatically for a manual record; never set for a processor-confirmed record | On create only (manual path) | No |

## Business Rules

- The payment-account-readiness gate: paying or retrying is available only while the freelancer's Payment Account Connection status is Connected; a Needs attention or Disconnected status blocks the action entirely with the "temporarily unavailable" message, never a failed payment attempt (XBR-19).
- Reject-with-refresh concurrency: the first confirmed full payment on an invoice wins; any later attempt -- whether a second card/bank-transfer submission or a manual record -- is refused and the actor is shown the invoice's current, refreshed status rather than being allowed to process against a stale view (dependency map, Entity: Payment, Contention).
- Processor-authoritative-over-manual: when a processor-confirmed payment and a manual record could both apply to the same invoice, the processor's confirmation is authoritative; a manual record attempted after the processor has already confirmed payment is refused (XBR-20).
- An invoice is paid once and in full only -- no rule in this spec, nor any enforcing spec, defines a path to a partial payment or a second successful payment on the same invoice (XBR-20, scope-boundaries.md SC-17).
- Dana's View access to Invoicing & Payments never includes a Pay or Record control under any condition -- her role is bounded to read-only regardless of invoice or connection state (XBR-29).

## Edge Cases

- **Two payment attempts (a card submission and a bank-transfer submission) are started by Owen in two open tabs on the same invoice** -- Both submissions are accepted individually by FEAT-10.SPEC-003, but only the first to reach a Succeeded outcome sets the invoice to Paid; FEAT-10.SPEC-004's already-resolved guard refuses to let the second outcome overwrite the first, and Owen's second tab refreshes to show the invoice already Paid.
- **The Payment Account Connection changes from Connected to Needs attention between Owen loading the Pay Invoice Screen and tapping Pay** -- The readiness gate is re-checked authoritatively at the moment of submission, not just at page load; if it has changed, the attempt is refused with the same unavailable message rather than being sent to a connection that will reject it.
- **Owen's own invoice reaches exactly its due date at the moment he pays (boundary condition on Overdue)** -- Not a boundary this spec governs; the Overdue flag (owned by FEAT-11) has no bearing on payment eligibility, so payment proceeds identically whether the invoice is Sent or Overdue.
- **Nadia enters a manual-record date of exactly today** -- Passes validation; the not-future-dated rule is inclusive of the current date, matching FEAT-10.SPEC-005's own boundary handling.
- **A client contact's role changes from Reviewer to Primary while an invoice is outstanding (cross-feature: FEAT-18)** -- The newly promoted Primary contact gains Pay access from the moment the role change takes effect; the Authorization Rules above apply to the role as currently assigned, never to a role held at some earlier point.
- **Dana's support session ends while she is viewing an invoice's payment status** -- Her View access ends immediately with the session; no lingering access to payment status persists after FEAT-31 closes the session.

## Acceptance Criteria

**FEAT-10.SPEC-006-AC-01:** Given Owen is on his own company's invoice with a Sent status and the Payment Account Connection is Connected, when he attempts to pay, then the attempt is allowed.

**FEAT-10.SPEC-006-AC-02:** Given the invoice is already Paid, when Owen attempts to pay against his stale view, then the attempt is refused and he is shown the refreshed, current status.

**FEAT-10.SPEC-006-AC-03:** Given the Payment Account Connection status is Needs attention, when Owen attempts to pay, then the attempt is blocked with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way."

**FEAT-10.SPEC-006-AC-04:** Given Owen's payment was declined, when he taps "Try again," then the same Own-only and readiness conditions are re-checked before the retry is submitted.

**FEAT-10.SPEC-006-AC-05:** Given Nadia views any of her own invoices, when she checks payment status, then she can always see it (Full access).

**FEAT-10.SPEC-006-AC-06:** Given Priya (Client Reviewer Contact) attempts to view any invoice, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-07:** Given Dana is inside a logged support session, when she views an invoice's payment status, then she sees it read-only with no Pay or Record control anywhere on the screen.

**FEAT-10.SPEC-006-AC-08:** Given Nadia enters a valid date and method for an eligible invoice, when she records a manual payment, then it is allowed and the amount is set to the invoice's full total automatically.

**FEAT-10.SPEC-006-AC-09:** Given the invoice was already paid online before Nadia's save commits, when she attempts to record a manual payment, then it is refused with "This invoice was already paid online. Refresh to see the current status."

**FEAT-10.SPEC-006-AC-10:** Given Owen looks for a "Record a payment received elsewhere" action on his own screens, when he searches for it, then no such action exists for his role under any condition.

**FEAT-10.SPEC-006-AC-11:** Given Priya looks for any payment-related action, when she searches for it, then none is ever shown, since Invoicing & Payments is None for her role.

**FEAT-10.SPEC-006-AC-12:** Given Nadia attempts to record a manual payment with a future date, when she submits, then the amount rule is never reached because the date rule already blocks the save with "Payment date cannot be in the future."

**FEAT-10.SPEC-006-AC-13:** Given a Payment record is created by any path, when its `amount` is set, then it always equals the invoice's full `total` -- never a partial amount.

**FEAT-10.SPEC-006-AC-14:** Given Nadia enters exactly today's date for a manual record, when she saves, then the date passes validation.

**FEAT-10.SPEC-006-AC-15:** Given two submissions are made on the same invoice in two open sessions, when both reach the point of resolution, then only the first confirmed full payment is applied and the second is refused with the refreshed status.

**FEAT-10.SPEC-006-AC-16:** Given a processor-confirmed payment and a concurrent manual record both target the same invoice, when both attempt to resolve, then the processor's confirmation is authoritative and the manual attempt is refused.

**FEAT-10.SPEC-006-AC-17:** Given the Payment Account Connection changes to Needs attention between page load and Owen's tap on Pay, when he taps Pay, then the readiness gate is re-checked at that moment and the attempt is blocked.

**FEAT-10.SPEC-006-AC-18:** Given an invoice's due date has passed (Overdue), when Owen or Nadia interacts with payment on it, then the Overdue flag has no effect on any authorization or validation outcome in this spec.

**FEAT-10.SPEC-006-AC-19:** Given a client contact's role changes from Reviewer to Primary, when the change takes effect, then that contact immediately gains Pay access per the Authorization Rules above, with no lag.

**FEAT-10.SPEC-006-AC-20:** Given Dana's support session ends, when the session closes, then her View access to that invoice's payment status ends immediately with it.

**FEAT-10.SPEC-006-AC-21:** Given Nadia is viewing one of her own invoices, when she looks for a way to pay it or retry a failed payment herself, then no Pay or "Try again" control exists for her under any condition -- paying and retrying are exclusively Owen's actions.

**FEAT-10.SPEC-006-AC-22:** Given Priya (Client Reviewer Contact) attempts to pay an invoice or retry a failed payment, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-23:** Given Dana is inside or outside a logged support session, when she looks for a way to pay an invoice or retry a failed payment, then no Pay or "Try again" control is ever rendered for her, regardless of session state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Payment Confirmation Notification

## Overview

**Name:** Payment Confirmation Notification
**ID:** FEAT-10.SPEC-007
**Type:** Notification
**Purpose:** Sends a confirmation email to Owen and Nadia the moment a card payment or a confirmed bank transfer succeeds, so both sides have the same timestamped record without checking the portal.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen and to Nadia when a card payment or a confirmed bank transfer succeeds
- The delayed-delivery rule for a bank transfer -- no receipt sends until the processor confirms it
- Preference, retry, and expiry behavior for this confirmation

**Non-Goals:**
- Confirming a manually recorded off-platform payment -- product-features.md's Communications field for this feature names only "a payment succeeds" (the processor-confirmed path) as triggering this email; Nadia's manual record is her own action and needs no confirmation email to herself, and Owen made no in-portal payment to confirm.
- Notifying anyone about a failed or declined payment -- the Brief's Side-Effect Inventory routes a decline to an inline screen state (FEAT-10.SPEC-001), not an email; a failure is not the record-worthy, permanent event this confirmation exists to echo.
- Notifying Priya -- she has no Invoicing & Payments access (Access Matrix: None), and a payment confirmation carries financial-commitment content she is not entitled to see (XBR-08).
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, when a card payment or a confirmed bank transfer succeeds | Owen's portal sessions are short and triggered by a specific email link (user-persona.md, Behavioral Context) -- he is not routinely inside the product afterward to see the confirmation happen live; Nadia works from her own inbox and needs the same permanent, timestamped record of money received without reopening the invoice to check |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payment succeeded | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires immediately and only once FEAT-10.SPEC-004 applies a Succeeded outcome to the Payment and Invoice records (card success, or a bank transfer's delayed confirmation) | Invoice `invoice_number`, `total`, `currency`; Payment `method`, `paid_at`; Client `client_name`; Client Contact name (Owen's); Freelancer Account contact details for Nadia |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Nadia (the Freelancer), per the Brief's Communications field ("Payment confirmation email to Owen and to Nadia when a payment succeeds"). Both are entitled to this content under the Access Matrix: Owen made or is party to the payment himself, and Nadia's Full access to Invoicing & Payments covers every payment event on her own account's invoices.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional record email, not an optional notification) |

This confirmation is a transactional email core to the record (XBR-30): it cannot be switched off by either recipient's notification preferences -- a payment confirmation is named explicitly in product-features.md's Communications rule as one that always sends.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this confirmation is transactional and sends immediately regardless of the time of day for either recipient, consistent with the payment it confirms being a permanent, timestamped record the moment it is confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** Payment confirmed for invoice {invoice_number}
- **Body:**
  Hi {owen_first_name},

  This confirms your payment of {total} {currency} for invoice {invoice_number}, paid by {payment_method_label} on {paid_at_formatted}.

  Keep this email as your receipt.
- **CTA (button):** View invoice -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (to Nadia):**
- **Subject:** {client_name} paid invoice {invoice_number}
- **Body:**
  Hi {nadia_first_name},

  {owen_name} at {client_name} paid {total} {currency} for invoice {invoice_number} by {payment_method_label} on {paid_at_formatted}. The payment has landed directly in your connected account.
- **CTA (button):** View invoice -- deep-links to the invoice's detail in her own project view (FEAT-09)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {invoice_number} | Invoice -- invoice_number | INV-0142 | Never empty -- unique and sequential per freelancer, required at generation (FEAT-09) |
| {total} | Invoice -- total | 1,200.00 | Never empty -- required at invoice generation |
| {currency} | Invoice -- currency | USD | Never empty -- required at invoice generation (FEAT-15) |
| {payment_method_label} | Payment -- method, rendered as "card" or "bank transfer" | card | Never empty -- set atomically by FEAT-10.SPEC-004 alongside the Succeeded status |
| {paid_at_formatted} | Payment -- paid_at, rendered in each recipient's own time zone (FEAT-15) | September 27, 2026, 3:14 PM | Never empty -- set atomically by FEAT-10.SPEC-004 at the moment this notification's trigger fires |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- required at client creation (FEAT-01) |
| {owen_name} | Client Contact -- name (the paying contact) | Owen Marsh | Never empty -- required at contact creation (FEAT-18) |
| {owen_first_name} | Client Contact -- name (first name portion) | Owen | Greeting renders as "Hi," |
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** None -- each successful payment is its own distinct, permanent event and is confirmed individually; two invoices paid close together each produce their own separate confirmation to each recipient, never merged into one summary email.
**Deduplication:** At most one confirmation email per recipient per Succeeded outcome. FEAT-10.SPEC-004's already-resolved guard is the deduplication boundary: this notification's trigger fires once per invoice reaching Paid via a processor-confirmed payment, so a duplicate delivery of the same underlying Succeeded event (per FEAT-10.SPEC-003's Edge Cases) never produces a second confirmation, since the guard prevents the trigger condition from being met twice.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30) -- for the copy addressed to Owen as well as her own, since a client contact never sees the freelancer's own delivery-warning surface and Nadia is positioned to notice and follow up.
**Expiry:** This confirmation never expires undelivered in the sense of becoming pointless to send late -- the payment it confirms is a permanent record, so a delayed delivery (after retries) still carries accurate, still-true information whenever it eventually lands. There is no cutoff after which the email is withheld; the retry window in the rule above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **A bank-transfer payment is Pending for several days before the processor confirms it** -- No receipt of any kind is sent while Pending (feature-overview.md, Key Capabilities: Pending bank transfers). This notification fires exactly once, only at the moment FEAT-10.SPEC-004 applies the confirmed Succeeded outcome -- never at the earlier Pending moment.
- **A Pending bank transfer is ultimately reported failed rather than confirmed** -- This notification never fires for that attempt, since its trigger condition (a Succeeded outcome) was never met; the failure is communicated inline on FEAT-10.SPEC-001, per that spec's States, not by this notification.
- **Owen's or Nadia's copy fails to deliver but the other's succeeds** -- Each recipient's copy is tracked and retried independently; one recipient's successful delivery has no bearing on the other's retry count or warning surfacing.
- **The same Succeeded event is delivered twice by the payment-processing capability (per FEAT-10.SPEC-003's Edge Cases)** -- FEAT-10.SPEC-004's already-resolved guard prevents a second confirmation-notification trigger; the second delivery changes nothing and no duplicate email is sent.
- **The underlying invoice reaches Paid through a processor confirmation but is later reversed as a chargeback (owned by FEAT-25)** -- This confirmation was accurate when sent, describing a payment that genuinely succeeded at that timestamp; a later reversal is a separate, new event handled entirely by FEAT-25's own notification, and does not retroactively make this confirmation false or worth recalling.
- **Quiet hours colliding with expiry** -- N/A, since this transactional confirmation observes neither quiet hours nor an expiry cutoff (both stated above); there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Triggered by (inbound) | A confirmed Succeeded outcome fires this notification |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Navigation (outbound) | Owen's CTA deep-links here |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Nadia's CTA deep-links to the invoice's detail in her own project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | References (inbound) | The card-succeeded and confirmed-bank-transfer inbound events are what ultimately produce this notification's trigger, via FEAT-10.SPEC-004 |

## Analytics and Success Signals

- **payment_confirmation_delivered** (recipient: owen / nadia, method: card / bank_transfer) -- supports success-metrics.md: "Time to Payment"
- **payment_confirmation_delivery_failed** (recipient: owen / nadia, retry_count) -- N/A -- no Stage 2 metric measures this feature's own delivery-failure rate directly; retained as a standard delivery-quality signal alongside the delivered event above.
- **payment_confirmation_opened** (recipient: owen / nadia) -- N/A -- no Stage 2 metric measures open rates for this specific confirmation; retained as a standard delivery-quality signal.

## Acceptance Criteria

**FEAT-10.SPEC-007-AC-01:** Given a card payment succeeds, when FEAT-10.SPEC-004 applies the Succeeded outcome, then Owen receives an email with subject "Payment confirmed for invoice {invoice_number}" and Nadia receives an email with subject "{client_name} paid invoice {invoice_number}."

**FEAT-10.SPEC-007-AC-02:** Given a bank-transfer payment is Pending, when it remains unconfirmed, then no confirmation email is sent to either recipient.

**FEAT-10.SPEC-007-AC-03:** Given a Pending bank-transfer payment is confirmed by the processor, when FEAT-10.SPEC-004 applies the Succeeded outcome, then this notification fires at that moment, releasing the receipt that was held while Pending.

**FEAT-10.SPEC-007-AC-04:** Given a Pending bank-transfer payment is ultimately reported failed, when the failure is applied, then this notification never fires for that attempt.

**FEAT-10.SPEC-007-AC-05:** Given Owen opens his confirmation email, when he taps "View invoice," then he lands on FEAT-10.SPEC-001 for that invoice.

**FEAT-10.SPEC-007-AC-06:** Given Nadia opens her confirmation email, when she taps "View invoice," then she lands on that invoice's detail in her own project view.

**FEAT-10.SPEC-007-AC-07:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-10.SPEC-007-AC-08:** Given a payment is confirmed at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours.

**FEAT-10.SPEC-007-AC-09:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-10.SPEC-007-AC-10:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-10.SPEC-007-AC-11:** Given the same Succeeded event is delivered twice by the payment-processing capability, when FEAT-10.SPEC-004's already-resolved guard blocks the second application, then this notification does not fire a second time.

**FEAT-10.SPEC-007-AC-12:** Given Priya (Client Reviewer Contact) is a contact at the same client company, when a payment succeeds on an invoice for that company, then Priya receives no copy of this confirmation.

**FEAT-10.SPEC-007-AC-13:** Given Nadia records an off-platform payment manually, when the manual record is saved, then this notification does not fire, since its trigger condition is a processor-confirmed Succeeded outcome, not a manual record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |

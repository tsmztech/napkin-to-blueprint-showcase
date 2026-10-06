---
document_type: feature-overview
feature_number: FEAT-10
feature_name: Invoice Payment Processing
feature_slug: invoice-payment-processing
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

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

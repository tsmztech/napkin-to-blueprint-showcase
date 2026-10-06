---
document_type: spec
spec_type: integration
spec_id: FEAT-10.SPEC-003
spec_name: Card & Bank-Transfer Payment Processing
spec_slug: card-bank-transfer-payment-processing
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

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

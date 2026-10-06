---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-005
spec_name: Payment Reversal (Chargeback) Recording
spec_slug: payment-reversal-chargeback-recording
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Payment Reversal (Chargeback) Recording

## Overview

**Name:** Payment Reversal (Chargeback) Recording
**ID:** FEAT-25.SPEC-005
**Type:** Automation
**Purpose:** Applies an inbound reversal or chargeback notice relayed from the payment-processing capability to a Paid invoice, setting it Disputed alongside its preserved Paid record and marking the underlying Payment Reversed.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Consuming the reversal/chargeback notice relayed by FEAT-32.SPEC-002's inbound event, already correlated to the affected Invoice
- Setting the Invoice's `status` to Disputed alongside its preserved Paid record
- Setting the underlying Payment's `status` to Reversed
- Notifying Nadia immediately once the reversal is recorded (via FEAT-25.SPEC-008)

**Non-Goals:**
- Receiving the reversal/chargeback event from the payment-processing capability, or correlating it to the right invoice -- owned entirely by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this automation begins with an already-correlated invoice reference
- Responding to the dispute or issuing any counter-evidence to the payment processor -- excluded per scope-boundaries.md (SC-18): dispute responses happen entirely inside Nadia's own processor account; this automation only records the outcome truthfully
- Applying Nadia's own manually entered refund -- owned entirely by FEAT-25.SPEC-003 (Refund & Partial Refund Recording); this automation handles only a processor-reported reversal, never a freelancer-initiated entry
- Marking the project cancelled as a consequence of a reversal -- a reversal never automatically cancels a project; if Nadia chooses to stop work as a result, that is her own separate action through FEAT-25.SPEC-002

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Reversal or chargeback reported | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the payment-processing capability reports a dispute on an invoice already recorded as Paid, at any time after payment | Invoice reference (already correlated by FEAT-32.SPEC-002), report timestamp |

## Processing Logic

1. Receive the correlated invoice reference and report timestamp from FEAT-32.SPEC-002's inbound event.
2. Read the Invoice's current `status` and the underlying Payment's current `status`.
3. Duplicate check, before any eligibility test: if the Invoice's `status` is already Disputed and the Payment's `status` is already Reversed, this notice is a duplicate of one already applied. Stop with the "Duplicate / already-Disputed no-op" outcome: write nothing, send no email (FEAT-25.SPEC-008 is not triggered), signal no trail entry, and emit payment_reversal_duplicate_ignored. Do not continue to step 4.
4. Confirm the Invoice is genuinely in a Paid-family status (Paid, Refunded, or Partially refunded) with a Payment in Succeeded status -- a reversal reported against an invoice with no processor-confirmed payment on record cannot apply and is discarded (see Edge Cases). This includes an invoice whose status is Paid (recorded by freelancer) with a Payment in Recorded manually status: an off-platform payment never passed through the payment-processing capability, so there is no processor charge to reverse. It also includes an invoice that is Disputed but whose Payment is not Reversed (an inconsistent state this automation does not repair).
5. Set the Invoice's `status` to Disputed, preserving its prior Paid (or Refunded/Partially refunded) record as the record that remains visible alongside the new status.
6. Set the underlying Payment's `status` to Reversed.
7. Trigger FEAT-25.SPEC-008 (Payment Reversal Notification) to email Nadia immediately.
8. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
9. Return the new status for display on FEAT-09.SPEC-002 (Invoice Detail).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Reversal recorded | The invoice is in a Paid-family status with a Succeeded Payment | Invoice `status` set to Disputed, preserving the prior Paid/Refunded/Partially-refunded record; Payment `status` set to Reversed | FEAT-09.SPEC-002 shows Disputed alongside the preserved prior record; Nadia is emailed immediately (FEAT-25.SPEC-008) | FEAT-09.SPEC-002, FEAT-25.SPEC-001, FEAT-25.SPEC-008, FEAT-13 |
| Duplicate / already-Disputed no-op | At step 3, the invoice is already Disputed and its Payment is already Reversed (a duplicate or repeated notice, including a concurrent second delivery) | None -- no write occurs; the invoice stays Disputed and the Payment stays Reversed | No feedback and no second email -- FEAT-25.SPEC-008 is not triggered and no trail entry is written, since no product event occurred; the ignored duplicate is recorded only through the payment_reversal_duplicate_ignored signal | -- |
| Notice discarded -- no matching payment | The correlated invoice has no Payment in Succeeded status (e.g., the invoice was never actually paid, was paid off-platform with a Recorded manually Payment, or its Payment record no longer exists) and is not an already-Disputed/Reversed duplicate | None -- no write occurs | No feedback -- there is no freelancer-visible state to update and no user waiting on this outcome; the discard itself is logged internally for diagnostic purposes only, not as an Activity Log Entry (no product event occurred) | -- |
| Automation failure | Persisting the Invoice/Payment changes fails after the Paid-family check passes (e.g., a transient write failure) | No partial write -- the Invoice and Payment are committed together or not at all | No end-user-facing feedback (this is a system-to-system event, not a screen submission); the event is retried by the same retry contract FEAT-32.SPEC-002 applies to its own inbound events | FEAT-32.SPEC-002 |

## Data Model

**Reads:** Invoice -- `status`. Payment -- `status`.
**Creates:** None.
**Updates:** Invoice -- `status` (Paid, Refunded, or Partially refunded -> Disputed). Payment -- `status` (Succeeded -> Reversed).
**Deletes:** None -- the prior Paid/Refunded/Partially-refunded record is preserved, never replaced (XBR-04).

## Business Rules

- XBR-21: When the payment processor reports a chargeback or reversal on a paid invoice, the invoice shows Disputed alongside its original Paid record and the freelancer is notified; refunds and dispute responses happen in her own processor account, never inside Clientroom.
- XBR-04: The original Paid record is never silently altered -- Disputed is a new, logged transition displayed alongside it, never a replacement of it.
- Per the dependency map's Invoice Contention note: processor-confirmed payment status is authoritative over a concurrent manual entry -- a reversal committed by this automation takes precedence over any concurrently submitted manual refund (FEAT-25.SPEC-003) or cancellation-adjacent action on the same invoice.
- This automation applies only to Payments that passed through the payment-processing capability (Payment `status` Succeeded). A manually recorded Payment (Recorded manually, invoice Paid (recorded by freelancer)) is outside its scope: a reversal notice against one is discarded, and Nadia's refund of such an invoice is recorded only through FEAT-25.SPEC-003.
- Idempotency: a repeated notice for an invoice already Disputed with a Payment already Reversed is a no-op (step 3) -- it changes nothing, sends no second email, and writes no second trail entry; FEAT-25.SPEC-008 relies on this so Nadia is emailed exactly once per reversal.
- A reversal can be reported against an invoice already Refunded or Partially refunded by Nadia's own prior action, not only one still plainly Paid -- this automation's Paid-family check (step 4) covers all three, since a processor-side dispute can surface after Nadia has already recorded her own refund.
- XBR-22: The Disputed status and the underlying Payment's Reversed status feed Financial Dashboard (FEAT-12) and Accounting Export (FEAT-22) totals once committed.

## Edge Cases

- **A reversal notice arrives for an invoice with no Payment ever recorded as Succeeded** -- Discarded per the Notice discarded outcome; there is no confirmed payment for a reversal to apply against, so no write occurs and no notification fires.
- **A reversal notice arrives for an invoice paid off-platform (status Paid (recorded by freelancer), Payment Recorded manually)** -- Discarded at step 4 per the Notice discarded outcome, since no processor-confirmed payment exists; no write, no notification.
- **Concurrent trigger firing -- two reversal notices for the same invoice arrive at effectively the same time (e.g., a duplicate delivery from the relaying integration)** -- The first to commit sets the Invoice to Disputed and the Payment to Reversed; the second finds the invoice already Disputed and the Payment already Reversed at step 3 and ends with the Duplicate / already-Disputed no-op outcome (no write, no second notification, no second trail entry).
- **A trigger fires while a previous run for the same invoice is still in flight** -- The second run for the same invoice cannot proceed to write until the first completes; once the first commits, the second's read in step 2 sees the already-Disputed/Reversed state and ends with the Duplicate / already-Disputed no-op outcome at step 3.
- **A reversal notice is delivered again long after it was applied (a later retry or redelivery)** -- Same Duplicate / already-Disputed no-op outcome; the invoice's Disputed status and Reversed payment are unchanged and Nadia is not emailed again.
- **Nadia submits a manual refund (FEAT-25.SPEC-003) for this exact invoice at the same moment this automation's reversal commits** -- Whichever commits first wins; if the reversal commits first, FEAT-25.SPEC-003's own commit-time check finds the invoice already Disputed (not Paid) and rejects the refund submission. If the refund commits first, this automation's step 4 still finds the invoice in a Paid-family status (Refunded or Partially refunded) and proceeds to Disputed, since a processor-confirmed reversal is authoritative over the prior manual entry.
- **The invoice was removed via FEAT-24 account deletion before the reversal notice arrives** -- FEAT-32.SPEC-002 discards the notice at its own correlation step, since it cannot be matched to a record that no longer exists; this automation never receives it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Triggered by (inbound) | The "Reversal or chargeback reported" inbound event fires this automation with an already-correlated invoice reference |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the resulting Disputed status alongside the preserved prior record |
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | References (outbound) | An invoice this automation sets to Disputed is no longer eligible for FEAT-25.SPEC-001's refund entry |
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | References (outbound) | A concurrent manual refund submission for the same invoice is superseded by this automation's reversal, per processor-authoritative-over-manual |
| FEAT-25.SPEC-008 (Payment Reversal Notification) | Triggers (outbound) | A recorded reversal fires Nadia's notification email immediately |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded reversal writes the append-only trail entry (XBR-05) |
| FEAT-12 (Freelancer Financial Dashboard) | Affects (outbound) | The Disputed status and Reversed payment are reflected in dashboard totals (XBR-22) |
| FEAT-22 (Accounting Export) | Affects (outbound) | The Disputed status and Reversed payment are reflected in export totals (XBR-22) |

## Analytics and Success Signals

- **payment_reversal_recorded** (invoice reference, prior status: paid / refunded / partially_refunded) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so reversal activity is observable rather than invisible.
- **payment_reversal_notice_discarded** (reason: no_succeeded_payment) -- N/A -- no success-metrics.md metric is connected to this feature; retained so a discarded notice is diagnostically visible rather than silently dropped.
- **payment_reversal_duplicate_ignored** (invoice reference, report timestamp) -- N/A -- no success-metrics.md metric is connected to this feature; retained so duplicate deliveries from the relaying integration are diagnostically visible while sending no second email.

## Acceptance Criteria

**FEAT-25.SPEC-005-AC-01:** Given a Paid invoice with a Succeeded Payment, when the payment-processing capability reports a reversal, then the Invoice's status is set to Disputed and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-005-AC-02:** Given a reversal is recorded, when the commit completes, then the underlying Payment's status is set to Reversed.

**FEAT-25.SPEC-005-AC-03:** Given a reversal is recorded, when the commit completes, then FEAT-25.SPEC-008 is triggered to email Nadia immediately and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-005-AC-04:** Given an invoice already Refunded or Partially refunded by Nadia's own prior action, when the payment-processing capability reports a reversal against it, then the Invoice's status is still set to Disputed, since the Paid-family check covers all three prior statuses.

**FEAT-25.SPEC-005-AC-05:** Given a reversal notice arrives for an invoice with no Payment ever recorded as Succeeded, when this automation checks the Payment's status, then no write occurs and no notification fires.

**FEAT-25.SPEC-005-AC-06:** Given two reversal notices for the same invoice arrive at effectively the same time, when the first commits, then the second finds the invoice already Disputed and the Payment already Reversed at step 3, ends with the "Duplicate / already-Disputed no-op" outcome, and applies no further change and sends no second email.

**FEAT-25.SPEC-005-AC-07:** Given Nadia submits a manual refund for an invoice at the same moment a reversal for that invoice commits, when the reversal commits first, then FEAT-25.SPEC-003 finds the invoice already Disputed and rejects the refund submission.

**FEAT-25.SPEC-005-AC-08:** Given Nadia's manual refund commits first for an invoice a moment before a reversal notice for that same invoice arrives, when this automation processes the reversal, then it still finds the invoice in a Paid-family status (Refunded or Partially refunded) and sets it to Disputed.

**FEAT-25.SPEC-005-AC-09:** Given a reversal is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the Disputed status and the Reversed payment.

**FEAT-25.SPEC-005-AC-10:** Given persisting the Invoice and Payment changes fails after the Paid-family check passes, when the failure occurs, then no partial write is left behind and the event is retried by FEAT-32.SPEC-002's own retry contract.

**FEAT-25.SPEC-005-AC-11:** Given an invoice already Disputed with its Payment already Reversed, when a further reversal notice for that invoice arrives at any later time, then the outcome is "Duplicate / already-Disputed no-op": no write occurs, FEAT-25.SPEC-008 is not triggered (no second email), no trail entry is written, and payment_reversal_duplicate_ignored is emitted, not payment_reversal_notice_discarded.

**FEAT-25.SPEC-005-AC-12:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when a reversal notice for it arrives, then the outcome is "Notice discarded -- no matching payment": no write, no notification, and payment_reversal_notice_discarded is emitted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (recorded, duplicate no-op, discarded, automation failure) | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |

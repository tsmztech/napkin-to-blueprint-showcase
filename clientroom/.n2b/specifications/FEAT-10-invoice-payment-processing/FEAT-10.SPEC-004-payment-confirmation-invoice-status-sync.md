---
document_type: spec
spec_type: automation
spec_id: FEAT-10.SPEC-004
spec_name: Payment Confirmation & Invoice Status Sync
spec_slug: payment-confirmation-invoice-status-sync
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

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

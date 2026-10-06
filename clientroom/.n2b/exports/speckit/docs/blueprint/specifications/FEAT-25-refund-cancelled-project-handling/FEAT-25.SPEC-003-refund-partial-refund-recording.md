---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-003
spec_name: Refund & Partial Refund Recording
spec_slug: refund-partial-refund-recording
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Refund & Partial Refund Recording

## Overview

**Name:** Refund & Partial Refund Recording
**ID:** FEAT-25.SPEC-003
**Type:** Automation
**Purpose:** Validates and persists Nadia's refund entry -- full or partial, never exceeding the amount paid -- sets the invoice to Refunded or Partially refunded, and preserves the original Paid record rather than overwriting it.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Re-validating the submitted refund amount against the amount paid at the moment of commit (not merely at form load)
- Setting the Invoice's `status` to Refunded (full amount) or Partially refunded (less than the full amount)
- Recording the refunded amount against the Payment record
- Preserving the Invoice's and Payment's prior paid record as the record that remains visible alongside the new status. The prior paid record is one of two refund-eligible combinations (FEAT-25.SPEC-006, Refund eligibility rule): Invoice `status` Paid with Payment `status` Succeeded, or Invoice `status` Paid (recorded by freelancer) with Payment `status` Recorded manually
- Notifying Owen once the refund is recorded (via FEAT-25.SPEC-007)

**Non-Goals:**
- Issuing the refund itself through a payment-processing capability -- excluded per scope-boundaries.md (SC-18): the freelancer has already issued the refund through her own processor account before reaching this automation; no money movement is initiated here
- Collecting the amount and reason from Nadia -- owned entirely by FEAT-25.SPEC-001 (Mark Invoice Refunded Screen); this automation begins where that screen's submission ends
- Applying a processor-reported reversal or chargeback -- owned entirely by FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording); this automation handles only Nadia's own manually entered refund, never an inbound processor event
- Writing the append-only activity trail entry -- owned entirely by Immutable Activity & Audit Trail (FEAT-13, XBR-05); this automation's role ends at the Invoice and Payment updates and the outbound trigger to FEAT-13

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Refund submitted | FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Fires when Nadia taps Mark Refunded on a form that passed FEAT-25.SPEC-006's field-level validation, for an invoice that was refund-eligible when the screen loaded (Paid with a Succeeded Payment, or Paid (recorded by freelancer) with a Recorded manually Payment) | Invoice reference, refunded amount, currency, optional reason |

## Processing Logic

1. Receive the invoice reference, refunded amount, and optional reason from the triggering screen.
2. Re-read the Invoice's current `status` and the Payment's current `status` and `amount` at the moment of commit (not the values the screen loaded with).
3. Confirm the invoice is still refund-eligible, meaning exactly one of these two combinations holds: (a) Invoice `status` is Paid and Payment `status` is Succeeded; (b) Invoice `status` is Paid (recorded by freelancer) and Payment `status` is Recorded manually. If neither holds (the invoice is already Refunded, Partially refunded, or Disputed by an intervening reversal, or the Payment `status` does not match its invoice status), stop and report a stale-state outcome.
4. Confirm the refunded amount is greater than zero and does not exceed the Payment's `amount` (FEAT-25.SPEC-006, XBR-20). If it exceeds the amount paid, stop and report a validation-failure outcome (this should already have been caught by the screen's own field validation; this step re-checks authoritatively at commit).
5. Compare the refunded amount to the Payment's `amount`:
   - If equal, set the Invoice's `status` to Refunded.
   - If less, set the Invoice's `status` to Partially refunded.
6. Record the refunded amount and the optional reason against the Payment record, alongside its existing record -- the Payment's prior paid amount, `paid_at`, and `status` (Succeeded, or Recorded manually) are never overwritten; the Payment `status` is left unchanged by a refund.
7. Persist the Invoice and Payment changes together as one committed outcome.
8. Trigger FEAT-25.SPEC-007 (Refund & Cancellation Notification) to email Owen.
9. Signal FEAT-13 (Immutable Activity & Audit Trail) to write the append-only trail entry (XBR-05).
10. Return the new status to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Full refund recorded | Refunded amount equals the amount paid | Invoice `status` set to Refunded; Payment's refunded amount recorded; prior paid record preserved (Paid/Succeeded, or Paid (recorded by freelancer)/Recorded manually) | Toast "Invoice marked Refunded"; FEAT-09.SPEC-002 shows Refunded alongside the preserved paid record | FEAT-25.SPEC-001, FEAT-09.SPEC-002, FEAT-25.SPEC-007, FEAT-13 |
| Partial refund recorded | Refunded amount is less than the amount paid, greater than zero | Invoice `status` set to Partially refunded; Payment's refunded amount recorded; prior paid record preserved (Paid/Succeeded, or Paid (recorded by freelancer)/Recorded manually) | Toast "Invoice marked Partially refunded"; FEAT-09.SPEC-002 shows Partially refunded alongside the preserved paid record | FEAT-25.SPEC-001, FEAT-09.SPEC-002, FEAT-25.SPEC-007, FEAT-13 |
| Stale-state rejection | At commit time the invoice is no longer in either refund-eligible combination (Paid + Succeeded, or Paid (recorded by freelancer) + Recorded manually) -- already refunded, partially refunded, or disputed, or the Payment `status` does not match | None -- no write occurs | FEAT-25.SPEC-001 shows "This invoice's status changed since you opened this page. Refresh to see the latest state." | FEAT-25.SPEC-001 |
| Validation failure at commit | The refunded amount exceeds the amount paid, or is zero/negative, when re-checked at commit | None -- no write occurs | FEAT-25.SPEC-001 shows the exact error message defined by FEAT-25.SPEC-006 | FEAT-25.SPEC-001 |
| Automation failure | Persisting the Invoice/Payment changes fails after validation passes (e.g., a transient write failure) | No partial write -- the Invoice and Payment are committed together or not at all | FEAT-25.SPEC-001 shows "Couldn't record this refund. Try again." with a Retry button; the invoice's prior confirmed status (Paid, or Paid (recorded by freelancer)) remains authoritative | FEAT-25.SPEC-001 |

## Data Model

**Reads:** Invoice -- `status` (re-checked at commit). Payment -- `amount`, `status` (re-checked at commit).
**Creates:** None.
**Updates:** Invoice -- `status` (Paid or Paid (recorded by freelancer) -> Refunded or Partially refunded). Payment -- refunded amount and the optional reason are recorded against the record, alongside its existing `amount`, `paid_at`, and `status` (Succeeded, or Recorded manually), which are never overwritten.
**Deletes:** None -- the prior paid record is preserved, never replaced (XBR-04).

## Business Rules

- XBR-20: A refund cannot exceed the amount paid; a refunded invoice cannot later be marked Paid again without a logged correction (that correction path belongs to FEAT-09/FEAT-10, not this automation).
- XBR-04: The original Paid record is never silently altered -- the refund is a new, logged transition displayed alongside it, never a replacement of it.
- XBR-22: The refunded amount feeds Financial Dashboard (FEAT-12) and Accounting Export (FEAT-22) totals once committed.
- Validation at commit is authoritative over validation at form load -- the screen's own field-level check (FEAT-25.SPEC-006) is a first pass for user feedback; this automation's step 3-4 re-check (eligible combination, then amount) is what actually gates the write.
- Refund eligibility (FEAT-25.SPEC-006): an invoice paid off-platform (Invoice `status` Paid (recorded by freelancer), Payment `status` Recorded manually via FEAT-10.SPEC-005) is refundable through this automation exactly like a platform-processed payment (Paid + Succeeded); the refund is recorded, never issued, so no processor is involved. Any other invoice status or Payment status pairing is refused as stale-state.
- A refund submitted against an invoice that a reversal has, in the same moment, set to Disputed is refused: processor-confirmed reversal status is authoritative over a concurrent manual refund entry (dependency map, Invoice Contention).

## Edge Cases

- **Two refund submissions for the same invoice arrive from two open sessions of Nadia's at effectively the same time** -- The first to commit sets the Invoice's `status` away from Paid; the second re-checks at step 3, finds the status no longer Paid, and is rejected with the stale-state outcome. Only one refund is ever recorded per invoice.
- **A trigger fires while a previous run for the same invoice is still in flight** -- FEAT-25.SPEC-001 disables Mark Refunded during submission, so a second run for the same invoice cannot start from the same screen instance; a second session's independent submission is handled by the concurrent-firing case above.
- **The refunded amount is exactly equal to the amount paid, entered through the "Partial amount" option** -- Recorded as Refunded, not Partially refunded -- the resulting status reflects the amount, never which form option produced it.
- **A reversal (FEAT-25.SPEC-005) commits for this invoice a moment before this automation's own commit** -- Step 3 finds the Invoice's status is now Disputed, not Paid, and the refund is rejected with the stale-state outcome; the invoice remains Disputed, never simultaneously Refunded.
- **Persisting the Invoice and Payment changes partially fails (one write succeeds, the other does not)** -- The two updates are committed as a single outcome; if either cannot be persisted, neither is applied, and the Invoice remains Paid (or Paid (recorded by freelancer)) until a successful retry.
- **The invoice is Paid (recorded by freelancer) and its Payment is Recorded manually** -- Eligible; step 3 combination (b) passes and the refund proceeds through steps 4-10 unchanged, with the Payment `status` left as Recorded manually.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Triggered by (inbound) | Fires on a validated Mark Refunded submission |
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | Affects (outbound) | Returns the new status or a rejection outcome to the screen |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the resulting Refunded/Partially refunded status alongside the preserved Paid record |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Supplies the refund-amount ceiling and the stale-state/reject-with-refresh behavior this automation enforces at commit |
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | References (inbound) | A reversal committed for the same invoice takes precedence over this automation's own commit |
| FEAT-25.SPEC-007 (Refund & Cancellation Notification) | Triggers (outbound) | A recorded refund fires Owen's notification email |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | A recorded refund writes the append-only trail entry (XBR-05) |
| FEAT-12 (Freelancer Financial Dashboard) | Affects (outbound) | The refunded amount is reflected in dashboard totals (XBR-22) |
| FEAT-22 (Accounting Export) | Affects (outbound) | The refunded amount is reflected in export totals (XBR-22) |

## Analytics and Success Signals

- **invoice_marked_refunded** (invoice reference, amount, currency) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so refund activity is observable rather than invisible.
- **partial_refund_recorded** (invoice reference, refunded amount, amount paid, currency) -- N/A -- no success-metrics.md metric is connected to this feature; retained per product-features.md's own Signals field so partial-refund activity is observable rather than invisible.
- **refund_recording_rejected** (reason: stale_state / validation_failure / automation_failure) -- N/A -- no success-metrics.md metric is connected to this feature; retained so refund friction and concurrency collisions are observable rather than silent.

## Acceptance Criteria

**FEAT-25.SPEC-003-AC-01:** Given Nadia submits a refund equal to the full amount paid on a Paid invoice, when this automation processes it, then the Invoice's status is set to Refunded and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-003-AC-02:** Given Nadia submits a refund less than the full amount paid, when this automation processes it, then the Invoice's status is set to Partially refunded and the refunded amount is recorded against the Payment.

**FEAT-25.SPEC-003-AC-03:** Given the invoice is no longer in a refund-eligible combination at the moment this automation commits (it changed since the screen was loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

**FEAT-25.SPEC-003-AC-04:** Given a refund amount that exceeds the amount paid somehow reaches this automation's commit-time check, when the check runs, then the write is refused and FEAT-25.SPEC-001 shows FEAT-25.SPEC-006's exact error message.

**FEAT-25.SPEC-003-AC-05:** Given a refund is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-003-AC-06:** Given persisting the Invoice and Payment changes fails after validation passes, when the failure occurs, then FEAT-25.SPEC-001 shows "Couldn't record this refund. Try again." and the Invoice's status remains as it was (Paid, or Paid (recorded by freelancer)).

**FEAT-25.SPEC-003-AC-07:** Given two refund submissions for the same invoice arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the invoice no longer refund-eligible and is rejected.

**FEAT-25.SPEC-003-AC-08:** Given a reversal is recorded for this exact invoice a moment before this automation's own commit, when the commit-time check runs, then it finds the Invoice already Disputed and rejects the refund.

**FEAT-25.SPEC-003-AC-09:** Given a refund amount exactly equal to the amount paid was entered through the "Partial amount" option, when this automation processes it, then the resulting status is Refunded, not Partially refunded.

**FEAT-25.SPEC-003-AC-10:** Given a refund is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the refunded amount.

**FEAT-25.SPEC-003-AC-11:** Given an optional reason accompanies the refund submission, when this automation commits, then the reason is recorded against the Payment alongside the refunded amount.

**FEAT-25.SPEC-003-AC-12:** Given no reason accompanies the refund submission, when this automation commits, then the refund is still recorded with no reason stored.

**FEAT-25.SPEC-003-AC-13:** Given an invoice with status Paid (recorded by freelancer) whose Payment `status` is Recorded manually, when Nadia submits a refund equal to the amount paid, then the Invoice's status is set to Refunded, the Payment `status` remains Recorded manually, and the prior paid record remains visible alongside the new status.

**FEAT-25.SPEC-003-AC-14:** Given an invoice whose status is Paid but whose Payment `status` is neither Succeeded nor a match for that invoice status (or whose status is Generated, Sent, Payment pending, Overdue, or Corrected), when a refund submission reaches the commit-time check, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (full refund, partial refund, stale-state, validation failure, automation failure) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

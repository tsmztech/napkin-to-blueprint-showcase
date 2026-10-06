---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-005
spec_name: Bank-Transfer-Pending Reminder Pause
spec_slug: bank-transfer-pending-reminder-pause
parent_feature: FEAT-11
parent_feature_name: Automated Payment Reminders
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Bank-Transfer-Pending Reminder Pause

## Overview

**Name:** Bank-Transfer-Pending Reminder Pause
**ID:** FEAT-11.SPEC-005
**Type:** Automation
**Purpose:** Automatically pauses an invoice's reminder schedule while a bank-transfer payment is pending, and resumes it if the pending transfer resolves or reverts, so a reminder never chases a payment that is already on its way.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Setting `pause_state` to Paused while bank transfer pending the moment an invoice enters Payment pending status
- Returning `pause_state` to Active when the pending transfer resolves (invoice reaches Paid) or reverts (invoice returns to Overdue/unpaid), unless a freelancer pause is already in effect
- Respecting the precedence rule that a freelancer-initiated pause is never overwritten by this automation, in either direction

**Non-Goals:**
- Deciding what "may send now" means once `pause_state` is set -- owned entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this automation only writes the `pause_state` value, it does not interpret it.
- Detecting or reporting the bank-transfer payment's own status (initiated, pending, succeeded, failed) -- owned by FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync), which this automation only listens to.
- Nadia's own pause/resume action -- owned by FEAT-11.SPEC-003 (Invoice Reminder Panel); this automation never triggers from a freelancer action, only from a payment-status event.
- Card payments -- card payments resolve near-instantly (per FEAT-10.SPEC-003) and never produce a Payment pending status; this automation's trigger fires only for the bank-transfer method, per the dependency map's Invoice status lifecycle.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invoice enters Payment pending status | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires when the payment-processing capability reports a bank-transfer payment as pending confirmation for this invoice | Invoice reference, new status (Payment pending), payment method (bank transfer) |
| Invoice's pending bank transfer resolves or reverts | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires when the payment-processing capability reports the pending transfer as succeeded (invoice reaches Paid) or failed (invoice reverts to its prior unpaid/Overdue status) | Invoice reference, new status (Paid or reverted), payment method (bank transfer) |

## Processing Logic

1. On a Payment pending event for an invoice: read the invoice's current Reminder Log `pause_state`.
2. If `pause_state` is Active, set it to Paused while bank transfer pending.
3. If `pause_state` is already Paused by freelancer, leave it unchanged -- a freelancer-initiated pause is never overwritten by this automation (FEAT-11.SPEC-002's Pause-reason precedence rule).
4. If `pause_state` is already Paused while bank transfer pending (a redundant event, e.g. a duplicate status report), leave it unchanged and take no further action.
5. On a resolve-or-revert event for an invoice: read the invoice's current `pause_state`.
6. If `pause_state` is Paused while bank transfer pending, set it back to Active.
7. If `pause_state` is Paused by freelancer, leave it unchanged -- the freelancer's own pause persists regardless of the payment outcome.
8. If `pause_state` is already Active (a redundant event), take no action.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Paused for pending transfer | Payment pending event arrives while `pause_state` is Active | `pause_state` set to Paused while bank transfer pending | The pause indicator on FEAT-11.SPEC-003 shows "Paused -- bank transfer pending" on next view | FEAT-11.SPEC-002 (Reminder Eligibility Rule), FEAT-11.SPEC-003 (Invoice Reminder Panel) |
| Resumed after resolution | Resolve/revert event arrives while `pause_state` is Paused while bank transfer pending | `pause_state` set to Active | The pause indicator shows "Active" on next view; the invoice becomes eligible for its next due reminder threshold again | FEAT-11.SPEC-002, FEAT-11.SPEC-003 |
| No-action -- freelancer pause preserved | Either event arrives while `pause_state` is Paused by freelancer | None | None -- the indicator continues to show "Paused by you" | -- |
| No-action -- redundant event | Either event arrives while `pause_state` already matches the event's target value | None | None | -- |
| Automation failure | The write to `pause_state` fails (e.g., a transient data-write error) | No change persists | No user-visible interruption; the write is retried automatically | FEAT-13.SPEC-003 (Activity Entry Recording), for logging the retry outcome |

## Data Model

**Reads:** Invoice -- current status, from the Payment pending / resolve-revert event data (FEAT-10.SPEC-004). Reminder Log -- current `pause_state` for the invoice.
**Creates:** None -- this automation never creates a Reminder Log entry, only updates the schedule's live pause state.
**Updates:** Reminder Log -- `pause_state`, set to or cleared from Paused while bank transfer pending.
**Deletes:** None.

## Business Rules

- XBR-15: reminders pause while a bank transfer is pending, per this automation, and resume when it resolves or reverts.
- FEAT-11.SPEC-002's Pause-reason precedence rule governs every write here: this automation may only move `pause_state` between Active and Paused while bank transfer pending; it never sets or clears Paused by freelancer.
- This automation's writes are idempotent per invoice: re-applying the same event twice (e.g., a duplicate status report) never changes an already-correct `pause_state`.
- Processor-reported status is authoritative (dependency map, Invoice Contention note); this automation trusts FEAT-10.SPEC-004's event data without re-verifying the payment status itself.

## Edge Cases

- **Payment pending and resolve/revert events for the same invoice arrive out of order (e.g., a resolve event is processed before its own pending event due to delivery timing)** -- The automation trusts the most recently reported event as authoritative, consistent with FEAT-10.SPEC-004 applying processor-reported status in order; if a resolve event is somehow processed first, the subsequent pending event's step 3/4 check finds `pause_state` already Active or already Paused while bank transfer pending, per the current state, and behaves per the No-action rules rather than producing an inconsistent result.
- **Nadia pauses the invoice herself (FEAT-11.SPEC-003) between the Payment pending event and the resolve event** -- Per the precedence rule, her pause takes priority: the resolve event finds `pause_state` is Paused by freelancer and leaves it unchanged, so the invoice remains paused after the resolve event until Nadia resumes it herself.
- **Concurrent trigger firing (a Payment pending event and a resolve event for the same invoice arrive at effectively the same time, e.g. a very short-lived pending window)** -- Whichever event's write commits first determines the resulting `pause_state`; the second event's read-then-write sees the state left by the first and applies its own step 2-8 logic against that current value, never against a stale read from before the first event committed.
- **A trigger fires for an invoice while a previous run (for the same invoice) is still in flight** -- Writes to `pause_state` for a given invoice are serialized: a second event for the same invoice waits for the in-flight write to commit before reading and applying its own logic, so no write is lost to a race between two events for the same invoice.
- **The pending bank transfer resolves after the invoice has already passed day 10 unpaused** -- If the transfer resolves to Paid, no further reminder is relevant regardless of pause history. If it reverts (fails), `pause_state` returns to Active, but per FEAT-11.SPEC-001, no automatic reminder exists past day 10 -- the invoice's only remaining reminder path is Nadia's manual send (FEAT-11.SPEC-003), now eligible again since `pause_state` is Active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Triggered by (inbound) | Both the Payment pending event and the resolve/revert event originate here |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | Affects (outbound) | Every write to `pause_state` changes what that rule evaluates as eligible |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Affects (outbound) | The pause-state indicator reflects this automation's writes |
| FEAT-11.SPEC-001 (Reminder Schedule) | Affects (outbound) | A pause set here can cause the schedule's next eligibility re-check to skip a send |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Automatic pause/resume transitions are logged as part of the invoice's reminder trail |

## Analytics and Success Signals

- **reminder_auto_paused_bank_transfer** (invoice reference) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_resumed_bank_transfer** (invoice reference, resolution: paid / reverted) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_pause_write_failed** (retry_count) -- N/A -- no Stage 2 metric measures this automation's internal write failures; retained so a silently failed pause/resume is observable rather than invisible, and surfaced through FEAT-13's trail rather than a dedicated success metric.

## Acceptance Criteria

**FEAT-11.SPEC-005-AC-01:** Given Nadia's invoice to Owen has an Active reminder schedule, when a bank-transfer payment for that invoice enters Payment pending, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-005-AC-02:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer resolves to Paid, then `pause_state` returns to Active.

**FEAT-11.SPEC-005-AC-03:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer fails and the invoice reverts to unpaid, then `pause_state` returns to Active and the invoice becomes eligible for its next due reminder.

**FEAT-11.SPEC-005-AC-04:** Given Nadia has already paused the invoice herself (Paused by freelancer), when a bank-transfer payment for it enters Payment pending, then `pause_state` remains Paused by freelancer, unchanged.

**FEAT-11.SPEC-005-AC-05:** Given the invoice's `pause_state` is Paused by freelancer, when the pending transfer that triggered no automatic pause resolves, then `pause_state` remains Paused by freelancer -- the resolve event never touches it.

**FEAT-11.SPEC-005-AC-06:** Given the invoice's `pause_state` is already Paused while bank transfer pending, when a duplicate Payment pending event is reported for the same invoice, then `pause_state` remains unchanged and no error occurs.

**FEAT-11.SPEC-005-AC-07:** Given the invoice's `pause_state` is already Active, when a duplicate resolve event is reported, then `pause_state` remains unchanged.

**FEAT-11.SPEC-005-AC-08:** Given a Payment pending event and a resolve event for the same invoice arrive at effectively the same time, when both are processed, then the resulting `pause_state` reflects whichever event's write committed first, with the second event's logic applied against that resulting state rather than a stale read.

**FEAT-11.SPEC-005-AC-09:** Given the write to `pause_state` fails on the first attempt, when the automation retries and the retry succeeds, then `pause_state` reflects the correct value with no user-visible interruption.

**FEAT-11.SPEC-005-AC-10:** Given a pending bank transfer for an invoice already past day 10 reverts to unpaid, when `pause_state` returns to Active, then no automatic reminder is scheduled (per FEAT-11.SPEC-001's day-10 ceiling), and only Nadia's manual send remains available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (Payment pending, resolve/revert) | 2 |
| Outcome Paths | 5 (paused, resumed, freelancer-pause preserved, redundant no-action, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

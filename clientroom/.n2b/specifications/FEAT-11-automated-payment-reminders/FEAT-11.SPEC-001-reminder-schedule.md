---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-001
spec_name: Reminder Schedule
spec_slug: reminder-schedule
parent_feature: FEAT-11
parent_feature_name: Automated Payment Reminders
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Reminder Schedule

## Overview

**Name:** Reminder Schedule
**ID:** FEAT-11.SPEC-001
**Type:** Automation
**Purpose:** Automatically sends the day-3 and day-10 overdue reminder email for an unpaid invoice, re-checking eligibility immediately before each send so a reminder never goes out to an invoice that has since been paid or paused.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Detecting when an unpaid invoice's due date has reached day 3 and day 10, counted in the freelancer's time zone
- Re-checking eligibility (paid, freelancer-paused, bank-transfer-pending) immediately before every send, per FEAT-11.SPEC-002
- Creating the day-3 and day-10 Reminder Log entries and handing each off to the Overdue Reminder Email
- Retrying a failed send and logging the failure rather than dropping it silently
- Stopping automatic sends after day 10 -- no further automatic reminder exists for an invoice

**Non-Goals:**
- Determining *whether* a reminder is allowed to send right now (paid/pause/pending checks, rate limits) -- owned entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this automation calls that rule rather than re-implementing its checks.
- Nadia's manual "Send Reminder Now" action -- handled by FEAT-11.SPEC-003 (Invoice Reminder Panel); this automation covers only the two automatic (day-3, day-10) trigger points.
- Pausing or resuming the schedule around a pending bank-transfer payment -- a distinct cross-feature trigger owned by FEAT-11.SPEC-005 (Bank-Transfer-Pending Reminder Pause).
- A configurable reminder cadence, additional reminder points beyond day 3 and day 10, or a workflow builder -- excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior instead of a configurable automation builder.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An unpaid invoice's due date reaches 3 full elapsed days | System (schedule-based; anchored to the invoice's `due_date` from FEAT-09.SPEC-007 and evaluated against the freelancer's stored time zone per FEAT-15.SPEC-006) | Fires once per invoice, the first time the elapsed-day count (in the freelancer's local calendar day) reaches exactly 3, and only if no `day 3` Reminder Log entry already exists for this invoice | Invoice reference, `due_date`, current invoice status, freelancer's time zone, invoice amount/currency, the invoice's Primary Contact reference |
| An unpaid invoice's due date reaches 10 full elapsed days | System (schedule-based; same anchoring as the day-3 trigger) | Fires once per invoice, the first time the elapsed-day count reaches exactly 10, and only if no `day 10` Reminder Log entry already exists for this invoice | Same as above |

## Processing Logic

1. On each schedule evaluation, read every invoice whose current status is not Paid, Refunded, Partially refunded, or Corrected (i.e., an invoice that can still be overdue).
2. For each such invoice, compute the number of full calendar days elapsed since `due_date`, using the freelancer's stored time zone (FEAT-15.SPEC-006) rather than a fixed offset.
3. If the elapsed count is exactly 3 and no `day 3` Reminder Log entry exists for this invoice, mark this invoice as a day-3 candidate. If the elapsed count is exactly 10 and no `day 10` Reminder Log entry exists, mark it as a day-10 candidate. An invoice with neither condition produces no action.
4. For each candidate, immediately before sending, invoke the Reminder Eligibility Rule (FEAT-11.SPEC-002) against the invoice's current state (not the state read in step 1 -- the state at this exact moment).
5. If the eligibility check returns eligible: create a Reminder Log entry (`invoice`, `reminder_type` = `day 3` or `day 10`, `scheduled_for` = the computed due-date offset, `pause_state` = Active) and hand the entry off to the Overdue Reminder Email (FEAT-11.SPEC-004) for sending. On successful hand-off, set the entry's `sent_at` to the current time.
6. If the eligibility check returns not eligible (paid, paused by freelancer, or paused while a bank transfer is pending): take no action for this candidate. No Reminder Log entry is created for a skipped candidate -- there is nothing to log, since no reminder was scheduled or sent.
7. If the hand-off to the Overdue Reminder Email fails (the send itself fails, not the eligibility check), retry automatically; see Outcome Definitions for the failure path.
8. After the day-10 reminder has been sent (or skipped as ineligible), no further automatic trigger evaluates this invoice -- day 3 and day 10 are the only two automatic reminder points, per XBR-15.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Day-3 reminder sent | Elapsed days = 3, no prior day-3 entry, eligibility check passes | Reminder Log entry created with `reminder_type` = `day 3`, `sent_at` set | Reminder appears in the invoice's reminder history; Owen receives the email | FEAT-11.SPEC-003 (Invoice Reminder Panel), FEAT-11.SPEC-004 (Overdue Reminder Email), FEAT-13.SPEC-003 (Activity Entry Recording) |
| Day-10 reminder sent | Elapsed days = 10, no prior day-10 entry, eligibility check passes | Reminder Log entry created with `reminder_type` = `day 10`, `sent_at` set | Reminder appears in the invoice's reminder history; Owen receives the email | FEAT-11.SPEC-003, FEAT-11.SPEC-004, FEAT-13.SPEC-003 |
| Send skipped -- not eligible | Elapsed days = 3 or 10, but the invoice is Paid, or its `pause_state` is Paused by freelancer or Paused while bank transfer pending, at the eligibility re-check | No Reminder Log entry created | Nothing new appears in the reminder history; the invoice's current pause indicator (FEAT-11.SPEC-003) already reflects why | FEAT-11.SPEC-002 (Reminder Eligibility Rule), FEAT-11.SPEC-003 |
| No action -- day not yet reached | Elapsed days is not 3 and not 10 | None | None | -- |
| Send failed | Eligibility check passed but the hand-off to the Overdue Reminder Email fails (the email delivery capability reports a failure at send time, per FEAT-14.SPEC-001) | Reminder Log entry is created in a not-yet-sent state; retried per the retry rule below | No user-visible interruption; if retries exhaust, a delivery warning appears to Nadia on the affected project (XBR-30) | FEAT-11.SPEC-004, FEAT-13.SPEC-003 |

## Data Model

**Reads:** Invoice -- `due_date`, `status` (to determine candidacy and to feed the eligibility re-check, sourced from FEAT-09 and FEAT-10). Freelancer Account -- `time_zone` (FEAT-15), used to count elapsed days in local calendar days rather than a fixed offset. Reminder Log -- existing entries for the invoice, to enforce the once-per-threshold dedup check.
**Creates:** Reminder Log entry -- `invoice`, `reminder_type` (`day 3` or `day 10`), `scheduled_for`, `sent_at`, `pause_state` (copied from the invoice's current schedule state at creation).
**Updates:** Reminder Log entry -- `sent_at` and delivery status once the hand-off to FEAT-11.SPEC-004 completes or fails.
**Deletes:** None -- this automation never removes a Reminder Log entry (dependency map: Reminder Log deletion belongs to FEAT-24 only).

## Business Rules

- XBR-15: overdue reminders go out at day 3 and day 10 after the due date, counted in the freelancer's time zone; no automatic reminder exists past day 10.
- Eligibility is re-checked immediately before every send, not at the moment the day-3/day-10 threshold is first detected -- the two moments can differ if the schedule evaluation is delayed, and the invoice's state may have changed in between.
- A day-3 or day-10 Reminder Log entry is created only on an eligible send; a skipped candidate leaves no entry, since Reminder Log records reminders that were scheduled and sent, not reminders considered and declined.
- The dependency map's Contention note for Reminder Log applies: "a pause saved first wins" -- if Nadia's pause action (FEAT-11.SPEC-003) is saved before this automation's eligibility re-check runs for that invoice, the send is skipped; if the send has already completed before the pause is saved, the completed send stands.
- This automation is the sole owner of the day-3/day-10 timing decision; FEAT-11.SPEC-003's manual send and FEAT-11.SPEC-005's bank-transfer-pending pause never alter the day-3/day-10 dates themselves, only whether a given send is eligible.

## Edge Cases

- **Invoice is paid in the instant between candidate detection and the eligibility re-check** -- The eligibility re-check (step 4) reads the invoice's current state, catches the payment, and the send is skipped. This is the exact purpose of re-checking immediately before send rather than relying on the state read during candidate detection.
- **Due date falls on a time-zone transition (e.g., a daylight-saving change) in the freelancer's local calendar** -- Elapsed-day counting uses the freelancer's local calendar date boundaries as currently in effect; a transition day is still counted as exactly one elapsed day, consistent with how any other calendar day is counted.
- **The schedule evaluation itself is delayed or missed for a period (e.g., the automation was unavailable) and later catches up** -- On the next evaluation, an invoice already past day 3 or day 10 without a corresponding Reminder Log entry is still treated as a candidate for that threshold; a reminder that is now, in effect, being sent late is preferable to one silently skipped. It is not sent for both day 3 and day 10 simultaneously if both are overdue and unsent -- each threshold is evaluated and, if eligible, sent independently, in order (day 3 before day 10 in the same catch-up pass).
- **Invoice becomes ineligible for day 3 but later becomes eligible again before day 10** -- The day-3 opportunity is not retried once its window (exactly 3 elapsed days) has passed; only the day-10 trigger is still pending for that invoice, and it evaluates independently at day 10.
- **Concurrent trigger firing (two schedule evaluations run for the same invoice at effectively the same time)** -- The once-per-threshold dedup check (an existing Reminder Log entry for that `reminder_type`) is authoritative: whichever evaluation completes its eligibility check and entry creation first is the one that sends; the second evaluation's dedup check finds the just-created entry and takes no further action for that threshold.
- **A trigger fires for an invoice while a previous run (for the same invoice) is still in flight (send not yet confirmed)** -- The in-flight run holds the not-yet-sent Reminder Log entry for that threshold; a second concurrent run for the same threshold on the same invoice defers to the dedup check in the same way as concurrent firing above. Runs for different invoices proceed independently and never queue behind each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Supplies the invoice's `due_date`, the anchor for the day-3/day-10 count |
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | References (inbound) | Supplies the freelancer's time zone used to count elapsed days |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | Triggers (outbound) | Every candidate send is gated by this rule's re-check immediately before sending |
| FEAT-11.SPEC-004 (Overdue Reminder Email) | Triggers (outbound) | An eligible day-3 or day-10 candidate is handed off here to compose and send the email |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Affects (outbound) | Every created Reminder Log entry appears in this panel's history |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Every automatic send writes a trail entry (XBR-05) |
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | References (inbound) | The invoice's paid status, read at the eligibility re-check, is set by this spec |

## Analytics and Success Signals

- **reminder_auto_sent** (reminder_type: day_3 / day_10) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_send_skipped** (reminder_type: day_3 / day_10; reason: paid / paused_by_freelancer / paused_bank_transfer_pending) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"
- **reminder_auto_send_failed** (reminder_type: day_3 / day_10; retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-11.SPEC-001-AC-01:** Given Nadia has an invoice for Owen that reaches exactly 3 elapsed days unpaid past its due date (freelancer's time zone), when the schedule evaluates that invoice, then a `day 3` Reminder Log entry is created, the eligibility check passes, and the Overdue Reminder Email is sent to Owen.

**FEAT-11.SPEC-001-AC-02:** Given the same invoice reaches exactly 10 elapsed days still unpaid, when the schedule evaluates it, then a `day 10` Reminder Log entry is created and sent, and no further automatic reminder is ever scheduled for this invoice.

**FEAT-11.SPEC-001-AC-03:** Given an invoice reaches day 3 but Owen paid it one minute before the schedule's eligibility re-check runs, when the re-check executes, then the send is skipped, no Reminder Log entry is created, and no email is sent.

**FEAT-11.SPEC-001-AC-04:** Given an invoice reaches day 3 while Nadia has paused reminders for it, when the schedule evaluates it, then the send is skipped per FEAT-11.SPEC-002 and no Reminder Log entry is created for day 3.

**FEAT-11.SPEC-001-AC-05:** Given an invoice reaches day 10 while it is Paused while bank transfer pending, when the schedule evaluates it, then the send is skipped, and the invoice never receives the day-10 reminder even if the pending transfer later fails and the invoice becomes unpaid again after day 10 has passed.

**FEAT-11.SPEC-001-AC-06:** Given an invoice's elapsed-day count is 5 (between the two thresholds), when the schedule evaluates it, then no reminder action is taken.

**FEAT-11.SPEC-001-AC-07:** Given the Overdue Reminder Email's hand-off fails on the first attempt for a day-3 candidate, when the automation retries per FEAT-14.SPEC-001's retry rule and the retry succeeds, then the Reminder Log entry's `sent_at` is set on the successful attempt and no delivery warning is shown to Nadia.

**FEAT-11.SPEC-001-AC-08:** Given the Overdue Reminder Email's hand-off fails on every retry attempt, when the final retry is exhausted, then the failure is logged rather than dropped silently and Nadia sees a delivery warning on the affected project.

**FEAT-11.SPEC-001-AC-09:** Given the schedule evaluation was unavailable for two days and an invoice passed day 3 unsent during that gap, when the schedule next runs, then the day-3 candidate is still detected (no existing `day 3` entry) and sent, provided the invoice is still eligible at that later moment.

**FEAT-11.SPEC-001-AC-10:** Given two schedule evaluations fire for the same invoice's day-3 threshold at effectively the same time, when both attempt to create the `day 3` entry, then only one Reminder Log entry is created and only one email is sent -- the second evaluation's dedup check finds the existing entry and takes no action.

**FEAT-11.SPEC-001-AC-11:** Given an invoice's day-3 threshold was skipped because Nadia had paused reminders, and she resumes before day 10, when the invoice reaches day 10 elapsed, then the eligibility check passes and the day-10 reminder sends normally -- the earlier skip does not block the later threshold.

**FEAT-11.SPEC-001-AC-12:** Given an invoice is Refunded before ever reaching day 3, when the schedule evaluates invoices, then this invoice is excluded from candidacy entirely, since its status is no longer one that can be overdue.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (day-3 threshold, day-10 threshold) | 2 |
| Outcome Paths | 5 (day-3 sent, day-10 sent, skipped-not-eligible, no-action, send failed) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

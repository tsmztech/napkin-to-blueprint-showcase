---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-11.SPEC-002
spec_name: Reminder Eligibility Rule
spec_slug: reminder-eligibility-rule
parent_feature: FEAT-11
parent_feature_name: Automated Payment Reminders
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Reminder Eligibility Rule

## Overview

**Name:** Reminder Eligibility Rule
**ID:** FEAT-11.SPEC-002
**Type:** Logic/Rule
**Purpose:** Defines whether a reminder -- automatic or manual -- may send right now: the paid check, the pause checks, the time-zone day-counting basis, and the one-manual-reminder-per-day limit, plus every role's authority over pausing, resuming, and manually sending.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders
**Governed Entity:** Reminder Log (including the per-invoice `pause_state` it carries and the derived Overdue status it feeds back to Invoice)

## Scope and Non-Goals

**In Scope:**
- The single "may this reminder send now" check: paid, freelancer-pause, and bank-transfer-pending-pause conditions
- Deriving elapsed overdue days from the invoice's `due_date` using the freelancer's time zone, and deriving the invoice's Overdue status from that count
- The one-manual-reminder-per-invoice-per-day limit
- Authorization for pausing, resuming, sending a manual reminder, and viewing reminder history/state, across every role in the Access Matrix
- Precedence between the two pause reasons (freelancer pause and bank-transfer-pending pause) when both could apply

**Non-Goals:**
- Deciding *when* day-3 and day-10 fall, and creating or sending the automatic reminder itself -- owned by FEAT-11.SPEC-001 (Reminder Schedule), which calls this rule immediately before each send.
- The screen behavior for pausing, resuming, or manually sending (button placement, feedback text, loading states) -- owned by FEAT-11.SPEC-003 (Invoice Reminder Panel), which enforces the rules defined here.
- Setting `pause_state` to Paused while bank transfer pending in the first place, and clearing it on resolution -- owned by FEAT-11.SPEC-005 (Bank-Transfer-Pending Reminder Pause); this spec defines only what the pause_state values *mean* for eligibility, not what sets them.
- A configurable reminder cadence or rate-limit value the freelancer can adjust -- excluded per scope-boundaries.md (SC-11); the one-per-day manual limit is a fixed product decision, not a setting.

## Governed Entity

**Entity:** Reminder Log
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice | reference | The Invoice this log entry belongs to |
| reminder_type | enum (day 3, day 10, manual) | Which reminder point this entry records |
| scheduled_for | date/time (freelancer's time zone) | The computed point at which this reminder was due to fire |
| sent_at | date/time | When the reminder was actually sent; empty until send completes |
| pause_state | enum (Active, Paused by freelancer, Paused while bank transfer pending) | The invoice's current reminder-schedule state, carried on the Reminder Log per the dependency map; tracked once per invoice rather than once per entry, since it describes the schedule's live state, not a historical send |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-001 | Reminder Schedule | Immediately before every automatic day-3/day-10 send |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | On screen load (to show/hide/disable the Pause, Resume, and Send Reminder Now controls) and immediately before a manual send commits |
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | When applying or clearing the bank-transfer-pending pause, to resolve precedence against a freelancer pause |
| FEAT-31.SPEC-002 | Operator Support Session Console | On screen load, to render Dana's read-only mirror of the same panel with every write control disabled |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| invoice | Required; must reference an existing Invoice | Always | On create | -- (system-set at creation, never user-entered) | Yes |
| reminder_type | Required; must be exactly one of day 3, day 10, manual | Always | On create | -- (system-set at creation, never user-entered) | Yes |
| scheduled_for | No validation beyond data type -- always system-derived, never user input | Always | -- | -- | -- |
| sent_at | No validation beyond data type -- empty until the send completes, then system-set | Always | -- | -- | -- |
| pause_state | Required; must be exactly one of Active, Paused by freelancer, Paused while bank transfer pending | Always | On any transition | "Reminders are paused for this invoice." (surfaced by FEAT-11.SPEC-003 when a send attempt is blocked by a non-Active state) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Eligible-to-send | invoice.status, pause_state, existing Reminder Log entries for the invoice, reminder_type | A reminder (automatic or manual) may send only when all of: (a) the invoice's status is not Paid, Refunded, Partially refunded, or Corrected; (b) `pause_state` is Active; (c) if `reminder_type` is manual, no other manual entry for this invoice has `sent_at` within the current calendar day in the freelancer's time zone | Manual attempt while ineligible on (a): control not shown (nothing to remind about). Manual attempt while ineligible on (b): "Reminders are paused for this invoice. Resume reminders to send one now." Manual attempt while ineligible on (c): "You've already sent a reminder for this invoice today. You can send another tomorrow." |
| Overdue-day derivation | invoice.due_date, Freelancer Account.time_zone, invoice.status | Elapsed overdue days = whole calendar days between `due_date` and the current date, both evaluated in the freelancer's stored time zone; the invoice's Overdue status applies whenever elapsed days ≥ 1 and the invoice is not Paid, Refunded, Partially refunded, or Corrected | Not user-facing -- this is a system derivation with no error state |
| Pause-reason precedence | pause_state (current value), the pause reason being applied | A freelancer-initiated pause (FEAT-11.SPEC-003) always sets `pause_state` to Paused by freelancer, overwriting any existing Paused while bank transfer pending value. A bank-transfer-pending transition (FEAT-11.SPEC-005) sets `pause_state` to Paused while bank transfer pending only when the current value is Active, and clears it back to Active only when the current value is Paused while bank transfer pending -- it never touches a Paused by freelancer value in either direction | Not user-facing -- this governs which automated writer may change the field, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View reminder history and current pause state | Nadia (Freelancer) | Always, for any of her own invoices | -- |
| View reminder history and current pause state | Dana (Support Operator) | Only inside a logged, time-limited support session opened on the freelancer's account (FEAT-31.SPEC-002) | Outside an open support session, the panel is unreachable to Dana at all |
| View reminder history and current pause state | Owen (Client Primary Contact) | Never | The Invoice Reminder Panel is never shown to Owen; he receives only the Overdue Reminder Email (FEAT-11.SPEC-004) |
| View reminder history and current pause state | Priya (Client Reviewer Contact) | Never | The panel is never shown to Priya, and she is never addressed by any reminder email (consistent with her having no Invoicing & Payments access) |
| Pause reminders for an invoice | Nadia (Freelancer) | Always, for any of her own invoices, regardless of the invoice's current pause reason | -- |
| Pause reminders for an invoice | Dana (Support Operator) | Never | The Pause control is rendered but disabled in Dana's read-only mirror, with the experience "Pause is unavailable in a read-only support session." |
| Pause reminders for an invoice | Owen, Priya | Never | Control not shown -- neither contact reaches this panel |
| Resume reminders for an invoice | Nadia (Freelancer) | Only when `pause_state` is currently Paused by freelancer (resuming a bank-transfer-pending pause is not a freelancer action -- see FEAT-11.SPEC-005) | If `pause_state` is currently Paused while bank transfer pending, the Resume control shows "Paused while a bank transfer is pending -- this resumes automatically." instead of an actionable button |
| Resume reminders for an invoice | Dana (Support Operator) | Never | Control disabled with "Resume is unavailable in a read-only support session." |
| Resume reminders for an invoice | Owen, Priya | Never | Control not shown |
| Send a manual reminder | Nadia (Freelancer) | Only when the Eligible-to-send cross-field rule passes (invoice not paid/refunded/corrected, `pause_state` is Active, no manual send already sent today for this invoice) | See the Eligible-to-send rule's per-condition denied messages above |
| Send a manual reminder | Dana (Support Operator) | Never | Control disabled with "Sending is unavailable in a read-only support session." |
| Send a manual reminder | Owen, Priya | Never | Control not shown |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| pause_state | Active | When an invoice first becomes eligible to carry a reminder schedule (i.e., once it is sent, per FEAT-09) | Yes -- Nadia may pause at any time (Authorization Rules); the automatic bank-transfer-pending trigger (FEAT-11.SPEC-005) may also change it, subject to the Pause-reason precedence rule |
| scheduled_for | `due_date` + 3 calendar days (for `reminder_type` = day 3) or `due_date` + 10 calendar days (for day 10), computed in the freelancer's time zone; not applicable to manual entries, which record `scheduled_for` as the moment Nadia triggered the send | On creation of each Reminder Log entry (FEAT-11.SPEC-001 or FEAT-11.SPEC-003) | No |
| Invoice.status = Overdue | Derived: true whenever elapsed overdue days ≥ 1 and the invoice is not Paid, Refunded, Partially refunded, or Corrected | Continuously, re-evaluated at every eligibility check and at invoice display | No -- this status is never set directly by any role; it is always computed |

## Business Rules

- XBR-15: reminders stop the instant the invoice is paid, pause while a bank transfer is pending or when the freelancer pauses that invoice, and manual reminders are limited to one per invoice per day.
- XBR-20: an invoice is paid once and in full only, and processor-confirmed payment status is authoritative -- the paid check in the Eligible-to-send rule always reflects the most recently confirmed status (FEAT-10.SPEC-004), never a stale read.
- The dependency map's Contention note for Reminder Log applies: the schedule re-checks pause, pending, and paid state immediately before each send; a pause saved first wins.
- The one-manual-reminder-per-day limit is measured against the freelancer's own local calendar day (FEAT-15.SPEC-006), not a fixed 24-hour rolling window -- a manual reminder sent at 11:58pm and another attempted at 12:02am the same freelancer-local night are on different calendar days and both may proceed.
- A second concurrent manual send attempt against the same invoice is refused with refresh (dependency map, Reminder Log Contention note): the second request's Eligible-to-send check reads the entry the first request just created and denies it under the one-per-day limit.

## Edge Cases

- **Invoice reaches Paid and a manual send is attempted in the same moment** -- The Eligible-to-send check reads the invoice's current (already-Paid) status and denies the send; the control is also hidden on the next panel refresh once Paid is visible.
- **Both pause reasons would apply at once (Nadia pauses while a bank transfer is already pending)** -- The Pause-reason precedence rule resolves this: Nadia's action always wins and sets Paused by freelancer, overwriting Paused while bank transfer pending. If the bank transfer later resolves (FEAT-11.SPEC-005), the pending-pause clearing logic finds the current value is Paused by freelancer, not Paused while bank transfer pending, and does not touch it -- the invoice remains paused until Nadia resumes it herself.
- **Manual send exactly at the boundary of the freelancer's local calendar day** -- A send at 23:59:59 and a second attempt at 00:00:01 the same freelancer-local night are on different calendar days per the one-per-day rule's basis; the second attempt is allowed.
- **Elapsed-day count crosses a daylight-saving transition in the freelancer's time zone** -- The calendar-day count is unaffected; a transition day still counts as exactly one elapsed day, consistent with FEAT-15.SPEC-006's local-date-boundary rule.
- **Dana opens a support session on an invoice that Nadia has paused** -- Dana sees the Paused by freelancer state exactly as Nadia would, but every action control (Pause, Resume, Send Reminder Now) remains disabled per the Authorization Rules -- read-only is unconditional and does not vary with the invoice's pause state.
- **Nadia's session pauses an invoice while the Reminder Schedule (FEAT-11.SPEC-001) has already begun its eligibility check for the same invoice's day-3 send** -- Whichever write commits first wins: if the pause commits before the schedule's eligibility read, the send is denied (skipped); if the send has already been recorded as sent before the pause commits, the completed send stands and the pause applies only to future thresholds.

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Nadia's invoice to Owen is Overdue with `pause_state` Active and no reminder sent today, when the Eligible-to-send rule is checked for a manual send, then it returns eligible.

**FEAT-11.SPEC-002-AC-02:** Given the same invoice has just been marked Paid, when the Eligible-to-send rule is checked (automatic or manual), then it returns not eligible and the Send Reminder Now control is not shown on FEAT-11.SPEC-003.

**FEAT-11.SPEC-002-AC-03:** Given the invoice's `pause_state` is Paused by freelancer, when a manual send is attempted, then it is denied with "Reminders are paused for this invoice. Resume reminders to send one now."

**FEAT-11.SPEC-002-AC-04:** Given Nadia already sent a manual reminder for this invoice earlier today (freelancer's local day), when she attempts to send another today, then it is denied with "You've already sent a reminder for this invoice today. You can send another tomorrow."

**FEAT-11.SPEC-002-AC-05:** Given Nadia sent a manual reminder at 23:59 freelancer-local time, when she attempts another at 00:02 the following freelancer-local day, then the send is allowed, since the two attempts fall on different calendar days.

**FEAT-11.SPEC-002-AC-06:** Given an invoice's `due_date` was yesterday in the freelancer's time zone, when the Overdue-day derivation runs, then the invoice's status is Overdue with elapsed days = 1.

**FEAT-11.SPEC-002-AC-07:** Given an invoice is Refunded, when the Overdue-day derivation runs, then the invoice's status is never set to Overdue, regardless of how many days have elapsed since its due date.

**FEAT-11.SPEC-002-AC-08:** Given an invoice's `pause_state` is currently Active, when the Bank-Transfer-Pending Reminder Pause (FEAT-11.SPEC-005) applies its pause, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-002-AC-09:** Given an invoice's `pause_state` is currently Paused by freelancer, when a bank transfer for that invoice enters pending status, then `pause_state` remains Paused by freelancer -- the automatic pause never overwrites Nadia's own pause.

**FEAT-11.SPEC-002-AC-10:** Given an invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps Pause on FEAT-11.SPEC-003, then `pause_state` becomes Paused by freelancer, overwriting the automatic pause reason.

**FEAT-11.SPEC-002-AC-11:** Given an invoice's `pause_state` is Paused while bank transfer pending, when the pending transfer resolves to Paid or reverts, then `pause_state` returns to Active (subject to the invoice's paid check for the Paid outcome).

**FEAT-11.SPEC-002-AC-12:** Given Nadia (Freelancer) opens the Invoice Reminder Panel for her own invoice, when the panel loads, then she can view the full reminder history and pause state, and the Pause/Resume and Send Reminder Now controls are active per her invoice's current state.

**FEAT-11.SPEC-002-AC-13:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens the same panel, then she can view the same history and state read-only, and every action control appears disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-002-AC-14:** Given Owen (Client Primary Contact) is signed in to his own portal, when he looks for any path to the Invoice Reminder Panel, then none exists -- the panel is never shown to him.

**FEAT-11.SPEC-002-AC-15:** Given Priya (Client Reviewer Contact) is signed in to her own portal, when she looks for any path to the panel or for a reminder email addressed to her, then neither exists.

**FEAT-11.SPEC-002-AC-16:** Given Nadia has paused reminders and two of her own browser sessions both attempt to send a manual reminder for the same invoice at effectively the same time, when both requests reach the Eligible-to-send check, then at most one succeeds and the second is refused with refresh, per the one-per-day limit reading the first request's just-created entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

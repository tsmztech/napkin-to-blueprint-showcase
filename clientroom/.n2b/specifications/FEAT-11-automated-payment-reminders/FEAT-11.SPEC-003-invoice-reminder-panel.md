---
document_type: spec
spec_type: screen
spec_id: FEAT-11.SPEC-003
spec_name: Invoice Reminder Panel
spec_slug: invoice-reminder-panel
parent_feature: FEAT-11
parent_feature_name: Automated Payment Reminders
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Invoice Reminder Panel

## Overview

**Name:** Invoice Reminder Panel
**ID:** FEAT-11.SPEC-003
**Type:** Screen
**Purpose:** Shows an invoice's reminder history and lets Nadia pause or resume the automatic reminder schedule and send a manual reminder, all scoped to one invoice.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- Displaying the invoice's Reminder Log history (day 3, day 10, and manual entries, in send order) with each entry's type and timestamp in Nadia's own time zone
- Displaying the invoice's current pause state
- Pausing and resuming the reminder schedule for this one invoice
- Sending a manual reminder, gated by FEAT-11.SPEC-002's eligibility rule
- Dana's read-only mirror of this same panel inside a logged support session

**Non-Goals:**
- Determining the day-3/day-10 schedule dates or performing the automatic sends themselves -- owned by FEAT-11.SPEC-001 (Reminder Schedule).
- The paid/pause/rate-limit eligibility logic itself -- owned by FEAT-11.SPEC-002 (Reminder Eligibility Rule); this panel enforces what that spec decides, it does not decide it.
- The content of the reminder email Owen receives -- owned by FEAT-11.SPEC-004 (Overdue Reminder Email).
- A standalone landing screen or list of reminders across invoices -- excluded per this feature's Shared Context: this feature has no cross-invoice reminder list; the invoice detail (FEAT-09.SPEC-002) is the only surface, consistent with the product's per-invoice, no-configurable-automation design (scope-boundaries.md, SC-11).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Nadia opens an invoice's detail view; this panel is embedded as a section within it | The invoice reference and its current status |
| FEAT-12.SPEC-001 (Dashboard Overview) | Nadia clicks an invoice flagged Overdue on her dashboard | Navigates to FEAT-09.SPEC-002 for that invoice, with this panel visible in place |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana opens a logged support session and navigates to the same invoice's detail on the freelancer's mirrored account | The invoice reference; every control on this panel renders disabled |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full panel: reminder history, current pause state | Pause, resume, and send a manual reminder, each gated by FEAT-11.SPEC-002 | -- |
| Owen (Client Primary Contact) | No | No | The panel does not exist anywhere in Owen's portal; he receives only the Overdue Reminder Email (FEAT-11.SPEC-004) and its pay link |
| Priya (Client Reviewer Contact) | No | No | The panel does not exist anywhere in Priya's portal, consistent with her having no Invoicing & Payments access at all |
| Dana (Support Operator) | Full panel, read-only, only inside a logged, time-limited support session (FEAT-31.SPEC-002) | None -- every control renders disabled with "unavailable in a read-only support session" | Outside an open support session, Dana has no path to this panel at all |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; after signing in, the user lands on FEAT-12.SPEC-001 (Dashboard Overview), not this panel directly |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if a pause/resume or manual-send action was mid-flight, it is discarded and must be retried after re-authentication |

## Layout and Content

**Section placement:** This panel is a "Reminders" section within FEAT-09.SPEC-002 (Invoice Detail), positioned below the invoice's payment status area.

**Header:** Section title "Reminders" with the current pause-state indicator to its right (a short label: "Active", "Paused by you", or "Paused -- bank transfer pending").

**Body:**
- A "Send Reminder Now" button, top-right of the section, active whenever FEAT-11.SPEC-002's eligibility rule allows a manual send for this invoice.
- A "Pause reminders" / "Resume reminders" toggle button, next to the pause-state indicator, whose label and behavior depend on the current `pause_state`: "Pause reminders" while Active, "Resume reminders" while Paused by freelancer, and "Pause reminders" (remaining tap-able) while Paused while bank transfer pending.
- Below the controls, a reminder history list, one row per Reminder Log entry, in send order (oldest first): reminder type (Day 3, Day 10, or Manual), and the date/time it was sent, in Nadia's own time zone. Dana's mirrored view shows the same rows.
- If `pause_state` is Paused while bank transfer pending, static text appears next to the toggle: "Paused while a bank transfer is pending -- this resumes automatically." The toggle itself remains present and reads "Pause reminders" -- tapping it invokes the same Pause action as the Active state, per FEAT-11.SPEC-002-AC-10. There is no Resume control in this state; resuming happens only automatically when the bank transfer resolves (FEAT-11.SPEC-005).

All fields use one consistent input treatment platform-wide, per the design layer.

### Responsive Behavior

- **Compact breakpoint:** The pause-state indicator and toggle stack below the "Reminders" header; "Send Reminder Now" remains a full-width button below them; history rows stack as single-column cards.
- **Medium size class and above:** Header, pause-state indicator, toggle, and "Send Reminder Now" sit on one row; history rows render as a compact table (type, timestamp).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Pause reminders toggle | Tap (while `pause_state` is Active) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Paused by freelancer | Indicator switches to "Paused by you"; toggle label switches to "Resume reminders" | Toast: "Reminders paused for this invoice." |
| Pause reminders toggle | Tap (while `pause_state` is Paused while bank transfer pending) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Paused by freelancer, overwriting the automatic pause reason (FEAT-11.SPEC-002-AC-10) | Indicator switches to "Paused by you"; the "Paused while a bank transfer is pending" static text is removed; toggle label switches to "Resume reminders" | Toast: "Reminders paused for this invoice." |
| Resume reminders toggle | Tap (while `pause_state` is Paused by freelancer) | Invokes FEAT-11.SPEC-002's authorization check, then sets `pause_state` to Active | Indicator switches to "Active"; toggle label switches to "Pause reminders" | Toast: "Reminders resumed for this invoice." |
| Send Reminder Now button | Tap | Invokes FEAT-11.SPEC-002's Eligible-to-send check; if eligible, triggers FEAT-11.SPEC-004 (Overdue Reminder Email) with `reminder_type` = manual, then creates the Reminder Log entry | Button shows a brief loading state during the check and send; a new "Manual" row appears at the bottom of the history on success | Success: toast "Reminder sent to {Primary Contact name}." and the new history row appears. Denied: exact message from FEAT-11.SPEC-002's Authorization Rules (e.g. "You've already sent a reminder for this invoice today. You can send another tomorrow.") shown inline near the button; no history row is added. |
| Send Reminder Now button (while loading) | Tap | No action -- debounced | Button remains in loading state | Button stays disabled until the in-flight attempt resolves |
| Reminder history row | -- | Display-only -- no interaction | None | -- |

### Accessibility Notes

- **Focus order:** Pause-state indicator -> Pause/Resume toggle -> Send Reminder Now button -> reminder history rows (in send order).
- **Dynamic announcements:** The pause-state indicator's change is announced to assistive technology when the toggle completes. The "Reminder sent" toast and any denied-send message are announced on appearance.
- **Keyboard alternatives:** The toggle and Send Reminder Now button are both standard activatable controls reachable and operable entirely by keyboard; there are no pointer-only gestures on this panel.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| No reminders yet | History area shows "No reminders sent yet -- automatic reminders begin 3 days after the due date if this invoice stays unpaid." Pause/Resume and Send Reminder Now controls remain visible per the current eligibility state | Invoice has no Reminder Log entries (not yet overdue, or overdue but not yet at day 3) | The first Reminder Log entry (automatic or manual) is created |
| Populated | History list shows one or more entries; pause-state indicator reflects the current value | At least one Reminder Log entry exists | Never exits -- remains populated once the first entry exists |
| Loading | Panel shows a loading placeholder in place of the history list and controls | Panel first mounts, before the invoice's Reminder Log data has loaded | Data load completes (success or error) |
| Error | Banner: "Couldn't load reminder history. Try again." with a Retry control; controls (Pause/Resume/Send) are disabled until retried | The reminder-history data load fails | Nadia taps Retry and the load succeeds |
| Action in progress | The specific control (toggle or Send Reminder Now) that was tapped shows a loading state; other controls remain interactive | Nadia taps Pause, Resume, or Send Reminder Now | The action completes (success or denied) |
| Offline/Degraded | Banner "You're offline -- reminder history is shown from your last successful load. Pause, resume, and manual sends are unavailable until you reconnect." History remains visible read-only; all action controls are disabled | Connectivity lost while the panel is open | Connectivity restored -- controls re-enable and the panel re-fetches the latest state |

## Validation Rules

Validation and authorization for pausing, resuming, and manual sending are governed entirely by FEAT-11.SPEC-002 (Reminder Eligibility Rule). See that spec for every allowed/denied condition and its exact denied message. This panel applies those checks at the moment each control is activated, never before.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| This panel has no navigation controls of its own -- it is a section within FEAT-09.SPEC-002 | FEAT-09.SPEC-002 (Invoice Detail) | -- (same screen; this is a section, not a distinct navigable screen) |

## Data Model

**Creates:** Reminder Log entry -- `invoice`, `reminder_type` = manual, `scheduled_for` = the moment of the tap, `sent_at` set on successful send.
**Reads:** Invoice -- `status`, `due_date` (for context and to gate eligibility, from FEAT-09/FEAT-10). Reminder Log -- every entry for this invoice, in send order. Client Contact -- the invoice's Primary Contact name, shown in the "Reminder sent to {name}" feedback.
**Updates:** Reminder Log -- `pause_state`, via the Pause/Resume toggle.
**Deletes:** None -- this panel never removes a Reminder Log entry.

## Business Rules

- FEAT-11.SPEC-002 owns every eligibility, authorization, and rate-limit rule enforced on this panel; this panel never re-implements those checks.
- XBR-15: manual reminders are limited to one per invoice per day, enforced by FEAT-11.SPEC-002 and surfaced here as a denied message rather than a hidden control, so Nadia understands why the button is unavailable.
- Pausing or resuming affects only the one invoice this panel is scoped to -- there is no bulk pause/resume across invoices, consistent with the product's fixed, per-invoice behavior (SC-11).

## Edge Cases

- **Nadia taps Send Reminder Now twice in rapid succession** -- The second tap is ignored while the first is in flight (button in loading state, debounced).
- **The automatic Reminder Schedule (FEAT-11.SPEC-001) sends a day-3 reminder while this panel is open** -- The panel is a snapshot at load time; the new history row does not appear live. Nadia sees it on her next visit or manual refresh. This is acceptable because reminder sending has no time-sensitive interactive component the freelancer must react to in the moment.
- **Nadia taps Pause at the exact moment the automatic schedule's eligibility re-check runs for a threshold that is currently due** -- Per the dependency map's Contention note (a pause saved first wins): if Nadia's pause commits first, the automatic send is skipped; if the automatic send has already completed, the pause takes effect only for the next threshold.
- **Two of Nadia's own sessions both tap Send Reminder Now for the same invoice at effectively the same time** -- Resolution: reject-with-refresh, per the dependency map's Contention note for Reminder Log. The first request to commit succeeds; the second's eligibility check reads the just-created entry and is denied under the one-per-day limit, with the panel refreshing to show the new entry.
- **Nadia navigates away mid-send (before the manual-send result returns) and comes back** -- On return, the panel re-fetches the current state; if the send had actually completed on the server before she navigated away, the new history row is present. No duplicate send is triggered by returning to the panel.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (inbound) | This panel is embedded as a section within that screen |
| FEAT-12.SPEC-001 (Dashboard Overview) | Navigation (inbound) | Nadia arrives here after clicking an Overdue invoice on her dashboard |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana's mirrored, read-only view of this same panel |
| FEAT-11.SPEC-001 (Reminder Schedule) | References (inbound) | Automatic sends populate the history this panel displays |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | References (inbound) | Every action on this panel is authorized and validated by that spec |
| FEAT-11.SPEC-004 (Overdue Reminder Email) | Triggers (outbound) | The Send Reminder Now action triggers this notification |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Affects (outbound) | Pause, resume, and manual-send actions each write a trail entry (XBR-05) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reminder_paused | invoice reference, elapsed overdue days at time of pause | Nadia successfully pauses reminders for an invoice | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| reminder_resumed | invoice reference | Nadia successfully resumes reminders for an invoice | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| manual_reminder_sent | invoice reference, elapsed overdue days at time of send | Nadia's manual send completes successfully | supports success-metrics.md: "Reminder-Driven Payment Recovery" |
| manual_reminder_send_denied | invoice reference, denial reason (paid / paused / rate_limited) | A manual send attempt is denied | supports success-metrics.md: "Reminder-Driven Payment Recovery" |

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given Nadia opens the Invoice Detail for an Overdue invoice that has already had its day-3 reminder sent, when the panel loads, then she sees one "Day 3" history row with its send timestamp in her own time zone.

**FEAT-11.SPEC-003-AC-02:** Given Nadia opens the panel for an invoice not yet at day 3, when the panel loads, then she sees "No reminders sent yet -- automatic reminders begin 3 days after the due date if this invoice stays unpaid."

**FEAT-11.SPEC-003-AC-03:** Given Nadia taps "Pause reminders" on an Active invoice, when the action completes, then the indicator switches to "Paused by you", the toggle becomes "Resume reminders", and she sees the toast "Reminders paused for this invoice."

**FEAT-11.SPEC-003-AC-04:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia views the panel, then she sees the static text "Paused while a bank transfer is pending -- this resumes automatically." next to a "Pause reminders" toggle that remains tap-able (there is no Resume button in this state).

**FEAT-11.SPEC-003-AC-05:** Given the invoice is eligible for a manual reminder, when Nadia taps "Send Reminder Now", then the reminder is sent, a new "Manual" row appears in the history, and she sees "Reminder sent to Owen." (or the Primary Contact's actual name).

**FEAT-11.SPEC-003-AC-06:** Given Nadia already sent a manual reminder for this invoice earlier today, when she taps "Send Reminder Now" again, then she sees the inline message "You've already sent a reminder for this invoice today. You can send another tomorrow." and no history row is added.

**FEAT-11.SPEC-003-AC-07:** Given Owen (Client Primary Contact) is viewing his own portal, when he looks for any way to reach this panel, then no such path exists anywhere in his portal.

**FEAT-11.SPEC-003-AC-08:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens this panel, then she sees the full history and pause state, but every control (Pause, Resume, Send Reminder Now) is disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-003-AC-09:** Given the reminder-history data fails to load, when the panel attempts its initial load, then Nadia sees "Couldn't load reminder history. Try again." with a Retry control, and all action controls are disabled until the retry succeeds.

**FEAT-11.SPEC-003-AC-10:** Given Nadia loses connectivity while the panel is open, when she attempts to tap Pause, then the offline banner is shown and the action is not attempted until connectivity returns.

**FEAT-11.SPEC-003-AC-11:** Given Nadia taps "Send Reminder Now" twice in rapid succession, when the first tap is still processing, then the second tap has no effect and the button remains in its loading state.

**FEAT-11.SPEC-003-AC-12:** Given Nadia's session expires while a Pause action is mid-flight, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the pause action is discarded, requiring her to retry after re-authenticating.

**FEAT-11.SPEC-003-AC-13:** Given two of Nadia's own sessions both tap "Send Reminder Now" for the same invoice at effectively the same time, when both requests are processed, then only one manual send succeeds and the other is denied with the one-per-day message, with its panel refreshing to show the new entry.

**FEAT-11.SPEC-003-AC-14:** Given the invoice becomes Paid while Nadia is viewing the panel, when she next taps "Send Reminder Now" (without having refreshed), then the send attempt is denied per FEAT-11.SPEC-002 rather than silently succeeding, since eligibility is re-checked at the moment of the tap.

**FEAT-11.SPEC-003-AC-15:** Given Priya (Client Reviewer Contact) is viewing her own portal, when she looks for any way to reach this panel or receive a reminder-related communication, then neither exists anywhere in her portal.

**FEAT-11.SPEC-003-AC-16:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps the "Pause reminders" toggle, then `pause_state` becomes Paused by freelancer (overwriting the automatic pause reason, per FEAT-11.SPEC-002-AC-10), the static "Paused while a bank transfer is pending" text is removed, the indicator switches to "Paused by you", the toggle switches to "Resume reminders", and she sees the toast "Reminders paused for this invoice."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (no reminders yet, populated, loading, error, action in progress, offline/degraded) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

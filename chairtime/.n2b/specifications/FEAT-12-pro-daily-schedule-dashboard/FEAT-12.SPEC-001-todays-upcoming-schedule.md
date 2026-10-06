---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-001
spec_name: Today's & Upcoming Schedule
spec_slug: todays-upcoming-schedule
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Screen Spec: Today's & Upcoming Schedule

## Overview

**Name:** Today's & Upcoming Schedule
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** The Pro's main dashboard: today's remaining bookings in time order plus upcoming bookings beyond today, each with a paid badge, balance due, "I'll be there" status, sync-reliability marking, and quick-action entry points.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying today's remaining bookings in time order and upcoming bookings beyond today
- Showing manual time blocks (FEAT-17) alongside bookings so blocked periods read as occupied
- Per-row paid badge, balance due, "I'll be there" status, and sync-reliability marking (derived by FEAT-12.SPEC-007)
- Quick-action entry points: view client note, mark no-show, jump to reschedule/cancel, mark completed
- Navigating to the Attention List (FEAT-12.SPEC-002), Past Bookings Browse (FEAT-12.SPEC-003), and cross-feature destinations reachable from the dashboard's navigation
- The empty-day state and the loading/offline/error states for this screen

**Non-Goals:**
- Displaying the aggregated Attention List itself -- owned by FEAT-12.SPEC-002; this screen only shows a summary banner that links to it
- Browsing past bookings by date -- owned by FEAT-12.SPEC-003
- Performing the no-show mark, cancel, reschedule, or refund actions themselves -- owned by FEAT-11 and FEAT-30 respectively; this screen only provides the entry point and navigates there
- Taking an in-app balance payment -- excluded per scope-boundaries.md SC-16: the balance is deliberately settled in person at MVP; this screen only offers the mark-completed action, which records the balance as settled in person, never a payment flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-001 (Sign-In Screen) / FEAT-29.SPEC-002 (Account Recovery Screen) (Pro Sign-In & Account Lifecycle) | Pro signs in / opens the app | None -- this is the default landing screen after sign-in |
| FEAT-12.SPEC-002 (Attention List) | Pro navigates back from the Attention List | None -- schedule reloads current data |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Pro navigates back from past bookings | None -- schedule reloads current data |
| FEAT-13 (Client Record Management) | Pro returns after viewing/editing a client record | None -- schedule reloads current data |
| FEAT-30 (Pro Booking Management), FEAT-11 (No-Show Marking), FEAT-17 (Manual Time Blocking) | Pro completes or cancels a cross-feature action and returns | None -- schedule reloads to reflect any change |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own schedule only | All quick actions and navigation | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review, except the client private-note preview, which is omitted entirely (FEAT-12.SPEC-008) | View only -- no quick actions, no mark-completed control | Any write control is simply not present; there is no denial dialog because no write path is ever rendered for Support |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29); Clients see only their own bookings through FEAT-06, never this screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen to preserve, since it is a viewing surface with no form state |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title showing the current date (e.g., "Today"), with an Attention banner directly below the title when one or more Open Attention Items exist (FEAT-12.SPEC-005) -- the banner states the count and a short label (e.g., "3 things need your attention") and is tappable. Navigation to Settings (FEAT-27), Money (FEAT-28), and Insights (FEAT-25) is reachable from a persistent navigation affordance in the header area.

**Body:** A single vertically scrolling list organized into two sections in order:
1. **Today** -- today's remaining bookings and any manual time blocks (FEAT-17), in strict time order. A booking whose start_time has already passed and is not yet marked Completed, No-Show, or Cancelled remains visible in this section (it does not disappear once its time passes) until the Pro acts or the Auto-Completion Sweep (FEAT-12.SPEC-004) resolves it.
2. **Upcoming** -- bookings beyond today, grouped by date, each date heading followed by that date's bookings in time order.

Each **booking row** (the shared pattern used identically here and on FEAT-12.SPEC-003, per the Brief's Shared UI Patterns) shows, left to right / top to bottom: start time, service name, client name (tappable), the paid badge and balance-due figure (derived by FEAT-12.SPEC-007), the "I'll be there" attendance label when present, a sync-reliability marking when the booking's reliability is uncertain, and a client note preview (a short excerpt of the Pro's private note for that client, when one exists) truncated to fit one line. A row-level action affordance exposes: view full client note, mark no-show, jump to reschedule/cancel, and (only once start_time has passed and the booking is still Confirmed or Awaiting Outcome) mark completed.

Each **time block row** shows its start/end and, for the Pro only, its private label; it has no quick actions except "add/edit time block," which navigates to FEAT-17.

**Footer:** None -- all actions are inline within rows or the header banner.

### Responsive Behavior

- **Compact breakpoint (phone width, including inside the Instagram in-app browser):** Single-column list as described above, full width; the Attention banner remains directly under the header at all times without requiring a scroll.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping, since the product's primary use is phone-first (ASMP: mobile-first for both roles).
- **Booking row on very narrow widths:** The client note preview truncates further or is omitted first, before any status badge (paid, balance due, attendance, reliability) is dropped -- badges never disappear to save space, since paid/attention status must remain visible at a glance.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Attention banner | Tap | Navigate to FEAT-12.SPEC-002 (Attention List) | Screen transitions | Standard navigation transition |
| Booking row -- client name | Tap | Navigate to FEAT-13 (Client Record Management) for that client | Screen transitions | Standard navigation transition |
| Booking row -- client note preview | Tap | Expand the full private note inline on the row | Row expands to show full note text | Note text becomes fully visible without navigating away |
| Booking row -- "no-show" action | Tap | Navigate to FEAT-11 (No-Show Marking & Deposit Forfeiture) for that booking | Screen transitions | Standard navigation transition |
| Booking row -- "reschedule/cancel" action | Tap | Navigate to FEAT-30 (Pro Booking Management) for that booking | Screen transitions | Standard navigation transition |
| Booking row -- "mark completed" action (shown only once eligible per FEAT-12.SPEC-006) | Tap | Validate eligibility via FEAT-12.SPEC-006; if valid, transition the booking to Completed and record the balance as settled in person | Row updates in place to show Completed status; action affordance for that row is removed | Brief inline confirmation (e.g., a check mark and "Completed") replaces the action; no full-screen transition |
| Booking row -- "mark completed" action (attempted before eligible) | Tap | No state change -- the control is disabled before start_time has passed | Control remains disabled | Control shows a disabled visual treatment; it is not tappable |
| "Add time block" affordance | Tap | Navigate to FEAT-17 (Manual Time Blocking) | Screen transitions | Standard navigation transition |
| Empty-day shortcut ("share your booking link") | Tap | Opens the Pro's own booking-link share action | Share action presented | Standard platform share affordance appears |
| Navigation -- Money | Tap | Navigate to FEAT-28 (Payout Account Connection & Payout Visibility) | Screen transitions | Standard navigation transition |
| Navigation -- Settings | Tap | Navigate to FEAT-27 (Pro Profile & Booking Page Settings) | Screen transitions | Standard navigation transition |
| Navigation -- Insights | Tap | Navigate to FEAT-25 (Booking & Revenue Insights) | Screen transitions | Standard navigation transition |
| Navigation -- Past Bookings | Tap | Navigate to FEAT-12.SPEC-003 (Past Bookings Browse) | Screen transitions | Standard navigation transition |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches today's and upcoming bookings, time blocks, and derived status | List reloads | Loading indicator during refresh; list content updates in place |

### Accessibility Notes

- **Focus order:** Screen title -> Attention banner (when present) -> navigation affordances -> Today section heading -> each booking/time-block row in time order (within a row: time, service, client name, badges, note preview, action affordances) -> Upcoming section date headings and their rows in order.
- **Dynamic-change announcements:** When a "mark completed" action succeeds, the row's updated status ("Completed") is announced to assistive technology. When the Attention banner's count changes on refresh, the updated count is announced. Loading and offline-state transitions are announced when they occur.
- **Status conveyed beyond color:** Every paid, balance-due, attendance, and reliability badge carries a word, never color alone (ASMP-28), so no accessibility gap exists for badge meaning.
- **Keyboard alternatives:** Every action on this screen (navigation, expand note, mark completed, refresh) is reachable without a pointer-only gesture; pull-to-refresh has an equivalent tappable refresh control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has bookings) | Today and Upcoming sections populated as described in Layout and Content | Data fetch succeeds with at least one booking or time block | Data changes (new fetch, action taken) |
| Empty day | Today section shows a friendly "Nothing booked yet today" message with a shortcut to share the booking link; Upcoming section still shows if it has content | Today section has zero bookings and zero time blocks | A booking or time block for today is created, or the date rolls to a day with bookings |
| Loading | A lightweight in-place indicator appears without blanking existing content already on screen; on first-ever load, a full-screen lightweight loading indicator appears briefly | Screen first opens, or a refresh is triggered | Data fetch completes (success or failure) |
| Error | Error banner at the top of the list: "Couldn't load your schedule. Check your connection and try again." with a Retry control; any previously loaded content remains visible below the banner if this is a refresh failure, not a first load | Data fetch fails | Pro taps Retry and the fetch succeeds, or connectivity is restored and an automatic retry succeeds |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded schedule." at the top; the most recently loaded schedule remains viewable read-only; quick actions that write data (mark no-show, mark completed, reschedule/cancel) are disabled with a note that they require reconnecting; navigation to other features that themselves require connectivity shows their own offline handling | Connectivity is lost while this screen is open, or the screen is opened while already offline with cached data available | Connectivity is restored -- the banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry form fields; its only rule-governed interaction is the "mark completed" action.

Validation and eligibility for "mark completed" is governed by FEAT-12.SPEC-006 (Booking Completion Rules). See that spec for the full eligibility window and denied-state messages. Checked at the moment the Pro taps the action, before the write is attempted.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Attention banner tap | FEAT-12.SPEC-002 (Attention List) | -- |
| Client name tap | FEAT-13.SPEC-001 (Client Record Detail) | FEAT-13 |
| "No-show" action tap | FEAT-11 (No-Show Marking & Deposit Forfeiture) | FEAT-11 |
| "Reschedule/cancel" action tap | FEAT-30 (Pro Booking Management) | FEAT-30 |
| "Add time block" tap | FEAT-17 (Manual Time Blocking) | FEAT-17 |
| Navigation -- Past Bookings | FEAT-12.SPEC-003 (Past Bookings Browse) | -- |
| Navigation -- Money | FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28 |
| Navigation -- Settings | FEAT-27 (Pro Profile & Booking Page Settings) | FEAT-27 |
| Navigation -- Insights | FEAT-25.SPEC-001 (Insights Summary Screen) | FEAT-25 |

## Data Model

**Creates:** None.
**Reads:** Booking (service, start_time, duration, client reference, price_agreed, deposit_amount, state, attendance_reply, balance_due -- derived by FEAT-12.SPEC-007) for today and upcoming dates; Client (name, private_note preview); Deposit Transaction (status, amount, via FEAT-12.SPEC-007); Message (delivery_status flags); Time Block (start, end, label); Calendar Connection (status, via FEAT-12.SPEC-007); Messaging Consent (state, textability display only); Open Attention Item count (via FEAT-12.SPEC-005, for the banner).
**Updates:** Booking -- `state` field, transitioned to `Completed` by the "mark completed" action (validated by FEAT-12.SPEC-006).
**Deletes:** None.

## Business Rules

- The "mark completed" action is governed entirely by FEAT-12.SPEC-006 -- this screen enforces but does not define the eligibility window.
- Paid badge, balance due, attendance label, and sync-reliability marking are derived entirely by FEAT-12.SPEC-007 -- this screen renders but does not compute them.
- Every screen in this feature, including this one, requires an authorized viewer per FEAT-12.SPEC-008 -- checked before any data loads and on every refresh.
- XBR-11: a setup change never silently removes a booking from this list -- a booking left outside changed hours remains visible here and is separately flagged through the Attention List (FEAT-12.SPEC-002/FEAT-12.SPEC-005), never hidden or auto-cancelled.
- XBR-13: if calendar sync lapses, this screen shows reduced confidence (the sync-reliability marking) to the Pro only, never to the client.
- A booking whose start_time has passed remains visible in the Today section (not silently removed) until it is completed, marked no-show, or the Auto-Completion Sweep resolves it (FEAT-12.SPEC-004) -- this keeps the Pro's glance trustworthy about what still needs a decision.

## Edge Cases

- **Pro taps "mark completed" on a booking that another device (the Pro's own second session) or the Auto-Completion Sweep (FEAT-12.SPEC-004) has already completed** -- Concurrent-edit conflict, governed by the Booking entity's reject-with-refresh contention resolution (dependency map): the action is refused, the row refreshes to show its current Completed state, and no error is shown beyond the row simply reflecting the up-to-date status.
- **Pro taps "no-show" or "reschedule/cancel" on a booking that was cancelled by the Client moments earlier** -- The Pro is navigated to FEAT-11 or FEAT-30, which itself reads the booking's current state and shows that destination feature's own defined current-state message for a cancelled booking (reject-with-refresh); this screen's own row refreshes to the new state on return.
- **More bookings exist for a date than fit on screen** -- The list scrolls; no pagination or truncation of booking rows occurs within a day's list.
- **Pro double-taps "mark completed" rapidly** -- The second tap is ignored while the first write is in progress; the control shows a brief disabled/loading treatment during the write.
- **Pro navigates away mid-action (e.g., taps a quick action, then backs out before the destination screen loads)** -- No partial state is created on this screen, since this screen never writes data itself except the atomic mark-completed transition; navigating away before mark-completed is tapped leaves the booking untouched.
- **Time block overlaps a confirmed booking** -- Both are shown on the schedule (the block is shown as occupied time alongside the booking); resolving the overlap is an explicit Pro action through FEAT-17/FEAT-30, not something this screen resolves automatically.
- **Empty day with zero bookings and zero time blocks** -- The friendly "Nothing booked yet today" message and share-link shortcut appear, per the Empty day state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-006 (Booking Completion Rules) | References (outbound) | Governs the mark-completed action's eligibility and mechanics |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid badge, balance due, attendance label, and reliability marking shown on every row |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | References (inbound) | Supplies the Open Attention Item count shown in the header banner |
| FEAT-12.SPEC-002 (Attention List) | Navigation (outbound) | Attention banner navigates here |
| FEAT-12.SPEC-003 (Past Bookings Browse) | Navigation (outbound) | Past Bookings navigation entry navigates here |
| FEAT-12.SPEC-004 (Auto-Completion Sweep) | References (inbound) | Automatic completion outcomes are reflected here on refresh |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound) | Default landing screen after sign-in |
| FEAT-13 (Client Record Management) | Navigation (outbound) | Client name tap navigates here |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Navigation (outbound) | No-show action navigates here |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | Reschedule/cancel action navigates here |
| FEAT-17 (Manual Time Blocking) | Navigation (outbound) | Add time block navigates here |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Navigation (outbound) | Money navigation entry |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound) | Settings navigation entry |
| FEAT-25 (Booking & Revenue Insights) | Navigation (outbound) | Insights navigation entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dashboard_viewed | booking_count_today, has_attention_items (boolean) | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| quick_action_taken | action_type (view_note / no_show / reschedule_cancel / add_time_block) | Pro taps a row-level or header quick action | supports success-metrics.md: "Daily Dashboard Glance Speed" (view_note actions) or supports success-metrics.md: "Pro Change Correctness" (no_show and reschedule_cancel actions, which initiate the pro-side change flows that metric measures) |
| booking_marked_completed | time_since_start_time | "Mark completed" action succeeds | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| empty_day_share_link_tapped | -- | Pro taps the share-link shortcut on an empty day | supports success-metrics.md: "Daily Dashboard Glance Speed" (measures whether the empty state still leads to useful action rather than a dead end) |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Talia opens the app after signing in, when the dashboard loads, then she sees today's remaining bookings in time order with paid badge, balance due, and attendance status on each row within a few seconds, per the Daily Dashboard Glance Speed target.

**FEAT-12.SPEC-001-AC-02:** Given Talia has zero bookings today, when she opens the dashboard, then she sees "Nothing booked yet today" with a shortcut to share her booking link.

**FEAT-12.SPEC-001-AC-03:** Given Talia has bookings scheduled beyond today, when she scrolls past the Today section, then she sees the Upcoming section grouped by date.

**FEAT-12.SPEC-001-AC-04:** Given Talia taps a booking row's client name, when the tap registers, then she is navigated to that client's record (FEAT-13).

**FEAT-12.SPEC-001-AC-05:** Given Talia taps "no-show" on a booking row, when the tap registers, then she is navigated to FEAT-11 for that booking.

**FEAT-12.SPEC-001-AC-06:** Given Talia taps "reschedule/cancel" on a booking row, when the tap registers, then she is navigated to FEAT-30 for that booking.

**FEAT-12.SPEC-001-AC-07:** Given a booking's start_time has passed and it is still Confirmed, when Talia taps "mark completed," then the booking transitions to Completed and the row updates in place to show that status.

**FEAT-12.SPEC-001-AC-08:** Given a booking's start_time has not yet arrived, when Talia looks at its row, then the "mark completed" control is disabled and not tappable.

**FEAT-12.SPEC-001-AC-09:** Given one or more Open Attention Items exist, when Talia opens the dashboard, then the Attention banner shows the count and is tappable to FEAT-12.SPEC-002.

**FEAT-12.SPEC-001-AC-10:** Given zero Open Attention Items exist, when Talia opens the dashboard, then no Attention banner is shown.

**FEAT-12.SPEC-001-AC-11:** Given the dashboard's data fetch fails, when the failure occurs, then an error banner "Couldn't load your schedule. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-001-AC-12:** Given Talia loses connectivity while viewing the dashboard, when the offline state activates, then the banner "You're offline -- showing your most recently loaded schedule." appears, the schedule remains viewable, and write actions are disabled with a reconnect note.

**FEAT-12.SPEC-001-AC-13:** Given Talia's connectivity is restored after the offline state, when reconnection is detected, then the offline banner clears and the schedule refreshes automatically.

**FEAT-12.SPEC-001-AC-14:** Given a booking's Calendar Connection reliability is uncertain, when Talia views that row, then it shows the "Reliability uncertain" marking rather than presenting false confidence.

**FEAT-12.SPEC-001-AC-15:** Given Talia taps "mark completed" twice in rapid succession on the same row, when the second tap registers while the first write is in progress, then the second tap has no additional effect.

**FEAT-12.SPEC-001-AC-16:** Given a booking Talia is about to mark completed was already auto-completed by FEAT-12.SPEC-004 moments earlier, when she taps "mark completed," then the action is refused and the row refreshes to show the already-Completed status.

**FEAT-12.SPEC-001-AC-17:** Given Platform Operator (Support) is viewing Talia's dashboard during an active support session, when Support looks at any booking row, then no client private-note preview is shown and no quick-action controls appear.

**FEAT-12.SPEC-001-AC-18:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any booking data.

**FEAT-12.SPEC-001-AC-19:** Given a manual time block overlaps a confirmed booking on today's schedule, when Talia views the Today section, then both the block and the booking are shown, with the block visible as occupied time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 13 | 13 |
| States | 5 (loaded, empty day, loading, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |

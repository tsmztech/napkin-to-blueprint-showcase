---
document_type: spec
spec_type: screen
spec_id: FEAT-11.SPEC-001
spec_name: No-Show Mark & Undo Prompt
spec_slug: no-show-mark-undo-prompt
parent_feature: FEAT-11
parent_feature_name: No-Show Marking & Deposit Forfeiture
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: No-Show Mark & Undo Prompt

## Overview

**Name:** No-Show Mark & Undo Prompt
**ID:** FEAT-11.SPEC-001
**Type:** Screen
**Purpose:** Talia reaches this one prompt from a past-due booking row to mark a no-show, choose a goodwill refund instead, or -- within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) -- undo a mark she already made, seeing the current deposit outcome reflected inline the instant she confirms.
**Parent Feature:** FEAT-11 -- No-Show Marking & Deposit Forfeiture

## Scope and Non-Goals

**In Scope:**
- The single prompt surface that toggles between a "Mark no-show?" state (booking not yet marked) and an "Undo no-show?" state (booking marked, undo grace window still open)
- Confirming a no-show mark, which hands off to FEAT-11.SPEC-002 to write the Booking and Deposit Transaction transitions
- Confirming an undo, which hands off to FEAT-11.SPEC-003 to reverse those transitions
- Offering "Refund as goodwill instead" as a navigation choice out to Pro Booking Management (FEAT-30)
- Reflecting the deposit outcome (kept, or restored) inline the instant the triggering automation completes, with no separate invoicing step
- Showing when the undo window has elapsed, so the mark is understood as permanent

**Non-Goals:**
- Performing the no-show marking window and ownership checks themselves -- owned by FEAT-11.SPEC-004 (Logic/Rule); this screen only reflects the eligibility outcome the rule spec returns
- Writing the Booking or Deposit Transaction state transitions directly -- owned by FEAT-11.SPEC-002 (mark) and FEAT-11.SPEC-003 (undo); this screen only triggers them and displays their result
- The goodwill refund flow itself -- owned entirely by Pro Booking Management (FEAT-30), per this feature's Key Capabilities and Interactions field; this screen only offers the navigation choice
- Sending any notice to the Client -- excluded per the feature's Communications field, which keeps the deposit outcome "visible in the client's own booking history rather than triggering a separate confrontational notification"; this is a same-screen-visibility disposition with no channel or delivery rule of its own

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) -- past-due booking row | Talia taps "no-show" on a booking row whose appointment start time has passed | The Booking's identity, current state, start_time, client name, and policy_version |

This feature has no standalone entry point of its own -- per the Brief's Internal Dependency Map, the only way in is a specific past-due booking row on FEAT-12's dashboard.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for her own bookings only | Mark no-show, undo within the grace window, choose goodwill refund instead | If Talia somehow reaches this prompt for a booking she does not own (e.g. a stale deep link), the prompt does not open; she sees "This booking could not be found." and returns to her dashboard |
| The Client (Riley) | No -- this screen is never reachable from any Client-facing surface | No | Riley has no path to this prompt at all; her Cancellation & No-Show Handling access is Own-only visibility of the resulting outcome in her own booking history (FEAT-16), never this action screen |
| Platform Operator (Support) | No -- Support's View access to Cancellation & No-Show Handling is served through Platform Support Read-Only Access (FEAT-19)'s own read-only screens, not this prompt | No | Support has no path to this prompt; a booking's no-show state and forfeiture outcome are visible to Support only through FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen; no in-progress prompt context is preserved since the prompt cannot open without an authenticated Pro session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if the prompt was open with a choice not yet confirmed, no partial write exists (this action is one atomic confirm), so nothing is lost or replayed after re-authentication; Talia returns to the dashboard and re-opens the prompt from the booking row |

## Layout and Content

**Header:** Prompt title that reflects the current state -- "Mark no-show?" when the Booking is not yet marked, or "Undo no-show?" when it is marked and the undo grace window is still open. A close control (X, top-right) dismisses the prompt without any change.

**Body (Mark no-show state):**
- The booking's client name and appointment time, for confirmation of which booking is being acted on
- A single line stating the deposit outcome that will result: the deposit is kept in full once marked (no percentage, no invoicing step)
- Three actions, stacked: "Mark no-show" (primary), "Refund as goodwill instead" (secondary -- navigates to FEAT-30), and "Cancel" (dismisses the prompt)

**Body (Undo no-show state):**
- The same booking identity line
- A line confirming the current outcome: the deposit is kept, marked as a no-show at the timestamp it was marked
- A line stating how much of the undo grace window remains (e.g., expressed as remaining hours), computed against the fixed grace window governed by FEAT-11.SPEC-004
- Two actions, stacked: "Undo no-show" (primary) and "Keep as no-show" (secondary -- dismisses the prompt, no change)

**Body (grace window elapsed):**
- The same booking identity and outcome line
- A line stating the mark is now permanent: "This no-show mark can no longer be undone."
- One action: "Close" (dismisses the prompt)

**Footer:** None -- all actions live in the body.

### Responsive Behavior

- **Compact breakpoint:** The prompt renders as a full-width bottom sheet with the layout described above, stacked vertically.
- **Medium size class and above:** The prompt renders as a centered modal dialog capped at a consistent platform-wide dialog width (exact value is the design layer's decision); content and action order are unchanged from the compact layout -- uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close (X) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row unchanged |
| "Mark no-show" button | Tap | Re-validate eligibility via FEAT-11.SPEC-004, then trigger FEAT-11.SPEC-002 | Button shows a brief loading state | On success: prompt switches in place to the Undo state showing the kept-deposit outcome. On ineligibility (window closed since the prompt opened): prompt shows the exact denied message from FEAT-11.SPEC-004 and offers only "Close" |
| "Refund as goodwill instead" link | Tap | Navigate to Pro Booking Management's goodwill refund flow (FEAT-30), carrying this Booking's identity | Prompt closes | FEAT-30's refund screen opens, pre-scoped to this booking |
| "Cancel" button (Mark state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row unchanged |
| "Undo no-show" button | Tap | Re-validate the grace window via FEAT-11.SPEC-004, then trigger FEAT-11.SPEC-003 | Button shows a brief loading state | On success: prompt switches in place to the Mark state, reflecting the booking as Completed again with the deposit restored to Captured. On ineligibility (window elapsed since the prompt opened): prompt shows "This no-show mark can no longer be undone." and offers only "Close" |
| "Keep as no-show" button (Undo state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard, booking row still shows no-show/kept |
| "Close" button (elapsed state) | Tap | Dismiss the prompt, no change | Prompt closes | Return to FEAT-12's dashboard |

### Accessibility Notes

- **Focus order:** Close (X) -> title -> booking identity line -> outcome line -> primary action -> secondary action -> tertiary action (Cancel, when present).
- **State-change announcements:** When the prompt switches from Mark to Undo (or vice versa) after a successful confirm, the new title and outcome line are announced to assistive technology as a live-region update, since the content changes in place without a full screen navigation.
- **Error announcements:** An ineligibility message (window closed, session expired) is announced when it appears and moves focus to the message text.
- **Keyboard alternatives:** Every action on this prompt is reachable by keyboard; there are no pointer-only gestures. Escape closes the prompt with the same effect as the Close/Cancel control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | The prompt shell (header, close control) renders immediately with a skeleton placeholder in place of the booking identity, outcome line, and action row | Talia taps "no-show" on a dashboard row and the prompt opens, triggering the fresh Booking and Deposit Transaction read described in Data Model Reads | The fresh read resolves -- the prompt renders directly into whichever of Mark, Undo, or Grace window elapsed the resolved state and outcome timestamp dictate |
| Empty | N/A -- this screen only ever opens against one specific past-due booking carried from the triggering dashboard row (per Entry Points); it is never a list or collection view and has no zero-items case to render | N/A | N/A |
| Mark (default) | "Mark no-show?" title, booking identity, kept-deposit-outcome preview, three actions | Prompt opens for a booking not yet marked no-show, within the eligible marking window | Talia confirms the mark, cancels, or closes |
| Undo (within grace window) | "Undo no-show?" title, booking identity, current kept-deposit outcome, remaining grace-window time, two actions | Prompt opens for a booking already marked no-show, within the undo grace window | Talia confirms the undo, keeps the mark, or closes |
| Grace window elapsed | "Undo no-show?" title suppressed in favor of a permanent-outcome message; one Close action | Prompt opens for a booking marked no-show whose undo grace window has elapsed | Talia closes the prompt |
| Confirming | Primary action button shows a loading state; other actions disabled | Talia taps "Mark no-show" or "Undo no-show" | The triggered automation (FEAT-11.SPEC-002 or FEAT-11.SPEC-003) returns an outcome |
| Error | Inline message describing the exact ineligibility reason, with only a Close action remaining | The re-validation on confirm (FEAT-11.SPEC-004) finds the booking no longer eligible, or the triggered automation reports a write failure that could not complete after retry | Talia closes the prompt and returns to the dashboard, where the booking row reflects the current true state |
| Offline/Degraded | Banner "You're offline. Marking or undoing a no-show needs a connection." replaces the action row; the outcome preview remains visible but no action can be confirmed | Connectivity is lost while the prompt is open | Connectivity is restored -- the action row reappears and the prompt re-validates eligibility before allowing a confirm |

## Validation Rules

Validation governed by FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules). See that spec for the marking-window bounds, the fixed undo grace window, and the Pro-ownership check. This screen re-validates on prompt open and again on confirm, and surfaces the exact denied message that spec defines.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Close, Cancel, or Keep as no-show | Returns to the triggering booking row | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful mark or undo confirm | Prompt stays open, switched to the resulting state (no navigation) | -- |
| "Refund as goodwill instead" | Pro Booking Management's goodwill refund flow, scoped to this booking | FEAT-30 (Pro Booking Management) |

## Data Model

**Creates:** None.
**Reads:** Booking -- client name, start_time, state, policy_version (to determine which prompt state to show and to preview the deposit outcome). Deposit Transaction -- status and outcome_reason/timestamps (to show the current kept-outcome and compute remaining grace-window time).
**Updates:** None directly -- all Booking and Deposit Transaction transitions are written by FEAT-11.SPEC-002 and FEAT-11.SPEC-003, which this screen triggers.
**Deletes:** None.

## Business Rules

- The prompt shows exactly one of its three states (Mark, Undo, or Grace window elapsed) at a time, determined entirely by the Booking's state and the Deposit Transaction's outcome timestamp evaluated against FEAT-11.SPEC-004's window rules -- never a manual toggle.
- Eligibility is re-checked at prompt open and again at confirm (FEAT-11.SPEC-004), so a window that closes while the prompt sits open is caught before any write is attempted.
- A failed mark or undo write is retried by the triggering automation and, if it still cannot complete, flagged to Talia rather than silently dropped (per the Brief's Side-Effect Inventory) -- this screen surfaces that flag as the Error state rather than pretending the action succeeded.
- XBR-12 bounds this screen's Undo state to a fixed 24-hour window (platform parameter: `no-show-undo-grace-window-hours`) and its Mark state to the span between the booking's start_time and its 7-day auto-completion boundary (platform parameter: `booking-auto-completion-window-days`, owned by FEAT-12).

## Edge Cases

- **Talia taps "Mark no-show" twice rapidly** -- The second tap is ignored while the first confirm is in flight (button in loading state); only one write is attempted.
- **The undo grace window elapses while the prompt is open in the Undo state** -- The prompt's remaining-time line reaches zero and the prompt switches in place to the Grace window elapsed state without requiring Talia to close and reopen it.
- **Talia reopens the prompt for a booking that was marked no-show, then undone, from a different device in the meantime** -- Concurrent-edit conflict: the Booking and Deposit Transaction are read fresh on every prompt open, so the prompt reflects the current true state (Mark, not Undo) rather than a stale cached one; no separate conflict dialog is needed since this screen never holds an editable draft, only a fresh read-then-confirm action. Resolution consistent with the dependency map's reject-with-refresh Contention rule for Booking and Deposit Transaction: if Talia's confirm targets a state that has already changed (e.g., she confirms "Mark no-show" on a booking that was just cancelled by an expiring hold or another action), the write is rejected and she sees "This booking's state has changed. " followed by the exact current state, with no partial write applied.
- **Talia taps "Mark no-show" then immediately closes the prompt before the confirm completes** -- The confirm already in flight completes regardless of the prompt being closed; the next time Talia opens this booking's prompt (or views the dashboard), it reflects the outcome of that completed write.
- **Network failure during confirm** -- Error state: "Could not complete this action. Check your connection and try again." with the booking's true state re-read and re-displayed; no partial Booking or Deposit Transaction change is left behind (FEAT-11.SPEC-002 / FEAT-11.SPEC-003 own atomicity).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound) | Talia arrives here by tapping "no-show" on a past-due booking row |
| FEAT-11.SPEC-004 (No-Show Marking Window & Authorization Rules) | References (inbound) | Governs which prompt state shows, the window checks re-run on open and confirm, and the exact denied messages |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggers (outbound) | "Mark no-show" confirm triggers this automation |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | Triggers (outbound) | "Undo no-show" confirm triggers this automation |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | "Refund as goodwill instead" hands off to FEAT-30's goodwill refund flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| no_show_prompt_opened | prompt_state (mark / undo / elapsed) | Prompt opens from the dashboard | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_marked | time_since_start_time | Talia confirms "Mark no-show" and the write succeeds | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_mark_undone | time_since_marked | Talia confirms "Undo no-show" and the write succeeds | supports success-metrics.md: "No-Show Recovery Rate" |
| no_show_goodwill_redirect | -- | Talia taps "Refund as goodwill instead" | N/A -- goodwill refund correctness is measured by FEAT-30's own success-metrics.md connection ("Pro Change Correctness"), not by this feature's No-Show Recovery Rate, which measures the no-show path specifically |
| no_show_action_failed | action (mark / undo), reason (window_closed / write_failed / offline) | The confirm is rejected by re-validation or the triggered automation reports an unrecoverable failure | supports success-metrics.md: "No-Show Recovery Rate" (a rate target of zero manual chasing requires visibility into every case the automatic path could not complete cleanly) |

## Acceptance Criteria

**FEAT-11.SPEC-001-AC-01:** Given Talia is viewing her dashboard (FEAT-12) and a booking's appointment start time has passed with no outcome marked yet, when she taps "no-show" on that row, then this prompt opens in the Mark state showing the client's name, appointment time, and the kept-deposit outcome preview.

**FEAT-11.SPEC-001-AC-02:** Given Talia is on the Mark state of this prompt, when she taps "Mark no-show," then FEAT-11.SPEC-004 re-validates eligibility, FEAT-11.SPEC-002 runs, and on success the prompt switches in place to the Undo state showing the deposit as kept.

**FEAT-11.SPEC-001-AC-03:** Given Talia is on the Mark state of this prompt, when she taps "Refund as goodwill instead," then the prompt closes and she is taken to Pro Booking Management's (FEAT-30) goodwill refund flow scoped to this booking.

**FEAT-11.SPEC-001-AC-04:** Given Talia is on the Mark state of this prompt, when she taps "Cancel," then the prompt closes with no change and she returns to the dashboard.

**FEAT-11.SPEC-001-AC-05:** Given Talia marked a booking as a no-show 3 hours ago, when she taps "no-show" on that row again, then this prompt opens in the Undo state showing the kept outcome and roughly 21 hours remaining in the undo grace window (platform parameter: `no-show-undo-grace-window-hours`).

**FEAT-11.SPEC-001-AC-06:** Given Talia is on the Undo state of this prompt, when she taps "Undo no-show," then FEAT-11.SPEC-004 re-validates the grace window, FEAT-11.SPEC-003 runs, and on success the prompt switches in place to the Mark state, reflecting the booking as Completed again with the deposit restored.

**FEAT-11.SPEC-001-AC-07:** Given Talia is on the Undo state of this prompt, when she taps "Keep as no-show," then the prompt closes with no change.

**FEAT-11.SPEC-001-AC-08:** Given Talia marked a booking as a no-show more than 24 hours ago (platform parameter: `no-show-undo-grace-window-hours` has elapsed), when she taps "no-show" on that row, then this prompt opens in the Grace window elapsed state showing "This no-show mark can no longer be undone." with only a Close action.

**FEAT-11.SPEC-001-AC-09:** Given Talia is on the Undo state with 10 minutes remaining in the grace window, when the window elapses while the prompt stays open, then the prompt switches in place to the Grace window elapsed state without requiring her to reopen it.

**FEAT-11.SPEC-001-AC-10:** Given Talia taps "Mark no-show" and the booking's state changed since the prompt opened (e.g., it was cancelled through another action in the meantime), when the re-validation in FEAT-11.SPEC-004 runs, then no write occurs and Talia sees the exact denied message stating the booking's current state.

**FEAT-11.SPEC-001-AC-11:** Given Talia loses connectivity while this prompt is open, when she looks at the action row, then it is replaced by the banner "You're offline. Marking or undoing a no-show needs a connection." and no action can be confirmed until connectivity returns.

**FEAT-11.SPEC-001-AC-12:** Given Talia taps "Mark no-show" and the write cannot complete after the automation's retry, when the failure is reported back to this prompt, then she sees an error state describing the failure rather than a silent success, and the booking's true state is re-read and shown.

**FEAT-11.SPEC-001-AC-13:** Given Talia taps "Mark no-show" twice in rapid succession, when the second tap registers, then it is ignored while the first confirm is in flight, and only one write is attempted.

**FEAT-11.SPEC-001-AC-14:** Given Riley (the Client) has no path that reaches this prompt, when she views her own booking history after a no-show mark, then she sees only the resulting deposit-kept outcome there (FEAT-16), never this prompt or its actions.

**FEAT-11.SPEC-001-AC-15:** Given Talia taps "no-show" on a past-due booking row, when the prompt opens and the fresh Booking and Deposit Transaction read has not yet resolved, then the prompt shows the Loading state (header and close control visible, skeleton in place of the booking identity, outcome line, and action row) rather than an empty or blank surface, and it renders directly into the Mark, Undo, or Grace window elapsed state once the read resolves.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loading, empty, mark, undo, elapsed, confirming, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

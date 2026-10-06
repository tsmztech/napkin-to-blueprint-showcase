---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-001
spec_name: Cancel Booking
spec_slug: cancel-booking
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Cancel Booking

## Overview

**Name:** Cancel Booking
**ID:** FEAT-10.SPEC-001
**Type:** Screen
**Purpose:** Client views the cancellation window countdown and deposit outcome for their own upcoming booking and confirms or backs out of cancelling it.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- The default landing screen when a client chooses to act on a booking from FEAT-06.SPEC-004 ("Cancel or Reschedule")
- Showing the cancellation window countdown and the deposit outcome preview (refund vs. kept) before the client commits to cancelling
- The explicit confirm step and a no-penalty way to back out with nothing changed
- Offering the reschedule path as an alternative to cancelling, without leaving the client stuck choosing wrong
- Blocking the action entirely, with a plain message, when the booking is no longer eligible (already Completed or No-Show)

**Non-Goals:**
- Computing the cancellation window countdown and eligibility itself -- owned by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule); this screen only displays what that rule returns
- Deriving the deposit outcome value -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this screen reads and displays FEAT-09.SPEC-003's rule result, never computes it
- Committing the cancellation to the Booking record -- owned by FEAT-10.SPEC-004 (Booking Update Commit), which this screen triggers but does not implement
- Selecting a new time -- owned by FEAT-10.SPEC-002 (Reschedule -- Select New Time), reached via this screen's "Reschedule instead" link

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps "Cancel or Reschedule" | The selected Booking's reference |
| FEAT-10.SPEC-004 (Booking Update Commit) | Cancellation fails to save, or a conflicting Pro-side transition wins first | Same Booking reference, refreshed to its current state; an error or conflict message |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client backs out of a late-reschedule confirmation and instead chooses to cancel from that screen's "or cancel instead" link | Same Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, for a booking that belongs to their own matched Client record with this Pro | Confirm cancel, back out, navigate to reschedule instead | -- |
| The Pro (Talia) | No | No | This is not the Pro's own change surface; the Pro's own cancel/reschedule actions are through Pro Booking Management (FEAT-30), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses a client's access link (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-06.SPEC-004's already-authenticated navigation; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The underlying access link governs the viewing session (FEAT-06); once it has transitioned to Used or Expired, reloading this screen is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Cancel Booking" with a back arrow (returns to FEAT-06.SPEC-004).

**Body:**
- Appointment summary: service name, date, time, duration
- Cancellation window countdown: plain-language statement of how much time remains before the booking enters the cancellation window (or that it has already passed), computed by FEAT-10.SPEC-005
- Deposit outcome preview: a plain statement of what happens to the deposit if the client cancels right now -- "Your deposit will be refunded" (outside the window) or "Your deposit will be kept, per the cancellation policy you agreed to" (inside the window), derived from FEAT-09.SPEC-003
- Policy wording: the exact plain-language cancellation policy text acknowledged at booking (FEAT-09.SPEC-002)
- "Cancel Booking" button (primary action)
- "Reschedule instead" link, below the Cancel button
- "Keep my booking" link, returning to FEAT-06.SPEC-004 with nothing changed

Appointment summary, cancellation window countdown, deposit outcome preview, and policy wording are display-only text with no interaction of their own; only the back arrow, "Cancel Booking," "Reschedule instead," and "Keep my booking" are interactive elements on this screen.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; "Cancel Booking" and "Reschedule instead" are full-width, stacked.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-004 | Screen closes | Standard navigation transition |
| "Cancel Booking" button | Tap | Opens the confirm dialog showing the same deposit outcome preview one more time | Dialog appears | Dialog title "Cancel this booking?" with the deposit outcome restated, and "Confirm Cancellation" / "Keep Booking" options |
| "Confirm Cancellation" (in dialog) | Tap | Triggers FEAT-10.SPEC-004 (Booking Update Commit) for a cancellation | Button shows loading state; dialog stays open during commit | Success: navigate to FEAT-06.SPEC-004, showing the booking's now-Cancelled state. Failure: dialog shows the error and a Retry option (see States, Error) |
| "Keep Booking" (in dialog) | Tap | Closes the dialog; nothing changes | Dialog closes | Client returns to this screen exactly as before |
| "Reschedule instead" link | Tap | Navigate to FEAT-10.SPEC-002 (Reschedule -- Select New Time) for this Booking | Screen transitions | Standard navigation transition |
| "Keep my booking" link | Tap | Navigate to FEAT-06.SPEC-004 | Screen closes | Standard navigation transition |
| "Confirm Cancellation" (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> appointment summary -> cancellation window countdown -> deposit outcome preview -> policy wording -> "Cancel Booking" -> "Reschedule instead" -> "Keep my booking".
- **Dynamic announcements:** The confirm dialog's appearance and its deposit outcome restatement are announced to assistive technology when it opens; the concurrent-edit conflict message (see Edge Cases) is announced as soon as it appears; a commit failure's error text is announced and focus moves to the Retry action.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the countdown and outcome preview will appear | Screen first opens | Data (booking, window, outcome preview) finishes loading |
| Populated | Full countdown, outcome preview, and policy wording shown as described in Layout and Content | Data loads successfully and the booking is eligible | Client navigates away or taps Cancel Booking |
| Ineligible | Plain message "This booking can no longer be cancelled or rescheduled." replaces the Cancel/Reschedule actions; appointment summary and status remain visible | FEAT-10.SPEC-005 reports the booking as ineligible (state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Pending Payment, or Expired (unpaid) -- the full ineligibility scope defined by FEAT-10.SPEC-005 AC-05 and AC-14) | Client navigates back to FEAT-06.SPEC-004 (no path forward on this screen) |
| Confirming | Confirm dialog open, showing the restated outcome | Client taps "Cancel Booking" | Client taps "Confirm Cancellation" or "Keep Booking" |
| Cancelling | "Confirm Cancellation" button shows a loading state; dialog remains open and non-dismissible | Client taps "Confirm Cancellation" | Commit succeeds or fails |
| Error | Dialog shows "We couldn't cancel this booking. Try again." with a Retry action; the original booking remains untouched and intact | FEAT-10.SPEC-004 reports the commit failed to save | Client taps Retry and the commit succeeds, or navigates away |
| Load Error | Error banner "We couldn't load this booking. Try again." with a retry action | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "Connectivity is required to cancel or reschedule a booking." appears; the countdown and outcome preview (if already loaded) remain visible read-only; "Cancel Booking" and "Reschedule instead" are disabled | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule). See that spec for the eligibility gate this screen enforces before offering the Cancel action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| Successful cancellation | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| "Reschedule instead" tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| "Keep my booking" tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, state, policy_version, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Deposit Transaction -- amount, for the outcome preview. Cancellation Policy -- window_hours and plain_language_wording, via the bound version (FEAT-09.SPEC-002).
**Updates:** None directly -- the cancellation itself is performed by FEAT-10.SPEC-004, which this screen triggers.
**Deletes:** None.

## Business Rules

- FEAT-10.SPEC-005 governs whether this booking is currently eligible to be cancelled and computes the window countdown shown here; this screen enforces its result but never re-derives it.
- The deposit outcome preview reflects FEAT-09.SPEC-003's Rule 1 (outside window: Refund Due) or Rule 2 (inside window: Forfeiture Flagged), read live at the moment this screen loads.
- XBR-12: a Completed or No-Show booking can never be cancelled from this screen; the Ineligible state applies instead.
- "See the outcome before confirming" pattern: this screen shows the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed, consistent with FEAT-10.SPEC-003's identical pattern for a late reschedule.

## Edge Cases

- **The Pro cancels, reschedules, or marks this booking no-show while Riley is viewing this screen** -- Per the Booking entity's Contention resolution (reject-with-refresh, feature-dependency-map.md), if Riley then taps "Confirm Cancellation," the commit is rejected because the Pro's transition already committed first; Riley sees the dialog "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action that reloads the booking's current state -- her original tap is never carried through against stale data.
- **Riley taps "Cancel Booking" twice rapidly** -- The second tap is ignored while the confirm dialog is already open.
- **Riley taps "Confirm Cancellation" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **The countdown crosses from outside to inside the window while this screen is open** -- The screen is a snapshot view; the countdown and outcome preview do not silently update mid-view. The values shown reflect the moment the screen loaded; the values used at commit time are recomputed fresh by FEAT-10.SPEC-004/FEAT-10.SPEC-005 at the instant of the confirm tap, so the actual outcome applied is always current even if the displayed preview was taken moments earlier. If the recomputed outcome at commit time differs from what was shown, the confirm dialog is re-shown once with the updated outcome before the commit proceeds, rather than silently applying a different outcome than the client just confirmed.
- **Riley navigates away with the confirm dialog open and returns later** -- The dialog does not persist; the screen reloads fresh data as on any new visit.
- **Riley loses connectivity while the confirm dialog is open** -- The dialog closes and the Offline/Degraded banner appears; no cancellation is submitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound/outbound) | Entry point into this screen and the destination on back/cancel/keep |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (inbound) | Supplies the window countdown and eligibility gate |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (inbound) | Supplies the deposit outcome preview |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (inbound) | Supplies the rendered cutoff time and plain-language wording |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggers (outbound) | "Confirm Cancellation" triggers the commit |
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Navigation (outbound) | "Reschedule instead" starts that flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancel_screen_viewed | window_state (outside/inside), outcome_preview (refund/kept) | Screen finishes loading with data | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_confirmed | window_state, outcome | Client confirms the cancellation and the commit succeeds | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_abandoned | reason (kept_booking / rescheduled_instead / navigated_away) | Client leaves this screen without cancelling | supports success-metrics.md: "Self-Service Reschedule Rate" |
| cancel_screen_conflict_shown | -- | Riley's confirm is rejected because a Pro-side change committed first | supports success-metrics.md: "Self-Service Reschedule Rate" |

## Acceptance Criteria

**FEAT-10.SPEC-001-AC-01:** Given Riley opens Cancel Booking for an upcoming booking outside the cancellation window, when the screen loads, then she sees the countdown, "Your deposit will be refunded" as the outcome preview, and the acknowledged policy wording.

**FEAT-10.SPEC-001-AC-02:** Given Riley opens Cancel Booking for a booking inside the cancellation window, when the screen loads, then she sees "Your deposit will be kept, per the cancellation policy you agreed to" as the outcome preview.

**FEAT-10.SPEC-001-AC-03:** Given Riley taps "Cancel Booking," when the confirm dialog opens, then it restates the same deposit outcome she saw on the screen.

**FEAT-10.SPEC-001-AC-04:** Given Riley taps "Confirm Cancellation" in the dialog, when the commit succeeds, then FEAT-10.SPEC-004 records the cancellation and she is returned to FEAT-06.SPEC-004 showing the booking as Cancelled.

**FEAT-10.SPEC-001-AC-05:** Given Riley taps "Keep Booking" in the dialog, when the tap registers, then the dialog closes and nothing about the booking changes.

**FEAT-10.SPEC-001-AC-06:** Given Riley taps "Reschedule instead," when the tap registers, then she is taken to FEAT-10.SPEC-002 for the same booking.

**FEAT-10.SPEC-001-AC-07:** Given Riley opens Cancel Booking for a booking already marked Completed, when the screen loads, then it shows the Ineligible state with the message "This booking can no longer be cancelled or rescheduled." and no Cancel or Reschedule action is offered.

**FEAT-10.SPEC-001-AC-08:** Given Riley opens Cancel Booking for a booking already marked No-Show, when the screen loads, then it shows the same Ineligible state.

**FEAT-10.SPEC-001-AC-09:** Given the initial data load for this screen fails, when the failure occurs, then the error banner "We couldn't load this booking. Try again." appears with a retry action.

**FEAT-10.SPEC-001-AC-10:** Given Riley taps "Confirm Cancellation" and the commit fails to save, when the failure occurs, then the dialog shows "We couldn't cancel this booking. Try again." with a Retry option, and the original booking remains untouched.

**FEAT-10.SPEC-001-AC-11:** Given Talia cancels this same booking on her side while Riley is viewing this screen, when Riley then taps "Confirm Cancellation," then her commit is rejected with the dialog "This booking's details changed. Refresh to see the latest before continuing." and no stale cancellation is recorded.

**FEAT-10.SPEC-001-AC-12:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the banner "Connectivity is required to cancel or reschedule a booking." appears and "Cancel Booking" and "Reschedule instead" are disabled.

**FEAT-10.SPEC-001-AC-13:** Given Riley taps "Cancel Booking" twice in rapid succession, when the second tap registers, then it is ignored because the confirm dialog is already open.

**FEAT-10.SPEC-001-AC-14:** Given Riley taps "Confirm Cancellation" twice in rapid succession, when the second tap registers, then it is ignored while the first commit is in progress.

**FEAT-10.SPEC-001-AC-15:** Given a client without a valid access link attempts to reach this screen directly, when the attempt is made, then they are redirected to FEAT-06.SPEC-001 and shown no booking data.

**FEAT-10.SPEC-001-AC-16:** Given Riley taps the back arrow, when the tap registers, then she returns to FEAT-06.SPEC-004 and nothing about the booking has changed.

**FEAT-10.SPEC-001-AC-17:** Given Riley taps "Keep my booking," when the tap registers, then she returns to FEAT-06.SPEC-004 and nothing about the booking has changed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loading, populated, ineligible, confirming, cancelling, error, load error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

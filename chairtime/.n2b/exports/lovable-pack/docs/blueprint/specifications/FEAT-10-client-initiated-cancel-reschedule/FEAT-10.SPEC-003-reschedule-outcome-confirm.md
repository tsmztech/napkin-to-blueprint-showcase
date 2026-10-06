---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-003
spec_name: Reschedule -- Outcome & Confirm
spec_slug: reschedule-outcome-confirm
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Reschedule -- Outcome & Confirm

## Overview

**Name:** Reschedule -- Outcome & Confirm
**ID:** FEAT-10.SPEC-003
**Type:** Screen
**Purpose:** Client sees the deposit outcome for the chosen new time (carried-over deposit, or late-reschedule deposit-kept-plus-new-deposit-needed) before confirming.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- Showing the deposit outcome for the chosen new time before the client confirms: carried-over deposit (outside the window) or the compound late-reschedule outcome (inside the window)
- The applicable cancellation window countdown, computed against the original booking's start_time
- The explicit confirm step and a no-penalty way to back out with nothing changed
- Committing the reschedule once confirmed

**Non-Goals:**
- Selecting the new time itself -- owned by FEAT-10.SPEC-002 (Reschedule -- Select New Time), the previous step
- Deriving the deposit outcome value -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, FEAT-09.SPEC-003); this screen reads and displays that rule set's result, never computes it
- Collecting the new deposit payment for a late reschedule -- handed off to FEAT-07 (Deposit Payment at Booking), the standard deposit-payment mechanism, once this screen's confirm step completes
- Committing the reschedule to the Booking record -- owned by FEAT-10.SPEC-004 (Booking Update Commit), which this screen triggers but does not implement

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Client's chosen time is re-validated as still free | The Booking reference, the chosen new time |
| FEAT-10.SPEC-004 (Booking Update Commit) | Reschedule fails to save, or a conflicting Pro-side transition wins first | Same Booking reference and chosen time, refreshed to current state; an error or conflict message |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, for a booking that belongs to their own matched Client record with this Pro | Confirm the reschedule, back out to slot selection, or (on a late reschedule) cancel instead | -- |
| The Pro (Talia) | No | No | This is not the Pro's own change surface; the Pro's own reschedule is through Pro Booking Management (FEAT-30), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses a client's access link (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-10.SPEC-002's already-authenticated navigation; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The underlying access link governs the viewing session (FEAT-06); once it has transitioned to Used or Expired, reloading this screen is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Confirm Reschedule" with a back arrow (returns to FEAT-10.SPEC-002).

**Body:**
- Appointment summary: service name, the original date/time struck through or clearly marked "current," and the new chosen date/time
- Cancellation window countdown: plain-language statement of how much time remains before the original booking would have entered the cancellation window (computed against the original start_time, per FEAT-10.SPEC-005)
- Deposit outcome, one of two variants:
  - **Outside the window:** "Your deposit carries over -- nothing further to pay now."
  - **Inside the window (late reschedule):** "Your original deposit is kept, per the cancellation policy you agreed to, and a new deposit is needed for this new time." followed by the new deposit amount
- "Confirm Reschedule" button (primary action)
- "Choose a different time" link, returning to FEAT-10.SPEC-002
- On a late reschedule only: "Cancel instead" link

Appointment summary and the cancellation window countdown are display-only text with no interaction of their own; only the back arrow, "Confirm Reschedule," "Choose a different time," and "Cancel instead" (when shown) are interactive elements on this screen.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; "Confirm Reschedule" is full-width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-10.SPEC-002 | Screen closes | Standard backward transition |
| "Confirm Reschedule" button | Tap | Triggers FEAT-10.SPEC-004 (Booking Update Commit) for a reschedule | Button shows loading state | Success (outside window): navigate to FEAT-06.SPEC-004 showing the updated time. Success (inside window): navigate into FEAT-07's deposit-payment step for the new booking's fresh deposit. Failure: error message and Retry option (see States, Error) |
| "Choose a different time" link | Tap | Navigate to FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Screen transitions | Standard navigation transition; nothing changes |
| "Cancel instead" link (late reschedule only) | Tap | Navigate to FEAT-10.SPEC-001 (Cancel Booking) for the original booking | Screen transitions | Standard navigation transition; nothing changes |
| "Confirm Reschedule" (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> appointment summary -> cancellation window countdown -> deposit outcome -> "Confirm Reschedule" -> "Choose a different time" -> "Cancel instead" (when shown).
- **Dynamic announcements:** The deposit outcome variant is announced on screen load; the concurrent-edit conflict message and any "just taken" recovery message are announced as soon as they appear; a commit failure's error text is announced and focus moves to the Retry action.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the outcome will appear | Screen first opens | Outcome computation finishes loading |
| Ineligible | Plain message "This booking can no longer be cancelled or rescheduled." replaces the deposit outcome and Confirm action; the appointment summary remains visible | FEAT-10.SPEC-005 reports the booking as ineligible (state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Pending Payment, or Expired (unpaid)) on this screen's load -- covering the case where the booking became ineligible between FEAT-10.SPEC-002's slot selection and this screen's load | Client navigates back to FEAT-10.SPEC-002 (no path forward on this screen) |
| Populated -- Outside Window | Carried-over deposit outcome shown | Loaded outcome is Rule 5 (no change, carryover) | Client confirms or navigates away |
| Populated -- Inside Window (Late Reschedule) | Compound outcome shown: original deposit kept plus new deposit needed, with the "Cancel instead" link visible | Loaded outcome is Rule 6 (compound outcome) | Client confirms or navigates away |
| Confirming | "Confirm Reschedule" button shows a loading state | Client taps "Confirm Reschedule" | Commit succeeds or fails |
| Error | Error banner "We couldn't reschedule this booking. Try again." with a Retry option; the original booking remains untouched and intact | FEAT-10.SPEC-004 reports the commit failed to save | Client taps Retry and the commit succeeds, or navigates away |
| Slot No Longer Available | Plain message "That time was just taken. Choose another." replaces the confirm action | The chosen new time is re-validated at commit time and found no longer free | Client taps "Choose a different time," returning to a refreshed FEAT-10.SPEC-002 |
| Load Error | Error banner "We couldn't load the reschedule outcome. Try again." with a retry action | The initial outcome computation fails to load | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "Connectivity is required to reschedule a booking." appears; the loaded outcome (if any) remains visible read-only; "Confirm Reschedule" is disabled | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and "Confirm Reschedule" re-enables |

## Validation Rules

Validation governed by FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule), which this screen enforces at two points: on screen load (gating whether the outcome and Confirm action are shown at all, per the Ineligible state) and again at the moment of confirm (the authoritative re-check performed by FEAT-10.SPEC-004). FEAT-09.SPEC-003 (Deposit Outcome Rules) governs which outcome branch applies once eligibility passes. The chosen slot is re-validated one final time at the moment of commit (FEAT-10.SPEC-004), per FEAT-03's slot rules, to guard against the time being taken between FEAT-10.SPEC-002's selection and this screen's confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| Successful reschedule, outside window | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 |
| Successful reschedule, inside window (late reschedule) | Deposit payment step for the new booking | FEAT-07 (Deposit Payment at Booking) |
| "Choose a different time" tap | FEAT-10.SPEC-002 (Reschedule -- Select New Time) | -- |
| "Cancel instead" tap (late reschedule only) | FEAT-10.SPEC-001 (Cancel Booking) | -- |

## Data Model

**Creates:** None on this screen directly -- a new Booking record for a late reschedule is created by FEAT-10.SPEC-004 at commit time, not here.
**Reads:** Booking -- service, original start_time, state, policy_version, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Deposit Transaction -- amount, for the outcome preview. Cancellation Policy -- window_hours and plain_language_wording, via the bound version (FEAT-09.SPEC-002).
**Updates:** None directly -- the reschedule itself is performed by FEAT-10.SPEC-004, which this screen triggers.
**Deletes:** None.

## Business Rules

- FEAT-10.SPEC-005 governs the window comparison used here, computed against the Booking's *original* start_time (not the newly chosen time), per FEAT-09.SPEC-003's Rule 5/Rule 6 definitions.
- FEAT-10.SPEC-005 also governs whether this screen is reachable at all: eligibility is checked on this screen's load (the Ineligible state applies if the booking became Completed, No-Show, or otherwise ineligible since FEAT-10.SPEC-002's slot selection) and re-checked again at the instant of confirm by FEAT-10.SPEC-004, consistent with FEAT-10.SPEC-005's "Enforced By" table naming both points for this screen.
- Outside the window: FEAT-09.SPEC-003 Rule 5 applies -- the existing deposit carries over to the new appointment time; no new deposit charge.
- Inside the window: FEAT-09.SPEC-003 Rule 6 applies -- the original deposit's disposition becomes Forfeiture Flagged (treated as a late cancellation) and the newly created Booking requires its own fresh deposit, shown to the client before they confirm; both halves of this compound outcome are always shown together, per the Brief's Cross-Field Rules ("Reschedule-inside-window compound outcome").
- "See the outcome before confirming" pattern: this screen shows the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed, consistent with FEAT-10.SPEC-001's identical pattern for a cancellation.
- XBR-01: the chosen slot is re-validated one final time at commit, since availability can change between selection (FEAT-10.SPEC-002) and confirm (this screen).

## Edge Cases

- **The Pro cancels, reschedules, or marks this booking no-show while Riley is viewing this screen** -- Per the Booking entity's Contention resolution (reject-with-refresh), if Riley then taps "Confirm Reschedule," the commit is rejected because the Pro's transition already committed first; Riley sees the dialog "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action, and her original tap is never carried through against stale data.
- **The chosen new time is taken by another client between FEAT-10.SPEC-002's re-validation and this screen's confirm tap** -- The commit attempt reports the slot is no longer free; the screen shows the "Slot No Longer Available" state with "That time was just taken. Choose another." and a link back to a refreshed FEAT-10.SPEC-002, per XBR-01's "never a payment error" guarantee.
- **Riley taps "Confirm Reschedule" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **Riley taps "Cancel instead" on a late reschedule** -- Nothing about the reschedule attempt is committed; she is taken to FEAT-10.SPEC-001 to evaluate cancelling the original booking on its own terms.
- **The window boundary is crossed between screen load and confirm tap (e.g., the client sits on this screen past the exact cutoff moment)** -- The outcome used is the one computed fresh at the moment of commit (FEAT-10.SPEC-004/FEAT-10.SPEC-005), not the one displayed when the screen first loaded; if the recomputed outcome differs from what was shown, the confirm proceeds once with the updated outcome re-shown for one additional explicit confirm, rather than silently applying a different outcome than the client saw.
- **Riley loses connectivity while viewing this screen** -- The Offline/Degraded banner appears and "Confirm Reschedule" is disabled; no reschedule is submitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | Navigation (inbound/outbound) | Entry point into this screen; "Choose a different time" returns there |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (inbound) | Supplies the window comparison against the original start_time |
| FEAT-09.SPEC-003 (Deposit Outcome Rules) | References (inbound) | Supplies which outcome branch (Rule 5 or Rule 6) applies |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggers (outbound) | "Confirm Reschedule" triggers the commit |
| FEAT-10.SPEC-001 (Cancel Booking) | Navigation (outbound) | "Cancel instead" (late reschedule only) starts that flow |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | A late reschedule's new deposit is collected here after confirm |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| late_reschedule_warning_shown | -- | Screen loads showing the inside-window compound outcome | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_outcome_viewed | window_state (outside/inside) | Screen finishes loading with data | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_confirmed | window_state | Client confirms and the commit succeeds | supports success-metrics.md: "Self-Service Reschedule Rate" |
| reschedule_abandoned | reason (chose_different_time / cancelled_instead / navigated_away) | Client leaves this screen without confirming | supports success-metrics.md: "Self-Service Reschedule Rate" |

## Acceptance Criteria

**FEAT-10.SPEC-003-AC-01:** Given Riley picks a time outside the cancellation window, when this screen loads, then she sees "Your deposit carries over -- nothing further to pay now."

**FEAT-10.SPEC-003-AC-02:** Given Riley picks a time that would put her reschedule inside the cancellation window, when this screen loads, then she sees both halves of the compound outcome together: the original deposit is kept, and a new deposit is needed.

**FEAT-10.SPEC-003-AC-03:** Given Riley is on the inside-window outcome, when she looks for a way to back out without penalty, then a "Cancel instead" link is shown alongside "Choose a different time."

**FEAT-10.SPEC-003-AC-04:** Given Riley taps "Confirm Reschedule" on the outside-window outcome, when the commit succeeds, then FEAT-10.SPEC-004 updates the booking to the new time and she lands on FEAT-06.SPEC-004 showing it.

**FEAT-10.SPEC-003-AC-05:** Given Riley taps "Confirm Reschedule" on the inside-window outcome, when the commit succeeds, then the original booking's deposit is flagged forfeited and she is taken into FEAT-07's deposit-payment step for the new booking's fresh deposit.

**FEAT-10.SPEC-003-AC-06:** Given Riley taps "Choose a different time," when the tap registers, then she returns to FEAT-10.SPEC-002 and nothing about the original booking has changed.

**FEAT-10.SPEC-003-AC-07:** Given Riley taps "Cancel instead" on the inside-window outcome, when the tap registers, then she is taken to FEAT-10.SPEC-001 and no reschedule was committed.

**FEAT-10.SPEC-003-AC-08:** Given the chosen new time is taken by another client between selection and confirm, when Riley taps "Confirm Reschedule," then she sees "That time was just taken. Choose another." and is returned to a refreshed FEAT-10.SPEC-002, never a payment error.

**FEAT-10.SPEC-003-AC-09:** Given Talia cancels this booking on her side while Riley is viewing this screen, when Riley then taps "Confirm Reschedule," then her commit is rejected with "This booking's details changed. Refresh to see the latest before continuing."

**FEAT-10.SPEC-003-AC-10:** Given Riley taps "Confirm Reschedule" and the commit fails to save for a reason other than slot contention, when the failure occurs, then the error banner "We couldn't reschedule this booking. Try again." appears with a Retry option, and the original booking remains untouched.

**FEAT-10.SPEC-003-AC-11:** Given the initial outcome computation fails to load, when the failure occurs, then the error banner "We couldn't load the reschedule outcome. Try again." appears with a retry action.

**FEAT-10.SPEC-003-AC-12:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the banner "Connectivity is required to reschedule a booking." appears and "Confirm Reschedule" is disabled.

**FEAT-10.SPEC-003-AC-13:** Given Riley taps "Confirm Reschedule" twice in rapid succession, when the second tap registers, then it is ignored while the first commit is in progress.

**FEAT-10.SPEC-003-AC-14:** Given Riley reaches exactly the cutoff moment while this screen is displayed, when she taps "Confirm Reschedule," then the outcome applied is the one recomputed at that exact moment, treated as outside the window per the inclusive-boundary rule (FEAT-10.SPEC-005).

**FEAT-10.SPEC-003-AC-15:** Given a client without a valid access link attempts to reach this screen directly, when the attempt is made, then they are redirected to FEAT-06.SPEC-001 and shown no outcome data.

**FEAT-10.SPEC-003-AC-16:** Given the booking becomes ineligible (e.g., already Completed, No-Show, or transitioned by the Pro) between Riley's slot selection on FEAT-10.SPEC-002 and this screen's load, when this screen loads, then it shows the Ineligible state with "This booking can no longer be cancelled or rescheduled." and no deposit outcome or Confirm action is offered.

**FEAT-10.SPEC-003-AC-17:** Given Riley taps the back arrow, when the tap registers, then she returns to FEAT-10.SPEC-002 and nothing about the booking has changed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 9 (loading, ineligible, populated-outside, populated-inside, confirming, error, slot-no-longer-available, load error, offline) | 9 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-003
spec_name: Time Block Conflict Review
spec_slug: time-block-conflict-review
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Time Block Conflict Review

## Overview

**Name:** Time Block Conflict Review
**ID:** FEAT-17.SPEC-003
**Type:** Screen
**Purpose:** Talia sees every confirmed booking a new or edited Time Block conflicts with and chooses, per booking, to cancel it, reschedule it, or keep the block with that booking as an exception, before the block commits.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Listing every confirmed booking that conflicts with the block Talia just tried to save
- Capturing Talia's per-booking choice: cancel, reschedule, or keep as an exception
- Showing the consequence of each choice before she confirms, matching the "see the outcome before confirming" pattern shared with FEAT-10 and FEAT-30
- Submitting the confirmed set of choices to FEAT-17.SPEC-006 for commit

**Non-Goals:**
- Detecting which bookings conflict -- owned by FEAT-17.SPEC-004; this screen only displays what that automation already found
- Executing the cancellation or reschedule itself -- owned by FEAT-30.SPEC-005 (bulk cancel) and FEAT-30.SPEC-002 (single reschedule), reached through FEAT-17.SPEC-006's hand-off
- Deciding which of the three choices is "correct" -- excluded per the Alternate flow's own wording and XBR-11: cancel, reschedule, and keep-as-exception are equally legitimate outcomes, and the product never defaults to one automatically
- A bulk-reschedule option for several conflicting bookings at once -- excluded per the Brief's own Non-Goals: FEAT-30 reschedules only one booking at a time, so this screen never offers a "reschedule all" choice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | A single-date block save finds one or more conflicting confirmed bookings | The pending block's values and the full set of conflicting Booking references |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | A generated recurring occurrence conflicts with a confirmed booking | The pending occurrence's values and the conflicting Booking reference(s) for that occurrence |
| FEAT-17.SPEC-001 (Create/Edit Time Block) | A save finds one or more conflicting confirmed bookings | The pending block values and the conflicting Booking references (via FEAT-17.SPEC-004) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- only reachable as part of her own save/generation flow | Choose cancel, reschedule, or keep-as-exception for each listed booking, and confirm | -- |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application, reached only mid-save; Riley has no navigation path to it and is never shown which of her bookings conflicted -- only the eventual outcome (a cancellation notice, a reschedule notice, or nothing, if kept as an exception) |
| Platform Operator (Support) | No | No | This screen is reachable only inside Talia's own active save flow, not from a read-only account review; support never opens it |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the pending block's values and conflict set are preserved and this screen is restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Review {count} conflicting bookings" with a back arrow (returns to FEAT-17.SPEC-001 without committing the block) and a "Confirm" action button (right-aligned, enabled once every listed booking has a choice).

**Body:** The pending block's own span is restated at the top ("Blocking {date/pattern}, {start}--{end}"), followed by one **row per conflicting booking**, each showing:
- Booking time, service name, and client name
- A three-way choice control: "Cancel," "Reschedule," "Keep as exception" -- exactly one selected per row, with no default pre-selected
- When "Cancel" is selected, an inline note: "Full refund, {client name} is notified."
- When "Reschedule" is selected, an inline note: "You'll pick a new time for {client name} next."
- When "Keep as exception" is selected, an inline note: "This booking stays as booked. It will be flagged on your dashboard as an exception to this block."

**Footer:** None -- Confirm is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column list of rows as described, full width.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-17.SPEC-001 without committing the block or resolving any conflict | Screen closes; the block save is abandoned | Confirmation dialog first (see Edge Cases) |
| Choice control per row | Select one of Cancel / Reschedule / Keep as exception | Captures that booking's chosen outcome | Row shows the corresponding inline consequence note | Selected choice highlighted |
| "Confirm" button | Tap (enabled once every row has a choice) | Submits the full set of per-booking choices to FEAT-17.SPEC-006 (Time Block Conflict Resolution Commit) | Button shows loading state | See Navigation Out for outcome routing |
| "Confirm" button (disabled state) | Tap while one or more rows have no choice | No action | None | Button remains visually disabled; a hint "Choose an outcome for every booking" appears |

### Accessibility Notes

- **Focus order:** Back arrow -> pending block summary -> each conflicting booking row in the order listed (within a row: booking details, choice control, consequence note) -> Confirm.
- **Dynamic-change announcements:** When a row's choice changes, its consequence note update is announced to assistive technology; when Confirm becomes enabled (every row has a choice), that state change is announced.
- **Keyboard alternatives:** Every choice control and the Confirm action are reachable and operable by keyboard.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Awaiting choices | Every row shows its three-way control with nothing selected; Confirm disabled | Screen opens with the conflicting set loaded from FEAT-17.SPEC-004 or FEAT-17.SPEC-005 | Every row receives a choice |
| Ready to confirm | Every row has a choice; Confirm enabled | The last unresolved row receives a choice | Talia taps Confirm, or changes a choice back to none (not offered -- see Edge Cases) |
| Confirming | Confirm button shows a loading indicator, choice controls disabled | Talia taps Confirm | FEAT-17.SPEC-006 returns an outcome |
| Error | Error banner: "Couldn't save your choices. Check your connection and try again." with a Retry control; all selected choices are preserved | FEAT-17.SPEC-006 reports a commit failure | Talia taps Retry and the commit succeeds, or she navigates away |
| Offline/Degraded | Plain message "Connect to the internet to confirm these choices." replaces Confirm's active state; choice controls remain usable but Confirm is disabled | Connectivity is lost while this screen is open, or it is opened while already offline | Connectivity is restored -- Confirm re-enables once every row has a choice |

## Validation Rules

Validation governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules), specifically the never-silently-affect-a-booking rule: Confirm is enabled only once every conflicting booking has an explicit choice, checked on each choice selection and again on Confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Confirm succeeds, one or more bookings chosen "Cancel" | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | FEAT-30 |
| Confirm succeeds, a booking chosen "Reschedule" | FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | FEAT-30 |
| Confirm succeeds, every remaining booking chosen "Keep as exception" | FEAT-17.SPEC-002 (Manage Time Blocks) | -- |

## Data Model

**Creates:** None directly -- this screen captures choices; FEAT-17.SPEC-006 performs the commit.
**Reads:** Booking (start_time, service, client, state) -- the full conflicting set handed over by FEAT-17.SPEC-004 or FEAT-17.SPEC-005; the pending Time Block's own start, end, and recurrence values.
**Updates:** None directly.
**Deletes:** None directly.

## Business Rules

- Confirm is never enabled while any conflicting booking has no chosen outcome -- the never-silently-affect-a-booking rule (FEAT-17.SPEC-008) is enforced at the UI level here, before FEAT-17.SPEC-006 ever runs.
- Every "Cancel" choice across the set is handed to FEAT-30.SPEC-005 together as one bulk batch; every "Reschedule" choice is handed to FEAT-30.SPEC-002 individually, one booking at a time (the Brief's own Non-Goal: no bulk-reschedule capability exists).
- A "Keep as exception" choice never alters the booking itself -- it only flags it for the Pro's own attention (XBR-11, surfaced via FEAT-12.SPEC-005).

## Edge Cases

- **Talia navigates away (back arrow) with choices already made but not confirmed** -- Confirmation dialog: "Discard these choices and the time block?" with "Discard" and "Keep Reviewing" options; discarding abandons the entire pending block, not just the choices.
- **Talia changes a row's choice after selecting one** -- Allowed at any time before Confirm; the consequence note updates immediately to match the new choice.
- **The conflicting set contains only one booking** -- The screen renders identically with a single row; the header reads "Review 1 conflicting booking."
- **A conflicting booking is cancelled or rescheduled by Talia through an unrelated path (e.g., another device) while this screen is open** -- On Confirm, FEAT-17.SPEC-006 re-validates the set against current booking state; a booking no longer confirmed is dropped from the set with a brief note, and the block commits against the remaining conflicts.
- **Talia taps Confirm twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Network failure during Confirm** -- Error banner as described in States; every selected choice is preserved and Talia can retry without re-selecting anything.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | Navigation (inbound) | Routes here when a single-date save finds conflicts |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | Navigation (inbound) | Routes here when a generated occurrence finds conflicts |
| FEAT-17.SPEC-006 (Time Block Conflict Resolution Commit) | Triggers (outbound) | Confirm submits the full set of per-booking choices |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Never-silently-affect-a-booking rule enforced here |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Navigation (outbound, via FEAT-17.SPEC-006) | Destination for bulk-cancelled bookings |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Navigation (outbound, via FEAT-17.SPEC-006) | Destination for a single rescheduled booking |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Navigation (outbound) | Destination once every conflict is resolved as an exception |

## Analytics and Success Signals

N/A -- this screen captures Talia's per-booking choices; the analytics this feature's Signals track for conflict handling (`time_block_conflict_flagged` and its resolution outcome) are emitted by the automations that detect and commit the resolution (FEAT-17.SPEC-004 and FEAT-17.SPEC-006), not by this review surface itself.

## Acceptance Criteria

**FEAT-17.SPEC-003-AC-01:** Given Talia's new block conflicts with two confirmed bookings, when FEAT-17.SPEC-004 routes her here, then she sees both bookings listed with a three-way choice control each, and Confirm disabled.

**FEAT-17.SPEC-003-AC-02:** Given Talia selects "Cancel" for a conflicting booking, when the row updates, then it shows the note "Full refund, {client name} is notified."

**FEAT-17.SPEC-003-AC-03:** Given Talia selects "Reschedule" for a conflicting booking, when the row updates, then it shows the note "You'll pick a new time for {client name} next."

**FEAT-17.SPEC-003-AC-04:** Given Talia selects "Keep as exception" for a conflicting booking, when the row updates, then it shows the note that the booking stays as booked and will be flagged on her dashboard.

**FEAT-17.SPEC-003-AC-05:** Given Talia has made a choice for every listed booking, when she looks at the header, then the Confirm button is enabled.

**FEAT-17.SPEC-003-AC-06:** Given Talia has left one booking's choice unselected, when she taps the disabled Confirm button, then nothing happens and the hint "Choose an outcome for every booking" appears.

**FEAT-17.SPEC-003-AC-07:** Given Talia has chosen "Cancel" for one booking and "Reschedule" for another and taps Confirm, when FEAT-17.SPEC-006 commits, then she is routed to FEAT-30.SPEC-005 for the cancelled booking's bulk-cancel review.

**FEAT-17.SPEC-003-AC-08:** Given Talia has chosen "Keep as exception" for every conflicting booking and taps Confirm, when FEAT-17.SPEC-006 commits, then the block is created and she is returned to FEAT-17.SPEC-002 with no further screen to visit.

**FEAT-17.SPEC-003-AC-09:** Given Talia is on this screen with choices made but not confirmed, when she taps the back arrow, then a confirmation dialog appears asking "Discard these choices and the time block?"

**FEAT-17.SPEC-003-AC-10:** Given Talia's conflicting set contains exactly one booking, when the screen loads, then the header reads "Review 1 conflicting booking" and one row is shown.

**FEAT-17.SPEC-003-AC-11:** Given a conflicting booking is cancelled through another path while this screen is open, when Talia taps Confirm, then FEAT-17.SPEC-006 drops that booking from the set with a brief note and commits the block against the remaining conflicts.

**FEAT-17.SPEC-003-AC-12:** Given Talia taps Confirm and it is in progress, when she taps Confirm again, then the second tap has no effect and the button remains in its loading state.

**FEAT-17.SPEC-003-AC-13:** Given a network failure occurs during Confirm, when the error banner appears, then every selected choice remains as Talia set it and she can retry without re-selecting.

**FEAT-17.SPEC-003-AC-14:** Given Talia loses connectivity on this screen, when the offline message appears, then choice controls remain usable but Confirm stays disabled until connectivity returns.

**FEAT-17.SPEC-003-AC-15:** Given the Client (Riley) whose booking conflicted with a new block, when Talia's resolution is committed, then Riley is never shown this review screen and only receives the eventual notice matching Talia's chosen outcome (cancellation, reschedule, or nothing if kept as an exception).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (awaiting choices, ready to confirm, confirming, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

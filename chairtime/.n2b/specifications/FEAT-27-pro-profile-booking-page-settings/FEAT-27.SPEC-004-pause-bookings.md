---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-004
spec_name: Pause Bookings
spec_slug: pause-bookings
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Pause Bookings

## Overview

**Name:** Pause Bookings
**ID:** FEAT-27.SPEC-004
**Type:** Screen
**Purpose:** Talia pauses new bookings with an optional message and end date, or resumes them, and sees whether a system-imposed pause is also in effect.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Turning a Pro-chosen pause on (with an optional message and optional end date) and off
- Showing whether a system-imposed (subscription-lapse) pause is also active, and what that means for the resume toggle
- Deferring to FEAT-27.SPEC-009 for precedence between the two pause sources

**Non-Goals:**
- The precedence logic itself (what the resume toggle can and cannot do when both pause sources are present) -- owned by FEAT-27.SPEC-009 (Pause State Precedence Rule); this screen only reflects its outcome
- Automatically resuming bookings when a chosen end date arrives -- owned by FEAT-27.SPEC-011 (Automatic Pause Resume); this screen only sets the end date and reflects the resulting state
- Setting or lifting the system-imposed pause -- owned by FEAT-18 (Pro Subscription Billing & Account Management); this screen only displays that a system-imposed pause exists when FEAT-18.SPEC-004 has set one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Pause bookings" row | Current pause state (Active / Paused, and which source(s)) |
| FEAT-27.SPEC-011 (Automatic Pause Resume) | The chosen end date is reached and bookings resume automatically | Refreshed state showing "Taking bookings" |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Turn a Pro-chosen pause on/off, edit its message and end date; the resume toggle's exact effect is governed by FEAT-27.SPEC-009 | If a system-imposed pause is also active, the "resume bookings" toggle is shown enabled but its exact behavior (per FEAT-27.SPEC-009) is stated inline: turning it on only clears the Pro-chosen pause, the account stays paused until billing is restored |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they see the pause message (or nothing, if not paused) directly on the booking page (FEAT-05.SPEC-008), never this settings screen |
| Platform Operator (Support) | Full screen, read-only | No actions -- the pause toggle, message, and end-date fields shown disabled | Controls disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress, unsaved pause message or end date is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Pause Bookings" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Status banner:** One of: "Taking bookings" (Active, no pause of either kind); "Paused by you" (Pro-chosen pause only); "Paused -- billing needs attention" (system-imposed pause present, per FEAT-18.SPEC-004); "Paused by you and by billing" (both present).

**Body, middle section -- Pro-chosen pause controls:**
- "Pause new bookings" toggle
- Message input (multi-line text, optional, shown when the toggle is on), helper text "For example, 'On holiday until 3 June'"
- End date input (optional date picker, shown when the toggle is on), helper text "Bookings resume automatically on this date. Leave blank to resume manually."

**Body, lower section -- System-imposed pause note (shown only when present):** Fixed text: "Your account is also paused because of a billing issue. Resolve it in Billing to fully resume bookings." with a link to FEAT-18's billing screen.

**Footer:** "Save" action button, enabled while there is an unsaved change to the toggle, message, or end date.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| "Pause new bookings" toggle (turning on) | Tap | Reveals the message and end-date inputs | Toggle shows on; inputs appear | Inputs animate into view |
| "Pause new bookings" toggle (turning off / "resume bookings") | Tap | Submits the resume action to FEAT-27.SPEC-009 for precedence evaluation | Toggle shows off if only a Pro-chosen pause was active; stays reflecting a remaining system-imposed pause otherwise | If only Pro-chosen: status banner updates to "Taking bookings." If a system-imposed pause remains: status banner updates to "Paused -- billing needs attention" and an inline note states "Your bookings stay paused until your billing issue is resolved." |
| Message input | Type | Captures text input | Field shows entered text | Standard input focus state |
| End date input | Select a date | Captures the date | Field shows the chosen date | Standard input display |
| Save button | Tap | Validates the end date (cannot be in the past, per FEAT-27.SPEC-009), then writes the pause toggle, message, and end date to the Pro Account | Button shows loading state during save | Success: toast "Settings saved" and the public booking page reflects the pause message immediately (enforced by FEAT-05.SPEC-008). Failure: inline error with retry, entered values preserved |
| "Resolve it in Billing" link | Tap | Navigate to FEAT-18's billing screen | Screen closes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Status banner (announced) -> Pause toggle -> Message input (when visible) -> End date input (when visible) -> System-imposed pause note and link (when visible) -> Save.
- **Status announcements:** The status banner's content is announced to assistive technology whenever it changes.
- **Resume-toggle clarification:** When a system-imposed pause is present, the exact wording "Your bookings stay paused until your billing issue is resolved" is announced immediately after the toggle is turned off, so the outcome is never ambiguous to a screen-reader user.
- **Keyboard alternatives:** Every action, including the date picker, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Taking bookings | Status banner "Taking bookings"; toggle off; message and end-date inputs hidden | No pause of either kind is active | Talia turns the toggle on, or FEAT-18.SPEC-004 sets a system-imposed pause |
| Paused by you | Status banner "Paused by you"; toggle on; message/end-date fields shown with saved values | Pro-chosen pause is active, no system-imposed pause | Talia turns the toggle off, the chosen end date is reached (FEAT-27.SPEC-011), or a system-imposed pause is added |
| Paused -- billing needs attention | Status banner "Paused -- billing needs attention"; toggle off; system-imposed note and billing link shown | Only a system-imposed pause is active | FEAT-18 restores billing and lifts the system-imposed pause |
| Paused by you and by billing | Status banner "Paused by you and by billing"; toggle on with fields shown; system-imposed note and billing link also shown | Both pause sources are active simultaneously | Either source clears (per FEAT-27.SPEC-009's precedence) |
| Saving | Save button shows loading state | Talia taps Save with valid data | Save completes or fails |
| Error | Inline error banner with retry; entered values preserved | Save fails, or the chosen end date is rejected as being in the past | Talia taps Retry or corrects the end date |
| Support view (read-only) | Status banner and all fields visible, all controls disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- pause changes need a connection." above the status banner; last-known state remains viewable, controls disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- controls re-enable |

## Validation Rules

**Option B -- Inline (validation not warranting duplication here, since the field belongs to this screen's own definition):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| end_date | Cannot be in the past | On selection and on submit | "The resume date can't be in the past. Choose today or a later date." |

Precedence between a Pro-chosen pause and a system-imposed pause (what the resume toggle can and cannot clear) is governed by FEAT-27.SPEC-009 (Pause State Precedence Rule). See that spec for the full rule.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| "Resolve it in Billing" link tap | Billing & Subscription Management Screen | FEAT-18 (FEAT-18.SPEC-002) |
| Successful save | Stays on this screen showing the newly saved pause state | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- status (pause sub-state: Active / Paused, and which source(s)), pause message, pause end date.
**Updates:** Pro Account -- status (Pro-chosen pause on/off, via FEAT-27.SPEC-009's precedence evaluation), pause message, pause end date.
**Deletes:** None.

## Business Rules

- FEAT-27.SPEC-009 governs precedence: the Pro's "resume bookings" toggle can only clear the Pro-chosen pause, never a system-imposed pause -- this screen states that outcome explicitly rather than implying a full resume.
- XBR-14: a paused account (either source) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- The booking page's pause message display is enforced by FEAT-05.SPEC-008 (Booking Page Availability Gate), not by this screen directly -- this screen only writes the message that FEAT-05.SPEC-008 reads.
- A chosen end date triggers FEAT-27.SPEC-011 to resume bookings automatically on that date, unless a system-imposed pause is still active at that time (per FEAT-27.SPEC-009).

## Edge Cases

- **Talia sets an end date, then a system-imposed pause is added by FEAT-18 before that date arrives** -- FEAT-27.SPEC-011 still resumes the Pro-chosen pause on the chosen date, but the account stays paused overall because the system-imposed pause remains, per FEAT-27.SPEC-009; this screen's status banner updates to "Paused -- billing needs attention" on that date.
- **Talia tries to turn off the pause toggle while a system-imposed pause is also active** -- The toggle is accepted for the Pro-chosen pause only; the status banner immediately updates to "Paused -- billing needs attention" and the inline note confirms bookings stay paused until billing is resolved.
- **Talia selects an end date in the past** -- Inline error "The resume date can't be in the past. Choose today or a later date." shown on selection and blocking Save.
- **Network failure during save** -- Error banner: "Could not save your pause settings. Check your connection and try again." with a Retry button; entered values preserved.
- **Talia's chosen end date is reached while she has this screen open** -- The screen is not live-updating; if she then attempts a save, FEAT-27.SPEC-011's already-applied resume is reflected on reload before her save proceeds, avoiding a stale overwrite.
- **Talia turns the pause toggle on and off repeatedly without saving** -- No pause takes effect until Save is tapped; the booking page is unaffected until a save completes.
- **Talia sets different pause messages from two signed-in devices at effectively the same time** -- Resolution: last-write-wins, per the dependency map's Contention note for the Pro Account entity's ordinary fields; whichever save commits last is the message shown on the booking page, and neither device is shown an error (the system-imposed pause source, by contrast, is never subject to this last-write-wins path -- it is set only by FEAT-18, per FEAT-27.SPEC-009).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Pause bookings" row; back arrow returns there |
| FEAT-27.SPEC-009 (Pause State Precedence Rule) | References (inbound) | Governs what the resume toggle can and cannot clear |
| FEAT-27.SPEC-011 (Automatic Pause Resume) | Triggers (outbound) / Triggered by (inbound) | A saved end date arms this automation; its firing refreshes this screen's state |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (outbound) | Enforces the pause state and message on the public booking page |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Sets the system-imposed pause this screen displays |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Navigation (outbound) | "Resolve it in Billing" link destination |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| bookings_paused | has_message (yes/no), has_end_date (yes/no) | Talia's save turns the Pro-chosen pause on | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so pause adoption is observable |
| bookings_resumed | trigger (manual / automatic) | The Pro-chosen pause clears, either by Talia's toggle or by FEAT-27.SPEC-011 | N/A -- no connected success-metrics.md metric; retained so resume activity is observable |

## Acceptance Criteria

**FEAT-27.SPEC-004-AC-01:** Given Talia's account is fully Active, when she opens this screen, then she sees "Taking bookings" and the pause toggle off.

**FEAT-27.SPEC-004-AC-02:** Given Talia turns the pause toggle on, enters "On holiday until 3 June" as her message and sets an end date, when she taps Save, then her booking page shows the pause message instead of available times immediately (FEAT-05.SPEC-008), and the status banner shows "Paused by you."

**FEAT-27.SPEC-004-AC-03:** Given Talia's account has only a Pro-chosen pause active, when she turns the toggle off and saves, then the status banner updates to "Taking bookings" and bookings resume.

**FEAT-27.SPEC-004-AC-04:** Given Talia's account has a system-imposed pause from a lapsed subscription and no Pro-chosen pause, when she opens this screen, then she sees "Paused -- billing needs attention" with a "Resolve it in Billing" link, and the pause toggle is off.

**FEAT-27.SPEC-004-AC-05:** Given Talia's account has both a Pro-chosen pause and a system-imposed pause, when she turns the toggle off and saves, then the Pro-chosen pause clears but the status banner still shows "Paused -- billing needs attention," per FEAT-27.SPEC-009.

**FEAT-27.SPEC-004-AC-06:** Given Talia selects an end date that is in the past, when she attempts to save, then she sees "The resume date can't be in the past. Choose today or a later date." and the save does not proceed.

**FEAT-27.SPEC-004-AC-07:** Given Talia has a Pro-chosen pause with an end date of today, when that date is reached, then FEAT-27.SPEC-011 resumes bookings automatically and this screen shows "Taking bookings" on next open.

**FEAT-27.SPEC-004-AC-08:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your pause settings. Check your connection and try again." and her entered values remain on screen.

**FEAT-27.SPEC-004-AC-09:** Given a support operator opens this screen via FEAT-19, when they view it, then the toggle, message, and end-date fields are visible but disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-004-AC-10:** Given Talia opens this screen with no connectivity, when the screen loads, then her last-known pause state is viewable and all controls are disabled with "You're offline -- pause changes need a connection."

**FEAT-27.SPEC-004-AC-11:** Given Talia's end date is reached while a system-imposed pause remains active, when the automatic resume fires (FEAT-27.SPEC-011), then the status banner updates to "Paused -- billing needs attention" rather than "Taking bookings."

**FEAT-27.SPEC-004-AC-12:** Given Talia has toggled the pause on and off several times without saving, when she navigates away without tapping Save, then her account's actual pause state is unchanged and the booking page is unaffected.

**FEAT-27.SPEC-004-AC-13:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (taking bookings, paused by you, paused billing, paused both, saving, error, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

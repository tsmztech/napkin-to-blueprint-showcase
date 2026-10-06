---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-002
spec_name: Booking Link Rename
spec_slug: booking-link-rename
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Booking Link Rename

## Overview

**Name:** Booking Link Rename
**ID:** FEAT-27.SPEC-002
**Type:** Screen
**Purpose:** Talia views and changes her booking link name, sees the forwarding guarantee on the old name before she confirms, and gets suggestions when a name is already taken.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying the current booking_link_name as a full shareable link
- Entering a new booking_link_name and previewing it before saving
- Surfacing format, length, and uniqueness feedback from FEAT-27.SPEC-007 inline
- Explaining the forwarding guarantee (old name keeps working) before the Pro confirms a rename

**Non-Goals:**
- The format, length, uniqueness, and reject-with-refresh rules themselves -- owned by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule); this screen only surfaces its outcomes
- Setting up the forwarding and reservation-release mechanism -- owned by FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry); this screen only informs the Pro that it will happen
- Choosing the booking link name for the first time during onboarding -- that happens on this same screen when reached from FEAT-15's profile step (Entry Points), and is not a distinct flow requiring its own spec

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Booking link" row | Current booking_link_name |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | A submitted name is rejected because another pro claimed it in the meantime | Refreshed availability state and suggested alternatives |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter and save a new booking_link_name | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; a client who visits a renamed link is forwarded transparently by FEAT-05, per XBR-27 |
| Platform Operator (Support) | Full screen, read-only (current and previous names, forwarding window remaining) | No actions -- the input field and Save are shown disabled | Input field and Save button are disabled with the label "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any entered but unsaved name is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Booking Link" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Current link:** The full shareable link displayed as read-only text (for example, "chairtime.app/talia-lashes") with a "Copy link" action.

**Body, middle section -- Rename:**
- New link name input (text input, prefixed with the fixed domain portion so only the link-name segment is editable), with helper text "3-40 letters, numbers, or hyphens"
- Live availability indicator below the input (checking / available / taken)

**Body, lower section -- Forwarding guarantee:** A fixed, always-visible explanatory line: "If you rename your link, your old one keeps working and forwards here for at least {forwarding_window} -- so an old Instagram bio link never breaks." ({forwarding_window} renders the duration held by platform parameter: `booking-link-forward-window-months`, expressed in months.)

**Footer:** "Save new name" action button, enabled only when the entered name passes format/length checks and availability shows available.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| "Copy link" action | Tap | Copies the current full link to the clipboard | None | Confirmation toast "Link copied" |
| New link name input | Type | Captures text input; triggers FEAT-27.SPEC-007's format/length check on change, and an availability check on pause in typing | Availability indicator updates to "Checking..." then "Available" or "Taken" | Live indicator below the field |
| New link name input | Blur | Runs FEAT-27.SPEC-007's format/length validation | Error state if invalid | Exact error message from FEAT-27.SPEC-007 |
| "Save new name" button | Tap | Submits the new name to FEAT-27.SPEC-007 for final validation; on acceptance, writes booking_link_name and triggers FEAT-27.SPEC-010 (forwarding setup) | Button shows loading state during save | Success: dialog "Your link is now {new-name}. Your old link keeps forwarding for at least {forwarding_window}." (per platform parameter: `booking-link-forward-window-months`) with a "Done" action returning to FEAT-27.SPEC-001. Failure (taken by another pro since the check): the field shows "Taken" with suggested alternatives, per FEAT-27.SPEC-007 |
| Suggested alternative chip | Tap | Fills the new link name input with the suggested value and re-runs the availability check | Input and indicator update | Indicator shows "Available" for the suggestion (suggestions are generated as already-available) |

### Accessibility Notes

- **Focus order:** Back arrow -> Current link -> Copy link action -> New link name input -> Suggested alternative chips (when shown) -> Save new name.
- **Availability announcements:** The availability indicator's state change ("Checking...", "Available", "Taken") is announced to assistive technology as it updates.
- **Save feedback:** The success dialog's full text is announced on save; on rejection, focus moves to the input field and its "Taken" state and suggestions are announced.
- **Keyboard alternatives:** Every action, including selecting a suggested alternative, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current link shown, input empty, Save disabled | Screen opens | Talia begins typing a new name |
| Checking | Availability indicator shows "Checking..." | Talia pauses typing after entering a candidate name | The availability check returns |
| Available | Indicator shows "Available"; Save enabled | The candidate name passes format, length, and uniqueness checks | Talia edits the field further (returns to Checking) or taps Save |
| Taken | Indicator shows "Taken" with up to 3 suggested alternatives; Save disabled | The candidate name fails the uniqueness check | Talia edits the field, selects a suggestion, or navigates away |
| Saving | Save button shows loading state | Talia taps Save on an Available name | Save completes or is rejected |
| Rejected (claimed since check) | Field reverts to Taken state with fresh suggestions, per FEAT-27.SPEC-007's reject-with-refresh behavior | Another pro claims the name between Talia's check and her save | Talia selects a new suggestion or types another name |
| Support view (read-only) | Current link and history shown; input and Save disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- link renaming needs a connection." above the current-link section; the current link remains viewable, the rename input is disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- input re-enables |

## Validation Rules

Validation governed by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule). See that spec for the format, length, uniqueness, and reject-with-refresh rules. This screen applies FEAT-27.SPEC-007's checks on field change (format/length) and on submit (uniqueness, final acceptance).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save, "Done" tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- booking_link_name (current value, displayed as the full link).
**Updates:** Pro Account -- booking_link_name (on successful save, via FEAT-27.SPEC-007's acceptance and FEAT-27.SPEC-010's forwarding setup).
**Deletes:** None.

## Business Rules

- Every rename is validated by FEAT-27.SPEC-007 before it is accepted -- Talia cannot save a name that fails format, length, or uniqueness checks.
- A successful rename always triggers FEAT-27.SPEC-010, which begins forwarding the old name and reserves it from reuse for at least platform parameter: `booking-link-forward-window-months` -- this screen never saves a rename without that follow-on automation firing.
- XBR-27: the old name keeps forwarding transparently and never resolves to another pro's page during its forwarding window.
- This screen never exposes a way to reuse a name still inside another pro's reservation window -- FEAT-27.SPEC-007's uniqueness check covers reserved names, not just currently active ones.

## Edge Cases

- **Talia's candidate name is claimed by another pro between her availability check and her Save tap** -- The save is rejected and the screen refreshes to show "Taken" with fresh suggestions, per FEAT-27.SPEC-007's reject-with-refresh resolution for the Pro Account's booking_link_name contention.
- **Talia enters a name identical to her current one** -- Save is disabled with a note "This is already your current link name" -- no rename action is offered for a no-op change.
- **Talia leaves this screen mid-check without saving** -- The candidate name is discarded; her current booking_link_name is unaffected.
- **Network failure during save** -- Error banner: "Could not save your new link. Check your connection and try again." with a Retry button; her current link name remains unchanged and the candidate name is preserved in the field.
- **Talia enters a name that was previously hers (renamed away from, now inside her own forwarding window)** -- FEAT-27.SPEC-007 treats it as available to reclaim, since it is reserved against other pros only, not against the pro who owns the forwarding record; the availability indicator shows "Available."
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound and outbound) | Entry from the "Booking link" row; back arrow and success both return there |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | References (inbound) | Format, length, uniqueness, and reject-with-refresh rules applied to the input |
| FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry) | Triggers (outbound) | A successful save fires this automation |
| FEAT-05.SPEC-001 / FEAT-05.SPEC-008 (Public Booking Page & Booking Flow) | References (outbound) | The renamed link and its forwarding old name are resolved by these specs when a client visits either |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| booking_link_renamed | had_prior_forward (yes/no -- whether the old name was itself a forwarded name) | A rename save completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so rename activity is observable |
| booking_link_rename_rejected | reason (format / length / taken) | A rename attempt is rejected by FEAT-27.SPEC-007 | N/A -- no connected success-metrics.md metric; retained so rejection frequency (a proxy for naming friction) is observable |

## Acceptance Criteria

**FEAT-27.SPEC-002-AC-01:** Given Talia is on the Booking Link Rename screen, when it loads, then she sees her current full link and a "Copy link" action.

**FEAT-27.SPEC-002-AC-02:** Given Talia types a candidate name that passes format and length checks and is unclaimed, when the availability check completes, then the indicator shows "Available" and Save becomes enabled.

**FEAT-27.SPEC-002-AC-03:** Given Talia types a candidate name already used by another pro, when the availability check completes, then the indicator shows "Taken" with up to 3 suggested alternatives and Save stays disabled.

**FEAT-27.SPEC-002-AC-04:** Given Talia taps a suggested alternative chip, when it is selected, then the input fills with that name and the indicator shows "Available."

**FEAT-27.SPEC-002-AC-05:** Given Talia has an available candidate name, when she taps "Save new name", then her booking_link_name updates and she sees "Your link is now {new-name}. Your old link keeps forwarding for at least {forwarding_window}." with {forwarding_window} rendering platform parameter: `booking-link-forward-window-months`.

**FEAT-27.SPEC-002-AC-06:** Given Talia's candidate name is claimed by another pro between her check and her Save tap, when the save is submitted, then it is rejected, the screen refreshes to "Taken" with fresh suggestions, and her prior booking_link_name is unchanged.

**FEAT-27.SPEC-002-AC-07:** Given Talia enters a name identical to her current booking_link_name, when she views the Save control, then it is disabled with the note "This is already your current link name."

**FEAT-27.SPEC-002-AC-08:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your new link. Check your connection and try again." and her candidate name remains in the field.

**FEAT-27.SPEC-002-AC-09:** Given a support operator opens this screen via FEAT-19, when they view it, then the input and Save are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-002-AC-10:** Given Talia opens this screen with no connectivity, when the screen loads, then her current link is viewable and the rename input is disabled with "You're offline -- link renaming needs a connection."

**FEAT-27.SPEC-002-AC-11:** Given Talia previously renamed away from "talia-lashes" and it is still inside her own forwarding window, when she types "talia-lashes" as a new candidate, then the availability indicator shows "Available" (reclaimable by its own former owner).

**FEAT-27.SPEC-002-AC-12:** Given Talia taps "Save new name" twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (viewing, checking, available, taken, saving, rejected, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-005
spec_name: Notification Preferences
spec_slug: notification-preferences
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Notification Preferences

## Overview

**Name:** Notification Preferences
**ID:** FEAT-27.SPEC-005
**Type:** Screen
**Purpose:** Talia chooses which Pro notifications she receives and on which channel(s) -- in-app, text, or email.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Setting the channel combination for booking-activity notifications and attention alerts (FEAT-08.SPEC-005, FEAT-08.SPEC-006)
- Writing the notification_preferences field on the Pro Account

**Non-Goals:**
- Sending any notification -- every Pro notification's delivery, timing, retry, and content is owned by FEAT-08 (Automated Booking Messaging); this screen only sets the preference FEAT-08 reads on its next send
- Client-facing messaging consent -- owned by FEAT-14 (Messaging Consent Management); this is a distinct preference for the Pro's own account notifications, never subject to client SMS-consent rules
- A quiet-hours window for Pro notifications -- excluded per product-features.md FEAT-27 (Validation & Limits names only channel choice, not timing); no quiet-hours control is defined for the Pro's own notifications anywhere in Stage 2

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Notifications" row | Current notification_preferences value |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Change the channel selection | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; their own texting-consent preference is a separate control owned by FEAT-14 |
| Platform Operator (Support) | Full screen, read-only | No actions -- selector shown disabled | Selector disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unsaved selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Notifications" with a back arrow (returns to FEAT-27.SPEC-001).

**Body:** A single selection control offering the four channel combinations used across the product's Pro notifications (FEAT-08.SPEC-005, FEAT-08.SPEC-006): "In-app only", "In-app + text", "In-app + email", "In-app + text + email". Helper text: "This applies to new booking activity and anything needing your attention -- delivery failures, calendar reconnection, refund issues, and disputes."

**Footer:** "Save" action button, enabled while there is an unsaved change.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Channel selection control | Select an option | Captures the choice, does not save yet | Save button enables | Selected option shown highlighted |
| Save button | Tap | Writes notification_preferences to the Pro Account | Button shows loading state | Success: toast "Settings saved" -- no client message is sent (this is a Pro-only preference). Failure: inline error with retry, selection preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Channel selection control -> Save.
- **Save feedback:** "Settings saved" is announced on success; failure moves focus to the selection control with its exact message.
- **Keyboard alternatives:** The selection control and Save are both reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current preference shown selected; Save disabled | Screen opens | Talia selects a different option |
| Editing | New selection shown; Save enabled | Talia changes the selection | Talia taps Save or navigates away |
| Saving | Save button shows loading state | Talia taps Save | Save completes or fails |
| Error | Inline error banner with retry; selection preserved | Save fails | Talia taps Retry |
| Support view (read-only) | Current preference shown, selector disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- these settings need a connection to change." above the form; current preference remains viewable, selector disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- selector re-enables |

## Validation Rules

No field-level validation beyond selecting one of the four defined channel combinations -- the control offers only these four values, so an invalid selection cannot be entered.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save | Stays on this screen showing the newly saved preference | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- notification_preferences (current value).
**Updates:** Pro Account -- notification_preferences.
**Deletes:** None.

## Business Rules

- This screen writes notification_preferences only -- it never sends a notification itself; FEAT-08 reads the new preference on its next send, per the Feature Breakdown Brief's Side-Effect Inventory.
- Preferences are evaluated at delivery time by FEAT-08, not at the moment they are changed here -- a change made after a notification's trigger but before its delivery governs that delivery, consistent with the product-wide rule stated in FEAT-08's own notification specs.
- Changing this preference sends no message to the Pro or to any client, per this feature's Non-Goals ("changes to settings send no client messages").

## Edge Cases

- **A notification triggers between Talia's edit and her Save tap** -- The preference in effect at delivery time governs (FEAT-08's own rule); an unsaved in-progress edit here has no effect until Save completes.
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button; Talia's selection remains shown.
- **Talia selects the option already saved (no actual change)** -- Save stays disabled; nothing is written.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Talia changes this preference from two signed-in devices at effectively the same time** -- Resolution: last-write-wins, per the dependency map's Contention note for the Pro Account entity's ordinary preference fields; whichever save commits last is the value FEAT-08 reads on its next send, and neither device is shown an error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Notifications" row; back arrow returns there |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | References (outbound) | Reads this screen's saved preference to select delivery channels |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Reads this screen's saved preference to select delivery channels |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| notification_preferences_updated | new_channel_combination (in_app_only / in_app_text / in_app_email / in_app_text_email) | A preference save completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so preference-change activity is observable |

## Acceptance Criteria

**FEAT-27.SPEC-005-AC-01:** Given Talia is on the Notification Preferences screen, when it loads, then her currently saved channel combination is shown selected.

**FEAT-27.SPEC-005-AC-02:** Given Talia selects "In-app + text + email" and taps Save, when the save completes, then she sees "Settings saved" and no client receives any message from this change.

**FEAT-27.SPEC-005-AC-03:** Given Talia has saved "In-app only", when FEAT-08 next sends her a booking-activity notification, then it is delivered in-app only, per FEAT-08.SPEC-005.

**FEAT-27.SPEC-005-AC-04:** Given Talia changes her preference after a notification has already triggered but before FEAT-08 delivers it, when delivery occurs, then the newly saved preference governs that delivery.

**FEAT-27.SPEC-005-AC-05:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your settings. Check your connection and try again." and her selection remains shown.

**FEAT-27.SPEC-005-AC-06:** Given Talia selects the option that is already her current saved preference, when she views the Save control, then it remains disabled.

**FEAT-27.SPEC-005-AC-07:** Given a support operator opens this screen via FEAT-19, when they view it, then the selection control is disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-005-AC-08:** Given Talia opens this screen with no connectivity, when the screen loads, then her current preference is viewable and the selector is disabled with "You're offline -- these settings need a connection to change."

**FEAT-27.SPEC-005-AC-09:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-27.SPEC-005-AC-10:** Given Talia has never changed this setting, when she completes onboarding, then her preference defaults to "In-app + text" (FEAT-15.SPEC-008), which this screen shows selected on first visit.

**FEAT-27.SPEC-005-AC-11:** Given Talia's session expires while she has an unsaved selection, when she signs in again, then her unsaved selection is restored and Save remains enabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 6 (viewing, editing, saving, error, support view, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

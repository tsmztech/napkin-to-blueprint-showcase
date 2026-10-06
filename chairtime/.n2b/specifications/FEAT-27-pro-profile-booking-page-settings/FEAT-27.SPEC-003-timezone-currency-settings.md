---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-003
spec_name: Timezone & Currency Settings
spec_slug: timezone-currency-settings
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Timezone & Currency Settings

## Overview

**Name:** Timezone & Currency Settings
**ID:** FEAT-27.SPEC-003
**Type:** Screen
**Purpose:** Talia sets her account timezone and currency, sees a warning before a timezone change is saved, and sees currency locked once it has been.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying and changing the Pro Account's timezone
- Displaying the Pro Account's currency, editable only while FEAT-27.SPEC-008 says it is still unlocked
- Showing the pre-save warning about how existing bookings display after a timezone change
- Showing the exact locked explanation once currency can no longer change

**Non-Goals:**
- The currency-lock determination itself (whether the first deposit has been taken) -- owned by FEAT-27.SPEC-008 (Currency Lock Rule); this screen only reflects and defers to it
- Recomputing or re-displaying existing bookings in the new timezone -- owned by FEAT-12 (Pro Daily Schedule Dashboard) and every feature that reads timezone (FEAT-01, FEAT-02, FEAT-03, FEAT-08); this screen only warns before the change
- Choosing timezone or currency for the first time during onboarding -- captured by FEAT-15's own setup flow at account creation; this screen is reached afterward, for changes

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Timezone & currency" row | Current timezone and currency values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Change timezone always; change currency only while unlocked (FEAT-27.SPEC-008) | If currency is locked, the currency control is shown disabled with the exact locked explanation (see Interactions) -- not a bare "not allowed" |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they experience the resulting timezone-labeled times and currency-formatted prices on the booking page (FEAT-05) |
| Platform Operator (Support) | Full screen, read-only | No actions -- both controls shown disabled | Both controls disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress timezone change (not yet confirmed past the warning) is discarded; a currency change already confirmed completes atomically before the session check applies |

## Layout and Content

**Header:** Screen title "Timezone & Currency" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Timezone:** A selection input showing the current timezone (for example, "America/Los_Angeles"), with helper text "Every booking time is shown in this timezone, labeled, so it's never ambiguous (XBR-25)."

**Body, lower section -- Currency:** A selection input showing the current currency (for example, "USD"). While unlocked, the control is editable with helper text "You can change this until your first deposit is taken." Once locked (per FEAT-27.SPEC-008), the control is disabled and the helper text is replaced with the exact locked explanation.

**Footer:** "Save" action button, enabled while there is an unsaved timezone or (while unlocked) currency change.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Timezone selector | Select a new timezone | Captures the selection, does not save yet | Save button enables | Selector shows the newly chosen timezone |
| Currency selector (unlocked) | Select a new currency | Captures the selection, does not save yet | Save button enables | Selector shows the newly chosen currency |
| Currency selector (locked) | Tap | No action -- control is disabled | None | The locked explanation is already visible as static text; no further interaction occurs |
| Save button (timezone changed) | Tap | Opens a confirmation dialog stating the exact warning before committing | Dialog appears | Dialog text: "Changing your timezone won't move your existing bookings in time -- they'll keep their real moment and now display converted to the new timezone. Continue?" with "Continue" and "Keep current timezone" |
| Confirmation dialog -- "Continue" | Tap | Writes the new timezone (and any unlocked currency change) to the Pro Account | Dialog closes, screen shows saved values | Toast "Settings saved" |
| Confirmation dialog -- "Keep current timezone" | Tap | Discards the timezone change only; any unlocked currency change already selected is preserved for a subsequent save | Dialog closes, timezone selector reverts | Timezone selector shows the prior value |
| Save button (currency changed only, timezone unchanged) | Tap | Writes the new currency directly (no warning dialog -- the warning is specific to timezone's effect on existing bookings) | Button shows loading state | Success: toast "Settings saved". Failure: FEAT-27.SPEC-008's exact rejection if currency has since locked |

### Accessibility Notes

- **Focus order:** Back arrow -> Timezone selector -> Currency selector -> Save.
- **Warning dialog announcement:** The full warning text is announced to assistive technology when the dialog opens.
- **Locked-currency announcement:** The locked explanation is announced when the screen loads with currency already locked, and again if a save attempt is rejected because currency locked between load and save.
- **Save feedback:** "Settings saved" is announced on success; failures move focus to the relevant control with its exact message.
- **Keyboard alternatives:** Every action, including the confirmation dialog's two choices, is reachable and dismissible by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current timezone and currency shown; Save disabled | Screen opens | Talia changes either selector |
| Editing | Selector(s) show new, unsaved value(s); Save enabled | Talia changes timezone and/or currency (while unlocked) | Talia taps Save or navigates away |
| Timezone-change warning | Confirmation dialog shown with the exact warning text | Talia taps Save with a changed timezone | Talia chooses "Continue" or "Keep current timezone" |
| Saving | Save button shows loading state | Talia confirms "Continue," or taps Save with only a currency change | Save completes or fails |
| Currency locked | Currency selector disabled; exact locked explanation shown in place of the "editable until first deposit" helper text | FEAT-27.SPEC-008 reports the account's first deposit has been taken | Never exits -- currency lock is permanent, per FEAT-27.SPEC-008 |
| Error | Inline error banner with retry; entered values preserved | Save fails (network failure, or currency locked since load) | Talia taps Retry, or acknowledges the lock and proceeds with the timezone-only portion of her change |
| Support view (read-only) | Both controls disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- these settings need a connection to change." above the form; values remain viewable, controls disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- controls re-enable |

## Validation Rules

Currency editability is governed by FEAT-27.SPEC-008 (Currency Lock Rule). See that spec for the exact locking condition and rejection message. Timezone has no restriction beyond selecting a valid supported timezone value; no additional field-level validation exists on this screen.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save | Stays on this screen showing the newly saved values | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- timezone, currency (current values); Booking -- read by FEAT-27.SPEC-008 and by this screen's warning composition to describe how many upcoming bookings display converted after a timezone change.
**Updates:** Pro Account -- timezone (on any confirmed change); currency (on a change, only while FEAT-27.SPEC-008 reports it unlocked).
**Deletes:** None.

## Business Rules

- XBR-25: timezone and currency are per-account settings; every slot and appointment time is computed and shown in the Pro's timezone, labeled; currency is fixed once the first deposit is taken.
- A timezone change never moves an existing Booking's real moment in time -- it only changes the timezone used to display it, per the warning dialog's exact text and XBR-11 ("setup changes never silently cancel a confirmed booking").
- Currency editability is governed entirely by FEAT-27.SPEC-008 -- this screen never independently determines or overrides the lock.
- FEAT-28.SPEC-004 checks that the payout account's country and currency match this screen's saved currency (SC-20); a currency change here does not retroactively validate that match -- FEAT-28 surfaces any resulting mismatch on its own screen.

## Edge Cases

- **Talia changes both timezone and currency in the same visit, and currency locks between her edit and her Save tap** -- The timezone-change warning dialog still appears and, on "Continue," the timezone change saves; the currency change is rejected with FEAT-27.SPEC-008's exact locked message, and the currency selector reverts to its prior (locked) value while the timezone save proceeds independently.
- **Talia attempts to change currency after her first deposit was taken moments earlier (from a booking completed while this screen was open)** -- The currency selector is not live-updating; her save attempt is rejected with FEAT-27.SPEC-008's exact message and the screen refreshes to show currency now locked.
- **Talia selects the same timezone she already has** -- Save is not enabled for a no-op timezone selection; no warning dialog appears.
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button; entered values preserved.
- **Talia has many upcoming bookings and changes timezone** -- The warning dialog's text is fixed regardless of booking count; per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix, this screen reads Booking only to compose the warning, and no booking data itself is altered by the save.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Timezone & currency" row; back arrow returns there |
| FEAT-27.SPEC-008 (Currency Lock Rule) | References (inbound) | Determines whether the currency selector is editable and supplies the exact locked message |
| FEAT-01, FEAT-02, FEAT-03, FEAT-08, FEAT-12 | References (outbound) | These features read the saved timezone and currency for computation, labeling, and pricing (XBR-25) |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (outbound) | Checks the saved currency against the payout account's country/currency match |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| timezone_updated | -- | A timezone change is confirmed and saved | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so timezone-change activity is observable |
| currency_updated | -- | A currency change is saved while unlocked | N/A -- no connected success-metrics.md metric; retained so currency-change activity (which stops entirely once locked) is observable |
| currency_change_blocked | -- | A currency change is rejected because the account is locked | N/A -- no connected success-metrics.md metric; retained so lock-related friction is observable |

## Acceptance Criteria

**FEAT-27.SPEC-003-AC-01:** Given Talia is on the Timezone & Currency Settings screen, when it loads, then it shows her current timezone and currency.

**FEAT-27.SPEC-003-AC-02:** Given Talia selects a new timezone and taps Save, when the confirmation dialog appears, then it reads "Changing your timezone won't move your existing bookings in time -- they'll keep their real moment and now display converted to the new timezone. Continue?"

**FEAT-27.SPEC-003-AC-03:** Given Talia sees the timezone-change warning dialog, when she taps "Continue", then her timezone updates and she sees "Settings saved."

**FEAT-27.SPEC-003-AC-04:** Given Talia sees the timezone-change warning dialog, when she taps "Keep current timezone", then the dialog closes and her timezone selector reverts to its prior value.

**FEAT-27.SPEC-003-AC-05:** Given Talia's account has not yet taken a first deposit, when she views the currency selector, then it is editable with the helper text "You can change this until your first deposit is taken."

**FEAT-27.SPEC-003-AC-06:** Given Talia's account has taken its first deposit, when she views the currency selector, then it is disabled and shows FEAT-27.SPEC-008's exact locked explanation.

**FEAT-27.SPEC-003-AC-07:** Given Talia's currency locks between her edit and her Save tap, when she attempts to save the currency change, then it is rejected with FEAT-27.SPEC-008's exact message and the selector reverts to the now-locked value; any accompanying timezone change still saves.

**FEAT-27.SPEC-003-AC-08:** Given Talia selects her currently-set timezone (no actual change), when she views the Save control, then no warning dialog is offered for that field since nothing changed.

**FEAT-27.SPEC-003-AC-09:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your settings. Check your connection and try again." and her selections remain on screen.

**FEAT-27.SPEC-003-AC-10:** Given a support operator opens this screen via FEAT-19, when they view it, then both selectors are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-003-AC-11:** Given Talia opens this screen with no connectivity, when the screen loads, then her current values are viewable and both selectors are disabled with "You're offline -- these settings need a connection to change."

**FEAT-27.SPEC-003-AC-12:** Given Talia taps Save twice in rapid succession after confirming the timezone warning, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (viewing, editing, warning, saving, locked, error, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

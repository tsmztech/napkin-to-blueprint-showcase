---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-001
spec_name: Working Hours, Buffer, Notice & Horizon Setup
spec_slug: working-hours-buffer-notice-horizon-setup
parent_feature: FEAT-02
parent_feature_name: Availability & Working Hours Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 21
---

# Screen Spec: Working Hours, Buffer, Notice & Horizon Setup

## Overview

**Name:** Working Hours, Buffer, Notice & Horizon Setup
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** Talia (the Pro) sets her recurring weekly working windows, default buffer time between bookings, minimum booking notice, and booking horizon — the base schedule the availability engine computes from.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Setting weekly working hours per day of week, with multiple non-overlapping windows allowed per day
- Setting a single default buffer time applied between consecutive bookings
- Setting minimum booking notice and booking horizon
- Creating the initial Availability Rule on first save, and saving every subsequent edit as a new dated version (via FEAT-02.SPEC-003)
- Navigating to the per-service buffer override screen (FEAT-02.SPEC-002)
- Navigating to the time block create screen (FEAT-17.SPEC-001) to close a single day

**Non-Goals:**
- Per-service buffer overrides -- handled entirely on FEAT-02.SPEC-002, reached from this screen
- Field-level and cross-field validation logic -- owned by FEAT-02.SPEC-005 (Availability Setup Validation & Limits); this screen only displays the outcome
- Browsing or restoring a past Availability Rule version -- excluded per feature-overview.md's Non-Goals: the product definition gives the Pro no version-history browser; superseded versions are retained only for internal conflict evaluation
- Setting the Pro's account timezone -- excluded per XBR-25 and feature-overview.md's Non-Goals: FEAT-27 is the sole owner of timezone; this screen only reads and displays it
- One-off closure of a normally-working day -- not edited here; the "Close a single day" row hands the Pro to FEAT-17.SPEC-001 (Manual Time Blocking), per feature-overview.md's Primary Flows & Alternates

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) -- setup step: hours | Pro continues setup after saving services | None -- screen opens in its Empty state (no Availability Rule yet exists for this account) |
| Default entry (Pro navigation, e.g. from settings or a dashboard prompt) | Pro navigates directly to availability setup | Loads the current (latest-effective) Availability Rule, if one exists |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Pro taps back/navigate | Returns to this screen showing the current Availability Rule values as last saved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions -- edit weekly windows, default buffer, minimum booking notice, booking horizon; navigate to per-service overrides; save | -- |
| The Client (Riley) | No | No | Clients have no access to Service & Availability Setup (Access Matrix, user-persona.md); there is no client-facing navigation path into this screen at all |
| Platform Operator (Support) | Full screen, read-only | No -- all edit and save controls are hidden | A direct save attempt is not reachable from the UI (controls are hidden, not merely disabled); if attempted through a stale or replayed request, the response is "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved edits on the form are preserved in the browser and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Working Hours" with a back arrow (returns to the Pro's settings or, when reached from FEAT-15, advances the wizard to its next step) and a "Save" action button (right-aligned).

**Body, in order:**
- **Weekly window editor:** One row per day of week (Sunday through Saturday), each showing its list of working windows as start-time/end-time field pairs, an "Add window" control per day, and a remove control on each window past the first. All times are labeled with the Pro's account timezone (read from Pro Account, FEAT-27) shown once at the top of this section (e.g., "All times shown in America/New_York").
- **Default buffer field:** A single numeric field labeled "Buffer time between bookings (minutes)," applied to every booking that has no per-service override.
- **Minimum booking notice field:** A numeric/unit field labeled "How close to an appointment can a client still book?" (value plus a days/hours unit).
- **Booking horizon field:** A numeric/unit field labeled "How far ahead can clients book?" (value plus a weeks/months unit).
- **Per-service overrides entry point:** A row labeled "Buffer overrides by service" with a "Manage" control that navigates to FEAT-02.SPEC-002.
- **Close a single day entry point:** A row labeled "Need to close just one day? Block time" with a "Block time" control that navigates to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode, pre-focused on the date field.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** The weekly window editor stacks one day per row, full width, with window pairs wrapping to a second line when both fields cannot fit one row. Save remains in the header.
- **Medium size class and above:** The weekly window editor and the buffer/notice/horizon fields render in a single centered column capped at a consistent platform-wide form width (exact value is the design layer's decision); no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the Pro's settings (or the wizard's next step if entered from FEAT-15) | Screen closes | Standard transition |
| "Add window" (per day) | Tap | Adds a new empty start/end field pair under that day | New window row appears | New fields shown empty, focus moves to the new start-time field |
| Remove window control | Tap | Removes that window from the day | Window row disappears | Remaining windows re-flow upward |
| Start time field | Select/type | Captures the window's start time | Field shows entered value | Standard input state; validated via FEAT-02.SPEC-005 |
| End time field | Select/type | Captures the window's end time | Field shows entered value | Standard input state; validated via FEAT-02.SPEC-005 |
| Default buffer field | Type | Captures the default buffer value in minutes | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| Minimum booking notice field | Type/select | Captures the notice value and unit | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| Booking horizon field | Type/select | Captures the horizon value and unit | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| "Manage" (per-service overrides) | Tap | Navigate to FEAT-02.SPEC-002 (Per-Service Buffer Override) | Screen closes | Standard transition; current unsaved edits on this screen prompt the discard dialog first if any exist |
| "Block time" (close a single day) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode, pre-focused on the date field | Screen closes | Standard transition; current unsaved edits on this screen prompt the discard dialog first if any exist |
| Save button | Tap | 1. Validate all fields via FEAT-02.SPEC-005. 2. If valid, trigger FEAT-02.SPEC-003 (Availability Rule Versioning), which in turn triggers FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging). 3. On success, show confirmation and remain on screen. | Button shows loading state during save | Success: "Hours saved" confirmation banner, remains on screen with saved values shown. Failure: inline field errors (validation) or a retry banner (save failure). |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> each day's windows in calendar order (start, end, remove, add) -> default buffer -> minimum booking notice -> booking horizon -> "Manage" per-service overrides -> "Block time" -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Hours saved" confirmation is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Adding and removing windows, and every other action on this screen, are reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no hours set) | Every day shows zero windows and a guided prompt ("Add your first working window to start taking bookings"); default buffer, notice, and horizon show their defaults; Save is enabled | A brand-new Pro Account with no Availability Rule yet | Pro adds at least one window and saves |
| Filling | Form fields contain entered values; Save enabled | Pro edits any field | Pro taps Save or navigates away |
| Validating | Save button shows a loading spinner | Pro taps Save | Validation (FEAT-02.SPEC-005) completes (pass or fail) |
| Validation Error | Failed field(s) highlighted with their error messages shown inline | Validation fails | Pro corrects the field(s) and re-triggers validation |
| Saving | Save button shows a loading spinner, form fields disabled | Validation passes | FEAT-02.SPEC-003 completes or fails |
| Success | Confirmation banner "Hours saved"; form shows the saved values; remains on this screen | Save completes successfully | Banner dismisses after a few seconds or on next edit |
| Error | Error banner "Could not save your hours. Check your connection and try again." with a Retry action; all entered values remain on screen | The save operation (FEAT-02.SPEC-003) fails | Pro taps Retry or navigates away |
| Offline/Degraded | N/A -- this is a setup screen used between clients on a stable connection, not an in-the-moment mobile flow (product-features.md, FEAT-02 States); a connection loss during save surfaces through the ordinary Error state rather than a distinct offline mode | -- | -- |

## Validation Rules

Validation governed by FEAT-02.SPEC-005 (Availability Setup Validation & Limits). See that spec for all field-level and cross-field rules (window ordering, no-overlap, buffer bounds, notice bounds, horizon bounds, timezone interpretation). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (direct entry) | Pro's settings screen | FEAT-27 (Pro Profile & Booking Page Settings) |
| Back arrow tap (entered from wizard) | Next wizard step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| "Manage" per-service overrides tap | FEAT-02.SPEC-002 (Per-Service Buffer Override) | -- |
| "Block time" (close a single day) tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | FEAT-17 (Manual Time Blocking) |
| Successful save | Remains on this screen with the Success state shown | -- |

## Data Model

**Creates:** Availability Rule -- on the Pro's first save, sets weekly_windows, default_buffer, minimum_booking_notice, booking_horizon; effective_from is set by FEAT-02.SPEC-003.
**Reads:** Availability Rule -- the current (latest-effective) version's weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon, to pre-fill the form. Pro Account -- timezone (read-only, for interpreting and labeling every entered time, per XBR-25).
**Updates:** Availability Rule -- weekly_windows, default_buffer, minimum_booking_notice, booking_horizon on every subsequent save; every update is versioned (never overwritten in place) by FEAT-02.SPEC-003.
**Deletes:** None -- superseded versions are retained, never deleted, per feature-overview.md's Non-Goals and the dependency map's Availability Rule lifecycle.

## Business Rules

- Every save is validated against FEAT-02.SPEC-005 before FEAT-02.SPEC-003 is triggered -- the Pro cannot save invalid values.
- A passing save always creates a new dated Availability Rule version rather than overwriting the prior one (FEAT-02.SPEC-003).
- Every new version triggers a check of existing confirmed bookings against the new rule (FEAT-02.SPEC-004); a conflicting booking is never silently cancelled -- it is flagged for the Pro's attention on Pro Booking Management (XBR-11).
- All entered and displayed times are interpreted and labeled in the Pro's account timezone, which this screen reads but never sets (XBR-25).
- Minimum booking notice and booking horizon limit every client-facing booking path computed downstream by FEAT-03 (XBR-03); the Pro alone may book inside notice or beyond horizon through Pro Booking Management.
- Completing this step (along with the other required setup steps) satisfies one of the conditions FEAT-15 checks before the Pro's booking link can go live (XBR-26).
- A one-off closed day is handled through Manual Time Blocking (FEAT-17) rather than by editing the recurring weekly windows here; the "Block time" row on this screen navigates to FEAT-17.SPEC-001 for that purpose.
- Every navigation to this screen requires a signed-in Pro (XBR-29).

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your hours. Check your connection and try again." with a Retry button; all entered values remain on screen.
- **Pro enters overlapping windows on the same day** -- Caught by FEAT-02.SPEC-005 validation; Save does not proceed until resolved.
- **Save succeeds but the resulting conflict check (FEAT-02.SPEC-004) finds a conflicting confirmed booking** -- This screen still shows its own "Hours saved" success state; the conflict itself surfaces separately, on Pro Booking Management's attention list, not as an error on this screen.
- **Two Pro sessions (e.g., phone and desktop) save different edits to the same Availability Rule at nearly the same time** -- Last-write-wins between the Pro's own sessions, per the dependency map's Contention note for Availability Rule; each save independently creates its own new version, and the later save's version becomes the latest-effective one.
- **Pro's account timezone changes (via FEAT-27) while this screen is open with unsaved edits** -- The screen re-reads the current timezone at save time and labels all times accordingly; if the timezone changed since the values were entered, the Pro is shown a one-time notice that the displayed timezone label has updated before the save proceeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Navigation (outbound), Navigation (inbound) | Pro navigates to per-service overrides from here and back |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | Validation rules applied to every field on save |
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggers (outbound) | A passing save triggers creation of a new dated Availability Rule version |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound, indirect via FEAT-02.SPEC-003) | Every new version saved through this screen triggers the conflict check |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound), Navigation (outbound) | Wizard hands the Pro in as its hours step and resumes here if setup is left mid-way; completing this step advances the wizard |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Reads the Pro's account timezone to interpret and label every entered time |
| FEAT-17.SPEC-001 (Create/Edit Time Block) -- within FEAT-17 (Manual Time Blocking) | Navigation (outbound) | The "Block time" row hands the Pro to the time block create form (create mode, date field focused) to close a single day instead of editing the weekly windows here |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) -- within FEAT-17 (Manual Time Blocking) | References (informational) | Recurring closures are created through FEAT-17, not edited here |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | References (inbound) | Requires a signed-in Pro for any access |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| availability_hours_updated | count of working windows, count of days with at least one window | A save completes successfully | supports success-metrics.md: "Availability Setup Accuracy" |
| buffer_time_updated | new default buffer value (minutes), previous value | A save completes successfully with a changed default_buffer | supports success-metrics.md: "Availability Setup Accuracy" |
| notice_horizon_updated | new minimum_booking_notice, new booking_horizon | A save completes successfully with a changed notice or horizon value | supports success-metrics.md: "Availability Setup Accuracy" |
| availability_setup_validation_failed | field(s) in error | Save is blocked by a validation failure | supports success-metrics.md: "Availability Setup Accuracy" (a validation failure reaching this screen after prior saves indicates the Pro's intent and the displayed offer of hours are diverging) |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Talia is on the Working Hours screen with no Availability Rule yet, then every day shows zero windows and a guided prompt to add her first working window.

**FEAT-02.SPEC-001-AC-02:** Given Talia is on the Working Hours screen, when she taps "Add window" under Tuesday, then a new empty start/end field pair appears under Tuesday with focus on the new start-time field.

**FEAT-02.SPEC-001-AC-03:** Given Talia has two windows under Monday, when she taps the remove control on the second window, then that window disappears and the first window remains.

**FEAT-02.SPEC-001-AC-04:** Given Talia enters a start time after its end time for a window, when the field loses focus, then FEAT-02.SPEC-005 flags the error inline on that window.

**FEAT-02.SPEC-001-AC-05:** Given Talia sets her default buffer to 15 minutes, when she taps Save and validation passes, then FEAT-02.SPEC-003 creates a new Availability Rule version with default_buffer set to 15.

**FEAT-02.SPEC-001-AC-06:** Given Talia sets her minimum booking notice to 2 days and her booking horizon to 10 weeks, when she taps Save and validation passes, then the new Availability Rule version reflects both values.

**FEAT-02.SPEC-001-AC-07:** Given Talia is on the Working Hours screen, when she taps "Manage" under per-service overrides, then she is navigated to FEAT-02.SPEC-002.

**FEAT-02.SPEC-001-AC-08:** Given Talia taps Save with a valid form, then the Save button shows a loading state, FEAT-02.SPEC-003 runs, and on success a "Hours saved" banner appears while she remains on this screen.

**FEAT-02.SPEC-001-AC-09:** Given Talia taps Save while a prior save for the same edit is still in progress, when she taps Save a second time, then the second tap is ignored and the button remains in its loading state.

**FEAT-02.SPEC-001-AC-10:** Given Talia's save fails due to a connectivity error, then an error banner reading "Could not save your hours. Check your connection and try again." appears with a Retry action, and all her entered values remain on screen.

**FEAT-02.SPEC-001-AC-11:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-02.SPEC-001-AC-12:** Given Talia saves a new Availability Rule version that leaves an existing confirmed booking outside the new hours, then this screen still shows its own "Hours saved" success state, and the conflict is surfaced separately on Pro Booking Management via FEAT-02.SPEC-004, never as an error here.

**FEAT-02.SPEC-001-AC-13:** Given Talia saves conflicting edits from two signed-in sessions at nearly the same time, then the later save's version becomes the latest-effective Availability Rule version, consistent with last-write-wins.

**FEAT-02.SPEC-001-AC-14:** Given Talia enters her hours, then every displayed and entered time is shown labeled with her account timezone as set in FEAT-27.

**FEAT-02.SPEC-001-AC-15:** Given Riley (the Client) has no navigational path to this screen, then no client-facing entry point into Working Hours exists anywhere in the product.

**FEAT-02.SPEC-001-AC-16:** Given Platform Operator (Support) opens Talia's account for troubleshooting, when Support views this screen, then all fields are shown read-only and no Save control is visible.

**FEAT-02.SPEC-001-AC-17:** Given a visitor who is not signed in as a Pro attempts to reach this screen, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-001-AC-18:** Given Talia's session expires while she has unsaved edits on this screen, when the expiry is detected, then a dialog reading "Your session has expired. Sign in to continue." appears, and her unsaved edits are restored after she signs back in.

**FEAT-02.SPEC-001-AC-19:** Given Talia arrives at this screen from FEAT-15's setup wizard, when she completes and saves her hours, then the wizard advances to its next step.

**FEAT-02.SPEC-001-AC-20:** Given Talia has an existing Availability Rule, when she opens this screen through direct navigation (not the wizard), then the form pre-fills with the current (latest-effective) version's weekly windows, default buffer, minimum booking notice, and booking horizon.

**FEAT-02.SPEC-001-AC-21:** Given Talia is on the Working Hours screen, when she taps "Block time" under "Need to close just one day?", then she is navigated to FEAT-17.SPEC-001 in create mode with the date field focused (after the discard dialog if she has unsaved edits).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 8 (empty, filling, validating, validation error, saving, success, error, offline/degraded) | 8 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |

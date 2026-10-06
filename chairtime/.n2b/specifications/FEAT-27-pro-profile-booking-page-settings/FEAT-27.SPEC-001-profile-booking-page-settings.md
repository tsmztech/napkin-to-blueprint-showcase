---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-001
spec_name: Profile & Booking Page Settings
spec_slug: profile-booking-page-settings
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Profile & Booking Page Settings

## Overview

**Name:** Profile & Booking Page Settings
**ID:** FEAT-27.SPEC-001
**Type:** Screen
**Purpose:** Talia edits her public profile (display name, photo, intro, general area) and her confirmation-only studio address, and reaches every other settings sub-flow (booking link, timezone/currency, pause, notifications, help) and the booking-page preview from one hub.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying and editing display_name, photo, intro, general_area, and studio_address on the Pro Account
- The settings hub's navigation into every sub-flow: booking link rename (FEAT-27.SPEC-002), timezone/currency (FEAT-27.SPEC-003), pause bookings (FEAT-27.SPEC-004), notification preferences (FEAT-27.SPEC-005), and help request (FEAT-27.SPEC-006)
- The "preview my booking page" action into FEAT-05's preview mode
- Rendering inside the FEAT-15 onboarding wizard's profile step, using the same fields and save behavior as the standalone settings entry

**Non-Goals:**
- Booking link renaming, timezone/currency, pause, notification preferences, and help request -- each owned by its own sibling spec (FEAT-27.SPEC-002 through FEAT-27.SPEC-006); this screen only links to them
- Storing or serving the uploaded photo file -- owned by FEAT-27.SPEC-012 (Profile Photo Storage Capability); this screen only triggers the upload and displays the result
- The client-facing rendering of these fields on the public booking page -- owned by FEAT-05 (Public Booking Page & Booking Flow); this screen only writes the values FEAT-05 reads
- Any additional role editing these settings -- excluded per scope-boundaries.md SC-02: the Access Matrix gives Full access to the Pro alone; no manager, staff, or admin role exists to share or delegate settings access

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard, navigation) | Talia opens settings from the app's navigation | None -- screen loads the current Pro Account values |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Talia completes the sign-in step and the wizard hands off to the profile step | Wizard-embedded mode flag; on save, control returns to the wizard shell rather than closing to FEAT-12 |
| FEAT-27 sub-screens (SPEC-002 through SPEC-006) | Talia taps back from any sub-screen | None -- returns to this hub with current values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit and save display_name, photo, intro, general_area, studio_address; navigate to every sub-flow; preview the booking page | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they see the resulting public profile fields on the booking page (FEAT-05) and the studio address only inside their own booking confirmation, never this settings screen |
| Platform Operator (Support) | Full screen, read-only (all fields visible, including studio_address, per ASMP-20 status-level troubleshooting access) | No actions -- every edit control is rendered disabled | Every edit control (photo upload, text fields, sub-flow navigation buttons that would change data) is shown disabled with the label "View-only in support mode"; the "preview my booking page" action remains available since it changes nothing on the Pro Account |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered but unsaved field edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Profile & Booking Page" with a back arrow (returns to FEAT-12, or hands control back to the FEAT-15 wizard shell when embedded) and a "Save" action button (right-aligned, enabled only while there are unsaved changes).

**Body, top section -- Public profile:**
- Photo (tap to upload or replace; shows a gentle prompt "Add a photo" when empty, per the Empty state)
- Display Name (text input, required, 1-60 characters)
- Intro (multi-line text input, optional, up to 300 characters, with a live remaining-character count)
- General Area (text input, the public area shown on the booking page -- for example, a neighborhood or city, never the full address)

**Body, middle section -- Studio location (confirmation-only):**
- Studio Address (text input, required before go-live per XBR-26) with a supporting line: "Shown only in a booked client's own confirmation and reminder -- never on your public booking page."

**Body, lower section -- More settings (navigation list):**
- "Booking link" row -- shows the current booking_link_name, navigates to FEAT-27.SPEC-002
- "Timezone & currency" row -- shows the current timezone and currency, navigates to FEAT-27.SPEC-003
- "Pause bookings" row -- shows current pause state ("Taking bookings" or "Paused"), navigates to FEAT-27.SPEC-004
- "Notifications" row -- navigates to FEAT-27.SPEC-005
- "Preview my booking page" row -- navigates to FEAT-05's preview mode
- "Get help" row -- navigates to FEAT-27.SPEC-006

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, sections stacked in the order above.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; the navigation list rows remain full-width within that cap.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (standalone entry) or hand control back to FEAT-15.SPEC-001 (wizard-embedded entry) | Screen closes | Standard transition |
| Photo | Tap | Opens the device's photo picker, then triggers FEAT-27.SPEC-012 (Profile Photo Storage Capability) upload | Photo shows an uploading state, then the new photo | Success: photo updates in place. Failure: photo reverts to its previous state with an inline message from FEAT-27.SPEC-012 |
| Display Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Display Name input | Blur | Validates required, 1-60 characters | Error state if invalid | "Display name is required" or "Display name must be 60 characters or fewer" |
| Intro input | Type | Captures text input, updates remaining-character count | Field and counter update | Counter shows characters remaining |
| General Area input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Studio Address input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Studio Address input | Blur | Validates required (before go-live) | Error state if empty and account is not yet live | "A studio location is required before your booking link can go live" |
| "Booking link" row | Tap | Navigate to FEAT-27.SPEC-002 (Booking Link Rename) | Screen closes | Standard transition |
| "Timezone & currency" row | Tap | Navigate to FEAT-27.SPEC-003 (Timezone & Currency Settings) | Screen closes | Standard transition |
| "Pause bookings" row | Tap | Navigate to FEAT-27.SPEC-004 (Pause Bookings) | Screen closes | Standard transition |
| "Notifications" row | Tap | Navigate to FEAT-27.SPEC-005 (Notification Preferences) | Screen closes | Standard transition |
| "Preview my booking page" row | Tap | Navigate to FEAT-05's preview mode; emits booking_page_previewed | Screen closes | FEAT-05's preview screen opens showing the identical client-facing screen with a preview banner |
| "Get help" row | Tap | Navigate to FEAT-27.SPEC-006 (Help Request) | Screen closes | Standard transition |
| Save button | Tap | Validates display_name and studio_address; if valid, writes display_name, photo, intro, general_area, studio_address to the Pro Account | Button shows loading state during save | Success: toast "Settings saved" and public booking page reflects the change immediately. Failure: inline error banner with retry, entered values preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Photo -> Display Name -> Intro -> General Area -> Studio Address -> Booking link row -> Timezone & currency row -> Pause bookings row -> Notifications row -> Preview row -> Get help row -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** "Settings saved" is announced on success; on validation failure, focus moves to the first field in error.
- **Photo upload announcements:** The uploading state and its outcome (success or failure message) are announced to assistive technology.
- **Keyboard alternatives:** Every action, including the photo picker trigger, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first visit) | Fields show the values captured during onboarding (FEAT-15); photo shows a gentle prompt "Add a photo" if none was set | First time the Pro opens this screen after sign-in | Talia edits any field |
| Filled | Fields show current saved values | Screen loads with existing data | Talia begins editing |
| Editing | Fields show in-progress input; Save enabled | Talia types in any field or uploads a photo | Talia taps Save or navigates away |
| Saving | Save button shows loading state, fields remain visible but read-only | Talia taps Save with valid data | Save completes or fails |
| Error | Inline error banner at the top of the form with a retry action; entered values preserved | Save fails | Talia taps Retry or corrects the field causing failure and retries |
| Support view (read-only) | Full screen with every edit control disabled and labeled "View-only in support mode" | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- showing your last saved settings." at the top; fields remain viewable but disabled for editing | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- fields re-enable |

## Validation Rules

**Option B -- Inline (simple validations not warranting a standalone Logic/Rule spec for this screen's own fields):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| display_name | Required, 1-60 characters | On blur and on submit | "Display name is required" / "Display name must be 60 characters or fewer" |
| intro | Optional, up to 300 characters | On change (counter) and on submit | "Intro must be 300 characters or fewer" |
| general_area | Optional, no format restriction beyond data type | On submit | -- |
| studio_address | Required before the booking link can go live (XBR-26); otherwise optional at this stage | On submit | "A studio location is required before your booking link can go live" |

Booking link name, timezone, currency, pause, and notification-preference validation are each governed by their own sibling specs (FEAT-27.SPEC-007, FEAT-27.SPEC-008, FEAT-27.SPEC-009) and are not duplicated here.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (standalone entry) | Pro Daily Schedule Dashboard | FEAT-12 |
| Back arrow tap (wizard-embedded entry) | Setup Wizard Shell | FEAT-15 (FEAT-15.SPEC-001) |
| "Booking link" row tap | FEAT-27.SPEC-002 (Booking Link Rename) | -- |
| "Timezone & currency" row tap | FEAT-27.SPEC-003 (Timezone & Currency Settings) | -- |
| "Pause bookings" row tap | FEAT-27.SPEC-004 (Pause Bookings) | -- |
| "Notifications" row tap | FEAT-27.SPEC-005 (Notification Preferences) | -- |
| "Preview my booking page" row tap | Booking page preview mode | FEAT-05 (FEAT-05.SPEC-001) |
| "Get help" row tap | FEAT-27.SPEC-006 (Help Request) | -- |
| Successful save (wizard-embedded) | Setup Wizard Shell, advances to the next step | FEAT-15 (FEAT-15.SPEC-001), and re-evaluates go-live readiness (XBR-26) via FEAT-15.SPEC-007 |

## Data Model

**Creates:** None -- the Pro Account record already exists, created by FEAT-15.SPEC-004 on first sign-in.
**Reads:** Pro Account -- display_name, photo, intro, general_area, studio_address (this screen's fields); booking_link_name, timezone, currency, status (pause sub-state) (summary values shown in the navigation-list rows only, not editable here).
**Updates:** Pro Account -- display_name, photo, intro, general_area, studio_address.
**Deletes:** None.

## Business Rules

- Public profile fields (display_name, photo, intro, general_area) are shown on the public booking page (FEAT-05) immediately on save, per the Feature Breakdown Brief's Non-Functional Notes.
- studio_address is disclosed only inside a booked client's own confirmation and reminder (FEAT-08), never on the public page or to anyone else, per ASMP-23 and the Pro Account entity's Data Sensitivity note.
- XBR-26: display_name and studio_address being set feeds the go-live readiness evaluation (FEAT-15.SPEC-007); this screen re-evaluates readiness on every successful save while the account has not yet gone live.
- Save-failure recovery is identical across every editing screen in this feature (FEAT-27.SPEC-001 through SPEC-006): entered values are always kept on screen with a retry action, per the Feature Breakdown Brief's Shared Context.
- Offline read-only posture is identical across every screen in this feature (ASMP-27): current settings remain viewable, changes require connectivity.

## Edge Cases

- **Talia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button. Form data preserved.
- **Talia uploads a photo that fails size or format limits** -- FEAT-27.SPEC-012 returns the exact rejection reason inline below the photo; the previous photo (or empty state) remains unaffected.
- **Talia's Pro Account was updated on another signed-in device while this screen was open (for example, she also renamed her booking link from her other phone in the same moment)** -- Save is not rejected: per the dependency map's Contention note for the Pro Account, ordinary profile fields (display_name, photo, intro, general_area, studio_address) resolve last-write-wins, so this screen's save simply overwrites those fields with Talia's latest edits here; it never touches booking_link_name, timezone, currency, or pause state, which are owned by their own sibling screens and cannot be stale-overwritten from this screen.
- **Talia opens this screen for the first time with no photo set** -- The Empty state's gentle "Add a photo" prompt is shown; saving with no photo is allowed (photo is optional, per Validation & Limits).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Booking Link Rename) | Navigation (outbound) | "Booking link" row navigates here |
| FEAT-27.SPEC-003 (Timezone & Currency Settings) | Navigation (outbound) | "Timezone & currency" row navigates here |
| FEAT-27.SPEC-004 (Pause Bookings) | Navigation (outbound) | "Pause bookings" row navigates here |
| FEAT-27.SPEC-005 (Notification Preferences) | Navigation (outbound) | "Notifications" row navigates here |
| FEAT-27.SPEC-006 (Help Request) | Navigation (outbound) | "Get help" row navigates here |
| FEAT-27.SPEC-012 (Profile Photo Storage Capability) | Triggers (outbound) | Photo upload/replace goes through this integration |
| FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, landing/service list) | Navigation (outbound) | "Preview my booking page" opens FEAT-05's preview mode; FEAT-05.SPEC-001 lists this spec as its preview entry point |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound and outbound) | Entry from app navigation; back arrow returns there |
| FEAT-15.SPEC-001 (Setup Wizard Shell & Step Navigation) | Navigation (inbound and outbound) | Renders inside the wizard's profile step; successful save hands control back to the shell |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (outbound) | display_name and studio_address completion feeds this rule's readiness evaluation |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view during a help request |
| FEAT-08 (Automated Booking Messaging) | References (outbound) | studio_address is read by FEAT-08 to compose confirmation and reminder content |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| profile_updated | fields changed (display_name / photo / intro / general_area / studio_address), viewer role (always pro -- support cannot save) | A save on this screen completes successfully | N/A -- success-metrics.md's 20 metrics cover the 14 Core and 2 Important features it names by Connected Feature, and none is connected to Pro Profile & Booking Page Settings; retained so profile edit activity is observable |
| booking_page_previewed | entry source (settings hub) | Talia taps "preview my booking page" | N/A -- no success-metrics.md metric is connected to this feature; retained so preview usage is observable |

## Acceptance Criteria

**FEAT-27.SPEC-001-AC-01:** Given Talia is on the Profile & Booking Page Settings screen for the first time after sign-in, when the screen loads, then it shows the values captured during onboarding and a gentle "Add a photo" prompt if no photo was set.

**FEAT-27.SPEC-001-AC-02:** Given Talia enters "Talia Reyes Lashes" as her display name and taps Save, when the save completes, then she sees "Settings saved" and the public booking page (FEAT-05) reflects the new name immediately.

**FEAT-27.SPEC-001-AC-03:** Given Talia clears her display name and taps Save, when validation runs, then the display name field shows "Display name is required" and the save does not proceed.

**FEAT-27.SPEC-001-AC-04:** Given Talia uploads a new photo, when FEAT-27.SPEC-012 confirms storage, then her photo updates in place on this screen and on the public booking page.

**FEAT-27.SPEC-001-AC-05:** Given Talia uploads a photo that exceeds the size limit, when FEAT-27.SPEC-012 rejects it, then she sees the exact rejection reason inline and her previous photo is unaffected.

**FEAT-27.SPEC-001-AC-06:** Given Talia taps the "Booking link" row, when the tap registers, then she is navigated to FEAT-27.SPEC-002 (Booking Link Rename).

**FEAT-27.SPEC-001-AC-07:** Given Talia taps "Preview my booking page", when the tap registers, then FEAT-05's preview mode opens showing the identical client-facing screen with a preview banner, and booking_page_previewed is emitted.

**FEAT-27.SPEC-001-AC-08:** Given Talia's Pro Account has never had a studio address set, when she leaves studio_address empty and taps Save, then she sees "A studio location is required before your booking link can go live" and go-live readiness (FEAT-15.SPEC-007) is not satisfied.

**FEAT-27.SPEC-001-AC-09:** Given Talia has unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-27.SPEC-001-AC-10:** Given a support operator opens this screen via FEAT-19, when they view it, then every field and edit control is visible but disabled and labeled "View-only in support mode", and the "preview my booking page" action remains available.

**FEAT-27.SPEC-001-AC-11:** Given Talia opens this screen with no connectivity, when the screen loads, then she sees her last saved settings read-only with the offline banner and cannot edit until connectivity returns.

**FEAT-27.SPEC-001-AC-12:** Given Talia is completing the FEAT-15 onboarding wizard's profile step, when she saves this screen's fields, then control returns to the Setup Wizard Shell (FEAT-15.SPEC-001) and it advances to the next step.

**FEAT-27.SPEC-001-AC-13:** Given Talia's save fails due to a network error, when the failure is returned, then she sees "Could not save your settings. Check your connection and try again." with a Retry button, and her entered values remain on screen.

**FEAT-27.SPEC-001-AC-14:** Given Talia renamed her booking link from a second signed-in device moments before saving this screen, when this screen's save completes, then only display_name, photo, intro, general_area, and studio_address are written -- booking_link_name is untouched and reflects the rename from the other device.

**FEAT-27.SPEC-001-AC-15:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 7 (empty, filled, editing, saving, error, support view, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

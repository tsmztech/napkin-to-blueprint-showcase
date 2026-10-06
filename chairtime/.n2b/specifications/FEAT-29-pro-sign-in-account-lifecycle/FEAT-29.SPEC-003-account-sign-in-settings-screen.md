---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-003
spec_name: Account & Sign-In Settings Screen
spec_slug: account-sign-in-settings-screen
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Screen Spec: Account & Sign-In Settings Screen

## Overview

**Name:** Account & Sign-In Settings Screen
**ID:** FEAT-29.SPEC-003
**Type:** Screen
**Purpose:** Talia's ongoing hub for her signed-in devices, signing out everywhere, and starting a sign-in-contact change, data export, or account closure; also Support's view-only entry point for account status.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Displaying the Pro's sign-in contacts (masked), signed-in devices, and account status
- Starting a sign-in-contact change (email or mobile), handed to FEAT-29.SPEC-010
- The "sign out everywhere" action
- Navigating to Data Export (FEAT-29.SPEC-004) and Account Closure (FEAT-29.SPEC-005)
- Support's view-only entry point for account status during a help request

**Non-Goals:**
- Editing profile fields (display name, photo, studio address, timezone, currency, notification preferences) -- owned by FEAT-27 (Pro Profile & Booking Page Settings); this screen owns only sign-in and account-lifecycle settings
- Performing the dual-confirmation contact change itself -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing); this screen only starts the change and shows its pending state
- Viewing or editing subscription and billing details -- owned by FEAT-18 (Pro Subscription Billing & Account Management), reached via a navigation link from this screen
- Support performing any account action -- excluded per scope-boundaries.md SC-05: support's access here is view-only, status only, and never includes sign-in codes or the ability to sign in as the Pro

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard, navigation) | Pro opens settings | None |
| FEAT-29.SPEC-009 (Account Reopening) | Pro's account is restored to Active during the cooling-off period | Confirmation banner context (account reopened) |
| FEAT-19 (Platform Support Read-Only Access) | Support opens the Pro's account after a help request | Support's view-only session context; no sign-in codes ever included |
| FEAT-29.SPEC-015 (New-Device Sign-In Alert) | Talia taps "Manage sign-in" in a new-device sign-in alert email | None -- opens at the devices section |
| FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) | Talia taps "Manage sign-in" in a contact-change confirmation email | None -- opens at the sign-in contacts section |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions: start a contact change, sign out everywhere, sign out one device, open export, open closure | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen; there is no client-facing entry point into it |
| Platform Operator (Support) | Account status only (Active / Paused / Closing) and the account activity log entry for their own view (XBR-24); never sign-in contacts, codes, or devices | No actions -- entirely view-only (Access Matrix: Profile & Account Settings = View, scoped to status) | Any attempt to act (there are no actionable controls rendered for Support) is prevented because the screen renders none for this role; Support sees a status-only reduced layout, never the full Pro layout |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29; after signing in, the Pro lands back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress contact-change entry is not preserved; the Pro restarts the change after re-authentication |

## Layout and Content

**Header:** Screen title "Account & Sign-In" with a back arrow (returns to FEAT-12).

**Body, section 1 -- Sign-in contacts (Pro view only):**
- "Email" row showing the masked sign-in email, with an "Edit" action
- "Mobile number" row showing the masked sign-in mobile, with an "Edit" action
- Each Edit action starts a contact change via FEAT-29.SPEC-010; while a change is pending, the row expands into the pending-confirmation area, showing two code-entry steps side by side -- the Shared UI Pattern's code entry, identical in form to FEAT-29.SPEC-001 (sign-in) and FEAT-29.SPEC-002 (recovery):
  - **Confirm from your current {field}:** "We sent a code to {masked old value}", a 6-digit numeric code input, a primary "Confirm" action, and a secondary "Send a new code" action
  - **Confirm from your new {field}:** "We sent a code to {masked new value}", a 6-digit numeric code input, a primary "Confirm" action, and a secondary "Send a new code" action
  - Both steps are shown together, not sequentially, since the Pro typically holds both contacts and may enter either code first; a step that has been confirmed shows a checkmark and "Confirmed" in place of its inputs
  - A wrong or expired code on either step shows the generic message "That code didn't work. Try again or send a new code." under that step, matching FEAT-29.SPEC-001/SPEC-002's code-entry pattern exactly
  - Once both steps show Confirmed, the row updates to the new masked value on next load

**Body, section 2 -- Signed-in devices (Pro view only):**
- A list of signed-in devices, one row per device: device description, approximate location/type if available, last-active time, and an individual "Sign out" action per device
- Below the list, a single "Sign out everywhere" action

**Body, section 3 -- Data and account (Pro view only):**
- "Download my data" row navigating to FEAT-29.SPEC-004 (Data Export Screen)
- "Close account" row navigating to FEAT-29.SPEC-005 (Account Closure & Reopening Screen)
- "Billing & subscription" row navigating to FEAT-18 (Pro Subscription Billing & Account Management)

**Support view (reduced layout, Platform Operator only):**
- A single status line: "Account status: {Active / Paused / Closing}." No sign-in contacts, devices, or actionable controls are rendered.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order above, full width.
- **Medium size class and above:** Sections remain stacked, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Email "Edit" | Tap (Pro only) | Starts a contact change via FEAT-29.SPEC-010, governed by FEAT-29.SPEC-012 | Inline entry field appears for the new email; on submission the row expands into the two code-entry steps | Each step shows "We sent a code to {masked identifier}" |
| Mobile "Edit" | Tap (Pro only) | Starts a contact change via FEAT-29.SPEC-010, governed by FEAT-29.SPEC-012 | Inline entry field appears for the new mobile number; on submission the row expands into the two code-entry steps | Each step shows "We sent a code to {masked identifier}" |
| Pending-confirmation code input (old or new side) | Type | Captures numeric input (6 digits) | That step's field shows entered digits | Standard input focus state; auto-submits at 6 digits |
| "Confirm" button (old or new side) | Tap | Submits that side's code for validation via FEAT-29.SPEC-012, through FEAT-29.SPEC-010 | That step's button shows loading state | Success: that step shows a checkmark and "Confirmed"; once both steps show Confirmed, the row updates to the new masked value on next load. Failure: "That code didn't work. Try again or send a new code." shown under that step; that step's code input clears |
| "Send a new code" link (old or new side) | Tap | Requests a fresh code for that side only, invalidating the previous one, via FEAT-29.SPEC-010 | That side's code input clears; its failed-attempt count resets | Confirmation text: "New code sent" |
| Individual device "Sign out" | Tap (Pro only) | Expires that one signed-in device via FEAT-29.SPEC-006 | Device row is removed from the list | Toast: "Signed out of {device description}" |
| "Sign out everywhere" | Tap (Pro only) | Expires every signed-in device via FEAT-29.SPEC-006, including the current one | Every device row is removed | Confirmation dialog before the action, then the Pro is signed out and redirected to FEAT-29.SPEC-001 |
| "Download my data" row | Tap (Pro only) | Navigate to FEAT-29.SPEC-004 (Data Export Screen) | Screen closes | Standard navigation transition |
| "Close account" row | Tap (Pro only) | Navigate to FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Screen closes | Standard navigation transition |
| "Billing & subscription" row | Tap (Pro only) | Navigate to FEAT-18 (Pro Subscription Billing & Account Management) | Screen closes | Standard navigation transition |
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order (Pro view):** Back arrow -> Email Edit -> Mobile Edit -> (while pending) old-side code input -> old-side Confirm -> old-side "Send a new code" -> new-side code input -> new-side Confirm -> new-side "Send a new code" -> device list (each device row, then its Sign out action) -> Sign out everywhere -> Download my data -> Close account -> Billing & subscription.
- **Dynamic-change announcements:** Each code-entry step's appearance, its "Confirmed" state, the generic failure message, the lockout message, and "Signed out of {device}" are announced to assistive technology when they appear.
- **Code-entry transition announcement:** When a contact change starts, the transition from the Edit field to the two code-entry steps is announced ("Codes sent to {masked old value} and {masked new value}"), matching FEAT-29.SPEC-001's transition-announcement pattern.
- **Sign out everywhere confirmation:** The confirmation dialog is a focus-trapping modal announced on open, with its two options reachable by keyboard.
- **Support view:** The reduced status-only layout has its own simple focus order: back arrow -> status line; no interactive controls follow.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (Pro) | Full three-section layout with current data | Screen opens for the Pro | Pro navigates away |
| Loaded (Support) | Reduced status-only layout | Screen opens for Support | Support navigates away |
| Loading | Section placeholders shown while account and device data load | Screen first opens | Data finishes loading |
| Contact-change code entry (per side) | That side's step shows the masked identifier and an empty code input, awaiting confirmation | A contact change starts (FEAT-29.SPEC-010) and that side has not yet entered a correct code | Correct code entered (step shows Confirmed) or that side locks |
| Contact-change code error (per side) | Generic message "That code didn't work. Try again or send a new code." shown under that step; that step's code input cleared | A submitted code for that side is wrong or expired | Pro re-enters a code for that side or requests a new one |
| Contact-change code locked (per side) | That step's code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt on that side (FEAT-29.SPEC-012) | platform parameter: `contact-change-code-lockout-pause-minutes` elapses |
| Contact change pending (overall) | Both code-entry steps visible, at least one not yet Confirmed | A contact change is started (FEAT-29.SPEC-010) | Both sides confirm (row updates to the new value) or the overall confirmation window expires (row reverts, FEAT-29.SPEC-012) |
| Error | Banner "Couldn't load your account settings. Try again." with retry | Data load fails | Pro taps Retry and load succeeds |
| Offline/Degraded | Existing loaded data remains visible read-only; all action controls (Edit, Sign out, Sign out everywhere, navigation into export/closure) are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Validation of the contact-change entry (format of the new email/mobile) and of each side's confirmation code (correctness, expiry, lockout) is governed by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules). This screen applies the following inline input validation on top of that.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Pending-confirmation code (old or new side) | Exactly 6 digits | On submission (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| "Download my data" tap | FEAT-29.SPEC-004 (Data Export Screen) | -- |
| "Close account" tap | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | -- |
| "Billing & subscription" tap | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | FEAT-18 (Pro Subscription Billing & Account Management) |
| "Sign out everywhere" confirmed | FEAT-29.SPEC-001 (Sign-In Screen) | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- sign_in_email, sign_in_mobile (masked), signed_in_devices, status (for the Support view). Fields displayed match the dependency map's Pro Account entity exactly.
**Updates:** Pro Account.signed_in_devices (device list changes on individual/everywhere sign-out, via FEAT-29.SPEC-006); Pro Account.sign_in_email / sign_in_mobile (only once a contact change fully commits, via FEAT-29.SPEC-010 -- this screen never writes the field directly).
**Deletes:** None.

## Business Rules

- Support's view is limited to account status only (Active/Paused/Closing); sign-in codes, contacts, and devices are never rendered for this role (XBR-24, ASMP-30).
- Every Support view of this screen is logged in the Pro's visible account activity (XBR-24) -- the Pro can see when and that support looked, through the activity record (FEAT-16).
- A contact change is never committed by this screen directly -- it only starts the change (FEAT-29.SPEC-010) and renders its two code-entry steps and their pending/confirmed/locked/committed state, governed by FEAT-29.SPEC-012.
- Each code-entry step's failure and lockout behavior is independent per side -- a locked or repeatedly wrong code on one side never blocks entry on the other side.
- "Sign out everywhere" (FEAT-29.SPEC-006) always requires an explicit confirmation dialog before it executes, since it also signs out the Pro's current session.

## Edge Cases

- **Pro signs out an individual device while viewing this screen from that same device** -- The current device can be signed out individually like any other; doing so signs the Pro out of the current session immediately and redirects to FEAT-29.SPEC-001, identical in effect to "Sign out everywhere" for that one session.
- **A pending contact change's overall confirmation window expires while the Pro is viewing this screen** -- The row silently reverts from the code-entry steps to the prior value on next data refresh, per FEAT-29.SPEC-012's expiry rule; no error is shown, since an expired pending change is a normal outcome, not a failure.
- **The old-side code locks from repeated wrong entries while the new side has already confirmed** -- The new side's Confirmed state persists; once the lockout pause elapses, the Pro can retry the old side without the new side needing to reconfirm, since each side's state is independent (FEAT-29.SPEC-012).
- **Support opens this screen while the Pro is simultaneously viewing it** -- No conflict: Support's view is read-only and status-only, so nothing the Pro sees or does is affected; the Pro's next activity-record check shows the Support view logged (XBR-24).
- **Pro taps "Sign out everywhere" twice rapidly** -- Second tap is ignored while the first request is in progress (confirmation dialog and subsequent action are not re-entrant).
- **Pro reopens the account during the cooling-off period and lands here** -- The screen reflects the restored Active status immediately (FEAT-29.SPEC-009); the "Close account" row still leads to FEAT-29.SPEC-005, which shows no more pending closure.
- **Concurrent-edit conflict -- device list changed on another signed-in device while this screen is open** -- The device list is a live-updating snapshot, not stale-write-prone (sign-out actions are idempotent per device); if a device this Pro is trying to sign out was already signed out from elsewhere, the action is a no-op and the row is simply already absent on next refresh -- no error shown.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Individual sign-out and sign-out-everywhere actions |
| FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Triggers (outbound) | Starting a contact change |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs the pending-state display and expiry |
| FEAT-29.SPEC-004 (Data Export Screen) | Navigation (outbound) | "Download my data" |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Navigation (outbound) | "Close account" |
| FEAT-29.SPEC-009 (Account Reopening) | Navigation (inbound) | Landing here after a reopening restores the account |
| FEAT-18 (Pro Subscription Billing & Account Management) | Navigation (outbound) | "Billing & subscription" |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's view-only entry point |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (inbound/outbound) | Entry from and return to the dashboard |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| signed_out_everywhere | device_count | Pro confirms "Sign out everywhere" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| contact_change_started | field (email / mobile) | Pro submits a new value for a contact field | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so contact-change initiation remains observable |

## Acceptance Criteria

**FEAT-29.SPEC-003-AC-01:** Given Talia opens Account & Sign-In from the dashboard, when the screen loads, then she sees her masked email and mobile, her signed-in devices, and the data/account section.

**FEAT-29.SPEC-003-AC-02:** Given Talia taps "Edit" next to her email, when she submits a new email, then FEAT-29.SPEC-010 starts the change and the row expands into two code-entry steps, each showing "We sent a code to {masked identifier}".

**FEAT-29.SPEC-003-AC-03:** Given Talia's old-email code-entry step is showing, when she enters the correct 6-digit code sent to her old email, then that step shows a checkmark and "Confirmed".

**FEAT-29.SPEC-003-AC-04:** Given Talia enters an incorrect code on her new-email step, when she submits it, then that step shows "That code didn't work. Try again or send a new code." and that step's code input clears.

**FEAT-29.SPEC-003-AC-05:** Given Talia's new-email step has failed 5 consecutive code attempts, when she tries another code on that step, then it shows "Too many attempts. Try again in {remaining minutes} minutes." while her old-email step remains available.

**FEAT-29.SPEC-003-AC-06:** Given Talia is on her old-email code-entry step, when she taps "Send a new code", then a fresh code is requested for that side only, that side's input clears with the confirmation "New code sent", and her new-email step is unaffected.

**FEAT-29.SPEC-003-AC-07:** Given both of Talia's code-entry steps show Confirmed, when she next loads this screen, then the email row shows her new masked email in place of the code-entry steps.

**FEAT-29.SPEC-003-AC-08:** Given Talia taps "Sign out" on one listed device, when the confirmation is not required for a single device, then that device is removed from the list and the toast "Signed out of {device description}" appears.

**FEAT-29.SPEC-003-AC-09:** Given Talia taps "Sign out everywhere", when she confirms in the dialog, then every device is signed out, including her current session, and she is redirected to FEAT-29.SPEC-001.

**FEAT-29.SPEC-003-AC-10:** Given Talia taps "Sign out everywhere", when the confirmation dialog appears, then choosing "Cancel" leaves every device signed in unchanged.

**FEAT-29.SPEC-003-AC-11:** Given Talia taps "Download my data", when the tap registers, then she is navigated to FEAT-29.SPEC-004 (Data Export Screen).

**FEAT-29.SPEC-003-AC-12:** Given Talia taps "Close account", when the tap registers, then she is navigated to FEAT-29.SPEC-005 (Account Closure & Reopening Screen).

**FEAT-29.SPEC-003-AC-13:** Given Platform Operator (Support) opens this screen during a help request, when the screen loads, then only the account status line is shown, with no sign-in contacts, codes, or devices rendered, and no actionable controls present.

**FEAT-29.SPEC-003-AC-14:** Given Platform Operator (Support) views this screen, when the view completes, then it is logged in the Pro's visible account activity per XBR-24.

**FEAT-29.SPEC-003-AC-15:** Given an unauthenticated visitor reaches this screen's URL directly, when the access check runs, then they are redirected to FEAT-29.SPEC-001 (Sign-In Screen).

**FEAT-29.SPEC-003-AC-16:** Given Talia has a contact change pending confirmation, when its overall confirmation window expires (FEAT-29.SPEC-012) before she returns to this screen, then the row reverts to the prior value with no error shown.

**FEAT-29.SPEC-003-AC-17:** Given Talia loses connectivity while viewing this screen, when the connection drops, then her already-loaded data remains visible read-only and every action control shows "Requires a live connection".

**FEAT-29.SPEC-003-AC-18:** Given Talia's account was reopened during the cooling-off period (FEAT-29.SPEC-009), when she lands on this screen afterward, then the account status reflects Active immediately.

**FEAT-29.SPEC-003-AC-19:** Given Talia taps "Billing & subscription", when the tap registers, then she is navigated to FEAT-18.SPEC-002 (Billing & Subscription Management Screen).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 7 (loading, code entry, code error, code locked, contact pending, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

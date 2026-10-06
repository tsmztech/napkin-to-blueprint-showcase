---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-002
spec_name: Notification Preferences
spec_slug: notification-preferences
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Notification Preferences

## Overview

**Name:** Notification Preferences
**ID:** FEAT-21.SPEC-002
**Type:** Screen
**Purpose:** Nadia turns optional notifications on or off, with transactional record emails always shown locked-on; Dana views the same preferences read-only inside a logged support session.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying every notification type the product defines, grouped as optional (toggleable) or transactional (locked-on)
- Toggling an optional notification type on or off, saved immediately
- Read-only rendering of the identical preference list inside a logged Dana support session (FEAT-31)

**Non-Goals:**
- Deciding which notification types exist or their delivery content -- owned by Notifications (Email) (FEAT-14); this screen only reads the type list and writes the on/off preference
- Allowing any preference to disable a transactional record email -- excluded per XBR-30: "Notification preferences can switch off only optional emails; transactional emails core to the record ... always send"; enforced by FEAT-21.SPEC-008
- In-app or push notification channel preferences -- product-features.md and the dependency map define email as the product's sole notification channel; no other channel exists to have a preference
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Notification Preferences" from the Settings navigation shell | None |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Notification Preferences | Read-only support session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own preferences only | Toggle any optional notification type | -- |
| Dana (Support Operator) | Full screen, read-only, inside a logged FEAT-31 support session | None -- toggles rendered as disabled, non-interactive indicators (FEAT-21.SPEC-010) | A direct attempt to change a toggle is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress toggle (already saved instantly, see Interactions) requires no restoration since each toggle persists on change |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Notification Preferences" selected.

**Body:** A list of notification types grouped under two headings:
- **Transactional (always on):** one row per transactional notification type (e.g., payment confirmation), each showing the type's name and a locked toggle in the on position with a "Always sent" label beside it -- no interaction available.
- **Optional:** one row per optional notification type, each showing the type's name, a short one-line description of what it notifies about, and an on/off toggle.

For Dana's read-only rendering, both groups render identically but every toggle (locked or optional) is shown as a static, disabled indicator with no interaction affordance.

**Footer:** None -- each toggle saves immediately on change; there is no separate Save action.

### Responsive Behavior

- **Compact breakpoint:** Notification type rows stack in a single column, full width, with the toggle right-aligned on each row.
- **Medium size class and above:** Same single-column list, capped at the platform-wide form width and horizontally centered alongside the navigation shell; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Optional notification toggle | Tap | Validates via FEAT-21.SPEC-008 that the type is optional, then saves the new on/off state immediately | Toggle switches to the new position; shows a brief loading indicator during save | Success: toggle settles in new position with a small "Saved" indicator that fades. Failure: toggle reverts to its prior position and an inline error appears next to the row |
| Transactional notification indicator | Tap | No action -- non-interactive | None | No visual change; the row's "Always sent" label communicates why it cannot be toggled |
| Optional notification toggle (while saving) | Tap | No action -- debounced | None | Toggle remains in its transitional loading state |

### Accessibility Notes

- **Focus order:** Navigation shell items -> Transactional rows (announced as non-interactive) -> Optional toggles, in the order listed on screen.
- **Toggle announcements:** Each toggle's new state and the "Saved" confirmation are announced to assistive technology when a save completes; a save failure announces the reverted state and the inline error message.
- **Locked toggle:** Transactional toggles are exposed to assistive technology as disabled controls with the accessible label "Always sent -- cannot be turned off."
- **Keyboard alternatives:** Every optional toggle is operable by keyboard (space/enter); there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All notification types listed with their current on/off state | Screen opens | Always the resting state between toggles |
| Toggle Saving | The toggled row shows a brief loading indicator | User taps an optional toggle | Save completes or fails |
| Toggle Save Error | The toggled row reverts to its prior state with an inline error message: "Could not save. Try again." | Save operation fails server-side | User re-taps the toggle and the retry succeeds |
| Read-only (Dana) | All toggles shown as static disabled indicators | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); toggles are shown disabled with a banner "You're offline. Reconnect to change preferences." | Connectivity lost while screen is open | Connectivity restored -- toggles re-enable |

## Validation Rules

Validation governed by FEAT-21.SPEC-008 (Notification Preference Rules): a toggle attempt on a transactional type is rejected before any save request is made, since the control is non-interactive by construction. This screen applies FEAT-21.SPEC-008's optional/transactional distinction on render (determining which rows show a toggle vs. a locked indicator) and re-checks it on toggle before saving.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |

## Data Model

**Creates:** None.
**Reads:** Notification -- the set of `notification_type` values and which are optional vs. transactional (dependency map, Referenced Entities: "the preferences screen reads the set of notification types ... to build the toggle list; it never reads or writes individual Notification instances"). Freelancer Account -- `notification_preferences` (current on/off state per optional type).
**Updates:** Freelancer Account -- `notification_preferences` (one optional type's on/off state per toggle).
**Deletes:** None.

## Business Rules

- Only optional notification types can be toggled; transactional types are always shown locked-on (XBR-30, FEAT-21.SPEC-008).
- Each toggle saves immediately and independently -- there is no batch save or unsaved-changes state on this screen.
- Access and read-only scope (FEAT-21.SPEC-010) governs Dana's non-interactive rendering.
- A saved preference change takes effect for the next notification of that type sent by FEAT-14 (dependency map, Cross-Feature Touchpoints: "Notification preferences set here control which optional emails FEAT-14 sends").

## Edge Cases

- **Nadia toggles the same preference twice rapidly** -- The second tap is ignored while the first save is in progress (row in loading state).
- **Toggle save fails server-side** -- The toggle reverts to its prior position with the inline error "Could not save. Try again."; the row remains interactive for a retry.
- **The set of notification types changes (a new optional type is added by FEAT-14) while this screen is open** -- The screen reflects the type list as of load; a new type appears the next time the screen is opened or refreshed, with its default preference value applied until Nadia changes it.
- **Nadia toggles a preference in one open session while another of her sessions has the same screen open** -- The other open session's toggle reflects the new state on its next refresh (this is a live preference value, not a form draft, so no merge conflict arises); the dependency map's Contention note for Freelancer Account applies last-write-wins per field, and here each toggle is its own field-level write.
- **Dana's support session ends while she is viewing this screen** -- The screen redirects her out of the freelancer's account view; no toggle was ever interactive so nothing is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-008 (Notification Preference Rules) | References (inbound) | Determines which types are optional vs. transactional and enforces the toggle restriction |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-14 (Notifications (Email)) | References (outbound) | Consumes the toggled preference when deciding whether to send an optional notification |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| notification_preference_changed | notification_type, new_state: on/off | An optional toggle save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management or to Notifications (Email)'s FEAT-21 side; retained per product-features.md's own Signals field ("notification_preference_changed") so the change is observable |
| notification_preference_save_failed | notification_type | A toggle save fails server-side | N/A -- no connected success-metrics.md metric; retained to make save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-002-AC-01:** Given Nadia is on the Notification Preferences screen, when she views a transactional notification type, then it shows locked-on with an "Always sent" label and no toggle interaction is available.

**FEAT-21.SPEC-002-AC-02:** Given Nadia is on the Notification Preferences screen, when she turns an optional notification type off, then the toggle switches off, briefly shows "Saved", and the preference is saved.

**FEAT-21.SPEC-002-AC-03:** Given Nadia turns an optional notification type back on, then the toggle switches on and is saved.

**FEAT-21.SPEC-002-AC-04:** Given Nadia's toggle save fails server-side, when the failure occurs, then the toggle reverts to its prior position and shows "Could not save. Try again."

**FEAT-21.SPEC-002-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the preferences, then every toggle -- transactional and optional -- is shown as a static, disabled indicator.

**FEAT-21.SPEC-002-AC-06:** Given Dana (Support Operator) is viewing preferences read-only, when she attempts to change a toggle through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-002-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-002-AC-08:** Given Nadia taps the same optional toggle twice rapidly, when the first save is still in progress, then the second tap is ignored.

**FEAT-21.SPEC-002-AC-09:** Given Nadia loses connectivity on this screen, then all toggles are disabled and a banner reads "You're offline. Reconnect to change preferences."

**FEAT-21.SPEC-002-AC-10:** Given a new optional notification type is added by FEAT-14 while Nadia's screen is already open, when she reopens or refreshes the screen, then the new type appears with its default preference applied.

**FEAT-21.SPEC-002-AC-11:** Given Nadia toggles a preference in one open session, when a second open session of hers with the same screen is refreshed, then it shows the newly saved state for that preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (loaded, saving, save error, read-only, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

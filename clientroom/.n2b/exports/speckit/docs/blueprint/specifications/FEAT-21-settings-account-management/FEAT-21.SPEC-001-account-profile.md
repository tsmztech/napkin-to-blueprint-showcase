---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-001
spec_name: Account Profile
spec_slug: account-profile
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Account Profile

## Overview

**Name:** Account Profile
**ID:** FEAT-21.SPEC-001
**Type:** Screen
**Purpose:** Nadia views and edits her name and account profile, and reaches every other Settings section (Notification Preferences, Login & Security, Business Details & Payment Terms, Branding, and account closure) from here; Dana views her name read-only inside a logged support session (never her sign-in email).
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying and editing Nadia's name and sign-in email display (the email value itself; changing it routes to FEAT-21.SPEC-003)
- The Settings entry shell shared by SPEC-001 through SPEC-004: a persistent way to reach every other section, plus "Branding" and "Close account" navigation items
- Saving profile edits immediately with retry-preserving error handling
- Read-only rendering of the name inside a logged Dana support session (FEAT-31), with no sign-in email row and no "Login & Security" or "Close account" shell items
- The account-critical change delivery-warning banner surfaced from FEAT-21.SPEC-011 (shown to Nadia only)

**Non-Goals:**
- Changing the sign-in email or login method -- that action is started on FEAT-21.SPEC-003 (Login & Security) and carried through by FEAT-21.SPEC-005; this screen only displays the current email as read display text, never as an editable email field
- Editing business details, payment terms, or notification preferences -- owned by FEAT-21.SPEC-004 and FEAT-21.SPEC-002 respectively; this screen only links to them
- Closing or deleting the account -- excluded per the Brief's own Primary Flows: "a freelancer wanting to close her account entirely is routed to Data Export & Account Deletion (FEAT-24) rather than duplicating that flow here"; this screen only navigates to FEAT-24
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02: the persona set establishes only Primary and Reviewer client-contact roles, with no self-service account areas for them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Global navigation (any authenticated screen) | Nadia selects "Settings" | None -- profile loads from her own account |
| FEAT-21.SPEC-002 / FEAT-21.SPEC-003 / FEAT-21.SPEC-004 | Nadia selects "Profile" from the shared Settings shell | None -- returns to this default entry |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Settings | Read-only support session context; profile renders without save controls |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Edit name/profile, navigate to every Settings section, Branding, and Close account | -- |
| Dana (Support Operator) | Screen read-only, inside a logged FEAT-31 support session: the name only -- the sign-in email row, the "Change" link, the "Login & Security" shell item, the "Close account" shell item, and the delivery-warning banner are never rendered (FEAT-21.SPEC-010) | None -- no save controls, no destructive actions shown | Editing controls are not rendered at all for Dana; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on this screen if Settings was her original destination |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved profile edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with Nadia's account name as a subheading.

**Navigation shell (shared across SPEC-001 through SPEC-004):** A persistent list of Settings sections, positioned to the left of the main content on wider layouts and as a top row of tabs on narrower ones: "Profile" (this screen, selected), "Notification Preferences" (FEAT-21.SPEC-002), "Login & Security" (FEAT-21.SPEC-003), "Business Details & Payment Terms" (FEAT-21.SPEC-004), "Payment account" (FEAT-32.SPEC-001), "Branding" (FEAT-19.SPEC-001), and "Close account" (routes to FEAT-24) at the bottom of the shell, visually separated from the other items.

**Body:** A single-column form with the following fields:
- Name (text input, required)
- Sign-in email (read-only display text, with a "Change" link that navigates to FEAT-21.SPEC-003) -- Nadia only

Below the form, a "Save" action button.

**Delivery-warning banner (Nadia only):** When FEAT-21.SPEC-011 reports that a security confirmation email for a sign-in email change failed after all retries, a warning banner appears at the top of the body, above the form, reading "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." with a "Go to Login & Security" link (navigates to FEAT-21.SPEC-003) and a "Dismiss" button. The banner names no email address. Several unresolved failures collapse into this single banner.

**Dana's rendering:** shows only the Name value as read-only text -- no sign-in email row, no "Save" button, no "Change" link, no delivery-warning banner -- and the navigation shell is present but omits "Login & Security", "Payment account", and "Close account" (sign-in credentials are never visible to the operator, account closure is never available inside a support session per FEAT-21.SPEC-010, and Dana never reaches the payment connection screen -- her read-only connection status lives on her own FEAT-31 support-session surface, per FEAT-32.SPEC-001).

**Footer:** None -- Save is directly below the form.

### Responsive Behavior

- **Compact breakpoint:** The Settings navigation shell collapses to a horizontal scrolling tab row above the form; the form itself is single-column, full width.
- **Medium size class and above:** The navigation shell renders as a fixed left-hand column; the form is capped at a consistent platform-wide form width and sits to its right. No structural change to the form beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Navigation shell item | Tap | Navigate to the corresponding Settings section (FEAT-21.SPEC-002, SPEC-003, SPEC-004) or FEAT-19.SPEC-001 (Branding) | Screen changes to the selected section | Selected item highlighted in the shell |
| "Payment account" item | Tap | Navigate to FEAT-32.SPEC-001 (Payment Account connection screen); Nadia only | Screen leaves the Settings profile view; FEAT-32.SPEC-001's back arrow returns her to this screen (FEAT-21.SPEC-001) | Standard navigation transition |
| "Close account" item | Tap | Navigate to FEAT-24 (Data Export & Account Deletion) | Screen leaves Settings | Standard navigation transition |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-21.SPEC-007 | Error state on field | "Name is required" below field |
| "Change" link (sign-in email) | Tap | Navigate to FEAT-21.SPEC-003 (Login & Security) | Screen changes | Standard navigation transition |
| Save button | Tap | Validate the name field via FEAT-21.SPEC-007; if valid, save the profile | Button shows loading state during save | Success: toast "Profile updated" and the form reflects the saved value. Failure: inline error banner, retry option, entered text preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Delivery-warning banner: "Go to Login & Security" | Tap | Navigate to FEAT-21.SPEC-003 | Screen changes; banner stays until dismissed | Standard navigation transition |
| Delivery-warning banner: "Dismiss" | Tap | Marks the reported delivery failure(s) as acknowledged | Banner is removed and does not return for those failures | Banner disappears; removal announced to assistive technology |

### Accessibility Notes

- **Focus order:** Navigation shell items (in listed order) -> delivery-warning banner ("Go to Login & Security", "Dismiss"; when shown) -> Name input -> "Change" link -> Save button.
- **Validation announcements:** When the name field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Profile updated" toast is announced on success; on validation failure, focus moves to the name field.
- **Keyboard alternatives:** Every action on this screen, including navigation shell selection, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with Nadia's current name and sign-in email, Save button enabled | Screen opens | User begins editing |
| Editing | Name field shows in-progress input, Save button enabled | User types in the name field | User taps Save or navigates away |
| Saving | Save button shows a loading spinner, name field disabled | User taps Save with valid input | Save completes or fails |
| Validation Error | Name field highlighted with error message below it | Name field is empty on blur or submit | User enters a valid name and re-triggers validation |
| Error | Error banner at top of form: "Could not save your profile. Check your connection and try again." with a Retry button | Save operation fails server-side | User taps Retry; entered name is preserved and resubmitted |
| Delivery warning | Warning banner above the form (text in Layout and Content), form otherwise as Loaded | FEAT-21.SPEC-011 reports a confirmation-email delivery failure after final retry and Nadia has not dismissed it | Nadia taps "Dismiss" |
| Read-only (Dana) | Name shown as read-only text; no sign-in email row, no Save button, no "Change" link, no delivery-warning banner | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); the form remains visible with current values but Save is disabled and a banner reads "You're offline. Reconnect to save changes." | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; no queued submission occurs |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules). See that spec for the name field's required/format rules. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |
| "Change" link (sign-in email) | FEAT-21.SPEC-003 (Login & Security) | -- |
| Successful save | Stays on FEAT-21.SPEC-001 with the confirmation toast | -- |

## Data Model

**Creates:** None.
**Reads:** Freelancer Account -- `name`, sign-in email (displayed read-only to Nadia only; never read into Dana's rendering, per the dependency map's Data Sensitivity note "sign-in credentials never visible to the operator"); the unacknowledged delivery-failure flag reported by FEAT-21.SPEC-011 (Nadia only).
**Updates (acknowledgement):** the delivery-failure flag is cleared when Nadia taps "Dismiss".
**Updates:** Freelancer Account -- `name`.
**Deletes:** None.

## Business Rules

- Field validation (FEAT-21.SPEC-007) is enforced before any save completes -- the user cannot save with an empty name.
- Access and read-only scope (FEAT-21.SPEC-010) governs what Dana sees and cannot act on -- this screen's Access and Visibility table is consistent with that spec: Dana sees the name only, never the sign-in email, and never the "Login & Security" or "Close account" shell items.
- The delivery-warning banner is driven solely by FEAT-21.SPEC-011's failure report; it persists across visits until Nadia dismisses it and is never rendered for Dana.
- The sign-in email itself is never directly editable here; XBR-30 and the Freelancer Account entity's Contention note require the dedicated re-verification path (FEAT-21.SPEC-003 -> FEAT-21.SPEC-005) for any email/login-method change.
- Two open sessions of Nadia's editing the name field resolve last-write-wins, per the dependency map's Contention note for the Freelancer Account entity.

## Edge Cases

- **Nadia navigates away with an unsaved name edit** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails server-side** -- Error banner: "Could not save your profile. Check your connection and try again." with a Retry button; the entered name is preserved and retried without being discarded (product-features.md, States field).
- **Name changed by Nadia in a second open session while this screen is open** -- Save is last-write-wins per the dependency map's Contention note for the Freelancer Account entity: Nadia's save here overwrites whatever the other session saved, with no merge and no warning dialog, consistent with "None across roles ... two open sessions of Nadia's resolve last-write-wins per field."
- **A second confirmation-email delivery failure is reported while the banner is already showing** -- The single banner remains; no second banner is added, and one "Dismiss" acknowledges both.
- **Dana's support session ends while she is viewing this screen** -- The screen redirects her out of the freelancer's account view; no data was ever editable so nothing is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | Name field validation rules |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound/inbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound/inbound) | Shared Settings navigation shell; "Change" link for sign-in email |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound/inbound) | Shared Settings navigation shell |
| FEAT-32.SPEC-001 (Payment Account connection screen) | Navigation (outbound) | "Payment account" navigation item, cross-feature; its back arrow returns to Settings (FEAT-21.SPEC-001) |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (outbound) | "Branding" navigation item; Nadia resumes a skipped branding step |
| FEAT-24 (Data Export & Account Deletion) | Navigation (outbound) | "Close account" navigation item, cross-feature |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | References (inbound) | Reports confirmation-email delivery failures that this screen surfaces as the delivery-warning banner |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| settings_updated | section: profile; field changed: name | Nadia's profile save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management (success-metrics.md carries no Connected Feature entry for FEAT-21); retained per product-features.md's own Signals field ("settings_updated") so the save is observable |
| settings_save_failed | section: profile | Save fails server-side | N/A -- no connected success-metrics.md metric; retained to make retry-preserving save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-001-AC-01:** Given Nadia is on the Account Profile screen, when she clears the name field and taps Save, then the name field shows an error state with the message "Name is required" and the save does not proceed.

**FEAT-21.SPEC-001-AC-02:** Given Nadia enters "Nadia Voss" as her name and taps Save, then the profile saves, a "Profile updated" toast appears, and the field shows "Nadia Voss".

**FEAT-21.SPEC-001-AC-03:** Given Nadia is on the Account Profile screen, when she taps "Change" next to her sign-in email, then she is navigated to FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-001-AC-04:** Given Nadia taps "Close account", then she is navigated to FEAT-24 (Data Export & Account Deletion) and no closure or deletion happens inside this screen.

**FEAT-21.SPEC-001-AC-05:** Given Nadia taps "Branding" from the navigation shell, then she is navigated to FEAT-19.SPEC-001 (Branding Settings).

**FEAT-21.SPEC-001-AC-06:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the profile, then she sees the current name only -- no sign-in email row, no Save button, no "Change" link, no delivery-warning banner, and neither a "Login & Security" nor a "Close account" item in the navigation shell.

**FEAT-21.SPEC-001-AC-07:** Given Dana (Support Operator) is viewing the profile read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-001-AC-08:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-001-AC-09:** Given Nadia's profile save fails server-side, when the failure occurs, then the error banner "Could not save your profile. Check your connection and try again." appears with a Retry button, and her entered name remains in the field.

**FEAT-21.SPEC-001-AC-10:** Given Nadia has an unsaved name edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-001-AC-11:** Given Nadia saves her name from a second open session while this screen is also open in a first session with a different unsaved name, when the first session's Save is tapped, then the first session's value overwrites the second session's saved value (last-write-wins), with no merge dialog shown.

**FEAT-21.SPEC-001-AC-12:** Given Nadia loses connectivity while on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-001-AC-13:** Given Nadia's expired session is detected while she has an unsaved name edit, when the "Your session has expired" dialog is dismissed by signing in again, then her unsaved name edit is restored on this screen.

**FEAT-21.SPEC-001-AC-14:** Given FEAT-21.SPEC-011 has reported a confirmation-email delivery failure after final retry, when Nadia opens this screen, then the banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears above the form with "Go to Login & Security" and "Dismiss".

**FEAT-21.SPEC-001-AC-15:** Given the delivery-warning banner is showing, when Nadia taps "Go to Login & Security", then she is navigated to FEAT-21.SPEC-003; when she taps "Dismiss", then the banner is removed and does not reappear on later visits for that failure.

**FEAT-21.SPEC-001-AC-16:** Given a delivery failure is unacknowledged, when Dana (Support Operator) opens this screen inside a logged support session, then no delivery-warning banner is rendered for her.

**FEAT-21.SPEC-001-AC-17:** Given Nadia is on the Account Profile screen, when she taps "Payment account" in the navigation shell, then she is navigated to FEAT-32.SPEC-001 (Payment Account connection screen), and when she taps that screen's back arrow she returns to this screen; and given Dana (Support Operator) is inside a logged support session, then no "Payment account" item is rendered in her navigation shell.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (loaded, editing, saving, validation error, error, delivery warning, read-only (Dana), offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

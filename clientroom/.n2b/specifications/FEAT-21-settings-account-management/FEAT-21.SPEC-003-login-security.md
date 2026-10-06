---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-003
spec_name: Login & Security
spec_slug: login-security
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Login & Security

## Overview

**Name:** Login & Security
**ID:** FEAT-21.SPEC-003
**Type:** Screen
**Purpose:** Nadia starts a sign-in email or login method change, views and manages her signed-in devices, and signs out other sessions; never surfaced to Dana or any client contact.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Starting a new sign-in email or login method change, handed off to FEAT-21.SPEC-005 for re-verification
- Showing the state of a pending, unconfirmed change (in progress, with a resend or cancel option)
- Listing Nadia's signed-in devices (current session marked, others listed)
- Triggering "Sign out other devices" (FEAT-21.SPEC-006)

**Non-Goals:**
- Carrying out the re-verification itself (sending the link/code, confirming it, expiring it) -- owned entirely by FEAT-21.SPEC-005; this screen only starts the change and reflects its pending/confirmed/expired state
- Alternate or multiple sign-in methods (e.g., third-party single sign-on) -- excluded per the Brief's Non-Goals: "neither BRIEF.md nor the Access Matrix names any alternate authentication method for Nadia's own account; login recovery here covers a single email-based sign-in method only"
- Any visibility for Dana -- excluded per scope-boundaries.md SC-04 and FEAT-21.SPEC-010: "Dana never sees ... any sign-in credential"; this screen is never rendered inside a support session
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Login & Security" from the Settings navigation shell | None |
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Change" next to her sign-in email | None -- lands on this screen ready to start a change |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | The pending change expires or Nadia abandons it | Reverted state -- the prior sign-in email/login method is shown active again with a restart option |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | Nadia taps the email's "Go to Login & Security" CTA | None -- the screen loads her account's current sign-in email, login method, and devices |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Start a sign-in email/login method change, view signed-in devices, sign out other devices | -- |
| Dana (Support Operator) | No -- this screen is never rendered inside a support session | No | The Settings navigation shell shown to Dana omits "Login & Security" entirely (FEAT-21.SPEC-010); there is no direct-link path into this screen from a support session |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress, not-yet-submitted email change entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Login & Security" selected.

**Body, section 1 -- Sign-in email and login method:**
- Current sign-in email (read-only display text)
- "Change sign-in email" action button
- If a change is pending (per FEAT-21.SPEC-005): a status banner showing "Verification pending for {new email}" with "Resend link", "Use a different email", and "Cancel" actions, replacing the "Change sign-in email" button for the duration of the pending state. "Use a different email" is the only control for starting a replacement change while one is pending.

**Body, section 2 -- Signed-in devices:**
- A list of Nadia's active sessions, each row showing: device/browser description, approximate location, and last-active time; the current session is labeled "This device" and cannot be individually signed out from this list
- "Sign out other devices" action button, shown only when at least one other session exists

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Both sections stack vertically, full width; the device list rows stack their device description above location/last-active on narrow widths.
- **Medium size class and above:** Both sections remain single-column, capped at the platform-wide form width alongside the navigation shell; device list rows show device description, location, and last-active in one row.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Change sign-in email" button | Tap | Opens an inline form to enter a new sign-in email | Form appears below the button | Form fields ready for input |
| New sign-in email field | Type / Submit | On submit, triggers FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) via validation from FEAT-21.SPEC-007 | Screen enters the pending state | Success: banner "Verification pending for {new email}" with instructions to check that inbox. Failure: inline error from FEAT-21.SPEC-007 |
| "Resend link" (pending state) | Tap | Triggers FEAT-21.SPEC-005's resend path | Button briefly disabled | Confirmation text "A new verification link has been sent." |
| "Use a different email" (pending state) | Tap | Opens the same inline new-email form as "Change sign-in email"; submitting it triggers FEAT-21.SPEC-005's new-submission path, which replaces the pending change | Inline form appears below the banner; the pending banner stays until the replacement is accepted | On accept: banner updates to "Verification pending for {newest email}". Failure: inline error from FEAT-21.SPEC-007 and the existing pending change is untouched |
| "Cancel" (pending state) | Tap | Triggers FEAT-21.SPEC-005's abandon path | Pending banner clears, prior state restored | Toast "Sign-in email change cancelled." |
| "Sign out other devices" button | Tap | Confirmation dialog, then triggers FEAT-21.SPEC-006 (Sign-Out Other Sessions) | Button shows loading state during processing | Success: toast "All other sessions signed out." and the device list refreshes to show only "This device". Failure: inline error, device list unchanged |

### Accessibility Notes

- **Focus order:** Navigation shell -> "Change sign-in email" (or the pending banner's "Resend link"/"Use a different email"/"Cancel") -> new-email input when open -> device list rows -> "Sign out other devices".
- **Pending-state announcements:** Entering or leaving the pending state is announced to assistive technology (the pending banner's appearance and its removal on cancel/confirm/expiry).
- **Sign-out feedback:** The "All other sessions signed out" confirmation is announced on success; a failure announcement names the reason from FEAT-21.SPEC-006's outcome.
- **Keyboard alternatives:** Every action on this screen, including opening the email-change form and triggering sign-out, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Current sign-in email shown with "Change sign-in email" button; device list loaded | Screen opens with no pending change | User starts a change, or a change becomes pending from elsewhere |
| Change form open | Inline new-email input visible below the "Change sign-in email" button | User taps "Change sign-in email" | User submits, or collapses the form |
| Pending verification | Banner "Verification pending for {new email}" with "Resend link", "Use a different email", and "Cancel"; the "Change sign-in email" button is not shown | FEAT-21.SPEC-005 accepts the submitted change | FEAT-21.SPEC-005 confirms, the change expires, or Nadia cancels; "Use a different email" keeps this state with the newest target |
| Sign-out processing | "Sign out other devices" button shows a loading spinner, device list temporarily non-interactive | User confirms "Sign out other devices" | FEAT-21.SPEC-006 completes or fails |
| Error | Inline error banner: "Could not complete this action. Check your connection and try again." with Retry | Change submission, resend, cancel, or sign-out fails server-side | User taps Retry |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); action buttons are disabled with a banner "You're offline. Reconnect to manage login and security." | Connectivity lost while screen is open | Connectivity restored -- actions re-enable |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules) for the new sign-in email's format. This screen applies that validation on submit, before handing off to FEAT-21.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |

## Data Model

**Creates:** None directly -- submitting a new sign-in email creates a pending change record owned by FEAT-21.SPEC-005.
**Reads:** Freelancer Account -- current sign-in email, signed-in devices list.
**Updates:** None directly -- "Change sign-in email" and "Sign out other devices" delegate their writes to FEAT-21.SPEC-005 and FEAT-21.SPEC-006 respectively.
**Deletes:** None.

## Business Rules

- The sign-in email/login method change is never applied directly from this screen -- it always goes through FEAT-21.SPEC-005's re-verification process (dependency map, Freelancer Account Contention: "the sign-in email change, which requires re-verification before it takes effect").
- Only one pending change can exist at a time; starting a replacement change while one is pending (via the banner's "Use a different email" control) replaces the pending target email (enforced by FEAT-21.SPEC-005 and reflected here as the pending banner updating to the newest target).
- "Sign out other devices" never signs out the current session -- only FEAT-21.SPEC-006's scope (every session except the current one).
- Access and read-only scope (FEAT-21.SPEC-010) excludes this entire screen from Dana's support session and from every client contact role.

## Edge Cases

- **Nadia submits the same email currently active as sign-in email** -- Rejected inline with "This is already your sign-in email." per FEAT-21.SPEC-007; no pending change is created.
- **Nadia starts a second email change while one is already pending (via "Use a different email" on the pending banner)** -- The pending banner updates to reflect the newest target email; the prior pending verification link/code is invalidated by FEAT-21.SPEC-005.
- **The pending change expires while Nadia is on this screen** -- The pending banner clears automatically and the "Change sign-in email" button returns, per FEAT-21.SPEC-005's expiry outcome; no data loss occurs since the prior email remains active throughout.
- **Nadia taps "Sign out other devices" with only her current session active** -- The button is not shown (per Layout and Content: "shown only when at least one other session exists").
- **A device Nadia signs out reconnects mid-action** -- The sign-out is processed against the session list as it stood when FEAT-21.SPEC-006 began; a session that reconnects after the invalidation is treated as a fresh unauthenticated session and must sign in again.
- **Nadia's own current session is somehow included in a stale device-list snapshot** -- The current session is always excluded from "Sign out other devices" by construction (FEAT-21.SPEC-006's scope), regardless of list staleness.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Triggers (outbound) | "Change sign-in email" submission starts the re-verification process |
| FEAT-21.SPEC-006 (Sign-Out Other Sessions) | Triggers (outbound) | "Sign out other devices" invalidates every other session |
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | New sign-in email format validation |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Excludes this screen from Dana's support session entirely |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point and "Change" link |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound) | Shared Settings navigation shell |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_email_change_started | N/A -- no personal data in the event payload beyond the fact of the attempt | Nadia submits a new sign-in email and FEAT-21.SPEC-005 accepts it as pending | N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("account_email_changed" family) so the start of the flow is observable |
| other_sessions_signed_out | session_count | FEAT-21.SPEC-006 completes successfully | N/A -- no connected success-metrics.md metric; retained per product-features.md's own Signals field so this security action is observable |

## Acceptance Criteria

**FEAT-21.SPEC-003-AC-01:** Given Nadia is on the Login & Security screen, when she taps "Change sign-in email" and enters a new valid email, then FEAT-21.SPEC-005 starts the change and the screen shows "Verification pending for {new email}".

**FEAT-21.SPEC-003-AC-02:** Given Nadia enters her currently active sign-in email as the "new" email, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-003-AC-03:** Given Nadia has a pending sign-in email change, when she taps "Resend link", then a new verification link is sent and she sees "A new verification link has been sent."

**FEAT-21.SPEC-003-AC-04:** Given Nadia has a pending sign-in email change, when she taps "Cancel", then the pending banner clears, she sees "Sign-in email change cancelled.", and her prior sign-in email remains active.

**FEAT-21.SPEC-003-AC-05:** Given Nadia has a pending change, when she taps "Use a different email" on the pending banner and submits a second, different valid email, then the pending banner updates to "Verification pending for {newest email}" and the "Change sign-in email" button remains hidden.

**FEAT-21.SPEC-003-AC-06:** Given Nadia's pending change expires while she is on this screen, then the pending banner clears automatically and the "Change sign-in email" button reappears.

**FEAT-21.SPEC-003-AC-07:** Given Nadia is signed in on two other devices in addition to her current one, when she opens this screen, then the device list shows all three sessions with the current one labeled "This device".

**FEAT-21.SPEC-003-AC-08:** Given Nadia has one other active session, when she taps "Sign out other devices" and confirms, then FEAT-21.SPEC-006 runs and the device list refreshes to show only "This device", with the toast "All other sessions signed out."

**FEAT-21.SPEC-003-AC-09:** Given Nadia has no other active sessions, when she views this screen, then "Sign out other devices" is not shown.

**FEAT-21.SPEC-003-AC-10:** Given Dana (Support Operator) is inside a logged FEAT-31 support session, when she views the Settings navigation shell, then "Login & Security" is not listed and this screen is not reachable.

**FEAT-21.SPEC-003-AC-11:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-003-AC-12:** Given Nadia's email-change submission fails server-side, when the failure occurs, then the error banner "Could not complete this action. Check your connection and try again." appears with a Retry button.

**FEAT-21.SPEC-003-AC-13:** Given Nadia loses connectivity on this screen, then all actions are disabled and a banner reads "You're offline. Reconnect to manage login and security."

**FEAT-21.SPEC-003-AC-14:** Given Nadia's expired session is detected while she has an unsubmitted new-email entry in the open change form, when she signs in again, then the entry is restored on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (default, change form open, pending, sign-out processing, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

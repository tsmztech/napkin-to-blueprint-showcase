---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-002
spec_name: Password Recovery
spec_slug: password-recovery
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Screen Spec: Password Recovery

## Overview

**Name:** Password Recovery
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** An adult who cannot sign in requests a sign-in reset by email and completes it.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Requesting a sign-in reset for a given email address
- Confirming the reset request was sent, without revealing whether the email is registered
- Completing the reset from the emailed link: setting a new password and returning to sign-in

**Non-Goals:**
- Sending the reset email itself -- delivered through FEAT-01.SPEC-017 (Transactional Email Integration); this screen only requests and completes the reset
- Account recovery for a forgotten email address -- product-features.md defines no email-recovery path; an adult who has lost access to both their password and their registered email must contact support (FEAT-18, general support contact)
- Kid-profile or operator sign-in recovery -- excluded per scope-boundaries.md SC-02 and this feature's Non-Goals: young kid profiles have no login to recover, and operator access is never authenticated through this consumer flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | User taps "Forgot password?" | Email address, if already entered on the sign-in form |
| Reset email link | User taps the reset link in the emailed message | A single-use reset token identifying the account |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Request and complete a sign-in reset for her own account | -- |
| Sam (Other Adult Member) | Full screen | Request and complete a sign-in reset for his own account | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles have no login to recover |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- the Later-phase older-kid login's recovery mechanism is out of this feature's scope |
| Riley (Operator, support) | No | No | N/A -- operator access is never authenticated through this consumer recovery flow |
| Unauthenticated | Yes | Yes (request and complete a reset) | This screen is reachable without an active session by design -- no restriction to describe |
| Expired session | Yes | Yes | Treated identically to unauthenticated: a reset can be requested or completed without a live session |

single-role restriction note: this screen serves both adult roles identically -- there is no role-specific behavior, since a password reset always acts on the requesting person's own account only.

## Layout and Content

**Header:** "Reset your sign-in" title with a back arrow returning to FEAT-01.SPEC-001.

**Body -- Request step:** Single field, "Email address," and a "Send reset link" button below it.

**Body -- Confirmation step (after request submitted):** A single message: "If an account exists for {entered email}, a reset link is on its way. Check your inbox." No other content or action beyond a "Back to sign in" link.

**Body -- Reset step (arrived via emailed link):** "New password" field with a strength indicator (same treatment as sign-up), a "Confirm new password" field, and a "Set new password" button.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single field or fields stack full width; button full width below.
- **Medium size class and above:** Content area caps at the same narrow platform-wide form width as FEAT-01.SPEC-001, horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Screen closes | Standard transition back to sign-in |
| Email address field (request step) | Type | Captures the email address | Field shows entered text | Standard input focus state |
| "Send reset link" button | Tap | 1. Validate email format. 2. Trigger FEAT-01.SPEC-017 to send a reset email if an account exists for that address. | Button shows loading state | Always transitions to the confirmation message, regardless of whether the email is registered (no account-existence disclosure) |
| "Back to sign in" link (confirmation step) | Tap | Navigate to FEAT-01.SPEC-001 | Screen closes | Standard transition |
| New password field (reset step) | Type | Captures the new password; strength indicator updates live | Indicator reflects strength | Live update |
| Confirm new password field (reset step) | Blur | Compares against the new password field via FEAT-01.SPEC-014 | Error state if mismatched | "Passwords don't match" below the field |
| "Set new password" button (reset step) | Tap | 1. Validate both password fields. 2. Apply the new sign-in credential to the account. 3. Sign the user in. | Button shows loading state | Success: navigates to FEAT-01.SPEC-010 (or FEAT-01.SPEC-003 if the account has no household yet) with a confirmation toast "Sign-in updated." Failure: inline error, reset link intact for retry |

### Accessibility Notes

- **Focus order:** Back arrow -> Email address (request step) or New password -> Confirm new password -> Set new password (reset step).
- **Validation announcements:** Password-mismatch and format errors are announced and associated with their field.
- **Confirmation announcement:** The "reset link is on its way" message is announced on the confirmation step so a screen reader user is not left waiting silently.
- **Keyboard alternatives:** Every action is keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Request (default) | Email field and "Send reset link" button | Arrived from "Forgot password?" | User submits the request |
| Confirmation | Neutral confirmation message, no form fields | Reset request submitted (regardless of outcome) | User taps "Back to sign in" or navigates away |
| Reset (link opened) | New password and confirm fields, "Set new password" button | Arrived via a valid, unexpired reset link | User submits a valid new password |
| Reset link invalid or expired | Message: "This reset link is no longer valid. Request a new one." with a "Request new link" button returning to the Request state | Reset token is expired, already used, or malformed | User requests a new link |
| Error | Inline error message; fields retain entered values | Reset submission fails validation or the credential update fails | User corrects input and resubmits |
| Offline/Degraded | Banner: "You're offline. Password reset needs a connection -- try again once you're back online." | Connectivity lost while this screen is open | Connectivity returns |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules) for email format and new-password strength/match rules. Checked on blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | -- |
| "Back to sign in" link tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | -- |
| Successful reset, household exists | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Successful reset, no household yet | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | -- |
| "Request new link" tap (expired link) | Request state of this same screen | -- |

## Data Model

**Creates:** None.
**Reads:** None displayed; the reset token is validated against the account it was issued for.
**Updates:** The account's sign_in credential, on successful completion of the reset step.
**Deletes:** None.

## Business Rules

- Requesting a reset never discloses whether the entered email is registered -- the confirmation message is identical either way, protecting household members' privacy.
- A reset link is single-use and time-limited; using it or letting it expire invalidates it (see FEAT-01.SPEC-017 for delivery and FEAT-01.SPEC-014 for the token's validity window).
- Completing a reset signs the user in immediately -- no separate sign-in step is required afterward.
- Sending the reset email is delegated entirely to FEAT-01.SPEC-017 (Transactional Email Integration); this screen only triggers the request and consumes the resulting link.

## Edge Cases

- **User requests a reset for an unregistered email** -- The confirmation message still appears (no disclosure); no email is actually sent.
- **User taps the reset link twice (opens it in two tabs)** -- The first completed reset invalidates the token; the second tab shows "This reset link is no longer valid. Request a new one." if submitted after the first succeeds.
- **User requests a second reset before completing the first** -- The newer link becomes the valid one; the earlier link is invalidated to prevent confusion about which is current.
- **User submits mismatched new passwords** -- Inline error "Passwords don't match"; the reset link remains valid for further attempts until it expires.
- **User taps "Send reset link" twice rapidly** -- Second tap is ignored while the first request is in flight.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound/outbound) | Entry via "Forgot password?"; returns here on cancel |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | Successful reset for an existing household lands here |
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (outbound) | Successful reset for a household-less account lands here |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Password strength/match and email format rules |
| FEAT-01.SPEC-017 (Transactional Email Integration) | Triggers (outbound) | Sends the reset-link email |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| password_reset_requested | -- (no email captured in the event to avoid logging account-existence signal) | User submits the request step | N/A -- no Stage 2 metric tracks recovery volume; retained as an operational signal for support load |
| password_reset_completed | time since request | User successfully sets a new password | supports success-metrics.md: "First-Session Onboarding Completion" (a blocked sign-in resolved quickly keeps a returning organiser's session productive rather than abandoned) |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Maya cannot remember her password, when she taps "Forgot password?" on FEAT-01.SPEC-001 and enters her registered email and taps "Send reset link", then she sees "If an account exists for {email}, a reset link is on its way. Check your inbox."

**FEAT-01.SPEC-002-AC-02:** Given Sam enters an email that is not registered and taps "Send reset link", then he sees the identical confirmation message as a registered email would produce, and no reset email is sent.

**FEAT-01.SPEC-002-AC-03:** Given Maya opens a valid, unexpired reset link, when she enters a new password meeting the strength requirement in both fields and taps "Set new password", then her sign-in credential updates and she is signed in and taken to FEAT-01.SPEC-010.

**FEAT-01.SPEC-002-AC-04:** Given Sam opens a reset link that has already expired, when the screen loads, then he sees "This reset link is no longer valid. Request a new one." with a button to request a new link.

**FEAT-01.SPEC-002-AC-05:** Given Maya enters two different values in "New password" and "Confirm new password", when she blurs the confirm field, then the error "Passwords don't match" appears and the form does not submit.

**FEAT-01.SPEC-002-AC-06:** Given Maya has requested a reset and then requests a second one before using the first link, when she opens the first (now superseded) link, then it shows as no longer valid.

**FEAT-01.SPEC-002-AC-07:** Given Sam loses connectivity while on the request step, when he taps "Send reset link", then the banner "You're offline. Password reset needs a connection -- try again once you're back online." appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (request, confirmation, reset, invalid link, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

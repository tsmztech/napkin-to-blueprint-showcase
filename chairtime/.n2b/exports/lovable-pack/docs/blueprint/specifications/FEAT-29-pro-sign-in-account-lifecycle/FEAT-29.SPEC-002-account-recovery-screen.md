---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-002
spec_name: Account Recovery Screen
spec_slug: account-recovery-screen
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Account Recovery Screen

## Overview

**Name:** Account Recovery Screen
**ID:** FEAT-29.SPEC-002
**Type:** Screen
**Purpose:** Talia, having lost access to one of her two sign-in contact methods, regains sign-in through whichever contact method she still controls.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Accepting whichever surviving contact method (email or mobile) the Pro still has
- Requesting and validating a one-time code on that surviving contact, identically to ordinary sign-in
- Returning the Pro to a signed-in state without support involvement

**Non-Goals:**
- Ordinary sign-in when both contact methods are still accessible -- handled by FEAT-29.SPEC-001 (Sign-In Screen); this screen exists only for the lost-access path
- Support-assisted recovery -- excluded per scope-boundaries.md SC-05: support has view-only access and can never see sign-in codes or sign in as the Pro; a Pro who loses both contact methods must regain one to recover, with no alternate path
- Changing a sign-in contact detail -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing), reached from FEAT-29.SPEC-003 after the Pro is signed back in; this screen only restores sign-in access, it does not update account data
- Code expiry, lockout, and anti-enumeration rules -- governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules), which this screen references rather than redefines

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Pro taps "Lost access to this contact method?" | None -- recovery starts fresh with no pre-filled identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter the surviving contact method, request and enter a code | -- |
| The Client (Riley) | No | No | Clients never hold a Pro-style sign-in and have no recovery need on this account (FEAT-06 covers client access independently) |
| Platform Operator (Support) | No | No | Recovery is a self-service Pro action only (SC-05); support has no role in it and no view of this screen |
| Unauthenticated | Yes -- this is a pre-sign-in screen | Yes | This screen is itself reachable without a session, by design |
| Expired session | Yes -- reachable via FEAT-29.SPEC-001 | Yes | No special handling beyond the standard expired-session dialog on the Sign-In Screen it is reached from |

## Layout and Content

**Header:** Screen title "Recover your account" with a back arrow (returns to FEAT-29.SPEC-001, Sign-In Screen).

**Body -- Contact step (default):**
- Introductory text: "Enter whichever email or mobile number you still have access to."
- A single text input labeled "Email or mobile number" (required)
- A primary action button "Send code"

**Body -- Code step (after a code is requested):**
- A confirmation line: "We sent a code to {masked identifier}"
- A single numeric code input, 6 digits
- A primary action button "Sign in"
- A secondary text action "Send a new code"

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width inputs and buttons, matching FEAT-29.SPEC-001's layout treatment per the Feature Breakdown Brief's Shared UI Patterns (code entry pattern).
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-001 (Sign-In Screen) | Screen closes | Standard navigation transition |
| Contact input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Contact input | Blur (empty) | Triggers field validation | Error state on field | "Enter your email or mobile number" below field |
| "Send code" button | Tap | Requests a one-time code on the entered contact via FEAT-29.SPEC-006 (Session & Device Management), which triggers FEAT-29.SPEC-014 (Sign-In Code Notification) | Button shows loading state; on success, screen transitions to the Code step | Screen transitions to the Code step showing the masked identifier |
| Code input | Type | Captures numeric input (6 digits) | Field shows entered digits | Standard input focus state; auto-submits at 6 digits |
| "Sign in" button | Tap | Submits the code for validation via FEAT-29.SPEC-011 (Sign-In & Recovery Rules), then FEAT-29.SPEC-006 on success | Button shows loading state | Success: navigates to FEAT-12.SPEC-001 (Today's Upcoming Schedule). Failure: generic error per FEAT-29.SPEC-011 |
| "Send a new code" link | Tap | Requests a fresh code, invalidating the previous one | Code input clears | Confirmation text: "New code sent" |

### Accessibility Notes

- **Focus order:** Back arrow -> Contact input -> "Send code" -> (Code step) Code input -> "Sign in" -> "Send a new code".
- **Validation announcements:** Field errors and the generic failure message are announced to assistive technology, identically to FEAT-29.SPEC-001's code-entry pattern.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Contact (default) | Contact input empty, "Send code" enabled once non-empty | Screen first opens | Pro submits a valid contact identifier |
| Requesting code | "Send code" shows loading spinner | Pro taps "Send code" | Code is sent or request fails |
| Code entry | Masked identifier shown, code input focused and empty | Code sent successfully | Pro submits a code or requests a new one |
| Validating code | "Sign in" shows loading spinner | Pro submits a 6-digit code | Validation succeeds or fails |
| Error | Generic message "That code didn't work. Try again or send a new code." shown below the code input; code input cleared | Code is wrong or expired | Pro re-enters a code or requests a new one |
| Locked | Code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt (FEAT-29.SPEC-011) | platform parameter: `sign-in-lockout-pause-minutes` elapses |
| Offline/Degraded | Banner "You're offline -- recovery needs a live connection." at top; both action buttons disabled | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Code validation (expiry, lockout, anti-enumeration) governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules). This screen applies the same inline input validation as FEAT-29.SPEC-001.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Contact input | Required, non-empty | On blur, on submit | "Enter your email or mobile number" |
| Contact input | Must resemble a valid email or mobile-number format | On submit | "Enter a valid email address or mobile number" |
| Code | Exactly 6 digits | On submit (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-001 (Sign-In Screen) | -- |
| Successful recovery sign-in | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |

## Data Model

**Creates:** None directly -- a successful recovery refreshes or creates a signed-in device via FEAT-29.SPEC-006, identically to ordinary sign-in.
**Reads:** None displayed.
**Updates:** None directly -- code verification and device handling performed by FEAT-29.SPEC-006.
**Deletes:** None.

## Business Rules

- Recovery uses the identical code/lockout/anti-enumeration rules as ordinary sign-in (FEAT-29.SPEC-011) -- there is no separate, weaker recovery-specific validation path.
- A recovery attempt on an identifier with no matching account behaves identically to a matching one (XBR-29, ASMP-30): a code step always follows a "Send code" tap.
- SC-05: recovery is entirely self-service; no support-assisted path exists at any point in this flow.
- Recovering access does not, by itself, change any sign-in contact detail -- the Pro's other, still-valid contact remains on the account exactly as before; changing it is a separate action through FEAT-29.SPEC-003 and FEAT-29.SPEC-010.

## Edge Cases

- **Pro enters the contact method that was actually lost, not the surviving one** -- No special detection exists; if that contact is genuinely unreachable, the code never arrives and the Pro cannot complete recovery through it. The Pro returns to the Contact step (via "Send a new code" or re-entry) and tries the other contact method instead.
- **Pro has lost both contact methods** -- No recovery path exists (SC-05, Non-Goal); the Pro cannot complete this screen and remains signed out. This is an accepted product boundary, not a screen defect.
- **Pro submits the code twice rapidly** -- Second submission is ignored while the first validation is in progress.
- **Network failure while requesting a code** -- Error banner "Could not send code. Check your connection and try again." with a Retry action; the Contact step is preserved.
- **Pro navigates away mid-code-entry and returns** -- The Code step re-appears with the code input empty; a previously requested code remains valid until expiry (platform parameter: `sign-in-code-expiry-minutes`) or a new one is requested.
- **No concurrent-edit conflict applies** -- This screen creates no record content of its own; there is nothing here for another actor to have changed concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Navigation (inbound/outbound) | Entry point via "Lost access" link; back arrow returns there |
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Requesting and validating the recovery code |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, and anti-enumeration rules |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Triggers (outbound) | A requested code is delivered through this notification |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (outbound) | Default landing after successful recovery |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_recovery_started | -- | Pro submits a contact identifier on this screen | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so recovery usage remains observable |
| account_recovery_succeeded | -- | Recovery code validates and the Pro is signed in | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so recovery reliability remains observable |

## Acceptance Criteria

**FEAT-29.SPEC-002-AC-01:** Given Talia has lost her old phone but still has her sign-in email, when she enters her email on this screen and taps "Send code", then the screen transitions to the Code step showing "We sent a code to t***a@gmail.com".

**FEAT-29.SPEC-002-AC-02:** Given Talia is on the Code step with a valid, unexpired recovery code, when she enters the correct 6 digits, then she is signed in and lands on FEAT-12.SPEC-001.

**FEAT-29.SPEC-002-AC-03:** Given Talia enters an incorrect recovery code, when she submits it, then the screen shows "That code didn't work. Try again or send a new code." and the code input clears.

**FEAT-29.SPEC-002-AC-04:** Given Talia has failed 5 consecutive recovery attempts, when she tries to enter another code, then the screen shows "Too many attempts. Try again in 15 minutes." for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-002-AC-05:** Given Talia enters a contact identifier with no matching account, when she taps "Send code", then the screen behaves identically to a matching identifier with no indication the account does not exist.

**FEAT-29.SPEC-002-AC-06:** Given Talia has lost both her sign-in contact methods, when she attempts recovery, then no recovery path completes for her, and no support-assisted alternative is offered anywhere on this screen.

**FEAT-29.SPEC-002-AC-07:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-001 (Sign-In Screen).

**FEAT-29.SPEC-002-AC-08:** Given Talia is on the Code step, when she taps "Send a new code", then a fresh code is requested, the previous code becomes invalid, and the confirmation "New code sent" appears.

**FEAT-29.SPEC-002-AC-09:** Given Talia loses connectivity while on this screen, when she is on either step, then the banner "You're offline -- recovery needs a live connection." appears and both action buttons are disabled.

**FEAT-29.SPEC-002-AC-10:** Given Talia completes recovery successfully, when sign-in succeeds, then her other, still-valid sign-in contact remains unchanged on her account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (requesting, code entry, error, locked, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-001
spec_name: Sign-In Screen
spec_slug: sign-in-screen
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Sign-In Screen

## Overview

**Name:** Sign-In Screen
**ID:** FEAT-29.SPEC-001
**Type:** Screen
**Purpose:** Talia enters her sign-in email or mobile number, requests a one-time code, and enters it to sign in -- and it is the screen every unauthenticated visitor to a Pro-only area lands on (XBR-29).
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Collecting the sign-in identifier (email or mobile number) and requesting a one-time code
- The code-entry step and its generic failure handling
- Serving as the redirect destination for every unauthenticated visitor to a Pro-only screen (XBR-29)
- Serving as the onboarding entry point where a brand-new Pro first establishes their sign-in identity (email, mobile, first device)

**Non-Goals:**
- Recovering access when one contact method is lost -- owned by FEAT-29.SPEC-002 (Account Recovery Screen); this screen assumes the Pro still has access to at least the one contact method they enter here
- Verifying the submitted code and creating the signed-in device -- owned by FEAT-29.SPEC-006 (Session & Device Management); this screen only collects input and displays the outcome
- Password-based sign-in or a "remember this password" pattern -- excluded per product-features.md's Key Capabilities ("a one-time code rather than a password to remember") and ASMP-30
- Full Pro Account profile creation -- owned by FEAT-15.SPEC-004, which attaches this screen's established sign-in identity to the new account record; this screen establishes sign-in identity only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard, account setup step) | A brand-new Pro starts setup | None -- first-time sign-in creation, no existing identifier |
| FEAT-12, FEAT-01, FEAT-02, FEAT-13, FEAT-15 (SPEC-001/003), FEAT-17, FEAT-27, FEAT-28 (SPEC-001/002), FEAT-30, and every other Pro-facing feature's own access-authorization check | A visitor with no valid session reaches a Pro-only screen (XBR-29) | The originally requested destination, so sign-in returns the Pro there rather than always to the dashboard |
| External / direct link | Pro opens the product with no active session (e.g., bookmarked link, new device) | None |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Talia confirms "Sign out everywhere" | None -- every session has ended; a fresh sign-in starts |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter sign-in identifier, request a code, enter the code | -- |
| The Client (Riley) | No | No | Clients never hold a Pro-style sign-in (FEAT-06 gives them a password-free identity instead); a client who reaches this URL directly sees the same generic screen but has no path that leads a signed-in client session here |
| Platform Operator (Support) | No | No | Support has no sign-in of their own on this screen; support access is a separate, internal entry point (FEAT-19) that never uses this Pro sign-in flow |
| Unauthenticated | Yes -- this is the destination screen | Yes -- this is the only action available | This is where every unauthenticated visitor to a Pro-only screen lands (XBR-29); no further redirect |
| Expired session | Yes -- redirected here | Yes | Dialog "Your session has expired. Sign in to continue." appears once on arrival; any in-progress, unsaved work on the screen the Pro was on is not preserved (per that screen's own Edge Cases), except read-only schedule data cached for offline viewing (ASMP-27), which remains available after re-authentication |

## Layout and Content

**Header:** Chairtime wordmark, centered, no navigation controls (this is an entry screen with no "back").

**Body -- Identifier step (default):**
- A single text input labeled "Email or mobile number" (accepts either format, required)
- A primary action button "Send code"
- Below the button, a line of text: "We'll text or email you a one-time code to sign in -- no password needed."

**Body -- Code step (after a code is requested):**
- A confirmation line: "We sent a code to {masked identifier}" (e.g., "t***a@gmail.com" or "(***) ***-1234")
- A single numeric code input, 6 digits
- A primary action button "Sign in"
- A secondary text action "Send a new code"
- A secondary text action "Use a different email or mobile number" (returns to the identifier step)

**Footer:** A single text link, "Lost access to this contact method?", navigating to FEAT-29.SPEC-002 (Account Recovery Screen).

### Responsive Behavior

- **Compact breakpoint:** Single-column, vertically centered content, full-width inputs and buttons.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Identifier input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Identifier input | Blur (empty) | Triggers field validation | Error state on field | "Enter your email or mobile number" below field |
| "Send code" button | Tap | Requests a one-time code via FEAT-29.SPEC-006 (Session & Device Management), which triggers FEAT-29.SPEC-014 (Sign-In Code Notification) | Button shows loading state; on success, screen transitions to the Code step | Screen transitions to the Code step showing the masked identifier |
| Code input | Type | Captures numeric input (6 digits) | Field shows entered digits | Standard input focus state; auto-submits when 6 digits are entered |
| "Sign in" button | Tap | Submits the code for validation via FEAT-29.SPEC-011 (Sign-In & Recovery Rules), then FEAT-29.SPEC-006 (Session & Device Management) on success | Button shows loading state | Success: navigates to FEAT-12.SPEC-001 (Today's Upcoming Schedule) or the originally requested Pro-only screen. Failure: generic error shown per FEAT-29.SPEC-011 |
| "Send a new code" link | Tap | Requests a fresh code, invalidating the previous one, via FEAT-29.SPEC-006 | Code input clears | Confirmation text: "New code sent" |
| "Use a different email or mobile number" link | Tap | Returns to the Identifier step | Screen reverts to Identifier step, code input cleared | Identifier field is empty and focused |
| "Lost access to this contact method?" link | Tap | Navigate to FEAT-29.SPEC-002 (Account Recovery Screen) | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Identifier input -> "Send code" -> (Code step) Code input -> "Sign in" -> "Send a new code" -> "Use a different email or mobile number" -> "Lost access to this contact method?".
- **Validation announcements:** Field errors and the generic "that code didn't work" message are announced to assistive technology and programmatically associated with the relevant field.
- **Transition announcements:** The transition from the Identifier step to the Code step is announced ("Code sent to {masked identifier}"), and the numeric code input receives focus automatically.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures. Code auto-submit on 6 digits does not remove the explicit "Sign in" button as a keyboard-operable alternative.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Identifier (default) | Identifier input empty, "Send code" enabled once non-empty | Screen first opens | Pro submits a valid identifier |
| Requesting code | "Send code" shows loading spinner | Pro taps "Send code" | Code is sent (success) or request fails |
| Code entry | Masked identifier shown, code input focused and empty | Code sent successfully | Pro submits a code, requests a new one, or goes back |
| Validating code | "Sign in" shows loading spinner | Pro submits a 6-digit code | Validation succeeds or fails |
| Error | Generic error message shown per FEAT-29.SPEC-011 ("That code didn't work. Try again or send a new code.") below the code input; code input cleared for re-entry | Code is wrong or expired | Pro re-enters a code or requests a new one |
| Locked | Code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt (FEAT-29.SPEC-011) | platform parameter: `sign-in-lockout-pause-minutes` elapses |
| Offline/Degraded | Banner "You're offline -- signing in needs a live connection." at top; both "Send code" and "Sign in" are disabled while offline | Connectivity lost while screen is open | Connectivity restored -- controls re-enable, no request was queued (signing in is never queued, per ASMP-27) |

## Validation Rules

Validation of the code itself (expiry, lockout, anti-enumeration) is governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules). This screen applies the following inline input validation.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Identifier | Required, non-empty | On blur, on submit | "Enter your email or mobile number" |
| Identifier | Must resemble a valid email or mobile-number format | On submit | "Enter a valid email address or mobile number" |
| Code | Exactly 6 digits | On submit (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful sign-in (default landing) | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful sign-in (redirected from a Pro-only screen) | The originally requested screen | Varies -- the feature that redirected here |
| "Lost access to this contact method?" tap | FEAT-29.SPEC-002 (Account Recovery Screen) | -- |

## Data Model

**Creates:** On first-time onboarding use only -- the sign-in identity slice of the Pro Account (sign_in_email, sign_in_mobile) and the first signed_in_devices entry, established by FEAT-29.SPEC-006 once the first code is verified; the full Pro Account record is then created by FEAT-15.SPEC-004 around this identity.
**Reads:** None displayed -- this screen never shows existing account data, only the masked identifier the Pro just entered.
**Updates:** None directly -- code verification and device creation/refresh are performed by FEAT-29.SPEC-006.
**Deletes:** None.

## Business Rules

- Sign-in code behavior (expiry, lockout, anti-enumeration) is governed entirely by FEAT-29.SPEC-011 -- this screen never defines or duplicates those rules.
- A failed sign-in never reveals whether an account exists for the entered identifier (XBR-29, ASMP-30): the same generic "that code didn't work" message and the same code-request behavior appear whether or not the identifier matches an account.
- XBR-29: every Pro-facing screen requires a signed-in Pro; this screen is the universal redirect destination for anyone who is not.
- On successful onboarding-time sign-in creation, FEAT-29.SPEC-006 hands the established identity to FEAT-15.SPEC-004, which creates the Pro Account record around it.

## Edge Cases

- **Pro enters an identifier with no matching account** -- The screen behaves identically to a matching identifier: a code is "sent" (or silently discarded server-side) and the Code step appears normally, so no enumeration signal is given (XBR-29).
- **Pro submits the code twice rapidly** -- Second submission is ignored while the first validation is in progress ("Sign in" in loading state).
- **Pro navigates away mid-code-entry and returns** -- The Code step re-appears with the code input empty; a previously requested code remains valid until it expires (platform parameter: `sign-in-code-expiry-minutes`) or a new one is requested.
- **Pro is redirected here from a deep Pro-only screen and abandons sign-in** -- No further redirect loop; the Pro simply remains on this screen until they sign in or leave the product.
- **Network failure while requesting a code** -- Error banner "Could not send code. Check your connection and try again." with a Retry action; the Identifier step is preserved.
- **No concurrent-edit conflict applies** -- This screen creates no record content of its own (only requests and validates a code); there is nothing here for another actor to have changed concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Requesting a code and submitting a code both invoke this automation |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, and anti-enumeration rules applied to this screen's code step |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Triggers (outbound) | A requested code is delivered through this notification |
| FEAT-29.SPEC-002 (Account Recovery Screen) | Navigation (outbound) | "Lost access to this contact method?" link |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound) | Onboarding's account setup step hands the Pro into sign-in creation here |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (outbound) | Default landing after sign-in |
| FEAT-12, FEAT-01, FEAT-02, FEAT-13, FEAT-15, FEAT-17, FEAT-27, FEAT-28, FEAT-30 | Navigation (inbound) | Every Pro-facing feature's own access-authorization check redirects an unauthenticated visitor here (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| pro_signed_in | entry context (default / redirected), new_device (yes/no) | Sign-in succeeds | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so sign-in reliability remains observable |
| sign_in_code_failed | reason (wrong_code / expired / locked_out) | A code attempt fails | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so sign-in failure patterns remain observable |
| sign_in_code_requested | request_type (initial / resend) | Pro requests a code | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so code-delivery volume is observable |

## Acceptance Criteria

**FEAT-29.SPEC-001-AC-01:** Given Talia is on the Sign-In Screen's Identifier step, when she enters her sign-in email and taps "Send code", then the screen transitions to the Code step showing "We sent a code to t***a@gmail.com".

**FEAT-29.SPEC-001-AC-02:** Given Talia is on the Identifier step, when she taps "Send code" with the field empty, then the field shows the error "Enter your email or mobile number" and no request is sent.

**FEAT-29.SPEC-001-AC-03:** Given Talia is on the Code step with a valid, unexpired code sent, when she enters the correct 6 digits, then she is signed in and lands on FEAT-12.SPEC-001 (Today's Upcoming Schedule).

**FEAT-29.SPEC-001-AC-04:** Given Talia enters an incorrect code, when she submits it, then the screen shows the generic message "That code didn't work. Try again or send a new code." and the code input clears.

**FEAT-29.SPEC-001-AC-05:** Given Talia has failed 5 consecutive attempts, when she tries to enter another code, then the screen shows "Too many attempts. Try again in 15 minutes." and further submission is disabled for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-001-AC-06:** Given an unauthenticated visitor reaches FEAT-12.SPEC-001 directly, when the dashboard's access check runs, then they are redirected to this Sign-In Screen, and on successful sign-in they land back on FEAT-12.SPEC-001.

**FEAT-29.SPEC-001-AC-07:** Given Talia enters an identifier with no matching account, when she taps "Send code", then the screen behaves identically to a matching identifier (transitions to the Code step) with no indication that the account does not exist.

**FEAT-29.SPEC-001-AC-08:** Given Talia is on the Code step, when she taps "Send a new code", then a fresh code is requested, the previous code becomes invalid, and the code input clears with the confirmation "New code sent".

**FEAT-29.SPEC-001-AC-09:** Given Talia signs in on a device not seen before, when sign-in succeeds, then FEAT-29.SPEC-015 (New-Device Sign-In Alert) is triggered.

**FEAT-29.SPEC-001-AC-10:** Given Talia loses connectivity while on this screen, when she is on either the Identifier or Code step, then the banner "You're offline -- signing in needs a live connection." appears and both action buttons are disabled until connectivity returns.

**FEAT-29.SPEC-001-AC-11:** Given Talia has an expired session while she had unsaved changes on another Pro screen, when she is redirected here, then the dialog "Your session has expired. Sign in to continue." appears once, and after she signs in successfully she is returned to the screen she was on (with unsaved changes handled per that screen's own Edge Cases).

**FEAT-29.SPEC-001-AC-12:** Given Talia taps "Lost access to this contact method?", when the tap registers, then she is navigated to FEAT-29.SPEC-002 (Account Recovery Screen).

**FEAT-29.SPEC-001-AC-13:** Given a brand-new Pro reaches this screen from FEAT-15's account setup step with no prior sign-in identity, when she completes the code step for the first time, then FEAT-29.SPEC-006 establishes her sign-in identity and first device, and FEAT-15.SPEC-004 attaches it to a newly created Pro Account record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (requesting, code entry, error, locked, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

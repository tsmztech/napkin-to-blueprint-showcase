---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-006
spec_name: Session & Device Management
spec_slug: session-device-management
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Session & Device Management

## Overview

**Name:** Session & Device Management
**ID:** FEAT-29.SPEC-006
**Type:** Automation
**Purpose:** Requests and verifies one-time sign-in codes, creates or refreshes a signed-in device on successful verification, expires devices after inactivity, and executes "sign out everywhere."
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Generating and delivering a one-time code on request (sign-in, recovery, or onboarding)
- Verifying a submitted code against the rules FEAT-29.SPEC-011 defines
- Creating the sign-in identity and first signed-in device on a brand-new Pro's first successful onboarding code entry
- Creating or refreshing a signed-in device on every successful verification thereafter
- Expiring a device after 30 days of inactivity (platform parameter: `session-inactivity-expiry-days`)
- Executing "sign out everywhere"

**Non-Goals:**
- Defining code expiry, the failed-attempt lockout, session duration, or the anti-enumeration rule -- owned by FEAT-29.SPEC-011 (Sign-In & Recovery Rules); this automation enforces those rules, it does not define them
- Creating the full Pro Account record -- owned by FEAT-15.SPEC-004, which attaches the sign-in identity this automation establishes to the new account record
- Delivering the code content itself -- owned by FEAT-29.SPEC-014 (Sign-In Code Notification); this automation only triggers that delivery
- Alerting the Pro to a new-device sign-in -- owned by FEAT-29.SPEC-015 (New-Device Sign-In Alert); this automation only detects the new-device condition and triggers that notification

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests a one-time code (sign-in) | FEAT-29.SPEC-001 (Sign-In Screen) | Pro submits an identifier and taps "Send code" | Entered identifier (email or mobile format) |
| Pro requests a one-time code (recovery) | FEAT-29.SPEC-002 (Account Recovery Screen) | Pro submits a surviving contact identifier | Entered identifier |
| Pro requests a one-time code (onboarding) | FEAT-29.SPEC-001 (Sign-In Screen, onboarding entry) | Brand-new Pro establishing sign-in for the first time | Entered identifier, no existing account reference |
| Pro submits a code | FEAT-29.SPEC-001 or FEAT-29.SPEC-002 | Pro enters 6 digits and submits (or auto-submits) | Submitted code, the identifier the code was requested for, device fingerprint of the requesting browser/device |
| Pro chooses "sign out everywhere" | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro confirms the "Sign out everywhere" dialog | Pro Account reference |
| Pro signs out an individual device | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Sign out" on one device row | Pro Account reference, the specific device identifier |
| Scheduled inactivity check | System | Runs on a recurring schedule to find signed-in devices past the inactivity limit | Every signed_in_devices entry across all Pro Accounts, with each entry's last-active timestamp |

## Processing Logic

**Code request path:**
1. Receive the identifier (email or mobile) submitted from the triggering screen.
2. Look up whether the identifier matches an existing Pro Account's sign_in_email or sign_in_mobile. Regardless of the result, proceed identically (per FEAT-29.SPEC-011's anti-enumeration rule).
3. Generate a new one-time code, superseding any previously issued, unexpired code for this identifier.
4. Record the code's issue time (for the expiry check on submission) against the identifier.
5. Trigger FEAT-29.SPEC-014 (Sign-In Code Notification) to deliver the code to the entered contact.
6. Signal the triggering screen that a code was sent (identical signal whether or not a matching account exists).

**Code submission path:**
1. Receive the submitted code, the identifier it was requested for, and the requesting device's fingerprint.
2. Evaluate the submission against FEAT-29.SPEC-011's rules: expiry, remaining lockout attempts.
3. If invalid (wrong or expired): increment the failed-attempt counter for this identifier and signal the generic failure back to the triggering screen. If this increment reaches the lockout threshold (FEAT-29.SPEC-011), begin the lockout pause.
4. If valid and the identifier matches an existing Pro Account: check whether the requesting device fingerprint matches an existing entry in signed_in_devices.
   - If it matches an existing entry: refresh that entry's last-active timestamp.
   - If it does not match any existing entry: create a new signed_in_devices entry (device description, approximate location/type if available, creation timestamp as last-active) and trigger FEAT-29.SPEC-015 (New-Device Sign-In Alert).
5. If valid and no Pro Account matches (brand-new onboarding sign-in): establish the sign-in identity (sign_in_email and/or sign_in_mobile, whichever was entered) and create the first signed_in_devices entry; hand this identity off to FEAT-15.SPEC-004 to attach to the new Pro Account record it creates. No new-device alert is sent for this first device (there is no prior device to alert from, and no established account yet to receive it on).
6. Reset the failed-attempt counter for this identifier on any successful verification.
7. Signal the triggering screen that sign-in succeeded.

**Sign-out path:**
1. Receive the sign-out request (individual device or everywhere) and the Pro Account reference.
2. For "sign out everywhere": remove every entry from signed_in_devices, including the requesting session's own entry.
3. For an individual device: remove only the specified entry from signed_in_devices.
4. Signal the triggering screen (and, for "everywhere," every other active session) that the affected device(s) are signed out.

**Scheduled inactivity path:**
1. On each scheduled run, read every signed_in_devices entry across all Pro Accounts.
2. For each entry, compare its last-active timestamp against the current time.
3. If the gap exceeds platform parameter: `session-inactivity-expiry-days`, remove that entry silently.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Code sent | Code request processed, regardless of account match | New code recorded against the identifier | Screen transitions to the Code step | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-014 |
| Code sent, request fails | Notification capability cannot be reached | No code state change persists as delivered | Error banner "Could not send code. Check your connection and try again." | FEAT-29.SPEC-001, FEAT-29.SPEC-002 |
| Verification succeeds, existing device | Correct, unexpired code; fingerprint matches an existing entry | signed_in_devices entry's last-active refreshed | Pro is signed in and navigated onward | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-12.SPEC-001 |
| Verification succeeds, new device | Correct, unexpired code; fingerprint matches no existing entry | New signed_in_devices entry created | Pro is signed in and navigated onward; FEAT-29.SPEC-015 fires separately | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-015 |
| Verification succeeds, first-time onboarding | Correct, unexpired code; no existing Pro Account for this identifier | Sign-in identity and first device established; handed to FEAT-15.SPEC-004 | Pro is signed in and continues onboarding | FEAT-15.SPEC-004 |
| Verification fails (wrong/expired) | Code does not match or has passed expiry | Failed-attempt counter incremented | Generic message: "That code didn't work. Try again or send a new code." | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-011 |
| Verification locked out | Failed-attempt counter reaches the lockout threshold | Lockout pause begins for this identifier | "Too many attempts. Try again in {remaining minutes} minutes." | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-011 |
| Individual device signed out | Pro confirms sign-out on one device row | That signed_in_devices entry removed | Device row disappears; toast confirms | FEAT-29.SPEC-003 |
| Signed out everywhere | Pro confirms "Sign out everywhere" | Every signed_in_devices entry removed | Every session (including the current one) is redirected to FEAT-29.SPEC-001 | FEAT-29.SPEC-003, FEAT-29.SPEC-001 |
| Device expired by inactivity | Scheduled check finds an entry past the inactivity limit | That entry removed | No direct user feedback at the moment of expiry; the device simply no longer appears in the list and requires a fresh sign-in next time it is used | FEAT-29.SPEC-003 |
| Automation failure (verification path) | Processing error during verification | No sign-in state changes | Error banner "Something went wrong. Try again." on the triggering screen; the Pro is not signed in | FEAT-29.SPEC-001, FEAT-29.SPEC-002 |

## Data Model

**Reads:** Pro Account -- sign_in_email, sign_in_mobile, signed_in_devices (for lookup and matching during code request and verification).
**Creates:** Pro Account's signed_in_devices entries (on new-device verification, and the first entry on onboarding); the sign-in identity slice (sign_in_email / sign_in_mobile) on first-time onboarding verification.
**Updates:** Pro Account.signed_in_devices (last-active refresh on existing-device verification; entry removal on sign-out or inactivity expiry).
**Deletes:** Pro Account.signed_in_devices entries (individual sign-out, sign-out-everywhere, inactivity expiry).

## Business Rules

- XBR-29: every Pro-facing screen requires a signed-in Pro; this automation is the sole mechanism that establishes that signed-in state.
- Code expiry, lockout threshold and pause duration, and the anti-enumeration guarantee are all defined by FEAT-29.SPEC-011 and enforced here without exception.
- A new-device sign-in always triggers FEAT-29.SPEC-015, except for the very first device created during onboarding (there is no prior device or established account to alert).
- "Sign out everywhere" always includes the requesting session's own device -- there is no way to sign out every device except the current one.
- Device inactivity expiry (platform parameter: `session-inactivity-expiry-days`) is silent -- it produces no notification, since it is a routine housekeeping outcome, not a security event.

## Edge Cases

- **Two devices submit the correct code for the same identifier at nearly the same time (e.g., a code shared or intercepted)** -- Both verifications succeed independently against the same valid code (the code itself is not single-use once issued, only time-limited); each creates or refreshes its own device entry. A resulting unexpected device is visible to the Pro on FEAT-29.SPEC-003 and can be signed out individually.
- **Pro requests a new code while a previous one is still valid** -- The new code supersedes the previous one; the previous code becomes invalid immediately, so a Pro who has an old code page open and a new one open only succeeds with the latest.
- **Concurrent trigger firing -- two code requests for the same identifier fire at effectively the same time (e.g., "Send code" tapped, then "Send a new code" tapped in quick succession before the first delivery completes)** -- Each request generates its own code and supersedes the previous; only the most recently generated code validates. Both notification sends are attempted independently; a duplicate arriving is a minor inconvenience, never a validation error.
- **Trigger fires while a previous run is in flight -- a code submission is in progress when the scheduled inactivity check runs against the same Pro Account** -- The inactivity check operates only on last-active timestamps unaffected by an in-flight verification; the verification's device-entry write (refresh or creation) and the inactivity check's entry removal cannot target the same entry in a way that conflicts, since a device actively completing verification has just updated its last-active timestamp and so cannot simultaneously qualify as inactive.
- **Scheduled inactivity check runs while the Pro is actively using a device that is, by clock skew, borderline past the limit** -- Any activity (a successful verification, which the check does not target, or an in-product action that refreshes last-active) updates the timestamp before the check can remove the entry; the check only ever removes entries with no qualifying activity at all in the window.
- **A device is removed by "sign out everywhere" while its scheduled inactivity check is also about to run** -- The sign-out removal is immediate and idempotent; the scheduled check finds no matching entry and takes no further action for that device.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Triggered by (inbound) | Code request and code submission |
| FEAT-29.SPEC-002 (Account Recovery Screen) | Triggered by (inbound) | Code request and code submission for recovery |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Triggered by (inbound) | Individual sign-out and sign-out-everywhere |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, anti-enumeration rules enforced here |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Affects (outbound) | Triggered to deliver every requested code |
| FEAT-29.SPEC-015 (New-Device Sign-In Alert) | Affects (outbound) | Triggered when a new device is created (post-onboarding) |
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Affects (outbound) | Receives the established sign-in identity for a brand-new Pro Account |

## Analytics and Success Signals

- **sign_in_code_requested** (path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so code-delivery volume is observable
- **pro_signed_in** (new_device: yes/no, path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so sign-in success remains observable
- **sign_in_code_failed** (reason: wrong_code / expired / locked_out) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so failure patterns remain observable
- **signed_out_everywhere** (device_count) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **device_expired_inactivity** (-- no user-facing properties, a silent housekeeping event) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so inactivity-expiry volume remains observable for operational monitoring

## Acceptance Criteria

**FEAT-29.SPEC-006-AC-01:** Given Talia submits her sign-in email on FEAT-29.SPEC-001, when the request is processed, then a new code is generated and FEAT-29.SPEC-014 is triggered to deliver it, regardless of whether an account matches.

**FEAT-29.SPEC-006-AC-02:** Given Talia submits the correct, unexpired code from a device that already has a signed_in_devices entry, when verification runs, then that entry's last-active timestamp is refreshed and no new-device alert fires.

**FEAT-29.SPEC-006-AC-03:** Given Talia submits the correct, unexpired code from a device with no existing entry, when verification runs, then a new signed_in_devices entry is created and FEAT-29.SPEC-015 (New-Device Sign-In Alert) is triggered.

**FEAT-29.SPEC-006-AC-04:** Given a brand-new Pro submits the correct code for the first time during onboarding, when verification runs, then her sign-in identity and first device are established and handed to FEAT-15.SPEC-004, with no new-device alert sent.

**FEAT-29.SPEC-006-AC-05:** Given Talia submits an incorrect code, when verification runs, then her failed-attempt counter increments and the generic failure message is returned.

**FEAT-29.SPEC-006-AC-06:** Given Talia's failed-attempt counter reaches the lockout threshold (FEAT-29.SPEC-011), when she attempts another submission, then the lockout pause begins and further attempts are blocked for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-006-AC-07:** Given Talia taps "Sign out" on one device, when the request is processed, then only that device's entry is removed from signed_in_devices.

**FEAT-29.SPEC-006-AC-08:** Given Talia confirms "Sign out everywhere", when the request is processed, then every signed_in_devices entry is removed, including her current session's.

**FEAT-29.SPEC-006-AC-09:** Given a signed_in_devices entry has had no activity for longer than platform parameter: `session-inactivity-expiry-days`, when the scheduled inactivity check runs, then that entry is removed silently with no notification.

**FEAT-29.SPEC-006-AC-10:** Given a signed_in_devices entry had activity within platform parameter: `session-inactivity-expiry-days`, when the scheduled inactivity check runs, then that entry is left unchanged.

**FEAT-29.SPEC-006-AC-11:** Given Talia requests a new code while a previously issued code for the same identifier is still unexpired, when the new code is generated, then the previous code becomes invalid immediately.

**FEAT-29.SPEC-006-AC-12:** Given two devices submit the same valid, unexpired code for the same identifier at nearly the same time, when both verifications process, then both succeed and each creates or refreshes its own device entry independently.

**FEAT-29.SPEC-006-AC-13:** Given the verification path encounters a processing error, when the failure occurs, then the triggering screen shows "Something went wrong. Try again." and the Pro is not signed in.

**FEAT-29.SPEC-006-AC-14:** Given a device is removed via "Sign out everywhere" at the same time its scheduled inactivity check would otherwise run, when the scheduled check executes, then it finds no matching entry and takes no further action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 11 | 11 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

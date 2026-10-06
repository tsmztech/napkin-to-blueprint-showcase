---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-29.SPEC-011
spec_name: Sign-In & Recovery Rules
spec_slug: sign-in-recovery-rules
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Sign-In & Recovery Rules

## Overview

**Name:** Sign-In & Recovery Rules
**ID:** FEAT-29.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs one-time-code expiry, the failed-attempt lockout, session duration, and the anti-enumeration rule that a failed sign-in never reveals whether an account exists.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle
**Governed Entity:** Pro Account (sign-in slice: sign_in_email, sign_in_mobile, signed_in_devices) and the transient one-time-code state associated with a sign-in or recovery attempt

## Scope and Non-Goals

**In Scope:**
- One-time-code expiry
- Failed-attempt lockout threshold and pause duration
- Signed-in device (session) inactivity duration
- The anti-enumeration rule applied to every sign-in and recovery attempt
- Authorization for every action on the sign-in identity slice of the Pro Account

**Non-Goals:**
- Contact-detail change confirmation rules -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this spec governs sign-in and recovery only, not changing the sign-in contacts themselves
- Account closure and retention rules -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules)
- The step-by-step processing of a code request or verification -- owned by FEAT-29.SPEC-006 (Session & Device Management), which enforces these rules; this spec defines the rules, it does not execute them
- Support-assisted recovery -- excluded per scope-boundaries.md SC-05: no rule in this spec creates a path for support to see codes, sign in as the Pro, or bypass lockout on the Pro's behalf

## Governed Entity

**Entity:** Pro Account (sign-in slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| sign_in_email | text | Required sign-in email; each change confirmed through old and new contact (governed by FEAT-29.SPEC-012, not this spec) |
| sign_in_mobile | text | Required sign-in mobile number; each change confirmed through old and new contact (governed by FEAT-29.SPEC-012, not this spec) |
| signed_in_devices | derived (list) | Active sign-ins, up to platform parameter: `session-inactivity-expiry-days` of inactivity each |
| one_time_code (transient) | derived | The current unexpired code issued for a sign-in or recovery attempt against a given identifier; not a persisted Pro Account field |
| failed_attempt_count (transient) | derived | Count of consecutive failed code submissions for a given identifier since the last success or lockout reset |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-001 | Sign-In Screen | On code submission; anti-enumeration applied on every code request regardless of match |
| FEAT-29.SPEC-002 | Account Recovery Screen | On code submission; anti-enumeration applied on every code request regardless of match |
| FEAT-29.SPEC-006 | Session & Device Management | During code generation, verification, and device inactivity expiry -- the automation that actually applies these rules |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| one_time_code | Must be submitted within platform parameter: `sign-in-code-expiry-minutes` of issue | Always | On code submission | "That code didn't work. Try again or send a new code." | Yes |
| one_time_code | Must exactly match the most recently issued code for the identifier | Always | On code submission | "That code didn't work. Try again or send a new code." | Yes |
| failed_attempt_count | Blocks further submission once it reaches platform parameter: `sign-in-lockout-threshold` | Always | On each submission attempt | "Too many attempts. Try again in {remaining minutes} minutes." | Yes |
| signed_in_devices entry | Expires after platform parameter: `session-inactivity-expiry-days` of no activity | Always | On the scheduled inactivity check (FEAT-29.SPEC-006) | No error message shown -- silent expiry per Business Rules | No (a housekeeping removal, not a validation failure) |
| sign_in_email / sign_in_mobile | No validation beyond data type in this spec | Always | -- | -- | -- (format and dual-confirmation validation for a change is FEAT-29.SPEC-012's) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Anti-enumeration | one_time_code, sign_in_email, sign_in_mobile | Whether or not the submitted identifier matches an existing Pro Account, the code-request and code-verification behavior (timing, messaging, and outcome shape) is identical | "That code didn't work. Try again or send a new code." (same message whether or not an account exists) |
| Lockout resets on success | failed_attempt_count, one_time_code | A successful verification resets failed_attempt_count to zero for that identifier | -- |
| New code supersedes prior code | one_time_code | Requesting a new code invalidates any previously issued, unexpired code for the same identifier immediately | -- |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Request a one-time code | The Pro (Talia) -- anyone submitting an identifier, since the action must not reveal account existence | Always | -- |
| Submit a one-time code | The Pro (Talia) | Only while failed_attempt_count is below platform parameter: `sign-in-lockout-threshold` | "Too many attempts. Try again in {remaining minutes} minutes." -- code input and further submission disabled for platform parameter: `sign-in-lockout-pause-minutes` |
| View signed-in devices | The Pro (Talia) | Only their own account's device list | -- |
| View signed-in devices | Platform Operator (Support) | Never -- Support sees account status only, never sign-in codes or devices (XBR-24, ASMP-30) | The signed-in device list is not rendered on Support's view of FEAT-29.SPEC-003; only the status line appears |
| Sign out a device | The Pro (Talia) | Only their own account's devices | -- |
| Sign out a device | Platform Operator (Support) | Never | No sign-out control is rendered anywhere on Support's view |
| Sign in as the Pro | Platform Operator (Support) | Never (SC-05) | No such action exists in the product at all -- there is no control, screen, or code path by which Support can assume the Pro's session |
| View another Pro's sign-in identity or devices | The Pro (Talia) | Never -- a sign-in identity belongs to exactly one Pro Account (dependency map: Pro Account Relationships) | No cross-account query path exists; the action is unreachable rather than denied with a message |
| View another Pro's sign-in identity or devices | Platform Operator (Support) | Never -- Support views one Pro account at a time, only after that Pro's own help request (XBR-24) | Support's view is always scoped to the single account opened via a help request; there is no listing or search across accounts |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| failed_attempt_count | Starts at zero for a new identifier or after a successful verification | On first code request for an identifier, and on every successful verification | No |
| signed_in_devices entry's last-active timestamp | Set to the current time on creation; refreshed to the current time on every successful verification from that device | On device creation and every subsequent successful verification | No |

## Business Rules

- One-time codes expire after platform parameter: `sign-in-code-expiry-minutes` (10 minutes, per product-features.md's stated Validation & Limits).
- After platform parameter: `sign-in-lockout-threshold` (5) consecutive failed attempts, further attempts are paused for platform parameter: `sign-in-lockout-pause-minutes` (15 minutes).
- A signed-in device stays active for up to platform parameter: `session-inactivity-expiry-days` (30) days of inactivity, after which FEAT-29.SPEC-006 expires it silently.
- XBR-29: every Pro-facing screen requires a signed-in Pro; anyone else is sent to FEAT-29.SPEC-001, and a failed sign-in never reveals whether an account exists.
- ASMP-30: account protection rests on one-time-code sign-in with new-device alerts (FEAT-29.SPEC-015); no password or remembered credential ever exists as an alternative path.

## Edge Cases

- **Code submitted at exactly platform parameter: `sign-in-code-expiry-minutes` after issue** -- Treated as expired; the boundary itself does not validate. A code must be submitted strictly before the expiry boundary.
- **Fifth failed attempt arrives at the exact same moment a valid code would have been accepted (e.g., the Pro's fifth submission is actually correct)** -- The lockout threshold is evaluated on failed attempts only; a correct fifth submission is a success, not a failure, and resets failed_attempt_count to zero rather than triggering lockout. Lockout triggers only when the fifth attempt is itself also incorrect.
- **Pro attempts a code submission during an active lockout pause** -- Rejected immediately with the lockout message, without evaluating the code's own correctness or expiry (the lockout gate is checked first).
- **A device's last-active timestamp sits at exactly platform parameter: `session-inactivity-expiry-days`** -- Treated as still active (not yet expired); expiry requires the gap to exceed the limit, not merely equal it.
- **Two failed attempts for the same identifier arrive from two different devices at nearly the same time** -- failed_attempt_count is a per-identifier counter, not per-device; both increments apply, and the lockout, once reached, blocks further submission attempts for that identifier from any device.
- **Support attempts to reach a control that would reveal a sign-in code or allow signing in as the Pro** -- No such control exists anywhere in the product for this role (SC-05); this is a structural absence, not a runtime denial to test against.

## Acceptance Criteria

**FEAT-29.SPEC-011-AC-01:** Given Talia's code was issued 9 minutes ago, when she submits it correctly, then verification succeeds.

**FEAT-29.SPEC-011-AC-02:** Given Talia's code was issued exactly platform parameter: `sign-in-code-expiry-minutes` ago, when she submits it, then it is treated as expired and the generic failure message appears.

**FEAT-29.SPEC-011-AC-03:** Given Talia has failed 4 consecutive attempts, when she submits a 5th incorrect code, then failed_attempt_count reaches platform parameter: `sign-in-lockout-threshold` and the lockout pause begins.

**FEAT-29.SPEC-011-AC-04:** Given Talia has failed 4 consecutive attempts, when she submits a 5th, correct code, then verification succeeds and failed_attempt_count resets to zero -- no lockout is triggered.

**FEAT-29.SPEC-011-AC-05:** Given Talia is inside an active lockout pause, when she attempts another submission, then it is rejected with "Too many attempts. Try again in {remaining minutes} minutes." without evaluating the code itself.

**FEAT-29.SPEC-011-AC-06:** Given Talia enters an identifier with no matching Pro Account, when she requests a code, then the request behaves identically (timing and messaging) to a matching identifier.

**FEAT-29.SPEC-011-AC-07:** Given a signed-in device's last activity was exactly platform parameter: `session-inactivity-expiry-days` ago, when the scheduled inactivity check runs, then the device is treated as still active.

**FEAT-29.SPEC-011-AC-08:** Given a signed-in device's last activity was one day more than platform parameter: `session-inactivity-expiry-days` ago, when the scheduled inactivity check runs, then the device is expired.

**FEAT-29.SPEC-011-AC-09:** Given Talia (the Pro) requests to view her own signed-in devices, when the request is made, then her full device list is shown.

**FEAT-29.SPEC-011-AC-10:** Given Platform Operator (Support) views a Pro's account, when the screen renders, then no signed-in device list, sign-in codes, or sign-out controls are shown to this role.

**FEAT-29.SPEC-011-AC-11:** Given two failed attempts for the same identifier arrive from two different devices in quick succession, when both are processed, then both increments apply to the same per-identifier failed_attempt_count.

**FEAT-29.SPEC-011-AC-12:** Given Support has an active help-request view of a Pro's account, when Support looks for any way to sign in as that Pro, then no such action exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

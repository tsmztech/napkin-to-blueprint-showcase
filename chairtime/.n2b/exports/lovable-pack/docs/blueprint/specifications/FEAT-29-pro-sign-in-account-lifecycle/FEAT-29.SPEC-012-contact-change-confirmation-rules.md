---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-29.SPEC-012
spec_name: Contact-Change Confirmation Rules
spec_slug: contact-change-confirmation-rules
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Contact-Change Confirmation Rules

## Overview

**Name:** Contact-Change Confirmation Rules
**ID:** FEAT-29.SPEC-012
**Type:** Logic/Rule
**Purpose:** Governs the dual-confirmation requirement for sign-in-contact changes -- each side proving control of its contact by entering a one-time code, identically to the Shared UI Pattern's code-entry step used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- and what a partial or abandoned change leaves in place.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The dual-confirmation requirement (both old and new contact must each enter a correct code)
- Per-side code generation, expiry, and failed-attempt lockout
- The overall confirmation window and its expiry
- What happens when only one side confirms and the other never responds

**Non-Goals:**
- Ordinary sign-in code rules -- owned by FEAT-29.SPEC-011 (Sign-In & Recovery Rules); this spec governs contact-change confirmation codes only, a distinct action from signing in, with its own code-expiry and lockout parameters
- Executing the change (updating the field, invalidating recovery state) -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing), which this spec's rules govern
- The confirmation code's message content and delivery channel -- owned by FEAT-29.SPEC-016 (Contact-Change Confirmation Notification); this spec defines when a code is valid and what counts as confirmation, not the message wording
- Recovering access when a contact method is entirely lost -- owned by FEAT-29.SPEC-002/FEAT-29.SPEC-011; a contact change requires access to both current contacts, which is a different situation from recovery
- The code-entry screen's layout and interactions -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), which surfaces this spec's rules as the Shared UI Pattern's code-entry step

## Governed Entity

**Entity:** Pending contact-change record
**Source:** Feature Dependency Map (Pro Account, sign-in identity slice)

| Field | Data Type | Description |
|-------|-----------|--------------|
| field | enum (sign_in_email \| sign_in_mobile) | Which sign-in contact is being changed |
| old_value | text | The current value at the time the change was started |
| new_value | text | The submitted new value |
| old_contact_code | opaque, not displayed | The current one-time code sent to the old contact |
| old_contact_code_expires_at | derived | Expiry timestamp for the old contact's current code |
| old_contact_failed_attempts | integer | Consecutive failed code attempts on the old-contact side since its last successful or reset state |
| old_contact_confirmed | boolean | Whether the old contact has entered a correct, unexpired code |
| new_contact_code | opaque, not displayed | The current one-time code sent to the new contact |
| new_contact_code_expires_at | derived | Expiry timestamp for the new contact's current code |
| new_contact_failed_attempts | integer | Consecutive failed code attempts on the new-contact side since its last successful or reset state |
| new_contact_confirmed | boolean | Whether the new contact has entered a correct, unexpired code |
| started_at | derived | Timestamp the change was started, for the overall confirmation-window expiry |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-003 | Account & Sign-In Settings Screen | On starting a contact change and rendering each side's code-entry step, generic failure message, lockout state, and reverted/committed state |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | During code generation, code validation, commit, and expiry processing of a pending change |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| new_value | Must resemble a valid email or mobile-number format matching the field being changed | Always | On submission (FEAT-29.SPEC-003) | "Enter a valid email address" / "Enter a valid mobile number" | Yes |
| new_value | Must differ from old_value | Always | On submission | "This is already your current {email/mobile number}" | Yes |
| old_contact_code / new_contact_code entry | Must be exactly 6 digits and match that side's current, unexpired code | Always | On submission (or auto-submit at 6 digits) on FEAT-29.SPEC-003 | "That code didn't work. Try again or send a new code." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Dual confirmation required | old_contact_confirmed, new_contact_confirmed | The change commits only when both are true; neither side's confirmation alone is sufficient | -- (no commit occurs; the pending state persists) |
| Overall window supersedes partial confirmation | started_at, old_contact_confirmed, new_contact_confirmed | If platform parameter: `contact-change-confirmation-window-hours` elapses from started_at with at most one side confirmed, the pending change is discarded regardless of which side confirmed | -- (silent discard, no error) |
| Per-side code expiry | old_contact_code_expires_at, new_contact_code_expires_at | Each side's code expires independently, platform parameter: `contact-change-code-expiry-minutes` after it was (re)sent; a code entered after its own side's expiry produces the same generic failure as a wrong code | "That code didn't work. Try again or send a new code." |
| Per-side lockout | old_contact_failed_attempts, new_contact_failed_attempts | After 5 consecutive failed attempts on one side, that side alone is locked for platform parameter: `contact-change-code-lockout-pause-minutes`; the other side's code entry is unaffected and can still be completed | "Too many attempts. Try again in {remaining minutes} minutes." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Start a contact change | The Pro (Talia) | Only for their own account's sign_in_email or sign_in_mobile | -- |
| Start a contact change | Platform Operator (Support) | Never (SC-05: support cannot change a Pro's sign-in details) | No control to start a contact change is rendered on Support's view of any screen |
| Enter a confirmation code for a side | Whoever holds the old contact, and whoever holds the new contact | Each side's code can only confirm that same side (an old-contact code cannot confirm the new-contact side, and vice versa) | A code entered against the wrong side is evaluated against that side's own current code and, not matching, produces the generic failure message like any other wrong code |
| View a pending change's status | The Pro (Talia) | Only their own account's pending change, as the code-entry steps and their confirmed/pending state on FEAT-29.SPEC-003 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| old_contact_confirmed | false | On creation of the pending change | No (only set true by a correct, unexpired code entry) |
| new_contact_confirmed | false | On creation of the pending change | No (only set true by a correct, unexpired code entry) |
| old_contact_code / new_contact_code | Randomly generated 6-digit code, independent per side | On creation of the pending change, and again whenever "Send a new code" is used for that side | No |
| old_contact_code_expires_at / new_contact_code_expires_at | started_at (or the time of the most recent "Send a new code" for that side) plus platform parameter: `contact-change-code-expiry-minutes` | Recomputed every time that side's code is (re)generated | No |
| started_at | Current time | On creation of the pending change; not reset by a "Send a new code" on either side | No |

## Business Rules

- A contact change is not committed until both the old and the new contact each enter a correct, unexpired code (FEAT-29.SPEC-010) -- there is no single-sided confirmation path under any circumstance.
- A side that fails 5 consecutive code attempts is locked for platform parameter: `contact-change-code-lockout-pause-minutes`, independent of the other side, which remains open for entry throughout.
- An unconfirmed pending change (one or both sides never enter a correct code) expires after platform parameter: `contact-change-confirmation-window-hours` from when it started; on expiry, the prior contact detail remains in place with no error shown to the Pro.
- Starting a new contact change for the same field while one is already pending discards the earlier pending change (and its codes) and starts fresh codes and confirmation for both sides (FEAT-29.SPEC-010's Business Rules).

## Edge Cases

- **The old contact enters a correct code, then the new contact never responds** -- The pending change remains pending, with the old side shown as confirmed, until platform parameter: `contact-change-confirmation-window-hours` elapses; it then discards regardless of the old side's earlier success, and the prior contact detail remains in place.
- **Both sides' codes are entered correctly at effectively the same time** -- Each side's confirmation is processed independently and idempotently; the change commits the moment both are true, regardless of which arrived first, and commits exactly once.
- **The Pro starts a change, then starts a completely different change for the other field (email pending, then starts a mobile change) before the first resolves** -- These are independent pending changes on different fields with independent codes; both may be pending simultaneously without conflict, since the "only one pending change per field" rule (FEAT-29.SPEC-010) applies per field, not across fields.
- **A correct code is entered twice from the same side** -- The second entry is a no-op; that side's confirmed state is already true and is not re-evaluated or reset by a duplicate correct entry.
- **A side enters a wrong code repeatedly** -- That side's failed_attempts increments each time and shows the generic failure message; at 5 consecutive failures that side alone locks for platform parameter: `contact-change-code-lockout-pause-minutes`, while the other side's code entry remains available and unaffected throughout.
- **"Send a new code" is requested for one side** -- A fresh code and expiry are generated for that side only, that side's failed_attempts resets to zero, and the prior code for that side becomes invalid; the other side's code, confirmed state, and the overall started_at (and therefore the overall confirmation-window deadline) are unaffected.
- **The overall confirmation window expires at the exact same moment the second side's code is entered** -- Whichever is processed first wins: if the code entry is recorded before the expiry check runs, the change commits; if the expiry check runs first, the change is discarded and a subsequently entered code is evaluated as a wrong/expired entry against a pending change that no longer exists.
- **The Pro's old contact is also the value used elsewhere as a recovery contact** -- Confirming via the old contact's code and account recovery via the same contact (FEAT-29.SPEC-002) are independent actions; entering a confirmation code does not itself constitute a sign-in or recovery attempt, and vice versa.

## Acceptance Criteria

**FEAT-29.SPEC-012-AC-01:** Given Talia's old email enters its correct code for a pending email change, when only that one side has confirmed, then the change remains pending and does not commit.

**FEAT-29.SPEC-012-AC-02:** Given both Talia's old and new email enter their correct codes for a pending change, when the second correct code is recorded, then the change commits.

**FEAT-29.SPEC-012-AC-03:** Given Talia's new email enters a wrong code, when the entry is submitted, then the generic message "That code didn't work. Try again or send a new code." appears for that side and the pending change is not discarded.

**FEAT-29.SPEC-012-AC-04:** Given Talia's new-email side has failed 5 consecutive code attempts, when she tries to enter another code on that side, then that side shows "Too many attempts. Try again in {remaining minutes} minutes." while the old-email side remains available for entry.

**FEAT-29.SPEC-012-AC-05:** Given only one side of Talia's pending change confirms and platform parameter: `contact-change-confirmation-window-hours` elapses from when the change started, when expiry is evaluated, then the pending change is discarded with no error shown.

**FEAT-29.SPEC-012-AC-06:** Given Talia's old contact enters its correct code first and then her new contact enters its correct code, when the second correct code arrives, then the change commits; confirming order does not matter.

**FEAT-29.SPEC-012-AC-07:** Given Talia has a pending email change and a separately pending mobile change at the same time, when either resolves, then the other is unaffected -- the two pending changes are independent.

**FEAT-29.SPEC-012-AC-08:** Given Talia's old contact enters its correct code twice, when the second entry is processed, then it is a no-op and does not affect the pending change's state.

**FEAT-29.SPEC-012-AC-09:** Given Support attempts to start a contact change on a Pro's behalf, when the attempt is made, then no such control exists anywhere on Support's view.

**FEAT-29.SPEC-012-AC-10:** Given the confirmation window is about to expire at the same moment the second side's code is entered, when the code entry is recorded before the expiry check runs, then the change commits.

**FEAT-29.SPEC-012-AC-11:** Given Talia taps "Send a new code" on her old-email side after two wrong attempts, when the fresh code is generated, then that side's failed-attempt count resets to zero, the prior code for that side is invalidated, and her new-email side's code and confirmed state are unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |

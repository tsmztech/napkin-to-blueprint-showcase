---
document_type: spec
spec_type: automation
spec_id: FEAT-21.SPEC-005
spec_name: Sign-In Email & Login Method Change
spec_slug: sign-in-email-login-method-change
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Sign-In Email & Login Method Change

## Overview

**Name:** Sign-In Email & Login Method Change
**ID:** FEAT-21.SPEC-005
**Type:** Automation
**Purpose:** Processes a pending sign-in email or login method change through re-verification before it takes effect, and expires or lets Nadia resend an unconfirmed change.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Starting a pending change when Nadia submits a new sign-in email on FEAT-21.SPEC-003
- Sending a re-verification link/code through transactional email delivery
- Confirming the change when Nadia completes re-verification, and committing the new sign-in email
- Expiring an unconfirmed change and reverting to the prior value
- Resending the re-verification link/code on request
- Triggering the account-critical change confirmation email (FEAT-21.SPEC-011) once the change commits

**Non-Goals:**
- Collecting the new sign-in email itself -- that is FEAT-21.SPEC-003's form; this automation begins once a syntactically valid new email is submitted
- Any alternate login method beyond email-based sign-in -- excluded per the Brief's Non-Goals: "login recovery here covers a single email-based sign-in method only"
- Signing out other devices -- a distinct capability owned by FEAT-21.SPEC-006; a sign-in email change does not itself invalidate other sessions
- Validating the new email's format -- owned by FEAT-21.SPEC-007 (Account Field Validation Rules) and enforced by FEAT-21.SPEC-003 before this automation ever starts

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New sign-in email submitted | FEAT-21.SPEC-003 (Login & Security), via "Change sign-in email" or, while a change is pending, the banner's "Use a different email" | Fires when Nadia submits a new sign-in email that passes FEAT-21.SPEC-007 validation and differs from her current sign-in email | New sign-in email, Nadia's current sign-in email, Freelancer Account reference |
| Resend requested | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia taps "Resend link" while a change is pending | The pending change's target email, Freelancer Account reference |
| Re-verification completed | FEAT-21.SPEC-003 (Login & Security), via the link/code the recipient followed | Fires when the re-verification link/code is followed or entered while the pending change has not expired | The pending change's target email and token/code |
| Cancel requested | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia taps "Cancel" while a change is pending | The pending change's target email, Freelancer Account reference |
| Pending change expiry | System (scheduled check against the pending change's expiry timestamp) | Fires when the pending change's platform parameter: `email-change-reverification-window` elapses with no completed re-verification | The pending change's target email, Freelancer Account reference |

## Processing Logic

1. On new-email submission: create a pending change record on the Freelancer Account holding the target email, a re-verification token, and an expiry timestamp set to platform parameter: `email-change-reverification-window` from now. If a pending change already existed, replace it and invalidate its prior token.
2. Send the re-verification link/code to the target email through the transactional email delivery capability (FEAT-14.SPEC-001).
3. On resend request: verify a pending change exists and has not expired; if so, issue a fresh token (invalidating the previous one) and resend to the same target email, resetting the expiry to a full platform parameter: `email-change-reverification-window` from the resend moment.
4. On re-verification completion: verify the presented token matches the pending change's current token and has not expired. If valid, commit the target email as the Freelancer Account's sign-in email, clear the pending change, and trigger the account-critical change confirmation email (FEAT-21.SPEC-011).
5. On cancel request: clear the pending change and invalidate its token; the prior sign-in email remains active throughout and requires no reversal since it was never changed.
6. On expiry: clear the pending change and invalidate its token; the prior sign-in email remains active (it was never altered mid-process).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Change started | New email submitted and accepted | Pending change record created on Freelancer Account | FEAT-21.SPEC-003 shows "Verification pending for {new email}" | FEAT-21.SPEC-003 |
| Resend succeeded | Resend requested against a still-pending, non-expired change | Token refreshed, expiry reset | FEAT-21.SPEC-003 shows "A new verification link has been sent." | FEAT-21.SPEC-003 |
| Change confirmed | Valid, non-expired token presented | Freelancer Account sign-in email updated; pending change cleared | FEAT-21.SPEC-003 shows the new sign-in email as active; account-critical change confirmation email sent | FEAT-21.SPEC-003, FEAT-21.SPEC-011 |
| Change cancelled | Nadia taps Cancel on a pending change | Pending change cleared; no account field altered | FEAT-21.SPEC-003 shows "Sign-in email change cancelled." | FEAT-21.SPEC-003 |
| Change expired | platform parameter: `email-change-reverification-window` elapses with no confirmation | Pending change cleared; no account field altered | FEAT-21.SPEC-003's pending banner clears automatically; Nadia can restart the change | FEAT-21.SPEC-003 |
| Automation failure (send or commit) | The re-verification email fails to send, or the commit step fails after a valid token is presented | No account field altered; pending change is retried per FEAT-14.SPEC-001's retry rules for the send case, or remains pending for the commit case | FEAT-21.SPEC-003 shows an inline error: "Could not complete this action. Check your connection and try again." with Retry | FEAT-21.SPEC-003 |

## Data Model

**Reads:** Freelancer Account -- current sign-in email, any existing pending change record.
**Creates:** A pending change record on the Freelancer Account -- target email, re-verification token, expiry timestamp (created on submission, replaced on resend/re-submission, cleared on confirm/cancel/expiry).
**Updates:** Freelancer Account -- sign-in email (set only on confirmed re-verification).
**Deletes:** The pending change record, on confirm, cancel, or expiry.

## Business Rules

- The prior sign-in email stays fully active and usable for sign-in until the moment a new email is confirmed -- there is no window in which Nadia is locked out (dependency map, Freelancer Account State Transition: "Active -> Pending re-verification -> Active (confirmed) or reverts to the prior value on expiry/abandonment").
- Only one pending change can exist at a time; a new submission or resend always replaces and invalidates the prior token (XBR-30's authority feature, FEAT-14, delivers only the current token's email).
- A confirmed change always triggers the account-critical change confirmation email (FEAT-21.SPEC-011) -- this is never skipped or batched with other notifications.
- This automation never signs out other devices on its own; that remains a separate, explicit action (FEAT-21.SPEC-006).

## Edge Cases

- **Nadia submits her current sign-in email as the "new" one** -- Rejected before this automation starts, per FEAT-21.SPEC-007 and FEAT-21.SPEC-003's inline check ("This is already your sign-in email."); no pending change is created.
- **The re-verification link/code is followed twice (e.g., email client pre-fetches it)** -- The first valid use commits the change and clears the pending record; the second use finds no pending change matching that token and shows "This verification link has already been used or has expired." with no further data change.
- **Nadia cancels a change at the exact moment the re-verification link is followed** -- Whichever request the system processes first wins: if the cancel commits first, the pending change is gone and the late verification attempt shows "This verification link has already been used or has expired."; if the verification commits first, the change is confirmed and the later cancel finds nothing pending and is a no-op.
- **The pending change expires at the exact moment re-verification is submitted** -- The expiry check is authoritative: a token presented at or after the expiry timestamp is treated as expired, and the presenter sees "This verification link has expired. Start the change again."
- **Concurrent trigger firing (a resend request and an independent new-email submission arrive at effectively the same time)** -- Whichever request is processed first sets the current pending state (token and target email); the second request either resends against that same state (if it targets the same email) or replaces it (if it targets a different email) -- there is no scenario where two pending changes coexist.
- **Trigger fires while a previous run is in flight (Nadia taps Resend twice rapidly)** -- FEAT-21.SPEC-003's "Resend link" button is disabled during the resend request, per that spec's Interactions; a second resend cannot start until the first completes.
- **The transactional email delivery capability is down when the re-verification email is due to send** -- The send is queued and retried per FEAT-14.SPEC-001's standing retry behavior; the pending change's expiry clock (platform parameter: `email-change-reverification-window`) still runs from the original submission time, so a prolonged outage can expire the change before delivery succeeds, in which case the expiry outcome applies and Nadia is shown the option to restart.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-003 (Login & Security) | Triggered by (inbound) | Every trigger in this automation originates from actions on that screen |
| FEAT-21.SPEC-003 (Login & Security) | Affects (outbound) | Returns pending/confirmed/expired/cancelled state to that screen |
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | New sign-in email format and "not the same as current" checks run before this automation starts |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | Triggers (outbound) | Fires once the change commits |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the re-verification link/code and its resends |

## Analytics and Success Signals

- **account_email_change_started** (has_prior_pending: yes/no) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("account_email_changed" family) so the start of the flow is observable
- **account_email_change_confirmed** (time_to_confirm_bucket) -- N/A -- no connected success-metrics.md metric; retained so the completion of a security-sensitive change is observable
- **account_email_change_expired** () -- N/A -- no connected success-metrics.md metric; retained so abandoned changes are observable rather than silently disappearing
- **account_email_change_cancelled** () -- N/A -- no connected success-metrics.md metric; retained for the same reason

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given Nadia submits a new, valid sign-in email on FEAT-21.SPEC-003, when this automation processes the submission, then a pending change is created, a re-verification link is sent to the new email, and FEAT-21.SPEC-003 shows "Verification pending for {new email}".

**FEAT-21.SPEC-005-AC-02:** Given Nadia has a pending change, when she taps "Resend link", then a fresh token is issued, a new link is sent, and the pending change's expiry resets to a full platform parameter: `email-change-reverification-window`.

**FEAT-21.SPEC-005-AC-03:** Given Nadia follows a valid, non-expired re-verification link, when this automation processes it, then her sign-in email is updated to the target email, the pending change clears, and FEAT-21.SPEC-011 sends the account-critical change confirmation email.

**FEAT-21.SPEC-005-AC-04:** Given Nadia has a pending change, when she taps "Cancel", then the pending change clears and her prior sign-in email remains active with no confirmation email sent.

**FEAT-21.SPEC-005-AC-05:** Given Nadia's pending change reaches platform parameter: `email-change-reverification-window` with no completed re-verification, when this automation's expiry check runs, then the pending change clears and her prior sign-in email remains active.

**FEAT-21.SPEC-005-AC-06:** Given Nadia has an already-confirmed change, when the same re-verification link is followed a second time, then she sees "This verification link has already been used or has expired." and no further data changes.

**FEAT-21.SPEC-005-AC-07:** Given Nadia taps "Use a different email" on FEAT-21.SPEC-003's pending banner and submits a different valid email while one change is already pending, when this automation processes the new submission, then the prior pending change and its token are invalidated and replaced by the new target email.

**FEAT-21.SPEC-005-AC-08:** Given a re-verification token is presented at or after its expiry timestamp, when this automation checks it, then the presenter sees "This verification link has expired. Start the change again." and no account field changes.

**FEAT-21.SPEC-005-AC-09:** Given the transactional email delivery capability is unavailable when a re-verification email is due to send, when the delivery capability recovers before the pending change's expiry, then the queued email is delivered and the change remains eligible for confirmation.

**FEAT-21.SPEC-005-AC-10:** Given the transactional email delivery capability remains unavailable past the pending change's expiry, when the expiry check runs, then the change expires per FEAT-21.SPEC-005-AC-05, regardless of the undelivered email.

**FEAT-21.SPEC-005-AC-11:** Given Nadia taps "Resend link" while a prior resend request for the same change is still processing, when the second tap occurs, then it is ignored because FEAT-21.SPEC-003 disables the control during the in-flight request.

**FEAT-21.SPEC-005-AC-12:** Given this automation's commit step fails after a valid token is presented, when the failure occurs, then no sign-in email change is applied, the pending change remains active, and FEAT-21.SPEC-003 shows "Could not complete this action. Check your connection and try again." with Retry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (submit, resend, confirm, cancel, expiry) | 5 |
| Outcome Paths | 6 (started, resend succeeded, confirmed, cancelled, expired, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

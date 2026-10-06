---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-010
spec_name: Contact-Detail Change Processing
spec_slug: contact-detail-change-processing
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Contact-Detail Change Processing

## Overview

**Name:** Contact-Detail Change Processing
**ID:** FEAT-29.SPEC-010
**Type:** Automation
**Purpose:** Carries a sign-in email or mobile-number change through code-entry confirmation on both the old and the new contact -- the same code-entry pattern used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- before committing it.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Starting a pending contact-detail change when the Pro submits a new email or mobile number
- Generating and delivering an independent one-time code to both the old and new contact
- Validating a submitted code against the correct side and handling wrong/expired entries and per-side lockout
- Handling a "send a new code" request for either side
- Committing the change only once both sides confirm with a correct code
- Invalidating outstanding recovery state and, when the mobile number changes, client access links tied to the old identity

**Non-Goals:**
- Defining the confirmation codes' expiry, lockout, and overall confirmation-window rules -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this automation enforces those rules, it does not define them
- Starting the change from the Settings screen, or rendering the code-entry steps -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen); this automation begins where that screen's submission ends
- Delivering the code content itself -- owned by FEAT-29.SPEC-016 (Contact-Change Confirmation Notification); this automation only triggers that delivery
- Changing any profile field other than sign_in_email or sign_in_mobile -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: this feature updates only sign-in contacts, devices, and closure status; other profile fields are FEAT-27's

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro submits a new sign-in email | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Edit" on the email row and submits a new value | New email value, current sign_in_email |
| Pro submits a new sign-in mobile number | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Edit" on the mobile row and submits a new value | New mobile value, current sign_in_mobile |
| Confirmation code submitted | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | The Pro enters a 6-digit code on either the old-contact or new-contact code-entry step | Which side, the code entered, the pending change reference |
| "Send a new code" requested | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | The Pro taps "Send a new code" on either side's code-entry step | Which side, the pending change reference |

## Processing Logic

1. Receive the new contact value and identify which field (sign_in_email or sign_in_mobile) is changing.
2. Create a pending contact-change record referencing the current value (old contact) and the submitted value (new contact), with both sides unconfirmed.
3. Generate an independent 6-digit code and expiry for each side per FEAT-29.SPEC-012, and trigger FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) to deliver each side's code to its own contact.
4. On a code submission for a side, validate it against that side's current, unexpired code per FEAT-29.SPEC-012:
   - Correct and unexpired: mark that side confirmed.
   - Incorrect or expired: increment that side's failed-attempt count; FEAT-29.SPEC-003 shows the generic failure message for that side; at 5 consecutive failures for that side, that side locks per FEAT-29.SPEC-012's lockout rule while the other side remains open.
5. On a "send a new code" request for a side: generate a fresh code and expiry for that side only, invalidate the prior code for that side, reset that side's failed-attempt count, and trigger FEAT-29.SPEC-016 to deliver the fresh code -- without affecting the other side or the overall confirmation window's start time.
6. When both sides have confirmed: commit the change -- update the Pro Account's sign_in_email or sign_in_mobile to the new value, invalidate any outstanding recovery code tied to the old identifier, and, if the mobile number changed, invalidate every outstanding Access Link tied to the old mobile number that a client might reuse to reach this Pro's account settings context (this automation invalidates only this feature's own recovery state; client-facing Access Link invalidation for the changed mobile number is a Client entity concern outside this feature's write scope and is not performed here).
7. Trigger FEAT-29.SPEC-016 to send the final confirmation of the committed change to both contacts.
8. If the overall confirmation window elapses per FEAT-29.SPEC-012 before both sides confirm, discard the pending change entirely, leaving the prior contact detail in place.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Change started | Pro submits a new email or mobile number | Pending contact-change record created; a code generated for each side | FEAT-29.SPEC-003 shows both code-entry steps, each with "We sent a code to {masked identifier}" | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Code entry failed | Pro submits a wrong or expired code for one side | That side's failed-attempt count incremented | FEAT-29.SPEC-003 shows "That code didn't work. Try again or send a new code." under that side's step; that side's code input clears | FEAT-29.SPEC-003 |
| Side locked | A side reaches 5 consecutive failed code attempts | That side is temporarily locked | FEAT-29.SPEC-003 shows "Too many attempts. Try again in {remaining minutes} minutes." on that side; the other side remains active | FEAT-29.SPEC-003 |
| New code requested | Pro taps "Send a new code" on one side | Fresh code generated for that side; prior code for that side invalidated; that side's failed-attempt count reset | FEAT-29.SPEC-003 clears that side's code input with the confirmation "New code sent"; FEAT-29.SPEC-016 delivers the fresh code | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Change committed | Both sides enter a correct, unexpired code | sign_in_email or sign_in_mobile updated; outstanding recovery code invalidated | FEAT-29.SPEC-003 shows the new value once loaded; FEAT-29.SPEC-016 sends the committed-change confirmation to both contacts | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Change expired | Neither side (or only one side) completes confirmation within FEAT-29.SPEC-012's overall confirmation window | Pending change discarded; prior value unchanged | FEAT-29.SPEC-003 row reverts to the prior value on next load | FEAT-29.SPEC-003 |
| Change failed (processing error) | An error occurs while committing | No update to sign_in_email/sign_in_mobile | FEAT-29.SPEC-003 shows "Couldn't update your contact details. Try again." | FEAT-29.SPEC-003 |

## Data Model

**Reads:** Pro Account.sign_in_email, sign_in_mobile (current values).
**Creates:** A pending contact-change record (old value, new value, and per side: current code, code expiry, failed-attempt count, confirmed flag).
**Updates:** Pro Account.sign_in_email or sign_in_mobile (on commit only); the pending record's per-side code, expiry, failed-attempt count, and confirmed flag throughout processing.
**Deletes:** The pending contact-change record (on commit or expiry -- it never persists once resolved).

## Business Rules

- A change is never committed until both the old and the new contact each enter a correct, unexpired code (FEAT-29.SPEC-012) -- there is no single-sided confirmation path.
- A side that fails 5 consecutive code attempts locks for platform parameter: `contact-change-code-lockout-pause-minutes` independent of the other side, which remains open for entry throughout.
- Changing a sign-in contact invalidates outstanding recovery state for the old identifier immediately on commit -- a code requested against the old identifier before the change can no longer be used to sign in afterward.
- Only one pending change per field may exist at a time -- submitting a new value while a change for the same field is already pending replaces the earlier pending change (and its codes) with a new one, restarting confirmation for both sides.
- This automation writes only sign_in_email and sign_in_mobile on the Pro Account -- it never touches any other Pro Account field or any Client field directly.

## Edge Cases

- **Pro submits a new email while a previous email change is still pending confirmation** -- The earlier pending change and its codes are discarded (no confirmation collected for it counts toward the new one) and a fresh pending change with fresh codes starts for the newly submitted value, per FEAT-29.SPEC-012.
- **The old contact enters a correct code but the new contact never enters any code** -- The change remains pending until FEAT-29.SPEC-012's overall confirmation window elapses, then is discarded; the prior contact detail stays in place, and the Pro is never left without a valid sign-in contact.
- **Concurrent trigger firing -- both sides' correct codes arrive at effectively the same time** -- Each code submission is processed independently and idempotently; when the second of the two is recorded, the commit step runs exactly once (the commit logic checks that both sides are now confirmed before proceeding, so a race between the two arrivals cannot produce two commits or none).
- **Trigger fires while a previous run is in flight -- the Pro submits a second contact change for the same field while the automation is mid-commit for the first** -- The mid-commit run is allowed to finish; only once it resolves (commit or expiry) does the newly submitted change become the active pending record, per the "only one pending change per field" rule.
- **The mobile number changes and a client holds an Access Link tied to the old number** -- Any client-facing Access Link scoping is governed and invalidated by FEAT-06's own phone-number-match rules (dependency map: Client Contention -- "a Pro phone-number change invalidates access links and requires fresh texting consent"); this automation's own recovery-state invalidation is limited to this feature's sign-in recovery codes, not client Access Links, which is why that boundary is stated explicitly in the Processing Logic.
- **A code arrives for a side that has already locked from 5 failed attempts** -- The submission is not evaluated against the code while that side is locked; FEAT-29.SPEC-003 continues to show the lockout message until platform parameter: `contact-change-code-lockout-pause-minutes` elapses, after which normal code entry resumes for that side.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Triggered by (inbound) | Submitting a new email or mobile number, a code entry, and a "send a new code" request all trigger this automation |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Affects (outbound) | Shows each side's code-entry, error, locked, committed, or reverted state |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs code expiry, per-side lockout, and the overall confirmation window |
| FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) | Affects (outbound) | Sends each side's code and the final committed-change confirmation |

## Analytics and Success Signals

- **contact_details_changed** (field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **contact_change_code_failed** (field: email / mobile, side: old / new) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so wrong/expired code attempts remain observable
- **contact_change_expired** (field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so incomplete contact changes remain observable rather than silently dropped

## Acceptance Criteria

**FEAT-29.SPEC-010-AC-01:** Given Talia submits a new sign-in email on FEAT-29.SPEC-003, when the submission processes, then a pending contact-change record is created and a one-time code is sent to both her old and new email addresses.

**FEAT-29.SPEC-010-AC-02:** Given both Talia's old and new email enter their correct codes, when the second correct code is recorded, then Pro Account.sign_in_email is updated to the new value and both contacts receive the committed-change confirmation.

**FEAT-29.SPEC-010-AC-03:** Given only Talia's old email enters its correct code and her new email never responds, when FEAT-29.SPEC-012's overall confirmation window expires, then the pending change is discarded and her sign-in email remains the old value.

**FEAT-29.SPEC-010-AC-04:** Given Talia's new-email side submits a wrong code, when the entry is processed, then FEAT-29.SPEC-003 shows "That code didn't work. Try again or send a new code." for that side and her sign-in email remains the old value.

**FEAT-29.SPEC-010-AC-05:** Given Talia's mobile-number change commits successfully, when the change is committed, then any outstanding sign-in recovery code tied to the old mobile number can no longer be used to sign in.

**FEAT-29.SPEC-010-AC-06:** Given Talia has a pending email change awaiting confirmation, when she submits another new email before it resolves, then the earlier pending change and its codes are discarded and a fresh one starts for the newly submitted value.

**FEAT-29.SPEC-010-AC-07:** Given both of Talia's sides submit their correct codes at effectively the same time, when both are processed, then the change commits exactly once.

**FEAT-29.SPEC-010-AC-08:** Given a processing error occurs while committing Talia's contact change, when the failure is detected, then FEAT-29.SPEC-003 shows "Couldn't update your contact details. Try again." and no field is updated.

**FEAT-29.SPEC-010-AC-09:** Given Talia's mobile number changes, when a client's Access Link tied to the old number is evaluated, then its invalidation is governed by FEAT-06's own phone-number-match rules, not by this automation directly.

**FEAT-29.SPEC-010-AC-10:** Given Talia submits a second contact change for the same field while an earlier one is mid-commit, when the mid-commit run resolves, then only then does the newly submitted change become the active pending record.

**FEAT-29.SPEC-010-AC-11:** Given a change this automation writes, when the write is inspected, then it touches only Pro Account.sign_in_email or sign_in_mobile, never any other field.

**FEAT-29.SPEC-010-AC-12:** Given Talia's old-email side has locked after 5 consecutive wrong codes, when she taps "Send a new code" on that side, then a fresh code is generated, that side's failed-attempt count resets, and her new-email side's code and confirmed state are unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

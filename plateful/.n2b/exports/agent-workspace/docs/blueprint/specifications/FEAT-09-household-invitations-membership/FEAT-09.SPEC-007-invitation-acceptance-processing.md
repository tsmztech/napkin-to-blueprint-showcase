---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-007
spec_name: Invitation Acceptance Processing
spec_slug: invitation-acceptance-processing
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Invitation Acceptance Processing

## Overview

**Name:** Invitation Acceptance Processing
**ID:** FEAT-09.SPEC-007
**Type:** Automation
**Purpose:** On acceptance, creates the new Member Profile with Other Adult Member access and routes the invitee into first-use onboarding.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the invitation is still Sent at the moment of processing (the accept/revoke/expiry race)
- Creating the new Member Profile record with Other Adult Member access, using the sign-in details captured on FEAT-09.SPEC-002
- Transitioning the Invitation's status to Accepted
- Triggering FEAT-09.SPEC-012 (Invitation Accepted Confirmation) to the organiser
- Routing the newly created member into FEAT-15 (Member Onboarding)

**Non-Goals:**
- Collecting the invitee's display name, sign-in email, and password -- captured by FEAT-09.SPEC-002 (Invitation Acceptance) and passed into this automation, not collected here
- The onboarding experience itself (first-use landing on the plan and list) -- owned by FEAT-15 (Member Onboarding); this automation only performs the hand-off
- Restoring a re-invited former member's prior profile, dietary rules, or history -- excluded per XBR-18; every acceptance creates a genuinely new Member Profile, even for someone who was previously a member of this same household

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitee taps "Accept & Join" | FEAT-09.SPEC-002 (Invitation Acceptance) | Fires on successful form validation, before the invitation's current validity is re-confirmed | Invitation reference, invitee's display name, sign-in email, password |

## Processing Logic

1. Receive the invitation reference and the invitee's entered display name, sign-in email, and password from FEAT-09.SPEC-002.
2. Re-check the invitation's current status per FEAT-09.SPEC-010's race resolution: proceed only if it is still Sent.
3. If no longer Sent (already Accepted, Revoked, or Expired since the invitee loaded the screen), stop processing and return the current status and reason to FEAT-09.SPEC-002 -- no Member Profile is created.
4. If still Sent, create a new Member Profile: display_name from the invitee's input, member_type set to Other Adult Member, sign_in set to the invitee's entered email and protected sign-in, status set to Active, notification_preferences set to the household's stated defaults (plan-ready on, nightly nudge on).
5. Set the Invitation's status to Accepted.
6. Trigger FEAT-09.SPEC-012 (Invitation Accepted Confirmation) to the household's current organiser.
7. Route the newly created member into FEAT-15 (Member Onboarding), landing in context on the household's current Weekly Plan and Grocery List.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Acceptance succeeds | Invitation is still Sent at processing time | New Member Profile created (Active, Other Adult Member); Invitation.status set to Accepted | Invitee is routed into FEAT-15 first-use onboarding; organiser receives FEAT-09.SPEC-012 | FEAT-09.SPEC-002, FEAT-09.SPEC-001, FEAT-15 (FEAT-15.SPEC-002), FEAT-09.SPEC-012 |
| Acceptance rejected -- race lost | Invitation is no longer Sent (Accepted, Revoked, or Expired) when processing runs | No data changes | FEAT-09.SPEC-002 switches to the invalid-invitation body with the specific current reason; no Member Profile is created | FEAT-09.SPEC-002 |
| Automation failure | Processing error after the race check passes but before the Member Profile is durably created | No partial Member Profile persists; the Invitation remains Sent | FEAT-09.SPEC-002 shows an inline error and offers Retry; the invitee's entered sign-in details are preserved for the retry | FEAT-09.SPEC-002 |

## Data Model

**Reads:** Invitation -- status, contact_detail, sent_by; Household -- the organiser reference, to address FEAT-09.SPEC-012.
**Creates:** Member Profile -- display_name, member_type (Other Adult Member), sign_in, status (Active), notification_preferences (household defaults).
**Updates:** Invitation -- status (Sent to Accepted).
**Deletes:** None.

## Business Rules

- An accepted invitation creates a Member Profile with Other Adult Member access and triggers first-use onboarding exactly once per accepted invitation (XBR-18); this automation never runs its creation step more than once for the same invitation.
- A re-invited former member (someone who previously left or was removed) onboards again as a genuinely new Member Profile rather than being restored to old data (XBR-18) -- this automation performs no lookup against any prior profile for the same person.
- The race between acceptance and a concurrent revocation or expiry resolves reject-with-refresh: whichever change lands first wins (FEAT-09.SPEC-010).
- Notification preferences on the new Member Profile default to the household's stated defaults; the new member can change them afterward through FEAT-07/FEAT-13's own settings, not through this automation.

## Edge Cases

- **Invitation is revoked between the invitee's tap and this automation's re-check** -- Reject-with-refresh: the revoke wins if recorded first, this automation's race check (Processing Logic, step 2-3) finds the invitation Revoked and stops, and no Member Profile is created.
- **Invitation expires (FEAT-09.SPEC-006) between the invitee's tap and this automation's re-check** -- Same resolution: whichever change lands first wins; if expiry is recorded first, this automation stops with no Member Profile created.
- **Two people somehow attempt to accept the same invitation link at effectively the same time** -- Only the first acceptance to be recorded succeeds and creates the Member Profile and transitions the Invitation to Accepted; the second is processed against the now-Accepted invitation and is rejected via the race-lost outcome, exactly as an ordinary "already accepted" case.
- **Concurrent trigger firing (this automation and FEAT-09.SPEC-006's expiry fire on the same invitation at effectively the same time)** -- Whichever transition is recorded first wins per FEAT-09.SPEC-010; the loser's spec (this automation, or FEAT-09.SPEC-006) takes its respective no-action/rejected path with no partial or conflicting Invitation state.
- **Trigger fires while a previous run is still in flight for the same invitation** -- A second acceptance attempt for the same invitation cannot start meaningfully while the first is in flight, since FEAT-09.SPEC-002's Accept & Join button is disabled during processing on that screen; a second attempt from a different session for the same invitation processes against whatever status the first run leaves behind (Accepted, if the first run succeeded), taking the race-lost path.
- **Member Profile creation succeeds but the notification trigger (FEAT-09.SPEC-012) fails** -- The acceptance itself is not rolled back; the organiser simply does not receive the confirmation and instead sees the new member on FEAT-09.SPEC-001 (Household Invitations Manager) directly, since a missed confirmation must never undo a completed membership change.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invitation Acceptance) | Triggered by (inbound) | Accept & Join fires this automation |
| FEAT-09.SPEC-002 (Invitation Acceptance) | Affects (outbound) | Returns success or the specific invalidity reason |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | The accept/revoke/expiry race resolution |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Affects (outbound) | The accepted invitation and new member become visible there |
| FEAT-09.SPEC-012 (Invitation Accepted Confirmation) | Triggers (outbound) | Notifies the organiser of the successful acceptance |
| FEAT-15 (Member Onboarding) | Affects (outbound) | Routes the new member into first-use onboarding |
| FEAT-09.SPEC-006 (Invitation Expiry) | References (inbound) | Shares the same race-resolution outcome space on the Invitation entity |

## Analytics and Success Signals

- **invitation_accepted** (time from sent to accepted, in days) -- supports success-metrics.md: "Household Member Participation" (the acceptance is the moment a household gains the other adult member the metric requires)
- **invitation_accept_rejected** (reason: revoked / expired / already_accepted) -- N/A -- no Stage 2 metric measures rejected acceptance attempts; retained as a funnel-diagnostic signal
- **member_profile_created** (member_type: other_adult) -- supports success-metrics.md: "Household Member Participation" (creation of the joined member's profile is the durable record the metric's "joined" condition is evaluated against)

## Acceptance Criteria

**FEAT-09.SPEC-007-AC-01:** Given Sam submits valid sign-in details on FEAT-09.SPEC-002 for a still-Sent invitation, when this automation processes the acceptance, then a new Member Profile is created for Sam with Other Adult Member access and Active status.

**FEAT-09.SPEC-007-AC-02:** Given the automation successfully creates Sam's Member Profile, when creation completes, then the Invitation's status is set to Accepted and Sam is routed into FEAT-15 (Member Onboarding) landing on the current Weekly Plan.

**FEAT-09.SPEC-007-AC-03:** Given the automation successfully processes Sam's acceptance, when processing completes, then Maya (the organiser) receives FEAT-09.SPEC-012 (Invitation Accepted Confirmation).

**FEAT-09.SPEC-007-AC-04:** Given the invitation was revoked by Maya moments before this automation's re-check, when the automation processes Sam's submission, then it stops with no Member Profile created and FEAT-09.SPEC-002 shows the revoked-invitation message.

**FEAT-09.SPEC-007-AC-05:** Given the invitation expired moments before this automation's re-check, when the automation processes the submission, then it stops with no Member Profile created and FEAT-09.SPEC-002 shows the expired-invitation message.

**FEAT-09.SPEC-007-AC-06:** Given Sam previously left this same household and is being re-invited, when he accepts the new invitation, then a genuinely new Member Profile is created for him with no data restored from his prior profile.

**FEAT-09.SPEC-007-AC-07:** Given two acceptance attempts for the same invitation are processed at effectively the same time, when the first is recorded, then the second is rejected as already-accepted with no second Member Profile created.

**FEAT-09.SPEC-007-AC-08:** Given the Member Profile creation succeeds but the confirmation trigger to Maya fails, when this is detected, then Sam's membership remains in effect and Maya instead sees the new member directly on FEAT-09.SPEC-001.

**FEAT-09.SPEC-007-AC-09:** Given a processing error occurs after the race check passes but before the Member Profile is durably created, when the failure occurs, then no partial Member Profile exists, the Invitation remains Sent, and FEAT-09.SPEC-002 shows an inline error with Sam's entered details preserved for retry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, rejected -- race lost, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

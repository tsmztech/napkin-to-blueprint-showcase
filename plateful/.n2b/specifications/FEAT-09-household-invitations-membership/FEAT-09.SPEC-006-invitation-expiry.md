---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-006
spec_name: Invitation Expiry
spec_slug: invitation-expiry
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Invitation Expiry

## Overview

**Name:** Invitation Expiry
**ID:** FEAT-09.SPEC-006
**Type:** Automation
**Purpose:** A pending invitation automatically expires 14 days after it was sent, so a stale invitation link stops working and the organiser sees its true state.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Transitioning a Sent invitation to Expired once 14 days have passed since it was sent
- Making the expired invitation's link resolve to the "no longer valid" experience on FEAT-09.SPEC-002
- Making the Expired status and a Resend action available on FEAT-09.SPEC-001

**Non-Goals:**
- Resending an expired invitation -- a distinct, user-initiated action on FEAT-09.SPEC-001 (Household Invitations Manager), owned by that screen, not this automation
- Notifying the organiser when an invitation expires -- feature-overview.md's Communications field and Side-Effect Inventory name a confirmation on acceptance and on a member leaving, but no notification for expiry; the organiser sees the Expired status passively on FEAT-09.SPEC-001, per that screen's Empty/List states
- Deleting the expired invitation record -- invitations are never hard-deleted, retained with their terminal status as household history (assumptions-constraints.md ASMP-24)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| 14 days elapsed since an invitation was sent | system (schedule-based) | Fires once per Invitation, 14 days after its sent date, only if its status is still Sent at that moment | Invitation contact_detail, status, sent_by, sent date |

## Processing Logic

1. On each scheduled run, identify every Invitation whose status is Sent and whose sent date is 14 or more days in the past.
2. For each identified invitation, re-confirm its status is still Sent at the moment of processing (it may have been accepted or revoked since the schedule last ran).
3. If still Sent, set the invitation's status to Expired.
4. If no longer Sent (already Accepted or Revoked), skip it -- no action, no error.
5. No notification is sent for any transition performed by this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invitation expired | Invitation was Sent and 14+ days have elapsed since it was sent | Invitation.status set to Expired | None immediately; the organiser sees the Expired badge and a "Resend" action the next time she opens FEAT-09.SPEC-001; an invitee opening the link afterward sees "This invitation is no longer valid. It expired after 14 days." on FEAT-09.SPEC-002 | FEAT-09.SPEC-001, FEAT-09.SPEC-002 |
| No action (already resolved) | Invitation reached 14 days but was already Accepted or Revoked before this run | None | None -- the invitation already reflects its resolved status | -- |
| Automation failure | Processing error during a scheduled run | None -- no partial expiry is ever applied | None immediately; the affected invitations remain Sent and are picked up on the next scheduled run | FEAT-09.SPEC-001 (continues to show the invitation as Sent, including one that is functionally overdue, until the next successful run) |

## Data Model

**Reads:** Invitation -- status and sent date, across all households, to find every Sent invitation past 14 days.
**Creates:** None.
**Updates:** Invitation -- status (Sent to Expired).
**Deletes:** None.

## Business Rules

- The 14-day expiry window is fixed at 14 days from the invitation's sent date (feature-overview.md, Validation & Limits) -- this is a feature-level product decision stated directly in the product definition, not a platform-wide policy value.
- Expiry never reverses an Accepted or Revoked invitation -- it applies only to invitations still in the Sent status at the moment of processing (FEAT-09.SPEC-010 governs this precedence).
- An Expired invitation can be resent as a fresh Sent invitation (a new Invitation record via FEAT-09.SPEC-001), never revived in place.

## Edge Cases

- **Invitation is accepted at almost exactly the same moment this automation would expire it** -- Reject-with-refresh, per the dependency map's Contention note for Invitation: whichever change is recorded first wins. If the acceptance is recorded first, this automation's re-confirmation step (Processing Logic, step 2) finds the invitation no longer Sent and skips it.
- **Invitation is revoked at almost exactly the same moment this automation would expire it** -- Same resolution: whichever change is recorded first wins; if the revoke is recorded first, this automation skips the invitation.
- **Scheduled run is delayed or missed** -- Invitations remain Sent past their 14-day window until the next successful run, which processes them as overdue at that point; no invitation expires "early," and no expiry is silently lost.
- **Concurrent trigger firing (two scheduled runs overlap)** -- Each invitation's expiry transition is idempotent: setting an already-Expired invitation to Expired again has no observable effect, so an overlapping run produces no double transition or duplicate side effect (there is none to duplicate, since no notification is sent).
- **Trigger fires while a previous run is still in flight** -- The next scheduled run is skipped if the prior run has not completed, preventing two runs from re-confirming and transitioning the same invitation simultaneously; a skipped run's candidates are picked up whole on the following run.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Household Invitations Manager) | Affects (outbound) | Expired status and Resend action shown there |
| FEAT-09.SPEC-002 (Invitation Acceptance) | Affects (outbound) | An expired invitation's link resolves to the "no longer valid" experience there |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Defines the 14-day window and the accept/revoke/expiry race resolution this automation follows |

## Analytics and Success Signals

- **invitation_expired** (days_outstanding: 14) -- N/A -- no Stage 2 metric measures invitation expiry; "Household Member Participation" measures a member's joining and use, not the invitation funnel's attrition, so this signal is retained only as a funnel-diagnostic event.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given an invitation was sent 14 days ago and is still Sent, when this automation runs, then the invitation's status becomes Expired.

**FEAT-09.SPEC-006-AC-02:** Given an invitation was sent 13 days ago and is still Sent, when this automation runs, then the invitation remains Sent.

**FEAT-09.SPEC-006-AC-03:** Given an invitation reached 14 days but was accepted moments before this automation runs, when the automation processes it, then it finds the invitation already Accepted and takes no action.

**FEAT-09.SPEC-006-AC-04:** Given an invitation reached 14 days but was revoked moments before this automation runs, when the automation processes it, then it finds the invitation already Revoked and takes no action.

**FEAT-09.SPEC-006-AC-05:** Given Maya opens FEAT-09.SPEC-001 after an invitation has expired, when the list loads, then that invitation shows the Expired badge and a Resend action.

**FEAT-09.SPEC-006-AC-06:** Given an invitee opens the link to an invitation that has since expired, when FEAT-09.SPEC-002 loads, then they see "This invitation is no longer valid. It expired after 14 days."

**FEAT-09.SPEC-006-AC-07:** Given two scheduled runs overlap for the same invitation, when both attempt the expiry transition, then the invitation ends in Expired status exactly once with no duplicate side effect.

**FEAT-09.SPEC-006-AC-08:** Given a scheduled run is missed and an invitation is now 20 days past its sent date, when the next successful run occurs, then that invitation is still correctly transitioned to Expired.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (expired, no action, automation failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

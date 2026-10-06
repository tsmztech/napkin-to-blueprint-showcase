---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-009
spec_name: Organiser Hand-Over Processing
spec_slug: organiser-hand-over-processing
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Organiser Hand-Over Processing

## Overview

**Name:** Organiser Hand-Over Processing
**ID:** FEAT-09.SPEC-009
**Type:** Automation
**Purpose:** On hand-over acceptance, transfers the Household's organiser field and swaps both members' roles atomically.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the hand-over request is still outstanding and addressed to the accepting member at the moment of processing
- Transferring Household.organiser to the accepting member
- Setting the accepting member's member_type to Organiser and the former organiser's member_type to Other Adult Member, as one atomic update
- Clearing the pending hand-over request

**Non-Goals:**
- The recipient's accept/decline decision itself -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance), which triggers this automation only on accept
- Initiating a hand-over or selecting a recipient -- owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation)
- Notifying either party of the completed transfer beyond what each screen shows inline (FEAT-09.SPEC-004's own success confirmation to the new organiser) -- feature-overview.md's Communications field names a request notification (FEAT-09.SPEC-014) but no separate "hand-over completed" notification; this automation triggers none

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Recipient taps "Accept" | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Fires on the recipient's accept action, before the request's current standing is re-confirmed | Household reference, accepting member's Member Profile reference, former organiser's Member Profile reference |

## Processing Logic

1. Receive the household reference and the accepting member's identity from FEAT-09.SPEC-004.
2. Re-check per FEAT-09.SPEC-010 that a hand-over request is still outstanding for this household and is still addressed to this accepting member.
3. If no longer outstanding (cancelled by the organiser, or the accepting member is no longer eligible), stop processing and return the withdrawn-request status to FEAT-09.SPEC-004 -- no data changes.
4. If still outstanding, set Household.organiser to the accepting member's Member Profile reference.
5. Set the accepting member's member_type to Organiser.
6. Set the former organiser's member_type to Other Adult Member.
7. Clear the household's pending hand-over request.
8. All of steps 4-7 apply together as one atomic update -- no household state exists with the organiser field pointing to one member while member_type fields have not yet caught up, or vice versa.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Hand-over succeeds | Request is still outstanding and addressed to the accepting member at processing time | Household.organiser transferred; both members' member_type swapped; pending request cleared | New organiser sees "You're now the organiser." and is routed to FEAT-01.SPEC-010 with organiser-level access; former organiser's next FEAT-01 action is now gated as Other Adult Member | FEAT-09.SPEC-004, FEAT-01.SPEC-010, FEAT-09.SPEC-003, FEAT-01.SPEC-016 |
| Hand-over rejected -- request withdrawn | Request was cancelled by the organiser, or the accepting member is no longer an eligible Active adult member | No data changes | FEAT-09.SPEC-004 shows the Withdrawn state | FEAT-09.SPEC-004 |
| Automation failure | Processing error after the outstanding-request check passes but before the atomic update completes | No partial change persists -- the organiser field and both member_type fields remain exactly as they were before this run | FEAT-09.SPEC-004 shows an inline error; the request remains outstanding and can be retried | FEAT-09.SPEC-004 |

## Data Model

**Reads:** Household -- organiser, pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003); Member Profile -- the accepting member's and the former organiser's current member_type and status.
**Creates:** None.
**Updates:** Household -- organiser (transferred to the accepting member), pending_organiser_handover (cleared); Member Profile -- member_type on both the accepting member (to Organiser) and the former organiser (to Other Adult Member).
**Deletes:** None.

## Business Rules

- The household always has exactly one organiser (XBR-15): this automation's atomic update (Processing Logic, step 8) guarantees no moment exists with zero or two organisers.
- Only an outstanding request addressed to the accepting member can be processed; a stale or already-resolved request is rejected, never silently re-applied.
- The former organiser retains full household access as an Other Adult Member immediately after the swap -- nothing about her access is revoked beyond the organiser-only entitlements themselves (feature-overview.md, Primary Flows & Alternates).
- This automation never runs against a household with no outstanding hand-over request; FEAT-09.SPEC-004 has no entry point without one.

## Edge Cases

- **Organiser cancels the request at almost exactly the same moment the recipient accepts** -- Reject-with-refresh: whichever change is recorded first wins. If the cancel is recorded first, this automation's re-check (Processing Logic, step 2-3) finds no outstanding request and stops with the Withdrawn outcome; if the accept is recorded first, the transfer proceeds and the near-simultaneous cancel attempt (FEAT-09.SPEC-003's own edge case) is itself rejected against the now-completed transfer.
- **Accepting member's status changed to Left between the request being sent and the accept being processed (defensive case, since FEAT-09.SPEC-003 already cancels the request when the recipient leaves)** -- The eligibility re-check finds the recipient no longer an Active adult member and rejects with the Withdrawn outcome, as a second line of defense behind FEAT-09.SPEC-003's own cancellation.
- **This automation runs twice for the same accepted request (defensive case, e.g., a double network retry)** -- The second run finds no outstanding request (already cleared by the first run) and takes the Withdrawn path with no further effect; the organiser field and member_type fields are not swapped a second time.
- **Former organiser has an organiser-only screen open at the moment the swap completes** -- Her next action against an organiser-only control is rejected per FEAT-01.SPEC-016's own edge case ("Only the organiser can change this"), since authorization is evaluated at action time, not at screen-load time; this automation itself performs no live push to her open screen.
- **Trigger fires while a previous run is still in flight for the same household** -- FEAT-09.SPEC-004's Accept button is disabled during processing, preventing a second trigger from the same session; a second trigger for the same household (e.g., a retried request) processes against the household's current state and, if the request is already cleared, takes the Withdrawn path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Triggered by (inbound) | Accept fires this automation |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Affects (outbound) | Returns success or the withdrawn-request status |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the request is still outstanding and addressed to the accepting member |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Affects (outbound) | The initiating organiser's screen no longer reflects an outstanding request afterward |
| FEAT-01.SPEC-010 (Household Settings Hub) | Affects (outbound) | Reflects the new organiser's access immediately after the swap |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Governs the former organiser's now-restricted access to organiser-only actions |

## Analytics and Success Signals

- **organiser_role_transferred** (household member count) -- N/A -- no Stage 2 metric measures organiser role transfers; "Household Member Participation" measures a member joining and using the shared plan and list, not which member holds the organiser role, so this signal is retained for operational visibility only

## Acceptance Criteria

**FEAT-09.SPEC-009-AC-01:** Given Sam accepts a still-outstanding hand-over request from Maya, when this automation processes the acceptance, then Household.organiser is set to Sam, Sam's member_type becomes Organiser, and Maya's member_type becomes Other Adult Member.

**FEAT-09.SPEC-009-AC-02:** Given the atomic update in AC-01 completes, when Sam next opens FEAT-01.SPEC-010, then he has full organiser-level access and Maya has Other Adult Member-level access.

**FEAT-09.SPEC-009-AC-03:** Given Maya cancels the request at almost exactly the same moment Sam accepts it, and the cancel is recorded first, when this automation processes Sam's accept, then it finds no outstanding request and FEAT-09.SPEC-004 shows the Withdrawn state.

**FEAT-09.SPEC-009-AC-04:** Given Sam's accept is recorded before Maya's near-simultaneous cancel, when both are processed, then the transfer completes and Maya's cancel attempt against the now-completed transfer is itself rejected.

**FEAT-09.SPEC-009-AC-05:** Given a processing error occurs after the outstanding-request check passes but before the atomic update completes, when the failure occurs, then Household.organiser and both members' member_type remain exactly as before, and FEAT-09.SPEC-004 shows an inline error with the request still outstanding.

**FEAT-09.SPEC-009-AC-06:** Given this automation is somehow triggered twice for the same already-processed request, when the second run executes, then it finds no outstanding request and performs no further change.

**FEAT-09.SPEC-009-AC-07:** Given Maya (the former organiser) has FEAT-01.SPEC-008 (an organiser-only screen) open at the moment the swap completes, when she next attempts to save a budget change, then it is rejected with "Only the organiser can change this," per FEAT-01.SPEC-016.

**FEAT-09.SPEC-009-AC-08:** Given the transfer completes successfully, when Maya next opens FEAT-09.SPEC-003, then it no longer shows an outstanding request and behaves as a non-organiser attempting to reach that screen (per FEAT-09.SPEC-011).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, rejected -- withdrawn, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

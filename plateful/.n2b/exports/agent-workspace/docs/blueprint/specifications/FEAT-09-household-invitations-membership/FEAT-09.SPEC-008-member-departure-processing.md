---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-008
spec_name: Member Departure Processing
spec_slug: member-departure-processing
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Member Departure Processing

## Overview

**Name:** Member Departure Processing
**ID:** FEAT-09.SPEC-008
**Type:** Automation
**Purpose:** On a member leaving, anonymises their ratings, removes their Member Profile from active membership, and stops future plans from accounting for them.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the leaving member is eligible to leave (an Other Adult Member, never the current organiser) at the moment of processing
- Anonymising every Rating recorded by the leaving member, so it keeps influencing future plans without attribution
- Setting the leaving member's Member Profile status to Left and removing it from the active member list
- Triggering FEAT-09.SPEC-013 (Member Left Household Notification) to the organiser
- Ending the leaving member's session

**Non-Goals:**
- Deciding whether to leave -- the confirmation is owned by FEAT-09.SPEC-005 (Leave Household), which triggers this automation
- Deleting or anonymising the leaving member's Dietary Rules -- feature-overview.md's Shared Context flags this as an open product ambiguity not resolved by this feature; this automation touches only Ratings and Member Profile status, never Dietary Rule records
- Removing a member by the organiser's own action, or cascading a member's data on removal -- owned by FEAT-18 (Account & Data Management), a distinct path from this self-service leave (XBR-16 draws this exact line between removal and self-leaving)
- Stopping future plan and grocery-list accounting for the departed member -- the plan-generation and list-recalculation logic itself belongs to FEAT-03 and FEAT-06, which read the updated Member Profile status this automation sets; this automation only performs the status change, not the downstream recalculation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member confirms leaving | FEAT-09.SPEC-005 (Leave Household) | Fires on the final confirmation ("Yes, Leave"), before eligibility is re-confirmed | Leaving member's Member Profile reference |

## Processing Logic

1. Receive the leaving member's Member Profile reference from FEAT-09.SPEC-005.
2. Re-check eligibility per FEAT-09.SPEC-011: proceed only if the member's current member_type is Other Adult Member (never the current organiser).
3. If not eligible (the member has become organiser since the confirmation screen loaded), stop processing and return the block reason to FEAT-09.SPEC-005 -- no data changes.
4. If eligible, locate every Rating recorded by this member and anonymise it: the rating's value is retained for its influence on future meal selection, but its association with this specific member is removed.
5. Set the Member Profile's status to Left.
6. Remove the Member Profile from the household's active member list (it no longer appears in member counts, plan-participant lists, or grocery-list "added by" attributions).
7. Trigger FEAT-09.SPEC-013 (Member Left Household Notification) to the household's current organiser.
8. End the leaving member's session and sign them out of the household.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Departure succeeds | Member is an Other Adult Member (not organiser) at processing time | Ratings anonymised; Member Profile.status set to Left and removed from the active list | Member is signed out and routed to FEAT-01.SPEC-001 with "You've left the household."; organiser receives FEAT-09.SPEC-013 | FEAT-09.SPEC-005, FEAT-01.SPEC-001, FEAT-09.SPEC-013, FEAT-03, FEAT-06, FEAT-12 |
| Departure blocked -- now organiser | Member's member_type has changed to Organiser since the confirmation screen loaded (a hand-over completed in another session) | No data changes | FEAT-09.SPEC-005 shows "You're the organiser now -- hand over the role or delete the household to leave." | FEAT-09.SPEC-005 |
| Automation failure | Processing error after eligibility passes but before both the rating anonymisation and the status change are durably applied | No partial change persists -- rating anonymisation and Member Profile status change apply together or not at all | FEAT-09.SPEC-005 shows an inline error; the member remains a full household member | FEAT-09.SPEC-005 |

## Data Model

**Reads:** Member Profile -- member_type (eligibility check); Rating -- every record where member is the leaving member.
**Creates:** None.
**Updates:** Rating -- member association removed (anonymised) on every rating recorded by the leaving member; Member Profile -- status set to Left.
**Deletes:** None -- the Member Profile record and its anonymised ratings are retained, not purged (feature-overview.md, Entity-Lifecycle Coverage Matrix: soft removal, retained indefinitely as historical record).

## Business Rules

- A member who leaves on their own keeps their ratings only as anonymous influence on future plans; this is distinct from organiser-initiated removal (FEAT-18), which deletes the departing member's dietary rules and ratings outright (XBR-16). This automation never deletes a Rating.
- The organiser can never trigger this automation against herself; XBR-15 requires a hand-over or household deletion first, enforced by FEAT-09.SPEC-011 and re-checked here at processing time, not only at screen entry.
- The leaving member's Member Profile has no restore path: a later re-invitation of the same person creates a genuinely new Member Profile rather than reactivating this one (XBR-18).
- Rating anonymisation and the Member Profile status change apply together as one unit -- no state exists where ratings are anonymised but the profile is still Active, or the profile is Left but ratings still carry attribution.

## Edge Cases

- **Member is handed the organiser role (accepts via FEAT-09.SPEC-004) at almost exactly the same moment they confirm leaving via FEAT-09.SPEC-005** -- Reject-with-refresh: whichever change is recorded first wins. If the hand-over acceptance is recorded first, this automation's eligibility re-check (Processing Logic, step 2-3) finds the member is now organiser and blocks the departure.
- **The member has multiple ratings across several past weeks' plans** -- Every rating recorded by the member is anonymised in the same processing run; none are left attributed and none are skipped.
- **The member has never rated any meal** -- Anonymisation step (Processing Logic, step 4) has no records to act on; the Member Profile status change (step 5) still proceeds normally.
- **The leaving member is the only Other Adult Member, leaving the household with only the organiser** -- Departure proceeds normally; a household with only its organiser as an active adult member is a valid state.
- **Concurrent trigger firing (defensive case -- the same member somehow triggers this automation twice, e.g., a double network retry)** -- The second run finds the Member Profile already Left (idempotent check against current status) and takes no further action; ratings are not anonymised a second time since the association removal is already applied and has no further effect to repeat.
- **Trigger fires while a previous run is still in flight for the same member** -- FEAT-09.SPEC-005's "Yes, Leave" button is disabled during processing, preventing a second trigger from the same session; a second trigger from a different session for the same member processes against the Member Profile's current status and, if already Left, takes the idempotent no-further-action path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-005 (Leave Household) | Triggered by (inbound) | The final "Yes, Leave" confirmation fires this automation |
| FEAT-09.SPEC-005 (Leave Household) | Affects (outbound) | Returns success or the organiser-blocked reason |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Eligibility re-check (Other Adult Member only) |
| FEAT-09.SPEC-013 (Member Left Household Notification) | Triggers (outbound) | Notifies the organiser of the departure |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Affects (outbound) | Destination after the leaving member's session ends |
| FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-06 (Shared Grocery List) | Affects (outbound) | Read the updated Member Profile status so future plans and lists stop accounting for the departed member |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound) | Continues to read the member's ratings, now anonymised, as ongoing influence on selection |

## Analytics and Success Signals

- **member_left_household** (ratings_anonymised_count) -- N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; a departure is the inverse signal and does not itself feed a positive contribution to that target, so this event is retained only to explain drops in household participation when reading that metric, not cited as a direct contributor
- **member_departure_blocked_organiser** -- N/A -- diagnostic signal only, confirming XBR-15's block is exercised correctly

## Acceptance Criteria

**FEAT-09.SPEC-008-AC-01:** Given Sam (Other Adult Member) confirms leaving on FEAT-09.SPEC-005, when this automation processes the departure, then every rating he recorded is anonymised and his Member Profile status is set to Left.

**FEAT-09.SPEC-008-AC-02:** Given Sam's departure is processed successfully, when processing completes, then he is signed out and routed to FEAT-01.SPEC-001, and Maya (the organiser) receives FEAT-09.SPEC-013.

**FEAT-09.SPEC-008-AC-03:** Given Sam accepts a hand-over and becomes organiser in another session at almost the same moment he confirms leaving, and the hand-over is recorded first, when this automation runs its eligibility check, then the departure is blocked with "You're the organiser now -- hand over the role or delete the household to leave."

**FEAT-09.SPEC-008-AC-04:** Given Sam has never rated any meal, when his departure is processed, then his Member Profile status is still set to Left and no rating anonymisation step has any records to act on.

**FEAT-09.SPEC-008-AC-05:** Given Sam is the household's only Other Adult Member, when his departure is processed, then the household is left with only Maya as an active adult member, which is a valid resulting state.

**FEAT-09.SPEC-008-AC-06:** Given a processing error occurs after eligibility passes but before both the rating anonymisation and the status change are durably applied, when the failure occurs, then Sam remains a full active member with no partial anonymisation or status change.

**FEAT-09.SPEC-008-AC-07:** Given this automation is somehow triggered twice for the same already-Left member, when the second run processes, then it finds the profile already Left and takes no further action, leaving the anonymised ratings unaffected.

**FEAT-09.SPEC-008-AC-08:** Given Sam's departure completes, when FEAT-03 next generates or FEAT-06 next recalculates a plan, then neither accounts for Sam, since his Member Profile no longer appears in the active member list.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, blocked -- now organiser, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

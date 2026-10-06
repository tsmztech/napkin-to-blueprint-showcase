---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-007
spec_name: Member Removal Processing
spec_slug: member-removal-processing
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Member Removal Processing

## Overview

**Name:** Member Removal Processing
**ID:** FEAT-18.SPEC-007
**Type:** Automation
**Purpose:** On confirmed member removal, permanently deletes the member's dietary rules and ratings and stops future plans from accounting for them.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Deleting the removed member's Member Profile, Dietary Rules, and Ratings
- Ensuring future plan generation and manual planning stop accounting for the removed member
- Reporting success or failure back to the triggering screen

**Non-Goals:**
- The removal confirmation UI itself -- owned by FEAT-18.SPEC-002 (Remove Member Profile), which triggers this automation
- A member's own self-initiated departure -- a distinct outcome (anonymised ratings, not deleted) owned by FEAT-09.SPEC-008 (Member Departure Processing)
- Removing the organiser -- never a valid input to this automation; FEAT-18.SPEC-002 never offers the organiser as a removable target (XBR-15)
- Household-wide deletion -- owned by FEAT-18.SPEC-008 (Household Deletion Processing), a distinct, larger-scope automation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member removal confirmed | FEAT-18.SPEC-002 (Remove Member Profile) | Fires when Maya confirms removal in the irreversible-action confirmation step | Member Profile reference (the member being removed), household reference |

## Processing Logic

1. Receive the confirmed removal request (member reference, household reference).
2. Re-verify the target member is still an active, non-organiser member of the household (guards against the race described in Edge Cases).
3. Delete every Dietary Rule belonging to the member.
4. Delete every Rating authored by the member.
5. Set the Member Profile's status to Removed and remove it from the household's active member list.
6. Ensure the removed member is excluded from any future plan generation (FEAT-03) or manual planning (FEAT-23) pick lists and from the safety check's per-member evaluation, from this point forward.
7. Signal FEAT-18.SPEC-002 that removal completed, so the screen can show its success toast and refresh the member list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Removal completed | All steps succeed | Member Profile status set to Removed; the member's Dietary Rules and Ratings deleted | FEAT-18.SPEC-002 shows "{member_name} has been removed." and returns to the member list | FEAT-18.SPEC-002, FEAT-01, FEAT-12 |
| Target no longer valid | The re-verification step finds the member already removed or no longer non-organiser (a race, per Edge Cases) | No data changes | FEAT-18.SPEC-002 shows "This member was already removed." | FEAT-18.SPEC-002 |
| Removal failed | A deletion step fails partway | No partial state is left visible: the member remains Active and none of its Dietary Rules or Ratings are deleted until the whole sequence can complete | FEAT-18.SPEC-002 shows "We couldn't remove {member_name}. Try again." with a Retry button | FEAT-18.SPEC-002 |

## Data Model

**Reads:** Member Profile -- status, member_type, for the target member and to confirm the requester is the organiser.
**Creates:** None.
**Updates:** Member Profile -- status (set to Removed).
**Deletes:** Dietary Rule -- every rule belonging to the removed member; Rating -- every rating authored by the removed member.

## Business Rules

- XBR-16: removing a member deletes their dietary rules and ratings and future plans stop accounting for them -- this is a hard delete, distinct from FEAT-09's self-leave path, which anonymises rather than deletes ratings.
- The organiser can never be the target of this automation (XBR-15); FEAT-18.SPEC-002 never offers her as a removable member, and this automation treats an organiser target as an invalid input it must never receive.
- This automation is non-reversible: once Dietary Rules and Ratings are deleted, no restore path exists (product-features.md, Primary Flows & Alternates: "permanently removed").
- The removed member's data feeds no future plan or safety check from the moment removal completes (Cross-Feature Touchpoints: FEAT-01, FEAT-12).

## Edge Cases

- **Concurrent trigger firing (two organiser sessions confirm removal of the same member at effectively the same time)** -- The re-verification step in Processing Logic step 2 ensures only the first-committed removal proceeds; the second run finds the member already Removed and reports "Target no longer valid" rather than attempting a duplicate deletion.
- **Trigger fires while a previous run is in flight for a different member** -- Runs for different members proceed independently; each member's Dietary Rule and Rating deletions are scoped to that member alone and do not block or interact with a concurrent removal of someone else.
- **Sam is mid-edit of his own account (FEAT-18.SPEC-004) when this automation removes him** -- Per the dependency map's Contention note for Member Profile, this automation's removal wins; Sam's concurrent save is refused with "This account no longer exists in this household."
- **The removed member has a rating pending recording by another adult (e.g., an adult was about to record a young kid's rating on the removed kid's behalf)** -- Once the removal completes, no new Rating can be created for the removed member; an in-flight rating submission for that member is rejected with the same "This member no longer exists" style message.
- **Household is deleted while a member removal is still processing** -- Household deletion (FEAT-18.SPEC-008) supersedes any in-flight member removal per the dependency map's Contention note for Household; the removal's remaining steps are superseded by the household-wide cascade rather than run to completion separately.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-002 (Remove Member Profile) | Triggered by (inbound) | Confirmed removal starts this automation |
| FEAT-18.SPEC-002 (Remove Member Profile) | Affects (outbound) | Success, race, and failure feedback surface here |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion supersedes an in-flight member removal |
| FEAT-01 (Household Setup & Member Profiles) | Affects (outbound, cross-feature) | The removed member's profile and dietary rules disappear from FEAT-01's lists |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound, cross-feature) | The removed member's ratings no longer feed learned preferences |

## Analytics and Success Signals

- **member_deleted** (member_type: adult / kid) -- N/A -- no Stage 2 success metric measures member removal directly; retained per product-features.md's Signals field (member_deleted) as the operational record of this lifecycle action
- **member_removal_failed** (step_failed) -- N/A -- no Stage 2 success metric measures removal failures; retained to observe whether the no-partial-state guarantee is ever actually exercised

## Acceptance Criteria

**FEAT-18.SPEC-007-AC-01:** Given Maya confirms removal of Sam on FEAT-18.SPEC-002, when this automation processes the request, then Sam's Dietary Rules and Ratings are deleted and his Member Profile status is set to Removed.

**FEAT-18.SPEC-007-AC-02:** Given a kid profile is removed, when this automation completes, then the kid's Dietary Rules and Ratings are deleted identically to an adult member's.

**FEAT-18.SPEC-007-AC-03:** Given a member is removed, when the next plan generation or manual planning session runs, then the removed member is excluded from all planning and safety-check evaluation.

**FEAT-18.SPEC-007-AC-04:** Given two organiser sessions confirm removal of the same member at effectively the same time, when the second run's re-verification runs, then it finds the member already Removed and FEAT-18.SPEC-002 shows "This member was already removed." on that session.

**FEAT-18.SPEC-007-AC-05:** Given Sam has an unsaved own-account edit open when this automation removes him, when he attempts to save, then he sees "This account no longer exists in this household." rather than a generic error.

**FEAT-18.SPEC-007-AC-06:** Given a deletion step fails partway through processing, when the failure is reported, then no partial state is visible -- the member remains Active with all Dietary Rules and Ratings intact -- and FEAT-18.SPEC-002 shows "We couldn't remove {member_name}. Try again."

**FEAT-18.SPEC-007-AC-07:** Given a household is deleted while a member removal for that household is in flight, when household deletion processing (FEAT-18.SPEC-008) runs its cascade, then the member removal's remaining steps are superseded by the household-wide deletion.

**FEAT-18.SPEC-007-AC-08:** Given this automation is invoked (defensively) with the organiser as its target, when processing begins, then it treats this as an invalid input and takes no action, since FEAT-18.SPEC-002 never offers the organiser as removable.

**FEAT-18.SPEC-007-AC-09:** Given two different members are removed by two separate confirmed requests at the same time, when both run, then each completes independently without blocking or interacting with the other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, target no longer valid, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-009
spec_name: Own Account Deletion Processing
spec_slug: own-account-deletion-processing
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Own Account Deletion Processing

## Overview

**Name:** Own Account Deletion Processing
**ID:** FEAT-18.SPEC-009
**Type:** Automation
**Purpose:** Permanently deletes an adult's own account, blocked for the organiser unless the role was handed over or the household deleted first.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Re-checking the organiser hand-over-or-delete-first precondition immediately before deletion
- Permanently deleting the requesting adult's own Member Profile, Dietary Rules, and Ratings
- Reporting success or a blocked outcome back to FEAT-18.SPEC-004

**Non-Goals:**
- The precondition's definition -- owned by FEAT-18.SPEC-010 (Account & Data Validation Rules), which this automation enforces rather than redefines
- Organiser role hand-over itself -- owned by FEAT-09 (Household Invitations & Membership); this automation only checks whether a hand-over has completed
- Deleting a member other than the requester's own account -- that is organiser-initiated removal, owned by FEAT-18.SPEC-007 (Member Removal Processing)
- Household-wide deletion -- owned by FEAT-18.SPEC-008 (Household Deletion Processing); an organiser may choose that path instead of hand-over, but this automation only deletes one adult's own account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Own-account deletion confirmed | FEAT-18.SPEC-004 (My Account) | Fires when the adult member confirms deletion in the irreversible-action confirmation step, after the precondition check has already passed once at that point | Member Profile reference (the requester), household reference |

## Processing Logic

1. Receive the confirmed own-account deletion request (member reference, household reference).
2. Re-check the organiser-hand-over-or-delete-first precondition (FEAT-18.SPEC-010): if the requester is currently the household's organiser and no hand-over has completed and the household has not been deleted, block the deletion.
3. If the precondition passes, delete every Dietary Rule belonging to the requester.
4. Delete every Rating authored by the requester.
5. Set the requester's Member Profile status to Removed and remove it from the household's active member list.
6. Ensure the requester is excluded from any future plan generation, manual planning, or safety-check evaluation from this point forward.
7. Sign the requester out.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Deletion completed | Precondition passes and all steps succeed | Member Profile status set to Removed; requester's Dietary Rules and Ratings deleted | Requester is signed out; FEAT-18.SPEC-004 shows a signed-out landing | FEAT-18.SPEC-004, FEAT-01, FEAT-12 |
| Precondition blocked | Requester is still the organiser with no completed hand-over and the household has not been deleted (re-checked at processing time, not just at the initial screen check) | No data changes | FEAT-18.SPEC-004 shows the deletion-blocked message again, since the precondition state changed between the screen check and processing | FEAT-18.SPEC-004 |
| Deletion failed | A deletion step fails partway | No partial state is left visible: the requester's Member Profile remains Active and none of its Dietary Rules or Ratings are deleted | FEAT-18.SPEC-004 shows "We couldn't delete your account. Try again." with a Retry button | FEAT-18.SPEC-004 |

## Data Model

**Reads:** Member Profile -- status, member_type, for the requester; Household -- organiser, status, to evaluate the precondition.
**Creates:** None.
**Updates:** Member Profile -- status (set to Removed).
**Deletes:** Dietary Rule -- every rule belonging to the requester; Rating -- every rating authored by the requester.

## Business Rules

- XBR-15: the organiser cannot delete her own account while she remains the organiser and has not handed over the role or deleted the household first -- this automation re-checks that precondition at processing time, not only at the screen-level check, since the household's organiser state could change between the two moments.
- Own-account deletion is a hard delete with no restore path (product-features.md, Primary Flows & Alternates), distinct from a member's self-leave path (FEAT-09.SPEC-005), which anonymises rather than deletes ratings.
- The deleted requester's data feeds no future plan or safety check from the moment deletion completes.
- Only the requesting adult's own account can be the target of this automation -- it is never invoked with a target other than the session's own Member Profile.

## Edge Cases

- **Concurrent trigger firing (the organiser starts a hand-over acceptance on one device while confirming her own deletion on another)** -- The re-check in Processing Logic step 2 reads the household's organiser state at the moment this automation runs; whichever change (hand-over completion or deletion confirmation) is durably recorded first determines the outcome, consistent with first-committed-wins.
- **Trigger fires while a previous run is in flight for the same requester (double confirmation)** -- FEAT-18.SPEC-004's disabled confirmation button during the Deleting state prevents a second trigger for the same requester while the first is processing.
- **The organiser's hand-over completes seconds before she confirms deletion, but the completion has not yet propagated to her own screen's precondition check** -- Because this automation re-checks the precondition at processing time (step 2), a hand-over that completed before this automation runs is honored even if FEAT-18.SPEC-004's earlier screen-level check was stale.
- **The requester is removed by the organiser (FEAT-18.SPEC-007) at the same moment they confirm their own deletion** -- Per the dependency map's Contention note for Member Profile, whichever change is recorded first wins; if the organiser's removal lands first, this automation finds the requester already Removed and reports a blocked-equivalent outcome ("This account no longer exists in this household.") rather than double-deleting.
- **Household is deleted while own-account deletion is processing** -- Household deletion (FEAT-18.SPEC-008) supersedes this automation per the dependency map's Contention note for Household; the household-wide cascade completes the requester's removal as part of its own scope.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-004 (My Account) | Triggered by (inbound) | Confirmed own-account deletion starts this automation |
| FEAT-18.SPEC-004 (My Account) | Affects (outbound) | Completed, blocked, and failure feedback surface here |
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Defines the hand-over-or-delete-first precondition this automation enforces |
| FEAT-09 (Household Invitations & Membership) | References (inbound, cross-feature) | Organiser hand-over completion satisfies the precondition |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion supersedes an in-flight own-account deletion |

## Analytics and Success Signals

- **own_account_deleted** (role_at_deletion: organiser / other_adult) -- N/A -- no Stage 2 success metric measures own-account deletion directly; retained per product-features.md's Signals field (own_account_deleted) as the operational record of this lifecycle action
- **own_account_deletion_blocked_at_processing** (reason: organiser_no_handover) -- N/A -- no Stage 2 success metric measures this race outcome; retained to observe whether the screen-level and processing-time precondition checks ever genuinely disagree

## Acceptance Criteria

**FEAT-18.SPEC-009-AC-01:** Given Sam (never the organiser) confirms his own account deletion, when this automation processes the request, then his Dietary Rules and Ratings are deleted, his Member Profile status is set to Removed, and he is signed out.

**FEAT-18.SPEC-009-AC-02:** Given Maya has completed handing over the organiser role and then confirms her own deletion, when this automation re-checks the precondition, then it passes and her account is deleted.

**FEAT-18.SPEC-009-AC-03:** Given Maya is still the organiser with no completed hand-over when this automation processes a deletion request for her, when the precondition check runs, then the deletion is blocked and FEAT-18.SPEC-004 shows the deletion-blocked message again.

**FEAT-18.SPEC-009-AC-04:** Given Maya's hand-over completes between her screen-level check and this automation's processing, when the automation re-checks the precondition, then it passes based on the now-current organiser state.

**FEAT-18.SPEC-009-AC-05:** Given Maya is removed by another organiser session (a defensive, unusual case) at the same moment she confirms her own deletion, and the removal is recorded first, when this automation runs, then it finds her already Removed and reports "This account no longer exists in this household." rather than deleting a second time.

**FEAT-18.SPEC-009-AC-06:** Given a deletion step fails partway through processing, when the failure is reported, then no partial state is visible -- the requester's Member Profile remains Active -- and FEAT-18.SPEC-004 shows "We couldn't delete your account. Try again."

**FEAT-18.SPEC-009-AC-07:** Given a household is deleted while a requester's own-account deletion is in flight, when household deletion processing (FEAT-18.SPEC-008) runs its cascade, then the requester's removal is completed as part of that cascade instead.

**FEAT-18.SPEC-009-AC-08:** Given Sam has already confirmed his own deletion once and it is processing, when he attempts to confirm again, then FEAT-18.SPEC-004's disabled Deleting state prevents a second trigger.

**FEAT-18.SPEC-009-AC-09:** Given a deleted adult's account is later checked against any future plan generation, when planning runs, then the deleted account is excluded from all planning and safety-check evaluation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, precondition blocked, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

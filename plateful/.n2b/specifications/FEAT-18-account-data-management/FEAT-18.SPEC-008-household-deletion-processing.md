---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-008
spec_name: Household Deletion Processing
spec_slug: household-deletion-processing
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Household Deletion Processing

## Overview

**Name:** Household Deletion Processing
**ID:** FEAT-18.SPEC-008
**Type:** Automation
**Purpose:** On confirmed household deletion, permanently removes every member profile, plan, list, rule, and rating within 30 days.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Signing out every household member once deletion begins
- Cascading the deletion across every household entity: Member Profile, Weekly Plan, Planned Meal, Grocery List, Grocery List Item, Pantry Item, Rating, Dietary Rule, Support Request, Subscription
- Disconnecting the household's calendar connection, if one exists, via FEAT-21.SPEC-002 (Family Calendar Integration)
- Completing the full cascade within 30 days and triggering the completion notification

**Non-Goals:**
- The deletion confirmation UI -- owned by FEAT-18.SPEC-003 (Delete Household), which triggers this automation
- Cancelling billing separately -- deletion supersedes the Subscription regardless of billing_state; this automation does not duplicate FEAT-14's own cancellation flow, it simply removes the Subscription record as part of the cascade
- Removing a single member without deleting the household -- owned by FEAT-18.SPEC-007 (Member Removal Processing)
- Any restore path -- this is an intentional hard delete with no recovery mechanism, per product-features.md's Primary Flows & Alternates and the Non-Goals in feature-overview.md

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household deletion confirmed | FEAT-18.SPEC-003 (Delete Household) | Fires when Maya taps the confirmation step's explicit affirmative "Delete Household Permanently" action | Household reference |

## Processing Logic

1. Receive the confirmed deletion request (household reference).
2. Mark the Household's status as Closed/Deleted immediately, so no further edits, plans, or actions can be made against it from this moment.
3. Sign out every member currently signed in to the household within a short window of confirmation.
4. Begin cascading deletion across every owned entity: every Member Profile, every Weekly Plan and its Planned Meals, the Grocery List and its Grocery List Items, every Pantry Item, every Rating, every Dietary Rule, every Support Request, and the Subscription.
5. Disconnect the household's calendar connection, if one exists, via FEAT-21.SPEC-002 (Family Calendar Integration) -- sent as part of the cascade regardless of the connection's current status; a household with no calendar connection has nothing to disconnect and this step completes as a no-op.
6. Continue the cascade until every listed entity is fully removed, completing within 30 days of confirmation.
7. Once the cascade completes, permanently remove the Household record itself.
8. Signal FEAT-18.SPEC-014 (Household Deletion Completed Notification) that deletion has fully completed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Deletion started | Confirmation accepted | Household status set to Closed/Deleted; every member signed out | FEAT-18.SPEC-003 shows "Your household is being deleted..." before members are signed out | FEAT-18.SPEC-003 |
| Deletion completed | The full cascade finishes within the 30-day window | Every listed entity permanently removed; any active calendar connection disconnected (FEAT-21.SPEC-002); the Household record itself removed | FEAT-18.SPEC-014 delivers the completion confirmation to Maya | FEAT-18.SPEC-014, FEAT-18.SPEC-012, FEAT-21.SPEC-002 |
| Deletion start failed | Step 2 or 3 cannot be completed (e.g., the status change itself fails) | No change -- the household remains Active and reachable | FEAT-18.SPEC-003 shows "We couldn't start the deletion. Try again." | FEAT-18.SPEC-003 |

## Data Model

**Reads:** Household -- all fields, to identify every owned entity for the cascade.
**Creates:** None.
**Updates:** Household -- status (set to Closed/Deleted at the start of processing).
**Deletes:** Member Profile, Weekly Plan, Planned Meal, Grocery List, Grocery List Item, Pantry Item, Rating, Dietary Rule, Support Request, Subscription -- every record belonging to the household; and, on cascade completion, the Household record itself.

## Business Rules

- Household deletion supersedes any in-flight edit by any member, per the dependency map's Contention note for Household -- once step 2 marks the household Closed/Deleted, no concurrent action on any owned entity is honored.
- Deletion completes within 30 days of confirmation and no deleted data is retained in any form usable for other purposes (product-features.md, Validation & Limits; ASMP-27) -- the 30-day window is a stated product decision, not an instant purge, and is a fixed, feature-level value stated as a concrete number because it is specific to this feature's own definition, not a platform-wide policy value delegated to build time.
- This is a hard delete with no restore path (XBR-16), distinct from a member's individual self-leave path (FEAT-09), which is restorable in spirit for that member alone.
- Only one household deletion runs per household at a time -- a second confirmation attempt while one is already processing is rejected, per FEAT-18.SPEC-003's Edge Cases.

## Edge Cases

- **Concurrent trigger firing (two confirmation attempts for the same household at effectively the same time)** -- Only the first-committed confirmation proceeds; a second attempt is rejected with "Deletion is already in progress for this household." per FEAT-18.SPEC-003's Edge Cases, since the household's status is already Closed/Deleted by the time the second attempt is evaluated.
- **Trigger fires while a previous run is in flight** -- Cannot occur for the same household: once status is Closed/Deleted, FEAT-18.SPEC-003 no longer offers a fresh confirmation. Runs for different households proceed entirely independently.
- **An export (FEAT-18.SPEC-006) is compiling when deletion begins** -- Deletion supersedes the in-flight export per the dependency map's Contention note for Household; the export compilation is cancelled and no export file is finalized.
- **A member is mid-swap, mid-plan-approval, or otherwise mid-action on any household entity when deletion begins** -- Every such in-flight action is superseded; the acting member's screen refreshes to reflect the household's Closed/Deleted state, and no partial data change from that in-flight action persists.
- **The household's Subscription is mid-grace-period (a payment failure) when deletion is confirmed** -- The Subscription record is deleted as part of the cascade regardless of billing_state; no separate cancellation flow is required to complete first.
- **The household has no calendar connection (never connected, or already disconnected) when deletion runs** -- The disconnect step (FEAT-21.SPEC-002) finds no active connection to disconnect and completes as a no-op; the rest of the cascade proceeds unaffected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-003 (Delete Household) | Triggered by (inbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-003 (Delete Household) | Affects (outbound) | Deletion-in-progress and error feedback surface here |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Triggers (outbound) | The completed outcome fires this notification |
| FEAT-18.SPEC-006 (Export Generation Processing) | Affects (outbound) | An in-flight export is cancelled by this cascade |
| FEAT-18.SPEC-007 (Member Removal Processing) | Affects (outbound) | An in-flight member removal is superseded by this cascade |
| FEAT-21.SPEC-002 (Family Calendar Integration) | Triggers (outbound, cross-feature) | Cascade disconnects any active household calendar connection |
| FEAT-01, FEAT-03, FEAT-06, FEAT-05, FEAT-12 (cross-feature) | Affects (outbound, cross-feature) | Every entity these features own within the household is removed by this cascade |

## Analytics and Success Signals

- **household_deleted** (member_count_at_deletion) -- N/A -- no Stage 2 success metric measures household deletion directly; retained per product-features.md's Signals field (household_deleted) as the terminal record of this lifecycle action
- **household_deletion_completed** (days_to_complete) -- N/A -- no Stage 2 success metric measures deletion completion timing; retained to confirm the 30-day commitment is actually met in practice

## Acceptance Criteria

**FEAT-18.SPEC-008-AC-01:** Given Maya confirms household deletion, when this automation starts, then the Household status is set to Closed/Deleted and every signed-in member is signed out shortly after.

**FEAT-18.SPEC-008-AC-02:** Given deletion has started, when the cascade runs, then every Member Profile, Weekly Plan, Grocery List, Pantry Item, Rating, Dietary Rule, and Support Request belonging to the household is permanently removed.

**FEAT-18.SPEC-008-AC-03:** Given the cascade completes within 30 days, when the Household record is finally removed, then FEAT-18.SPEC-014 delivers the completion confirmation to Maya.

**FEAT-18.SPEC-008-AC-04:** Given a second confirmation attempt is made for a household already marked Closed/Deleted, when it is evaluated, then it is rejected with "Deletion is already in progress for this household."

**FEAT-18.SPEC-008-AC-05:** Given an export is compiling for a household when deletion is confirmed, when deletion processing begins, then the export compilation is cancelled and no export file is finalized.

**FEAT-18.SPEC-008-AC-06:** Given Sam is mid-edit of a shared entity when deletion is confirmed, when the household status changes to Closed/Deleted, then his screen refreshes to reflect the closed household and his in-flight action is superseded.

**FEAT-18.SPEC-008-AC-07:** Given the household's Subscription is in a Payment failed grace period when deletion is confirmed, when the cascade runs, then the Subscription record is deleted regardless of its billing_state.

**FEAT-18.SPEC-008-AC-08:** Given step 2 (marking the household Closed/Deleted) fails, when this occurs, then the household remains Active and FEAT-18.SPEC-003 shows "We couldn't start the deletion. Try again."

**FEAT-18.SPEC-008-AC-09:** Given a member removal for the household is in flight when household deletion is confirmed, when the household-wide cascade runs, then the individual member removal's remaining steps are superseded by it.

**FEAT-18.SPEC-008-AC-10:** Given two different households each confirm deletion at the same time, when both are processed, then each household's cascade completes independently without interacting with the other.

**FEAT-18.SPEC-008-AC-11:** Given a household with an active calendar connection confirms deletion, when the cascade runs, then this automation signals FEAT-21.SPEC-002 (Family Calendar Integration) to disconnect that connection as part of the cascade.

**FEAT-18.SPEC-008-AC-12:** Given a household with no calendar connection confirms deletion, when the cascade reaches the calendar-disconnect step, then the step finds nothing to disconnect and completes as a no-op without affecting the rest of the cascade.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (started, completed, start failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-003
spec_name: Delete Household
spec_slug: delete-household
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Delete Household

## Overview

**Name:** Delete Household
**ID:** FEAT-18.SPEC-003
**Type:** Screen
**Purpose:** Maya sees exactly what will be lost and confirms permanent deletion of the entire household.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Showing a summary of the household's members, plans, and history that deletion will remove
- The irreversible-action confirmation for household deletion
- Triggering deletion and showing its immediate, in-progress, and completed states

**Non-Goals:**
- Performing the cascade deletion across every household entity -- owned by FEAT-18.SPEC-008 (Household Deletion Processing), which this screen triggers
- Removing a single member without deleting the whole household -- owned by FEAT-18.SPEC-002 (Remove Member Profile)
- Deleting only the organiser's own account while the household continues -- owned by FEAT-18.SPEC-004 and FEAT-18.SPEC-009; this screen deletes the entire household, every member included
- Any billing cancellation mechanics -- deletion supersedes the household's Subscription regardless of billing_state; FEAT-14 owns billing cancellation as its own standalone action, which this screen does not duplicate

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Delete household" in the organiser-only Household Data & Deletion section | None -- screen loads a fresh deletion-preview summary |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Confirm household deletion | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can delete the household." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable through this login; a direct attempt shows "Only the household organiser can delete the household." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no deletion was in progress to preserve, since confirmation is a single action |

## Layout and Content

**Header:** Screen title "Delete Household" with a back arrow (returns to FEAT-18.SPEC-004).

**Body:** A single content column.
- A summary card reading exactly what will be lost, built from the household's current data: the number of member profiles (naming each by display_name), the number of weeks of plan history, the number of items on the current grocery list, and the line "Every dietary rule, rating, and support request will be permanently removed too."
- A warning line: "This cannot be undone. Deletion is permanent and there is no way to recover this household's data afterward."
- The confirmation control below the summary: a "Delete Household Permanently" button (destructive, enabled directly -- no precondition beyond the summary having loaded) and a "Cancel" action that returns to FEAT-18.SPEC-004.

**Confirmation step (appears in place of the Preview body when Delete Household Permanently is tapped):** The same summary of what will be lost, restated, plus the warning line, and two actions: "Delete Household Permanently" (destructive, requires the explicit affirmative tap) and "Cancel" (returns to the Preview state). No default-confirmed state -- neither action is preselected or triggered by any other interaction, consistent with the Shared Context's irreversible-action confirmation pattern (also used identically by FEAT-18.SPEC-002 and within FEAT-18.SPEC-004).

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-004 | Screen closes | Standard transition back |
| "Delete Household Permanently" (Preview) | Tap | Opens the confirmation step | Preview body is replaced by the confirmation step | Confirmation step appears restating the loss summary |
| Confirmation "Delete Household Permanently" | Tap | Triggers FEAT-18.SPEC-008 (Household Deletion Processing) | Screen switches to the in-progress state | Progress indicator with the label "Deleting your household..." |
| Confirmation "Cancel" | Tap | Discards the confirmation | Confirmation step is replaced by the Preview body | Preview body reappears unchanged |
| Cancel (Preview) | Tap | Discards the action | Navigates to FEAT-18.SPEC-004 | Standard transition back, no deletion occurs |

### Accessibility Notes

- **Focus order:** Back arrow -> summary card -> warning line -> Delete Household Permanently button -> Cancel button; on the confirmation step -> restated loss summary -> confirmation's Delete Household Permanently button -> Cancel button.
- **Confirmation announcement:** Entering the confirmation step announces the restated loss summary to assistive technology, so the destructive action's consequences are read before the confirmation's Delete Household Permanently control is reachable, identical to FEAT-18.SPEC-002's confirmation-step announcement.
- **Progress announcement:** Entering the in-progress state announces "Deleting your household" to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Summary card area shows loading placeholders in place of member/plan/list counts | Screen first opens, before the initial fetch of the deletion-preview summary completes | Fetch succeeds (-> Preview) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your household's deletion summary. Try again." with a Retry button; no confirmation controls are shown until the summary loads | The initial fetch of the deletion-preview summary fails | Maya taps Retry (re-fetches) or the back arrow (returns to FEAT-18.SPEC-004) |
| Preview (default) | Summary card, warning, and an enabled Delete Household Permanently button | Deletion-preview summary loads successfully | Maya taps Delete Household Permanently or Cancel |
| Confirmation | Restated loss summary, warning, and Delete Household Permanently/Cancel buttons; no default-confirmed state | Maya taps Delete Household Permanently on the Preview state | Maya taps the confirmation's Delete Household Permanently (confirmed) or Cancel |
| Deleting | Progress indicator "Deleting your household..."; screen is non-interactive except a note that deletion continues even if she closes the app | Maya taps the confirmation's Delete Household Permanently button | Deletion processing (FEAT-18.SPEC-008) begins its cascade |
| Deletion in progress (post-confirmation) | A persistent notice replacing the household's usual areas: "Your household is being deleted. This can take up to 30 days to fully complete; you'll be signed out shortly and a completion email will follow." | Confirmation accepted by FEAT-18.SPEC-008 | The household record is fully removed and Maya is signed out |
| Confirm Error | Error banner "We couldn't start the deletion. Try again." with a Retry button; the confirmation step's summary remains | FEAT-18.SPEC-008 fails to begin processing | Maya taps Retry or Cancel |
| Offline/Degraded | Banner "You're offline -- deletion will begin once you reconnect." at top; the confirmation's Delete Household Permanently button queues the request locally | Connectivity lost while the confirmation step is open | Connectivity restored -- the queued confirmation submits automatically and the screen shows the Deleting state |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules), which defines the irreversible-action confirmation requirement. This screen's confirmation step is the enforcement of that rule for this specific action, identical to FEAT-18.SPEC-002's and FEAT-18.SPEC-004's confirmation steps.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-18.SPEC-004 (My Account) | -- |
| Cancel tap | FEAT-18.SPEC-004 (My Account) | -- |
| Deletion confirmed | Signed-out landing (the product's sign-in screen) once deletion processing begins | -- |

## Data Model

**Creates:** None.
**Reads:** Household -- household_name, and a derived summary count of Member Profiles, Weekly Plans, and current Grocery List items belonging to the household.
**Updates:** None directly -- deletion is performed by FEAT-18.SPEC-008.
**Deletes:** None directly -- the Household record and its cascade are deleted by FEAT-18.SPEC-008.

## Business Rules

- Deletion requires the same explicit-affirmative-tap confirmation step used by FEAT-18.SPEC-002 (Remove Member Profile) and FEAT-18.SPEC-004 (My Account's own-account deletion), per FEAT-18.SPEC-010's irreversible-action confirmation rule -- the confirmation step states plainly what will be lost and has no default-confirmed state.
- Only Maya (Organiser) can reach this screen and confirm deletion, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- Household deletion supersedes any in-flight edit by any member, per the dependency map's Contention note for Household -- no concurrent edit can block or be lost silently; deletion always wins.
- Deletion is a hard delete with no restore path, distinct from any member's individual departure (FEAT-09), which is restorable in spirit for that member alone.

## Edge Cases

- **Sam is editing household settings on another screen (a capability outside this feature) at the moment Maya confirms deletion** -- Per the dependency map's Contention note for Household, deletion supersedes the in-flight edit; Sam's screen is refreshed to reflect the household's deletion-in-progress state, and any unsaved change of his is discarded. This is the concurrent-edit conflict entry for this screen.
- **Maya taps Delete Household Permanently, then taps Cancel on the confirmation step** -- No deletion occurs; the confirmation step closes back to the Preview state unchanged, identical in mechanics to FEAT-18.SPEC-002's Cancel behavior.
- **Maya closes the app immediately after confirming deletion** -- Deletion processing continues independently of the app being open; FEAT-18.SPEC-008 completes its cascade and FEAT-18.SPEC-014 delivers the completion notification regardless.
- **Maya navigates back to this screen after already confirming deletion in a prior session (deletion still in progress)** -- The screen shows the "Deletion in progress" state directly rather than the Preview summary, since the household is no longer in a state that supports a fresh deletion request.
- **A second organiser session attempts to confirm deletion while a first confirmation is already processing** -- The second confirmation is rejected with "Deletion is already in progress for this household." since only one deletion can be in flight per household at a time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Irreversible-action confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-008 (Household Deletion Processing) | Triggers (outbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Affects (outbound) | Delivered once FEAT-18.SPEC-008 completes, independent of this screen |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| household_deletion_preview_viewed | member_count, plan_week_count | Maya opens this screen | N/A -- no Stage 2 success metric measures deletion-flow engagement; retained to observe how often the preview is reached versus confirmed, given the feature's Rationale that a credible export/delete path builds trust |
| household_deleted | -- | Deletion is confirmed and processing begins | N/A -- no Stage 2 success metric measures household deletion directly; retained since this is the terminal lifecycle event for a household and must be observable for the operator's own record-keeping |

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Maya is on the Delete Household screen, when it loads, then the summary card shows her household's member count, plan-history length, and current grocery-list item count, plus the warning that deletion cannot be undone.

**FEAT-18.SPEC-003-AC-02:** Given Maya is on the Preview state, when she taps Delete Household Permanently, then the confirmation step opens restating what will be lost, and no deletion has occurred yet.

**FEAT-18.SPEC-003-AC-03:** Given Maya is on the confirmation step, when she taps the confirmation's Delete Household Permanently button, then the screen shows "Deleting your household..." and FEAT-18.SPEC-008 begins.

**FEAT-18.SPEC-003-AC-04:** Given Maya has confirmed deletion, when processing begins, then she sees "Your household is being deleted. This can take up to 30 days to fully complete; you'll be signed out shortly and a completion email will follow."

**FEAT-18.SPEC-003-AC-05:** Given Maya taps Cancel on the Preview state, when the action is processed, then no deletion occurs and she returns to FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-06:** Given Maya taps Cancel on the confirmation step, when the action is processed, then no deletion occurs and she returns to the Preview state, not FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-07:** Given Sam has an unsaved household-settings edit open when Maya confirms deletion, when deletion begins, then Sam's screen refreshes to the deletion-in-progress state and his unsaved edit is discarded.

**FEAT-18.SPEC-003-AC-08:** Given Maya loses connectivity on the confirmation step, when she taps the confirmation's Delete Household Permanently button, then the banner "You're offline -- deletion will begin once you reconnect." appears and the confirmation queues locally.

**FEAT-18.SPEC-003-AC-09:** Given FEAT-18.SPEC-008 fails to begin processing, when Maya views the confirmation step, then she sees "We couldn't start the deletion. Try again." with a Retry button.

**FEAT-18.SPEC-003-AC-10:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can delete the household." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-11:** Given Maya's session expires while she is on this screen, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no deletion was confirmed.

**FEAT-18.SPEC-003-AC-12:** Given Maya returns to this screen after a deletion she confirmed in a prior session is still processing, when the screen loads, then it shows the deletion-in-progress state directly, not the Preview summary.

**FEAT-18.SPEC-003-AC-13:** Given Maya opens this screen, when the initial fetch of the deletion-preview summary is in progress, then the summary card area shows loading placeholders and no confirmation control is shown.

**FEAT-18.SPEC-003-AC-14:** Given the initial fetch of the deletion-preview summary fails, when Maya views this screen, then she sees "We couldn't load your household's deletion summary. Try again." with a Retry button, and no confirmation control is shown until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 8 (loading, load error, preview, confirmation, deleting, deletion in progress, confirm error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

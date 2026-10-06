---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-002
spec_name: Remove Member Profile
spec_slug: remove-member-profile
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Remove Member Profile

## Overview

**Name:** Remove Member Profile
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Maya selects a household member (adult or kid) and permanently removes them and their associated data.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Listing every removable member (every member except the organiser herself)
- Showing what will be lost for the selected member before removal
- The irreversible-action confirmation for removal
- Triggering the removal and reflecting its outcome

**Non-Goals:**
- Performing the cascade deletion of the member's Dietary Rules and Ratings -- owned by FEAT-18.SPEC-007 (Member Removal Processing), which this screen triggers
- Editing a member's profile details -- owned by FEAT-01 (Household Setup & Member Profiles); this screen only removes
- An adult member leaving the household on their own -- a distinct, self-initiated action owned by FEAT-09.SPEC-005 (Leave Household); this screen is organiser-initiated removal of another member (feature-overview.md, Shared Context's flagged ambiguity)
- Removing the organiser herself -- the organiser is never a selectable target on this screen; she must hand over the role (FEAT-09) or delete the household (FEAT-18.SPEC-003) first, per XBR-15

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Remove a member" in the organiser-only Household Data & Deletion section | None -- screen loads the current removable-member list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Select and remove any non-organiser member | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can remove a member." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable through this login; a direct attempt shows "Only the household organiser can remove a member." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the in-progress selection (before confirmation) is discarded |

## Layout and Content

**Header:** Screen title "Remove a Member" with a back arrow (returns to FEAT-18.SPEC-004).

**Body:** A single-column list, one row per removable member (every household member except Maya): the member's display name, member_type (Other Adult Member, or a kid label with age_band), and a "Remove" action per row. Selecting a row's Remove action opens the confirmation step below the list.

**Confirmation step (appears in place of the list when a member is selected):** A summary naming the selected member and stating plainly what will be lost: their dietary rules, their ratings, and their removal from all future plans -- consistent with the Shared Context's irreversible-action confirmation pattern (also used by FEAT-18.SPEC-003 and within FEAT-18.SPEC-004). Two actions: "Remove {member_name}" (destructive, requires the explicit affirmative tap) and "Cancel" (returns to the list). No default-confirmed state -- neither action is preselected or triggered by any other interaction.

### Responsive Behavior

- **Compact breakpoint:** Single-column list and confirmation step as described, full width.
- **Medium size class and above:** Content capped at a consistent platform-wide list width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-004 | Screen closes | Standard transition back |
| Member row "Remove" action | Tap | Opens the confirmation step for that member | List is replaced by the confirmation step | Confirmation step appears with the member's name and loss summary |
| Confirmation "Remove {member_name}" | Tap | Triggers FEAT-18.SPEC-007 (Member Removal Processing) for the selected member | Confirmation step shows a progress state | Progress indicator; on completion, success toast "{member_name} has been removed." and return to the member list |
| Confirmation "Cancel" | Tap | Discards the selection | Confirmation step is replaced by the member list | List reappears unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> member list rows in display order -> (on confirmation) loss summary -> Remove button -> Cancel button.
- **Confirmation announcement:** Entering the confirmation step announces the member's name and the loss summary to assistive technology, so the destructive action's consequences are read before the Remove control is reachable.
- **Success/failure announcement:** The "has been removed" toast and any error banner are announced on completion.
- **Keyboard alternatives:** Every action (select, confirm, cancel) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | List area shows loading placeholders in place of member rows | Screen first opens, before the initial fetch of the removable-member list completes | Fetch succeeds (-> Member list or Empty, whichever matches) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load the household's members. Try again." with a Retry button; no member rows or Remove actions are shown until data loads | The initial fetch of the removable-member list fails | Maya taps Retry (re-fetches) or the back arrow (returns to FEAT-18.SPEC-004) |
| Member list (default) | List of removable members | Initial fetch succeeds, or Cancel is tapped | Maya taps a member's Remove action |
| Confirmation | Loss summary and Remove/Cancel buttons for the selected member | Maya taps Remove on a member row | Maya taps Remove (confirmed) or Cancel |
| Removing | Confirmation step shows a progress indicator, both buttons disabled | Maya taps the confirmation Remove button | Removal completes or fails |
| Error | Error banner "We couldn't remove {member_name}. Try again." with a Retry button; confirmation step remains | FEAT-18.SPEC-007 reports failure | Maya taps Retry (re-attempts) or Cancel (returns to list, no removal occurred) |
| Empty (no removable members) | List area shows "There's no one else to remove yet -- invite a member from Household Settings to add one." | The household has only the organiser as a member | A member is added elsewhere (FEAT-01 or FEAT-09) |
| Offline/Degraded | Banner "You're offline -- this removal will be sent when you reconnect." at top of the confirmation step; Remove queues the request locally | Connectivity lost while the confirmation step is open | Connectivity restored -- the queued removal submits automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules), which defines the irreversible-action confirmation requirement enforced by the confirmation step above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-18.SPEC-004 (My Account) | -- |
| Successful removal | FEAT-18.SPEC-002 (this screen, member list state) | -- |
| "invite a member" link (Empty state) | FEAT-09.SPEC-001 (Household Invitations Manager) | FEAT-09 |

## Data Model

**Creates:** None.
**Reads:** Member Profile -- display_name, member_type, age_band (kid rows), status, for every member except the organiser.
**Updates:** None directly -- removal is performed by FEAT-18.SPEC-007.
**Deletes:** None directly -- deletion of Member Profile, Dietary Rule, and Rating records is performed by FEAT-18.SPEC-007.

## Business Rules

- The organiser is never a selectable target on this screen (XBR-15) -- she does not appear in the removable-member list.
- Removal requires the explicit affirmative confirmation defined by FEAT-18.SPEC-010; there is no one-tap removal from the list row itself.
- Only Maya (Organiser) can reach this screen and remove members, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- XBR-16: removing a member deletes their dietary rules and ratings and future plans stop accounting for them, distinct from a member's own self-leave path (FEAT-09.SPEC-005), which anonymises rather than deletes ratings.

## Edge Cases

- **Maya taps Remove on a member while another device session (Maya on a second device) is also viewing this screen** -- Both sessions read the same member list; whichever session's removal completes first wins, and the other session's stale confirmation step (if still open for the same member) is rejected with refresh: "This member was already removed." and returns to the now-updated list. This is the concurrent-edit conflict entry for this screen, consistent with the dependency map's Contention note for Member Profile.
- **Sam edits his own account (FEAT-18.SPEC-004) at the same moment Maya removes him here** -- Per the dependency map's Contention note for Member Profile, removal wins and Sam's concurrent edit is refused with a clear message on his own screen.
- **Maya taps the confirmation Remove button twice rapidly** -- The second tap is ignored while the first removal is in progress (buttons disabled during the Removing state).
- **The selected member is a young kid profile with no login** -- The loss summary and removal proceed identically to an adult member's; no login-specific behavior differs, since the kid profile itself (not a session) is what is removed.
- **Maya navigates away mid-confirmation without tapping Remove or Cancel** -- No removal occurs; returning to this screen later shows the member list with the previously-selected member still present.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Irreversible-action confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-007 (Member Removal Processing) | Triggers (outbound) | Confirmed removal starts this automation |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Navigation (outbound) | Empty-state link to invite a new member |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| member_removal_confirmation_shown | member_type (adult / kid) | Maya opens the confirmation step for a member | N/A -- no Stage 2 success metric measures removal-flow engagement; retained to observe how often the confirmation step is reached versus completed |
| member_removed | member_type (adult / kid) | Removal completes successfully | N/A -- no Stage 2 success metric measures member removal; retained so this lifecycle action's frequency is observable given its cross-feature cascade (XBR-16) |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Maya is on the Remove a Member screen, when she taps Remove on Sam's row, then the confirmation step shows Sam's name and a summary of what will be lost.

**FEAT-18.SPEC-002-AC-02:** Given Maya is on the confirmation step for Sam, when she taps "Remove Sam", then Sam is removed and she sees the toast "Sam has been removed." and returns to the member list.

**FEAT-18.SPEC-002-AC-03:** Given Maya is on the confirmation step for a kid profile, when she taps "Cancel", then no removal occurs and she returns to the member list unchanged.

**FEAT-18.SPEC-002-AC-04:** Given Maya opens the Remove a Member screen, when the list loads, then her own organiser profile never appears as a removable row.

**FEAT-18.SPEC-002-AC-05:** Given Maya's household has no members besides herself, when she opens this screen, then she sees "There's no one else to remove yet -- invite a member from Household Settings to add one."

**FEAT-18.SPEC-002-AC-06:** Given Maya confirms Sam's removal on one device while Sam is simultaneously editing his own account on another device, when Maya's removal completes first, then Sam's concurrent edit is refused with a clear message, per the dependency map's Contention note.

**FEAT-18.SPEC-002-AC-07:** Given Maya has the confirmation step open for a member that a second organiser session already removed, when she taps Remove, then she sees "This member was already removed." and returns to the updated list.

**FEAT-18.SPEC-002-AC-08:** Given Maya loses connectivity on the confirmation step and taps Remove, when she is offline, then the banner "You're offline -- this removal will be sent when you reconnect." appears and the removal queues locally.

**FEAT-18.SPEC-002-AC-09:** Given FEAT-18.SPEC-007 reports a failure while removing a member, when Maya views the confirmation step, then she sees "We couldn't remove {member_name}. Try again." with a Retry button.

**FEAT-18.SPEC-002-AC-10:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can remove a member." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-002-AC-11:** Given Maya's session expires while she is on the confirmation step, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no removal was made.

**FEAT-18.SPEC-002-AC-12:** Given Maya opens this screen, when the initial fetch of the removable-member list is in progress, then the list area shows loading placeholders instead of member rows.

**FEAT-18.SPEC-002-AC-13:** Given the initial fetch of the removable-member list fails, when Maya views this screen, then she sees "We couldn't load the household's members. Try again." with a Retry button, and tapping Retry either loads the list normally or shows the same error again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 4 | 4 |
| States | 8 (loading, load error, member list, confirmation, removing, error, empty, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

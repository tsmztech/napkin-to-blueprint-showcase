---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-005
spec_name: Leave Household
spec_slug: leave-household
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Leave Household

## Overview

**Name:** Leave Household
**ID:** FEAT-09.SPEC-005
**Type:** Screen
**Purpose:** An other adult member confirms and completes leaving the household on their own.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The confirmation step for an Other Adult Member choosing to leave the household on their own
- Triggering FEAT-09.SPEC-008 (Member Departure Processing) on confirmation
- The organiser-blocked experience when the current organiser attempts to reach this action (XBR-15)

**Non-Goals:**
- Actually anonymising ratings and soft-removing the Member Profile -- owned by FEAT-09.SPEC-008, which this screen triggers but does not perform itself
- Removing a member by the organiser's own action -- owned by FEAT-18 (Account & Data Management); this screen is exclusively the self-service leave path
- Deleting the leaving member's own account entirely (closing the account, not just membership) -- owned by FEAT-18; leaving the household and deleting one's account are distinct actions per the Access Matrix's separate Household Invitations and Account & Data columns

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Other Adult Member opens "Leave household" from settings | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member) | Full screen | Confirm leaving | -- |
| Maya (Organiser) | No | No | "Leave household" entry point is not shown to the organiser in settings; a direct navigation attempt shows "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18, per XBR-15 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles have no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Account & Data is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; no destructive action has occurred yet, so nothing needs to be preserved |

## Layout and Content

**Header:** Screen title "Leave Household" with a back arrow (returns to FEAT-01.SPEC-010).

**Body:** A confirmation panel using the same modal pattern as Organiser Hand-Over Acceptance (FEAT-09.SPEC-004): a plain statement of consequences, an explicit affirmative action, no default-confirmed state.
- "If you leave, you'll lose access to this household's plan, grocery list, and settings. Your past ratings will keep quietly influencing future meal choices, but they'll no longer be tied to your name."
- "Leave Household" button (destructive styling) and "Cancel" button

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Message and buttons stack full width, "Cancel" above "Leave Household" to avoid an accidental destructive tap being the first reachable action.
- **Medium size class and above:** Content area caps at a consistent platform-wide narrow width and is horizontally centered; buttons sit side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 | Screen closes | Standard navigation transition |
| "Cancel" button | Tap | Navigate to FEAT-01.SPEC-010, no change made | Screen closes | Standard navigation transition |
| "Leave Household" button | Tap | Opens a final confirmation dialog: "Are you sure? This can't be undone." | Dialog appears | Standard dialog transition |
| Final confirmation dialog, "Yes, Leave" | Tap | Triggers FEAT-09.SPEC-008 (Member Departure Processing) | Button shows loading state | Success: signed out of the household and routed to FEAT-01.SPEC-001 with the message "You've left the household." Failure: inline error, member remains a household member |
| Final confirmation dialog, "Stay" | Tap | Closes the dialog, no change made | Dialog closes | Returns to the confirmation panel |
| "Yes, Leave" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> message -> "Cancel" -> "Leave Household" -> (dialog) "Stay" -> "Yes, Leave".
- **Destructive-action announcement:** The final confirmation dialog is announced as a destructive, irreversible action when it opens.
- **Completion announcement:** "You've left the household." is announced on success.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Confirming | Confirmation panel as described in Layout and Content | Screen first opens | Member taps Cancel, Leave Household, or the back arrow |
| Final confirmation | Dialog: "Are you sure? This can't be undone." | Member taps "Leave Household" | Member taps "Yes, Leave" or "Stay" |
| Leaving | "Yes, Leave" button shows loading state, dialog non-dismissible | Member taps "Yes, Leave" | Departure processing completes or fails |
| Error | Inline error banner; confirmation panel remains, member is still a household member | Departure processing fails (e.g., a transient failure) | Member retries and the leave succeeds |
| Offline/Degraded | Banner: "You're offline -- leaving needs a connection." "Leave Household" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and the button re-enables |

## Validation Rules

Validation governed by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules): only an Other Adult Member may reach and complete this action; the current organiser is blocked per XBR-15.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow / Cancel tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| Successful leave | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | FEAT-01 |
| Organiser-blocked message links | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), FEAT-18 (Account & Data Management) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** The signed-in member's own Member Profile.member_type, to confirm eligibility (Other Adult Member only) before the confirmation panel renders.
**Updates:** None directly -- Member Profile.status and Rating anonymisation are performed by FEAT-09.SPEC-008.
**Deletes:** None directly.

## Business Rules

- Only an Other Adult Member (never the current organiser) can complete this action, per XBR-15; the organiser must hand over the role (FEAT-09.SPEC-003) or delete the household (FEAT-18) first.
- Leaving is final and offers no restore path: a re-invited former member onboards again as a new Member Profile rather than being restored to their prior profile or history (XBR-18, feature-overview.md Entity-Lifecycle Coverage Matrix).
- The leaving member's Dietary Rules disposition is not decided by this screen or by FEAT-09.SPEC-008 -- feature-overview.md flags this as an open product ambiguity for resolution outside this feature; this screen's confirmation message therefore covers only what this feature itself performs (ratings, membership), never implying a Dietary Rules outcome it does not control.

## Edge Cases

- **Member's role changes to organiser (accepts a hand-over) while this screen is open in another tab** -- The next action on this screen (Leave Household or the final confirmation) is rejected with "You're the organiser now -- hand over the role or delete the household to leave." since eligibility is re-checked at action time, not screen-load time.
- **Member taps "Yes, Leave" twice rapidly** -- Second tap is ignored while the first departure is in progress (dialog and button disabled).
- **Departure processing fails after the member has already been shown the final confirmation** -- The member remains a full household member; the inline error is shown and no partial departure state exists (Rating anonymisation and Member Profile removal happen atomically together in FEAT-09.SPEC-008).
- **Member loses connectivity between the final confirmation and processing completing** -- The offline banner replaces the loading state; the leave has not taken effect and the member remains a household member until connectivity returns and the action completes or is retried.
- **Only member (aside from the organiser) leaves, dropping the household to a single active adult** -- Leaving proceeds normally; the household continuing with only the organiser is a valid state (feature-overview.md, Household holds 1-12 members with no stated minimum beyond the one required organiser).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-008 (Member Departure Processing) | Triggers (outbound) | "Yes, Leave" fires ratings anonymisation and membership removal |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Other Adult Members, blocks the organiser per XBR-15 |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; Cancel and back arrow return there |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (outbound) | Destination after a successful leave |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), FEAT-18 (Account & Data Management) | Navigation (outbound) | Organiser-blocked message's alternative paths |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_left_household_confirmed | -- | Member completes the final confirmation and departure processing succeeds | N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; a departure is the inverse signal and is not itself a positive contribution to that target, so no Stage 2 metric is fed directly by this event; it is retained to explain drops in household participation when reading that metric |
| leave_blocked_organiser | -- | The organiser attempts to reach this action and is blocked | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given Sam (Other Adult Member) opens this screen, when it loads, then he sees the confirmation panel explaining that ratings become anonymous influence and access is lost.

**FEAT-09.SPEC-005-AC-02:** Given Sam taps "Leave Household" and then "Yes, Leave" on the final confirmation, when departure processing completes, then he is signed out and routed to FEAT-01.SPEC-001 with "You've left the household."

**FEAT-09.SPEC-005-AC-03:** Given Sam taps "Leave Household", when the final confirmation dialog appears, then it reads "Are you sure? This can't be undone." with "Yes, Leave" and "Stay" options and no default-confirmed state.

**FEAT-09.SPEC-005-AC-04:** Given Maya (Organiser) attempts to navigate directly to this screen, when the request is made, then she sees "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18.

**FEAT-09.SPEC-005-AC-05:** Given Sam taps "Cancel" on the confirmation panel, when the tap registers, then he returns to FEAT-01.SPEC-010 with no change to his membership.

**FEAT-09.SPEC-005-AC-06:** Given Sam accepts a hand-over and becomes organiser in another tab while this screen is open, when he then taps "Leave Household", then the action is rejected with "You're the organiser now -- hand over the role or delete the household to leave."

**FEAT-09.SPEC-005-AC-07:** Given Sam loses connectivity on the confirmation panel, when he attempts to tap "Leave Household", then the banner "You're offline -- leaving needs a connection." appears and the button is disabled.

**FEAT-09.SPEC-005-AC-08:** Given Sam's departure processing fails after he confirms, when the failure occurs, then he remains a full household member and an inline error is shown with no partial departure state.

**FEAT-09.SPEC-005-AC-09:** Given Sam taps "Yes, Leave" twice rapidly on the final confirmation, when the first tap is already processing, then the second tap has no additional effect and departure processing runs exactly once.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (confirming, final confirmation, leaving, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

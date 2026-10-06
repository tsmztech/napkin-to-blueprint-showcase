---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-003
spec_name: Organiser Hand-Over Initiation
spec_slug: organiser-hand-over-initiation
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Organiser Hand-Over Initiation

## Overview

**Name:** Organiser Hand-Over Initiation
**ID:** FEAT-09.SPEC-003
**Type:** Screen
**Purpose:** The organiser picks an active adult member and starts handing over the organiser role to them.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Listing every Active adult member eligible to receive the organiser role (excludes the organiser herself, excludes kid profiles, excludes Invited/Left/Removed members)
- Selecting a recipient and sending the hand-over request (triggers FEAT-09.SPEC-014, the recipient's notification)
- Showing the status of an outstanding hand-over request (pending, or the outcome of a decline)
- Cancelling an outstanding hand-over request before the recipient responds

**Non-Goals:**
- The recipient's accept/decline decision -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance); this screen only initiates and tracks the request
- Actually transferring the organiser role and swapping member types -- owned by FEAT-09.SPEC-009 (Organiser Hand-Over Processing), which runs only after the recipient accepts
- Household deletion as an alternative to handing over -- owned by FEAT-18 (Account & Data Management); this screen offers hand-over only, per XBR-15's two paths for an organiser who wants to leave or delete their own account

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens "Hand over organiser role" from settings | None |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Recipient declines the hand-over | The declined recipient's name, so the organiser sees the outcome and can pick someone else |
| FEAT-18.SPEC-004 (My Account) | Organiser taps "Hand over role" on the Deletion blocked dialog after attempting to delete her own account while still organiser (XBR-15) | A prompt explaining she must hand over the role first, linking here |
| FEAT-09.SPEC-005 (Leave Household) | Organiser attempts to leave while still organiser and follows the organiser-blocked message link | A prompt explaining she must hand over the role first |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Select a recipient, send the request, cancel a pending request | -- |
| Sam (Other Adult Member) | No | No | Screen entry point is not shown in household settings; a direct navigation attempt shows "Only the organiser can hand over the role" and returns Sam to FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles have no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; a selected-but-not-yet-sent recipient choice is preserved and restored after re-authentication |

## Layout and Content

**Header:** Screen title "Hand Over Organiser Role" with a back arrow (returns to FEAT-01.SPEC-010).

**Body -- no pending request:**
- A brief explanation: "Handing over makes {selected name} the organiser. You'll keep full access as an Other Adult Member -- they'll take over planning, budget, and settings."
- A list of eligible Active adult members (radio selection, one at a time)
- "Send Request" button, enabled once a recipient is selected

**Body -- pending request outstanding:**
- Status line: "Waiting for {recipient name} to accept." with the date sent
- "Cancel Request" button

**Body -- request declined (organiser's next visit after a decline):**
- Status line: "{recipient name} declined. You're still the organiser." with a "Choose Someone Else" button that returns to the no-pending-request body

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Recipient list and body stack full width, single column.
- **Medium size class and above:** Content area caps at a consistent platform-wide width and is horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 | Screen closes | Standard navigation transition |
| Recipient radio row | Tap | Selects that member as the candidate recipient | Selected row highlighted, "Send Request" enabled | Standard selection state |
| "Send Request" button | Tap | 1. Validate recipient eligibility via FEAT-09.SPEC-010. 2. If eligible, record the pending hand-over request and trigger FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification). | Button shows loading state; on success the body switches to the pending-request state | Success: "Request sent to {recipient name}." Failure: inline error, selection preserved |
| "Cancel Request" button (pending state) | Tap | Opens a confirmation dialog: "Cancel this hand-over request?" | Dialog appears | Confirm clears the pending request and returns to the no-pending-request body with the toast "Request cancelled"; Cancel closes the dialog with no change |
| "Choose Someone Else" button (declined state) | Tap | Clears the declined-request status | Body switches to the no-pending-request state | Immediate transition |
| "Send Request" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> recipient list (radio group) -> "Send Request", or -> "Cancel Request" / "Choose Someone Else" depending on state.
- **Validation announcements:** A failed send is announced along with its inline error text.
- **State-change announcements:** The switch between no-pending, pending, and declined bodies is announced so the outcome is not silently missed.
- **Keyboard alternatives:** The recipient list is a standard radio group, fully keyboard-operable; every action is reachable by keyboard.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| No eligible recipients | Message: "You need at least one other active adult member to hand over the role. Invite someone first." with a link to FEAT-09.SPEC-001 | Household has zero Active Other Adult Members | A member becomes Active (accepts an invitation) |
| No pending request | Recipient list with "Send Request" | No hand-over request is currently outstanding and none was just declined | Organiser sends a request |
| Sending | "Send Request" button shows loading state, list disabled | Organiser taps Send Request with a valid selection | Send completes or fails |
| Pending | Status line and "Cancel Request" as described in Layout and Content | A hand-over request has been sent and not yet accepted, declined, or cancelled | Recipient accepts (screen leaves this feature area), declines (switches to Declined), or organiser cancels (returns to No pending request) |
| Declined | Status line and "Choose Someone Else" as described in Layout and Content | The recipient declined the most recent request | Organiser taps "Choose Someone Else" |
| Send error | Inline error banner; recipient selection preserved | Sending the request fails for a reason other than eligibility (e.g., a transient failure) | Organiser retries and the send succeeds |
| Offline/Degraded | Banner: "You're offline -- sending a hand-over request needs a connection." Recipient selection remains but "Send Request" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and the button re-enables |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules), specifically the hand-over-recipient eligibility rule (must be an Active adult member, not the current organiser).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| "No eligible recipients" link | FEAT-09.SPEC-001 (Household Invitations Manager) | -- |

## Data Model

**Creates:** Household -- pending_organiser_handover (a workflow-only marker this feature introduces to track an outstanding request: the requested recipient's Member Profile reference and the request date; it is not one of the dependency map's listed Household fields because no other feature reads or writes it, but it does not conflict with any listed field).
**Reads:** Household.organiser (to exclude the current organiser from the recipient list) and the household's Member Profile list, filtered to Active adult members (Other Adult Member type).
**Updates:** Household -- pending_organiser_handover is cleared on cancel or decline; it is cleared (and Household.organiser transferred) by FEAT-09.SPEC-009 on acceptance.
**Deletes:** None.

## Business Rules

- Only one hand-over request may be outstanding at a time (XBR-15's "exactly one organiser" invariant): the recipient list and "Send Request" are only reachable from the No pending request state.
- A recipient must be an Active adult member (Other Adult Member type) who has not left or been removed; kid profiles of any kind are never eligible (feature-overview.md, Validation & Limits).
- Cancelling a pending request does not notify the recipient beyond removing their pending action from FEAT-09.SPEC-004 -- there is no cancellation notification, since the recipient simply no longer has anything to act on.
- This screen is one of the two paths XBR-15 requires before an organiser may leave or delete her own account; the other is household deletion through FEAT-18.

## Edge Cases

- **Recipient's status changes to Left or Removed while a request is pending** -- The pending request is silently cancelled and the screen returns to the No pending request state with the note "{recipient name} is no longer an active member; the request was cancelled."
- **Organiser cancels the request at the same moment the recipient accepts it** -- Reject-with-refresh: whichever change is recorded first wins. If the accept was recorded first, the cancel is rejected and the screen shows the hand-over as completed (organiser role already transferred, this screen no longer applies to her). If the cancel was recorded first, the recipient's accept attempt fails per FEAT-09.SPEC-004's own edge case.
- **Two eligible recipients exist and the organiser selects one while another is added mid-session** -- The recipient list is a snapshot loaded on screen entry; a newly Active member does not appear until the organiser reopens the screen, with no effect on an already-selected candidate.
- **Organiser taps "Send Request" twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **Household has exactly one eligible recipient and they decline** -- The Declined state is shown exactly as with any recipient; "Choose Someone Else" returns to a recipient list that, if still only one eligible member exists, shows that same person available to re-select.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Hand-over-recipient eligibility |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Maya (Organiser) |
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Triggers (outbound) | "Send Request" fires the recipient's notification |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (inbound) | A decline returns the organiser here with the outcome |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | Affects (inbound) | An acceptance elsewhere transfers the role, ending this screen's relevance to the former organiser |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; back arrow returns there |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | XBR-15's own-account deletion gate (Deletion blocked state) links here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric | 
|-------|-----------|--------------|-----------------|
| organiser_handover_requested | -- | Organiser sends a hand-over request | N/A -- no Stage 2 metric measures hand-over activity; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer |
| organiser_handover_cancelled | reason (organiser_cancelled / recipient_no_longer_eligible) | A pending request is cleared without a decision | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Maya has one Active Other Adult Member (Sam), when she opens this screen, then Sam appears as the sole eligible recipient and "Send Request" is disabled until she selects him.

**FEAT-09.SPEC-003-AC-02:** Given Maya selects Sam and taps "Send Request", when the send succeeds, then the screen switches to "Waiting for Sam to accept." and Sam receives the notification (FEAT-09.SPEC-014).

**FEAT-09.SPEC-003-AC-03:** Given Maya has a pending request to Sam, when she taps "Cancel Request" and confirms, then the request clears, the screen returns to the recipient list, and the toast "Request cancelled" appears.

**FEAT-09.SPEC-003-AC-04:** Given Sam declines Maya's hand-over request, when Maya next opens this screen, then she sees "Sam declined. You're still the organiser." with a "Choose Someone Else" button.

**FEAT-09.SPEC-003-AC-05:** Given Sam is removed or leaves while Maya's request to him is pending, when the change is recorded, then the request is silently cancelled and Maya sees "Sam is no longer an active member; the request was cancelled."

**FEAT-09.SPEC-003-AC-06:** Given Maya's household has no Active Other Adult Members, when she opens this screen, then she sees "You need at least one other active adult member to hand over the role. Invite someone first." with a link to FEAT-09.SPEC-001.

**FEAT-09.SPEC-003-AC-07:** Given Sam attempts to navigate directly to this screen, when the request is made, then he sees "Only the organiser can hand over the role" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-003-AC-08:** Given Maya cancels her pending request at the same moment Sam accepts it and Sam's acceptance is recorded first, when Maya's cancel is processed, then it is rejected and the screen reflects that the organiser role has already transferred.

**FEAT-09.SPEC-003-AC-09:** Given Maya loses connectivity while a recipient is selected, when she attempts to tap "Send Request", then the banner "You're offline -- sending a hand-over request needs a connection." appears and the button is disabled.

**FEAT-09.SPEC-003-AC-10:** Given Maya taps "Send Request" twice rapidly, when the first tap is already processing, then the second tap has no additional effect and only one request is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (no eligible recipients, no pending, sending, pending, declined, send error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

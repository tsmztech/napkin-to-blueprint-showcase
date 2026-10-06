---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-004
spec_name: Organiser Hand-Over Acceptance
spec_slug: organiser-hand-over-acceptance
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Organiser Hand-Over Acceptance

## Overview

**Name:** Organiser Hand-Over Acceptance
**ID:** FEAT-09.SPEC-004
**Type:** Screen
**Purpose:** The chosen adult member accepts or declines becoming the new organiser.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Showing the outstanding hand-over request addressed to the signed-in member
- Accepting the request, which triggers FEAT-09.SPEC-009 (Organiser Hand-Over Processing)
- Declining the request, which returns the organiser to FEAT-09.SPEC-003 with the outcome

**Non-Goals:**
- Selecting who receives a hand-over request -- owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation)
- Performing the actual role transfer and member-type swap -- owned by FEAT-09.SPEC-009, which this screen triggers on accept but does not perform itself
- Cancelling a request from the organiser's side -- owned by FEAT-09.SPEC-003; this screen only offers the recipient's own accept/decline choice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Recipient opens the notification | The pending hand-over request addressed to them |
| FEAT-01.SPEC-010 (Household Settings Hub) | Recipient has a pending request and taps the pending hand-over entry ("{organiser display name} wants to make you the organiser") | Same pending request |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member, addressed recipient) | Full screen | Accept or decline | -- |
| Sam (Other Adult Member, not the addressed recipient -- e.g., a future household with more than one other adult) | No | No | N/A -- there is at most one outstanding hand-over request per household (XBR-15); a member with no pending request addressed to them sees no entry point |
| Maya (Organiser) | No | No | The organiser tracks her own request through FEAT-09.SPEC-003, not this screen; opening this screen while still organiser (e.g., stale link) shows "You're the organiser -- there's nothing to accept here." |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles are never eligible hand-over recipients |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the pending request is unaffected and re-shown after re-authentication |

## Layout and Content

**Header:** Screen title "Become the Organiser?" with no back navigation (this is a decision screen reached from a notification or a settings entry, not a browsing flow).

**Body:**
- Plain statement of what accepting means, matching the same confirmation-required modal pattern as Leave Household (FEAT-09.SPEC-005): "{organiser display name} wants to make you the organiser. You'll take over the weekly budget, schedule, and household settings. {organiser display name} keeps full access as an Other Adult Member."
- "Accept" and "Decline" buttons, no default-confirmed state

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Message and buttons stack full width, "Accept" above "Decline."
- **Medium size class and above:** Content area caps at a consistent platform-wide narrow width and is horizontally centered; buttons sit side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Accept" button | Tap | 1. Re-check the request is still outstanding via FEAT-09.SPEC-010. 2. If still outstanding, trigger FEAT-09.SPEC-009 (Organiser Hand-Over Processing). | Button shows loading state | Success: screen transitions to the Success state with "You're now the organiser." confirmation. Failure (request no longer outstanding): switches to the withdrawn-request message |
| "Decline" button | Tap | Opens a confirmation dialog: "Decline becoming the organiser? {organiser display name} will keep the role." | Dialog appears | Confirm clears the request and navigates back to FEAT-01.SPEC-010 with the toast "You declined."; Cancel closes the dialog with no change |
| "Accept" / "Decline" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Go to Household Settings" button (Success state) | Tap | Navigates to FEAT-01.SPEC-010 with organiser-level access | Screen unmounts | Household Settings Hub loads with the new organiser's elevated access |
| "Review your account details" link (Success state) | Tap | Navigates to FEAT-18.SPEC-004 (My Account), now showing the organiser-only Household Data & Deletion section | Screen unmounts | My Account loads the new organiser's own profile, with the organiser-only section now visible |

### Accessibility Notes

- **Focus order:** Message, then "Accept", then "Decline". On the Success state, focus moves to the confirmation message, then "Go to Household Settings", then "Review your account details".
- **Confirmation announcements:** "You're now the organiser." and "You declined." are announced when they appear.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Pending decision | Message and Accept/Decline buttons as described | Screen loads with a request still outstanding, addressed to this member | Member taps Accept or Decline |
| Accepting | "Accept" button shows loading state, both buttons disabled | Member taps Accept | Processing completes or the request is found no longer outstanding |
| Success | "You're now the organiser." confirmation, with "Go to Household Settings" and "Review your account details" buttons | FEAT-09.SPEC-009 (Organiser Hand-Over Processing) completes the role transfer | Member taps "Go to Household Settings" (routes to FEAT-01.SPEC-010) or "Review your account details" (routes to FEAT-18.SPEC-004) |
| Declining | Confirmation dialog shown | Member taps Decline | Member confirms or cancels the dialog |
| Withdrawn | Message: "This request is no longer available -- {organiser display name} may have cancelled it or the role has already changed hands." with a single "Back to Household" button | The request is found no longer outstanding when Accept is attempted, or the screen is opened after the request already resolved | Member taps "Back to Household" |
| Error | Inline error banner; Pending decision body remains | Accept processing fails for a reason other than the request being withdrawn (e.g., a transient failure) | Member retries and processing succeeds |
| Offline/Degraded | Banner: "You're offline -- accepting or declining needs a connection." Buttons disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and buttons re-enable |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) for whether the hand-over request is still outstanding at the moment of accept or decline.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Go to Household Settings" (Success state) | FEAT-01.SPEC-010 (Household Settings Hub), now with organiser access | FEAT-01 |
| "Review your account details" (Success state) | FEAT-18.SPEC-004 (My Account), now showing the organiser-only section | FEAT-18 |
| Decline confirmed | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| "Back to Household" (withdrawn state) | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |

## Data Model

**Creates:** None directly -- Household.organiser and both members' member_type are updated by FEAT-09.SPEC-009 on accept.
**Reads:** Household -- pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003) to confirm the request addressed to this member and to display the current organiser's display name.
**Updates:** Household -- pending_organiser_handover is cleared on decline (processing owns clearing it on accept, per FEAT-09.SPEC-009).
**Deletes:** None.

## Business Rules

- Only the addressed recipient can act on a given request; there is never more than one outstanding hand-over request per household (XBR-15).
- Declining leaves the organiser role unchanged and does not notify the organiser through a separate Notification spec -- the outcome surfaces inline the next time she opens FEAT-09.SPEC-003 (feature-overview.md, Side-Effect Inventory).
- Accepting is the sole trigger for FEAT-09.SPEC-009's atomic role transfer; no partial state exists between accept and the transfer completing.

## Edge Cases

- **Organiser cancels the request at the same moment the recipient taps Accept** -- Reject-with-refresh: whichever change is recorded first wins. If the cancel is recorded first, the accept fails and the screen switches to the Withdrawn state; if the accept is recorded first, it proceeds normally regardless of the near-simultaneous cancel attempt.
- **Recipient's own account changes state while the request is pending (e.g., they leave the household through another session)** -- The request becomes void along with their membership; opening this screen shows the Withdrawn state, since a departed member cannot become organiser.
- **Recipient taps Accept and Decline in quick succession (double interaction)** -- The first action to register is processed; the second is ignored once the button/dialog for the first action is in progress.
- **Recipient opens the screen after the request already resolved (stale notification link)** -- Withdrawn state, with the specific outcome unknown to this screen (it does not distinguish "already accepted by someone else" from "cancelled," since at most one recipient and one outcome exist at a time -- the message stays generic).
- **Recipient loses connectivity mid-accept** -- The offline banner appears and Accept/Decline are disabled; no partial role transfer occurs, since FEAT-09.SPEC-009 only runs after a successfully recorded accept.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Navigation (inbound) | The notification's CTA opens this screen |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | References (inbound), Navigation (outbound) | Declining reports the outcome there |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | Triggers (outbound) | Accept fires the atomic role transfer |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the request is still outstanding |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | Destination after accept, decline, or withdrawal |
| FEAT-18.SPEC-004 (My Account) | Navigation (outbound) | The Success state offers a link back to the new organiser's own account details |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| organiser_handover_accepted | -- | Recipient's accept is recorded | N/A -- no Stage 2 metric measures hand-over outcomes; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer |
| organiser_handover_declined | -- | Recipient's decline is recorded | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given Sam has a pending hand-over request from Maya, when he opens this screen, then he sees "Maya wants to make you the organiser..." with "Accept" and "Decline" buttons.

**FEAT-09.SPEC-004-AC-02:** Given Sam taps "Accept", when the acceptance is processed, then FEAT-09.SPEC-009 transfers the organiser role and Sam sees the Success state with "You're now the organiser." along with "Go to Household Settings" and "Review your account details" buttons.

**FEAT-09.SPEC-004-AC-03:** Given Sam taps "Decline" and confirms, when the decline is recorded, then he is returned to FEAT-01.SPEC-010 with the toast "You declined." and Maya remains the organiser.

**FEAT-09.SPEC-004-AC-04:** Given Maya cancels the request at the same moment Sam taps Accept, and the cancel is recorded first, when Sam's accept is processed, then it fails and Sam sees the Withdrawn state.

**FEAT-09.SPEC-004-AC-05:** Given Sam opens this screen after the request has already resolved, when the screen loads, then he sees "This request is no longer available -- Maya may have cancelled it or the role has already changed hands." with a "Back to Household" button.

**FEAT-09.SPEC-004-AC-06:** Given Maya (still organiser) opens this screen directly via a stale or mistaken link, when the screen loads, then she sees "You're the organiser -- there's nothing to accept here."

**FEAT-09.SPEC-004-AC-07:** Given Sam loses connectivity while viewing the pending request, when he attempts to tap "Accept" or "Decline", then the banner "You're offline -- accepting or declining needs a connection." appears and both buttons are disabled.

**FEAT-09.SPEC-004-AC-08:** Given Sam leaves the household from another session while this request is pending to him, when he (or anyone) opens this screen for that request, then it shows the Withdrawn state.

**FEAT-09.SPEC-004-AC-09:** Given Sam taps "Accept" and then rapidly taps "Decline" before the accept completes, when the accept is already processing, then the decline tap has no effect and only the accept outcome applies.

**FEAT-09.SPEC-004-AC-10:** Given Sam is on the Success state after accepting the organiser role, when he taps "Review your account details", then he is navigated to FEAT-18.SPEC-004 (My Account), which loads his own profile now showing the organiser-only Household Data & Deletion section.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (pending, accepting, declining, success, withdrawn, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

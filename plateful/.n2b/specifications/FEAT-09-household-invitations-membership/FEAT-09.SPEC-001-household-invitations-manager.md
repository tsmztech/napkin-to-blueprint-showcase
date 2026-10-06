---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-001
spec_name: Household Invitations Manager
spec_slug: household-invitations-manager
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Household Invitations Manager

## Overview

**Name:** Household Invitations Manager
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** The organiser sends a new invitation and sees, resends, or revokes every outstanding and accepted invitation for the household.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Sending a new invitation by entering a contact detail
- Listing every outstanding (Sent), Accepted, Revoked, and Expired invitation for the household
- Revoking a Sent invitation
- Resending (re-creating) an Expired invitation
- Displaying the generated shareable invitation link/message for the organiser to send through her own channel of choice

**Non-Goals:**
- Accepting an invitation -- handled by FEAT-09.SPEC-002 (Invitation Acceptance); this screen is organiser-only
- Automatically delivering the invitation by email or text on the product's behalf -- excluded per scope-boundaries.md SC-14 (no in-product messaging layer) and the feature's zero Integration-spec footprint (feature-dependency-map.md, External Touchpoints): the product generates a unique shareable link and the organiser sends it herself, exactly as household-to-household referral links work in FEAT-24
- Automatically expiring a Sent invitation -- owned by FEAT-09.SPEC-006 (Invitation Expiry); this screen only reflects the resulting status
- Organiser hand-over -- a distinct action owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), reached from elsewhere in household settings, not from this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Invite a partner" during guided setup, step 5 | None -- screen opens directly into the send-invitation form, empty invitation list |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens "Household Invitations" from settings | None -- screen opens to the invitation list |
| Default entry | Organiser navigates to this feature area directly | None |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Organiser taps the "No eligible recipients" link | None -- screen opens to the invitation list so she can invite an adult first |
| FEAT-18.SPEC-002 (Remove Member Profile) | Organiser taps the "invite a member" link from that screen's empty state | None -- screen opens to the invitation list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Send, resend, and revoke invitations | -- |
| Sam (Other Adult Member) | No | No | Screen entry point is not shown in household settings; a direct navigation attempt shows "Only the organiser can manage invitations" and returns Sam to FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles have no login and no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14); this screen has no operator path |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In); no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress, unsent invitation draft is preserved and restored to this screen after re-authentication |

## Layout and Content

**Header:** Screen title "Household Invitations" with a back arrow (returns to FEAT-01.SPEC-010) and a "+ Invite" action button (top right).

**Body:**
- **Send Invitation panel** (shown expanded when the list is empty, or opened by "+ Invite" otherwise): a single text field labeled "Contact detail (email or phone)" and a "Send Invitation" button. This is a label for the organiser's own reference and for re-invite blocking (FEAT-09.SPEC-010) -- the product does not deliver anything to this contact detail itself.
- **Invitation List**: one row per invitation, most recent first, each showing:
  - The contact detail entered for that invitation
  - A status badge: Sent, Accepted, Revoked, or Expired
  - The date sent
  - Row actions, shown per status: Sent shows "Revoke" and "Copy link"; Expired shows "Resend"; Accepted and Revoked show no actions (history only)
- **Shareable link confirmation** (appears immediately after a successful send): a read-only field containing the generated invitation link and a "Copy link" button, with the text "Share this with {contact_detail} yourself -- we don't send it for you."

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Send Invitation panel and list stack full width, single column; row actions collapse into a per-row overflow menu.
- **Medium size class and above:** Content area caps at a consistent platform-wide width and is horizontally centered; row actions remain inline (no overflow menu needed); no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard navigation transition |
| "+ Invite" button | Tap | Opens the Send Invitation panel | Panel expands | Panel animates open, focus moves to the contact detail field |
| Contact detail field | Type | Captures text input | Field shows entered text | Standard input focus state |
| Contact detail field | Blur | Validates via FEAT-09.SPEC-010 | Error state if invalid | "Enter an email address or phone number" below the field |
| "Send Invitation" button | Tap | 1. Validate contact detail and re-invite blocking via FEAT-09.SPEC-010. 2. If valid, create the Invitation (status: Sent) and generate its shareable link. | Button shows loading state; on success the new invitation appears at the top of the list | Success (within a couple of seconds): "Invitation ready to share" confirmation with the Shareable link confirmation panel. Failure: inline error, entered contact detail preserved, "Try again" retry offered |
| "Copy link" button (per Sent row, and in the send confirmation) | Tap | Copies the invitation's shareable link to the clipboard | None | "Link copied" toast |
| "Revoke" button (Sent rows only) | Tap | Opens a confirmation dialog: "Revoke this invitation? {contact_detail} will no longer be able to join with this link." | Dialog appears | Confirm triggers the revoke (via FEAT-09.SPEC-010); Cancel closes with no change |
| Revoke confirmation | Tap "Revoke" | Sets the invitation's status to Revoked via FEAT-09.SPEC-010 | Row status badge updates to Revoked, row actions clear | Toast: "Invitation revoked" |
| "Resend" button (Expired rows only) | Tap | Creates a fresh Sent invitation to the same contact detail via FEAT-09.SPEC-010, replacing the Expired one at the top of the list | New Sent row appears; the prior Expired row remains as history | Toast: "New invitation ready to share" with the Shareable link confirmation panel |
| "Send Invitation" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> "+ Invite" -> contact detail field (when panel open) -> "Send Invitation" -> invitation list rows, each row's actions in reading order.
- **Validation announcements:** Field errors are announced to assistive technology and programmatically associated with the contact detail field.
- **Dynamic announcements:** "Invitation ready to share," "Link copied," "Invitation revoked," and "New invitation ready to share" are all announced as they appear.
- **Keyboard alternatives:** Every action, including Copy link, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Plain prompt: "No invitations yet -- invite someone to share the plan and list with you." Send Invitation panel shown expanded by default. | Household has zero invitations of any status | An invitation is sent |
| Loaded (with invitations) | Invitation List populated, Send Invitation panel collapsed behind "+ Invite" | One or more invitations exist | Always the resting state once invitations exist |
| Loading (list) | Skeleton rows in place of the invitation list | Screen first opens, list is being fetched | List loads successfully or fails |
| List load error | Error banner: "Couldn't load invitations. Try again." with a Retry button | Fetching the invitation list fails | User taps Retry and the fetch succeeds |
| Sending | "Send Invitation" button shows loading state, contact detail field disabled | User taps Send Invitation with valid input | Send completes or fails |
| Send error | Inline error banner above the form; entered contact detail preserved; "Try again" retry offered | Sending the invitation fails | User taps Retry and the send succeeds, or corrects the contact detail |
| Offline/Degraded | Banner: "You're offline -- this invitation will send once you're back online." The contact detail field remains editable; tapping Send queues the invitation locally rather than failing | Connectivity lost while composing an invitation | Connectivity restored -- queued invitation sends automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules). This screen applies validation on field blur and on Send Invitation submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| Successful send, resend, revoke | Stays on this screen (list updates in place) | -- |

## Data Model

**Creates:** Invitation -- contact_detail (organiser input), status (set to Sent), sent_by (the current organiser, Maya). A shareable link is generated for the new record.
**Reads:** Invitation -- all fields, filtered to this household, for every status, ordered newest first. Household.organiser and the household's active Member Profile list (read to run the re-invite-blocking check in FEAT-09.SPEC-010).
**Updates:** Invitation -- status (Sent to Revoked on revoke).
**Deletes:** None -- invitations are never hard-deleted (feature-overview.md, Entity-Lifecycle Coverage Matrix; assumptions-constraints.md ASMP-24).

## Business Rules

- Contact-detail requirement, format, and re-invite blocking are governed by FEAT-09.SPEC-010 -- this screen never restates them.
- Only Maya (Organiser) reaches this screen at all, per FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules).
- Resending an Expired invitation creates a new Sent invitation to the same contact detail rather than reviving the expired record; the expired record remains as household history (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- A household may hold multiple outstanding (Sent) invitations at once (feature-overview.md, Validation & Limits).

## Edge Cases

- **Organiser revokes an invitation that was accepted moments earlier on another device** -- Revoke is rejected with refresh: the row updates to show Accepted and the toast "This invitation was already accepted" appears instead of "Invitation revoked." Resolution: reject-with-refresh, per the dependency map's Contention note for the Invitation entity.
- **Organiser resends an invitation that expired moments ago vs. one that is still Sent** -- "Resend" is shown only for rows already in Expired status; if the underlying invitation is still Sent when tapped (stale row), the action is rejected with refresh and the row updates to its current status.
- **Organiser taps Revoke twice rapidly** -- Second tap is ignored while the first revoke is in progress (dialog and button disabled).
- **Organiser sends an invitation to a contact detail matching an existing active member** -- Blocked per FEAT-09.SPEC-010 with the error "This person is already a member of your household."
- **Organiser navigates away with the Send Invitation panel open and unsent text** -- No confirmation dialog; the draft contact detail is not destructive state and is simply discarded, consistent with the screen's non-blocking send flow.
- **Two devices signed in as Maya both viewing this screen** -- The list is a snapshot refreshed on entry and after each action on this screen; an invitation sent, revoked, or resent on one device is not live-pushed to the other, so the other device's next action against a changed row hits the reject-with-refresh path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Contact-detail validation, re-invite blocking, and the accept/revoke/expiry race resolution |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Maya (Organiser) |
| FEAT-09.SPEC-006 (Invitation Expiry) | Affects (inbound) | Automatically transitions Sent rows to Expired, shown on this screen |
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Affects (inbound) | Transitions a row to Accepted, shown on this screen |
| FEAT-09.SPEC-002 (Invitation Acceptance) | References (outbound) | The shareable link this screen generates opens that screen for the invitee |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound) | Guided setup step 5 opens directly into this screen's send form |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; back arrow returns there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invitation_sent | origin (new / resend), entry source (guided setup / settings) | An invitation is successfully created (new or resend) | N/A -- no Stage 2 metric measures the invitation-sending step itself; the connected metric, "Household Member Participation," measures the invited adult's subsequent joining and use, tracked at acceptance (FEAT-09.SPEC-007) |
| invitation_revoked | time outstanding before revoke | Organiser confirms a revoke | N/A -- no Stage 2 metric measures revocations; retained as a funnel-diagnostic signal |
| invitation_send_failed | failure reason | A send attempt fails | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Maya is on the Household Invitations Manager with no existing invitations, when the screen loads, then she sees the empty-state prompt "No invitations yet -- invite someone to share the plan and list with you." with the Send Invitation panel already expanded.

**FEAT-09.SPEC-001-AC-02:** Given Maya enters a valid email address and taps "Send Invitation", when the send succeeds, then a new Sent row appears at the top of the list within a couple of seconds and the Shareable link confirmation panel appears with a "Copy link" button.

**FEAT-09.SPEC-001-AC-03:** Given Maya enters a contact detail matching an active member of her household, when she taps "Send Invitation", then the error "This person is already a member of your household." appears and no invitation is created.

**FEAT-09.SPEC-001-AC-04:** Given Maya taps "Revoke" on a Sent invitation and confirms, when the revoke completes, then the row's status badge updates to Revoked and the toast "Invitation revoked" appears.

**FEAT-09.SPEC-001-AC-05:** Given Maya sees an Expired invitation, when she taps "Resend", then a new Sent row is created for the same contact detail and the toast "New invitation ready to share" appears with a fresh shareable link.

**FEAT-09.SPEC-001-AC-06:** Given Sam attempts to navigate directly to this screen, when the request is made, then he sees "Only the organiser can manage invitations" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-001-AC-07:** Given Maya's send attempt fails due to a connectivity error, when the failure occurs, then her entered contact detail is preserved and a "Try again" retry option appears.

**FEAT-09.SPEC-001-AC-08:** Given Maya loses connectivity while composing an invitation, when she taps "Send Invitation", then the banner "You're offline -- this invitation will send once you're back online." appears and the invitation sends automatically once connectivity returns.

**FEAT-09.SPEC-001-AC-09:** Given Maya taps "Revoke" on a Sent invitation that was accepted moments earlier from another device, when the revoke request completes, then she sees "This invitation was already accepted" and the row refreshes to show Accepted status.

**FEAT-09.SPEC-001-AC-10:** Given the invitation list fails to load, when the screen opens, then the error banner "Couldn't load invitations. Try again." appears with a Retry button.

**FEAT-09.SPEC-001-AC-11:** Given Maya taps "Copy link" on a Sent invitation's row, when the tap registers, then the invitation's shareable link is copied and the toast "Link copied" appears.

**FEAT-09.SPEC-001-AC-12:** Given Maya double-taps "Revoke" rapidly on the confirmation dialog, when the first tap is already processing, then the second tap has no additional effect and only one revoke is applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 7 (empty, loaded, loading, list load error, sending, send error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-003
spec_name: Remove Client Contact
spec_slug: remove-client-contact
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Remove Client Contact

## Overview

**Name:** Remove Client Contact
**ID:** FEAT-18.SPEC-003
**Type:** Screen
**Purpose:** Nadia confirms removing a contact, sees the last-Primary block when it applies, and is shown erasure-request framing before the removal executes.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The confirmation dialog for removing one Client Contact
- The last-Primary block, with a prompt to add or promote a replacement first
- Framing the removal as ending access and erasing personal details, for both a routine departure and an explicit erasure request
- Handing off to FEAT-18.SPEC-009 (Contact Removal & Data Erasure) once confirmed

**Non-Goals:**
- Performing the actual access revocation, data erasure, and evidence retention -- owned by FEAT-18.SPEC-009 (Contact Removal & Data Erasure); this screen only confirms intent and hands off
- Adding or promoting a replacement Primary contact -- that happens on FEAT-18.SPEC-002 (Add or Edit Client Contact), which this screen links to when the last-Primary block applies
- Restoring a removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: removal is the erasure mechanism (XBR-27), so there is no restore path from this or any other screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps "Remove" on a contact row | The selected contact's identity, role, and status |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Confirm removal, or cancel | -- |
| Owen (Client Primary Contact) | No | No | Owen has no route to this screen; removal is Nadia-only everywhere in the product (Access Matrix: Client Contact Management is Own-only for invites, not removal) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | No | No | Dana's read-only mirrored view of FEAT-18.SPEC-001 never exposes a Remove action, so this screen is unreachable during a support session (FEAT-31.SPEC-005) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- returning to this screen re-loads the contact fresh rather than preserving any prior confirmation state |

## Layout and Content

**Header:** Screen title "Remove Contact" with a back arrow (returns to FEAT-18.SPEC-001, cancelling without removing).

**Body (standard case -- not the client's last Primary):** A confirmation dialog:
- The contact's name, email, and role
- Body text: "Removing {contact_name} ends their access immediately and erases their email address and other contact details. Any approvals or acceptances they gave stay on the record under their name only, as evidence of what was agreed."
- A checkbox or acknowledgement: "I understand this cannot be undone."
- "Remove Contact" action (destructive style) and "Cancel"

**Body (blocked case -- this is the client's last Primary):** A blocking message replaces the confirmation:
- "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them."
- "Go to Contacts" action, linking back to FEAT-18.SPEC-001 (from which Nadia can open FEAT-18.SPEC-002 to designate a replacement)
- No "Remove Contact" action is shown in the blocked case.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Confirmation content stacks in a single column, full width; actions stack vertically with "Remove Contact" or "Go to Contacts" above "Cancel".
- **Medium size class and above:** Content is capped at a consistent platform-wide dialog width and horizontally centered; actions display side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-001 (Client Contact List), no change made | Screen closes | Standard navigation transition |
| "I understand this cannot be undone" acknowledgement | Check | Enables the "Remove Contact" action | "Remove Contact" becomes enabled | Standard checkbox state |
| "Remove Contact" action | Tap | 1. Re-check the last-Primary condition (FEAT-18.SPEC-006) at the moment of commit. 2. If still eligible, trigger FEAT-18.SPEC-009 (Contact Removal & Data Erasure). | Button shows loading state | Success: toast "Contact removed" and navigate to FEAT-18.SPEC-001. Failure (became the last Primary since load): the screen switches to the blocked case in place. |
| "Cancel" action | Tap | Navigate to FEAT-18.SPEC-001, no change made | Screen closes | Standard navigation transition |
| "Go to Contacts" action (blocked case) | Tap | Navigate to FEAT-18.SPEC-001 | Screen closes | Standard navigation transition |
| "Retry" button (load-failure banner) | Tap | Re-reads the contact and re-derives whether it is the client's sole Primary | Screen returns to Loading, then to the standard or blocked case | Loading placeholder, then the resolved case; on repeated failure the same banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> confirmation body text -> acknowledgement checkbox (standard case) -> "Remove Contact" / "Go to Contacts" -> "Cancel".
- **Dynamic updates:** A switch from the standard case to the blocked case (because the contact became the last Primary between load and commit) is announced to assistive technology as a content change, and focus moves to the blocking message.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton placeholder for the contact summary; no "Remove Contact" or "Go to Contacts" action shown, so neither case is presented before it is known | Screen opens and the contact and its sole-Primary status are being read | Read completes (standard or blocked case) or fails (Load Error) |
| Load Error | Error banner: "Couldn't load this contact. Try again." with a Retry button; no "Remove Contact" action shown, so removal cannot be confirmed without the sole-Primary check | The read of the contact or of the client's Primary contacts fails | Retry succeeds, or Nadia taps the back arrow |
| Standard confirmation | As described in Layout and Content, standard case | Screen opens for a contact that is not currently the client's last Primary | "Remove Contact" or "Cancel" is tapped |
| Blocked (last Primary) | As described in Layout and Content, blocked case | Screen opens for the client's last Primary contact, or the removal attempt's commit-time re-check finds this contact has become the last Primary | Nadia navigates to FEAT-18.SPEC-001 |
| Removing | "Remove Contact" shows a loading spinner, actions disabled | Nadia confirms removal | Removal completes or fails |
| Error | Error banner: "Couldn't remove this contact. Try again." with a Retry button | The removal automation (FEAT-18.SPEC-009) fails for a reason other than the last-Primary block | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); a removal attempted without connectivity surfaces the Error state above | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-006 (Primary Contact Requirement Rule), which defines the last-Primary block enforced by this screen's standard-vs-blocked states. This screen defines no field-level validation of its own -- there is no input to validate, only a confirmation decision.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow / "Cancel" tap | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Successful removal | FEAT-18.SPEC-001 (Client Contact List) | -- |
| "Go to Contacts" tap (blocked case) | FEAT-18.SPEC-001 (Client Contact List) | -- |

## Data Model

**Creates:** None directly -- FEAT-18.SPEC-009 creates the resulting audit trail entry.
**Reads:** Client Contact -- name, email, role, status, for the selected contact; whether the contact is currently the client's sole Primary (derived, for the blocked-case check).
**Updates:** None directly -- FEAT-18.SPEC-009 performs the actual status change and field erasure.
**Deletes:** None directly on this screen -- the personal-detail erasure is performed by FEAT-18.SPEC-009.

## Business Rules

- The last-Primary condition (FEAT-18.SPEC-006, XBR-07) is checked both when this screen loads and again at the moment "Remove Contact" is confirmed, since the client's Primary contacts can change between the two moments.
- Removal always ends access immediately and erases contact details, whether the trigger is a routine departure or an explicit data-subject erasure request (feature-overview.md, Key Capabilities) -- this screen shows the same confirmation framing in both cases, since the product does not distinguish the two at the point of removal.
- Confirmed removal always hands off to FEAT-18.SPEC-009; there is no partial or reversible removal path.

## Edge Cases

- **The contact becomes the client's last Primary between screen load and Nadia tapping "Remove Contact"** -- The commit-time re-check finds the block condition now applies; the screen switches in place to the blocked case with no removal performed. Resolution: reject-with-refresh, per the dependency map's Contention note for the Client Contact entity.
- **Nadia taps "Remove Contact" twice rapidly** -- The second tap is ignored while the first removal is in progress (button in loading state).
- **The contact or the client's Primary contacts fail to load** -- The Load Error state shows "Couldn't load this contact. Try again." with Retry; the standard confirmation is never shown on a guess, because whether the last-Primary block applies is unknown until the read succeeds.
- **Network failure during removal** -- Error banner: "Couldn't remove this contact. Try again." with a Retry button; no partial removal occurs -- the contact's status and details are unchanged until the removal automation completes successfully.
- **The contact being removed is currently signed in to the client portal** -- The removal proceeds; FEAT-18.SPEC-009 ends the contact's access immediately, so any of their subsequent actions in an already-open portal session are rejected per FEAT-05 (Client Portal Access) rather than by this screen.
- **Nadia removes a Reviewer contact (no last-Primary concern)** -- The standard confirmation case always applies to a Reviewer, since the last-Primary block only ever governs a Primary contact.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Defines the last-Primary block this screen enforces at load and at commit |
| FEAT-18.SPEC-009 (Contact Removal & Data Erasure) | Triggers (outbound) | Confirmed removal hands off to this automation |
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Navigation (outbound, via FEAT-18.SPEC-001) | Where Nadia designates a replacement Primary when blocked |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_removal_confirmed | contact_role (primary / reviewer) | Nadia confirms removal and the commit-time check passes | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so removal activity is observable |
| contact_removal_blocked_last_primary | -- | The last-Primary block is shown, at load or at commit-time re-check | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Nadia taps "Remove" on a Reviewer contact, when the screen loads, then she sees the standard confirmation naming the contact and explaining that removal ends access and erases their email and other contact details while their approvals stay on record under their name only.

**FEAT-18.SPEC-003-AC-02:** Given Nadia is on the standard confirmation, when she checks "I understand this cannot be undone" and taps "Remove Contact", then FEAT-18.SPEC-009 is triggered and she sees "Contact removed" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-003-AC-03:** Given Nadia taps "Remove" on the client's only Primary contact, when the screen loads, then she sees the blocked case: "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them." with no "Remove Contact" action.

**FEAT-18.SPEC-003-AC-04:** Given Nadia opens the standard confirmation for a Primary contact who is not the client's last Primary, when she confirms removal, then the removal proceeds normally.

**FEAT-18.SPEC-003-AC-05:** Given Nadia has the standard confirmation open for a Primary contact and another Primary contact is removed from this client in a different session first, when Nadia taps "Remove Contact", then the commit-time re-check finds this is now the last Primary and the screen switches in place to the blocked case with no removal performed.

**FEAT-18.SPEC-003-AC-06:** Given Nadia is on the blocked case, when she taps "Go to Contacts", then she is navigated to FEAT-18.SPEC-001, from which she can add or promote a replacement Primary.

**FEAT-18.SPEC-003-AC-07:** Given Nadia has not checked the acknowledgement checkbox, when she looks at "Remove Contact", then it is disabled and cannot be tapped.

**FEAT-18.SPEC-003-AC-08:** Given Nadia taps "Remove Contact" twice in rapid succession, when the first removal is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-003-AC-09:** Given a network failure occurs during removal, when the failure is detected, then the error banner "Couldn't remove this contact. Try again." appears with a Retry button, and the contact remains unremoved.

**FEAT-18.SPEC-003-AC-10:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them.

**FEAT-18.SPEC-003-AC-11:** Given Nadia removes a contact who is currently signed in to the client portal, when the removal completes, then any subsequent action that contact attempts in the client portal is rejected per FEAT-05's access rules.

**FEAT-18.SPEC-003-AC-12:** Given Nadia taps "Remove" on a contact, when the contact and the client's Primary contacts are still being read, then she sees a loading placeholder with no "Remove Contact" action until the standard or blocked case is determined.

**FEAT-18.SPEC-003-AC-13:** Given the read of the contact or the client's Primary contacts fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no "Remove Contact" action, and when she taps Retry and the read succeeds, then the standard or blocked case is shown as appropriate.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (loading, load error, standard, blocked, removing, error, offline-degraded N/A) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

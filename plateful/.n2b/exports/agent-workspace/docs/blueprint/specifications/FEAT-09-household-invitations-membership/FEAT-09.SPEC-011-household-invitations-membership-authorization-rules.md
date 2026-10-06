---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-011
spec_name: Household Invitations & Membership Authorization Rules
spec_slug: household-invitations-membership-authorization-rules
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Household Invitations & Membership Authorization Rules

## Overview

**Name:** Household Invitations & Membership Authorization Rules
**ID:** FEAT-09.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs who can send, revoke, or hand over invitations, who can leave on their own, and the unauthorized-visitor experience, as the single authoritative home for role-gated behavior across every screen in this feature.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership
**Governed Entity:** Invitation and Member Profile (the access dimension -- which role may view or act on each, across FEAT-09.SPEC-001 through FEAT-09.SPEC-005)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines: send/revoke/resend an invitation, accept an invitation, initiate/accept/decline a hand-over, and leave the household
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an unauthorized visitor (someone with no invitation addressed to them) see across this feature's screens
- The organiser's leave/delete block per XBR-15

**Non-Goals:**
- Field-level validation and the accept/revoke/expiry race resolution -- governed by FEAT-09.SPEC-010, which is more specific to the Invitation entity's own data rules
- Authorization for features outside this one (e.g., who may edit a member's dietary rules) -- each feature owning its own entities defines its own authorization; this spec covers only the actions FEAT-09 itself defines
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley has no access of any kind to this feature's actions or data (Access Matrix: Household Invitations column is None for Riley), not even a read-only view

## Governed Entity

**Entity:** Invitation and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Invitation and Member Profile fields already defined in FEAT-09.SPEC-010) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | On screen entry and on every send/revoke/resend action |
| FEAT-09.SPEC-002 | Invitation Acceptance | On screen entry (unauthorized-visitor and already-a-member states) |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | On screen entry and on Send Request |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | On screen entry and on Accept/Decline |
| FEAT-09.SPEC-005 | Leave Household | On screen entry and on the final leave confirmation |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | On processing (re-check that acceptance is not attempted by an already-authenticated household member) |
| FEAT-09.SPEC-008 | Member Departure Processing | On processing (re-check the leaving member is not the organiser) |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | On processing (re-check the accepting member is not already the organiser) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-09.SPEC-010's scope. No field in the governed entities carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Send an invitation | Maya (Organiser) | No matching duplicate (FEAT-09.SPEC-010) | Screen entry point (FEAT-09.SPEC-001) not shown to Sam at all; a direct navigation attempt shows "Only the organiser can manage invitations" |
| Revoke a Sent invitation | Maya (Organiser) | Always, on any Sent invitation for her household | Same as above -- Sam never reaches the screen where this action lives |
| Resend an Expired invitation | Maya (Organiser) | Always, on any Expired invitation for her household | Same as above |
| View the invitation list and history | Maya (Organiser) | Always | Sam, both Jordan rows, and Riley see no entry point (Access Matrix: Household Invitations is None for all four) |
| Accept an invitation | The specific unauthorized visitor addressed by the invitation link | The invitation referenced by the link is still Sent (FEAT-09.SPEC-010) | An invitation no longer Sent shows "This invitation is no longer valid" with the specific reason; the invitation content itself is only ever shown to the holder of its own link, never to anyone else |
| Accept an invitation | Any already-authenticated household member (Maya, Sam) | Never -- an existing member has nothing to accept | "You're already a member of this household." with a link to the current plan, not the acceptance form |
| Accept an invitation | Kid profiles (young, no login; older, limited login, Later) | Never | N/A -- kid profiles are never invited through this feature (scope-boundaries.md SC-02); no kid-addressed invitation link can exist |
| Initiate a hand-over | Maya (Organiser) | At least one Active Other Adult Member exists and no hand-over request is already outstanding | Screen entry point (FEAT-09.SPEC-003) not shown to Sam; a direct navigation attempt shows "Only the organiser can hand over the role" |
| Accept or decline a hand-over | The specific Active adult member the outstanding request is addressed to | A hand-over request is outstanding and addressed to them | A member with no request addressed to them has no entry point to FEAT-09.SPEC-004; the current organiser opening it directly sees "You're the organiser -- there's nothing to accept here." |
| Leave the household on her own | Sam (Other Adult Member) | Always, for any Active Other Adult Member -- her member_type is not Organiser | -- |
| Leave the household on her own | Maya (Organiser) | Never, while she holds the organiser role (XBR-15) | Entry point (FEAT-09.SPEC-005) not shown in settings to the organiser; a direct navigation attempt shows "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18 |
| Leave the household on her own | Kid profiles (young, no login; older, limited login, Later) | Never | N/A -- kid profiles are not household members with an account of their own to leave from (Access Matrix: Household Invitations and Account & Data are None) |
| View or act on any action this feature defines | Riley (Operator, support) | Never | N/A -- Riley's access is limited entirely to the separate read-only support view (FEAT-22, XBR-14); no screen or action in this feature has any operator path, not even a read-only one |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults for Invitation fields are defined in FEAT-09.SPEC-010.

## Business Rules

- Every screen in this feature (FEAT-09.SPEC-001 through FEAT-09.SPEC-005) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household, and holding no invitation link addressed to them -- sees only a sign-in screen (FEAT-01.SPEC-001) or the specific invitation-acceptance screen for a link genuinely addressed to them (FEAT-09.SPEC-002), never any other household data, per user-persona.md's Access Matrix notes.
- A session that expires mid-flow on any screen in this feature returns the user to FEAT-01.SPEC-001 (or, for an unauthenticated invitee, remains on FEAT-09.SPEC-002 with the invitation preserved) with the message "Your session has expired. Sign in to continue."
- XBR-15 (exactly one organiser at all times) is enforced at both the screen layer (Leave Household and Organiser Hand-Over Initiation each check the acting member's current role) and the processing layer (FEAT-09.SPEC-008 and FEAT-09.SPEC-009 each re-check at the moment of processing, not only at screen entry), since a role can change between screen load and action.
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1), plus the unauthorized-visitor and unauthenticated states this feature's own screens define. No role beyond these is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to FEAT-09.SPEC-001** -- Redirected to FEAT-01.SPEC-001; no household or invitation data is ever rendered, even momentarily, during the redirect.
- **Sam attempts to bypass the hidden invitation-management controls via a direct navigation URL to FEAT-09.SPEC-001** -- The destination screen itself enforces the same authorization check on entry, showing "Only the organiser can manage invitations" regardless of how the screen was reached.
- **The organiser role changes hands mid-session (FEAT-09.SPEC-009 completes while the former organiser has FEAT-09.SPEC-001 open)** -- Her next action against an organiser-only control on that screen (e.g., attempting to revoke an invitation) is rejected with "Only the organiser can manage invitations," since authorization is evaluated at action time, not at screen-load time.
- **A visitor holds a link to an invitation addressed to someone else (guessed or shared incorrectly)** -- The invitation is shown only to whoever holds its unique link; this spec does not distinguish "wrong person" from "right person" at the authorization layer, since the link itself is the sole credential FEAT-09.SPEC-002 checks -- possession of a valid, still-Sent link is what "addressed to them" means functionally.
- **Riley's operator session somehow reaches a FEAT-09 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; no invitation or membership data is ever rendered to an operator session outside FEAT-22's own read-only support view.
- **A kid profile somehow attempts a direct action against this feature (defensive case, e.g., a stale token)** -- No login exists for a young kid profile and the older-kid login (Later) has no entitlement to this feature's actions (Access Matrix: None); no action succeeds under either identity.

## Acceptance Criteria

**FEAT-09.SPEC-011-AC-01:** Given Maya (Organiser) is signed in, when she opens FEAT-09.SPEC-001, then she has full access to send, revoke, and resend invitations.

**FEAT-09.SPEC-011-AC-02:** Given Sam (Other Adult Member) attempts to navigate directly to FEAT-09.SPEC-001, when the request is made, then he sees "Only the organiser can manage invitations" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-011-AC-03:** Given an unauthorized visitor opens a link to an invitation addressed to them that is still Sent, when FEAT-09.SPEC-002 loads, then they can view and accept it.

**FEAT-09.SPEC-011-AC-04:** Given Sam (already a household member) opens any invitation link, when FEAT-09.SPEC-002 loads, then he sees "You're already a member of this household." instead of the acceptance form.

**FEAT-09.SPEC-011-AC-05:** Given Maya (Organiser) attempts to navigate directly to FEAT-09.SPEC-005 (Leave Household), when the request is made, then she sees "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18.

**FEAT-09.SPEC-011-AC-06:** Given Sam (Other Adult Member) opens FEAT-09.SPEC-005, when the screen loads, then he can proceed to confirm leaving.

**FEAT-09.SPEC-011-AC-07:** Given Sam attempts to navigate directly to FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), when the request is made, then he sees "Only the organiser can hand over the role" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-011-AC-08:** Given Maya (still organiser) opens FEAT-09.SPEC-004 directly via a stale link, when the screen loads, then she sees "You're the organiser -- there's nothing to accept here."

**FEAT-09.SPEC-011-AC-09:** Given the organiser role transfers to Sam while Maya still has FEAT-09.SPEC-001 open, when Maya (now Other Adult Member) attempts to revoke an invitation, then it is rejected with "Only the organiser can manage invitations."

**FEAT-09.SPEC-011-AC-10:** Given an unauthenticated visitor attempts to open FEAT-09.SPEC-003 directly, when the request is made, then they are redirected to FEAT-01.SPEC-001 with no household data rendered.

**FEAT-09.SPEC-011-AC-11:** Given Riley (Operator) is signed in only through the separate read-only support view (FEAT-22), when any FEAT-09 screen is requested under his identity, then no such entry point exists and no invitation or membership data is returned.

**FEAT-09.SPEC-011-AC-12:** Given a young kid profile has no login, when any request is made under that profile's identity against any action in this feature, then no action succeeds, since no valid session can exist for it.

**FEAT-09.SPEC-011-AC-13:** Given Sam attempts to bypass the hidden invitation-management controls via a direct URL to FEAT-09.SPEC-001, then the destination screen itself blocks the action with "Only the organiser can manage invitations," regardless of the navigation path used.

**FEAT-09.SPEC-011-AC-14:** Given a member with no hand-over request addressed to them opens FEAT-09.SPEC-004 (defensive case, e.g., a guessed URL), then no request is shown and no accept/decline action is available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-09.SPEC-010) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

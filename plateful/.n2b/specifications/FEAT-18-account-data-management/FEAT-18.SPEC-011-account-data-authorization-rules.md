---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-011
spec_name: Account & Data Authorization Rules
spec_slug: account-data-authorization-rules
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Account & Data Authorization Rules

## Overview

**Name:** Account & Data Authorization Rules
**ID:** FEAT-18.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs who can export household data, remove members, delete the household, manage their own account, or contact support, as the single authoritative home for role-gated behavior across every screen in this feature.
**Parent Feature:** FEAT-18 -- Account & Data Management
**Governed Entity:** Household and Member Profile (the access dimension -- which role may view or act on each, across FEAT-18.SPEC-001 through FEAT-18.SPEC-005)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines: export household data, remove a member, delete the household, manage own account (edit, delete), and contact support
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an expired session see across this feature's screens

**Non-Goals:**
- Field-level validation, rate-limiting, confirmation patterns, the 30-day purge rule, and the organiser hand-over-or-delete-first precondition -- governed by FEAT-18.SPEC-010, which is more specific to those entity-level and process rules
- Authorization for features outside this one (e.g., who may edit household budget or schedule) -- each feature owning its own entities defines its own authorization; this spec covers only the actions FEAT-18 itself defines
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley has no access of any kind to this feature's actions or data (Access Matrix: Account & Data column is None for Riley), not even a read-only view

## Governed Entity

**Entity:** Household and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Household and Member Profile fields already defined in FEAT-18.SPEC-010) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-18.SPEC-001 | Export Household Data | On screen entry and on Request Export |
| FEAT-18.SPEC-002 | Remove Member Profile | On screen entry and on the confirmation Remove action |
| FEAT-18.SPEC-003 | Delete Household | On screen entry and on the confirmation Delete Household Permanently action |
| FEAT-18.SPEC-004 | My Account | On screen entry (own-only field scope and the organiser-only section's visibility) and on every save/delete action |
| FEAT-18.SPEC-005 | Contact Support | On screen entry and on Send |
| FEAT-18.SPEC-007 | Member Removal Processing | On processing (re-check the requester is the organiser and the target is not the organiser) |
| FEAT-18.SPEC-008 | Household Deletion Processing | On processing (re-check the requester is the organiser) |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | On processing (re-check the requester acts only on their own account) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-18.SPEC-010's scope. No field in the governed entities carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Export household data | Maya (Organiser) | Under the export rate limit (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-001) not shown to Sam at all; a direct navigation attempt shows "Only the household organiser can export household data." and returns him to FEAT-18.SPEC-004 |
| Remove a member (adult or kid, excluding the organiser) | Maya (Organiser) | The target is not the organiser (XBR-15) and irreversible-action confirmation is completed (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-002) not shown to Sam; a direct navigation attempt shows "Only the household organiser can remove a member." and returns him to FEAT-18.SPEC-004 |
| Delete the household | Maya (Organiser) | Irreversible-action confirmation is completed (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-003) not shown to Sam; a direct navigation attempt shows "Only the household organiser can delete the household." and returns him to FEAT-18.SPEC-004 |
| View own account details | Maya, Sam | Always, for the viewer's own record only | -- |
| Edit own name, email, or sign-in | Maya, Sam | Own-only -- only the requesting member's own Member Profile fields | No control exists on FEAT-18.SPEC-004 to edit any field of a Member Profile other than the viewer's own; there is no "select another member" path on this screen |
| Delete own account | Sam | Always, for Sam's own account, subject to no additional precondition | -- |
| Delete own account | Maya | Only when she is not the current organiser or the household has been deleted (FEAT-18.SPEC-010's precondition) | Blocking dialog: "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to FEAT-09.SPEC-003 and FEAT-18.SPEC-003 |
| View the Household Data & Deletion section (Export, Remove Member, Delete Household links) | Maya (Organiser) | Always | Section is not rendered at all on FEAT-18.SPEC-004 for Sam -- not hidden-but-present, absent from the layout entirely |
| Contact support | Maya, Sam | Always, for any adult member | -- |
| View, export, or manage household data in any form | Jordan (young kid profile, no login -- MVP) | Never | N/A -- no login exists for a young kid profile; no session can reach any screen in this feature |
| View, export, or manage household data in any form | Jordan (older kid, limited login -- Later) | Never | This login has no Account & Data access (Access Matrix: None); a direct attempt shows "This isn't available for your login." |
| View or act on any action this feature defines | Riley (Operator, support) | Never | N/A -- Riley's access is limited entirely to the separate read-only support view (FEAT-22, XBR-14); no screen or action in this feature has any operator path, not even a read-only one |
| Manage another member's account details (edit or delete) | Maya | Never -- the organiser cannot edit or delete another adult's own account through this feature | No control exists on any FEAT-18 screen for the organiser to edit or delete another member's own-account fields; removal (a distinct action, deleting the whole profile including data) is the only organiser-initiated action against another member, via FEAT-18.SPEC-002 |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults relevant to this feature are defined in FEAT-18.SPEC-010.

## Business Rules

- Every screen in this feature (FEAT-18.SPEC-001 through FEAT-18.SPEC-005) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household -- sees only the sign-in screen; no screen in this feature ever renders household or account data to an unauthenticated session.
- A session that expires mid-flow on any screen in this feature shows the dialog "Your session has expired. Sign in to continue." and returns the user to the sign-in screen, preserving in-progress own-account field edits (FEAT-18.SPEC-004 only, since it is the sole screen in this feature with editable, non-destructive form fields).
- Organiser-only authorization (export, remove member, delete household, own-account-deletion precondition) is enforced at both the screen layer (FEAT-18.SPEC-001, FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-004 each check the acting member's current role or ownership) and the processing layer (FEAT-18.SPEC-007, FEAT-18.SPEC-008, FEAT-18.SPEC-009 each re-check at the moment of processing, not only at screen entry), since a role can change between screen load and action -- for example, an organiser hand-over completing mid-session.
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1), plus the unauthenticated and expired-session states this feature's own screens define. No role beyond these is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to any FEAT-18 screen** -- Redirected to the sign-in screen; no household or account data is ever rendered, even momentarily, during the redirect.
- **Sam attempts to bypass the hidden organiser-only controls via a direct navigation URL to FEAT-18.SPEC-001, FEAT-18.SPEC-002, or FEAT-18.SPEC-003** -- The destination screen itself enforces the same authorization check on entry, showing the exact denied message for that action regardless of how the screen was reached.
- **The organiser role transfers to Sam while Maya still has FEAT-18.SPEC-004 open with the Household Data & Deletion section visible** -- Her next attempt to use an organiser-only link on that screen (e.g., tapping Delete Household) is rejected at the destination screen's own entry check, showing the exact denied message, since authorization is evaluated at action time, not at screen-load time.
- **Riley's operator session somehow reaches a FEAT-18 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; no household or account data is ever rendered to an operator session outside FEAT-22's own read-only support view.
- **A kid profile somehow attempts a direct action against this feature (defensive case, e.g., a stale token)** -- No login exists for a young kid profile and the older-kid login (Later) has no entitlement to this feature's actions (Access Matrix: None); no action succeeds under either identity.
- **Maya attempts to reach another member's own-account edit fields through any means (e.g., a manipulated request)** -- No control or path exists on any FEAT-18 screen to expose another member's own-account fields to the organiser; the only organiser-initiated action against another member's data is removal (FEAT-18.SPEC-002), a distinct, whole-profile action, never a field-level edit.

## Acceptance Criteria

**FEAT-18.SPEC-011-AC-01:** Given Maya (Organiser) is signed in and under the export rate limit, when she opens FEAT-18.SPEC-001, then she has full access to request and download an export.

**FEAT-18.SPEC-011-AC-02:** Given Sam (Other Adult Member) attempts to navigate directly to FEAT-18.SPEC-001, when the request is made, then he sees "Only the household organiser can export household data." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-03:** Given Sam attempts to navigate directly to FEAT-18.SPEC-002, when the request is made, then he sees "Only the household organiser can remove a member." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-04:** Given Sam attempts to navigate directly to FEAT-18.SPEC-003, when the request is made, then he sees "Only the household organiser can delete the household." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-05:** Given Sam opens FEAT-18.SPEC-004, when the screen loads, then no Household Data & Deletion section is rendered anywhere on the screen.

**FEAT-18.SPEC-011-AC-06:** Given Maya opens FEAT-18.SPEC-004, when the screen loads, then the Household Data & Deletion section with Export, Remove a member, and Delete household links is shown.

**FEAT-18.SPEC-011-AC-07:** Given Sam is on FEAT-18.SPEC-004, when he attempts to delete his own account, then no precondition blocks him and he proceeds to the confirmation step.

**FEAT-18.SPEC-011-AC-08:** Given Maya is still the organiser and attempts to delete her own account, when the check runs, then she sees "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to FEAT-09.SPEC-003 and FEAT-18.SPEC-003.

**FEAT-18.SPEC-011-AC-09:** Given Maya and Sam are both on FEAT-18.SPEC-005, when either submits a support description, then it is accepted, since Contact Support is available to any adult member.

**FEAT-18.SPEC-011-AC-10:** Given the older-kid limited login (Later) attempts to reach any FEAT-18 screen, when the screen loads, then "This isn't available for your login." is shown and no account or household data is exposed.

**FEAT-18.SPEC-011-AC-11:** Given the organiser role transfers to Sam while Maya still has FEAT-18.SPEC-004 open, when Maya (now Other Adult Member) attempts to tap Delete Household, then it is rejected at FEAT-18.SPEC-003's own entry check with "Only the household organiser can delete the household."

**FEAT-18.SPEC-011-AC-12:** Given an unauthenticated visitor attempts to open any FEAT-18 screen directly, when the request is made, then they are redirected to the sign-in screen with no household data rendered.

**FEAT-18.SPEC-011-AC-13:** Given Riley (Operator) is signed in only through the separate read-only support view (FEAT-22), when any FEAT-18 screen is requested under his identity, then no such entry point exists and no household or account data is returned.

**FEAT-18.SPEC-011-AC-14:** Given a young kid profile has no login, when any request is made under that profile's identity against any action in this feature, then no action succeeds, since no valid session can exist for it.

**FEAT-18.SPEC-011-AC-15:** Given Sam attempts to bypass the hidden organiser-only controls via a direct URL to FEAT-18.SPEC-002, then the destination screen itself blocks the action with "Only the household organiser can remove a member.", regardless of the navigation path used.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-18.SPEC-010) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

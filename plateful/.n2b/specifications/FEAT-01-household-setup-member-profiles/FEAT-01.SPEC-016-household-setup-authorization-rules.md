---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-016
spec_name: Household Setup Authorization Rules
spec_slug: household-setup-authorization-rules
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Household Setup Authorization Rules

## Overview

**Name:** Household Setup Authorization Rules
**ID:** FEAT-01.SPEC-016
**Type:** Logic/Rule
**Purpose:** Governs who can view or change each part of household setup, including kid-profile data restrictions and the unauthorized-visitor experience, as the single authoritative home for role-gated behavior across every setup screen.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Household and Member Profile (the access dimension -- which role may view or act on each, across every setup screen FEAT-01.SPEC-001 through FEAT-01.SPEC-010)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines on Household and Member Profile data
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an expired session see across this feature's screens

**Non-Goals:**
- Dietary Rule-specific authorization (who may edit/remove a rule) -- governed by FEAT-01.SPEC-015, which is more specific to that entity's own removal-confirmation gate
- Authorization for features outside this one (e.g., who may approve a Weekly Plan) -- each feature owning its own entities defines its own authorization; this spec covers only Household and Member Profile actions within FEAT-01
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley's access to household setup facts is View-only and delivered exclusively through that separate view, per XBR-14

## Governed Entity

**Entity:** Household and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Household and Member Profile fields already defined in FEAT-01.SPEC-014) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | On screen entry (pre-authentication) and on sign-in routing |
| FEAT-01.SPEC-002 | Password Recovery | On screen entry (pre-authentication) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On screen entry and on save |
| FEAT-01.SPEC-004 | Member List & Add Member | On screen entry (Add/Invite visibility) and on save |
| FEAT-01.SPEC-005 | Member Profile Detail | On screen entry (edit vs. read-only) and on save |
| FEAT-01.SPEC-006 | Dietary Rules Editor | On screen entry (edit vs. read-only) |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | On screen entry (organiser only) |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On screen entry (organiser only) and on save |
| FEAT-01.SPEC-009 | Setup Complete & Next Steps | On screen entry (organiser only) |
| FEAT-01.SPEC-010 | Household Settings Hub | On screen entry (row visibility per role) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-01.SPEC-014's and FEAT-01.SPEC-015's scope. No field in the governed entity carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Sign up / sign in | Maya, Sam (any adult) | Always, pre-authentication | -- |
| Create household | Maya (Organiser) | Account has no existing household | An account that already has a household is routed directly to FEAT-01.SPEC-010, never shown FEAT-01.SPEC-003 |
| View/edit household name, budget, schedule | Maya (Organiser) | Full, always | -- |
| View household name, budget, schedule | Sam (Other Adult Member) | View only, always | Edit entry points not shown; a direct attempt shows "Only the organiser can change this" |
| View household name, budget, schedule | Jordan (young kid profile, no login), Jordan (older kid, limited login) | Never | N/A -- no login exists (young kid); household setup is outside the older-kid login's entitlements |
| View household name, budget, schedule | Riley (Operator, support) | View only, exclusively through the separate read-only support view (FEAT-22), from v1 | Riley never reaches this feature's own screens directly |
| Add/invite members | Maya (Organiser) | Household has fewer than 12 Active members | Add/Invite controls not shown to Sam at all; a direct navigation attempt shows "Only the organiser can add members" |
| View member list | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Edit a member's own display name/age band/type | Maya (Organiser) | Any member | -- |
| Edit a member's own display name/age band/type | Sam (Other Adult Member) | Never | Edit controls not shown; a direct attempt shows "Only the organiser can change member details" |
| View a kid profile's details and dietary rules | Sam (Other Adult Member) | View only, always | -- |
| Edit a kid profile's details or dietary rules | Sam (Other Adult Member) | Never | Fields render read-only with the note "Only the organiser can change member details" (FEAT-01.SPEC-005) or "Only the organiser can change dietary rules" (FEAT-01.SPEC-006) |
| Edit own notification preferences | Maya, Sam (each, own record only) | Always | Editing another member's preference is never exposed as a control |
| Confirm parental consent for a kid profile | Maya (Organiser) | Always, required before kid-profile creation | No other role has an entry point to this action |
| View kid profile data | Riley (Operator, support) | Never, except the allergy details inside a specific open safety report (XBR-14) | Riley never sees a kid profile's full detail; broader visibility is structurally absent from the support view |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults for Household and Member Profile fields are defined in FEAT-01.SPEC-014.

## Business Rules

- Every screen in this feature (FEAT-01.SPEC-001 through FEAT-01.SPEC-010) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household -- sees only a sign-in screen (FEAT-01.SPEC-001), an invitation-acceptance screen for a link addressed to them (owned by FEAT-09), or the public welcome page of a household referral link (owned by FEAT-24) -- never household data, per user-persona.md's Access Matrix notes.
- A session that expires mid-setup returns the user to FEAT-01.SPEC-001 with the message "Your session has expired. Sign in to continue."; any unsaved draft is preserved per FEAT-01.SPEC-013 and restored to the same screen after re-authentication.
- Riley's (Operator) access to any fact this feature owns is always View-only and delivered exclusively through the separate read-only support view (FEAT-22, from v1) -- never through this feature's own screens, and every such visit is recorded where the organiser can see it (XBR-14).
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1). No role beyond these five is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to FEAT-01.SPEC-004 (Member List)** -- Redirected to FEAT-01.SPEC-001; no household data is ever rendered, even momentarily, during the redirect.
- **Sam's session expires while he is viewing a kid profile's read-only dietary rules** -- He is redirected to FEAT-01.SPEC-001 with the expired-session message; since he had no unsaved edits (his access is read-only there), nothing needs to be preserved.
- **Riley's operator session somehow reaches a FEAT-01 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; household data is never rendered to an operator session outside FEAT-22's own read-only support view.
- **The organiser role changes hands mid-session (FEAT-09 hand-over completes while the former organiser has a setup screen open)** -- The former organiser's next action against an organiser-only control (e.g., attempting to save a budget change) is rejected with "Only the organiser can change this," since authorization is evaluated at action time, not at screen-load time.
- **A kid profile somehow attempts a direct action (defensive case, e.g., a stale token)** -- No login exists for a young kid profile, so no valid session can be associated with one; no action succeeds.
- **Sam attempts to bypass the hidden Add/Invite controls via a direct navigation URL** -- The destination screen itself enforces the same authorization check on entry, showing "Only the organiser can add members" regardless of how the screen was reached.

## Acceptance Criteria

**FEAT-01.SPEC-016-AC-01:** Given an unauthenticated visitor attempts to open FEAT-01.SPEC-010 directly, when the request is made, then they are redirected to FEAT-01.SPEC-001 and see no household data.

**FEAT-01.SPEC-016-AC-02:** Given Maya (Organiser) is signed in, when she opens any FEAT-01 screen, then she has full view and edit access per the Authorization Rules table.

**FEAT-01.SPEC-016-AC-03:** Given Sam (Other Adult Member) is signed in, when he opens FEAT-01.SPEC-004, then he sees the member list but no Add or Invite controls.

**FEAT-01.SPEC-016-AC-04:** Given Sam attempts to navigate directly to a member edit screen, when the screen loads, then he sees "Only the organiser can change member details" and no edit control is reachable.

**FEAT-01.SPEC-016-AC-05:** Given Sam views a kid profile's dietary rules, when the screen loads, then every rule renders read-only.

**FEAT-01.SPEC-016-AC-06:** Given Maya's session expires while she is mid-edit on FEAT-01.SPEC-008, when she is redirected, then she sees "Your session has expired. Sign in to continue." and her unsaved entry is restored after she signs back in.

**FEAT-01.SPEC-016-AC-07:** Given Riley (Operator) is using the separate read-only support view (FEAT-22), when he views household facts, then he sees them as View-only and never through any FEAT-01 screen directly.

**FEAT-01.SPEC-016-AC-08:** Given a young kid profile has no login, when any request is made under that profile's identity, then no action succeeds, since no valid session can exist for it.

**FEAT-01.SPEC-016-AC-09:** Given the organiser role is handed over to Sam via FEAT-09 while Maya still has FEAT-01.SPEC-008 open, when Maya (now a non-organiser) attempts to save a budget change, then it is rejected with "Only the organiser can change this."

**FEAT-01.SPEC-016-AC-10:** Given an unauthorized visitor follows a household referral link (FEAT-24), when the welcome page loads, then they see only the inviter's first name and no other household data.

**FEAT-01.SPEC-016-AC-11:** Given Sam attempts to reach the add-member flow via a direct URL, then the destination screen itself blocks the action with "Only the organiser can add members," regardless of the navigation path used.

**FEAT-01.SPEC-016-AC-12:** Given Riley's session somehow constructs a direct request to a FEAT-01 screen, then no household data is returned, since the operator role has no entry point into this feature's own screens.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-01.SPEC-014) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

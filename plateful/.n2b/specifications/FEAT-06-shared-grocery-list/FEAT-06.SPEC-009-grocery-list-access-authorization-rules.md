---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-009
spec_name: Grocery List Access & Authorization Rules
spec_slug: grocery-list-access-authorization-rules
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Grocery List Access & Authorization Rules

## Overview

**Name:** Grocery List Access & Authorization Rules
**ID:** FEAT-06.SPEC-009
**Type:** Logic/Rule
**Purpose:** Defines who can view, add, tick, edit, or remove list items, and what an unauthorized visitor or unauthorized role experiences instead.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List (with its Grocery List Items)

## Scope and Non-Goals

**In Scope:**
- Who can view the household's current Grocery List, and under what conditions
- Who can add, tick, edit, remove, and mark "already have it" on Grocery List Items
- What an unauthorized visitor, an unauthenticated user, and an out-of-scope role each experience
- Riley's (Operator, support) conditional, read-only access tied to an open Support Request

**Non-Goals:**
- Household-level aisle names and unit-system settings -- owned by Units, Currency & Locale Configuration (FEAT-16); no role gains the ability to change those settings through this feature, including the older-kid role, whose access is scoped to list items, not list settings
- Field-level validation of list item content -- owned by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual)
- Conflict resolution for concurrent or offline writes -- owned by FEAT-06.SPEC-008; this spec governs who may act, not how simultaneous actions merge

## Governed Entity

**Entity:** Grocery List
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The plan week this list serves |
| aisle_grouping | text | The household's configured aisle names and order (owned by FEAT-16; read-only from this feature) |
| status | enum | Generated, Active, or Archived |

Item-level actions governed by this spec (add, tick, edit, remove, already-have-it) apply to Grocery List Item, whose fields are defined and field-validated in FEAT-06.SPEC-006 and FEAT-06.SPEC-007; this spec addresses authorization for actions on those items, not their field content.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Grocery List | On screen entry (which controls render at all) and on every action attempt (add, tick, edit, remove, already-have-it) |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Authorization is not evaluated here directly -- recalculation is system-driven, not a member action, and operates regardless of who is viewing |
| FEAT-06.SPEC-003 | "Already Have It" Handling | On trigger, to confirm the requesting member already has Grocery List access before differentiating the pantry-logging outcome |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type -- this spec governs access, not field validation; set by FEAT-06.SPEC-002 | Always | -- | -- | -- |
| aisle_grouping | No validation beyond data type -- owned and validated by FEAT-16, read-only from this feature | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions governed by FEAT-06.SPEC-002 (Generated/Active) and FEAT-06.SPEC-004 (Archived) | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs access and authorization, not cross-field validation. Field- and cross-field validation for list items is governed by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual).

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the current Grocery List | Maya (Organiser) | Always | -- |
| View the current Grocery List | Sam (Other Adult Member) | Always | -- |
| View the current Grocery List | Jordan (older kid, limited login -- Later) | Always | -- |
| View the current Grocery List | Jordan (young kid profile, no login -- MVP) | Never | This profile has no login and cannot reach the screen at all |
| View the current Grocery List | Riley (Operator, support -- from v1) | Only while an open Support Request exists for the household (XBR-14) | Outside an open Support Request, access is blocked entirely: Riley cannot open the household's list, and no household list data is shown |
| Add a manual item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Add a manual item | Riley | Never | The Add-item control is not rendered; Riley's view is strictly read-only per Operator Read-Only Support Access (FEAT-22) |
| Tick or untick an item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Tick or untick an item | Riley | Never | Tick controls are not rendered |
| Edit an item's quantity | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Edit an item's quantity | Riley | Never | Edit controls are not rendered |
| Remove an item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Remove an item | Riley | Never | Remove controls are not rendered |
| Mark "already have it" on a plan-derived item | Maya, Sam | Always; also creates a Pantry Item in the same tap, since both hold Full Pantry Input access (FEAT-06.SPEC-003) | -- |
| Mark "already have it" on a plan-derived item | Jordan (older kid, limited login) | Always; the item is removed from the list, but no Pantry Item is created, since this role's Pantry Input access is None (FEAT-06.SPEC-003) | -- |
| Mark "already have it" on a plan-derived item | Riley | Never | The control is not rendered |
| Change household aisle names or unit-system settings from this feature | Every role | Never -- this action does not exist within FEAT-06 at all | No control for this action is ever shown here, for any role, since aisle names and units are configured exclusively through FEAT-16 |
| Open a household's list directly (e.g., a stale or guessed link) | Unauthenticated visitor | Never | Redirected to the sign-in screen; no household list data is exposed |

## Defaults and Derivations

N/A -- this spec governs authorization only; it defines no default or derived field values. Field derivations for Grocery List Item are governed by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual); Grocery List's own field transitions are governed by FEAT-06.SPEC-002 (creation, status) and FEAT-06.SPEC-004 (archive).

## Business Rules

- XBR-14: Riley's support access opens only for one household with an open Support Request, is strictly read-only, never shows more than the list itself, closes the moment the request is resolved, and every visit is recorded where Maya (the organiser) can see it.
- This spec's Authorization Rules table is the single source of truth for FEAT-06.SPEC-001's Access and Visibility table -- the two must never diverge.
- The older-kid role's Full access to Grocery List actions never extends to household-level list settings (aisle names, unit system) or to the grocery-ordering handoff (FEAT-20, Later); both are outside this feature's authorization surface entirely, not merely restricted for this role.

## Edge Cases

- **Riley's open Support Request is resolved while Riley is actively viewing the list** -- Access is revoked immediately: the screen redirects out with a message that support access has ended, and no further household list data is shown.
- **A household member is removed from the household (XBR-16) while they have the list open** -- Their access is revoked immediately; any further action attempt on the screen is rejected as if by an unauthenticated visitor.
- **Jordan's older-kid session expires mid-edit** -- Handled per FEAT-06.SPEC-001's Expired session state: no data is lost, but no further list actions succeed until re-authentication.
- **A member's role changes mid-session (e.g., organiser hand-over, per FEAT-09)** -- The Grocery List access level for both the outgoing and incoming organiser is unaffected by the hand-over, since both Maya and Sam already hold Full access to this feature; no authorization change is triggered by an organiser hand-over specifically.

## Acceptance Criteria

**FEAT-06.SPEC-009-AC-01:** Given Maya (Organiser) opens the Grocery List, when the screen loads, then she can view and act on every item without restriction.

**FEAT-06.SPEC-009-AC-02:** Given Sam (Other Adult Member) opens the Grocery List, when the screen loads, then he can view and act on every item without restriction.

**FEAT-06.SPEC-009-AC-03:** Given Jordan (young kid profile, no login), when any attempt is made to view the list, then it never succeeds, since this profile has no login and cannot reach the screen.

**FEAT-06.SPEC-009-AC-04:** Given Jordan (older kid, limited login) opens the Grocery List, when the screen loads, then he can add, tick, edit, remove, and mark "already have it" on items, but no household list-setting control is ever shown to him.

**FEAT-06.SPEC-009-AC-05:** Given Riley (Operator) attempts to open a household's Grocery List with no open Support Request, when the attempt is made, then access is blocked entirely and no household list data is shown.

**FEAT-06.SPEC-009-AC-06:** Given Riley opens a household's Grocery List while a Support Request is open, when the screen loads, then he sees every item read-only, with no tick, add, edit, remove, or already-have-it controls.

**FEAT-06.SPEC-009-AC-07:** Given the open Support Request Riley was viewing under is resolved while he is on the screen, when the resolution occurs, then his access is revoked immediately and he is redirected out with a message that support access has ended.

**FEAT-06.SPEC-009-AC-08:** Given Maya taps "Already have it" on a plan-derived item, when the action completes, then the item is removed and a Pantry Item is created, since Maya holds Full Pantry Input access.

**FEAT-06.SPEC-009-AC-09:** Given Jordan (older kid, limited login) taps "Already have it" on a plan-derived item, when the action completes, then the item is removed but no Pantry Item is created, since his Pantry Input access is None.

**FEAT-06.SPEC-009-AC-10:** Given any household member looks for a way to change aisle names from the Grocery List screen, when they look, then no such control exists anywhere on this feature's screens, for any role.

**FEAT-06.SPEC-009-AC-11:** Given an unauthenticated visitor requests a household's Grocery List directly, when the request is made, then they are redirected to the sign-in screen and no household data is exposed.

**FEAT-06.SPEC-009-AC-12:** Given a household member is removed from the household while their Grocery List screen is open, when the removal takes effect, then any further action they attempt is rejected as if they were unauthenticated.

**FEAT-06.SPEC-009-AC-13:** Given the organiser role is handed over from Maya to Sam, when the hand-over completes, then both continue to have Full Grocery List access exactly as before, since the hand-over does not itself change either member's Grocery List authorization.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 0 (N/A -- see Cross-Field Rules section) | 0 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A -- see Defaults and Derivations section) | 0 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

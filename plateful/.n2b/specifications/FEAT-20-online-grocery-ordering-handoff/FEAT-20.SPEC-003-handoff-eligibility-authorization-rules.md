---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-20.SPEC-003
spec_name: Handoff Eligibility & Authorization Rules
spec_slug: handoff-eligibility-authorization-rules
parent_feature: FEAT-20
parent_feature_name: Online Grocery Ordering Handoff
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 25
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Handoff Eligibility & Authorization Rules

## Overview

**Name:** Handoff Eligibility & Authorization Rules
**ID:** FEAT-20.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs whether the handoff entry point and screen are available, combining regional online-ordering availability with the Access Matrix's per-role handoff permission.
**Parent Feature:** FEAT-20 -- Online Grocery Ordering Handoff
**Governed Entity:** Grocery List (read-only, for eligibility determination only)

## Scope and Non-Goals

**In Scope:**
- The single regional-availability gate: whether an online-ordering capability exists for the household's region
- The per-role handoff permission: which roles may see the entry point, reach the screen, and confirm a handoff
- What each role and eligibility outcome experiences at both evaluation points -- FEAT-06's Grocery List screen entry point and this feature's own Handoff screen
- Keeping the two evaluation points consistent with each other and with the Access Matrix in user-persona.md

**Non-Goals:**
- Field-level validation or derivation of Grocery List or Grocery List Item content -- owned by FEAT-06 (FEAT-06.SPEC-006, FEAT-06.SPEC-007, FEAT-06.SPEC-009); this spec governs handoff eligibility only, not list content
- The request/response contract with the online-ordering capability -- owned by FEAT-20.SPEC-002; this spec only decides whether that integration may be initiated
- The screen mechanics of the Ineligible, Confirming, Success, and Failure states -- owned by FEAT-20.SPEC-001; this spec defines only the eligibility outcome those states render
- Persisting eligibility history or an audit trail of eligibility checks -- excluded per feature-overview.md's Non-Goals: this feature persists no entity of its own, and eligibility is re-evaluated fresh at each check rather than logged

## Governed Entity

**Entity:** Grocery List
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The plan week this list serves -- not used by this spec's eligibility check |
| aisle_grouping | text | The household's configured aisle names and order -- not used by this spec's eligibility check |
| status | enum | Generated, Active, or Archived -- not used by this spec's eligibility check |

This spec's actual governed object is the handoff action itself (an action evaluated against the Household's region and the initiating member's role), rather than any field of the Grocery List or Grocery List Item entities. The Grocery List entity is named as the nominal governed entity because the handoff action is reached from and evaluated against it; its fields carry no validation rules from this spec, which are owned entirely by FEAT-06.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Grocery List | On screen render, to decide whether the "hand off list" entry point is shown at all |
| FEAT-20.SPEC-001 | Grocery Handoff Screen | On screen load, to decide the Ineligible vs. Ready-to-Confirm rendering |
| FEAT-20.SPEC-002 | Online Grocery Ordering Integration | Refuses to initiate a request for an ineligible household or member rather than re-deriving either check itself, per the Brief's Shared Validation section |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type -- this spec governs handoff eligibility and authorization, not list field content; owned by FEAT-06 | Always | -- | -- | -- |
| aisle_grouping | No validation beyond data type -- owned by FEAT-16 and FEAT-06; read-only from this spec | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions owned by FEAT-06.SPEC-002 and FEAT-06.SPEC-004 | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs a single regional-availability gate combined with per-role authorization, not multi-field validation of Grocery List or Grocery List Item content. Field- and cross-field validation for those entities is governed entirely by FEAT-06.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Maya (Organiser) | Only while the household's region has an available online-ordering capability | Entry point is not shown on FEAT-06.SPEC-001 at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability | Entry point is not shown on FEAT-06.SPEC-001 at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Jordan (older kid, limited login -- Later) | Never | Entry point is never shown to this role regardless of the household's region, since this role's Grocery List Full access (FEAT-06.SPEC-009) covers adding and ticking only, not handoff (user-persona.md, Access Matrix notes) |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach FEAT-06.SPEC-001 or any household screen at all |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Riley (Operator, support -- from v1) | Never | Entry point is never shown; Riley's Grocery List View access does not extend to the handoff action, which carries no access for this role (product-features.md, Access field) |
| View the "hand off list" entry point on FEAT-06.SPEC-001 | Unauthenticated visitor | Never | Cannot reach FEAT-06.SPEC-001 at all; redirected to the sign-in screen |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Maya (Organiser) | Only while the household's region has an available online-ordering capability | Screen loads in the Ineligible state: "Online grocery ordering isn't available in your area yet." |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability | Screen loads in the Ineligible state: "Online grocery ordering isn't available in your area yet." |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Jordan (older kid, limited login -- Later) | Never | Screen loads in the Permission Denied state: "This action needs an adult household member." -- shown regardless of region eligibility |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach this screen through any path |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Riley (Operator, support -- from v1) | Never | Access is denied entirely; no household grocery data is shown |
| Reach the Handoff screen (FEAT-20.SPEC-001), including direct navigation | Unauthenticated visitor | Never | Redirected to the sign-in screen; no household data is exposed |
| Confirm the handoff (send the list) | Maya (Organiser) | Only while the household's region has an available online-ordering capability and the screen has loaded in the Ready-to-Confirm state | Confirm Handoff control is not rendered; the screen shows the Ineligible state instead |
| Confirm the handoff (send the list) | Sam (Other Adult Member) | Only while the household's region has an available online-ordering capability and the screen has loaded in the Ready-to-Confirm state | Confirm Handoff control is not rendered; the screen shows the Ineligible state instead |
| Confirm the handoff (send the list) | Jordan (older kid, limited login -- Later) | Never | Confirm Handoff control is never rendered for this role |
| Confirm the handoff (send the list) | Jordan (young kid profile, no login -- MVP) | Never | Has no login and cannot reach the control at all |
| Confirm the handoff (send the list) | Riley (Operator, support -- from v1) | Never | Confirm Handoff control is never rendered; Riley's access to this feature is denied entirely |
| Confirm the handoff (send the list) | Unauthenticated visitor | Never | Redirected to the sign-in screen before any control is reachable |

## Defaults and Derivations

N/A -- this spec governs eligibility gating and authorization only; it defines no default or derived field values. Grocery List and Grocery List Item field derivations are governed entirely by FEAT-06 (FEAT-06.SPEC-006, FEAT-06.SPEC-007).

## Business Rules

- Regional online-ordering availability is checked identically at both evaluation points -- the "hand off list" entry point on FEAT-06.SPEC-001 and the Handoff screen's own load on FEAT-20.SPEC-001 -- per feature-overview.md's Side-Effect Inventory, so the household never sees an entry point that leads to an inconsistent eligibility outcome on the destination screen.
- The older-kid role's Grocery List Full access (FEAT-06.SPEC-009) never extends to this feature's handoff action, consistent with the Access Matrix notes in user-persona.md: "The older-kid row's Grocery List Full covers adding and ticking items, not household list settings ... or grocery-ordering handoff."
- This spec's Authorization Rules table is the single source of truth for both FEAT-06.SPEC-001's entry-point visibility and FEAT-20.SPEC-001's Access and Visibility table -- the three must never diverge.
- FEAT-20.SPEC-002 refuses to initiate a request for an ineligible household or member rather than re-deriving either the regional or role check itself, per the Feature Breakdown Brief's Shared Validation section.

## Edge Cases

- **The household's region availability changes from eligible to ineligible while the Handoff screen is already open, before Confirm Handoff is tapped** -- The next evaluation (screen reload or re-entry) shows the Ineligible state; a confirm already sent before the change completes as initiated, since eligibility is not re-checked mid-flight for an already-triggered request.
- **The household's region becomes eligible after previously being ineligible, while a member is viewing the Ineligible state** -- The screen does not update automatically; the member must return to FEAT-06.SPEC-001 and re-enter the Handoff screen to see the updated eligibility, since no live-polling behavior is defined for this rarely-changing condition.
- **Jordan (older kid, limited login) attempts direct URL navigation to the Handoff screen, bypassing the FEAT-06.SPEC-001 entry point** -- Denied identically to reaching it through the entry point: eligibility is enforced on the Handoff screen's own load, not only through entry-point visibility.
- **The organiser role is handed over from Maya to Sam (FEAT-09) while a handoff confirm is mid-flight** -- Unaffected: both Maya and Sam hold identical Full handoff eligibility under this spec, so the hand-over changes nothing about the in-flight request or either member's standing eligibility.

## Acceptance Criteria

**FEAT-20.SPEC-003-AC-01:** Given the household's region has an available online-ordering capability, when Maya (Organiser) opens FEAT-06.SPEC-001, then the "hand off list" entry point is shown to her.

**FEAT-20.SPEC-003-AC-02:** Given the household's region has no available online-ordering capability, when Maya opens FEAT-06.SPEC-001, then the "hand off list" entry point is not shown at all.

**FEAT-20.SPEC-003-AC-03:** Given the household's region is eligible, when Sam taps "hand off list" and reaches FEAT-20.SPEC-001, then it loads in the Ready-to-Confirm state.

**FEAT-20.SPEC-003-AC-04:** Given the household's region is ineligible, when Sam reaches FEAT-20.SPEC-001 by any means, then it shows the Ineligible state: "Online grocery ordering isn't available in your area yet."

**FEAT-20.SPEC-003-AC-05:** Given Maya is on FEAT-20.SPEC-001 with an eligible region, when she taps Confirm Handoff, then the handoff proceeds and FEAT-20.SPEC-002 is triggered.

**FEAT-20.SPEC-003-AC-06:** Given Jordan (older kid, limited login) attempts to reach FEAT-20.SPEC-001 directly, when the attempt is made, then it is denied with "This action needs an adult household member." regardless of the household's region eligibility.

**FEAT-20.SPEC-003-AC-07:** Given Jordan (older kid, limited login) is viewing FEAT-06.SPEC-001, when he looks for the "hand off list" entry point, then it is never shown to him, even when the household's region is eligible.

**FEAT-20.SPEC-003-AC-08:** Given Jordan (young kid profile, no login), when any attempt is made to reach either FEAT-06.SPEC-001's entry point or FEAT-20.SPEC-001, then it never succeeds, since this profile has no login at all.

**FEAT-20.SPEC-003-AC-09:** Given Riley (Operator, support) attempts to open a household's Handoff screen, when the attempt is made, then it is denied entirely under every condition, since Riley holds no handoff permission at all.

**FEAT-20.SPEC-003-AC-10:** Given an unauthenticated visitor requests FEAT-20.SPEC-001 directly, when the request is made, then they are redirected to the sign-in screen and no household data is exposed.

**FEAT-20.SPEC-003-AC-11:** Given the household's region availability changes from eligible to ineligible while Maya has FEAT-20.SPEC-001 open but has not yet tapped Confirm Handoff, when she reloads or re-enters the screen, then it shows the Ineligible state.

**FEAT-20.SPEC-003-AC-12:** Given a household's region becomes eligible after previously being ineligible, when Sam is viewing the Ineligible state, then the screen does not update automatically; he must return to FEAT-06.SPEC-001 and re-enter to see the current eligibility.

**FEAT-20.SPEC-003-AC-13:** Given the organiser role is handed over from Maya to Sam mid-session (FEAT-09), when the hand-over completes, then both continue to have identical Full handoff eligibility under this spec, unaffected by the hand-over.

**FEAT-20.SPEC-003-AC-14:** Given Jordan (older kid) attempts direct URL navigation to FEAT-20.SPEC-001 bypassing the FEAT-06.SPEC-001 entry point, when the attempt is made, then it is denied identically to reaching it through the entry point, since eligibility is enforced on the screen's own load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 0 (N/A -- see Cross-Field Rules section) | 0 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 0 (N/A -- see Defaults and Derivations section) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-008
spec_name: Plan Approval Authorization Rule
spec_slug: plan-approval-authorization-rule
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Plan Approval Authorization Rule

## Overview

**Name:** Plan Approval Authorization Rule
**ID:** FEAT-03.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs who may approve a plan, that approval can be given once per week, and that later changes happen only through swaps.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Weekly Plan (status, approval fields)

## Scope and Non-Goals

**In Scope:**
- Who may approve a Weekly Plan and under what conditions
- The once-per-week limit on approval
- The distinction between organiser approval and auto-adoption
- What happens when approval is attempted against a plan already approved or adopted

**Non-Goals:**
- The auto-adoption process itself -- owned by FEAT-03.SPEC-005 (Auto-Adoption at Week Start), which this spec's approval-state read feeds; this spec defines what "already approved" means, not the adoption mechanics
- Executing a swap after approval -- owned by One-Tap Meal Swap (FEAT-04); this spec establishes only that swaps are the sole post-approval change path, per XBR-07
- The organiser role itself (who holds it, hand-over) -- owned by Household Invitations & Membership (FEAT-09, XBR-15); this spec reads the current organiser but does not manage the role

## Governed Entity

**Entity:** Weekly Plan
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The calendar week the plan covers |
| origin | enum | AI-generated or manually built |
| status | enum | Generated/Started, Reviewed, Approved, Active, Archived -- governed by this spec's transitions |
| approval | derived | Organiser approval (once per week) or auto-adoption at week start -- governed by this spec |
| estimated_total | number | Computed by FEAT-03.SPEC-006; not governed here |
| over_budget_note | text | Computed by FEAT-03.SPEC-006; not governed here |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-001 | Weekly Plan View | On the Approve action tap; authorization checked on both screen entry (control shown only to Maya) and on the action itself |
| FEAT-03.SPEC-005 | Auto-Adoption at Week Start | Reads this spec's approval state at the week boundary to decide whether adoption is needed |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| approval | Must be set at most once per Weekly Plan, either by an explicit organiser approval or by auto-adoption -- never both, and never more than one of either | Always | On the Approve action attempt | "This week's plan has already been approved." (organiser attempts approval on an already-approved or already-adopted plan) | Yes |
| status | Must follow the sequence Generated -> Approved (or -> Active via auto-adoption) -> Archived; a status transition out of order is rejected | Always | On any transition attempt | N/A -- transitions are system-invoked, not directly user-editable; an out-of-order transition is a defensive rule, not a user-facing validation | Yes (system-level) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Approval implies status transition | approval, status | Setting approval (by organiser action) transitions status from Generated to Approved in the same action; the two fields are never set independently of one another | N/A |
| Auto-adoption implies status transition | approval, status | Auto-adoption (FEAT-03.SPEC-005) transitions status from Generated to Active and sets approval to reflect auto-adoption, in the same action | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Approve the Weekly Plan | Maya (Organiser) | Only when the plan's status is Generated (not yet Approved or Active) and it is the current household's organiser attempting it | The Approve control shows the toast "This week's plan has already been approved." if attempted after approval or adoption already occurred |
| Approve the Weekly Plan | Sam (Other Adult Member) | Never | The Approve control is not shown to Sam; only Maya's role carries this entitlement, per the Access Matrix's Weekly Plan Full/View split |
| Approve the Weekly Plan | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| Approve the Weekly Plan | Jordan (older kid, limited login -- Later) | Never | The Approve control is not shown; this row has View-only access to Weekly Plan |
| Approve the Weekly Plan | Riley (Operator, support -- from v1) | Never | Riley's access is read-only in every case, including when a Support Request is open (FEAT-22, XBR-14); no approval control is ever shown |
| View the plan's approval/status | Maya, Sam, Jordan (older kid, Later) | Always, per each role's View or Full access to Weekly Plan | -- |
| View the plan's approval/status | Riley (Operator) | Only while a Support Request for the household is open | Outside an open Support Request, no access |
| Change the plan after approval or adoption | Maya (via swap) | Only through One-Tap Meal Swap (FEAT-04); no direct bulk edit of an approved or active plan | Any attempt to re-open bulk editing of an already-approved or -adopted plan is not offered; only per-slot swap actions are available |
| Change the plan after approval or adoption | Sam (via swap suggestion) | Only through suggesting a swap (FEAT-04, Own-only); requires Maya's acceptance | Sam sees only the suggest-a-swap path, never a direct edit |

**Note on Roles Touched:** The Feature Breakdown Brief's Spec Inventory lists this spec's Roles Touched as "Maya, Sam" only. The Authorization Rules table above also governs both Jordan rows (young kid profile, no login -- MVP; older kid, limited login -- Later) and Riley (Operator, support), since every action x role combination for the governed entity must have a defined Authorization Rules row per this spec's own methodology, and all four roles trace to the Access Matrix. This agent's contract does not permit editing the Brief, so the discrepancy between the Brief's summary column and this spec's full role coverage is recorded here rather than resolved by changing the Brief; the Authorization Rules table above is the complete and authoritative role coverage for this rule.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| approval | Unset at plan creation (Generated status) | On Weekly Plan creation (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | N/A -- becomes set only through the organiser's approval action or auto-adoption |
| status | Generated, at plan creation | On Weekly Plan creation | N/A -- transitions only through this spec's governed paths |

## Business Rules

- XBR-07: Only the organiser approves the week's plan, once per week; later changes happen through swaps; a plan not approved by the start of the week is adopted as proposed so the household is never without a plan.
- Approval and auto-adoption are mutually exclusive outcomes for a given week's plan -- a plan reaches Active status through exactly one of the two paths, never both.
- An organiser hand-over (FEAT-09, XBR-15) transfers the approval entitlement to the new organiser immediately; the outgoing organiser loses the Approve control the moment the hand-over completes, and any approval already recorded under the outgoing organiser remains valid.
- Approval authorization is checked both on screen entry (whether the Approve control is shown at all, per role) and on the action itself (whether the plan's current status still permits approval) -- a role check alone is not sufficient, since a valid organiser can still be denied if the plan state has already moved past Generated.

## Edge Cases

- **Maya taps Approve twice in rapid succession (double-tap)** -- The second tap is rejected with "This week's plan has already been approved." once the first approval completes; no duplicate approval record is created.
- **Maya's approval and the week-start auto-adoption boundary occur at effectively the same moment** -- First-decision-wins: whichever transition (explicit approval or auto-adoption) completes first sets status to Approved or Active respectively, and the other is rejected as already-approved/adopted (per FEAT-03.SPEC-005's Edge Cases, which this spec's approval-state read governs).
- **Organiser hand-over occurs mid-week while the plan is still unapproved** -- The incoming organiser gains the Approve control immediately; the plan's approval state is unaffected by the hand-over itself, and either organiser (before or after hand-over) approving it still counts as the single permitted approval for that week.
- **Sam attempts to approve by directly invoking the underlying approval action (not through the UI control)** -- Rejected regardless of entry point, since approval authorization is enforced independently of which screen initiated the attempt; Sam's role is never permitted this action.
- **A plan is auto-adopted, and Maya later wants to change it** -- She uses a swap (FEAT-04) exactly as she would on an organiser-approved plan; auto-adoption does not reopen a bulk-edit path that approval would not also have closed.

## Acceptance Criteria

**FEAT-03.SPEC-008-AC-01:** Given Maya is the organiser and the week's plan is still Generated, when she taps Approve, then the plan's status transitions to Approved and approval is recorded as her explicit approval.

**FEAT-03.SPEC-008-AC-02:** Given Maya has already approved the week's plan, when she attempts to approve it again, then she sees "This week's plan has already been approved." and no duplicate approval is recorded.

**FEAT-03.SPEC-008-AC-03:** Given Sam is viewing the week's plan, when he looks for an Approve control, then none is shown to him.

**FEAT-03.SPEC-008-AC-04:** Given the older-kid login (Later) is viewing the week's plan, when the screen renders, then no Approve control is shown to that row.

**FEAT-03.SPEC-008-AC-05:** Given Riley (Operator) is viewing a household's plan under an open Support Request, when the screen renders, then no Approve control is shown to Riley under any condition.

**FEAT-03.SPEC-008-AC-06:** Given Maya taps Approve twice in rapid succession, when the second tap registers after the first has completed, then it is rejected as already-approved and no second approval record is created.

**FEAT-03.SPEC-008-AC-07:** Given the week begins with no approval recorded, when the week-start boundary is reached, then FEAT-03.SPEC-005 adopts the plan and this spec's approval field reflects auto-adoption rather than organiser approval.

**FEAT-03.SPEC-008-AC-08:** Given a plan has already been auto-adopted, when Maya attempts to approve it afterward, then she sees "This week's plan has already been approved." since the plan is already Active.

**FEAT-03.SPEC-008-AC-09:** Given a household's organiser role is handed over mid-week while the plan is unapproved, when the hand-over completes, then the incoming organiser gains the Approve control immediately and the outgoing organiser loses it.

**FEAT-03.SPEC-008-AC-10:** Given a plan has already been approved by Maya, when she wants to change a dinner afterward, then she does so through a swap (FEAT-04), with no bulk-edit path offered.

**FEAT-03.SPEC-008-AC-11:** Given a plan has been auto-adopted, when Sam suggests a swap on a dinner, then the suggestion follows the same Own-only, organiser-accepts flow as it would on an organiser-approved plan.

**FEAT-03.SPEC-008-AC-12:** Given Sam attempts to trigger the approval action directly rather than through the Approve control, when the attempt is evaluated, then it is rejected on the same authorization grounds regardless of entry point.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

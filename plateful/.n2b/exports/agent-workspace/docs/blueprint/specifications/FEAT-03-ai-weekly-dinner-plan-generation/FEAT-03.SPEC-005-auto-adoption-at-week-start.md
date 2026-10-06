---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-005
spec_name: Auto-Adoption at Week Start
spec_slug: auto-adoption-at-week-start
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Auto-Adoption at Week Start

## Overview

**Name:** Auto-Adoption at Week Start
**ID:** FEAT-03.SPEC-005
**Type:** Automation
**Purpose:** System adopts a plan the organiser has not approved by the start of the week, so the household is never without a plan.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Firing at the start of each household's week for a Weekly Plan still in Generated status (no organiser approval recorded)
- Transitioning that plan to Active as proposed, without requiring any user action
- The resulting display and downstream effects of an auto-adopted plan

**Non-Goals:**
- The organiser's explicit approval action itself -- owned by FEAT-03.SPEC-008 (Plan Approval Authorization Rule); this automation only handles the case where that action never happened in time
- Generating the plan being adopted -- owned by FEAT-03.SPEC-003 or FEAT-03.SPEC-004; this automation acts only on a plan that already exists in Generated status
- Manual weekly plans -- Manual Weekly Planning (FEAT-23) has no approval step to auto-adopt around; a manually built plan is simply used as picked, so this automation applies only to AI-generated Weekly Plans

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Start of the week arrives with no organiser approval recorded | Household.week boundary (system schedule, derived from the household's week definition) | Fires once per household per week, at the moment the covered week begins, only when the current Weekly Plan's status is still Generated (approval field unset) | Weekly Plan (week, origin, status, approval), the household's organiser (Member Profile) |

## Processing Logic

1. At the start of each household's week, check the current Weekly Plan's status via FEAT-03.SPEC-008 (Plan Approval Authorization Rule)'s approval-state read.
2. If the plan's status is already Approved, do nothing -- the week proceeds under the organiser's own approval.
3. If the plan's status is still Generated (no approval recorded), transition the Weekly Plan's status to Active and set its approval to reflect auto-adoption (distinct from an organiser's explicit approval, per FEAT-03.SPEC-008's field definition).
4. Signal the change so FEAT-03.SPEC-001 reflects the Active status and FEAT-03.SPEC-011 propagates the change live to every household member's device.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Plan already approved | Weekly Plan.status is Approved when the week starts | None | No visible change -- the week proceeds as already approved | FEAT-03.SPEC-001 |
| Plan auto-adopted | Weekly Plan.status is Generated when the week starts | Weekly Plan.status transitions to Active; approval set to auto-adopted | FEAT-03.SPEC-001 shows "Active (adopted)" in place of an Approve control; no blocking message, since the household is never left without a plan | FEAT-03.SPEC-001, FEAT-03.SPEC-011, FEAT-21.SPEC-003 |
| No plan exists to adopt | Generation failed and no Weekly Plan exists for the covered week | None | FEAT-03.SPEC-001 continues to show its Error/Retry state from the failed generation; auto-adoption has nothing to act on | FEAT-03.SPEC-001, FEAT-03.SPEC-003 |

## Data Model

**Reads:** Weekly Plan -- status, approval, week.
**Creates:** None.
**Updates:** Weekly Plan -- status (Generated -> Active), approval (set to reflect auto-adoption).
**Deletes:** None.

## Business Rules

- XBR-07: A plan not approved by the start of the week is adopted as proposed, so the household is never without a plan; auto-adoption applies only when no approval has been recorded, per FEAT-03.SPEC-008.
- Auto-adoption never blocks or delays the week -- it runs silently at the week boundary with no user action required and no error state if it fires as designed.
- Once auto-adopted, later changes to the plan happen only through swaps (FEAT-04), identically to an organiser-approved plan -- auto-adoption does not reopen the plan to bulk editing.

## Edge Cases

- **Organiser approves at the exact moment the week begins** -- First-decision-wins: if the approval write completes before this automation's check reads the plan's status, the plan is already Approved and auto-adoption takes no action; if this automation's transition completes first, the organiser's approval attempt is rejected per FEAT-03.SPEC-008's "already approved" handling, since the plan has already moved to Active by another path.
- **No Weekly Plan exists for the covered week (prior generation failed)** -- Auto-adoption has nothing to transition; the household continues to see the previous week's plan with the Error/Retry state from FEAT-03.SPEC-003, and this automation logs a no-action outcome rather than creating a placeholder plan.
- **Household has no organiser at the moment the week starts (mid-hand-over, FEAT-09)** -- Auto-adoption proceeds regardless, since it requires no organiser action; the household is never left without a plan even during a role hand-over.
- **Trigger fires while a swap suggestion is pending review** -- Auto-adoption transitions the plan's status only; it does not resolve pending Swap Suggestions, which continue to lapse or await review under FEAT-04's own rules once the plan is Active.
- **Two households' week boundaries occur at effectively the same time** -- Each household's check and transition runs independently against its own Weekly Plan; neither affects the other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | References (inbound) | Supplies the approval-state read this automation checks, and defines what "already approved" means |
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | References (inbound) | Supplies the Weekly Plan this automation may adopt; a failed generation leaves nothing to adopt |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Shows "Active (adopted)" once this automation fires |
| FEAT-03.SPEC-011 (Real-Time Plan Sync Integration) | Triggers (outbound) | Propagates the status change live to every household member's device |

## Analytics and Success Signals

- **plan_auto_adopted** (household id, week) -- supports success-metrics.md: "Weekly Planning Time" (a household that frequently relies on auto-adoption is not completing its weekly review within the target window, a signal worth surfacing against the same metric)

## Acceptance Criteria

**FEAT-03.SPEC-005-AC-01:** Given Maya's household's Weekly Plan is still Generated (unapproved) when the week begins, when the week-start trigger fires, then the plan's status transitions to Active and FEAT-03.SPEC-001 shows "Active (adopted)."

**FEAT-03.SPEC-005-AC-02:** Given Maya approved the week's plan before it started, when the week-start trigger fires, then no change occurs and the plan remains Approved.

**FEAT-03.SPEC-005-AC-03:** Given no Weekly Plan exists for the covered week because generation failed, when the week-start trigger fires, then no adoption occurs and FEAT-03.SPEC-001 continues showing its Error/Retry state.

**FEAT-03.SPEC-005-AC-04:** Given the plan is auto-adopted, when any household member opens FEAT-03.SPEC-001, then they see the plan with an "Active (adopted)" indicator instead of an Approve control.

**FEAT-03.SPEC-005-AC-05:** Given the plan is auto-adopted, when the household wants to change a dinner afterward, then the change happens only through a swap (FEAT-04), identically to an organiser-approved plan.

**FEAT-03.SPEC-005-AC-06:** Given Maya's approval and the week-start boundary occur at effectively the same moment and her approval is recorded first, when the week-start trigger runs its check, then it finds the plan already Approved and takes no action.

**FEAT-03.SPEC-005-AC-07:** Given Maya's approval and the week-start boundary occur at effectively the same moment and the auto-adoption transition completes first, when Maya's approval attempt then reaches the system, then it is rejected as already-approved per FEAT-03.SPEC-008, since the plan is already Active.

**FEAT-03.SPEC-005-AC-08:** Given a household is mid-organiser-hand-over with no active organiser at the moment the week starts, when the week-start trigger fires, then auto-adoption proceeds normally and the household is not left without a plan.

**FEAT-03.SPEC-005-AC-09:** Given a plan is auto-adopted while a swap suggestion from Sam is still pending, when adoption completes, then the pending suggestion remains unresolved and continues to follow FEAT-04's own review-or-lapse behavior.

**FEAT-03.SPEC-005-AC-10:** Given two households' week boundaries occur at effectively the same time, when both trigger, then each household's plan is evaluated and adopted (or not) independently of the other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (week-start boundary) | 1 |
| Outcome Paths | 3 (already approved, auto-adopted, no plan to adopt) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

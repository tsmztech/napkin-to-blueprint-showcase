---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-009
spec_name: Generation Eligibility & Tier-Gating Rule
spec_slug: generation-eligibility-tier-gating-rule
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Generation Eligibility & Tier-Gating Rule

## Overview

**Name:** Generation Eligibility & Tier-Gating Rule
**ID:** FEAT-03.SPEC-009
**Type:** Logic/Rule
**Purpose:** Governs the prerequisites a household must meet -- complete dietary data, a schedule, a paid subscription -- before generation runs.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Household (generation-eligibility state, derived from Subscription, Member Profile, and Dietary Rule data it reads)

## Scope and Non-Goals

**In Scope:**
- The tier-gating check: is the household's subscription paid?
- The non-tier eligibility checks: does at least one member have complete dietary-rule data, and is a schedule set?
- Routing to FEAT-03.SPEC-001 (with incomplete-setup messaging) versus FEAT-03.SPEC-002 (free-tier placeholder) versus proceeding to generation
- What the household experiences at each failure point

**Non-Goals:**
- Setting or editing dietary rules, schedule, or subscription tier -- owned by Household Setup & Member Profiles (FEAT-01) and Subscription & Billing Management (FEAT-14); this rule only reads their current state
- The generation process itself once eligibility passes -- owned by FEAT-03.SPEC-003 and FEAT-03.SPEC-004, which call this rule rather than duplicating its checks
- The free-tier placeholder screen's own layout and actions -- owned by FEAT-03.SPEC-002; this rule only determines when that screen is shown

## Governed Entity

**Entity:** Household (eligibility state derived from Subscription, Member Profile, Dietary Rule)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Subscription.tier | enum | Free or paid -- read for tier-gating |
| Subscription.billing_state | enum | Active, Payment failed (grace period), Cancelled, Reverted to free -- read to determine effective access during grace/cancellation windows |
| Household.weekly_schedule | text | Which nights are time-constrained; absent schedule means no time constraint, which is itself a valid, complete state (FEAT-01: "no schedule set defaults to no time constraint, not a hard block") |
| Member Profile (per member) | reference | Read to confirm each active member has a Dietary Rule statement on file |
| Dietary Rule (per member) | reference | Must include at least an explicit "no restrictions" statement or actual rules for at least one household member before generation can run |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | Checked at the start of every scheduled generation attempt, before any candidate assembly begins |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Checked immediately on the upgrade-confirmed event, before any candidate assembly begins |
| FEAT-03.SPEC-002 | Free-Tier Plan Placeholder & Upgrade Prompt | Reads this rule's tier-gating half to decide when it, rather than FEAT-03.SPEC-001, is shown |
| FEAT-03.SPEC-001 | Weekly Plan View | Reads this rule's non-tier half to show incomplete-setup messaging to a paid household that has not yet met dietary-data or schedule prerequisites |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Subscription.tier | Must be paid for generation to proceed | Always | At the start of every generation attempt | N/A -- a free-tier household is routed to FEAT-03.SPEC-002, not shown a blocking error | Yes |
| Subscription.billing_state | Must be Active or within its grace period (Payment failed, 7-day grace) for generation to proceed; Cancelled (past period end) or Reverted to free is treated as free-tier | During the grace period, per ASMP/XBR-05's tier-boundary logic | At the start of every generation attempt | N/A -- routes to FEAT-03.SPEC-002 once the grace period lapses | Yes |
| At least one member's Dietary Rule statement | Must exist -- either explicit rules or an explicit "no restrictions" statement -- for at least one active Member Profile | Always | At the start of every generation attempt | "Add at least one household member's dietary information before your plan can generate." (shown on FEAT-03.SPEC-001) | Yes |
| Household.weekly_schedule | No validation beyond data type -- an absent schedule is a valid, complete state meaning no time constraint; this is not a blocking condition | Always | At the start of every generation attempt | N/A -- schedule absence never blocks generation | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Tier gate precedes non-tier checks | Subscription.tier, Dietary Rule completeness | The tier check is evaluated first; a free-tier household is routed to FEAT-03.SPEC-002 regardless of its dietary-data completeness, so an incomplete-setup message is never shown to a household that would not generate anyway | N/A |
| Paid + incomplete dietary data | Subscription.tier, Dietary Rule completeness | A paid household with no member's dietary data on file sees FEAT-03.SPEC-001's incomplete-setup messaging, never the free-tier placeholder (FEAT-03.SPEC-002), since the gap is data completeness, not tier | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the eligibility check | System (invoked by FEAT-03.SPEC-003, FEAT-03.SPEC-004) | Always, as part of generation | N/A -- invoked internally |
| View the eligibility outcome (which screen is shown) | Maya (Organiser), Sam (Other Adult Member) | Always, since both roles reach the plan section and are routed by this rule's outcome | -- |
| View the eligibility outcome | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| View the eligibility outcome | Jordan (older kid, limited login -- Later) | Always, per this row's View access to Weekly Plan | -- |
| View the eligibility outcome | Riley (Operator, support -- from v1) | Only while a Support Request for the household is open | Outside an open Support Request, no access |
| Resolve an incomplete-setup eligibility gap (add dietary data, set a schedule) | Maya (Organiser) | Always -- Household Setup is Full for Maya | -- |
| Resolve an incomplete-setup eligibility gap | Sam, both Jordan rows, Riley | Never -- Household Setup is View or None for these roles | These roles see the incomplete-setup messaging but no control to resolve it; only Maya can act on it |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective tier-gating outcome | Derived: paid (Active or in-grace) -> eligible on tier; free, cancelled past period end, or reverted -> not eligible on tier | Evaluated fresh at every generation attempt | No -- households change the outcome only by changing their subscription (FEAT-14) |
| Effective non-tier eligibility outcome | Derived: at least one member's Dietary Rule statement present AND (schedule set OR absence of schedule accepted as "no time constraint") -> eligible; otherwise not eligible | Evaluated fresh at every generation attempt | No -- households change the outcome only by completing Household Setup (FEAT-01) |

## Business Rules

- XBR-05: Tier gating is a hard block, not a degraded experience -- AI plan generation, pantry-weighted suggestions, and learning from ratings are paid; free and downgraded households plan through Manual Weekly Planning (FEAT-23) instead, never a limited AI form.
- Generation requires at least one household member with complete dietary-rule data (even "no restrictions" is an explicit statement) and a schedule; an absent schedule defaults to no time constraint rather than blocking generation (product-features.md, FEAT-03 Validation & Limits).
- Partial household setup blocks plan generation only for the specific facts genuinely missing -- it never blocks generation for facts that have a valid default (e.g., no schedule set).
- This rule is the single source of truth for both FEAT-03.SPEC-003's and FEAT-03.SPEC-004's eligibility checks -- neither automation re-implements or diverges from this rule's logic.

## Edge Cases

- **Household is in its 7-day payment-failed grace period** -- Generation still proceeds as if paid, per Subscription.billing_state's grace-period allowance; the household is not routed to the free-tier placeholder until the grace period lapses without resolution.
- **Household has some members with complete dietary data and others with none entered yet** -- Eligibility passes as long as at least one active member's statement is on file; generation proceeds using whatever dietary data exists, and the incomplete-setup message is not shown once at least one member qualifies.
- **Household sets its schedule for the first time between two generation cycles** -- The next generation cycle picks up the new schedule; the eligibility check re-evaluates fresh at every attempt, so no stale "ineligible" state persists once the gap is closed.
- **A paid household's subscription lapses (grace period expires) between one generation cycle and the next** -- The following cycle's eligibility check re-evaluates and now fails tier-gating; the household is routed to FEAT-03.SPEC-002 starting that cycle, with its existing plan history unaffected (ASMP-19).
- **Household member count changes (a member is removed) leaving zero members with dietary data on file** -- Eligibility now fails on the non-tier check at the next attempt; the household sees FEAT-03.SPEC-001's incomplete-setup messaging asking for at least one member's dietary information.
- **A free-tier household's member has complete dietary data and a schedule set** -- Eligibility still fails on tier-gating alone; non-tier completeness never overrides the tier gate, per the Cross-Field Rules above.

## Acceptance Criteria

**FEAT-03.SPEC-009-AC-01:** Given a household is on the paid tier with at least one member's dietary data on file, when generation is due, then eligibility passes and generation proceeds.

**FEAT-03.SPEC-009-AC-02:** Given a household is on the free tier, when generation would otherwise be due, then eligibility fails on tier-gating and the household is routed to FEAT-03.SPEC-002 instead of generation running.

**FEAT-03.SPEC-009-AC-03:** Given a paid household has no member's dietary data on file, when generation is due, then eligibility fails on the non-tier check and FEAT-03.SPEC-001 shows "Add at least one household member's dietary information before your plan can generate."

**FEAT-03.SPEC-009-AC-04:** Given a paid household has not set a weekly_schedule, when generation is due, then eligibility does not fail on that basis alone, since an absent schedule defaults to no time constraint.

**FEAT-03.SPEC-009-AC-05:** Given a household is within its 7-day payment-failed grace period, when generation is due, then eligibility passes on tier-gating as if the household were fully paid.

**FEAT-03.SPEC-009-AC-06:** Given a household's grace period lapses without payment resolution, when the next generation cycle is due, then eligibility fails on tier-gating and the household is routed to FEAT-03.SPEC-002.

**FEAT-03.SPEC-009-AC-07:** Given a household has some members with dietary data and others without, when generation is due, then eligibility passes as long as at least one active member's statement is on file.

**FEAT-03.SPEC-009-AC-08:** Given a household closes its dietary-data gap after a prior failed eligibility check, when the next generation cycle is due, then eligibility re-evaluates fresh and passes.

**FEAT-03.SPEC-009-AC-09:** Given a free-tier household has complete dietary data and a schedule set, when generation would otherwise be due, then eligibility still fails on tier-gating alone.

**FEAT-03.SPEC-009-AC-10:** Given Maya sees the incomplete-setup message on FEAT-03.SPEC-001, when she opens Household Setup, then she can add the missing dietary data or schedule herself.

**FEAT-03.SPEC-009-AC-11:** Given Sam sees the incomplete-setup message on FEAT-03.SPEC-001, when he looks for a control to resolve it, then none is available to him, since Household Setup is View-only for his role.

**FEAT-03.SPEC-009-AC-12:** Given a household's last dietary-data-bearing member is removed, leaving none on file, when the next generation cycle is due, then eligibility fails on the non-tier check and the incomplete-setup message reappears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

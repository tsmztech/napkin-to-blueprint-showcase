---
document_type: spec
spec_type: screen
spec_id: FEAT-03.SPEC-002
spec_name: Free-Tier Plan Placeholder & Upgrade Prompt
spec_slug: free-tier-plan-placeholder-upgrade-prompt
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Free-Tier Plan Placeholder & Upgrade Prompt

## Overview

**Name:** Free-Tier Plan Placeholder & Upgrade Prompt
**ID:** FEAT-03.SPEC-002
**Type:** Screen
**Purpose:** Free-tier households see a clear, unobtrusive upgrade option in place of an AI-generated plan and are routed to manual weekly planning instead.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- The placeholder shown to a free-tier or downgraded household in place of FEAT-03.SPEC-001
- The unobtrusive upgrade prompt and its route into FEAT-14 (Subscription & Billing Management)
- The route into FEAT-23 (Manual Weekly Planning), the free tier's actual planning surface
- Describing what a free-tier household does not yet see, using the dinner card pattern shared with FEAT-03.SPEC-001

**Non-Goals:**
- Building the manual week itself -- owned by Manual Weekly Planning (FEAT-23); this screen only routes there
- Processing the upgrade (payment, plan selection) -- owned by Subscription & Billing Management (FEAT-14); this screen only opens that flow
- Showing any AI-generated content -- excluded per XBR-05 and BRIEF.md's Business Context: AI plan generation is a paid-tier capability, and this screen exists specifically because that capability is not available to the household viewing it
- Explaining the free tier's other capabilities (pantry logging, ratings, the shared list) in depth -- those are covered by their own features (FEAT-05, FEAT-12, FEAT-06); this screen focuses narrowly on the AI-plan gap and the path around it

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | The household's plan slot is opened while its subscription tier is free or its prerequisites are otherwise unmet in a way that routes to the free-tier experience | Reason for the placeholder (free tier, vs. a paid household missing other prerequisites -- see Edge Cases) |
| Default entry (in-app navigation) | A free-tier household member opens the app's plan section | None -- placeholder shown directly |
| FEAT-15.SPEC-001 (Onboarding Landing) | A newly joined Other Adult Member of a free-tier household completes first-use join | Lands in context on this placeholder rather than FEAT-03.SPEC-001 |
| FEAT-14.SPEC-008 (Apply Subscription Change) | A household's subscription reverts to free (downgrade, lapsed payment past grace period) | Placeholder shown starting the household's next plan slot; the household's history and existing manual plans are unaffected (ASMP-19) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Tap "Upgrade" (opens FEAT-14.SPEC-002, Upgrade to Paid); tap "Plan this week by hand" (opens FEAT-23.SPEC-001) | -- |
| Sam (Other Adult Member) | Full screen | Tap "Plan this week by hand" leads into FEAT-23's Own-only suggestion path (FEAT-23.SPEC-003, Suggest a Pick); no upgrade action (Billing is None for Sam per the Access Matrix) | The "Upgrade" action is not shown to Sam |
| Jordan (young kid profile, no login -- MVP) | None -- no login exists for this row | None | No sign-in path exists for this profile |
| Jordan (older kid, limited login -- Later) | Full screen View | None -- no planning or billing action for this row | Upgrade and "plan by hand" actions are not shown; this row has no Manual Planning or Billing entitlement |
| Riley (Operator, support -- from v1) | Full screen View, read-only, only while a Support Request for this household is open (FEAT-22, XBR-14) | None | Every action is disabled with "Support access is read-only." |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." |

**Note on Roles Touched:** The Feature Breakdown Brief's Spec Inventory lists this spec's Roles Touched as "Maya, Sam, Jordan (older kid, Later)"; Riley (Operator) is not named in that summary column, though Riley's read-only access while a Support Request is open is grounded in the Access Matrix (FEAT-22, XBR-14) and is fully specified in the table above, consistent with sibling spec FEAT-03.SPEC-001. This agent's contract does not permit editing the Brief, so the discrepancy is recorded here; the Access and Visibility table above is the complete and authoritative role coverage for this screen.

## Layout and Content

**Header:** Screen title showing the plan's week range, consistent with FEAT-03.SPEC-001's header, so the two screens read as the same place in different states.

**Body:** A single explanatory panel in place of the dinner list:
- A short statement that the AI-generated weekly plan is a paid feature, without upsell pressure language
- One example dinner card, shown in a visually muted/inactive treatment, illustrating what an AI-generated plan would show (recipe name placeholder, cook time, rough cost, safety badge, pantry callout) -- consistent with the dinner card pattern from FEAT-03.SPEC-001, so households recognize what they are being offered
- Two primary actions, stacked: "Plan this week by hand" (routes to FEAT-23.SPEC-001, Weekly Plan (Manual Week Builder)) and "Upgrade" (routes to FEAT-14.SPEC-002, Upgrade to Paid), with "Plan this week by hand" listed first since it requires no commitment
- A short line noting that the shared grocery list, pantry logging, and ratings already work on the free tier, so the household understands the gap is specifically the AI-generated plan

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Panel content stacks full-width in the order described above.
- **Medium size class and above:** Panel content is capped at the same reading width used by FEAT-03.SPEC-001 and horizontally centered; the two action buttons render side by side rather than stacked.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Plan this week by hand" | Tap | Navigate to FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) for the current week | Screen transitions to Manual Weekly Planning | Standard navigation transition |
| "Upgrade" (Maya only) | Tap | Navigate to FEAT-14.SPEC-002 (Upgrade to Paid) | Screen transitions to Subscription & Billing Management | Standard navigation transition |
| Example dinner card | Tap | No action -- the illustrative card is display-only and not interactive | None | -- |

### Accessibility Notes

- **Focus order:** Header title -> explanatory text -> illustrative dinner card (announced as non-interactive) -> "Plan this week by hand" -> "Upgrade" (when shown).
- **Dynamic announcements:** None -- this screen has no dynamic state changes beyond navigation.
- **Keyboard alternatives:** Both actions are reachable and activatable by keyboard; the illustrative card carries no interactive semantics so it is not part of the tab order.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Placeholder (default) | The explanatory panel and two actions as described in Layout and Content | Household's plan slot is opened while tier-gated per FEAT-03.SPEC-009 | Household upgrades (routes to FEAT-03.SPEC-001 on next visit) or navigates away |
| Loading | N/A -- the free-tier/paid routing decision is already resolved by FEAT-03.SPEC-009 before this screen is entered, so there is no gating fetch on entry; the screen's own reads (Subscription.tier confirmation, Household.name for the optional personalized heading) are non-blocking, so the placeholder panel and both actions render immediately with the generic (non-personalized) heading while Household.name resolves in the background, with no spinner or skeleton state | N/A | N/A |
| Error | N/A -- the only read this screen performs beyond entry-time tier gating is Household.name for an optional personalized heading; if that read fails, the screen silently falls back to its generic heading with no error message or retry control, since none of this screen's content or actions (the illustrative dinner card, "Plan this week by hand", "Upgrade") depend on that read succeeding | N/A | N/A |
| Offline/Degraded | N/A -- this screen shows only static explanatory content and navigation actions; both destinations (FEAT-23.SPEC-001, FEAT-14.SPEC-002) handle their own offline behavior independently once opened | N/A | N/A |

## Validation Rules

Validation of when this placeholder is shown instead of FEAT-03.SPEC-001 is governed by FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule), specifically its tier-gating half. See that spec for the full eligibility logic.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Plan this week by hand" tap | FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | FEAT-23 (Manual Weekly Planning) |
| "Upgrade" tap | FEAT-14.SPEC-002 (Upgrade to Paid) | FEAT-14 (Subscription & Billing Management) |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier (to confirm the household is free-tier, gating whether this screen or FEAT-03.SPEC-001 is shown). Household -- name, for a personalized placeholder heading (optional).
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-05: AI plan generation, pantry-weighted suggestions, and learning from ratings are paid-tier capabilities; this screen exists so free and downgraded households are routed to Manual Weekly Planning (FEAT-23.SPEC-001) rather than a degraded or partial AI experience.
- FEAT-03.SPEC-009 determines when this screen is shown instead of FEAT-03.SPEC-001; this screen performs no independent tier check.
- A downgrade or lapsed payment never removes a household's past plans, ratings, recipes, pantry items, or list (ASMP-19) -- this screen's copy and the household's accessible history elsewhere (FEAT-19.SPEC-001, Weekly Plan History Browse) both honor this.

## Edge Cases

- **A paid household is routed here in error due to another unmet prerequisite (e.g., no schedule set)** -- Per FEAT-03.SPEC-009, a paid household with unmet non-tier prerequisites is never routed to this screen; it instead sees FEAT-03.SPEC-001 in its own blocked/incomplete-setup messaging. This screen is reserved for the tier-gated case only.
- **Household upgrades while this screen is open** -- The next time the household opens its plan section, FEAT-03.SPEC-001 is shown instead of this placeholder; this screen does not live-update mid-view since an upgrade requires leaving to FEAT-14.SPEC-002 to complete.
- **Sam opens this screen** -- He sees "Plan this week by hand" and no "Upgrade" action, since Billing is None for his role; he is never blocked from seeing the placeholder itself.
- **Household downgrades mid-week with an already-adopted AI plan active** -- The current week's already-generated plan remains visible on FEAT-03.SPEC-001 for the remainder of that week (its data is not deleted); this placeholder appears starting the household's next plan slot, per ASMP-19.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | Triggered by (inbound) | Tier-gating determines when this screen replaces FEAT-03.SPEC-001 |
| FEAT-03.SPEC-001 (Weekly Plan View) | References (inbound) | Shares the dinner card pattern this screen's illustrative card follows |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Navigation (outbound) | "Plan this week by hand" opens the free tier's planning surface |
| FEAT-14.SPEC-002 (Upgrade to Paid) | Navigation (outbound) | "Upgrade" opens the paid-tier flow |
| FEAT-15.SPEC-001 (Onboarding Landing) | Navigation (inbound) | First-use landing for a newly joined Other Adult Member of a free-tier household opens here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| upgrade_prompt_shown | entry source | Screen loads for a free-tier household | N/A -- no Stage 2 metric measures placeholder impressions directly; "Paid Conversion Rate" (product-features.md/success-metrics.md, Connected Feature: Subscription & Billing Management) is the metric this prompt ultimately feeds, but it is credited on FEAT-14's completed-upgrade event, not on impression, to avoid double-crediting a metric this feature does not own |
| upgrade_prompt_upgrade_tapped | -- | Maya taps "Upgrade" | N/A -- see above; the conversion outcome itself is credited within FEAT-14 |
| upgrade_prompt_manual_plan_tapped | -- | A member taps "Plan this week by hand" | N/A -- no metric in this feature's connected-metric slice measures the manual-planning routing itself; success-metrics.md's "Manual Week Completion" metric (Connected Feature: Manual Weekly Planning) is fed within FEAT-23, not here |

## Acceptance Criteria

**FEAT-03.SPEC-002-AC-01:** Given Maya's household is on the free tier, when she opens the plan section, then she sees this placeholder instead of FEAT-03.SPEC-001, with "Plan this week by hand" and "Upgrade" actions.

**FEAT-03.SPEC-002-AC-02:** Given Maya is on this placeholder, when she taps "Plan this week by hand", then the screen navigates to FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) for the current week.

**FEAT-03.SPEC-002-AC-03:** Given Maya is on this placeholder, when she taps "Upgrade", then the screen navigates to FEAT-14.SPEC-002 (Upgrade to Paid).

**FEAT-03.SPEC-002-AC-04:** Given Sam is on this placeholder, when he looks for an "Upgrade" action, then none is shown to him, and "Plan this week by hand" remains available.

**FEAT-03.SPEC-002-AC-05:** Given Maya is on this placeholder, when she views the illustrative dinner card, then it shows a muted example recipe, cook time, cost, and safety badge, and tapping it has no effect.

**FEAT-03.SPEC-002-AC-06:** Given a household completes its upgrade to paid, when a member next opens the plan section, then FEAT-03.SPEC-001 is shown instead of this placeholder.

**FEAT-03.SPEC-002-AC-07:** Given a household's subscription reverts to free mid-week with an already-adopted AI plan active, when a member opens the current plan, then the current week's AI-generated plan remains visible on FEAT-03.SPEC-001, and this placeholder appears only starting the household's next plan slot.

**FEAT-03.SPEC-002-AC-08:** Given a newly joined member of a free-tier household completes first-use join, when they land in-app, then they arrive on this placeholder rather than FEAT-03.SPEC-001.

**FEAT-03.SPEC-002-AC-09:** Given Riley (Operator) opens this screen against an open Support Request, when the screen loads, then it is fully read-only with every action disabled and "Support access is read-only." shown on attempted use.

**FEAT-03.SPEC-002-AC-10:** Given the older-kid login (Later) opens this screen, when it loads, then neither "Plan this week by hand" nor "Upgrade" is shown, consistent with that row's None entitlement for Manual Planning and Billing.

**FEAT-03.SPEC-002-AC-11:** Given an unauthenticated visitor attempts to reach this screen, when the request is made, then they are redirected to the sign-in screen.

**FEAT-03.SPEC-002-AC-12:** Given Maya's session expires while viewing this placeholder, when she taps "Upgrade", then a dialog reads "Your session has expired. Sign in to continue." before any navigation occurs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 4 (placeholder, loading N/A, error N/A, offline N/A) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

# FEAT-03 — AI Weekly Dinner Plan Generation

This chapter covers FEAT-03, AI Weekly Dinner Plan Generation, a Core-tier feature. It contains 11 specifications carrying 135 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-03.SPEC-001 | Weekly Plan View | screen | 20 |
| FEAT-03.SPEC-002 | Free-Tier Plan Placeholder & Upgrade Prompt | screen | 12 |
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | automation | 14 |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | automation | 10 |
| FEAT-03.SPEC-005 | Auto-Adoption at Week Start | automation | 10 |
| FEAT-03.SPEC-006 | Budget Fit & Estimated Total Rule | logic-rule | 11 |
| FEAT-03.SPEC-007 | Household-Scaled Quantity Rule | logic-rule | 10 |
| FEAT-03.SPEC-008 | Plan Approval Authorization Rule | logic-rule | 12 |
| FEAT-03.SPEC-009 | Generation Eligibility & Tier-Gating Rule | logic-rule | 12 |
| FEAT-03.SPEC-010 | AI Plan Generation Capability Integration | integration | 12 |
| FEAT-03.SPEC-011 | Real-Time Plan Sync Integration | integration | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: AI Weekly Dinner Plan Generation

## Summary

**Feature:** AI Weekly Dinner Plan Generation
**ID:** FEAT-03
**Description:** Every week, the household receives a proposed 7-day dinner plan that fits everyone's allergies, diets, dislikes, schedule, and budget, and makes use of what the household says is already in the fridge.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** This is the product's central promise: "AI proposes a realistic 7-day dinner plan every week that respects all of it" (BRIEF.md, Vision). It is the paid tier's primary value (BRIEF.md, Business Context) and the feature every other Core feature exists to support.

**Key Capabilities:**
- Generate the week's plan — Household receives seven dinners that respect every hard dietary rule, the stated schedule, and the budget
- Use up the pantry — The plan favors recipes that use ingredients the household has already logged as on hand
- Show cost and time per meal — Each dinner shows its rough cost and cook time, and the week shows its estimated total against the household's budget
- Scale to the household — Ingredient quantities are sized for the number of people eating, so the grocery list buys the right amount
- Approve the week's plan — The organiser reviews the proposal, including any swap suggestions from other adults, and approves it in one tap
- Learn from ratings over time — Later plans favor meals the household has rated highly and avoid ones rated poorly

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-03.SPEC-001 | Weekly Plan View | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household views the current week's seven dinners, cost/time, safety badges, pantry callouts, and weekly total, and the organiser approves the week |
| FEAT-03.SPEC-002 | Free-Tier Plan Placeholder & Upgrade Prompt | Screen | Maya, Sam, Jordan (older kid, Later) | Free-tier households see a clear, unobtrusive upgrade option in place of an AI-generated plan and are routed to Manual Weekly Planning |
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | Automation | All | System generates the week's seven-dinner plan on the household's chosen schedule, drawing on safety filtering, pantry weighting, rating history, budget, and schedule |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Automation | Maya | System generates a household's very first AI plan immediately after it upgrades to the paid tier |
| FEAT-03.SPEC-005 | Auto-Adoption at Week Start | Automation | All | System adopts a plan the organiser has not approved by the start of the week, so the household is never without a plan |
| FEAT-03.SPEC-006 | Budget Fit & Estimated Total Rule | Logic/Rule | All | Computes the week's estimated cost against the household budget and determines the closest-fitting plan with an overrun note when no safe week fits |
| FEAT-03.SPEC-007 | Household-Scaled Quantity Rule | Logic/Rule | All | Derives per-dinner ingredient quantities sized to the number of people the household is planning for |
| FEAT-03.SPEC-008 | Plan Approval Authorization Rule | Logic/Rule | Maya, Sam | Governs who may approve a plan, that approval can be given once per week, and that later changes happen only through swaps |
| FEAT-03.SPEC-009 | Generation Eligibility & Tier-Gating Rule | Logic/Rule | All | Governs the prerequisites a household must meet (complete dietary data, a schedule, a paid subscription) before generation runs |
| FEAT-03.SPEC-010 | AI Plan Generation Capability Integration | Integration | All | Product boundary to the AI text/plan-generation capability that turns household constraints and candidate recipes into a proposed seven-dinner week |
| FEAT-03.SPEC-011 | Real-Time Plan Sync Integration | Integration | All | Product boundary to the real-time data-synchronization capability that reflects generation, approval, and plan changes live across every household member's device |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Generate the week's plan | FEAT-03.SPEC-003, FEAT-03.SPEC-001 | Scheduled automation produces the seven dinners; the view screen displays them | Phase 2 (Explicit) |
| Use up the pantry | FEAT-03.SPEC-003, FEAT-03.SPEC-001 | Generation weights candidates toward logged Pantry Items (weighting owned by FEAT-05); the view shows the pantry callout on each dinner | Phase 2 (Explicit) |
| Show cost and time per meal | FEAT-03.SPEC-001, FEAT-03.SPEC-006 | The view displays per-dinner cook time and rough cost; the rule computes the week's estimated total against budget | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Scale to the household | FEAT-03.SPEC-003, FEAT-03.SPEC-007 | Generation calls the scaling rule to size ingredient quantities to the household | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Approve the week's plan | FEAT-03.SPEC-001, FEAT-03.SPEC-008, FEAT-03.SPEC-005 | The view carries the approve action; the rule governs who/how often; auto-adoption covers the unapproved case | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Learn from ratings over time | FEAT-03.SPEC-003 | Generation reads Rating history as a weighting input (the learning mechanism itself is owned by Meal Rating & Preference Learning, FEAT-12) | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-03.SPEC-002 | Free-Tier Plan Placeholder & Upgrade Prompt | Phase 6 (Negative/Failure Analysis) + Phase 2 | The "Free tier" alternate flow and the Access Matrix's tier gating imply a distinct view for households that cannot see AI generation at all |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Phase 4 (Trigger-Response) | The Empty state ("your first plan is on its way") and the navigation connection from FEAT-14's upgrade confirmation imply an event-triggered generation distinct from the weekly schedule |
| FEAT-03.SPEC-009 | Generation Eligibility & Tier-Gating Rule | Phase 3 (Entity-Lifecycle) + Phase 5 (Rule Discovery) | The Validation & Limits field's generation prerequisites, and the Subscription entity's read-for-gating relationship, needed an explicit rule surface rather than staying buried in the generation automation |
| FEAT-03.SPEC-010 | AI Plan Generation Capability Integration | Phase 4 (External Dependencies lens) | assumptions-constraints.md's Dependencies section names the AI text/plan-generation capability (ASMP-30) as required by this feature; the External Touchpoints slice lists FEAT-03 as needing this Integration spec |
| FEAT-03.SPEC-011 | Real-Time Plan Sync Integration | Phase 4 (External Dependencies lens) | assumptions-constraints.md's Dependencies section names the real-time data-synchronization capability (ASMP-35) as required by this feature; the External Touchpoints slice lists FEAT-03 among the features needing it for live plan updates |

## Entity-Lifecycle Coverage Matrix

**Entity: Weekly Plan**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-03.SPEC-003, FEAT-03.SPEC-004 | Scheduled generation and first-plan-on-upgrade both create a new Weekly Plan with status Generated | Manual creation of a Weekly Plan is a separate path owned by Manual Weekly Planning (FEAT-23) |
| Read (single) | FEAT-03.SPEC-001 | The current week's plan is displayed with its seven dinners and totals | -- |
| Read (list) | N/A | Browsing past weeks is Weekly Plan History's responsibility (FEAT-19), not this feature's | -- |
| Update | FEAT-03.SPEC-001, FEAT-03.SPEC-008 | Organiser approval sets the plan's approval field and status (Generated/Reviewed -> Approved) | Swap-driven updates to plan contents are FEAT-04's responsibility |
| Delete/Archive | FEAT-03.SPEC-003 (state transition only) | Soft archival: when the next week's plan generates, the prior week's Active plan transitions to Archived automatically; no restore path is needed since Archived plans remain fully browsable (FEAT-19) and are never hard-deleted except by household deletion (FEAT-18), which removes all household data within 30 days per ASMP-27 | Hard delete/cascade is owned by FEAT-18, not this feature |
| State Transition | FEAT-03.SPEC-003, FEAT-03.SPEC-008, FEAT-03.SPEC-005 | Generated -> Approved (organiser, SPEC-008) or -> Active via auto-adoption at week start (SPEC-005) -> Archived at the next generation cycle (SPEC-003) | -- |

**Entity: Planned Meal**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-03.SPEC-003, FEAT-03.SPEC-004 | Generation creates exactly seven Planned Meals (one per night) per Weekly Plan, each carrying recipe, cook time, rough cost, safety badge, and pantry callout | Manual picks (FEAT-23) and leftover-lunch suggestions (FEAT-11) create Planned Meals through their own paths |
| Read (single) | FEAT-03.SPEC-001 | Each dinner card on the view screen | -- |
| Read (list) | FEAT-03.SPEC-001 | The full week of seven dinners displayed together | -- |
| Update | N/A | Swaps (FEAT-04), manual changes (FEAT-23), safety removal (FEAT-02), and leftover status (FEAT-11) all update Planned Meal outside this feature | This feature only sets the initial Proposed/Picked state at creation |
| Delete/Archive | N/A | Removal after a safety concern is owned by FEAT-02; clearing a night is owned by FEAT-23; cascade on household deletion is FEAT-18's | This feature never deletes a Planned Meal directly |
| State Transition | FEAT-03.SPEC-003, FEAT-03.SPEC-004 | Sets initial status Proposed/Picked on creation; later transitions (Swapped, Removed, Cooked) are owned by FEAT-04, FEAT-02, and FEAT-12/FEAT-11 respectively | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Recipe | FEAT-03.SPEC-003, FEAT-03.SPEC-010 | Candidate pool for the seven dinners, filtered for safety and scored against pantry, rating, budget, and schedule |
| Dietary Rule | FEAT-03.SPEC-003, FEAT-03.SPEC-009 | Hard rules and learned soft dislikes constrain candidate selection and gate whether generation can run at all |
| Pantry Item | FEAT-03.SPEC-003 | Weighting input toward dinners that use ingredients already on hand (weighting logic owned by FEAT-05) |
| Rating | FEAT-03.SPEC-003 | Weighting input favoring highly-rated meals and avoiding poorly-rated ones (learning mechanism owned by FEAT-12) |
| Subscription | FEAT-03.SPEC-009, FEAT-03.SPEC-002 | Gates whether a household receives AI generation (paid) or the free-tier placeholder |
| Household | FEAT-03.SPEC-003, FEAT-03.SPEC-006, FEAT-03.SPEC-007 | Supplies weekly_budget, weekly_schedule, and member count used for budget-fit, schedule-fit, and quantity scaling |
| Member Profile | FEAT-03.SPEC-003, FEAT-03.SPEC-009 | Supplies household size for scaling and confirms each member's dietary-rule completeness before generation runs |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household's chosen plan-arrival day and time arrives | Generate the new week's seven-dinner plan | Standalone Automation | FEAT-03.SPEC-003 |
| Generation completes | Send the "next week's plan is ready" message | Cross-feature — owned by Weekly Plan Ready Notification | FEAT-07 responsibility, triggered by FEAT-03.SPEC-003 |
| Generation completes | Create/recalculate the week's Grocery List from the new plan | Cross-feature — owned by Shared Grocery List | FEAT-06 responsibility, triggered by FEAT-03.SPEC-003 |
| A new week's plan is generated | The previous week's Active plan transitions to Archived | Inline in triggering automation | FEAT-03.SPEC-003 |
| Household upgrades from free to paid tier | Generate the household's first AI plan | Standalone Automation | FEAT-03.SPEC-004 |
| Start of the week arrives with no organiser approval recorded | Adopt the plan as proposed | Standalone Automation | FEAT-03.SPEC-005 |
| Generation runs | Compute the week's estimated total against budget; if no safe week fits, select the closest-fitting plan and attach an overrun note | Standalone Logic/Rule | FEAT-03.SPEC-006 |
| Generation runs | Scale each recipe's ingredient quantities to the household's size | Standalone Logic/Rule | FEAT-03.SPEC-007 |
| Organiser taps Approve on the Weekly Plan View | Validate that the tapping member is the organiser and that the week has not already been approved, then record approval | Standalone Logic/Rule | FEAT-03.SPEC-008 |
| Generation is due to run (scheduled or first-plan) | Verify at least one member has complete dietary-rule data, a household schedule is set, and the subscription is paid before generation proceeds | Standalone Logic/Rule | FEAT-03.SPEC-009 |
| Generation proceeds | Compose Recipe, Dietary Rule, Pantry Item, and Rating data into a request to the AI plan-generation capability and receive a proposed seven-dinner week | Standalone Integration | FEAT-03.SPEC-010 |
| Approval, auto-adoption, or an accepted swap/safety removal changes the plan | Reflect the updated plan live on every household member's device | Standalone Integration | FEAT-03.SPEC-011 |
| Generation fails | Keep the previous week's plan visible and offer an explicit retry, never leaving the household with no plan | Inline in triggering automation, surfaced as an Error state on the view screen | FEAT-03.SPEC-003 / FEAT-03.SPEC-001 |
| Household member taps swap on a dinner | Open the safe-alternatives picker | Cross-feature | FEAT-04 responsibility |
| Organiser accepts another adult's swap suggestion | Plan slot updates to the suggested recipe | Cross-feature | FEAT-04 responsibility, reflected live via FEAT-03.SPEC-011 |

## Shared Context

**Shared Entities:**
- Weekly Plan — created by SPEC-003 and SPEC-004, approved via SPEC-008, auto-adopted via SPEC-005, archived (state transition) by SPEC-003, displayed by SPEC-001, synchronized live by SPEC-011. Fields: week, origin, status, approval, estimated_total, over_budget_note.
- Planned Meal — created by SPEC-003 and SPEC-004 (seven per plan), displayed by SPEC-001, sourced from AI proposals via SPEC-010, quantities set by SPEC-007. Fields: night, meal_kind, recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status, swap_history.

**Shared UI Patterns:**
- Dinner card — a single Planned Meal's cook time, rough cost, "checked against allergies" badge, and pantry callout (if any), used across every state of SPEC-001 (populated, empty pre-fill, degraded/offline) and referenced by SPEC-002 when describing what free-tier households do not yet see. Spec Writers for SPEC-001 and SPEC-002 should describe this card consistently.
- Weekly total banner — the week's estimated cost against budget plus the over-budget note when present, shown at the top of SPEC-001 and computed by SPEC-006.

**Shared Validation:**
- SPEC-009 defines the generation-eligibility and tier-gating rule. SPEC-003 and SPEC-004 both reference SPEC-009 rather than duplicating the prerequisite checks; SPEC-002 references SPEC-009's tier-gating half to decide when it, rather than SPEC-001, is shown.
- SPEC-006 and SPEC-007 define budget-fit and scaling computations that SPEC-003, SPEC-004, and SPEC-001 all reference for display and generation rather than recomputing independently.
- SPEC-008 defines approval authorization that SPEC-001 (the approve action) and SPEC-005 (the auto-adoption fallback) both reference.

## Internal Dependency Map

```
FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) -> [checks eligibility via] -> FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule)
FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) -> [checks eligibility via] -> FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule)
FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) -> [free tier, or prerequisites unmet] -> FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt)
FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) -> [requests a proposal from] -> FEAT-03.SPEC-010 (AI Plan Generation Capability Integration)
FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) -> [requests a proposal from] -> FEAT-03.SPEC-010 (AI Plan Generation Capability Integration)
FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) -> [scales quantities via] -> FEAT-03.SPEC-007 (Household-Scaled Quantity Rule)
FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) -> [checks cost via] -> FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule)
FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) -> [generation completes] -> FEAT-03.SPEC-001 (Weekly Plan View)
FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) -> [generation completes] -> FEAT-03.SPEC-001 (Weekly Plan View)
FEAT-03.SPEC-001 (Weekly Plan View) -> [organiser taps Approve] -> FEAT-03.SPEC-008 (Plan Approval Authorization Rule)
FEAT-03.SPEC-005 (Auto-Adoption at Week Start) -> [checks approval state via] -> FEAT-03.SPEC-008 (Plan Approval Authorization Rule)
FEAT-03.SPEC-005 (Auto-Adoption at Week Start) -> [no approval recorded] -> FEAT-03.SPEC-001 (Weekly Plan View shows Active status)
FEAT-03.SPEC-008 (Plan Approval Authorization Rule) / FEAT-03.SPEC-005 (Auto-Adoption at Week Start) -> [plan state changes] -> FEAT-03.SPEC-011 (Real-Time Plan Sync Integration) -> [reflects to every device] -> FEAT-03.SPEC-001 (Weekly Plan View)
```

**Default Entry:** FEAT-03.SPEC-001 (Weekly Plan View) — the screen shown when a paid household navigates to its current week's plan; free-tier households land on FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) instead.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-03.SPEC-001 | Inbound | FEAT-07 (Weekly Plan Ready Notification) | Tapping the "next week's plan is ready" notification lands on the new week's plan | Household member taps the notification |
| FEAT-03.SPEC-003 | Outbound | FEAT-07 (Weekly Plan Ready Notification) | Generation completion triggers the plan-ready message | Generation finishes successfully |
| FEAT-03.SPEC-003 | Outbound | FEAT-06 (Shared Grocery List) | A newly generated (or auto-adopted) plan drives the week's grocery list | Generation or auto-adoption completes |
| FEAT-03.SPEC-001 | Outbound | FEAT-04 (One-Tap Meal Swap) | Reviewing pending swap suggestions or tapping swap on a dinner opens FEAT-04's picker | Organiser reviews suggestions or taps swap |
| FEAT-03.SPEC-001 | Inbound | FEAT-04 (One-Tap Meal Swap) | An accepted swap or suggestion updates the displayed plan | Organiser accepts a swap or suggestion |
| FEAT-03.SPEC-001 | Outbound | FEAT-17 (Older-Kid Dinner Voting, Later) | The week plan shows a vote split for a night when voting is enabled | Organiser reviews an older kid's vote |
| FEAT-03.SPEC-001 | Outbound | FEAT-08 (Recipe Library) | Tapping a dinner opens its recipe detail | Household member taps a meal |
| FEAT-03.SPEC-001 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Reporting a safety concern on a dinner hands off to the safety-report flow | Household member taps "report a safety concern" |
| FEAT-03.SPEC-001 | Outbound | FEAT-06 (Shared Grocery List) | Opening the list from the approved week | Household member opens the list from the plan |
| FEAT-03.SPEC-001 | Outbound | FEAT-11 (Leftover Rollover to Lunches) | Confirming or skipping a leftover lunch tied to a dinner | Household member responds to the leftover-lunch card |
| FEAT-03.SPEC-001 | Outbound | FEAT-12 (Meal Rating & Preference Learning) | A rating prompt appears on a cooked meal | Household member rates after dinner |
| FEAT-03.SPEC-001 | Outbound | FEAT-25 (End-of-Week Check-In) | The weekly check-in card at the top of the plan | Household member answers the check-in |
| FEAT-03.SPEC-001 | Inbound | FEAT-13 (Tonight's Dinner Nudge) | The nudge deep-links to tonight's dinner within the plan | Household member taps the "Tonight: …" nudge |
| FEAT-03.SPEC-003 | Inbound | FEAT-05 (Pantry-Aware Suggestions) | Logged pantry items weight the next generation toward using them up | Household opens a plan that used logged items |
| FEAT-03.SPEC-001 | Inbound | FEAT-15 (Invite & Join Household) | A newly joined member lands in context on the current plan | Household member completes first-use join |
| FEAT-03.SPEC-004 | Inbound | FEAT-14 (Subscription & Billing Management) | Upgrade confirmation triggers the first-plan generation | Household completes an upgrade |
| FEAT-03.SPEC-002 | Outbound | FEAT-23 (Manual Weekly Planning) | Free-tier households start the week manually instead of receiving an AI plan | Free-tier household opens its weekly plan slot |
| FEAT-03.SPEC-001 | Inbound | FEAT-16 (Units, Currency & Locale Configuration) | Cost figures display in the household's configured unit system and currency | Household has set locale preferences |
| FEAT-03.SPEC-003 | Inbound | FEAT-09 (Household Roles & Hand-Over) | Organiser hand-over changes who may approve the plan going forward | Organiser role changes hands |

## Non-Functional Notes

**Data volumes / growth:** Each household holds at most one active Weekly Plan of seven Planned Meals at a time, growing by one archived week per household per week for the life of the account; at the expected scale of several thousand households in the first year (scope-boundaries.md, SC-15) this is a small, steadily-growing dataset per household, and every past week is retained indefinitely per SC-18.

**Responsiveness:** A weekly plan generates within well under a minute with an explained wait rather than an indefinite spinner (assumptions-constraints.md, ASMP-23); approval, auto-adoption, and swap-driven updates to the plan must reflect across household members' devices within the same near-instant window the brief sets for the grocery list (assumptions-constraints.md, ASMP-22), since the plan and its list are never allowed to disagree (dependency map, XBR-03).

**Data sensitivity / privacy:** The Weekly Plan and its Planned Meals are household personal data — what the family eats — private to the household and never sold or used for advertising (assumptions-constraints.md, ASMP-26); the plan is generated from Dietary Rule data that includes children's allergy information, the product's most sensitive data class, so every path onto the plan must carry the fail-closed safety check before display (dependency map, XBR-01).

**Compliance flags:** Because generation reads children's Dietary Rule data (age band, allergies) to build the plan, this feature operates under children's-privacy-class handling — minimal collection, parental control, no behavioral advertising (assumptions-constraints.md, ASMP-27) — even though the Weekly Plan and Planned Meal entities themselves hold no children's data directly; no medical-data regime applies, since the plan carries no medical or diet advice (scope-boundaries.md, SC-06).

## Non-Goals

- **AI-generated plans on the free tier** — Excluded per the dependency map's XBR-05 and BRIEF.md's Business Context: AI plan generation, pantry-weighted suggestions, and learning from ratings are paid-tier capabilities; free and downgraded households plan through Manual Weekly Planning (FEAT-23) instead, with generation blocked (not degraded) rather than offered in a limited form.
- **Medical or diet advice, including calorie/macro guidance** — Excluded per scope-boundaries.md (SC-06): BRIEF.md states directly "there is no medical or diet advice," so generation surfaces cost, time, and safety information only — never nutrition scoring or dietary prescriptions.
- **Other adult members executing swaps directly on the generated plan** — Excluded per scope-boundaries.md (SC-04): BRIEF.md's Target Users & Roles gives other adult members the ability to suggest swaps only; only the organiser accepts a suggestion or swaps directly, so this feature's approval and swap-review surface never grants Sam a direct-edit path.
- **Browsing or re-using past weeks' plans from within this feature** — Intentional lifecycle decision surfaced by the Entity-Lifecycle Coverage Matrix: Weekly Plan Read (list) and any "start from a past week" action are owned by Weekly Plan History (FEAT-19, product-features.md), which is the browsable, exportable historical record; this feature manages only the current week's generation, display, and approval.
- **Independent AI generation for more than one household per account** — Excluded per scope-boundaries.md (SC-03): BRIEF.md states plainly there is one household per account in v1, so generation always resolves to exactly one household's constraints and never needs multi-household disambiguation.



# Screen Spec: Weekly Plan View

## Overview

**Name:** Weekly Plan View
**ID:** FEAT-03.SPEC-001
**Type:** Screen
**Purpose:** Household views the current week's seven dinners with cost, time, safety badges, and pantry callouts, and the organiser approves the week.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Displaying the current week's seven Planned Meals with cook time, rough cost, safety badge, pantry callout, and swap-suggestion review
- The weekly total banner (estimated cost against budget, over-budget note)
- The organiser's one-tap approval action
- Entry points arriving from the plan-ready notification, the nightly nudge, and first-use onboarding
- Cross-feature entry points into swap, recipe detail, safety reporting, the grocery list, leftover lunches, ratings, the weekly check-in, and (Later) dinner voting

**Non-Goals:**
- Building or editing the plan by hand -- owned by Manual Weekly Planning (FEAT-23); this screen is a paid-tier, AI-generated plan's display and approval surface
- Executing a swap, reviewing a suggestion in detail, or picking a safe alternative -- owned by One-Tap Meal Swap (FEAT-04.SPEC-001 Meal Swap (Direct), FEAT-04.SPEC-002 Suggest a Swap, FEAT-04.SPEC-003 Review Swap Suggestions); this screen only opens those flows and reflects their outcome
- Browsing or reusing a past week's plan -- owned by Weekly Plan History (FEAT-19) per the Entity-Lifecycle Coverage Matrix's Read (list) disposition; this screen shows only the current week
- Free-tier display -- free-tier households land on FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) instead of this screen, per XBR-05

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Household member taps the "next week's plan is ready" notification | The newly generated week is opened directly |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Household member taps the "Tonight: ..." nudge | Scrolled/highlighted to tonight's dinner within the current week |
| FEAT-15.SPEC-001 (Onboarding Landing) | A newly joined Other Adult Member completes first-use join | Lands in context on the current week's plan |
| Default entry (in-app navigation) | Household member opens the app's plan section | Current week's plan, no special context |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Generation completes for a household's first AI plan | The newly generated first plan is opened directly with the "your first plan is ready" framing |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the paid tier taps "Go to your plan" | Current week's plan, no special context |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction | Scrolled/highlighted to tonight's dinner, showing the new dinner |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen: all seven dinners, weekly total, pending swap suggestions, (Later) vote splits | Approve the week (FEAT-03.SPEC-008); tap swap on any dinner (opens FEAT-04.SPEC-001); open pending suggestions for review (opens FEAT-04.SPEC-003); report a safety concern; open the grocery list; respond to leftover-lunch and check-in cards; rate meals | -- |
| Sam (Other Adult Member) | Full screen: all seven dinners, weekly total, his own pending suggestions | Suggest a swap (opens FEAT-04.SPEC-002, Own-only); report a safety concern (Own-only); mark a leftover lunch eaten or skipped (a status update, not a plan change); respond to check-in and rating prompts; open the grocery list. No Approve control. | Approve control is not shown to Sam; a direct swap (rather than a suggestion) is not offered -- Sam only sees "Suggest a swap," per the Access Matrix's Meal Swap Own-only entitlement |
| Jordan (young kid profile, no login -- MVP) | None -- no login exists for this row | None | No sign-in path exists for this profile; an adult acts on Jordan's behalf (e.g., recording a rating) |
| Jordan (older kid, limited login -- Later) | Full screen View: all seven dinners, weekly total; (Later) vote split for a night when voting is enabled | Respond to (Later) voting only, via FEAT-17; no swap, approval, or safety-report actions | Approve, swap, and "report a safety concern" controls are not shown to this row, per the Access Matrix's None entries for Meal Swap and Safety Reports |
| Riley (Operator, support -- from v1) | Full screen View, read-only, and only while a Support Request for this household is open (FEAT-22, XBR-14) | None -- no actions available | Every control (Approve, Suggest a swap, report a safety concern, rating, check-in, leftover response) is disabled; attempting one shows "Support access is read-only." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as a household member, the user lands on this screen (or FEAT-03.SPEC-002 if free tier) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No plan data is shown until re-authentication succeeds; no in-progress action (e.g., a swap suggestion draft) survives -- the user re-opens the relevant flow after signing back in |

## Layout and Content

**Header:** Screen title showing the plan's week range (e.g., "This Week"). A "grocery list" shortcut icon (right-aligned) navigates to FEAT-06.SPEC-001 (Grocery List).

**Top banner area:** The weekly check-in card (FEAT-25.SPEC-001, Weekly Check-In Card) appears first when due, above the plan content. Below it, the **weekly total banner**: the week's estimated cost against the household's weekly_budget, in the household's configured currency (per FEAT-16.SPEC-004, Cross-Feature Value Conversion Rule), plus the over-budget note when FEAT-03.SPEC-006 has attached one. If pending swap suggestions exist from Sam, a "Sam suggested N swaps -- review" summary row appears here, above the dinner list, visible to Maya only.

**Body:** A vertically stacked list of seven dinner cards, one per night (Sunday through Saturday, ordered by the household's week start), plus any leftover-lunch cards interleaved on the day they apply. Each **dinner card** (the shared UI pattern used consistently across this screen's states and referenced by FEAT-03.SPEC-002) shows:
- The night's label (e.g., "Monday") and, when tonight falls within the current week, a "Tonight" emphasis marker
- Recipe name
- Cook time and rough cost
- The "checked against allergies" safety badge with the "always check labels" disclaimer, per XBR-01
- A vegetarian-option indicator when the dinner carries one
- A pantry callout line when the dinner uses a logged Pantry Item (e.g., "uses the spinach and feta you already have")
- A "Swap" action
- A vote-split indicator (Later, older-kid voting enabled) below the recipe name
- A pending-suggestion indicator when Sam has an open suggestion for that night, visible to Maya, tapping through to FEAT-04.SPEC-003 (Review Swap Suggestions) to accept or decline it

**Footer:** The Approve action, visible to Maya only, fixed at the bottom of the screen while any dinner remains unapproved for the week; replaced by an "Approved" status indicator once approval is recorded, or an "Active (adopted)" indicator after auto-adoption (FEAT-03.SPEC-005).

### Responsive Behavior

- **Compact breakpoint:** Dinner cards stack full-width, one per row, in the vertical list described above; the Approve footer remains pinned to the bottom of the viewport.
- **Medium size class and above:** The dinner list renders as a fixed-width column capped at a consistent platform-wide reading width and horizontally centered; the weekly total banner and check-in card remain full-width above it. No structural change beyond width capping.
- **Dinner card:** Cook time, cost, and badge remain on one line at every breakpoint; the pantry callout and vote-split line wrap to their own line rather than truncating.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Dinner card | Tap (recipe name/body) | Navigate to FEAT-08.SPEC-002 (Recipe Detail View) | Screen transitions to recipe detail | Standard navigation transition |
| Swap action on a dinner card | Tap (Maya) | Navigate to FEAT-04.SPEC-001 (Meal Swap (Direct)) for that slot | Screen transitions to swap flow | Standard navigation transition |
| Swap action on a dinner card | Tap (Sam) | Navigate to FEAT-04.SPEC-002 (Suggest a Swap) for that slot | Screen transitions to suggest flow | Standard navigation transition |
| Pending-suggestion indicator | Tap (Maya) | Navigate to FEAT-04.SPEC-003 (Review Swap Suggestions) for that night | Screen transitions to the review flow | Standard navigation transition |
| "Report a safety concern" control on a dinner card | Tap (Maya, Sam) | Hand off to FEAT-02.SPEC-001 (Report a Safety Concern) for that meal | Screen transitions to the safety-report flow | Standard navigation transition |
| Leftover-lunch card | Tap "Confirm" or "Skip" | Records the leftover-lunch response via FEAT-11.SPEC-001 (Leftover Lunch Card) | Card shows the recorded response | Toast: "Marked as [eaten/skipped]." |
| Rating prompt (on a cooked meal) | Tap thumbs up/down | Records the rating via FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) | Prompt clears from that dinner card | Brief confirmation animation |
| Weekly check-in card | Answer the prompt | Records the answer via FEAT-25.SPEC-001 (Weekly Check-In Card) | Card clears or shows the recorded trend | Toast: "Thanks -- your answer is saved." |
| Vote-split indicator (Later) | Tap | Navigate to FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Screen transitions to vote detail | Standard navigation transition |
| Grocery list shortcut icon | Tap | Navigate to FEAT-06.SPEC-001 (Grocery List) | Screen transitions to the grocery list | Standard navigation transition |
| Approve action (footer, Maya only) | Tap | Trigger approval via FEAT-03.SPEC-008 | Footer changes to "Approved" indicator; loading state shown briefly during approval | Toast: "This week's plan is approved." Success sets Weekly Plan status to Approved. |
| Approve action | Tap, while already approved this week | No action -- control is replaced by the Approved indicator, so a second approval cannot be initiated | None | -- |
| Retry control (Error state) | Tap | Re-request the current week's plan data | Screen re-attempts load | Loading indicator shown during retry |

### Accessibility Notes

- **Focus order:** Header title -> grocery list shortcut -> check-in card (when present) -> weekly total banner -> pending-suggestions summary (Maya, when present) -> dinner cards in night order (each card: recipe name -> badge -> swap/suggestion controls -> report-concern control) -> Approve footer action.
- **Dynamic announcements:** A successful approval, rating, leftover response, or check-in answer is announced to assistive technology via the toast text at the moment it appears. An over-budget note appearing after generation is announced when the banner first renders. A live update to a dinner card from an accepted swap (decided on FEAT-04.SPEC-003) is announced when it arrives via FEAT-03.SPEC-011.
- **Safety badge:** The "checked against allergies" badge and "always check labels" disclaimer are conveyed as text, never by color or icon alone, per ASMP-29.
- **Keyboard alternatives:** Every action on this screen (swap, accept/decline, report, rate, respond to check-in/leftover, approve) is reachable by keyboard focus and activation; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first plan pending) | "Your first plan is on its way" message in place of the dinner list, with an explanation of the typical wait | A paid household has not yet had a first plan generated (FEAT-03.SPEC-004 has not yet completed) | First-plan generation completes and the screen re-loads with the plan |
| Loading (generation in progress) | A short, explained wait message (e.g., "Building this week's plan...") replaces the dinner list; no indefinite spinner | Scheduled or first-plan generation (SPEC-003, SPEC-004) is in progress for this household | Generation completes (success or failure) |
| Populated | Full dinner list, weekly total banner, and Approve footer as described in Layout and Content | Plan data has loaded successfully | User navigates away |
| Error (generation failed) | The previous week's plan remains visible in full, with a banner: "This week's plan couldn't be generated. Retry?" and a Retry control | Generation fails (SPEC-003 outcome: Automation failure) | User taps Retry and generation succeeds, or the next scheduled attempt succeeds |
| Offline/Degraded | The most recently loaded plan remains fully viewable, including its badges, callouts, and total; Approve, Swap, and suggestion Accept/Decline controls are disabled with the inline note "Reconnect to approve or change the plan." Rating and leftover responses queue locally and sync on reconnect. | Connectivity is lost while this screen is open or opened | Connectivity restored -- controls re-enable and any queued responses sync automatically |

## Validation Rules

Validation of who may approve the week, when, and how often is governed by FEAT-03.SPEC-008 (Plan Approval Authorization Rule). See that spec for the full authorization logic; this screen only surfaces the resulting Approve control or Approved/Active indicator.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Dinner card tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 (Recipe Library) |
| Swap action (Maya) | FEAT-04.SPEC-001 (Meal Swap (Direct)) | FEAT-04 (One-Tap Meal Swap) |
| Swap action (Sam) | FEAT-04.SPEC-002 (Suggest a Swap) | FEAT-04 (One-Tap Meal Swap) |
| Pending-suggestion indicator tap (Maya) | FEAT-04.SPEC-003 (Review Swap Suggestions) | FEAT-04 (One-Tap Meal Swap) |
| "Report a safety concern" | FEAT-02.SPEC-001 (Report a Safety Concern) | FEAT-02 (Dietary Rules & Allergy Safety Engine) |
| Grocery list shortcut | FEAT-06.SPEC-001 (Grocery List) | FEAT-06 (Shared Grocery List) |
| Vote-split indicator (Later) | FEAT-17.SPEC-003 (Vote Outcome & Resolution) | FEAT-17 (Older-Kid Dinner Voting) |
| Check-in card "invite another family" follow-up | Invite Another Household | FEAT-24 (Invite Another Household) |

## Data Model

**Creates:** None -- this screen displays and approves an existing Weekly Plan; it does not create one.
**Reads:** Weekly Plan -- week, origin, status, approval, estimated_total, over_budget_note. Planned Meal (seven per plan, plus leftover-lunch entries) -- night, meal_kind, recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status, swap_history. Household -- weekly_budget, currency (for the total banner), member count context. Swap Suggestion -- suggesting_member, night, proposed_recipe, outcome (for the pending-suggestion indicator).
**Updates:** Weekly Plan -- approval and status, set by the Approve action via FEAT-03.SPEC-008. Planned Meal status may update indirectly through actions handed off to FEAT-04.SPEC-004 (Apply Meal Swap), FEAT-02.SPEC-004 (Safety Concern Intake & Removal), and FEAT-11.SPEC-001/FEAT-12.SPEC-001 -- this screen surfaces those outcomes rather than writing the fields itself.
**Deletes:** None.

## Business Rules

- XBR-01: Every meal shown on this screen has already passed the app-enforced allergy and religious-rule check; the "checked against allergies" badge and "always check labels" disclaimer are shown on every dinner card without exception.
- XBR-03: The grocery list shortcut always reflects the current plan -- no meal on this screen is ever shown without its ingredients already present in the linked Grocery List.
- XBR-06: Sam's swap action opens a suggestion flow (FEAT-04.SPEC-002), never a direct swap; only Maya's accept/decline decision on FEAT-04.SPEC-003 (Review Swap Suggestions) changes the plan from a suggestion.
- XBR-07: Only Maya may tap Approve, once per week; if the week reaches its start with no approval recorded, FEAT-03.SPEC-005 adopts the plan automatically and this screen shows "Active (adopted)" instead of "Approved."
- The weekly total banner and any over-budget note are computed by FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) and displayed here verbatim -- this screen performs no independent cost calculation.
- Ingredient quantities behind the displayed cook time/cost figures are sized to the household by FEAT-03.SPEC-007 (Household-Scaled Quantity Rule).
- Approval authorization (who, how often, what happens on a second attempt) is governed by FEAT-03.SPEC-008.

## Edge Cases

- **User navigates here before any plan has ever been generated** -- Empty state shows "Your first plan is on its way" rather than a blank week or an error.
- **User taps Swap and Approve in rapid succession** -- Approve is disabled while any swap or suggestion action for this week is in flight, to avoid approving a plan mid-change; it re-enables once the action resolves.
- **Another household member changes the plan while this screen is open (accepted suggestion, safety removal, auto-adoption)** -- The screen live-updates via FEAT-03.SPEC-011 (Real-Time Plan Sync Integration); the affected dinner card updates in place with a brief highlight, and no reload is required. Resolution follows the dependency map's Contention note for Weekly Plan: a safety removal always wins over any concurrent change, and slot changes are reject-with-refresh with at most one active swap per slot -- a second, conflicting swap attempt on the same slot is rejected with "This dinner just changed -- here's the latest." and the card refreshes to the current recipe.
- **Maya taps Approve while a safety removal is mid-flight on some dinner** -- Approve is rejected with "One of this week's dinners just changed for safety reasons -- review it before approving." and the affected card is highlighted; Maya retries once the removal has resolved.
- **A dinner card has no pantry callout and no vote split** -- Those lines are simply omitted from the card; no placeholder text appears.
- **Household loses connectivity mid-approval** -- The approval attempt fails gracefully with the Offline/Degraded messaging; approval is not recorded until connectivity returns and the action is retried.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Generation completion populates this screen with the new week |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | First-plan completion populates this screen from the Empty state |
| FEAT-03.SPEC-005 (Auto-Adoption at Week Start) | References (inbound) | Adoption sets the "Active (adopted)" status shown here in place of Approved |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | References (inbound) | Supplies the weekly total banner and any over-budget note |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | References (inbound) | Supplies the household-sized cook time/cost figures behind each dinner card |
| FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | Triggers (outbound) | Approve action invokes this rule |
| FEAT-03.SPEC-011 (Real-Time Plan Sync Integration) | References (inbound) | Live-updates this screen when the plan changes from any source |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Navigation (inbound) | Notification tap opens this screen |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggered by (inbound) | Fires when this screen's generation source (FEAT-03.SPEC-003/004) completes |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Navigation (inbound) | Nudge tap opens this screen scrolled to tonight |
| FEAT-15.SPEC-001 (Onboarding Landing) | Navigation (inbound) | First-use landing for a newly joined Other Adult Member opens this screen |
| FEAT-04.SPEC-001 (Meal Swap (Direct)) | Navigation (outbound) | Maya's swap action opens the direct-swap flow |
| FEAT-04.SPEC-002 (Suggest a Swap) | Navigation (outbound) | Sam's swap action opens the suggest-a-swap flow |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Navigation (outbound) | Maya's pending-suggestion indicator opens the review flow |
| FEAT-04.SPEC-004 (Apply Meal Swap) | References (inbound) | Underlies both the direct swap and an accepted suggestion; its outcome is what FEAT-03.SPEC-011 propagates back to this screen |
| FEAT-02.SPEC-001 (Report a Safety Concern) | Navigation (outbound) | "Report a safety concern" opens this flow |
| FEAT-06.SPEC-001 (Grocery List) | Navigation (outbound) | Grocery list shortcut opens this screen |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Dinner card tap opens recipe detail |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | References (outbound) | Leftover-lunch card responses recorded here; the card pattern is embedded within this screen |
| FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) | References (outbound) | Rating prompts recorded here |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution, Later) | Navigation (outbound) | Vote-split indicator opens vote review |
| FEAT-25.SPEC-001 (Weekly Check-In Card) | References (outbound) | Check-in card recorded here |
| FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) | References (inbound) | Cost figures display in the household's configured unit system and currency |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_viewed | entry source (notification / nudge / onboarding / default nav), origin (AI-generated) | Screen loads with a populated plan | supports success-metrics.md: "Weekly Planning Time" |
| plan_approved | time from open to approve, suggestion count reviewed | Maya taps Approve and it succeeds | supports success-metrics.md: "Weekly Planning Time" |
| weeknight_dinner_time_fit_viewed | night, whether the household marked that night time-constrained, dinner's cook time | A dinner card for a schedule-constrained night renders | supports success-metrics.md: "Weeknight Time-Fit Accuracy" |
| plan_over_budget_shown | estimated overrun amount | The over-budget note renders in the weekly total banner | N/A -- no Stage 2 metric measures over-budget display frequency; retained to observe how often the budget-fit rule's fallback surfaces to users |

## Acceptance Criteria

**FEAT-03.SPEC-001-AC-01:** Given Maya opens the app from the "next week's plan is ready" notification, when the screen loads, then she sees all seven dinners with cook time, rough cost, safety badge, and the weekly total banner.

**FEAT-03.SPEC-001-AC-02:** Given Maya is on the Weekly Plan View with an unapproved week, when she taps Approve, then the plan's status changes to Approved, the footer shows an "Approved" indicator, and a toast confirms "This week's plan is approved."

**FEAT-03.SPEC-001-AC-03:** Given Sam is on the Weekly Plan View, when he looks for an Approve control, then none is shown to him.

**FEAT-03.SPEC-001-AC-04:** Given Maya taps a dinner card's recipe name, when the tap registers, then the screen navigates to that recipe's detail view (FEAT-08.SPEC-002).

**FEAT-03.SPEC-001-AC-05:** Given Sam taps Swap on Friday's dinner, when he picks a safe alternative on FEAT-04.SPEC-002 (Suggest a Swap), then Maya sees a pending-suggestion indicator on Friday's card on this screen.

**FEAT-03.SPEC-001-AC-06:** Given Maya sees Sam's pending suggestion on Friday, when she taps the indicator, then the screen navigates to FEAT-04.SPEC-003 (Review Swap Suggestions), and once she accepts it there, Friday's card on this screen updates to the suggested recipe via FEAT-03.SPEC-011.

**FEAT-03.SPEC-001-AC-07:** Given Maya declines Sam's pending suggestion on FEAT-04.SPEC-003, when the decline is recorded, then this screen's pending-suggestion indicator for that night clears and the original recipe remains.

**FEAT-03.SPEC-001-AC-08:** Given Maya or Sam taps "Report a safety concern" on a dinner card, when the tap registers, then the screen hands off to FEAT-02's safety-report flow for that meal.

**FEAT-03.SPEC-001-AC-09:** Given a paid household has never had a plan generated, when a member opens this screen, then it shows "Your first plan is on its way" instead of a blank week.

**FEAT-03.SPEC-001-AC-10:** Given generation is in progress for the household, when a member opens this screen, then a short explained-wait message appears in place of the dinner list.

**FEAT-03.SPEC-001-AC-11:** Given generation fails for the week, when a member opens this screen, then the previous week's plan remains fully visible with a banner offering Retry.

**FEAT-03.SPEC-001-AC-12:** Given Maya loses connectivity while viewing an already-loaded plan, when she attempts to tap Approve, then the control is disabled with the note "Reconnect to approve or change the plan," and the plan itself remains fully viewable.

**FEAT-03.SPEC-001-AC-13:** Given a dinner uses a logged pantry item, when the plan renders, then that dinner's card shows the pantry callout naming the item.

**FEAT-03.SPEC-001-AC-14:** Given the household's estimated total exceeds its weekly_budget for the closest-fitting safe plan, when the screen renders, then the weekly total banner shows the over-budget note computed by FEAT-03.SPEC-006.

**FEAT-03.SPEC-001-AC-15:** Given the week has not been approved by the start of the week, when the week begins, then FEAT-03.SPEC-005 auto-adopts the plan and this screen shows "Active (adopted)" in place of the Approve control.

**FEAT-03.SPEC-001-AC-16:** Given another household member accepts a swap suggestion while Maya has this screen open, when the change is saved, then Maya's screen updates the affected dinner card in place via FEAT-03.SPEC-011, with no manual reload.

**FEAT-03.SPEC-001-AC-17:** Given a safety concern removes a dinner from the plan while Maya is attempting to approve, when she taps Approve, then the approval is rejected with "One of this week's dinners just changed for safety reasons -- review it before approving," and the affected card is highlighted.

**FEAT-03.SPEC-001-AC-18:** Given Riley (Operator) opens this screen against an open Support Request for the household, when the screen loads, then it is fully read-only and every action control (Approve, Swap, report, rate, respond) is disabled with "Support access is read-only."

**FEAT-03.SPEC-001-AC-19:** Given the older-kid login (Later) opens this screen with dinner voting enabled for a night, when the screen renders, then that night's card shows the vote-split indicator and no swap or approval control.

**FEAT-03.SPEC-001-AC-20:** Given Maya's session expires while this screen is open with a pending suggestion showing, when she attempts to tap the pending-suggestion indicator, then a dialog reads "Your session has expired. Sign in to continue." and the pending suggestion is still shown unresolved after she signs back in.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 13 | 13 |
| States | 5 (empty, loading, populated, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



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



# Automation Spec: Scheduled Weekly Plan Generation

## Overview

**Name:** Scheduled Weekly Plan Generation
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** System generates the week's seven-dinner plan on the household's chosen schedule, drawing on safety filtering, pantry weighting, rating history, budget, and schedule.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Firing on each household's chosen plan-arrival day and time to generate the next week's seven Planned Meals
- Checking generation eligibility before running
- Requesting a proposal from the AI plan-generation capability, scaling quantities, and computing budget fit
- Archiving the previous week's Active plan when the new week generates
- The generation-failure path that keeps the previous week visible

**Non-Goals:**
- Determining whether a household is eligible to generate at all -- owned by FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule); this automation calls that rule rather than re-implementing its checks
- The very first plan a household ever receives after upgrading -- owned by FEAT-03.SPEC-004 (First-Plan Generation on Upgrade), a distinct event-triggered path
- Sending the "plan ready" notification -- owned by Weekly Plan Ready Notification (FEAT-07), triggered by this automation's completion but specified separately per the Side-Effect Inventory
- Recalculating the grocery list -- owned by Shared Grocery List (FEAT-06), triggered by this automation's completion but specified separately

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household's chosen plan-arrival day and time arrives | Household.plan_arrival_day_time (system schedule, set via FEAT-07) | Fires once per household per week, at the day/time the organiser configured (Sunday evening by default); does not fire for a household whose eligibility check (FEAT-03.SPEC-009) fails | Household constraints (weekly_budget, weekly_schedule, member count), all Member Profiles' Dietary Rules, current Pantry Items, Rating history, the Recipe candidate pool (starter + imported), the prior week's Weekly Plan (for archival) |

## Processing Logic

1. At the household's configured plan-arrival day/time, check generation eligibility and tier via FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule).
2. If eligibility fails, do not generate; the outcome is handled entirely within FEAT-03.SPEC-009 (its own outcome table governs what the household sees).
3. If eligible, assemble the household's constraint set: every active member's Dietary Rules (hard allergies and religious rules, soft dislikes and learned dislikes), the weekly_schedule (time-constrained nights), the weekly_budget, and the household's member count.
4. Read the current Recipe candidate pool (starter library plus the household's imported recipes) and current Pantry Items.
5. Read Rating history for the household to weight candidate selection toward highly-rated meals and away from repeatedly down-rated ones (learned dislikes are supplied by FEAT-12 as Dietary Rule entries and are already reflected in step 3).
6. Compose this data into a request to the AI plan-generation capability via FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) and receive a proposed set of candidate dinners.
7. Pass every candidate dinner through the app-enforced allergy and religious-rule safety check (owned by FEAT-02); exclude any candidate that fails or whose ingredient data is incomplete, per XBR-01.
8. From the safety-passed candidates, select exactly seven dinners (one per night), weighting toward dinners that use logged Pantry Items and toward highly-rated meals, and fitting each time-constrained night's cook time within its stated limit.
9. Scale each selected dinner's ingredient quantities to the household's size via FEAT-03.SPEC-007 (Household-Scaled Quantity Rule).
10. Compute the week's estimated cost against the household's weekly_budget via FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule); if no safe week fits the budget, select the closest-fitting safe combination and attach an over-budget note.
11. Create a new Weekly Plan (status: Generated, origin: AI-generated) and seven new Planned Meals (status: Proposed), each carrying its recipe, safety badge, cook time, rough cost, and pantry callout.
12. If a prior week's Weekly Plan exists in Active status, transition it to Archived.
13. Signal completion to trigger the "plan ready" notification (FEAT-07.SPEC-001, Plan-Ready Notification Trigger) and grocery list recalculation (FEAT-06.SPEC-002, Grocery List Generation & Recalculation).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Generation succeeded | Steps 1-13 complete with a full seven-dinner plan fitting the budget | New Weekly Plan (Generated) and seven Planned Meals (Proposed) created; prior Active plan archived | FEAT-03.SPEC-001 shows the new week; FEAT-07.SPEC-001 sends the plan-ready notification; FEAT-06.SPEC-002 recalculates the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |
| Generation succeeded, over budget | Steps 1-13 complete, but no safe combination of seven dinners fits weekly_budget | Same as above, plus Weekly Plan.over_budget_note set by FEAT-03.SPEC-006 | FEAT-03.SPEC-001 shows the plan with the over-budget note in the weekly total banner | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |
| Not eligible (no generation) | FEAT-03.SPEC-009's eligibility check fails (missing dietary data, no schedule, free tier) | No Weekly Plan created | Handled entirely by FEAT-03.SPEC-009's own outcome definitions (this automation defers to it) | FEAT-03.SPEC-009 |
| Generation failure | The AI plan-generation capability cannot return a usable proposal, or the safety-passed candidate pool cannot fill all seven nights | No new Weekly Plan created; the prior week's plan is not archived | The previous week's plan remains visible on FEAT-03.SPEC-001 with an Error state banner and Retry control; no member is ever left with no plan at all | FEAT-03.SPEC-001, FEAT-03.SPEC-010 |
| Retry succeeds | Household or system retries generation after a failure | Same as "Generation succeeded" | FEAT-03.SPEC-001 clears the Error state and shows the new week | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Household -- weekly_budget, weekly_schedule, plan_arrival_day_time, member count. Member Profile -- for household size and eligibility context. Dietary Rule -- all active rules per member. Recipe -- candidate pool with ingredients, cook_time, rough_cost. Pantry Item -- Active items for weighting. Rating -- history for weighting. Weekly Plan -- the prior week's plan, for archival.
**Creates:** Weekly Plan -- week, origin (AI-generated), status (Generated), estimated_total, over_budget_note (when applicable). Planned Meal -- seven records: night, meal_kind (dinner), recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status (Proposed).
**Updates:** Weekly Plan (prior week) -- status transitions from Active to Archived.
**Deletes:** None.

## Business Rules

- XBR-01: Every candidate dinner passes the app-enforced allergy and religious-rule check before it can be selected; a recipe with incomplete ingredient data is excluded, never shown unchecked.
- XBR-03: Generation completion always triggers a grocery list recalculation (FEAT-06.SPEC-002) in the same cycle -- no household ever sees a week's plan without its matching list.
- XBR-07: The plan a household receives here starts in Generated status and requires the organiser's approval (FEAT-03.SPEC-008) or auto-adoption (FEAT-03.SPEC-005) before the week begins.
- XBR-12: This automation's completion triggers at most one "plan ready" message per household per week (FEAT-07.SPEC-001); the plan's in-app availability (this automation's own effect) never depends on that notification's delivery.
- XBR-17: Learned soft dislikes (from repeated down-ratings, FEAT-12.SPEC-005 Repeated-Dislike Learned Update) influence selection weighting but never block a suggestion and never override an explicit hard rule.
- Generation always proposes exactly seven dinners, one per day, regardless of household size (product-features.md, Validation & Limits).
- This automation calls FEAT-03.SPEC-009 for eligibility, FEAT-03.SPEC-006 for budget fit, FEAT-03.SPEC-007 for quantity scaling, and FEAT-03.SPEC-010 for the AI proposal itself -- it does not duplicate any of those rules.

## Edge Cases

- **A brand-new household with no pantry items logged** -- Generation proceeds normally; pantry-awareness simply has nothing to weight toward that week, and the plan is still complete and safe.
- **Fewer than seven safety-passed candidates exist for a household's rules** -- Treated as a generation failure; the previous week's plan remains visible with the Error state and Retry, since a plan that cannot honor every hard rule for all seven nights is never partially generated or filled with an unsafe placeholder.
- **A household member's dietary data changes mid-generation (race with a Household Setup edit)** -- The generation run uses the constraint set read at step 3; if a hard rule tightens after that read but before completion, the newly created plan is immediately re-checked by the mid-week rule-change process (FEAT-01/FEAT-02, XBR-02) the moment it becomes Active, exactly as any other existing plan would be.
- **The AI plan-generation capability returns candidates that all fail the safety check** -- Treated as a generation failure per the Outcome Definitions; the safety check (FEAT-02) is never bypassed to fill a night.
- **Concurrent trigger firing (two schedule fires for the same household at effectively the same time -- e.g., a manual admin retry overlapping the scheduled fire)** -- Only one generation run may be in progress per household at a time; a second trigger for the same household while a run is in flight is treated as the "trigger fires while a previous run is in flight" case below rather than starting a parallel run.
- **Trigger fires while a previous run is in flight** -- The new trigger is deferred until the in-flight run completes (success or failure); if the in-flight run succeeds, the deferred trigger is discarded as redundant for that week; if the in-flight run fails, the deferred trigger proceeds as a retry attempt.
- **The household's plan-arrival time changes (FEAT-07.SPEC-004, Plan-Arrival Day & Time Setting Rule) after this week's generation already ran** -- The change takes effect for the following week's schedule; it does not retroactively re-fire generation for the current week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | Triggers (outbound) | Eligibility and tier gate checked before any generation attempt |
| FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) | Triggers (outbound) | Requests the candidate seven-dinner proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | Triggers (outbound) | Computes the week's estimated total and any over-budget note |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | Triggers (outbound) | Scales each dinner's ingredient quantities to the household |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Displays the generated plan, including the Error/Retry state on failure |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggers (outbound) | Completion fires the plan-ready message |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Completion triggers grocery list recalculation |
| FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout) | References (inbound) | Supplies logged Pantry Items and the matching logic behind step 4's pantry callout |
| FEAT-12.SPEC-005 (Repeated-Dislike Learned Update) | References (inbound) | Supplies Rating history and learned dislikes read at steps 3 and 5 |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | References (inbound) | An organiser hand-over changes who approves the plan this automation produces, without affecting generation itself |

## Analytics and Success Signals

- **plan_generation_started** (household id, scheduled vs. retry) -- N/A -- no Stage 2 metric measures generation start events directly; retained to observe generation volume and retry rate operationally
- **plan_generation_completed** (outcome: on_budget / over_budget, pantry items used count, duration) -- supports success-metrics.md: "Weekly Planning Time"
- **plan_generation_failed** (reason: insufficient_candidates / capability_unavailable) -- supports success-metrics.md: "Weekly Planning Time" (a failure that delays a usable plan works directly against the under-10-minutes goal)
- **plan_weeknight_time_fit** (night, cook_time, within_limit: yes/no) -- supports success-metrics.md: "Weeknight Time-Fit Accuracy"
- **plan_used_pantry_item** (count of pantry items used) -- N/A -- no metric in this feature's connected-metric slice measures pantry usage; success-metrics.md's "Pantry Items Used" metric (Connected Feature: Pantry-Aware Suggestions) is fed within FEAT-05, not here

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Maya's household reaches its configured Sunday-evening plan-arrival time and meets every eligibility prerequisite, when generation fires, then a new Weekly Plan with seven Planned Meals is created, each passing the allergy safety check.

**FEAT-03.SPEC-003-AC-02:** Given a household has one member with a peanut allergy, when generation runs, then every generated dinner excludes recipes containing peanuts, per XBR-01.

**FEAT-03.SPEC-003-AC-03:** Given a household has logged spinach and feta as pantry items, when generation runs and a candidate dinner uses both, then that dinner is weighted into the plan and its Planned Meal carries a pantry callout naming them.

**FEAT-03.SPEC-003-AC-04:** Given a household's weekly_schedule marks Tuesday as a 30-minute night, when generation selects Tuesday's dinner, then its cook_time is at or under that limit.

**FEAT-03.SPEC-003-AC-05:** Given no safe combination of seven dinners fits the household's weekly_budget, when generation completes, then the closest-fitting safe plan is created with an over-budget note attached via FEAT-03.SPEC-006.

**FEAT-03.SPEC-003-AC-06:** Given a household fails FEAT-03.SPEC-009's eligibility check (e.g., no schedule set), when the plan-arrival time arrives, then no new Weekly Plan is created and the outcome is handled by FEAT-03.SPEC-009.

**FEAT-03.SPEC-003-AC-07:** Given the AI plan-generation capability cannot return a usable proposal, when generation runs, then the previous week's plan remains visible on FEAT-03.SPEC-001 with an Error state and Retry control, and no new plan is created.

**FEAT-03.SPEC-003-AC-08:** Given a household retries generation after a failure and the retry succeeds, when the retry completes, then FEAT-03.SPEC-001 shows the new week and the Error state clears.

**FEAT-03.SPEC-003-AC-09:** Given generation succeeds for a household with a prior Active Weekly Plan, when the new plan is created, then the prior week's plan transitions to Archived.

**FEAT-03.SPEC-003-AC-10:** Given generation succeeds, when completion is signaled, then FEAT-07 sends the plan-ready notification and FEAT-06 recalculates the grocery list in the same cycle.

**FEAT-03.SPEC-003-AC-11:** Given a household has rated three past meals down repeatedly, when generation runs, then those meals are weighted away from selection per the learned dislikes supplied by FEAT-12, without ever appearing as a blocked "error."

**FEAT-03.SPEC-003-AC-12:** Given fewer than seven safety-passed candidates exist for a household's combined dietary rules, when generation runs, then the run is treated as a failure and the previous week's plan remains visible rather than a partially filled or unsafe week being created.

**FEAT-03.SPEC-003-AC-13:** Given a second scheduled trigger fires for a household while its prior week's generation run is still in flight, when the second trigger arrives, then it is deferred until the in-flight run completes, and no parallel run for that household starts.

**FEAT-03.SPEC-003-AC-14:** Given two households' plan-arrival times occur at effectively the same moment, when both trigger, then each household's generation runs independently against its own data and neither affects the other's outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (scheduled plan-arrival) | 1 |
| Outcome Paths | 5 (success, success over-budget, not eligible, failure, retry succeeds) | 5 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |



# Automation Spec: First-Plan Generation on Upgrade

## Overview

**Name:** First-Plan Generation on Upgrade
**ID:** FEAT-03.SPEC-004
**Type:** Automation
**Purpose:** System generates a household's very first AI plan immediately after it upgrades to the paid tier.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Firing once, immediately, when a household's subscription transitions from free to paid
- Running the same eligibility check, safety filtering, pantry weighting, budget fit, and scaling logic as the weekly schedule, for this one out-of-cycle run
- Producing the household's first Weekly Plan and seven Planned Meals so the "your first plan is on its way" empty state resolves promptly

**Non-Goals:**
- The recurring weekly generation cycle -- owned by FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), which continues on the household's normal schedule from the following cycle onward
- Processing the upgrade itself (payment, plan selection) -- owned by Subscription & Billing Management (FEAT-14); this automation only reacts to the upgrade's completion
- Re-running for a household that upgrades, downgrades, and upgrades again within the same week -- see Edge Cases for the exact re-trigger behavior, which stays within this spec's scope but does not duplicate FEAT-03.SPEC-003's recurring cadence

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household upgrades from free to paid tier | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires once, immediately, when Subscription.tier transitions to paid and Subscription.billing_state becomes Active; does not fire again for a household that later renews, switches billing period, or re-upgrades after a downgrade within the same billing cycle (see Edge Cases) | Household constraints (weekly_budget, weekly_schedule, member count), all Member Profiles' Dietary Rules, current Pantry Items, Rating history (if any exists from prior free-tier use), the Recipe candidate pool |

## Processing Logic

1. On the upgrade-confirmed event from FEAT-14.SPEC-008 (Apply Subscription Change), check generation eligibility via FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) -- the tier-gating half now passes by definition, but the non-tier prerequisites (complete dietary data, a schedule set) are still checked.
2. If eligibility fails on a non-tier prerequisite, do not generate; the outcome is handled by FEAT-03.SPEC-009 (the household sees its incomplete-setup messaging, not the free-tier placeholder, since it is now paid).
3. If eligible, assemble the household's constraint set exactly as FEAT-03.SPEC-003 does at its step 3: all active Dietary Rules, weekly_schedule, weekly_budget, and member count.
4. Read the current Recipe candidate pool and current Pantry Items (a household may have logged pantry items on the free tier already, since pantry logging is free per XBR-04).
5. Read any Rating history the household accumulated on the free tier via Manual Weekly Planning (FEAT-23.SPEC-001) ratings, if present.
6. Compose the request to the AI plan-generation capability via FEAT-03.SPEC-010 and receive proposed candidates.
7. Pass every candidate through the safety check (FEAT-02.SPEC-002, Candidate Safety Check Execution); exclude failures per XBR-01.
8. Select seven dinners, weighting toward pantry items and any existing ratings, fitting time-constrained nights.
9. Scale quantities via FEAT-03.SPEC-007.
10. Compute budget fit via FEAT-03.SPEC-006, attaching an over-budget note if no safe week fits.
11. Create the household's first Weekly Plan (status: Generated, origin: AI-generated) and seven Planned Meals (status: Proposed). There is no prior Active AI-generated plan to archive; any existing free-tier manually built plan for the current week remains a separate record owned by FEAT-23.SPEC-001 and is not modified by this automation (see Edge Cases).
12. Signal completion to trigger FEAT-07.SPEC-001 (plan-ready notification, using the "first plan" framing) and FEAT-06.SPEC-002 (grocery list recalculation).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| First-plan generation succeeded | Steps 1-12 complete with a full seven-dinner plan fitting the budget | First Weekly Plan (Generated) and seven Planned Meals (Proposed) created | FEAT-03.SPEC-001 resolves from its Empty state to a populated plan; FEAT-07.SPEC-001 sends the plan-ready notification with first-plan framing; FEAT-06.SPEC-002 builds the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002 |
| First-plan generation succeeded, over budget | Steps 1-12 complete, but no safe week fits weekly_budget | Same as above, plus over_budget_note set | FEAT-03.SPEC-001 shows the plan with the over-budget note | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002 |
| Not eligible (non-tier prerequisite unmet) | FEAT-03.SPEC-009's non-tier eligibility check fails | No Weekly Plan created | Handled by FEAT-03.SPEC-009's own outcome definitions; Maya sees what setup is still missing, not the free-tier placeholder | FEAT-03.SPEC-009 |
| Generation failure | The AI plan-generation capability cannot return a usable proposal, or fewer than seven safety-passed candidates exist | No Weekly Plan created | FEAT-03.SPEC-001's Empty state persists with a note that the first plan is taking longer than expected, plus a Retry option | FEAT-03.SPEC-001 |
| Retry succeeds | Maya or the system retries after a failure | Same as "First-plan generation succeeded" | FEAT-03.SPEC-001 resolves to the populated plan | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Subscription -- tier, billing_state (to detect the free-to-paid transition). Household -- weekly_budget, weekly_schedule, member count. Member Profile -- household size context. Dietary Rule -- all active rules. Recipe -- candidate pool. Pantry Item -- Active items. Rating -- any existing free-tier history.
**Creates:** Weekly Plan -- week, origin (AI-generated), status (Generated), estimated_total, over_budget_note (when applicable). Planned Meal -- seven records with the same fields as FEAT-03.SPEC-003 creates.
**Updates:** None -- no prior AI-generated Weekly Plan exists to archive on a household's very first run.
**Deletes:** None.

## Business Rules

- XBR-01: Every candidate dinner passes the app-enforced safety check before selection, identically to the scheduled cycle.
- XBR-05: This automation only fires because the household is now paid; a household that upgrades and immediately downgrades before this run completes is handled per Edge Cases, never left mid-generation on a tier it no longer holds.
- XBR-12: Completion triggers at most one plan-ready message, using first-plan framing distinct from the recurring weekly message (FEAT-07.SPEC-002, Plan-Ready Notification Message, owns the exact wording distinction).
- This automation fires exactly once per upgrade event; it does not replace or pre-empt the household's subsequent regular weekly cycle owned by FEAT-03.SPEC-003, which continues on the household's configured plan-arrival schedule from the following cycle.
- Eligibility, safety filtering, budget fit, and scaling logic are identical to FEAT-03.SPEC-003's -- this spec reuses those rules rather than defining separate ones, so a household's first plan and its subsequent plans behave consistently.

## Edge Cases

- **Household has an existing free-tier manual plan for the current week when it upgrades** -- The manually built Weekly Plan (owned by FEAT-23.SPEC-001) is left untouched; this automation creates a separate, new AI-generated Weekly Plan. FEAT-03.SPEC-001 (now reachable, since the household is paid) shows the new AI-generated plan; the household's prior manual plan remains part of its history (FEAT-19.SPEC-001) and is not merged or overwritten.
- **Household upgrades, then downgrades, then upgrades again within a short window** -- Each free-to-paid transition fires this automation independently; a downgrade that occurs before this automation completes does not cancel an in-flight run (the run still completes, since the household did pay for that period), but no further first-plan run fires for the same household until another genuine free-to-paid transition occurs.
- **Household has already accumulated ratings on the free tier before upgrading** -- Those ratings are read and weighted at step 5, exactly as later cycles would; the household's very first AI plan is not treated as a cold start if preference data already exists.
- **This automation and a scheduled cycle (FEAT-03.SPEC-003) become due for the same household at effectively the same time** -- Only one generation run is in flight per household at a time (the same rule FEAT-03.SPEC-003 applies); if the upgrade-triggered run is already in progress when the schedule would otherwise fire, the schedule's trigger is deferred and, if it becomes redundant once the first-plan run completes for the current week, it is discarded.
- **The AI plan-generation capability is unavailable at the moment of upgrade** -- The Empty state on FEAT-03.SPEC-001 persists with a "taking longer than expected" note and Retry, rather than showing a hard error immediately after a household has just paid.
- **Household upgrades but has incomplete dietary data** -- Generation does not run; FEAT-03.SPEC-009 surfaces what is missing on FEAT-03.SPEC-001, and the household is guided to complete it rather than silently waiting for a plan that will never generate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | Upgrade confirmation fires this automation |
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | Triggers (outbound) | Non-tier eligibility checked before generation |
| FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) | Triggers (outbound) | Requests the candidate seven-dinner proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | Triggers (outbound) | Computes the week's estimated total |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | Triggers (outbound) | Scales ingredient quantities |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Resolves the Empty state to a populated plan, or shows the retry note on failure |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggers (outbound) | Completion fires the first-plan-ready message |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Completion triggers grocery list creation |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | References (inbound) | Any existing free-tier manual plan for the current week is left untouched |

## Analytics and Success Signals

- **first_plan_generation_started** (household id) -- N/A -- no Stage 2 metric measures generation start events directly; retained to observe first-plan latency operationally
- **first_plan_generation_completed** (outcome: on_budget / over_budget, duration) -- supports success-metrics.md: "Weekly Planning Time"
- **first_plan_generation_failed** (reason) -- supports success-metrics.md: "Weekly Planning Time"
- **first_plan_weeknight_time_fit** (night, cook_time, within_limit: yes/no) -- supports success-metrics.md: "Weeknight Time-Fit Accuracy"

## Acceptance Criteria

**FEAT-03.SPEC-004-AC-01:** Given Maya's household completes its upgrade to paid and meets every non-tier eligibility prerequisite, when the upgrade is confirmed, then this automation fires immediately and creates the household's first Weekly Plan with seven safety-passed Planned Meals.

**FEAT-03.SPEC-004-AC-02:** Given Maya's household upgrades but has an incomplete member's dietary data, when the upgrade is confirmed, then no plan is generated and FEAT-03.SPEC-001 shows what is still missing, per FEAT-03.SPEC-009.

**FEAT-03.SPEC-004-AC-03:** Given the household's first-plan generation succeeds, when completion is signaled, then FEAT-07 sends a plan-ready message using first-plan framing and FEAT-06 builds the grocery list.

**FEAT-03.SPEC-004-AC-04:** Given no safe combination of seven dinners fits the household's weekly_budget on its first generation, when generation completes, then the closest-fitting safe plan is created with an over-budget note.

**FEAT-03.SPEC-004-AC-05:** Given the AI plan-generation capability is unavailable at the moment of upgrade, when generation is attempted, then FEAT-03.SPEC-001's Empty state shows a "taking longer than expected" note with Retry, rather than a hard failure.

**FEAT-03.SPEC-004-AC-06:** Given a household has an existing free-tier manual plan for the current week when it upgrades, when this automation completes, then the manual plan remains unchanged and a separate AI-generated Weekly Plan is created and displayed.

**FEAT-03.SPEC-004-AC-07:** Given a household accumulated ratings while on the free tier, when its first AI plan generates, then those ratings weight the selection exactly as they would in a later scheduled cycle.

**FEAT-03.SPEC-004-AC-08:** Given a household's first-plan generation is already in flight when its regular scheduled generation would also become due, when the schedule trigger arrives, then it is deferred and discarded once the first-plan run completes for that week, rather than starting a second parallel run.

**FEAT-03.SPEC-004-AC-09:** Given a household upgrades, downgrades, and upgrades again, when each free-to-paid transition completes, then this automation fires independently for each genuine transition, without duplicating a run already completed for the same period.

**FEAT-03.SPEC-004-AC-10:** Given Maya retries generation after a first-plan failure and the retry succeeds, when the retry completes, then FEAT-03.SPEC-001 shows the populated plan and the retry note clears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (upgrade confirmed) | 1 |
| Outcome Paths | 5 (success, success over-budget, not eligible, failure, retry succeeds) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



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



# Logic/Rule Spec: Budget Fit & Estimated Total Rule

## Overview

**Name:** Budget Fit & Estimated Total Rule
**ID:** FEAT-03.SPEC-006
**Type:** Logic/Rule
**Purpose:** Computes the week's estimated cost against the household budget and determines the closest-fitting plan with an overrun note when no safe week fits.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Weekly Plan (estimated_total, over_budget_note fields)

## Scope and Non-Goals

**In Scope:**
- Computing a Weekly Plan's estimated_total from its seven Planned Meals' rough_cost figures
- Comparing estimated_total against the household's weekly_budget
- Selecting the closest-fitting safe combination and attaching over_budget_note when no safe combination fits
- Currency and unit display consistency for the computed total

**Non-Goals:**
- Setting the household's weekly_budget itself -- owned by Household Setup & Member Profiles (FEAT-01); this rule only reads that value
- Computing each recipe's rough_cost -- owned by Recipe Library (FEAT-08) and Recipe Import (FEAT-10) at the recipe level; this rule sums already-computed per-dinner costs after they are sized to the household by FEAT-03.SPEC-007
- Enforcing the allergy/religious-rule safety check on candidates -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02); this rule operates only on the already safety-passed candidate pool handed to it during generation

## Governed Entity

**Entity:** Weekly Plan
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The calendar week the plan covers |
| origin | enum | AI-generated or manually built |
| status | enum | Generated/Started, Reviewed, Approved, Active, Archived |
| approval | derived | Organiser approval (once per week) or auto-adoption at week start |
| estimated_total | number | The week's estimated cost against the household budget, in the household's currency -- governed by this spec |
| over_budget_note | text | Shown when no safe week fits the budget -- governed by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | During candidate selection, to compute estimated_total and decide whether an over-budget fallback is needed, before the Weekly Plan is created |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Same enforcement point, for a household's first generation run |
| FEAT-03.SPEC-001 | Weekly Plan View | Displays estimated_total and over_budget_note verbatim in the weekly total banner; performs no independent calculation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| estimated_total | Must be a positive number, sum of the seven Planned Meals' household-scaled rough_cost values | Always | On computation, during generation | No validation blocking is applicable -- this is a computed field, not user input | No |
| over_budget_note | No validation beyond data type -- either absent (plan fits budget) or a plain-language overrun note | Always | On computation, during generation | -- | -- |
| weekly_budget (read from Household) | Must be a positive amount in the household's configured currency; if entirely unset, generation treats budget fit as unconstrained until the household sets one (FEAT-01, Validation & Limits: "weekly budget must be a positive amount... optional during partial setup") | Household has not yet set a budget | Read at generation time | N/A -- this field is validated by FEAT-01, not this spec; this spec only reads its current value | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Budget comparison | estimated_total, weekly_budget (Household) | estimated_total is compared against weekly_budget; if estimated_total exceeds weekly_budget for every safety-passed seven-dinner combination available, the closest-fitting combination is selected and over_budget_note is set | N/A -- no error is raised; this produces a note, not a validation failure, since the household must always end up with a plan (XBR-07) |
| Over-budget note presence | estimated_total, over_budget_note, weekly_budget | over_budget_note is present if and only if estimated_total exceeds weekly_budget; it is absent whenever a safe, on-budget combination was selected | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the budget-fit computation | System (invoked by FEAT-03.SPEC-003, FEAT-03.SPEC-004 during generation) | Always, as part of generation | N/A -- no user-facing action exists to deny; this rule is invoked internally, not by direct user action |
| View estimated_total and over_budget_note | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always, wherever the Weekly Plan is displayed (FEAT-03.SPEC-001), per each role's View access to Weekly Plan | -- |
| View estimated_total and over_budget_note | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| View estimated_total and over_budget_note | Riley (Operator, support -- from v1) | Only while a Support Request for the household is open (FEAT-22, XBR-14) | Outside an open Support Request, Riley has no access to any household screen showing this data |
| Change weekly_budget (the input this rule reads) | Maya (Organiser) | Always | -- |
| Change weekly_budget | Sam (Other Adult Member), both Jordan rows, Riley | Never -- Household Setup is View or None for these roles per the Access Matrix | The budget field is not editable for these roles; Household Setup (FEAT-01) shows it read-only or hides the control entirely |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| estimated_total | Sum of the seven selected Planned Meals' rough_cost (each already scaled to household size by FEAT-03.SPEC-007) | On generation (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | No -- always derived; it is never directly edited |
| over_budget_note | Set to a plain-language overrun statement (e.g., naming the estimated overrun amount) when estimated_total exceeds weekly_budget for every safety-passed combination considered; otherwise absent | On generation | No -- always derived |
| Currency and unit display of estimated_total | Rendered in the household's configured currency (FEAT-16) | On display (FEAT-03.SPEC-001) | No -- households change their currency setting through FEAT-16, not by overriding this display |

## Business Rules

- XBR-07 and XBR-11 govern the surrounding context: the plan always reaches an approvable or auto-adoptable state (this rule never blocks generation, only annotates it), and the total displays in the household's configured units and currency, converting for display if the household's locale settings change later.
- When no safe week fits the budget, the household receives the closest-fitting plan with a plain overrun note, never a silent overspend (product-features.md, FEAT-03 Primary Flows & Alternates).
- estimated_total is always computed after FEAT-03.SPEC-007's household-scaling, since an unscaled cost figure would not reflect what the household will actually spend.
- A household with no weekly_budget set yet (partial setup) receives a plan with estimated_total computed and displayed, but no over_budget_note is ever attached, since there is no budget to compare against.

## Edge Cases

- **Household has not yet set a weekly_budget** -- estimated_total is still computed and shown; over_budget_note is never attached in this case, since there is nothing to measure an overrun against.
- **estimated_total exactly equals weekly_budget** -- Treated as on-budget; over_budget_note is not attached (the comparison is "exceeds," not "meets or exceeds").
- **Every safety-passed candidate combination exceeds the budget by a large margin** -- The closest-fitting combination (smallest overrun) is still selected; over_budget_note states the estimated overrun in plain language rather than refusing to produce a plan.
- **Household changes its weekly_budget mid-week after a plan has already generated** -- The change applies to the household's next generation cycle; the current week's already-computed estimated_total and over_budget_note are not retroactively recalculated, consistent with FEAT-01's "later edit" behavior (changes apply to the next plan, not retroactively).
- **Household's currency setting changes mid-week (FEAT-16)** -- The existing estimated_total is converted for display in the new currency per XBR-11, without changing the underlying computed value's basis.
- **A single very expensive dinner makes the week over budget even though six other dinners are inexpensive** -- The rule operates on the total across all seven dinners, not per-dinner limits; a per-dinner cost is shown for transparency, but budget fit is judged only at the weekly total.

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given a household's seven selected dinners sum to less than its weekly_budget, when generation computes the total, then estimated_total is set to that sum and over_budget_note is absent.

**FEAT-03.SPEC-006-AC-02:** Given no safe combination of seven dinners fits the household's weekly_budget, when generation runs, then the closest-fitting safe combination is selected and over_budget_note states the estimated overrun in plain language.

**FEAT-03.SPEC-006-AC-03:** Given a household has not yet set a weekly_budget, when generation computes estimated_total, then the total is still shown and no over_budget_note is attached.

**FEAT-03.SPEC-006-AC-04:** Given a household's seven dinners sum to exactly its weekly_budget, when the comparison runs, then the plan is treated as on-budget and no over_budget_note appears.

**FEAT-03.SPEC-006-AC-05:** Given Maya views the Weekly Plan View, when the weekly total banner renders, then it shows estimated_total in the household's configured currency.

**FEAT-03.SPEC-006-AC-06:** Given Sam views the Weekly Plan View, when the weekly total banner renders, then he sees the same estimated_total and over_budget_note as Maya, consistent with his View access to Weekly Plan.

**FEAT-03.SPEC-006-AC-07:** Given Riley (Operator) has no open Support Request for a household, when Riley attempts to view that household's plan, then no screen showing estimated_total is reachable.

**FEAT-03.SPEC-006-AC-08:** Given Maya changes the household's weekly_budget mid-week, when the change is saved, then the current week's already-computed estimated_total and over_budget_note remain unchanged, and the new budget applies starting the next generation cycle.

**FEAT-03.SPEC-006-AC-09:** Given the household changes its currency setting (FEAT-16) mid-week, when the plan is next displayed, then estimated_total is converted for display in the new currency without recomputing the underlying total.

**FEAT-03.SPEC-006-AC-10:** Given Sam attempts to change the household's weekly_budget, when he looks for an editable budget control, then none is available to him, per his View access to Household Setup.

**FEAT-03.SPEC-006-AC-11:** Given a plan has one very expensive dinner among six inexpensive ones and the total still fits the budget, when generation computes estimated_total, then no over_budget_note is attached, since fit is judged on the weekly total, not per-dinner cost.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Household-Scaled Quantity Rule

## Overview

**Name:** Household-Scaled Quantity Rule
**ID:** FEAT-03.SPEC-007
**Type:** Logic/Rule
**Purpose:** Derives per-dinner ingredient quantities sized to the number of people the household is planning for.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Planned Meal (cook_time, rough_cost fields, and the scaled ingredient quantities that feed the Grocery List)

## Scope and Non-Goals

**In Scope:**
- Scaling a candidate recipe's ingredient quantities to the household's member count for each of the seven Planned Meals created at generation
- Deriving the household-scaled rough_cost shown per dinner and fed into FEAT-03.SPEC-006's budget computation
- Ensuring scaled quantities carry through consistently to the Grocery List

**Non-Goals:**
- Defining a recipe's base (unscaled) ingredient list, cook_time, or base cost -- owned by Recipe Library (FEAT-08) and Recipe Import (FEAT-10); this rule only transforms those base figures for the household
- Combining scaled quantities across multiple dinners into single grocery-list lines -- owned by Shared Grocery List (FEAT-06), which consumes this rule's per-dinner output
- Determining which household members count toward the scaling number -- household member count itself is owned by Household Setup & Member Profiles (FEAT-01); this rule only reads that count

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | enum | The day of the week; at most one dinner per night |
| meal_kind | enum | Dinner, or leftover lunch linked to one source dinner |
| recipe | reference | The chosen Recipe |
| safety_badge | text | "Checked against allergies" plus the "always check labels" disclaimer |
| vegetarian_option | boolean | Whether a shared meal carries a vegetarian variant |
| cook_time | number | Carried from the recipe -- not scaled by household size |
| rough_cost | number | Carried from the recipe, sized for the household -- governed by this spec |
| pantry_callout | text | Which logged pantry items this dinner uses |
| status | enum | Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked |
| swap_history | list | Prior recipes in this slot |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | During candidate selection, after safety filtering and before budget-fit computation, for each of the seven selected dinners |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Same enforcement point, for a household's first generation run |
| FEAT-03.SPEC-006 | Budget Fit & Estimated Total Rule | Consumes this rule's household-scaled rough_cost as its input for estimated_total |
| FEAT-06 | Shared Grocery List | Consumes this rule's scaled ingredient quantities when building the week's list |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| rough_cost (scaled) | Must be a positive number, derived from the recipe's base rough_cost scaled to household member count | Always | On computation, during generation | N/A -- computed field, not user input | No |
| cook_time | No validation beyond data type -- carried from the recipe unscaled, since cook time does not change with portion count | Always | -- | -- | -- |
| Scaled ingredient quantities (feed Grocery List, not a Planned Meal field directly) | Must be positive, non-zero quantities for every ingredient the recipe defines, scaled to household member count | Always | On computation, during generation | N/A -- computed, not user input | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Cost scales with quantity | rough_cost, household member count | rough_cost scales proportionally with the same scaling factor applied to ingredient quantities, so a doubled household sees roughly double the per-dinner cost | N/A |
| Cook time does not scale | cook_time, household member count | cook_time remains the recipe's base value regardless of household size, since preparation time does not scale linearly with portion count | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the scaling computation | System (invoked by FEAT-03.SPEC-003, FEAT-03.SPEC-004 during generation) | Always, as part of generation | N/A -- invoked internally, not by direct user action |
| View scaled cook_time and rough_cost | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always, wherever the Weekly Plan is displayed (FEAT-03.SPEC-001) | -- |
| View scaled cook_time and rough_cost | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| View scaled cook_time and rough_cost | Riley (Operator, support -- from v1) | Only while a Support Request for the household is open (FEAT-22, XBR-14) | Outside an open Support Request, no access to any household screen showing this data |
| Change the household member count this rule scales to | Maya (Organiser) | Always, through adding or removing Member Profiles (FEAT-01) | -- |
| Change the household member count | Sam, both Jordan rows, Riley | Never -- Household Setup is View or None for these roles | Member management controls are not shown to these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Scaled ingredient quantities | Recipe's base ingredient quantities multiplied by a scaling factor derived from the household's current member count | On generation, per selected dinner | No -- always derived; a household changes the outcome only by changing its member count (FEAT-01) |
| rough_cost (scaled) | Recipe's base rough_cost multiplied by the same scaling factor applied to ingredient quantities | On generation, per selected dinner | No -- always derived |
| cook_time | Recipe's base cook_time, unscaled | On generation, per selected dinner | No -- cook_time is never scaled |

## Business Rules

- Ingredient quantities are sized for the number of people eating, so the grocery list buys the right amount (product-features.md, FEAT-03 Key Capabilities: "Scale to the household").
- Scaling always precedes the budget-fit computation (FEAT-03.SPEC-006), since an unscaled cost would misstate what the household will actually spend.
- Scaled quantities feed the Grocery List (FEAT-06) directly and consistently -- a household that changes size before its next generation sees the new size reflected starting with that cycle, not retroactively on the current week's already-built list (mirroring FEAT-01's "later edit" rule).
- Household member count for scaling includes every active Member Profile the household is currently planning meals for, consistent with the count Household Setup (FEAT-01) maintains.

## Edge Cases

- **A recipe's base ingredient list includes an ingredient with no meaningful fractional scaling (e.g., "1 lemon")** -- The scaled quantity rounds to the nearest sensible whole unit for that ingredient type rather than producing a fractional or zero quantity; the exact rounding convention per ingredient type is a Recipe Library (FEAT-08) data concern, not this rule's.
- **Household size changes (a member is added or removed) after this week's plan has already generated** -- The current week's already-scaled quantities and rough_cost are not retroactively recalculated; the new member count applies starting the household's next generation cycle, consistent with FEAT-01's later-edit behavior.
- **Household has only one member** -- Scaling still applies; a scaling factor of one produces the recipe's base quantities and cost unchanged.
- **Household is at its maximum of 12 member profiles** -- Scaling applies the same proportional logic at the upper bound as at any other household size; no separate cap or different formula applies.
- **A recipe carries a vegetarian_option variant with different ingredients from the main dish** -- Both the main and vegetarian_option ingredient sets are scaled independently to the respective number of people eating each variant, so the grocery list buys the right amount of each.

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given a household of four people, when generation selects a dinner whose base recipe serves two, then the Planned Meal's ingredient quantities and rough_cost are scaled to four servings.

**FEAT-03.SPEC-007-AC-02:** Given a household of four people, when generation selects a dinner, then its cook_time is shown unchanged from the recipe's base cook_time.

**FEAT-03.SPEC-007-AC-03:** Given a household with exactly one member, when generation selects a dinner, then its scaled quantities and rough_cost equal the recipe's base values.

**FEAT-03.SPEC-007-AC-04:** Given a household with 12 member profiles (the maximum), when generation selects a dinner, then quantities and cost scale proportionally using the same logic as any other household size.

**FEAT-03.SPEC-007-AC-05:** Given Maya adds a new member to the household mid-week, when the change is saved, then the current week's already-generated Planned Meals keep their existing scaled quantities, and the new member count applies starting the next generation cycle.

**FEAT-03.SPEC-007-AC-06:** Given a dinner carries a vegetarian_option variant, when generation scales the dinner, then the main dish and the vegetarian variant are each scaled to their respective number of people eating.

**FEAT-03.SPEC-007-AC-07:** Given Maya views the Weekly Plan View, when a dinner card renders, then the shown cook_time and rough_cost reflect this rule's household-scaled output.

**FEAT-03.SPEC-007-AC-08:** Given Sam views the Weekly Plan View, when a dinner card renders, then he sees the same scaled cook_time and rough_cost as Maya.

**FEAT-03.SPEC-007-AC-09:** Given Riley (Operator) has no open Support Request for a household, when Riley attempts to view that household's plan, then no screen showing scaled quantities or cost is reachable.

**FEAT-03.SPEC-007-AC-10:** Given the household's grocery list is built from the current week's plan, when it is generated, then it uses this rule's household-scaled ingredient quantities, not the recipe's unscaled base quantities.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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



# Integration Spec: AI Plan Generation Capability Integration

## Overview

**Name:** AI Plan Generation Capability Integration
**ID:** FEAT-03.SPEC-010
**Type:** Integration
**Purpose:** Product boundary to the AI text/plan-generation capability that turns household constraints and candidate recipes into a proposed seven-dinner week.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Sending household constraints and candidate recipe data to the AI plan-generation capability and receiving a proposed set of candidate dinners
- The product's behavior when the capability is slow, unavailable, or returns a proposal the product cannot use
- Disclosure of what household data is shared with this capability
- Feeding the raw proposal into the safety check, scaling rule, and budget-fit rule owned elsewhere in this feature

**Non-Goals:**
- Choosing the AI vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific provider
- Verifying the safety of proposed candidates -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02); this spec treats the capability's output as unverified until FEAT-02's check runs
- Scaling quantities or computing budget fit on the proposal -- owned by FEAT-03.SPEC-007 and FEAT-03.SPEC-006 respectively, which run after this integration returns its result
- Swap-time alternative generation -- FEAT-04's own Integration spec (FEAT-04.SPEC-007) covers the AI capability's use during a single-slot swap; this spec covers only full-week generation (FEAT-03.SPEC-003, FEAT-03.SPEC-004)

## Capability Category

**Category:** AI text/plan generation
**Dependency Source:** ASMP-30 -- "AI text/plan-generation capability -- Required to generate the weekly dinner plan and swap alternatives on the paid tier" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "AI text/plan generation (ASMP-30)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-03, FEAT-04; Integration Specs: FEAT-03.SPEC-010, FEAT-04.SPEC-007)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya's household receives a proposed seven-dinner week that respects every hard dietary rule, the stated schedule, and the budget | Generate the week's plan | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) |
| The proposal favors recipes that use ingredients the household has already logged as on hand | Use up the pantry | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Later proposals favor meals the household has rated highly and avoid ones rated poorly | Learn from ratings over time | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Household constraints | Household -- weekly_budget, weekly_schedule; Member Profile -- count of active members | Generation is triggered (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | The capability needs to know the budget ceiling, time constraints per night, and how many people the week must feed |
| Dietary constraints | Dietary Rule -- rule_kind, strength, allergen (per active member, without member-identifying detail beyond what is needed to apply the rule) | Every generation attempt | The capability must avoid proposing candidates that could not pass the household's hard rules, even though FEAT-02's app-side check remains the enforced safety boundary |
| Candidate recipe pool | Recipe -- name, ingredients, cook_time, rough_cost, dietary_badges (starter and imported recipes available to the household) | Every generation attempt | The capability selects and arranges dinners from this pool rather than inventing recipes outside the household's verified candidate set |
| Pantry weighting input | Pantry Item -- item_name (Active items only) | Every generation attempt, paid tier only | The capability weights candidate selection toward dinners that use these ingredients |
| Rating weighting input | Rating -- planned_meal reference, value (thumbs up/down), aggregated per recipe, not per individual member | Every generation attempt | The capability weights candidate selection toward highly-rated meals and away from poorly-rated ones |

Member names, sign-in details, payment information, household budget beyond the weekly figure, and any data belonging to other households never leave the product. Individual members' ratings are never sent broken out by member -- only aggregated per recipe, consistent with product-features.md's Rating Data Sensitivity note that individual ratings are never shown broken out to other members.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Proposed candidate dinners (up to and including seven selections, or a candidate set the household's own selection logic narrows to seven) | The capability returns a generation result | Held transiently as generation working data; each selected candidate becomes a new Planned Meal (recipe, night, meal_kind) only after passing FEAT-02's safety check |
| Generation confidence/coverage signal (e.g., whether the capability found a full week of proposals or a partial set) | The capability returns a generation result | Read by FEAT-03.SPEC-003/004's processing logic to decide between a normal completion and a generation-failure outcome |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Proposal returned, full week available | The capability returns seven or more safety-passable candidates | None directly (the receiving automation processes the proposal into Planned Meals after FEAT-02's check) | None at this boundary -- feedback is owned by the receiving automation's own outcome (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Proposal returned, partial or insufficient | The capability returns fewer usable candidates than needed to fill seven nights after safety filtering | None | The receiving automation treats this as a generation failure per its own Outcome Definitions | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Capability request times out | No response is received within the product's expected generation window | None | The receiving automation treats this as a generation failure | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Capability reports an error | The capability responds with an explicit processing error | None | The receiving automation treats this as a generation failure | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | The Loading state's explained-wait message continues to show ("Building this week's plan...") for up to the product's expected generation window (well under a minute, per ASMP-23); if the wait extends meaningfully beyond that, the screen shows "This is taking longer than usual" while continuing to wait rather than failing immediately. | The previous week's plan remains fully visible with the Error banner "This week's plan couldn't be generated. Retry?" and a Retry control; the household is never shown a blank week. | Same as Capability Down -- a rejected request (e.g., malformed constraints) is treated identically to an unavailable capability from the household's point of view: the previous plan stays visible with Retry offered. |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | N/A -- this screen never sends a request to the capability; a free-tier household is routed here before any request is made. | N/A -- see above. | N/A -- see above. |

## Consent and Disclosure

- **Generation data-sharing notice** -- Shown once, the first time a household's plan is ever generated (surfaced during FEAT-03.SPEC-004's first-plan flow or, for a household that never previously saw it, alongside its first scheduled generation): "To build your weekly plan, we share your household's dietary rules, budget, schedule, pantry items, and meal ratings with an AI planning service. Recipes and their sources are also shared. Your name, sign-in details, and payment information are never shared." A "How your plan is built" link on FEAT-03.SPEC-001 reopens the same notice at any time; no "cancel" option is offered since generation is core to the paid tier the household has subscribed to, but the household may downgrade (FEAT-14) if it does not want data shared this way.
- **What is never shared** -- Member names, sign-in details, payment information, and any other household's data are stated in the same notice as never leaving the product; individual, member-attributed ratings are also confirmed as never shared -- only aggregated per-recipe ratings.

## Edge Cases

- **Capability returns a proposal referencing a recipe that was removed from the household's pool between candidate-pool read and response** -- The receiving automation discards that candidate before selection; if this drops the usable pool below seven, it is treated as a partial/insufficient result per Inbound Events.
- **The same generation request is somehow submitted twice (e.g., a retry racing an already-in-flight request)** -- Only one generation run is in flight per household at a time, per FEAT-03.SPEC-003's own concurrency rule; a duplicate submission for the same household and week is not sent while a prior request is outstanding.
- **Capability response arrives after the household has already been shown a generation-failure state (very late response)** -- A late-arriving success response for a request the household has already been told failed is discarded; the household's next attempt (manual retry or next scheduled cycle) issues a fresh request rather than relying on the stale one.
- **Capability goes down mid-request, after constraints were sent but before a response returns** -- No Weekly Plan or Planned Meal record is created from a partial exchange; the household sees the Capability Down messaging with no half-created plan.
- **A household's candidate recipe pool is very small (e.g., a brand-new household with only starter recipes and one allergy)** -- The capability may return a valid but repetitive proposal; this is accepted as a full week if it passes safety and fills seven nights, since recipe pool variety is Recipe Library's (FEAT-08) concern, not this integration's.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Initiates a generation request each scheduled cycle |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | Initiates a generation request on upgrade |
| FEAT-03.SPEC-003 / FEAT-03.SPEC-004 | Affects (outbound) | Both automations' Outcome Definitions depend on this integration's response, timeout, or error |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | References (outbound) | Every proposal this integration returns is subject to this safety check before use |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | References (outbound) | Runs on the safety-passed subset of this integration's proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | References (outbound) | Runs on the scaled subset of this integration's proposal |
| FEAT-04.SPEC-007 (Swap Alternatives Generation) | References (sibling) | Covers the same capability category for single-slot swap alternatives; this spec covers full-week generation only |
| FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout) | References (inbound) | Supplies the pantry weighting input this integration sends |
| FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) | References (inbound) | Supplies the aggregated rating weighting input this integration sends |

## Analytics and Success Signals

- **ai_generation_requested** (household id, candidate pool size) -- N/A -- no Stage 2 metric measures request volume directly; retained to observe generation demand and capability usage operationally
- **ai_generation_response_received** (outcome: full_week / partial / timeout / error, latency) -- supports success-metrics.md: "Weekly Planning Time" (latency here directly composes the under-10-minutes-a-week goal)
- **ai_generation_degradation_shown** (condition: slow / down / rejected; screen: FEAT-03.SPEC-001) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-03.SPEC-010-AC-01:** Given a household's generation is triggered, when this integration sends its request, then it includes weekly_budget, weekly_schedule, member count, active dietary rules, the candidate recipe pool, current pantry items, and aggregated ratings -- and nothing else.

**FEAT-03.SPEC-010-AC-02:** Given the capability returns seven or more safety-passable candidates, when the response arrives, then the receiving automation proceeds to safety filtering and selection.

**FEAT-03.SPEC-010-AC-03:** Given the capability returns fewer usable candidates than needed to fill seven nights after safety filtering, when the response arrives, then the receiving automation treats this as a generation failure.

**FEAT-03.SPEC-010-AC-04:** Given the capability does not respond within the product's expected generation window, when the timeout is reached, then the household sees FEAT-03.SPEC-001's Error state with Retry, and the previous week's plan remains visible.

**FEAT-03.SPEC-010-AC-05:** Given the capability is slow but within the expected window, when the household is on FEAT-03.SPEC-001, then it continues to show the explained-wait Loading state without an error.

**FEAT-03.SPEC-010-AC-06:** Given the capability rejects a generation request, when the rejection is received, then the household experiences the same Retry-offering messaging as a Capability Down condition, with the previous plan unchanged.

**FEAT-03.SPEC-010-AC-07:** Given a household generates its plan for the first time, when generation is about to run, then the household is shown the generation data-sharing notice naming exactly what is shared and what never leaves the product.

**FEAT-03.SPEC-010-AC-08:** Given Maya wants to review what data her plan generation shares, when she taps "How your plan is built" on FEAT-03.SPEC-001, then the same disclosure notice reopens.

**FEAT-03.SPEC-010-AC-09:** Given the capability's response references a recipe removed from the household's pool after the request was sent, when the automation processes the response, then that candidate is discarded before selection.

**FEAT-03.SPEC-010-AC-10:** Given the capability goes down after receiving a request but before returning a response, when the household checks its plan, then no partial Weekly Plan or Planned Meal record exists, and the Capability Down messaging is shown.

**FEAT-03.SPEC-010-AC-11:** Given a late response arrives for a request the household has already been shown as failed, when it is received, then it is discarded and does not silently create a plan behind the household's back.

**FEAT-03.SPEC-010-AC-12:** Given individual members' ratings exist for the household, when this integration sends rating data, then only aggregated per-recipe values are sent, never a rating attributed to a specific member.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (FEAT-03.SPEC-001 x 3 conditions; FEAT-03.SPEC-002 N/A cells excluded) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



# Integration Spec: Real-Time Plan Sync Integration

## Overview

**Name:** Real-Time Plan Sync Integration
**ID:** FEAT-03.SPEC-011
**Type:** Integration
**Purpose:** Product boundary to the real-time data-synchronization capability that reflects generation, approval, and plan changes live across every household member's device.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Propagating Weekly Plan and Planned Meal changes (generation, approval, auto-adoption, accepted swaps, safety removals) live to every household member's device viewing FEAT-03.SPEC-001
- Reconciling a device's plan view after it reconnects following an offline period
- The product's behavior when the sync capability is slow, unavailable, or rejects an update

**Non-Goals:**
- Synchronizing the Grocery List -- owned by Shared Grocery List's own Integration spec (FEAT-06.SPEC-005), which covers the same capability category for list data; this spec covers only Weekly Plan and Planned Meal data
- The swap, approval, or safety-removal logic itself -- owned by FEAT-04, FEAT-03.SPEC-008, and FEAT-02 respectively; this spec only propagates their already-decided outcomes
- Offline creation of a new Weekly Plan -- generation requires connectivity (FEAT-03.SPEC-001's States); this spec covers only reflecting a plan that some connected process has already changed

## Capability Category

**Category:** Real-time data synchronization
**Dependency Source:** ASMP-35 -- "Real-time data-synchronization capability -- Required for the shared grocery list and plan to update live across household members' devices and to reconcile changes made while offline" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Real-time data synchronization (ASMP-35)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-06, FEAT-03, FEAT-04, FEAT-23; Integration Specs: FEAT-03.SPEC-011 (live plan sync), FEAT-06.SPEC-005 (live grocery list sync and offline reconciliation))
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Every household member sees the same current week's plan update the moment it changes, without a manual refresh | Approve the week's plan (and see the result of others' actions live) | FEAT-03.SPEC-001 (Weekly Plan View) |
| A household member who was offline sees the plan reconcile correctly to the latest state once reconnected | Never show a stale or conflicting plan | FEAT-03.SPEC-001 |
| The organiser sees a swap suggestion, an accepted swap, a safety removal, or auto-adoption reflected live while reviewing the week | Approve the week's plan, including any swap suggestions from other adults | FEAT-03.SPEC-001 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Weekly Plan change | Weekly Plan -- status, approval, estimated_total, over_budget_note | Generation completes, approval is recorded, or auto-adoption fires | Propagates the plan's current state to every household member's connected device |
| Planned Meal change | Planned Meal -- recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status, swap_history | A dinner is created, swapped, or removed for safety | Propagates the affected dinner's current state to every household member's connected device |

Only Weekly Plan and Planned Meal fields already defined in the dependency map are sent; no member-identifying data beyond what already appears on these entities (e.g., swap_history's prior recipes) leaves the product through this channel, and no data belonging to another household is ever included.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Sync acknowledgement / conflict signal | The capability confirms a change was propagated, or reports that a device's locally queued change conflicts with a newer server-side state | Weekly Plan / Planned Meal -- no field changes from the acknowledgement itself; a conflict signal is handled per the dependency map's Contention resolution (reject-with-refresh for plan slot changes, safety removal always wins) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Plan or meal update propagated | Any connected device's Weekly Plan or Planned Meal state changes from any source (generation, approval, auto-adoption, swap, safety removal) | The receiving device's local view updates to match | The affected dinner card or banner updates in place on FEAT-03.SPEC-001, with a brief highlight; no full-screen reload | FEAT-03.SPEC-001 |
| Reconnection reconciliation | A device that was offline regains connectivity | The device's local plan view is reconciled to the latest server-side state; any of that device's own queued actions (e.g., a queued rating or leftover-lunch response) are resubmitted | The plan view updates silently to the current state; a queued action's outcome (success or conflict) surfaces per its own owning spec (FEAT-11, FEAT-12) | FEAT-03.SPEC-001 |
| Sync conflict reported | Two devices attempt conflicting changes to the same Planned Meal slot at effectively the same time | No plan data change from this event alone -- the conflict is resolved per the dependency map's Contention note (reject-with-refresh per slot; safety removal always wins) and the losing device's action is rejected | The losing device sees "This dinner just changed -- here's the latest." per FEAT-03.SPEC-001's Edge Cases | FEAT-03.SPEC-001, FEAT-04 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | Updates from other devices may take longer than the near-instant window (ASMP-22) to appear; no error is shown, and the screen continues to reflect its last-known state while waiting. | The screen falls back to its last-synced state and shows the Offline/Degraded messaging described in FEAT-03.SPEC-001's States table: "Reconnect to approve or change the plan." Viewing the already-loaded plan remains fully available. | A rejected local action (e.g., a swap attempt that conflicts with a newer server state) is treated per the Contention resolution: the action fails with "This dinner just changed -- here's the latest." and the screen refreshes to the current state; no other action on the screen is blocked by one rejected update. |

## Consent and Disclosure

- **No separate disclosure needed** -- Weekly Plan and Planned Meal data already displayed to every household member on FEAT-03.SPEC-001 is the same data propagated by this integration; since every household member already sees this data in-app, no additional consent moment is required beyond the household's general data-sharing posture (ASMP-14, ASMP-26: household data is never sold or used for advertising, and stays private to the household). This integration moves data only between the household's own devices, never to a party outside the household.
- **What is never shared** -- No plan or meal data is sent to any device outside the household's own member sessions; Riley's read-only support access (FEAT-22) is a separate, permissioned path and not part of this synchronization channel.

## Edge Cases

- **Two household members swap the same dinner slot at effectively the same moment on different devices** -- Per the dependency map's Contention note for Weekly Plan (reject-with-refresh per night slot, no more than one active swap operation per slot), the first swap to be confirmed wins; the second device's attempt is rejected and its screen refreshes to show the confirmed recipe.
- **A safety removal and a swap acceptance target the same slot concurrently** -- The safety removal always wins over any concurrent change, per the dependency map's Contention note; the swap acceptance is rejected and the device is shown the safety-driven state instead.
- **A device is offline for an extended period spanning a full plan regeneration (a new week generated while offline)** -- On reconnection, the device reconciles directly to the new week's plan; it does not attempt to replay stale actions against the now-archived prior week.
- **Sync event arrives for a Weekly Plan that has since been archived (a new week already generated)** -- The event is discarded for display purposes on the current-week view; the archived week's own record (FEAT-19) retains the historical state it had at archival.
- **The same plan-update event is delivered to a device twice** -- The second delivery changes nothing further, since the device's local state already matches; no duplicate highlight animation or duplicate toast fires.
- **Capability goes down mid-approval** -- If the approval write was not confirmed as propagated, the approving device shows the Offline/Degraded messaging and the approval is retried once connectivity returns, rather than leaving other devices with a stale unapproved view indefinitely.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Live-updates this screen whenever the plan changes from any source |
| FEAT-03.SPEC-005 (Auto-Adoption at Week Start) | Triggers (outbound) | Adoption's status change is propagated via this integration |
| FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | Triggers (outbound) | Approval's status change is propagated via this integration |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggers (outbound) | Accepted swaps and suggestions propagate via this integration |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound) | Safety removals propagate via this integration and always win conflicts |
| FEAT-06.SPEC-005 (Shared Grocery List's live sync integration) | References (sibling) | Covers the same capability category for Grocery List data; this spec covers Weekly Plan and Planned Meal data only |

## Analytics and Success Signals

- **plan_sync_update_delivered** (latency, source: generation / approval / adoption / swap / safety_removal) -- N/A -- no Stage 2 metric in this feature's connected slice measures sync latency directly; retained to observe the near-instant propagation the product depends on for trust
- **plan_sync_conflict_resolved** (resolution: reject_with_refresh / safety_wins) -- N/A -- no Stage 2 metric measures conflict frequency; retained to observe how often concurrent plan changes occur across a household
- **plan_sync_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for sync trouble is observable

## Acceptance Criteria

**FEAT-03.SPEC-011-AC-01:** Given Maya accepts a swap suggestion on her device, when the change is confirmed, then Sam's device reflects the updated dinner card within the near-instant window without a manual refresh.

**FEAT-03.SPEC-011-AC-02:** Given a safety concern removes a dinner from the plan on one device, when the removal is confirmed, then every other household member's open plan view updates to show the removal live.

**FEAT-03.SPEC-011-AC-03:** Given a device was offline and reconnects, when reconciliation runs, then its plan view updates to the latest server-side state, discarding any now-stale local view.

**FEAT-03.SPEC-011-AC-04:** Given two devices attempt to swap the same dinner slot at effectively the same time, when the first swap is confirmed, then the second device's attempt is rejected with "This dinner just changed -- here's the latest." and its screen refreshes to the confirmed recipe.

**FEAT-03.SPEC-011-AC-05:** Given a safety removal and a swap acceptance target the same slot concurrently, when both are evaluated, then the safety removal wins and the swap acceptance is rejected.

**FEAT-03.SPEC-011-AC-06:** Given the sync capability is down, when Maya's device shows the current plan, then it falls back to its last-synced state with the Offline/Degraded messaging, and the plan remains viewable.

**FEAT-03.SPEC-011-AC-07:** Given the sync capability is slow but not down, when a change occurs elsewhere, then the update eventually appears without an error, even if delayed beyond the near-instant window.

**FEAT-03.SPEC-011-AC-08:** Given Maya's approval was not confirmed as propagated when connectivity was lost, when connectivity returns, then the approval is retried and other devices then show the Approved state.

**FEAT-03.SPEC-011-AC-09:** Given a device was offline through a full weekly regeneration, when it reconnects, then it reconciles directly to the new week's plan rather than replaying actions against the archived prior week.

**FEAT-03.SPEC-011-AC-10:** Given the same plan-update event is delivered to a device twice, when the second delivery arrives, then no duplicate visual change or toast occurs.

**FEAT-03.SPEC-011-AC-11:** Given a sync event arrives for a Weekly Plan that has since been archived, when it is processed, then it is discarded for the current-week display and does not alter the archived record's historical state.

**FEAT-03.SPEC-011-AC-12:** Given Riley (Operator) views a household's plan under an open Support Request, when plan changes occur elsewhere, then Riley's read-only view updates live exactly as any other viewer's would, without gaining any write access.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (one screen x three conditions) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |

---
document_type: feature-overview
feature_number: FEAT-03
feature_name: AI Weekly Dinner Plan Generation
feature_slug: ai-weekly-dinner-plan-generation
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 11
screen_count: 2
automation_count: 3
logic_rule_count: 4
integration_count: 2
notification_count: 0
---

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

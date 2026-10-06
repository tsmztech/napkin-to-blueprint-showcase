---
document_type: spec
spec_type: integration
spec_id: FEAT-08.SPEC-004
spec_name: Starter Recipe Content Seeding & Maintenance
spec_slug: starter-recipe-content-seeding-maintenance
parent_feature: FEAT-08
parent_feature_name: Recipe Library (Starter Recipes)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Starter Recipe Content Seeding & Maintenance

## Overview

**Name:** Starter Recipe Content Seeding & Maintenance
**ID:** FEAT-08.SPEC-004
**Type:** Integration
**Purpose:** Creates, corrects, and retires the shared starter-recipe content through the product's recipe/food-content data capability, so the library ships pre-populated at launch and stays complete enough to pass the allergy safety check.
**Parent Feature:** FEAT-08 -- Recipe Library (Starter Recipes)

## Scope and Non-Goals

**In Scope:**
- Creating starter Recipe records (name, ingredients with quantity/unit, steps, cook_time, rough_cost, origin = starter library, no owning_household, prep_requirements) before any household exists
- Delivering corrections to existing starter content over time (e.g., an ingredient fix, a cost refresh)
- Retiring (soft-archiving) starter content that is no longer offered
- User-facing behavior on FEAT-08.SPEC-001 and FEAT-08.SPEC-002 when this capability is slow, unavailable, or rejects a delivery
- Satisfying the coverage breadth the Recipe Library Coverage at Launch metric requires (no repeated dinners, at least one recipe per major cuisine style, for a household with typical dietary rules)

**Non-Goals:**
- Choosing the recipe/food-content data vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Household-imported recipe content -- creating, editing, or removing an imported Recipe record is FEAT-10's responsibility; this spec seeds and maintains starter content only.
- Bulk import tooling that lets households add their own recipes in quantity -- excluded per scope-boundaries.md SC-12: households bring their own recipes in one link at a time through FEAT-10, never in bulk, and this spec does not create such a path.
- Performing the allergy/religious-rule safety determination itself -- that determination is FEAT-02's authority (XBR-01); this spec's obligation is to supply ingredient data complete enough for that determination to run, not to run it.

## Capability Category

**Category:** Recipe/food-content data
**Dependency Source:** ASMP-34 -- "Recipe/food-content data capability -- Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Recipe/food-content data (ASMP-34)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-02; Integration Spec: FEAT-08.SPEC-004)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A brand-new household sees a pre-populated recipe library from the moment setup completes, with no recipes of its own yet | Browse recipes | FEAT-08.SPEC-001 (Recipe Library Browse & Search) |
| A brand-new household's first-week plan draws from a varied, complete starter corpus rather than an empty candidate pool | View recipe detail; feeds AI Weekly Dinner Plan Generation (FEAT-03) as its day-one candidate pool | FEAT-08.SPEC-001, FEAT-08.SPEC-002 (Recipe Detail View) |
| Starter recipe content stays accurate over time as ingredient data is corrected or content is retired | Browse recipes; View recipe detail | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| A starter recipe's ingredient data is complete enough that FEAT-02's safety check can pass it rather than fail it closed | (Cross-feature precondition for XBR-01's safety determination) | FEAT-02 (Dietary Rules & Allergy Safety Engine) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Coverage requirements | Not an entity -- functional targets: target cuisine styles, dietary-badge variety needed, minimum active corpus size | A new seeding batch is commissioned by the product's content operations (not triggered by any household action) | Tells the capability what content the library still needs so a new batch fills real gaps rather than duplicating existing coverage |

No household data, member data, or any personal data ever leaves the product through this capability -- starter recipe content carries no personal data (Recipe entity, Data Sensitivity: None, per feature-dependency-map.md), and this integration's outbound side is limited to content-coverage targets set by the product itself.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| New starter recipe content | The capability delivers a new recipe | Recipe -- name, ingredients (quantity + unit), steps, cook_time, rough_cost, origin (set to starter library), prep_requirements; owning_household left unset |
| Corrected starter recipe content | The capability delivers a correction to an existing starter recipe (e.g., an ingredient fix, a cost refresh) | Recipe -- the corrected field(s) only, identified by the existing recipe's name and origin |
| Retirement notice | The capability (or content operations) marks a starter recipe as no longer offered | Recipe -- soft-archived; no field content is deleted |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| New starter recipe delivered | The capability supplies a new recipe meeting the completeness standard (every ingredient has a quantity and unit) | A new Recipe record is created: origin = starter library, owning_household unset, full content populated | None directly -- the recipe simply appears in browse/search and detail on next load | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| New starter recipe delivered with incomplete ingredient data | The capability supplies a recipe missing a quantity or unit on one or more ingredients | The record is held out of the Active/browsable state -- it is neither created as visible nor offered to the candidate pool | None -- this is a content-operations condition, not a household-facing one; no household is affected since the recipe never becomes visible | None -- fails closed before reaching any household-facing spec |
| Existing starter recipe corrected | The capability delivers a correction to a recipe already in the library | The named field(s) update on the existing Recipe record | None directly -- the correction is reflected the next time the recipe is loaded in FEAT-08.SPEC-001 or opened in FEAT-08.SPEC-002; FEAT-02's badges re-evaluate live at that next load, with no separate re-check trigger needed | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Starter recipe retired | Content operations marks a starter recipe as no longer offered | The Recipe record is soft-archived: hidden from browse/search, dropped from the candidate pool for future plan generation; no cascade to Planned Meals that already used it (their history stands unchanged); no restore path | None directly -- the recipe simply stops appearing in browse/search and future plan candidates on next load | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-03 (AI Weekly Dinner Plan Generation, candidate pool) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | No visible impact -- this screen reads already-seeded Recipe records directly and is not itself waiting on the capability | No visible impact to households -- new or corrected content simply does not appear until the capability is reachable again; households continue browsing the existing library uninterrupted | A delivery batch that fails the completeness standard is held out of the Active/browsable state (per the Inbound Events row above); households see no error and continue browsing only previously accepted content |
| FEAT-08.SPEC-002 (Recipe Detail View) | No visible impact -- this screen reads already-seeded Recipe records directly | No visible impact -- a recipe already in the library remains fully viewable; a recipe that would have arrived from a delayed delivery simply is not yet available to open | N/A -- this screen never submits content to the capability; it only reads what has already been accepted |

## Consent and Disclosure

- **Coverage requirements are not user-facing** -- N/A: the only data leaving the product through this capability is content-coverage targets, set by the product's own content operations rather than on behalf of any household member; no household member is ever asked to consent to this exchange, since no personal or household data is included.
- **What is never shared** -- No household data, member data, dietary rule data, or any other personal data crosses this boundary in either direction; this capability exchanges only static recipe content and coverage targets (Recipe entity, Data Sensitivity: None).

## Edge Cases

- **The same corrected recipe is delivered twice** -- The second delivery is a no-op if the content is unchanged from the first; if the two deliveries differ, the most recently delivered correction wins.
- **A retirement notice arrives for a recipe already retired** -- No-op; the recipe stays archived.
- **A correction arrives for a recipe that has already been retired** -- The correction is discarded; a retired starter recipe accepts no further corrections, since re-adding equivalent content is a fresh seed action, not an undo (per the Recipe entity's Delete/Archive lifecycle in feature-overview.md).
- **A seeding delivery arrives with incomplete ingredient data** -- The recipe is held out of the Active/browsable state and excluded from both browse/search and any candidate pool until a corrected delivery completes the missing data, consistent with FEAT-02's fail-closed safety posture (XBR-01).
- **Seeding events arrive before any household exists** -- Content is created directly into the shared pool (owning_household unset); it is available to the first household that completes setup, with no household-specific step required.
- **A retirement event arrives for a recipe currently referenced by an active Planned Meal in some household's plan** -- The Planned Meal's existing reference and history are unaffected; the recipe is simply excluded from future candidate pools and no longer browsable, per the Recipe entity's Delete/Archive notes in feature-dependency-map.md.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Affects (outbound) | New, corrected, or retired starter content is reflected in browse/search results on next load |
| FEAT-08.SPEC-002 (Recipe Detail View) | Affects (outbound) | New, corrected, or retired starter content is reflected in the detail view on next load |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | The seeded and maintained starter corpus is part of the candidate pool plan generation draws from; a retirement removes a recipe from future candidate pools |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Affects (outbound) | Complete ingredient data delivered by this integration is the precondition FEAT-02's safety check requires to pass a starter recipe rather than fail it closed (XBR-01) |

## Analytics and Success Signals

- **starter_recipe_corpus_updated** (active recipe count, count added / corrected / retired in this delivery, cuisine styles represented in the active corpus) -- supports success-metrics.md: "Recipe Library Coverage at Launch"
- **starter_recipe_seed_incomplete** (recipe name, missing-data reason) -- N/A -- no Stage 2 metric tracks rejected-delivery volume; retained so content operations can see what a rejected delivery still needs corrected

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given no household has completed setup yet, when the recipe/food-content data capability delivers the initial starter batch, then Recipe records are created with origin set to starter library and owning_household unset, ready for the first household's library to show them.

**FEAT-08.SPEC-004-AC-02:** Given a household's library already shows a starter recipe, when the capability delivers a correction to that recipe's rough cost, then the corrected cost is reflected the next time Maya opens the recipe in FEAT-08.SPEC-002.

**FEAT-08.SPEC-004-AC-03:** Given a starter recipe is retired by content operations, when the retirement is processed, then the recipe is hidden from FEAT-08.SPEC-001's browse/search results and dropped from future plan-generation candidate pools, while any Planned Meal that already used it keeps its history unchanged.

**FEAT-08.SPEC-004-AC-04:** Given the capability delivers a new recipe missing a quantity on one ingredient, when the delivery is processed, then the recipe is held out of the Active/browsable state and never appears to any household until a corrected delivery completes the data.

**FEAT-08.SPEC-004-AC-05:** Given a household is browsing the library while the capability is unreachable, when they search or scroll, then browsing continues uninterrupted against the already-seeded content, with no visible error.

**FEAT-08.SPEC-004-AC-06:** Given a household opens a recipe detail while the capability is unreachable, when the recipe was already seeded, then it opens normally, since this screen reads already-accepted content directly rather than the capability itself.

**FEAT-08.SPEC-004-AC-07:** Given the same corrected recipe delivery is processed twice with identical content, when the second delivery is processed, then nothing changes and no duplicate update is recorded.

**FEAT-08.SPEC-004-AC-08:** Given a retirement notice arrives for a recipe already retired, when it is processed, then the recipe remains archived with no change.

**FEAT-08.SPEC-004-AC-09:** Given a correction arrives for a recipe that has already been retired, when it is processed, then the correction is discarded and the recipe stays archived.

**FEAT-08.SPEC-004-AC-10:** Given a new household completes setup for the first time, when they open the recipe library, then they see a starter corpus with no repeated dinners and at least one recipe per major cuisine style, per the corpus's coverage requirement.

**FEAT-08.SPEC-004-AC-11:** Given content operations commissions a new seeding batch, when the coverage requirements are sent to the capability, then no household or personal data is included in that request.

**FEAT-08.SPEC-004-AC-12:** Given a starter recipe passes FEAT-02's safety check because its ingredient data is complete, when a household views it, then it shows the standard dietary badge rather than an ineligibility explanation, confirming this integration's data supplied what FEAT-02's determination required.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 5 (2 screens; 1 N/A cell excluded) | 5 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |

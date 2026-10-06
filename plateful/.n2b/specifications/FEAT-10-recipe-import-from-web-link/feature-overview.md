---
document_type: feature-overview
feature_number: FEAT-10
feature_name: Recipe Import from Web Link
feature_slug: recipe-import-from-web-link
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 4
automation_count: 2
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Recipe Import from Web Link

## Summary

**Feature:** Recipe Import from Web Link
**ID:** FEAT-10
**Description:** A household member can save a recipe from any website by pasting its link, adding it to their own recipe pool alongside the starter library.
**Priority:** Important
**Phase:** v1
**Type:** User-Facing
**Rationale:** The brief names this directly: "saving recipes from any website by pasting a link" (BRIEF.md, Ecosystem & Integrations), while also flagging that "whether importing recipes from other sites is legal is an open question" (BRIEF.md, Open Questions). Phased to v1 rather than MVP because the starter library (FEAT-08) already gives new households a working candidate pool on day one; import extends personalization once the core loop is proven and the legal question is resolved. Research flags that import reliability is a known weak point among competitors and that a lifetime import cap draws user criticism, so review-before-save, manual fallback, and an uncapped (but rate-limited) import allowance are load-bearing product decisions, not incidental details.

**Key Capabilities:**
- Import by link — Household member pastes a web link and the recipe's ingredients, steps, and cook time are extracted into the household's recipe pool
- Review before saving — Household member confirms or edits extracted details before the recipe is saved
- See imported recipes alongside starter ones — Imported recipes appear in the same library and are eligible for the weekly plan
- Edit or remove an imported recipe — Household member corrects an imported recipe's details later, or removes it from the household's pool; the change is re-checked for safety before it can appear in a plan again

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-10.SPEC-001 | Import by Link | Screen | Maya, Sam | Household member pastes a web link to start an import, with an explained progress indicator while extraction runs |
| FEAT-10.SPEC-002 | Review Extracted Recipe | Screen | Maya, Sam | Household member confirms or edits the extracted ingredients, steps, and cook time, then saves the recipe |
| FEAT-10.SPEC-003 | Manual Recipe Entry | Screen | Maya, Sam | Household member types in ingredients, steps, and cook time by hand when extraction fails, so the import never dead-ends |
| FEAT-10.SPEC-004 | Edit Imported Recipe | Screen | Maya, Sam, Riley | Household member corrects an already-saved imported recipe's details or removes it from the household's pool |
| FEAT-10.SPEC-005 | Web Page Recipe Extraction | Integration | Maya, Sam | Reads a pasted web page through the product's web-page recipe extraction capability (ASMP-36) and returns ingredients, steps, and cook time, or reports that extraction failed |
| FEAT-10.SPEC-006 | Duplicate Import Detection | Automation | Maya, Sam | Checks whether the pasted link is already saved for the household and, if so, surfaces the existing recipe instead of creating a duplicate |
| FEAT-10.SPEC-007 | Recipe Import Validation & Rate Limit Rules | Logic/Rule | Maya, Sam | Governs link well-formedness, ingredient/step length caps, the minimum-one-ingredient save requirement, and the 30-import-per-week misuse guard |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | Automation | All | Runs the household's allergy/religious-rule safety check on every initial save and every later edit of an imported recipe before it can appear in a plan (XBR-01, XBR-19) |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Import by link | FEAT-10.SPEC-001, FEAT-10.SPEC-005 | The paste-link screen collects the link and hands it to the web-page recipe extraction capability, which returns ingredients, steps, and cook time | Phase 2 (Explicit) |
| Review before saving | FEAT-10.SPEC-002 | Primary purpose of the review/confirm screen — extracted details are shown for confirmation or edit before saving | Phase 2 (Explicit) |
| See imported recipes alongside starter ones | FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Saving (from either the review screen or manual entry) writes the recipe into the shared Recipe pool that FEAT-08.SPEC-001 lists and searches (cross-feature display) | Phase 2 (Explicit) |
| Edit or remove an imported recipe | FEAT-10.SPEC-004, FEAT-10.SPEC-008 | The edit screen lets a member correct details or remove the recipe; every accepted edit re-triggers the safety re-check before the recipe can appear in a plan again | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-10.SPEC-003 | Manual Recipe Entry | Phase 6 (Negative/Failure Analysis) | The States field's Error state and the Primary Flows & Alternates field both require that "a link that cannot be parsed prompts the member to enter the recipe details manually instead of failing silently" — a required fallback screen, not an incidental note |
| FEAT-10.SPEC-005 | Web Page Recipe Extraction | Phase 4 (External Dependencies lens) | ASMP-36 (Dependencies) names the web-page recipe extraction capability this feature relies on; the dependency map's External Touchpoints row for this capability was "pending — awaiting validated Brief for FEAT-10," which this decomposition resolves |
| FEAT-10.SPEC-006 | Duplicate Import Detection | Phase 4 (Trigger-Response) | Primary Flows & Alternates names duplicate-import behavior explicitly; a cross-entity uniqueness check triggered on save is processing logic with cross-record effects, crossing the standalone-Automation threshold |
| FEAT-10.SPEC-007 | Recipe Import Validation & Rate Limit Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names five distinct conditions (well-formed link, length caps, minimum-one-ingredient requirement, unrecognizable-ingredient block, 30-imports-per-week cap) shared across four screens — well past the 5+-rules / shared-across-screens threshold for a standalone Logic/Rule spec |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | Phase 4 (Trigger-Response) | XBR-01 and XBR-19 require every save and edit of an imported recipe to pass FEAT-02's safety check again before it can appear in a plan; this is a cross-feature-triggering side-effect distinct from the edit screen's own data write |

## Entity-Lifecycle Coverage Matrix

**Entity: Recipe** (imported slice — this feature's slice of the shared Recipe entity; starter content is FEAT-08's slice)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Review Extracted Recipe and Manual Recipe Entry both save a new Recipe record (origin = imported, with source link; owning_household = this household) once FEAT-10.SPEC-007's validation passes and FEAT-10.SPEC-006 finds no existing duplicate; FEAT-10.SPEC-008 then runs the safety check before the record can appear in any plan | Starter Recipe records are created by FEAT-08.SPEC-004, not this feature |
| Read (single) | FEAT-10.SPEC-004 | Edit Imported Recipe loads one household-owned imported recipe's full content for correction or removal | -- |
| Read (list) | N/A | This feature has no standing list of its own (feature's own States field: "this feature has no standing list of its own — imported recipes appear inside Recipe Library"); listing and searching the household's combined pool is FEAT-08.SPEC-001's responsibility | -- |
| Update | FEAT-10.SPEC-004 | Household member edits ingredients, steps, cook time, or other fields on an already-saved imported recipe; FEAT-10.SPEC-007 re-validates and FEAT-10.SPEC-008 re-runs the safety check before the edited recipe can appear in a plan again (XBR-19) | Starter recipes have no household-facing edit surface; editing them is FEAT-08's content-maintenance responsibility, not this feature's |
| Delete/Archive | FEAT-10.SPEC-004 | Hard delete: removing an imported recipe deletes it from the household's pool outright — distinct from starter content's soft archive. No cascade to Planned Meals that already used it (past plan history stands unchanged). No restore path: re-adding the same recipe requires re-importing the link, which is treated as a fresh import (FEAT-10.SPEC-006 no longer finds a duplicate). No retention window: removal is immediate and permanent, since the imported recipe is the household's own data under their direct delete control, not shared maintained content | Distinct from FEAT-08's starter-content soft-archive-with-no-purge pattern |
| State Transition | N/A | The Recipe entity's functional fields (dependency map, Shared Data Entities) give imported content no lifecycle state beyond existing-or-deleted; dietary_badges are computed live by FEAT-02 at view time rather than stored as a state, so a save or edit is reflected as soon as FEAT-10.SPEC-008's re-check completes, with no separate state field to transition | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-10.SPEC-006 | Duplicate Import Detection scopes its uniqueness check to the current household's own imported recipes only — a link another household imported is not a duplicate for this one |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household member pastes a link and submits it | Validate the link is well-formed; emit recipe_import_started | Standalone Logic/Rule | FEAT-10.SPEC-007 |
| Link passes validation | Request extraction from the web-page recipe extraction capability, showing a brief, explained progress indicator | Standalone Integration | FEAT-10.SPEC-005 |
| Extraction succeeds | Populate the Review Extracted Recipe screen with the draft ingredients, steps, and cook time; emit recipe_import_succeeded | Inline in triggering screen | FEAT-10.SPEC-002 |
| Extraction fails or the page layout can't be parsed | Offer manual entry instead of a dead end; emit recipe_import_failed, then recipe_import_manual_fallback if the member proceeds | Standalone Integration outcome feeding a screen | FEAT-10.SPEC-005 → FEAT-10.SPEC-003 |
| Household member submits a link while offline | Hold the pasted link as a draft; process it automatically once connectivity returns (Offline/Degraded state) | Inline in triggering screen | FEAT-10.SPEC-001 |
| Household member confirms/edits and saves (review screen or manual entry) | Check for a link already saved for this household | Standalone Automation | FEAT-10.SPEC-006 |
| Duplicate link found | Surface the existing recipe instead of creating a second record; no new Recipe is saved | Standalone Automation | FEAT-10.SPEC-006 |
| No duplicate found and validation passes | Create the Recipe record, then run the household's safety check before the recipe can appear in any plan | Standalone Automation | FEAT-10.SPEC-008 |
| Saved or edited recipe fails the safety check (e.g., an unrecognizable ingredient) | Recipe still saves to the household's pool, but is excluded from every candidate path until the ingredient is clarified (XBR-01 fail-closed) | Standalone Logic/Rule (validation) / cross-feature (FEAT-02 owns the determination) | FEAT-10.SPEC-007 / FEAT-02 responsibility |
| Household member edits an already-saved imported recipe | Re-run the safety check before the edited recipe can appear in a plan again; emit imported_recipe_edited | Standalone Automation | FEAT-10.SPEC-008 |
| Household member removes an imported recipe | Delete the Recipe record from the household's pool; it drops out of the candidate pool immediately; past Planned Meals that used it are unaffected; emit imported_recipe_removed | Inline in triggering screen (destructive action, confirmation required) | FEAT-10.SPEC-004 |
| Household member attempts an import beyond 30 in the current rolling week | Block the import with a plain explanation of the weekly limit | Standalone Logic/Rule | FEAT-10.SPEC-007 |
| A recipe finishes saving and passes the safety check | Recipe becomes visible in the household's library alongside starter recipes and eligible for the weekly plan | Cross-feature — logged in touchpoints | FEAT-08 (display) / FEAT-03 (plan eligibility) responsibility |

## Shared Context

**Shared Entities:**
- Recipe (imported slice) — created by SPEC-002 and SPEC-003; read, updated, and deleted by SPEC-004; checked for duplicates by SPEC-006; validated and rate-limited by SPEC-007; safety-rechecked by SPEC-008. Fields this feature owns for imported content: name, ingredients (quantity + unit), steps, cook_time, origin (= imported, with source link), owning_household. dietary_badges is never written by this feature — it is computed live by FEAT-02 and only reflected once FEAT-10.SPEC-008's re-check completes.
- Household (read-only) — owning_household boundary read by SPEC-006 to scope duplicate detection to this household's own imports.

**Shared UI Patterns:**
- Recipe details form — SPEC-002 (pre-filled from extraction), SPEC-003 (empty, manual), and SPEC-004 (pre-filled from the existing saved recipe) all present the same ingredients/steps/cook-time form; Spec Writers for all three should describe the fields identically and vary only the pre-fill source and screen entry point.
- Explained progress indicator — SPEC-001's "typically a few seconds" extraction wait (feature's own States field) is the timing contract SPEC-005 must honor; any future long-running step in this feature should reuse the same pattern rather than a bare spinner.
- Destructive-action confirmation — SPEC-004's remove action follows the same confirm-before-destroy pattern other features use for irreversible deletes, given this entity's delete is hard and has no restore path.

**Shared Validation:**
- SPEC-007 (Recipe Import Validation & Rate Limit Rules) is the single source of truth for link well-formedness, ingredient/step length caps, the minimum-one-ingredient save requirement, and the 30-imports-per-week guard. SPEC-001, SPEC-002, SPEC-003, and SPEC-004 all reference it rather than each defining validation independently.
- The safety pass/fail determination itself remains FEAT-02's authority (XBR-01); SPEC-008 governs only when this feature triggers that check, not how it decides.

## Internal Dependency Map

```
SPEC-001 (Import by Link) -> [member submits a link] -> SPEC-007 (Recipe Import Validation & Rate Limit Rules)
SPEC-001 (Import by Link) -> [validation passes] -> SPEC-005 (Web Page Recipe Extraction)
SPEC-005 (Web Page Recipe Extraction) -> [extraction succeeds] -> SPEC-002 (Review Extracted Recipe)
SPEC-005 (Web Page Recipe Extraction) -> [extraction fails] -> SPEC-003 (Manual Recipe Entry)
SPEC-002 (Review Extracted Recipe) -> [member confirms/saves] -> SPEC-006 (Duplicate Import Detection)
SPEC-003 (Manual Recipe Entry) -> [member saves] -> SPEC-006 (Duplicate Import Detection)
SPEC-006 (Duplicate Import Detection) -> [duplicate found] -> SPEC-002 / SPEC-003 (surfaces existing recipe instead of saving)
SPEC-006 (Duplicate Import Detection) -> [no duplicate] -> SPEC-008 (Safety Re-check on Import Save/Edit)
SPEC-004 (Edit Imported Recipe) -> [member saves a change, or removes the recipe] -> SPEC-008 (Safety Re-check on Import Save/Edit)
SPEC-002 (Review Extracted Recipe) / SPEC-003 (Manual Recipe Entry) -> [validates fields using] -> SPEC-007 (Recipe Import Validation & Rate Limit Rules)
SPEC-004 (Edit Imported Recipe) -> [validates fields using] -> SPEC-007 (Recipe Import Validation & Rate Limit Rules)
```

**Default Entry:** SPEC-001 (Import by Link) — the screen shown when a household member starts an import from the library's "import from link" entry point; the feature is also entered directly at SPEC-004 when a household member opens an existing imported recipe to edit or remove it.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-10.SPEC-001 | Inbound | FEAT-08 (Recipe Library, Starter Recipes) | The library's "import from link" entry point starts this feature's flow (Recipe Import & Pantry Update, step 1) | Household member taps "import from link" from the library |
| FEAT-10.SPEC-002 / FEAT-10.SPEC-003 | Outbound | FEAT-08 (Recipe Library, Starter Recipes) | A saved recipe appears in the household's combined library alongside starter recipes (Recipe Import & Pantry Update, step 3) | Member saves a reviewed or manually entered recipe |
| FEAT-10.SPEC-006 / FEAT-10.SPEC-008 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Every save or edit of an imported recipe passes FEAT-02's app-enforced allergy/religious-rule check, which owns the pass/fail determination (XBR-01, XBR-19) | Recipe first saved, or an already-saved imported recipe edited |
| FEAT-10.SPEC-002 / FEAT-10.SPEC-003 / FEAT-10.SPEC-004 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | A saved and safety-checked imported recipe joins the candidate pool plan generation draws from, and the following week's plan can use it (Recipe Import & Pantry Update, step 5) | Recipe passes the safety check after save or edit |
| FEAT-10.SPEC-002 / FEAT-10.SPEC-003 / FEAT-10.SPEC-004 | Outbound | FEAT-23 (Manual Weekly Planning) | An eligible imported recipe is selectable in manual picks the same way a starter recipe is | Household member picks a night manually |
| FEAT-10.SPEC-005 | Outbound | (none within this Brief) | The web-page recipe extraction capability (ASMP-36) is this feature's own Integration spec; no other feature owns it | Member submits a link |

## Non-Functional Notes

**Data volumes / growth:** A household may import up to 30 recipes a week, with no lifetime cap on either tier (Validation & Limits field). Across the several-thousand-household, 2–6-member-per-household growth described in ASMP-24, imported-recipe volume grows per household rather than in a shared corpus, but import processing (extraction requests, duplicate checks, safety re-checks) must stay equally responsive as the household base grows.

**Responsiveness:** Extraction shows a brief, explained progress indicator, typically a few seconds (feature's own States field). Once saved, an imported recipe must appear in library search results within about a second like any other recipe (ASMP-23). Every primary action (paste, confirm, edit, remove) is reachable with one thumb and uses large tap targets (ASMP-29).

**Data sensitivity / privacy:** The Recipe entity itself carries no personal data (dependency map, Recipe entity, Data Sensitivity: None). An imported recipe's source link and household ownership are household data under the general no-sale posture (ASMP-14) and are exportable/deletable under general personal-data rights, private to the importing household. The dietary badge shown on an imported recipe is FEAT-02's derived output, not raw household allergy data — this feature never reads or stores Dietary Rule records directly.

**Compliance flags:** N/A — no compliance regime is named for recipe import itself; the founder's own open question about the legality of importing content from other websites (BRIEF.md, Open Questions; ASMP-36) is a product/legal decision to resolve before v1 ships, not a technical compliance obligation this functional Brief can specify further.

## Non-Goals

- **Bulk import of plans, lists, or recipe collections from other apps** — Excluded per scope-boundaries.md (SC-12): the brief names only per-link recipe import; households bring recipes in one link at a time, and bulk import would add a data path the solo founder must maintain with no supporting brief goal.
- **Cross-household recipe sharing of an imported recipe** — Excluded per scope-boundaries.md (SC-09): an imported recipe belongs to the importing household only; there is no public or cross-household visibility for any recipe, imported or starter, since the brief confirms "otherwise the product stands alone."
- **A free-tier or lifetime cap on the number of imports** — Explicit product decision named in product-features.md's Rationale: research flagged a lifetime import cap as a hard limit competitors' users criticized, so the founder deliberately rejected one; only the 30-per-week misuse guard (FEAT-10.SPEC-007) applies, never a lifetime or tier-based ceiling.
- **Automatic purge or retention window for a removed imported recipe** — Intentional lifecycle decision surfaced by the CRUD matrix: removal (FEAT-10.SPEC-004) is immediate and permanent with no retention window, distinct from starter content's soft-archive-with-no-purge pattern (FEAT-08), because an imported recipe is the household's own data under their direct delete control rather than shared maintained content.
- **Resolving the legality of importing content from third-party websites** — Named as an open question in BRIEF.md (Open Questions) and recorded as a dependency to settle before v1 (ASMP-36); this Brief specifies the functional import behavior approved for build, not the legal resolution itself, which sits outside the Feature Analyst's decision authority.

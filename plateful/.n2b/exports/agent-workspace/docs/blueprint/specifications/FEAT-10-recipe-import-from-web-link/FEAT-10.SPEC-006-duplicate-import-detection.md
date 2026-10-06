---
document_type: spec
spec_type: automation
spec_id: FEAT-10.SPEC-006
spec_name: Duplicate Import Detection
spec_slug: duplicate-import-detection
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Duplicate Import Detection

## Overview

**Name:** Duplicate Import Detection
**ID:** FEAT-10.SPEC-006
**Type:** Automation
**Purpose:** Checks whether the recipe being saved is already saved for the household and, if so, surfaces the existing recipe instead of creating a duplicate.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Checking for an existing household-owned imported Recipe matching the link (when a source link is available) or the entered recipe details (when saving from manual entry)
- Scoping the check to the current household's own imported recipes only
- Surfacing the existing recipe to the member instead of creating a second record when a match is found

**Non-Goals:**
- Deduplicating starter library content -- starter recipes are shared, read-only content with no household ownership; this automation only checks a household's own imported recipes against each other (XBR-19).
- Detecting near-duplicate recipes with different links (e.g., the same recipe re-published on two sites) -- excluded per product-features.md's Primary Flows & Alternates, which defines duplicate import narrowly as "importing a link already saved for the household"; content-similarity matching across different sources is not part of the defined product.
- Merging or reconciling differences between a newly submitted import and the existing matched recipe -- the existing recipe is simply surfaced as-is; any correction to it is a separate action through FEAT-10.SPEC-004 (Edit Imported Recipe).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member taps Save with a link-based import | FEAT-10.SPEC-002 (Review Extracted Recipe) | Fires after FEAT-10.SPEC-007 validation passes | Source link, confirmed/edited ingredients, steps, cook time, owning household |
| Member taps Save with a manually entered import | FEAT-10.SPEC-003 (Manual Recipe Entry) | Fires after FEAT-10.SPEC-007 validation passes | No source link; confirmed/edited ingredients, steps, cook time, owning household |

## Processing Logic

1. Receive the recipe data submitted for save (from FEAT-10.SPEC-002 or FEAT-10.SPEC-003), including the source link when one exists.
2. Read all existing Recipe records with origin = imported and owning_household equal to the saving household (per feature-dependency-map.md's Contention note: the check is scoped to this household's own imports only -- a link another household imported is not a duplicate for this one).
3. If a source link is present (the import came from FEAT-10.SPEC-002): compare it against the source link of each existing household-owned imported recipe for an exact match.
4. If no source link is present (the import came from FEAT-10.SPEC-003, manual entry): compare the entered recipe name against the name of each existing household-owned imported recipe for an exact match (case-insensitive).
5. If a match is found, do not create a new Recipe record; return the matched recipe's identity to the triggering screen.
6. If no match is found, signal the triggering screen to proceed with creating the new Recipe record.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No duplicate found | Zero matches against the household's existing imported recipes | None -- the triggering screen proceeds to create the Recipe record | Save proceeds normally; "Recipe saved" toast on completion | FEAT-10.SPEC-002, FEAT-10.SPEC-003 |
| Duplicate found (link match) | The submitted source link matches an existing household-owned imported recipe's source link | None -- no new Recipe record is created | "You've already imported this recipe." with a "View existing recipe" action, navigating to the matched recipe in FEAT-08.SPEC-002 | FEAT-10.SPEC-002 |
| Duplicate found (name match, manual entry) | The submitted recipe name matches an existing household-owned imported recipe's name (case-insensitive), and no source link is available for comparison | None -- no new Recipe record is created | "You've already imported this recipe." with a "View existing recipe" action, navigating to the matched recipe in FEAT-08.SPEC-002 | FEAT-10.SPEC-003 |
| Automation failure | The duplicate check itself cannot complete (e.g., a processing error reading existing recipes) | None | Save proceeds with a non-blocking warning: "Couldn't check for a duplicate -- the recipe was saved anyway." The Recipe record is created despite the check's failure, since blocking every save on this non-critical check would contradict the feature's explicit design goal that import never dead-ends. | FEAT-10.SPEC-002, FEAT-10.SPEC-003 |

## Data Model

**Reads:** Recipe records -- source link, name, owning_household, origin -- for every existing imported recipe belonging to the saving household.
**Creates:** None -- this automation only determines whether the triggering screen's own create action should proceed; it does not itself create the Recipe record.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The duplicate check is scoped to the saving household's own imported recipes only (Household entity, read-only, per feature-dependency-map.md's Referenced Entities) -- a link or name matching another household's import is never treated as a duplicate for this household.
- Link-based imports are matched by exact source link; manual entries (which carry no source link) are matched by exact recipe name, case-insensitive.
- The check is non-blocking on its own failure: if the automation itself cannot complete, the save proceeds rather than dead-ending the import, consistent with the feature's fallback-first design (product-features.md's States field).
- A duplicate finding never edits or merges the existing recipe -- it is surfaced unchanged (XBR-19).

## Edge Cases

- **Household has no existing imported recipes** -- The check completes immediately with no duplicate found; the save proceeds normally.
- **Two different links resolve to the same underlying recipe on the same site (e.g., a tracking parameter differs)** -- Treated as no match, since this automation compares links exactly; this is an accepted limitation given product-features.md's narrow definition of duplicate import as "importing a link already saved," not content-similarity matching.
- **Manually entered recipe name differs only in capitalization or trailing whitespace from an existing recipe** -- Matched as a duplicate; the comparison is case-insensitive and ignores leading/trailing whitespace.
- **Concurrent trigger firing (Maya and Sam each submit the same link for import from separate devices at effectively the same time)** -- Each save runs its own duplicate check independently against the recipes that exist at the moment its check runs. The check that starts second, if it runs after the first save has completed, finds the first save's newly created recipe and surfaces it; if both checks run before either save completes, both may find no duplicate and both proceed to create -- in that case, the second Recipe record's creation is the create action's own responsibility to resolve (FEAT-10.SPEC-002/FEAT-10.SPEC-004's reject-with-refresh applies to edits, not to this narrow race on first creation, which the dependency map's Contention note treats as an acceptable low-probability outcome given imported recipes have no uniqueness constraint enforced below the level of this check).
- **Trigger fires while a previous run is in flight for the same household** -- The triggering screen's Save button is disabled during save (FEAT-10.SPEC-002, FEAT-10.SPEC-003), so a second run for the same save action cannot start while the first is in flight. Runs triggered by different household members' separate save actions proceed independently, per the concurrent trigger firing entry above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Triggered by (inbound) | Save action, after validation, triggers this check |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Triggered by (inbound) | Save action, after validation, triggers this check |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Affects (outbound) | Returns duplicate outcome to the review screen |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Affects (outbound) | Returns duplicate outcome to the manual entry screen |
| FEAT-08.SPEC-002 (Recipe Detail View) | Affects (outbound) | "View existing recipe" navigates to the matched recipe here |

## Analytics and Success Signals

- **duplicate_import_check_completed** (result: no_duplicate / duplicate_found_by_link / duplicate_found_by_name; entry path: link / manual) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into how often households re-import the same recipe.
- **duplicate_import_check_failed** (reason: processing_error) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; this event measures how often the non-blocking guarantee (save proceeds despite a check failure) is exercised.

## Acceptance Criteria

**FEAT-10.SPEC-006-AC-01:** Given Maya's household has no recipe imported from a given link, when she saves a freshly extracted recipe from that link, then the duplicate check finds no match and the Recipe record is created.

**FEAT-10.SPEC-006-AC-02:** Given Maya's household already imported a recipe from a given link, when Sam saves a new import from the same link, then no new Recipe record is created and Sam sees "You've already imported this recipe." with "View existing recipe".

**FEAT-10.SPEC-006-AC-03:** Given Maya's household already imported a recipe named "Weeknight Chili" (manually entered, no source link), when Sam manually enters a recipe also named "weeknight chili" (different capitalization), then the duplicate check matches it and he sees the existing-recipe message.

**FEAT-10.SPEC-006-AC-04:** Given another household (not Maya's) has imported a recipe from the exact same link, when Maya imports that link for her own household, then the duplicate check finds no match, since the check is scoped to Maya's household only.

**FEAT-10.SPEC-006-AC-05:** Given Sam taps "View existing recipe" after a duplicate is found, when the navigation completes, then he is shown the existing recipe's detail on FEAT-08.SPEC-002, and no second Recipe record exists.

**FEAT-10.SPEC-006-AC-06:** Given the duplicate check itself fails to complete due to a processing error, when Maya's save proceeds, then the Recipe record is created anyway with the non-blocking warning "Couldn't check for a duplicate -- the recipe was saved anyway."

**FEAT-10.SPEC-006-AC-07:** Given Sam's household has multiple existing imported recipes, when he saves a recipe from a link that matches none of them, then the check completes with no duplicate found and the save proceeds.

**FEAT-10.SPEC-006-AC-08:** Given Maya and Sam each submit the same link from separate devices at effectively the same time, when Sam's save completes first, then Maya's check (running after) finds Sam's newly created recipe and surfaces it to her instead of creating a duplicate.

**FEAT-10.SPEC-006-AC-09:** Given a member removes an imported recipe and later re-imports the exact same link, when the re-import is saved, then the duplicate check finds no match, since the removed recipe no longer exists in the household's pool (per FEAT-10.SPEC-004's hard-delete, no-restore lifecycle), and a fresh Recipe record is created.

**FEAT-10.SPEC-006-AC-10:** Given Maya submits a link that resolves to the same underlying recipe as an existing import but with a differing tracking parameter, when the exact-link comparison runs, then no duplicate is detected and a second Recipe record is created, per this automation's accepted exact-match limitation.

**FEAT-10.SPEC-006-AC-11:** Given Sam's Save button is disabled while his own save is in flight, when he attempts to trigger a second save for the same action, then no second duplicate-check run starts for that action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (link-based save, manual save) | 2 |
| Outcome Paths | 4 (no duplicate, duplicate by link, duplicate by name, automation failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

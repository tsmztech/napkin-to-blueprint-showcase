# FEAT-10 — Recipe Import from Web Link

This chapter covers FEAT-10, Recipe Import from Web Link, a Important-tier feature. It contains 8 specifications carrying 105 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-10.SPEC-001 | Import by Link | screen | 12 |
| FEAT-10.SPEC-002 | Review Extracted Recipe | screen | 13 |
| FEAT-10.SPEC-003 | Manual Recipe Entry | screen | 11 |
| FEAT-10.SPEC-004 | Edit Imported Recipe | screen | 13 |
| FEAT-10.SPEC-005 | Web Page Recipe Extraction | integration | 12 |
| FEAT-10.SPEC-006 | Duplicate Import Detection | automation | 11 |
| FEAT-10.SPEC-007 | Recipe Import Validation & Rate Limit Rules | logic-rule | 22 |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | automation | 11 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Import by Link

## Overview

**Name:** Import by Link
**ID:** FEAT-10.SPEC-001
**Type:** Screen
**Purpose:** Household member pastes a web link to start a recipe import, with an explained progress indicator while extraction runs.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Capturing a pasted web link and submitting it to start an import
- Well-formedness and weekly-limit checks before extraction is requested (via FEAT-10.SPEC-007)
- Showing an explained progress indicator while extraction runs (FEAT-10.SPEC-005)
- Holding a submitted link as a draft when offline and processing it automatically once connectivity returns

**Non-Goals:**
- Reviewing or editing the extracted recipe details -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this screen only collects the link and shows extraction progress.
- Manual recipe entry -- handled by FEAT-10.SPEC-003 (Manual Recipe Entry), reached only when extraction fails and the member chooses to proceed manually.
- Bulk import of multiple links in one submission -- excluded per scope-boundaries.md SC-12: the brief names only per-link recipe import; households bring recipes in one link at a time.
- Determining whether the pasted page's content may legally be imported -- BRIEF.md's Open Questions leaves the legality of importing third-party recipe content as a product/legal decision to resolve before v1 ships; this screen assumes that resolution and only implements the functional import behavior approved for build.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Household member taps "Import from link" | None -- link input starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Paste a link and submit it | -- |
| Sam (Other Adult Member) | Full screen | Paste a link and submit it | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen; Maya manages all content on this profile's behalf |
| Jordan (older kid, limited login -- Later) | No | No | "Import from link" is not offered on FEAT-08.SPEC-001 for this role (Recipe Library access is View, not Full); if reached directly, the screen shows "Importing recipes isn't available on this profile." with a link back to the recipe library |
| Riley (Operator, support) | No | No | This screen is not reachable from the read-only support view; Riley's Recipe Library access is View only and carries no import entry point |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-08.SPEC-001 (Recipe Library), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- a partially typed link is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Import from Link" with a back arrow (returns to FEAT-08.SPEC-001, Recipe Library Browse & Search).

**Body:** A single-column form with:
- Link input (text input, required) -- placeholder "Paste a recipe link", accepts pasted or typed text
- "Import" action button, below the link input, disabled until the input is non-empty

Below the Import button, a helper line states: "We'll pull the ingredients, steps, and cook time from the page -- you'll be able to review and edit before it's saved."

While extraction runs (Extracting state), the link input and Import button are replaced by a progress indicator with the label "Reading the recipe... this usually takes a few seconds."

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Import button spans the input's width.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-08.SPEC-001 (Recipe Library) | Screen closes | Animated transition back to library |
| Link input | Type or paste | Captures link text | Import button enables once non-empty | Standard input focus state |
| Import button | Tap | 1. Validate the link and check the weekly import allowance via FEAT-10.SPEC-007. 2. If valid, request extraction via FEAT-10.SPEC-005 (Web Page Recipe Extraction). | Screen enters Validating then Extracting state | Progress indicator "Reading the recipe... this usually takes a few seconds." |
| Import button (while extracting) | Tap | No action -- debounced | None | Button/indicator area unchanged; a second submission cannot start while one is in flight |

### Accessibility Notes

- **Focus order:** Back arrow -> Link input -> Import button.
- **Validation announcements:** When the link fails well-formedness or the weekly limit is reached, the resulting message is announced to assistive technology and programmatically associated with the input.
- **Progress announcements:** Entry into the Extracting state announces "Reading the recipe" to assistive technology; completion (success or failure) announces the resulting state.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Link input empty, Import button disabled | Screen first opens | Member types or pastes into the link input |
| Filling | Link input contains text, Import button enabled | Member types or pastes a non-empty value | Member taps Import or navigates away |
| Validating | Import button shows a brief loading state | Member taps Import | FEAT-10.SPEC-007's well-formedness and weekly-limit checks complete |
| Validation Error | Link input shows an error state with the message from FEAT-10.SPEC-007 | A validation rule fails | Member edits the link and re-submits |
| Extracting | Link input and Import button replaced by the progress indicator "Reading the recipe... this usually takes a few seconds." | Validation passes and extraction is requested (FEAT-10.SPEC-005) | Extraction succeeds or fails |
| Error (extraction failed) | Message "We couldn't read that page." with a "Enter details manually" action and a "Try a different link" action | FEAT-10.SPEC-005 reports extraction failure | Member chooses manual entry (navigates to FEAT-10.SPEC-003) or re-submits a different link |
| Offline/Degraded | Banner "You're offline -- this link will be imported when you reconnect." at top; link input remains editable and submittable; submitting queues the link as a draft locally | Connectivity lost while the screen is open, or the member submits while already offline | Connectivity restored -- the queued link is validated and extraction is requested automatically, and the screen proceeds through Validating/Extracting as normal |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for link well-formedness and the weekly import allowance. This screen checks on Import button tap.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-08.SPEC-001 (Recipe Library Browse & Search) | FEAT-08 |
| Extraction succeeds | FEAT-10.SPEC-002 (Review Extracted Recipe) | -- |
| Extraction fails, member chooses "Enter details manually" | FEAT-10.SPEC-003 (Manual Recipe Entry) | -- |

## Data Model

**Creates:** None -- this screen does not create a Recipe record; it only initiates extraction.
**Reads:** Household's current-week import count (used by FEAT-10.SPEC-007's weekly-limit check) -- read-only, no fields displayed on this screen.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Link well-formedness and the 30-import-per-week allowance are enforced by FEAT-10.SPEC-007 -- the member cannot submit a link that fails either check.
- A link submitted while offline is held as a draft and processed automatically once connectivity returns, per the feature's own States field.
- Only one extraction request runs at a time from this screen -- a second Import tap while one is in flight is ignored (debounced).

## Edge Cases

- **Member submits an empty link input** -- Import button remains disabled; no submission is possible.
- **Member taps Import twice rapidly** -- Second tap is ignored while the first request is in flight (button/indicator area unchanged).
- **Member navigates away while extraction is running** -- The extraction request continues; if it completes after the member has left, no notification interrupts them -- the result (populated review draft, or the failure state) is present the next time they return to this flow or reopen the same link.
- **Member pastes a link that was already imported by this household** -- The screen does not detect duplicates itself; duplicate handling occurs at save time on FEAT-10.SPEC-002/FEAT-10.SPEC-003 via FEAT-10.SPEC-006 (Duplicate Import Detection), so extraction still runs normally here.
- **Connectivity is lost mid-extraction** -- The Extracting state does not silently stall; if the request cannot complete, the screen shows the Offline/Degraded banner and holds the link as a draft, retrying automatically once connectivity returns.
- **No concurrent-edit conflict applies to this screen** -- This screen does not create, read, or update any existing shared Recipe record; it only initiates extraction, so no stale-write scenario exists here (the dependency map's Contention notes for Recipe apply starting at save time, covered by FEAT-10.SPEC-002/003/004).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (inbound) | Member arrives here from the library's "Import from link" entry point |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Link well-formedness and weekly-limit rules applied on Import tap |
| FEAT-10.SPEC-005 (Web Page Recipe Extraction) | Triggers (outbound) | Import tap, after validation passes, requests extraction |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Navigation (outbound) | Extraction success navigates here with the extracted draft |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Navigation (outbound) | Extraction failure, if the member proceeds manually, navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_started | entry source (library entry point), offline-at-submission (yes/no) | Link passes FEAT-10.SPEC-007's well-formedness check and extraction is requested | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into the import funnel |
| recipe_import_weekly_limit_blocked | current week's import count | Member's submission is blocked by FEAT-10.SPEC-007's 30-import-per-week guard | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so the misuse guard's real-world trigger rate is observable |

## Acceptance Criteria

**FEAT-10.SPEC-001-AC-01:** Given Maya is on the Import by Link screen, when she pastes a well-formed link and taps Import, then the screen shows "Reading the recipe... this usually takes a few seconds." and requests extraction via FEAT-10.SPEC-005.

**FEAT-10.SPEC-001-AC-02:** Given Sam is on the Import by Link screen with the link input empty, when he looks at the Import button, then it is disabled and cannot be tapped.

**FEAT-10.SPEC-001-AC-03:** Given Maya pastes a malformed link, when she taps Import, then the link input shows the error message defined by FEAT-10.SPEC-007 and no extraction is requested.

**FEAT-10.SPEC-001-AC-04:** Given Sam has already imported 30 recipes this week, when he attempts to submit another link, then he sees the weekly-limit block message defined by FEAT-10.SPEC-007 and no extraction is requested.

**FEAT-10.SPEC-001-AC-05:** Given extraction succeeds for Maya's submitted link, when the result returns, then she is navigated to FEAT-10.SPEC-002 (Review Extracted Recipe) with the extracted draft populated.

**FEAT-10.SPEC-001-AC-06:** Given extraction fails for Sam's submitted link, when the failure is reported, then the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link" actions.

**FEAT-10.SPEC-001-AC-07:** Given Sam sees the extraction-failed message, when he taps "Enter details manually", then he is navigated to FEAT-10.SPEC-003 (Manual Recipe Entry).

**FEAT-10.SPEC-001-AC-08:** Given Maya loses connectivity while the link input is filled, when she taps Import, then the banner "You're offline -- this link will be imported when you reconnect." appears and the link is held as a draft.

**FEAT-10.SPEC-001-AC-09:** Given Maya's link was held as a draft while offline, when connectivity returns, then validation and extraction proceed automatically without her needing to re-submit.

**FEAT-10.SPEC-001-AC-10:** Given Maya taps Import while extraction from a prior tap is still running, when she taps a second time, then the second tap has no effect and the progress indicator remains unchanged.

**FEAT-10.SPEC-001-AC-11:** Given the older-kid limited-login role (Later) does not see "Import from link" on FEAT-08.SPEC-001, when they nonetheless reach this screen directly, then it shows "Importing recipes isn't available on this profile." with a link back to the recipe library.

**FEAT-10.SPEC-001-AC-12:** Given Maya taps the back arrow with the link input filled but not yet submitted, when she confirms leaving, then she returns to FEAT-08.SPEC-001 and no import is started.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 7 (empty, filling, validating, validation error, extracting, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Review Extracted Recipe

## Overview

**Name:** Review Extracted Recipe
**ID:** FEAT-10.SPEC-002
**Type:** Screen
**Purpose:** Household member confirms or edits the extracted ingredients, steps, and cook time, then saves the recipe to the household's pool.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Displaying the draft ingredients, steps, and cook time extracted by FEAT-10.SPEC-005
- Allowing the member to edit any field before saving
- Validating the edited or confirmed content via FEAT-10.SPEC-007
- Triggering duplicate detection (FEAT-10.SPEC-006) and the safety re-check (FEAT-10.SPEC-008) on save

**Non-Goals:**
- Extracting the recipe content from the web page -- performed by FEAT-10.SPEC-005 (Web Page Recipe Extraction); this screen only displays and edits the result.
- Manual entry from a blank form -- handled by FEAT-10.SPEC-003 (Manual Recipe Entry), reached only when extraction fails.
- Editing an already-saved imported recipe later -- handled by FEAT-10.SPEC-004 (Edit Imported Recipe); this screen only covers the initial save of a fresh import.
- Setting a rough cost for the imported recipe -- product-features.md's Data Notes list this feature's owned fields as name, ingredients, steps, cook_time, origin, and owning_household only; rough_cost is not captured for imported content by this feature and the field remains unset for imported Recipe records.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-001 (Import by Link) | Extraction succeeds | The extracted draft: ingredients (quantity + unit), steps, cook_time, plus the source link |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Edit any field and save | -- |
| Sam (Other Adult Member) | Full screen | Edit any field and save | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | This screen is not reachable -- reaching it requires the "Import from link" entry point, which is not offered to this role (Recipe Library access is View, not Full) |
| Riley (Operator, support) | No | No | Not reachable -- Riley's Recipe Library access is View only and carries no import or save path |
| Unauthenticated | No | No | Redirected to the sign-in screen; the in-flight extracted draft is discarded since it was never associated with a signed-in session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the extracted draft and any edits made so far are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Review Recipe" with a back arrow (returns to FEAT-10.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A single-column form, pre-filled from the extracted draft, with the following fields in order (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-003 and FEAT-10.SPEC-004):
- Recipe Name (text input, required, pre-filled from extraction)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), pre-filled from extraction
- Steps (multi-line text input, method text in order, pre-filled from extraction)
- Cook Time (number input, minutes, required, pre-filled from extraction)

Below the form, a read-only line shows the source: "Imported from {source link's site name}" with the source link itself available on tap (opens the original page in a new context).

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If unsaved changes exist, show confirmation; otherwise navigate to FEAT-10.SPEC-001 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures edited text | Field shows edited text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type or edit | Captures edited values | Line shows edited values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; if it was the only ingredient, validation blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures edited method text | Field shows edited text | Standard input focus state |
| Cook Time input | Type | Captures edited value | Field shows edited value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, run duplicate detection via FEAT-10.SPEC-006. 3. If no duplicate, create the Recipe record and trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe saved" and navigate to the recipe's detail in FEAT-08.SPEC-002. Duplicate found: FEAT-10.SPEC-006 surfaces the existing recipe instead. Failure: inline error messages from FEAT-10.SPEC-007. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |
| Source link | Tap | Opens the original source page | -- | Original page opens in a new context; this screen remains open behind it |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> source link -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Recipe saved" toast is announced on success; on validation failure, focus moves to the first field in error; when a duplicate is found, the redirect to the existing recipe is announced.
- **Keyboard alternatives:** Every action, including adding and removing ingredient lines, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Populated (default) | Form fields pre-filled from the extracted draft, Save enabled | Screen opens after successful extraction | Member edits any field |
| Editing | Form fields contain member edits, Save enabled | Member types in or changes any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes, duplicate check (FEAT-10.SPEC-006) finds no match | Save completes or fails |
| Duplicate Found | Modal or inline message: "You've already imported this recipe." with a "View existing recipe" action | FEAT-10.SPEC-006 finds a link already saved for this household | Member taps "View existing recipe", navigating to that recipe in FEAT-08.SPEC-002; no new Recipe is created |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save operation fails | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this recipe will be saved when you reconnect." at top; form remains editable; Save queues the recipe locally | Connectivity lost while the screen is open | Connectivity restored -- the queued save proceeds automatically (validation, duplicate check, create) and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-10.SPEC-001 (Import by Link) | -- |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Duplicate found, "View existing recipe" tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Source link tap | External source page (opens outside the product) | -- |

## Data Model

**Creates:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time set from the confirmed/edited form; origin set to imported with the source link recorded; owning_household set to the saving member's household. Created only after FEAT-10.SPEC-007 validation passes and FEAT-10.SPEC-006 finds no duplicate.
**Reads:** The extracted draft passed from FEAT-10.SPEC-005 (not yet a persisted Recipe record).
**Updates:** None (this screen creates; FEAT-10.SPEC-004 owns later edits).
**Deletes:** None.

## Business Rules

- Duplicate detection (FEAT-10.SPEC-006) runs automatically on save -- the member cannot skip it.
- If a duplicate is found, no new Recipe is created; the existing recipe is surfaced instead (XBR-19).
- Field validation (FEAT-10.SPEC-007) rules are enforced -- the member cannot save with invalid data, including fewer than one ingredient.
- Once saved, the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan (XBR-01, XBR-19); the save itself completes regardless of the safety outcome, per FEAT-02's fail-closed posture.
- dietary_badges are never set by this screen -- they are computed live by FEAT-02 (Dietary Rules & Allergy Safety Engine) once the safety re-check completes.

## Edge Cases

- **Member navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **Network failure during save** -- Error banner: "Couldn't save this recipe. Check your connection and try again." with a Retry button. Form data preserved.
- **All ingredients removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Another household member (e.g., Sam) saves the same link from a separate device while this screen is open for Maya** -- No live conflict on this creation screen (no existing Recipe record is loaded here); duplicate detection (FEAT-10.SPEC-006) catches the collision at whichever save completes second, surfacing the already-saved recipe to that member instead of creating a second record. This is the concurrent-edit conflict entry for this screen, consistent with the Recipe entity's Contention note (reject-with-refresh at the second save) in feature-dependency-map.md.
- **Extracted content contains an ingredient with an unrecognizable name** -- The save still proceeds if all other validation passes (per FEAT-10.SPEC-007); the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture. No blocking error appears here -- the exclusion is enforced downstream by FEAT-02.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Navigation (inbound) | Extraction success navigates here with the draft |
| FEAT-10.SPEC-005 (Web Page Recipe Extraction) | References (inbound) | Supplies the extracted draft this screen displays |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-006 (Duplicate Import Detection) | Triggers (outbound) | Save action triggers duplicate detection |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Successful save triggers the safety re-check |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Successful save or duplicate resolution navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_succeeded | field-edit count (how many extracted fields the member changed before saving), ingredient count, cook time | Save completes and the Recipe record is created | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into how often extraction requires member correction |
| recipe_import_review_abandoned | last field focused | Member discards unsaved changes and navigates away | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into import funnel drop-off |

## Acceptance Criteria

**FEAT-10.SPEC-002-AC-01:** Given Maya's link extraction succeeded, when she opens the Review Extracted Recipe screen, then the recipe name, ingredients, steps, and cook time are pre-filled from the extracted draft.

**FEAT-10.SPEC-002-AC-02:** Given Sam is reviewing an extracted recipe, when he edits the cook time and taps Save, then the edited cook time is saved on the new Recipe record.

**FEAT-10.SPEC-002-AC-03:** Given Maya removes all ingredient lines, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-002-AC-04:** Given Sam taps Save on a valid extracted recipe with no existing duplicate, when duplicate detection (FEAT-10.SPEC-006) finds nothing, then the Recipe record is created, a "Recipe saved" toast appears, and he is navigated to FEAT-08.SPEC-002.

**FEAT-10.SPEC-002-AC-05:** Given Maya taps Save on a recipe whose link was already imported by her household, when duplicate detection (FEAT-10.SPEC-006) finds the existing recipe, then no new Recipe is created and she is shown "You've already imported this recipe." with "View existing recipe".

**FEAT-10.SPEC-002-AC-06:** Given Maya saves a recipe, when the save completes, then the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan.

**FEAT-10.SPEC-002-AC-07:** Given Sam is on this screen with unsaved edits, when he taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-002-AC-08:** Given Maya taps Save twice rapidly, when the first save is still in progress, then the second tap has no effect.

**FEAT-10.SPEC-002-AC-09:** Given Sam loses connectivity while editing the review form, when he taps Save, then the banner "You're offline -- this recipe will be saved when you reconnect." appears and the save is queued.

**FEAT-10.SPEC-002-AC-10:** Given Maya and Sam each submit the same link for import from separate devices at nearly the same time, when both reach Save, then whichever save completes second is shown the existing recipe instead of creating a duplicate, per FEAT-10.SPEC-006.

**FEAT-10.SPEC-002-AC-11:** Given Sam saves a recipe containing an ingredient the safety engine cannot recognize, when the save completes, then the recipe is saved to the household's pool but is excluded from every plan candidate path until the ingredient is clarified.

**FEAT-10.SPEC-002-AC-12:** Given the network fails during Maya's save attempt, when the failure is reported, then the error banner "Couldn't save this recipe. Check your connection and try again." appears with a Retry button, and her form data is preserved.

**FEAT-10.SPEC-002-AC-13:** Given Maya taps the source link shown below the form, when it opens, then the original page opens in a new context and this screen remains open behind it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 8 (populated, editing, validating, validation error, saving, duplicate found, error, offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Manual Recipe Entry

## Overview

**Name:** Manual Recipe Entry
**ID:** FEAT-10.SPEC-003
**Type:** Screen
**Purpose:** Household member types in ingredients, steps, and cook time by hand when link extraction fails, so the import never dead-ends.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Presenting a blank Recipe details form when extraction has failed and the member chooses to proceed manually
- Validating the entered content via FEAT-10.SPEC-007
- Triggering duplicate detection (FEAT-10.SPEC-006) and the safety re-check (FEAT-10.SPEC-008) on save

**Non-Goals:**
- Attempting extraction again -- retrying extraction happens by returning to FEAT-10.SPEC-001 (Import by Link) and re-submitting a link; this screen assumes extraction has already failed.
- Reviewing a system-extracted draft -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this screen's form always starts empty.
- General ad-hoc recipe creation unconnected to an attempted import -- product-features.md's Primary Flows & Alternates ties manual entry specifically to the "extraction failure" alternate path; there is no standalone "create a recipe from scratch" entry point in the product definition.
- Capturing a source link for the manually entered recipe -- since extraction failed, no page was successfully read; the recipe is saved with origin = imported but without a confirmed source link, distinct from FEAT-10.SPEC-002's saved recipes which always carry one.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-001 (Import by Link) | Member taps "Enter details manually" after an extraction failure | None -- form starts empty; the originally submitted link is not carried into this screen's data model |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Fill in the form and save | -- |
| Sam (Other Adult Member) | Full screen | Fill in the form and save | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | Not reachable -- reaching it requires FEAT-10.SPEC-001, which is not offered to this role (Recipe Library access is View, not Full) |
| Riley (Operator, support) | No | No | Not reachable -- Riley's Recipe Library access is View only and carries no import or save path |
| Unauthenticated | No | No | Redirected to the sign-in screen; any in-progress manual entry is discarded since it was never associated with a signed-in session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Recipe Details" with a back arrow (returns to FEAT-10.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A single-column form, empty by default (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-002 and FEAT-10.SPEC-004), with the following fields in order:
- Recipe Name (text input, required, empty)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), starting with one empty line
- Steps (multi-line text input, method text in order, empty)
- Cook Time (number input, minutes, required, empty)

Below the form, a note states: "We couldn't read the original page, so add the details yourself." No source link is shown, since none was successfully confirmed.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If any field is filled, show confirmation; otherwise navigate to FEAT-10.SPEC-001 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type | Captures entered values | Line shows entered values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; removing the only remaining line blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures method text | Field shows entered text | Standard input focus state |
| Cook Time input | Type | Captures entered value | Field shows entered value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, run duplicate detection via FEAT-10.SPEC-006. 3. If no duplicate, create the Recipe record and trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe saved" and navigate to FEAT-08.SPEC-002. Duplicate found: FEAT-10.SPEC-006 surfaces the existing recipe instead. Failure: inline error messages from FEAT-10.SPEC-007. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Recipe saved" toast is announced on success; on validation failure, focus moves to the first field in error; when a duplicate is found, the redirect to the existing recipe is announced.
- **Keyboard alternatives:** Every action, including adding and removing ingredient lines, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All form fields empty except one blank ingredient line, Save enabled | Screen opens after an extraction failure | Member types in any field |
| Filling | Form fields contain member input | Member types in any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes, duplicate check (FEAT-10.SPEC-006) finds no match | Save completes or fails |
| Duplicate Found | Modal or inline message: "You've already imported this recipe." with a "View existing recipe" action | FEAT-10.SPEC-006 finds a matching saved recipe for this household (matched on the recipe details the member entered, since no link is available for this path) | Member taps "View existing recipe", navigating to that recipe; no new Recipe is created |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save operation fails | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this recipe will be saved when you reconnect." at top; form remains editable; Save queues the recipe locally | Connectivity lost while the screen is open | Connectivity restored -- the queued save proceeds automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap. The well-formedness and weekly-limit rules governing link submission (also in FEAT-10.SPEC-007) do not apply here, since this screen has no link field.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no data entered) | FEAT-10.SPEC-001 (Import by Link) | -- |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Duplicate found, "View existing recipe" tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |

## Data Model

**Creates:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time set from the entered form; origin set to imported with no confirmed source link; owning_household set to the saving member's household. Created only after FEAT-10.SPEC-007 validation passes and FEAT-10.SPEC-006 finds no duplicate.
**Reads:** None (this screen starts from an empty form).
**Updates:** None (this screen creates; FEAT-10.SPEC-004 owns later edits).
**Deletes:** None.

## Business Rules

- Duplicate detection (FEAT-10.SPEC-006) runs automatically on save -- the member cannot skip it, matched against the entered recipe details since no source link exists for this path.
- If a duplicate is found, no new Recipe is created; the existing recipe is surfaced instead (XBR-19).
- Field validation (FEAT-10.SPEC-007) rules are enforced -- the member cannot save with invalid data, including fewer than one ingredient.
- Once saved, the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan (XBR-01, XBR-19); the save itself completes regardless of the safety outcome, per FEAT-02's fail-closed posture.
- This screen never dead-ends the import: reaching it always means the member has a working path to save a recipe even when extraction fails, per product-features.md's Error state definition.

## Edge Cases

- **Member navigates away with data entered** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **Network failure during save** -- Error banner: "Couldn't save this recipe. Check your connection and try again." with a Retry button. Form data preserved.
- **All ingredient lines removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Member manually enters details for a recipe that matches one already imported (e.g., re-typing a recipe from memory that was previously imported by link)** -- No live conflict on this creation screen (no existing record is loaded); duplicate detection (FEAT-10.SPEC-006) still runs on the entered details and, if it finds a match, surfaces the existing recipe instead of creating a second record. This is the concurrent-edit conflict entry for this screen, consistent with the Recipe entity's Contention note in feature-dependency-map.md.
- **Entered content contains an ingredient with an unrecognizable name** -- The save still proceeds if all other validation passes; the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Navigation (inbound) | Extraction failure, if the member proceeds manually, navigates here |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-006 (Duplicate Import Detection) | Triggers (outbound) | Save action triggers duplicate detection |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Successful save triggers the safety re-check |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Successful save or duplicate resolution navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_manual_fallback | -- | Member taps "Enter details manually" from FEAT-10.SPEC-001 and this screen opens | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into how often extraction fails badly enough to require manual fallback |
| recipe_import_succeeded | field count filled, ingredient count, cook time, entry_path: manual | Save completes and the Recipe record is created | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into the import funnel |

## Acceptance Criteria

**FEAT-10.SPEC-003-AC-01:** Given Maya's link extraction failed and she taps "Enter details manually", when the screen opens, then all form fields are empty except one blank ingredient line, and the note "We couldn't read the original page, so add the details yourself." is shown.

**FEAT-10.SPEC-003-AC-02:** Given Sam fills in the recipe name, one ingredient, steps, and cook time, when he taps Save, then duplicate detection (FEAT-10.SPEC-006) runs and, finding no match, the Recipe record is created and he sees a "Recipe saved" toast.

**FEAT-10.SPEC-003-AC-03:** Given Maya has removed every ingredient line, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-003-AC-04:** Given Sam manually enters details matching a recipe his household already imported, when he taps Save, then no new Recipe is created and he sees "You've already imported this recipe." with "View existing recipe".

**FEAT-10.SPEC-003-AC-05:** Given Maya saves a manually entered recipe, when the save completes, then the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan.

**FEAT-10.SPEC-003-AC-06:** Given Sam has entered data and taps the back arrow, when the confirmation dialog appears, then it reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-003-AC-07:** Given Maya taps Save twice rapidly, when the first save is still in progress, then the second tap has no effect.

**FEAT-10.SPEC-003-AC-08:** Given Sam loses connectivity while filling the manual entry form, when he taps Save, then the banner "You're offline -- this recipe will be saved when you reconnect." appears and the save is queued.

**FEAT-10.SPEC-003-AC-09:** Given Maya enters a recipe with an ingredient the safety engine cannot recognize, when the save completes, then the recipe is saved to the household's pool but is excluded from every plan candidate path until the ingredient is clarified.

**FEAT-10.SPEC-003-AC-10:** Given the network fails during Sam's save attempt, when the failure is reported, then the error banner "Couldn't save this recipe. Check your connection and try again." appears with a Retry button, and his form data is preserved.

**FEAT-10.SPEC-003-AC-11:** Given the older-kid limited-login role (Later) attempts to reach this screen directly without going through FEAT-10.SPEC-001, when the attempt is made, then it fails since this role's Recipe Library access is View, not Full, and no import entry point is available to it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (empty, filling, validating, validation error, saving, duplicate found, error, offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Edit Imported Recipe

## Overview

**Name:** Edit Imported Recipe
**ID:** FEAT-10.SPEC-004
**Type:** Screen
**Purpose:** Household member corrects an already-saved imported recipe's details, or removes it from the household's pool.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Loading an existing imported Recipe record's full content for correction
- Editing name, ingredients, steps, and cook time via the shared Recipe details form
- Removing (hard delete) an imported recipe from the household's pool, with confirmation
- Triggering the safety re-check (FEAT-10.SPEC-008) on every accepted edit before the recipe can appear in a plan again

**Non-Goals:**
- Editing a starter recipe -- product-features.md and the dependency map state starter recipes are read-only for households; this screen only opens for imported recipes (origin = imported).
- Undoing a removal or recovering a deleted imported recipe -- excluded per the feature's own Entity-Lifecycle Coverage Matrix: removal is immediate and permanent with no retention window and no restore path, since an imported recipe is the household's own data under their direct delete control, not shared maintained content; re-adding requires a fresh import.
- The initial save of a freshly imported recipe -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe) and FEAT-10.SPEC-003 (Manual Recipe Entry); this screen only covers edits and removal after the first save.
- Editing the recipe's source link -- the source link is fixed at import time and is not an editable field on this screen; correcting a link requires removing the recipe and re-importing.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-002 (Recipe Detail View) | Maya or Sam taps the header "Edit" button on an imported recipe owned by the household | The Recipe record's ID and current field values |

**Cross-feature reconciliation note:** product-features.md's Key Capabilities for this feature require an edit-and-remove path for imported recipes, and this entry point is the only place that path can start (FEAT-08.SPEC-002 is the sole detail view for any recipe, starter or imported). FEAT-08.SPEC-002 now shows an "Edit" control to Maya and Sam specifically when the recipe's origin is imported and it is owned by the household -- never on starter recipes, which stay read-only for households -- and navigates here carrying the recipe's ID and current field values (FEAT-08.SPEC-002 Interactions, Navigation Out and AC-13). The "Remove recipe" path stays on this screen, per this spec's Footer. Reconciled in Pass D (SG-05).

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Edit any field and save; remove the recipe | -- |
| Sam (Other Adult Member) | Full screen | Edit any field and save; remove the recipe | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | "Edit" is not shown on FEAT-08.SPEC-002 for this role (Recipe Library access is View, not Full); if this screen is reached directly, it shows "Editing recipes isn't available on this profile." with a link back to the recipe detail |
| Riley (Operator, support) | View only, through the read-only support view against an open Support Request | No | Edit and Remove controls are not shown to Riley; the recipe's content is visible read-only for diagnosis only |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-08.SPEC-001 (Recipe Library), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- unsaved edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Edit Recipe" with a back arrow (returns to FEAT-08.SPEC-002) and a "Save" action button (right-aligned).

**Body:** A single-column form, pre-filled from the existing saved recipe (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-002 and FEAT-10.SPEC-003), with the following fields in order:
- Recipe Name (text input, required, pre-filled)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), pre-filled
- Steps (multi-line text input, method text in order, pre-filled)
- Cook Time (number input, minutes, required, pre-filled)

Below the form, a read-only line shows the source: "Imported from {source link's site name}" with the source link available on tap, when the recipe carries one (recipes saved through FEAT-10.SPEC-003, Manual Recipe Entry, carry no source link and this line is omitted for them).

**Footer:** A "Remove recipe" destructive action, visually separated below the form.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header; "Remove recipe" spans the footer width.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If unsaved changes exist, show confirmation; otherwise navigate to FEAT-08.SPEC-002 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures edited text | Field shows edited text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type or edit | Captures edited values | Line shows edited values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; removing the only remaining line blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures edited method text | Field shows edited text | Standard input focus state |
| Cook Time input | Type | Captures edited value | Field shows edited value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, update the Recipe record. 3. Trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe updated" and navigate to FEAT-08.SPEC-002. Failure: inline error messages from FEAT-10.SPEC-007, or a conflict dialog if the record changed since load. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |
| "Remove recipe" | Tap | Shows a confirmation dialog | Dialog appears | Dialog: "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel" |
| Confirmation dialog "Remove" | Tap | Deletes the Recipe record | Screen closes | Toast "Recipe removed" and navigate to FEAT-08.SPEC-001 (Recipe Library) |
| Confirmation dialog "Cancel" | Tap | Dismisses the dialog | Dialog closes | Returns to the edit form unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> source link (when present) -> Save -> Remove recipe.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save and removal feedback:** The "Recipe updated" and "Recipe removed" toasts are announced on success; the removal confirmation dialog's message is announced when it opens; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action, including adding/removing ingredient lines and the removal confirmation, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form fields pre-filled from the current saved recipe, Save enabled | Screen opens from FEAT-08.SPEC-002 | Member edits any field or taps Remove recipe |
| Editing | Form fields contain member edits, Save enabled | Member types in or changes any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes | Save completes or fails |
| Conflict | Dialog: "This recipe was updated on another device while you were editing. Review the latest version before saving." with "View Latest" and "Keep Editing" options | Save is rejected because the record changed since load | Member taps "View Latest" (reloads the record; local edits discarded after confirmation) or "Keep Editing" (dialog closes, edits remain, member may retry Save) |
| Removal Confirmation | Dialog: "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel" | Member taps "Remove recipe" | Member confirms (recipe deleted) or cancels (dialog closes) |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save or remove operation fails for a reason other than a conflict | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; form remains editable; Save and Remove queue locally | Connectivity lost while the screen is open | Connectivity restored -- the queued change proceeds automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap. The link well-formedness and weekly-import-limit rules do not apply here, since this screen edits an already-saved recipe rather than starting a new import.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Successful removal | FEAT-08.SPEC-001 (Recipe Library Browse & Search) | FEAT-08 |
| Source link tap | External source page (opens outside the product) | -- |

## Data Model

**Creates:** None.
**Reads:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time, origin, owning_household -- for the imported recipe being edited.
**Updates:** Recipe record -- name, ingredients, steps, cook_time, subject to FEAT-10.SPEC-007 validation; origin and owning_household are never changed by this screen.
**Deletes:** Recipe record -- hard delete, immediate and permanent, on confirmed removal; no cascade to Planned Meals that already used the recipe (their history stands unchanged, per the feature's Entity-Lifecycle Coverage Matrix).

## Business Rules

- Field validation (FEAT-10.SPEC-007) rules are enforced on every edit -- the member cannot save with invalid data, including fewer than one ingredient.
- Every accepted edit re-triggers the safety re-check (FEAT-10.SPEC-008) before the edited recipe can appear in a plan again (XBR-19).
- Removal is a hard delete with no retention window and no restore path -- re-adding the same recipe requires a fresh import via FEAT-10.SPEC-001, which FEAT-10.SPEC-006 treats as a new, non-duplicate import.
- Removing a recipe drops it from the household's candidate pool immediately; it does not affect Planned Meals that already used it, per the feature's Entity-Lifecycle Coverage Matrix.
- Concurrent edits to the same imported recipe resolve reject-with-refresh, per the Recipe entity's Contention note in feature-dependency-map.md.

## Edge Cases

- **Recipe changed by another household member between load and save** -- Save is rejected with the Conflict dialog: "This recipe was updated on another device while you were editing. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for the Recipe entity.
- **Member navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **All ingredient lines removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Recipe is removed by one member while another has it open in this edit screen** -- The second member's next Save or Remove attempt is rejected with "This recipe no longer exists in your household's pool." and they are returned to FEAT-08.SPEC-001; their in-progress edits are discarded.
- **Member removes a recipe that is currently used by a Planned Meal in the current week** -- The removal proceeds as specified (no cascade); the Planned Meal keeps its existing reference and cooked/past history unchanged, but the recipe is no longer available for any future candidate pool or re-selection, per the feature's Entity-Lifecycle Coverage Matrix.
- **Edited recipe now contains an ingredient the safety engine cannot recognize** -- The save still proceeds if all other validation passes; the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (inbound/outbound) | Member arrives from and returns to the recipe detail |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Every accepted edit re-triggers the safety re-check |
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (outbound) | Successful removal navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| imported_recipe_removed | had_planned_meal_history (yes/no) | Removal is confirmed and the Recipe record is deleted | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into household-driven pool cleanup |

Note: `imported_recipe_edited` is emitted by FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit), which this screen's Save action triggers -- see that spec's Analytics and Success Signals section. This screen's own edit-save action carries no separate analytics event beyond the removal event above.

## Acceptance Criteria

**FEAT-10.SPEC-004-AC-01:** Given Maya opens Edit Imported Recipe on a recipe she previously imported, when the screen loads, then all fields are pre-filled with the recipe's current saved values.

**FEAT-10.SPEC-004-AC-02:** Given Sam edits the cook time of an imported recipe and taps Save, when validation passes, then the Recipe record is updated, a "Recipe updated" toast appears, and the safety re-check (FEAT-10.SPEC-008) runs.

**FEAT-10.SPEC-004-AC-03:** Given Maya removes all ingredient lines while editing, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-004-AC-04:** Given Sam is editing an imported recipe, when another household member updates the same recipe on a different device before Sam saves, then his Save is rejected with "This recipe was updated on another device while you were editing. Review the latest version before saving." and "View Latest" / "Keep Editing" options.

**FEAT-10.SPEC-004-AC-05:** Given Maya taps "Remove recipe", when the confirmation dialog appears, then it reads "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel".

**FEAT-10.SPEC-004-AC-06:** Given Maya confirms removal of an imported recipe, when the deletion completes, then the recipe is deleted from the household's pool, a "Recipe removed" toast appears, and she is navigated to FEAT-08.SPEC-001.

**FEAT-10.SPEC-004-AC-07:** Given Sam cancels the removal confirmation dialog, when he taps "Cancel", then the dialog closes and the recipe remains unchanged on the edit form.

**FEAT-10.SPEC-004-AC-08:** Given a Planned Meal already used an imported recipe that Maya then removes, when the removal completes, then the Planned Meal's history is unaffected and the recipe is no longer available for future selection.

**FEAT-10.SPEC-004-AC-09:** Given the older-kid limited-login role (Later) does not see "Edit" on FEAT-08.SPEC-002 for an imported recipe, when they reach this screen directly, then it shows "Editing recipes isn't available on this profile." with a link back to the recipe detail.

**FEAT-10.SPEC-004-AC-10:** Given Riley (Operator) is viewing a household's recipe under an open Support Request, when Riley opens the recipe, then no Edit or Remove controls are shown.

**FEAT-10.SPEC-004-AC-11:** Given Maya has unsaved edits and taps the back arrow, when the confirmation dialog appears, then it reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-004-AC-12:** Given Sam loses connectivity while editing, when he taps Save, then the banner "You're offline -- changes will be saved when you reconnect." appears and the change is queued.

**FEAT-10.SPEC-004-AC-13:** Given Maya edits a recipe to include an ingredient the safety engine cannot recognize, when the save completes, then the recipe stays saved but is excluded from every plan candidate path until the ingredient is clarified.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 9 (loaded, editing, validating, validation error, saving, conflict, removal confirmation, error, offline) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Integration Spec: Web Page Recipe Extraction

## Overview

**Name:** Web Page Recipe Extraction
**ID:** FEAT-10.SPEC-005
**Type:** Integration
**Purpose:** Reads a pasted web page through the product's web-page recipe extraction capability and returns ingredients, steps, and cook time, or reports that extraction failed.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Sending a submitted web link to the web-page recipe extraction capability for a single recipe import
- Receiving extracted ingredients, steps, and cook time, or a failure report
- User-facing behavior on FEAT-10.SPEC-001 when this capability is slow, unavailable, or rejects a link
- Disclosure to the user about what is shared with the capability (the pasted link itself)

**Non-Goals:**
- Choosing the web-page recipe extraction vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Reviewing or editing the extracted draft -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this spec only defines what crosses the boundary with the extraction capability.
- Determining whether extracting content from a given third-party site is legally permitted -- BRIEF.md's Open Questions names this as an unresolved product/legal decision to settle before v1 ships; this spec defines the functional extraction contract approved for build, not that resolution.
- Bulk extraction of multiple links in one request -- excluded per scope-boundaries.md SC-12: the brief names only per-link recipe import; this integration handles exactly one link per request.

## Capability Category

**Category:** Web-page recipe extraction
**Dependency Source:** ASMP-36 -- "Web-page recipe extraction capability, v1 -- Required to read a pasted recipe link and extract ingredients, steps, and cook time" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Web-page recipe extraction (ASMP-36, v1)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-10; Integration Spec: FEAT-10.SPEC-005)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A household member pastes a web link and receives a pre-filled draft of ingredients, steps, and cook time to review | Import by link | FEAT-10.SPEC-001 (Import by Link), FEAT-10.SPEC-002 (Review Extracted Recipe) |
| A member sees a brief, explained progress indicator while the page is read, rather than a bare wait | Import by link | FEAT-10.SPEC-001 (Import by Link) |
| A link that cannot be parsed prompts the member to enter the recipe details manually instead of failing silently | Import by link | FEAT-10.SPEC-003 (Manual Recipe Entry) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| The pasted web link | Not a stored entity at send time -- the raw link text the member entered on FEAT-10.SPEC-001 | Member taps Import and FEAT-10.SPEC-007's well-formedness and weekly-limit checks pass | The capability must fetch and parse the page at this address to extract recipe content |

No household data, member data, dietary rule data, or any other personal or account data ever leaves the product through this capability -- only the pasted link itself is sent.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Extracted ingredients (each with a quantity and unit) | Extraction succeeds | Recipe draft (not yet persisted) -- ingredients, held by FEAT-10.SPEC-002 for review before save |
| Extracted steps | Extraction succeeds | Recipe draft (not yet persisted) -- steps, held by FEAT-10.SPEC-002 for review before save |
| Extracted cook time | Extraction succeeds | Recipe draft (not yet persisted) -- cook_time, held by FEAT-10.SPEC-002 for review before save |
| Extraction failure report | Extraction fails or the page layout cannot be parsed | No Recipe draft is populated; FEAT-10.SPEC-001 routes the member to FEAT-10.SPEC-003 (Manual Recipe Entry) instead |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Extraction succeeded | The capability successfully reads and parses the linked page | No persisted data changes -- the extracted ingredients, steps, and cook time populate an unsaved draft | FEAT-10.SPEC-001's progress indicator completes and the member is navigated to FEAT-10.SPEC-002 with the draft pre-filled | FEAT-10.SPEC-001, FEAT-10.SPEC-002 |
| Extraction failed | The capability cannot reach the page, or the page layout cannot be parsed into recipe content | No persisted data changes; no draft is created | FEAT-10.SPEC-001 shows "We couldn't read that page." with "Enter details manually" and "Try a different link" | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |

Both outcomes are direct consequences (populate a draft, or route to a fallback screen) with no branching decisions or cross-entity effects, so processing stays inline in this spec rather than routing to a standalone Automation spec.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-10.SPEC-001 (Import by Link) | The progress indicator "Reading the recipe... this usually takes a few seconds." remains visible; if the wait exceeds what the feature's own timing contract describes as typical, no separate warning replaces it -- the indicator continues until the capability responds or the request ultimately times out into the Extraction failed path. The link input and Import button stay disabled during the wait. | The request cannot be sent; the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link" -- the same experience as an extraction failure, since the member's next useful action is identical either way. | The capability reports that the page's layout could not be parsed; the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link". |

No other screen sends requests to or displays results from this capability -- FEAT-10.SPEC-002 and FEAT-10.SPEC-003 only consume the draft or the failure outcome already delivered here; they cannot themselves experience a degradation state from this capability.

## Consent and Disclosure

- **Link-sharing disclosure** -- The helper line on FEAT-10.SPEC-001 ("We'll pull the ingredients, steps, and cook time from the page -- you'll be able to review and edit before it's saved.") discloses, in plain terms, that the pasted page is read to extract content, at the moment the member is about to submit a link. No separate consent gate blocks submission -- pasting a link and tapping Import is the member's affirmative action to proceed, consistent with this being a link the member has chosen to share, not incidental personal data.
- **What is never shared** -- No household data, member data, dietary rule data, or any other personal or account data crosses this boundary in either direction; only the pasted link leaves the product, and only extracted recipe content (ingredients, steps, cook time) or a failure report returns.

## Edge Cases

- **Extraction succeeds after the member has already navigated away from FEAT-10.SPEC-001** -- The result is held for the same import attempt; if the member returns to the flow by re-submitting the same link, extraction runs again rather than reusing a stale prior result, since no draft persists across navigation away from an in-progress request.
- **The same link is submitted twice in quick succession (double tap already prevented on the screen, but two separate submissions)** -- Each submission is an independent extraction request; whichever completes is shown to the member on FEAT-10.SPEC-002, and duplicate handling at save time is FEAT-10.SPEC-006's responsibility, not this integration's.
- **Extraction request times out mid-read** -- Treated as an Extraction failed event; the member is routed to the same "We couldn't read that page." experience as any other failure, with no half-populated draft ever shown.
- **Capability goes down mid-extraction** -- If no result was confirmed received, FEAT-10.SPEC-001 shows the extraction-failed message and no draft is created -- no half-created Recipe state, since nothing is persisted until FEAT-10.SPEC-002 or FEAT-10.SPEC-003's own save action.
- **Member submits a link while offline** -- Per FEAT-10.SPEC-001's Offline/Degraded state, the link is held as a draft locally and this integration is not invoked until connectivity returns, at which point the request proceeds normally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Triggered by (inbound) | Import tap, after validation, requests extraction |
| FEAT-10.SPEC-001 (Import by Link) | Affects (outbound) | Progress indicator, success navigation, and failure messaging surface here |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Affects (outbound) | Successful extraction populates this screen's draft |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Affects (outbound) | Extraction failure, if the member proceeds manually, routes here |

## Analytics and Success Signals

- **recipe_import_failed** (failure reason category: unreachable / unparseable / timeout) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so extraction reliability -- a known competitor weak point named in product-features.md's Rationale -- is observable.
- **recipe_extraction_completed** (outcome: succeeded / failed; duration bracket) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained to monitor whether the "typically a few seconds" timing contract stated in the feature's own States field is being met.

## Acceptance Criteria

**FEAT-10.SPEC-005-AC-01:** Given Maya submits a well-formed link on FEAT-10.SPEC-001, when the web-page recipe extraction capability successfully parses the page, then ingredients, steps, and cook time populate the draft shown on FEAT-10.SPEC-002.

**FEAT-10.SPEC-005-AC-02:** Given Sam submits a link whose page cannot be parsed, when the capability reports the failure, then FEAT-10.SPEC-001 shows "We couldn't read that page." with "Enter details manually" and "Try a different link".

**FEAT-10.SPEC-005-AC-03:** Given Maya submits a link to a page the capability cannot reach, when the capability is down, then she sees the same "We couldn't read that page." message and can proceed to FEAT-10.SPEC-003.

**FEAT-10.SPEC-005-AC-04:** Given Sam submits a link while the capability is responding slowly, when the response has not yet returned, then the progress indicator "Reading the recipe... this usually takes a few seconds." remains visible and the Import button stays disabled.

**FEAT-10.SPEC-005-AC-05:** Given Maya submits a link, when extraction is requested, then only the pasted link itself is sent to the capability -- no household, member, or dietary data is included.

**FEAT-10.SPEC-005-AC-06:** Given Sam is about to submit his first link, when he views FEAT-10.SPEC-001, then the helper line discloses that the page will be read to extract ingredients, steps, and cook time before any link is sent.

**FEAT-10.SPEC-005-AC-07:** Given an extraction request times out mid-read, when the timeout occurs, then Maya sees the standard extraction-failed experience and no partially populated draft is ever shown.

**FEAT-10.SPEC-005-AC-08:** Given the capability goes down after Sam's request is sent but before any result is confirmed, when the request fails to complete, then no Recipe record or draft exists in any half-created state.

**FEAT-10.SPEC-005-AC-09:** Given Maya submits the same link twice in quick succession as two separate requests, when both complete, then each is treated as an independent extraction result, with duplicate handling deferred to FEAT-10.SPEC-006 at save time.

**FEAT-10.SPEC-005-AC-10:** Given Sam submits a link while offline, when the screen is offline, then this capability is not invoked until connectivity returns, per FEAT-10.SPEC-001's Offline/Degraded state.

**FEAT-10.SPEC-005-AC-11:** Given Maya navigates away from FEAT-10.SPEC-001 after submitting a link but before extraction completes, when she later re-submits the same link, then a fresh extraction request runs rather than reusing any prior result.

**FEAT-10.SPEC-005-AC-12:** Given extraction succeeds for Sam's link, when the draft is populated, then no data beyond ingredients, steps, and cook time is carried into FEAT-10.SPEC-002 from this integration.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 2 | 2 |
| Degradation Paths | 3 (1 screen) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: Recipe Import Validation & Rate Limit Rules

## Overview

**Name:** Recipe Import Validation & Rate Limit Rules
**ID:** FEAT-10.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs link well-formedness, ingredient/step length caps, the minimum-one-ingredient save requirement, and the 30-import-per-week misuse guard for the imported slice of the Recipe entity.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link
**Governed Entity:** Recipe (imported slice)

## Scope and Non-Goals

**In Scope:**
- Field-level validation for every field this feature owns on an imported Recipe record (name, source link, ingredients, steps, cook_time)
- The minimum-one-ingredient requirement for saving an import
- The 30-import-per-week misuse guard on the Import action
- Authorization rules for Import, Edit, and Remove on the imported slice of the Recipe entity
- Default and derived values this feature sets on create (origin, owning_household)

**Non-Goals:**
- Validating starter recipe content -- starter recipes are seeded and maintained by FEAT-08.SPEC-004 (Starter Recipe Content Seeding & Maintenance); this spec governs only the imported slice households create.
- Determining whether an ingredient is safe against a household's allergies or religious rules -- that determination belongs to FEAT-02 (Dietary Rules & Allergy Safety Engine, XBR-01); this spec only requires that ingredient data be complete enough for FEAT-02's check to run, and defines the save-time consequence of an ingredient FEAT-02 cannot recognize as an eligibility exclusion, not a save-blocking validation failure.
- A lifetime or tier-based cap on the number of imports -- explicitly excluded per product-features.md's Rationale: research flagged a lifetime cap as a hard limit competitors' users criticized, so the founder deliberately rejected one; only the 30-per-week misuse guard defined here applies.
- Validating the recipe's rough cost or dietary badges -- feature-overview.md's Shared Context states this feature never writes rough_cost or dietary_badges for imported content; those fields carry no validation obligation from this spec.

## Governed Entity

**Entity:** Recipe (imported slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The recipe's title |
| source_link | text | The web address the recipe was imported from (link-based imports only) |
| ingredients | list (quantity + unit + name per line) | The recipe's ingredient lines |
| steps | text | The recipe's method, in order |
| cook_time | number | Minutes required to cook the recipe, required for schedule fit |
| rough_cost | number | Shown in the household's currency |
| dietary_badges | derived | Computed live by FEAT-02 against the household's dietary rules |
| origin | enum | starter library or imported (with source link) |
| owning_household | reference | The household that owns this imported recipe |
| prep_requirements | text | Early-prep needs such as defrosting |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-10.SPEC-001 | Import by Link | Link well-formedness and weekly-limit checks on Import tap |
| FEAT-10.SPEC-002 | Review Extracted Recipe | Field-level validation on field blur and Save tap; duplicate/save authorization |
| FEAT-10.SPEC-003 | Manual Recipe Entry | Field-level validation on field blur and Save tap; duplicate/save authorization |
| FEAT-10.SPEC-004 | Edit Imported Recipe | Field-level validation on field blur and Save tap; edit/remove authorization; concurrent-edit conflict handling |
| FEAT-10.SPEC-006 | Duplicate Import Detection | Reads owning_household boundary and source_link/name fields this spec defines |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | Reads the completed, validated ingredient data this spec requires before triggering the safety re-check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, max 120 characters | Always | On blur | "Recipe name is required" / "Recipe name must be 120 characters or fewer" | Yes |
| source_link | Must be a well-formed web address (a valid scheme and host) | Import started from FEAT-10.SPEC-001 (link-based import); not applicable to manual entry, which carries no source_link | On Import tap (FEAT-10.SPEC-001) | "Enter a valid web link" | Yes |
| ingredients | At least one ingredient line required | Always | On blur (list changes) and on submit | "At least one ingredient is required" | Yes |
| ingredients (per line: quantity) | Required, positive number | Always, per line | On blur | "Enter a quantity for this ingredient" | Yes |
| ingredients (per line: unit) | Required, non-empty | Always, per line | On blur | "Enter a unit for this ingredient" | Yes |
| ingredients (per line: name) | Required, non-empty, max 80 characters | Always, per line | On blur | "Enter an ingredient name" / "Ingredient name must be 80 characters or fewer" | Yes |
| steps | Max 4,000 characters | Always | On blur | "Steps must be 4,000 characters or fewer" | Yes |
| cook_time | Required, positive whole number of minutes, max 600 | Always | On blur | "Enter the cook time in minutes" / "Cook time must be 600 minutes or fewer" | Yes |
| rough_cost | No validation beyond data type -- this feature never sets this field for imported content | Always | -- | -- | -- |
| dietary_badges | No validation beyond data type -- computed live by FEAT-02; this feature never writes it | Always | -- | -- | -- |
| prep_requirements | No validation beyond data type -- this feature does not capture or set this field for imported content | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Source link required only for link-based imports | source_link, entry path | source_link is required and validated only when the save originated from FEAT-10.SPEC-001/FEAT-10.SPEC-002 (link-based); a save originating from FEAT-10.SPEC-003 (manual entry) carries no source_link and is not blocked by its absence | "Enter a valid web link" (link-based path only; not shown on the manual entry path) |
| Ingredient completeness for the safety check | ingredients (quantity, unit, name per line) | An ingredient with a name the safety engine cannot recognize does not block the save (this rule's own validation only requires quantity, unit, and name to be present); it instead triggers an eligibility exclusion in FEAT-10.SPEC-008/FEAT-02 (XBR-01), which is out of this spec's authority | N/A -- no save-blocking error; the exclusion is communicated as an eligibility state, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Import a recipe (create) | Maya (Organiser), Sam (Other Adult Member) | Always, subject to the weekly import limit below | -- |
| Import a recipe (create) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no path to any import screen |
| Import a recipe (create) | Jordan (older kid, limited login -- Later) | Never | "Import from link" is not shown (Recipe Library access is View, not Full); a direct attempt shows "Importing recipes isn't available on this profile." |
| Import a recipe (create) | Riley (Operator, support) | Never | No import entry point exists in the read-only support view |
| Edit an imported recipe | Maya (Organiser), Sam (Other Adult Member) | Always, for any imported recipe owned by their household (not ownership-restricted to the importing member -- the Recipe Library column of the Access Matrix grants both roles Full access to the household's shared pool) | -- |
| Edit an imported recipe | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile |
| Edit an imported recipe | Jordan (older kid, limited login -- Later) | Never | "Edit" is not shown (Recipe Library access is View, not Full); a direct attempt shows "Editing recipes isn't available on this profile." |
| Edit an imported recipe | Riley (Operator, support) | Never | Edit controls are not shown in the read-only support view |
| Remove an imported recipe | Maya (Organiser), Sam (Other Adult Member) | Always, for any imported recipe owned by their household | -- |
| Remove an imported recipe | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile |
| Remove an imported recipe | Jordan (older kid, limited login -- Later) | Never | "Remove recipe" is not shown (Recipe Library access is View, not Full) |
| Remove an imported recipe | Riley (Operator, support) | Never | Remove controls are not shown in the read-only support view |

**Rate limit condition on Import (create):** For Maya and Sam, the Import action is allowed only while the household's import count for the current rolling week is below 30. At 30 imports in the current rolling week, further imports are blocked for every household member until the rolling week resets, with the denied behavior: "You've reached this week's import limit (30). You can import more recipes once next week starts." No lifetime cap applies to either household member (product-features.md's Validation & Limits field, and the explicit product decision recorded in Non-Goals).

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| origin | Set to "imported" | On create (FEAT-10.SPEC-002 or FEAT-10.SPEC-003 save) | No |
| owning_household | Set to the saving member's household | On create | No |
| source_link | Set to the link the member submitted on FEAT-10.SPEC-001 | On create, link-based path only | No |
| source_link | Left unset | On create, manual-entry path (FEAT-10.SPEC-003) | Not applicable -- no value is captured to override |

## Business Rules

- Field validation (this spec) runs before duplicate detection (FEAT-10.SPEC-006) -- invalid data is never checked for duplicates.
- The 30-import-per-week guard counts imports across both link-based and manual-entry saves for the household, reset on a rolling weekly basis; it is the only import limit the product defines, per the explicit no-lifetime-cap decision in product-features.md's Rationale.
- An ingredient with a name the safety engine cannot recognize does not block the save; it results in the recipe being excluded from every plan candidate path until clarified, per XBR-01's fail-closed posture, enforced through FEAT-10.SPEC-008 and FEAT-02, not through this spec's field validation.
- All validation rules apply identically whether the save originates from FEAT-10.SPEC-002 (review after extraction), FEAT-10.SPEC-003 (manual entry), or FEAT-10.SPEC-004 (later edit) -- the product definition establishes no save-path-specific exceptions beyond the source_link rule above.
- Authorization for Edit and Remove is not ownership-restricted to the member who originally imported the recipe -- both Maya and Sam hold Full access to the household's shared Recipe Library per the Access Matrix, so either may edit or remove a recipe the other imported.

## Edge Cases

- **Recipe name at exactly 120 characters** -- Passes validation. 121 characters shows the length error.
- **Ingredient name at exactly 80 characters** -- Passes validation. 81 characters shows the length error.
- **Steps at exactly 4,000 characters** -- Passes validation. 4,001 characters shows the length error.
- **Cook time at exactly 600 minutes** -- Passes validation. 601 minutes shows the length error; 0 or a negative value shows the required/positive-number error.
- **Household at exactly 29 imports for the current week** -- The 30th import is allowed; the household is not blocked until the count reaches 30.
- **Household at exactly 30 imports for the current week** -- The next import attempt is blocked with the weekly-limit message; the count does not reset until the rolling week boundary passes.
- **Ingredient list has entries with quantity and unit but an ingredient name the safety engine cannot recognize** -- Save is not blocked by this spec's own field rules (name is present and within length); the recipe is excluded from plan eligibility by FEAT-10.SPEC-008/FEAT-02 instead.
- **Manual entry save with no source_link value at all** -- Passes validation; the cross-field rule explicitly exempts the manual-entry path from the source_link requirement.
- **Ownership of an imported recipe "changes" conceptually when Sam edits a recipe Maya originally imported** -- No authorization boundary is crossed: Edit and Remove are not ownership-gated between Maya and Sam, so this is simply an allowed action, not an edge case requiring special handling.
- **A household member attempts to import while at 30/30 and simultaneously another member's in-flight import (started just under the limit) completes, pushing the count to 30 before the first member's check runs** -- The check reads the count at the moment each Import action's validation runs; a member whose check runs after the count reaches 30 is blocked, even if their submission started slightly before the count-pushing import completed. This is a first-decision-wins outcome consistent with the automation processing order in FEAT-10.SPEC-006.

## Acceptance Criteria

**FEAT-10.SPEC-007-AC-01:** Given Maya leaves the recipe name empty on FEAT-10.SPEC-002, when she moves to the next field, then she sees "Recipe name is required."

**FEAT-10.SPEC-007-AC-02:** Given Maya enters a recipe name of exactly 120 characters, when she moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-03:** Given Maya enters a recipe name of 121 characters, when she moves to the next field, then she sees "Recipe name must be 120 characters or fewer."

**FEAT-10.SPEC-007-AC-04:** Given Sam pastes a malformed link on FEAT-10.SPEC-001, when he taps Import, then he sees "Enter a valid web link" and no extraction is requested.

**FEAT-10.SPEC-007-AC-05:** Given Sam is on FEAT-10.SPEC-003 (Manual Recipe Entry), when he saves a recipe with no source link entered, then no "Enter a valid web link" error appears, since manual entries carry no source_link requirement.

**FEAT-10.SPEC-007-AC-06:** Given Maya removes every ingredient line on FEAT-10.SPEC-002, when she taps Save, then she sees "At least one ingredient is required" and the save does not proceed.

**FEAT-10.SPEC-007-AC-07:** Given Maya leaves an ingredient's quantity empty, when she moves to the next field, then she sees "Enter a quantity for this ingredient."

**FEAT-10.SPEC-007-AC-08:** Given Sam enters an ingredient name of exactly 80 characters, when he moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-09:** Given Sam enters steps text of exactly 4,000 characters, when he moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-10:** Given Maya enters steps text of 4,001 characters, when she moves to the next field, then she sees "Steps must be 4,000 characters or fewer."

**FEAT-10.SPEC-007-AC-11:** Given Sam enters a cook time of 0 minutes, when he moves to the next field, then he sees "Enter the cook time in minutes."

**FEAT-10.SPEC-007-AC-12:** Given Maya enters a cook time of exactly 600 minutes, when she moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-13:** Given Sam's household has imported 29 recipes this week, when he submits a 30th, then the import is allowed and proceeds to extraction.

**FEAT-10.SPEC-007-AC-14:** Given Maya's household has imported 30 recipes this week, when any household member attempts a 31st import, then they see "You've reached this week's import limit (30). You can import more recipes once next week starts." and no extraction is requested.

**FEAT-10.SPEC-007-AC-15:** Given a household has been active for many months with no lifetime import cap, when they attempt an import within this week's allowance, then the import is allowed regardless of their total imports to date.

**FEAT-10.SPEC-007-AC-16:** Given Maya (Organiser) is on FEAT-10.SPEC-002, when she saves a recipe that passes all field rules, then the save proceeds to duplicate detection (FEAT-10.SPEC-006).

**FEAT-10.SPEC-007-AC-17:** Given the older-kid limited-login role (Later) attempts to import a recipe, when the attempt is made, then it is denied since Import is never allowed for this role, and "Importing recipes isn't available on this profile." is shown if the screen is reached directly.

**FEAT-10.SPEC-007-AC-18:** Given Riley (Operator) is viewing a household's recipe library, when Riley looks for an import, edit, or remove control, then none is shown, since all three actions are Never for this role.

**FEAT-10.SPEC-007-AC-19:** Given Sam (Other Adult Member) opens a recipe Maya originally imported, when he edits and saves it, then the save is allowed, since Edit is not ownership-restricted between Maya and Sam.

**FEAT-10.SPEC-007-AC-20:** Given Maya saves a recipe with an ingredient named clearly (e.g., "chicken breast") that the safety engine cannot recognize, when the save completes, then it is not blocked by this spec's field rules -- the recipe saves and is excluded from plan eligibility by FEAT-10.SPEC-008/FEAT-02 instead.

**FEAT-10.SPEC-007-AC-21:** Given Maya saves a recipe from FEAT-10.SPEC-002 (link-based), when the record is created, then origin is set to "imported", owning_household is set to Maya's household, and source_link is set to the submitted link, none of which she can override.

**FEAT-10.SPEC-007-AC-22:** Given Sam saves a recipe from FEAT-10.SPEC-003 (manual entry), when the record is created, then origin is set to "imported", owning_household is set to Sam's household, and source_link remains unset.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 11 | 11 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 10 | 10 |



# Automation Spec: Safety Re-check on Import Save/Edit

## Overview

**Name:** Safety Re-check on Import Save/Edit
**ID:** FEAT-10.SPEC-008
**Type:** Automation
**Purpose:** Runs the household's allergy/religious-rule safety check on every initial save and every later edit of an imported recipe before it can appear in a plan.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Triggering the household's safety check (owned by FEAT-02) immediately after an imported recipe is created or edited
- Defining the import-side consequence of a Pass, an incomplete-data Fail, or a rule-conflict Fail
- Ensuring an edited imported recipe cannot appear in a plan again until it passes the re-check (XBR-19)

**Non-Goals:**
- Performing the allergy/religious-rule determination itself -- that determination, including how ingredients are matched against dietary rules, is FEAT-02's authority (XBR-01); this spec governs only when the check is triggered for imported recipes, not how it decides (feature-overview.md, Shared Validation).
- Computing or storing the dietary badge shown on a recipe -- feature-overview.md states dietary_badges is never written by this feature; FEAT-02 computes it live at view time once this automation's trigger has run.
- Re-checking starter recipes -- starter content's safety-relevant completeness is FEAT-08.SPEC-004's responsibility at seeding time; this spec covers only the imported slice of the Recipe entity.
- Notifying the household when a recipe fails the re-check -- product-features.md's Communications field states import is a self-initiated, in-app action with no notifications; a failed recipe is surfaced only as an ineligibility explanation the next time it is viewed, not through a push or email notification.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Imported recipe created (link-based) | FEAT-10.SPEC-002 (Review Extracted Recipe) | Fires immediately after the Recipe record is created (no duplicate found, FEAT-10.SPEC-006) | The newly created Recipe's ingredients (quantity + unit + name), owning_household |
| Imported recipe created (manual entry) | FEAT-10.SPEC-003 (Manual Recipe Entry) | Fires immediately after the Recipe record is created (no duplicate found, FEAT-10.SPEC-006) | The newly created Recipe's ingredients (quantity + unit + name), owning_household |
| Imported recipe edited | FEAT-10.SPEC-004 (Edit Imported Recipe) | Fires immediately after an accepted edit updates the Recipe record | The updated Recipe's ingredients (quantity + unit + name), owning_household |

## Processing Logic

1. Receive the Recipe record just created or edited, including its confirmed ingredient list (quantity, unit, and name per line) and owning_household.
2. Hand the ingredient data off to FEAT-02 (Dietary Rules & Allergy Safety Engine) for evaluation against the owning household's dietary rules.
3. Receive FEAT-02's determination, which is one of: every ingredient recognized and no hard-rule conflict (Pass); one or more ingredients missing a quantity/unit or carrying a name FEAT-02 cannot recognize (Fail -- incomplete data); an ingredient conflicts with a stated allergy or religious rule held by a member of the owning household (Fail -- rule conflict).
4. If Pass, no further action is required from this automation -- the recipe is eligible to appear in candidate pools from this moment, and FEAT-02 computes its dietary badge live at each subsequent view.
5. If Fail (either reason), the recipe remains saved in the household's pool but is excluded from every candidate path (FEAT-03 generation, FEAT-23 manual picks, FEAT-04 swap alternatives) until a further edit through FEAT-10.SPEC-004 resolves the issue and this automation re-runs.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Safety check passed | FEAT-02 finds every ingredient recognized and no hard-rule conflict | None stored on the Recipe record (dietary_badges is computed live by FEAT-02, never persisted) | None shown at save time (import has no notifications); the recipe's next view on FEAT-08.SPEC-002 shows the standard "checked against allergies" badge and "always check labels" disclaimer | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Safety check failed -- incomplete ingredient data | One or more ingredients are missing a quantity/unit, or carry a name FEAT-02 cannot recognize | None stored directly; the recipe's eligibility for any candidate pool is excluded on every subsequent evaluation until corrected | None shown at save time; the recipe's detail view (FEAT-08.SPEC-002) shows an ineligibility explanation in place of the standard badge | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Safety check failed -- allergy/religious-rule conflict | An ingredient conflicts with a hard dietary rule held by a member of the owning household | None stored directly; excluded from every candidate path for this household under its current dietary rules | None shown at save time; the recipe's detail view shows an ineligibility explanation naming that it does not currently meet the household's dietary rules | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Automation failure (hand-off to FEAT-02 cannot complete) | A processing error prevents the check from running | None -- the recipe remains saved | No error is shown to the household; the recipe is treated as not-yet-checked and excluded from candidate pools, since XBR-01's fail-closed default treats an unconfirmed check the same as a failed one | FEAT-08.SPEC-002, FEAT-03, FEAT-23 |

## Data Model

**Reads:** Recipe record -- ingredients (quantity + unit + name), owning_household; Dietary Rule records (via FEAT-02) belonging to members of the owning household.
**Creates:** None.
**Updates:** None on the Recipe record itself -- dietary_badges is a live-computed value FEAT-02 derives at view time, never written by this automation.
**Deletes:** None.

## Business Rules

- Every initial save of an imported recipe (link-based or manual) triggers this automation exactly once, immediately (XBR-01, XBR-19).
- Every accepted edit of an already-saved imported recipe re-triggers this automation before the edited recipe can appear in a plan again (XBR-19).
- The save or edit itself always completes regardless of the safety outcome -- a Fail never blocks the save or edit action, per FEAT-02's fail-closed posture applying to plan eligibility, not to the ability to keep the recipe in the household's pool.
- Removing an imported recipe (FEAT-10.SPEC-004) does not trigger this automation -- there is nothing to re-check once the record no longer exists.
- The determination itself (Pass, incomplete, or conflict) is entirely FEAT-02's authority; this automation only defines when the check runs for imported recipes and what happens to import-side eligibility as a result.

## Edge Cases

- **Recipe is removed while this automation's check is still in flight** -- The check's result is discarded on completion; there is no Recipe record left to update eligibility for, and no error is surfaced to any household member.
- **Household's dietary rules change (e.g., a new allergy added) after this automation already returned a Pass for a recipe** -- This automation does not re-run on its own from a dietary-rule change; that re-evaluation is XBR-02's responsibility (a new or tightened hard rule triggers its own re-check across the household's plan and recipes), not a trigger this spec defines.
- **Concurrent trigger firing (Maya and Sam each edit different fields of the same imported recipe from separate devices at effectively the same time)** -- Only one edit succeeds at the data layer, per FEAT-10.SPEC-004's reject-with-refresh conflict resolution; this automation fires exactly once, against the single edit that was actually accepted. The rejected edit never reaches this automation, since it never became a persisted change.
- **Trigger fires while a previous run is in flight for the same recipe** -- FEAT-10.SPEC-004's Save button is disabled during save, so a second edit-triggered run cannot start for the same recipe until the first save (and its automation run) completes. A run triggered by this recipe's initial creation (FEAT-10.SPEC-002 or FEAT-10.SPEC-003) and a later edit-triggered run cannot overlap either, since the edit screen only becomes reachable once the recipe already exists and its creation-time run has resolved.
- **Two different imported recipes are saved by different household members at the same time** -- Each recipe's safety check runs independently against the same household's dietary rules; neither run affects or waits on the other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Triggered by (inbound) | Successful creation triggers this automation |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Triggered by (inbound) | Successful creation triggers this automation |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | Triggered by (inbound) | Every accepted edit re-triggers this automation |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggers (outbound) | Hands off ingredient data for the safety determination |
| FEAT-08.SPEC-002 (Recipe Detail View) | Affects (outbound) | The badge or ineligibility explanation shown reflects this automation's outcome |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | Candidate pool eligibility reflects this automation's outcome |
| FEAT-23 (Manual Weekly Planning) | Affects (outbound) | Candidate pool eligibility reflects this automation's outcome |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | Swap alternative eligibility reflects this automation's outcome |

## Analytics and Success Signals

- **imported_recipe_edited** (fields changed, safety outcome: passed / failed_incomplete / failed_conflict) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so households and support can see how often an edit changes a recipe's plan eligibility.
- **safety_recheck_completed** (trigger: create / edit; outcome: passed / failed_incomplete / failed_conflict) -- supports success-metrics.md: "Zero Allergy Incidents" (this event is the operational trace confirming every imported-recipe save and edit actually passed through the app-enforced check the metric's zero-incidents target depends on).
- **safety_recheck_failed_processing** (trigger: create / edit) -- N/A -- no success-metrics.md metric tracks automation-failure volume directly; retained so the fail-closed guarantee (an unconfirmed check excludes the recipe) is observable when the hand-off itself breaks.

## Acceptance Criteria

**FEAT-10.SPEC-008-AC-01:** Given Maya saves a freshly extracted recipe on FEAT-10.SPEC-002 with every ingredient complete and no allergy conflict, when the save completes, then this automation runs and the recipe becomes eligible to appear in candidate pools immediately.

**FEAT-10.SPEC-008-AC-02:** Given Sam saves a manually entered recipe missing a unit on one ingredient, when the save completes, then this automation returns a Fail (incomplete data), and the recipe is excluded from every candidate path until corrected.

**FEAT-10.SPEC-008-AC-03:** Given Maya saves a recipe containing an ingredient that conflicts with a household member's stated allergy, when the save completes, then this automation returns a Fail (rule conflict), and the recipe is excluded from every candidate path under the household's current dietary rules.

**FEAT-10.SPEC-008-AC-04:** Given Sam edits an already-saved imported recipe's ingredients, when the edit is accepted, then this automation re-runs before the edited recipe can appear in a plan again, per XBR-19.

**FEAT-10.SPEC-008-AC-05:** Given Maya's recipe previously failed the safety check for an unrecognizable ingredient, when she edits the recipe to clarify that ingredient and saves, then this automation re-runs and, if it now passes, the recipe becomes eligible for candidate pools again.

**FEAT-10.SPEC-008-AC-06:** Given Sam removes an imported recipe, when the removal completes, then this automation does not run, since there is no record left to check.

**FEAT-10.SPEC-008-AC-07:** Given the hand-off to FEAT-02 fails due to a processing error, when Maya's recipe was just saved, then the recipe remains saved but is treated as not-yet-checked and excluded from candidate pools until a retry succeeds.

**FEAT-10.SPEC-008-AC-08:** Given Maya and Sam each attempt to edit the same imported recipe from separate devices at effectively the same time, when FEAT-10.SPEC-004's conflict resolution accepts only one edit, then this automation fires exactly once, against the accepted edit only.

**FEAT-10.SPEC-008-AC-09:** Given a household adds a new allergy rule after an imported recipe already passed this automation's check, when the new rule is added, then this automation does not itself re-run -- any re-evaluation of the recipe under the new rule is governed by XBR-02, not this spec.

**FEAT-10.SPEC-008-AC-10:** Given a recipe passes this automation's check, when a household member later views it on FEAT-08.SPEC-002, then no data was written by this automation to the Recipe record -- the "checked against allergies" badge is computed live by FEAT-02 at that view.

**FEAT-10.SPEC-008-AC-11:** Given the safety check itself fails to complete (processing error) on an initial save, when the failure occurs, then no notification is sent to the household, consistent with the feature's no-notifications communications posture -- the recipe simply shows an ineligibility explanation on its next view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (create link-based, create manual, edit) | 3 |
| Outcome Paths | 4 (passed, failed incomplete, failed conflict, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-19.SPEC-003
spec_name: Past Plan Reuse
spec_slug: past-plan-reuse
parent_feature: FEAT-19
parent_feature_name: Weekly Plan History
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Past Plan Reuse

## Overview

**Name:** Past Plan Reuse
**ID:** FEAT-19.SPEC-003
**Type:** Automation
**Purpose:** System copies a selected past week's archived plan structure into a chosen future week, re-runs the current household allergy and religious-rule safety check against today's dietary rules, and hands the safety-clean pre-filled week to Manual Weekly Planning for further editing.
**Parent Feature:** FEAT-19 -- Weekly Plan History

## Scope and Non-Goals

**In Scope:**
- Copying the past week's per-night recipe structure (which recipe was in which night's slot) into a target future week Maya chooses
- Re-running XBR-01's allergy and religious-rule safety check against the household's current Dietary Rules for every copied meal, since rules may have changed since the archived week
- Excluding any copied meal that no longer passes the safety check, and leaving its night's slot empty and flagged for the household to fill
- Handing the resulting pre-filled future week to Manual Weekly Planning (FEAT-23) once the safety-clean copy is produced

**Non-Goals:**
- Copying the archived Grocery List directly into the new week -- excluded because a fresh Grocery List is recalculated once the reused week is edited in Manual Weekly Planning, per FEAT-06's ownership of Grocery List recalculation (feature-overview.md, Entity-Lifecycle Coverage Matrix); this automation only produces the plan structure, never a list
- Copying Ratings, pantry callouts, or cost/time figures verbatim from the archived week -- excluded because these are recomputed fresh for the target week from current Recipe and Pantry Item data once the week lands in Manual Weekly Planning (FEAT-23), rather than carried forward as stale archived values
- Allowing any role other than Maya to trigger a reuse copy -- excluded per this feature's Access field (product-features.md: "Maya (Full, including re-using a past week)... Sam (View)... older-kid login (View)") and enforced by FEAT-19.SPEC-004 (History Access & Reuse Authorization), which this automation does not re-derive independently

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Maya taps "Reuse this week" and picks a target future week | FEAT-19.SPEC-002 (Past Week Detail View) | Fires only when the acting user is Maya (Organiser), per FEAT-19.SPEC-004; the target week must be within the planning horizon Manual Weekly Planning allows (up to one week ahead, per product-features.md, Validation & Limits) | Source week's archived Weekly Plan identifier and its full set of archived Planned Meals (night, recipe, meal_kind); the chosen target future week's identifier |

## Processing Logic

1. Receive the source week's archived Weekly Plan identifier and the target future week's identifier from the triggering screen.
2. Read every archived Planned Meal belonging to the source week: its night, its recipe, and its meal_kind (dinner or leftover lunch).
3. For each archived Planned Meal, in night order:
   a. Read the household's current Dietary Rules (today's allergies, religious rules, and per-person vegetarian settings) for every household member.
   b. Re-run the safety check (XBR-01) for the archived meal's recipe against those current rules, exactly as any other path onto the plan is checked.
   c. If the recipe passes, copy it into the same night's slot in the target future week.
   d. If the recipe fails the check, exclude it from the copy: leave that night's slot in the target week empty and mark it as flagged for the household to fill.
4. For each leftover-lunch Planned Meal linked to a copied dinner, copy the leftover-lunch slot only if its source dinner was itself successfully copied (per Business Rules, below).
5. Once every night has been evaluated, assemble the resulting week: a partially or fully pre-filled future week with each night either holding a safety-checked recipe or flagged empty.
6. Hand the resulting week to Manual Weekly Planning (FEAT-23) as the target future week's starting content, ready for further editing.
7. Signal the triggering screen that the copy is complete and where to find the result (the target week's view in FEAT-23).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Full safety-clean copy | Every archived meal in the source week still passes today's safety check | Target future week's Weekly Plan is created/updated with every night filled from the source week | "Copying this week..." indicator clears; Maya is taken to the target week in Manual Weekly Planning, showing every night filled | FEAT-23 (Manual Weekly Planning, target week view) |
| Partial copy -- one or more meals excluded | At least one archived meal's recipe no longer passes the household's current safety check | Target future week's Weekly Plan is created/updated with passing nights filled and failing nights left empty and flagged | Maya is taken to the target week in Manual Weekly Planning; flagged empty nights show a plain notice, e.g., "This night was left empty -- the previous recipe no longer fits your household's dietary rules," inviting her to fill it | FEAT-23 (Manual Weekly Planning, target week view) |
| No eligible meals to copy | Every archived meal in the source week fails today's safety check | Target future week's Weekly Plan is created empty | Maya is taken to the target week in Manual Weekly Planning showing all seven nights flagged empty, with the same per-night notice as the partial-copy outcome | FEAT-23 (Manual Weekly Planning, target week view) |
| Automation failure | The copy cannot complete (e.g., the source week's archived data or the target week cannot be read or written) | No partial write persists -- the target future week is left exactly as it was before the attempt | Error banner on FEAT-19.SPEC-002: "Couldn't copy this week. Try again." with a Retry option; Maya remains on the past week's detail view | FEAT-19.SPEC-002 (Past Week Detail View) |

## Data Model

**Reads:** Weekly Plan (archived, source week) -- week, and every linked archived Planned Meal (night, recipe, meal_kind, status). Dietary Rule -- every current rule for every household Member Profile, read fresh at copy time (not from the archived week). Recipe -- ingredient and current pool-membership data, needed to re-run the safety check.
**Creates:** Weekly Plan (target future week) -- created if the target week does not yet have one, with origin recorded as manually built (the Weekly Plan `origin` field is a two-value enum -- AI-generated or manually built, per feature-dependency-map.md -- and a reuse copy is not AI-generated, so it is recorded as manually built, consistent with the resulting week being finished editing in Manual Weekly Planning); Planned Meal (target week) -- one created per night that passes the safety check, each referencing the copied recipe, meal_kind carried from the source, and status set to Proposed/Picked so it is editable in Manual Weekly Planning.
**Updates:** Weekly Plan (target future week) -- if a Weekly Plan already exists for the target week (e.g., a partially started manual week), its empty nights are filled by this copy; nights the household had already picked are left untouched by this automation (see Business Rules).
**Deletes:** None.

## Business Rules

- XBR-01 (feature-dependency-map.md): every path onto the plan, including a reused past week, passes the same app-enforced allergy and religious-rule check before anyone sees it; the check fails closed -- a recipe with incomplete ingredient data is excluded, never shown unchecked.
- A leftover-lunch Planned Meal is copied only if its linked source dinner was itself successfully copied; a leftover lunch cannot exist without its source dinner in the copy, consistent with FEAT-11's rule that a leftover lunch links to exactly one source dinner (feature-dependency-map.md, XBR-10).
- This automation never overwrites a night in the target future week that the household has already picked through Manual Weekly Planning before the reuse copy runs; it fills only nights that are empty at the moment the copy executes, so an in-progress manual pick is never silently replaced.
- Only Maya may trigger this automation, per FEAT-19.SPEC-004 (History Access & Reuse Authorization); the trigger source screen (FEAT-19.SPEC-002) does not present the launching control to any other role.
- The target week must fall within the one-week-ahead planning horizon Manual Weekly Planning enforces (product-features.md, Validation & Limits); this automation does not extend or bypass that horizon.

## Edge Cases

- **Source week contained a night with nothing planned** -- That night is skipped entirely in the copy (there is nothing to check or copy); the target week's corresponding night stays exactly as it was before the copy started.
- **All members' dietary rules are unchanged since the archived week** -- Every meal passes the re-check and the outcome is a full safety-clean copy; the re-check still runs in full, since XBR-01 requires the check on every path regardless of whether rules have changed.
- **A copied recipe has since been removed from the household's pool entirely (FEAT-10)** -- Treated the same as a safety-check failure for that night: the night is left empty and flagged, since a removed recipe cannot be re-checked or copied.
- **Concurrent trigger firing (Maya reuses the same past week into two different target weeks in quick succession)** -- Each launch is evaluated against its own distinct target week identifier; both copies proceed independently since they write to different Weekly Plan records, and neither blocks the other.
- **Trigger fires while a previous run for the same target week is still in flight** -- A second reuse launch targeting the same future week cannot start while the first is still copying: FEAT-19.SPEC-002 disables "Reuse this week" and shows the "Copying this week..." indicator for the duration of the first run, so no second run for that target week can be initiated until the first completes or fails.
- **Household's dietary rules change mid-copy (a rule is edited in FEAT-01 while this automation is running)** -- The re-check for each meal uses the current rules as read at the moment that meal is evaluated (step 3a); a rule change that lands after a given meal has already been checked and copied does not retroactively re-open that meal within this run -- XBR-02's mid-week re-check on the household's live Weekly Plan (not the reuse process itself) covers any rule change once the target week is active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Past Week Detail View) | Triggered by (inbound) | "Reuse this week" plus a target-week pick launches this automation |
| FEAT-19.SPEC-004 (History Access & Reuse Authorization) | References (inbound) | Gates the reuse action to Maya only |
| FEAT-23.SPEC-* (Manual Weekly Planning) | Affects (outbound) | Receives the pre-filled target future week for further editing |
| FEAT-02.SPEC-* (Dietary Rules & Allergy Safety Engine) | References (outbound) | Re-runs the current household safety check (XBR-01) against every copied meal |
| FEAT-10.SPEC-* (Recipe Import from Web Link) | References (outbound) | A recipe removed from the household's pool by this feature is treated as a copy-time exclusion |

## Analytics and Success Signals

- **past_plan_reused** (source week identifier, target week identifier, count of nights copied, count of nights flagged empty) -- N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); this event is retained as an operational signal so the gap is visible rather than silently dropped, per Phase 2.5 Category 8
- **past_plan_reuse_meal_excluded** (excluded recipe reference, night, reason: rule-change / recipe-removed) -- N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); see past_plan_reused
- **past_plan_reuse_failed** (source week identifier, target week identifier, failure point) -- N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); see past_plan_reused

## Acceptance Criteria

**FEAT-19.SPEC-003-AC-01:** Given Maya picks a past week where every meal still passes her household's current dietary rules, when she chooses a target future week, then every night of that target week is filled with the corresponding archived recipe and she is taken to the target week in Manual Weekly Planning.

**FEAT-19.SPEC-003-AC-02:** Given Maya reuses a past week where one meal's recipe no longer passes a dietary rule added since the week was archived, when the copy completes, then that night is left empty and flagged with the notice that the previous recipe no longer fits the household's dietary rules, while every other night is filled.

**FEAT-19.SPEC-003-AC-03:** Given Maya reuses a past week where every meal now fails the household's current dietary rules, when the copy completes, then the target week's Weekly Plan is created with all seven nights flagged empty.

**FEAT-19.SPEC-003-AC-04:** Given Maya reuses a past week that includes a leftover lunch linked to a dinner that no longer passes the safety check, when the copy completes, then neither the dinner nor its linked leftover lunch is copied into the target week.

**FEAT-19.SPEC-003-AC-05:** Given the reuse copy cannot complete because the source week's archived data cannot be read, when the failure occurs, then Maya sees "Couldn't copy this week. Try again." on FEAT-19.SPEC-002 with a Retry option, and the target future week is left exactly as it was before the attempt.

**FEAT-19.SPEC-003-AC-06:** Given Maya has already hand-picked three nights of the target future week in Manual Weekly Planning before reusing a past week, when the reuse copy runs, then only the four empty nights are filled from the source week and her three existing picks remain untouched.

**FEAT-19.SPEC-003-AC-07:** Given Sam or Jordan (older kid, limited login) is viewing a past week's detail, when either looks for a way to trigger this automation, then no "Reuse this week" control is available to launch it, per FEAT-19.SPEC-004.

**FEAT-19.SPEC-003-AC-08:** Given a recipe in the source week has since been removed entirely from the household's recipe pool, when the copy evaluates that night, then the night is left empty and flagged, the same as a rule-change exclusion.

**FEAT-19.SPEC-003-AC-09:** Given Maya launches a reuse copy targeting a future week and immediately launches a second reuse copy targeting a different future week, when both run, then each proceeds independently and both complete without blocking each other.

**FEAT-19.SPEC-003-AC-10:** Given a reuse copy targeting a specific future week is still in progress, when Maya attempts to launch a second reuse copy at the same target week before the first finishes, then "Reuse this week" remains disabled with the "Copying this week..." indicator shown, preventing a second run from starting for that target week.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (full copy, partial copy, no eligible meals, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

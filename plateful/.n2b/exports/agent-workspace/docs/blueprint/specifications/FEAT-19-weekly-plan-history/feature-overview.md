---
document_type: feature-overview
feature_number: FEAT-19
feature_name: Weekly Plan History
feature_slug: weekly-plan-history
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 4
screen_count: 2
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Weekly Plan History

## Summary

**Feature:** Weekly Plan History
**ID:** FEAT-19
**Description:** Household can look back at previous weeks' plans and grocery lists.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** Not named directly in the brief, but a natural extension once several weeks of plans exist — useful for households wanting to repeat a past week or remember what they ate. Nice-to-Have because the core weekly loop (plan, swap, shop) functions completely without it. Phased to v1, once households have accumulated enough history for it to be useful.

**Key Capabilities:**
- Browse past weeks — Household navigates back through previously completed weekly plans
- Re-use a past plan — Household can copy a liked past week's plan into a future week as a starting point

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-19.SPEC-001 | Weekly Plan History Browse | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household browses backward through previously archived weekly plans in a chronological list |
| FEAT-19.SPEC-002 | Past Week Detail View | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household views a single past week's full plan and archived grocery list, with the reuse action available to Maya |
| FEAT-19.SPEC-003 | Past Plan Reuse | Automation | Maya | System copies a selected past week's plan into a chosen future week and re-runs the current allergy and religious-rule safety check before handing the pre-filled week to Manual Weekly Planning |
| FEAT-19.SPEC-004 | History Access & Reuse Authorization | Logic/Rule | All | Governs which roles may view history and which role may trigger reuse, consistently across both screens and the reuse automation |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Browse past weeks | FEAT-19.SPEC-001, FEAT-19.SPEC-002 | SPEC-001 lists archived weeks chronologically; SPEC-002 opens the full detail of a selected week | Phase 2 (Explicit) |
| Re-use a past plan | FEAT-19.SPEC-003 | SPEC-003 copies the selected past week into a future week and re-runs the safety check before the week is editable in Manual Weekly Planning | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-19.SPEC-004 | History Access & Reuse Authorization | Phase 5 (Rule-Constraint Discovery) | The Access field names four differentiated view levels (Full for Maya including reuse, View for Sam/older-kid/Riley, None for unauthorized visitors) applied consistently across both screens plus a reuse-only-for-Maya gate on the automation — a rule shared across three specs, crossing the standalone-spec threshold |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own and creates or updates no entity directly — its Connected Entities are read-only (product-features.md: "Weekly Plan (read), Grocery List (read)"). The reuse capability initiates a copy but the resulting record is created and owned by FEAT-23 (Manual Weekly Planning) once the pre-filled future week lands there; see Side-Effect Inventory and Cross-Feature Touchpoints. Accordingly, no full CRUD matrix applies; all entities this feature touches are listed below as Referenced Entities.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-19.SPEC-001, FEAT-19.SPEC-002, FEAT-19.SPEC-003 | Archived Weekly Plan records (created by FEAT-03 or FEAT-23) are listed, opened in detail, and — for SPEC-003 — read as the source copied into a new future week; the write that lands the copy as a live plan is completed in FEAT-23, per the dependency map's "Updated by ... FEAT-19 (re-use of a past week as a starting point)" entry and the FEAT-19 → FEAT-23 navigation connection |
| Grocery List | FEAT-19.SPEC-001, FEAT-19.SPEC-002 | Archived Grocery List records (created by FEAT-06) are shown alongside each past week; not copied by the reuse action itself — a fresh list is recalculated once the reused week is edited in FEAT-23, per FEAT-06's ownership of Grocery List recalculation |
| Recipe | FEAT-19.SPEC-002, FEAT-19.SPEC-003 | Past weeks reference the Recipes they contained; SPEC-002 displays recipe names, dietary badges, and details within the archived plan; SPEC-003 reads them as the source of the safety re-check. A recipe later removed from the household's pool by FEAT-10 is handled per the Offline/Degraded and reuse-failure notes on SPEC-002 and SPEC-003 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household opens Weekly Plan History | Retrieve and list archived Weekly Plan records for the household, most recent first | Inline in triggering screen | FEAT-19.SPEC-001 |
| Household selects a past week from the list | Retrieve the full archived Weekly Plan and its linked Grocery List for detail display | Inline in triggering screen | FEAT-19.SPEC-002 |
| Maya taps "Reuse this week" and picks a target future week | Copy the past week's plan structure into the target week, then re-run the current household allergy and religious-rule safety check (XBR-01) against today's dietary rules before the week becomes visible for editing | Standalone Automation | FEAT-19.SPEC-003 |
| Past Plan Reuse completes a safety-clean copy | Hand the pre-filled future week to Manual Weekly Planning for further editing | Cross-feature — logged in touchpoints | FEAT-23 responsibility |
| Past Plan Reuse's safety re-check finds a meal that no longer passes (household's dietary rules changed since the archived week) | Exclude the affected meal from the copied week and flag the empty slot for the household to fill, consistent with XBR-01's fail-closed rule; never silently include an unchecked meal | Standalone Automation (part of reuse outcome handling) | FEAT-19.SPEC-003 |
| Non-member or unauthorized visitor attempts to reach history | Deny access; show only sign-in/invitation/referral screens per the product's unauthorized-visitor rule | Standalone Logic/Rule | FEAT-19.SPEC-004 |
| Sam, older-kid (Later), or Riley opens history or a past week | Grant read access, hide the reuse control | Standalone Logic/Rule | FEAT-19.SPEC-004 |

## Shared Context

**Shared Entities:**
- Weekly Plan (archived) — read by SPEC-001 (list), SPEC-002 (detail), and SPEC-003 (reuse source); never updated in place by this feature.
- Grocery List (archived) — read by SPEC-001 (summary) and SPEC-002 (full detail); never copied directly by SPEC-003.
- Recipe — read within plan detail by SPEC-002 and as the reuse safety-check subject by SPEC-003; dietary badges shown are recomputed per household by FEAT-02, not stored on the archived record.

**Shared UI Patterns:**
- Past-week card/list-item — the chronological entry used in SPEC-001's list and as the navigation source into SPEC-002's detail; both must render the same week identifier, status, and at-a-glance summary (e.g., meal count) consistently.
- "Reuse this week" control — appears on SPEC-002 (and, optionally, as a quick action on SPEC-001's list-item), gated by SPEC-004's role rule so it is visible only to Maya.
- Empty/Loading/Error/Offline state language — SPEC-001 and SPEC-002 both draw their non-populated states from the same States field (product-features.md) and should describe them identically: "no past weeks yet" for empty, a couple-second loading budget, a retry-offering error, and offline availability limited to previously viewed history.

**Shared Validation:**
- SPEC-004 defines the role-based view and reuse-authorization rules referenced by SPEC-001 (list visibility), SPEC-002 (detail visibility and reuse-control visibility), and SPEC-003 (reuse-action gate) — none of the three re-derives the rule independently.

## Internal Dependency Map

```
SPEC-001 (Weekly Plan History Browse) -> [user selects a past week] -> SPEC-002 (Past Week Detail View)
SPEC-002 (Past Week Detail View) -> [Maya taps "Reuse this week", picks a target future week] -> SPEC-003 (Past Plan Reuse)
SPEC-003 (Past Plan Reuse) -> [safety-clean copy produced] -> FEAT-23 (Manual Weekly Planning, future week pre-filled)
SPEC-001 (Weekly Plan History Browse) -> [determines list visibility and reuse-control visibility using] -> SPEC-004 (History Access & Reuse Authorization)
SPEC-002 (Past Week Detail View) -> [determines detail visibility and reuse-control visibility using] -> SPEC-004 (History Access & Reuse Authorization)
SPEC-003 (Past Plan Reuse) -> [gates the reuse action using] -> SPEC-004 (History Access & Reuse Authorization)
```

**Default Entry:** SPEC-001 (Weekly Plan History Browse) — the screen shown when the household navigates to Weekly Plan History.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-19.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Reads archived Weekly Plan records originally generated by FEAT-03 | History list loads |
| FEAT-19.SPEC-001 | Inbound | FEAT-06 (Shared Grocery List) | Reads archived Grocery List records | History list loads |
| FEAT-19.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) / FEAT-06 (Shared Grocery List) | Reads the full archived plan and list for the selected week | Household opens a past week's detail |
| FEAT-19.SPEC-003 | Outbound | FEAT-23 (Manual Weekly Planning) | Copied past week becomes a pre-filled future week, editable through Manual Weekly Planning | Maya taps "Reuse this week" |
| FEAT-19.SPEC-003 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Re-runs the current household allergy and religious-rule check (XBR-01) on every meal in the copied week before it is shown | Copy action fires |

## Non-Functional Notes

**Data volumes / growth:** History is retained for the life of the household account with no cap on how far back a household may browse; as households accumulate several years of weekly plans within a base of several thousand households, browsing must stay equally responsive as this history grows (assumptions-constraints.md ASMP-24; product-features.md Validation & Limits).

**Responsiveness:** Past weeks load within a couple of seconds, per the feature's States field, and the product's general heavier-moment responsiveness expectation applies to this retrieval as well (assumptions-constraints.md ASMP-23).

**Data sensitivity / privacy:** Weekly Plan and Grocery List records carry household personal data (what the family eats) — private to the household, never sold or used for advertising, and this posture applies equally to archived history as to active data (assumptions-constraints.md ASMP-26). History remains available after a downgrade to the free tier and is exportable and deletable only as part of full account export/deletion (scope-boundaries.md SC-18; FEAT-18).

**Compliance flags:** N/A — no compliance regime beyond the product's general no-sale, no-advertising privacy posture is named for this feature in assumptions-constraints.md.

## Non-Goals

- **Editing an archived week's contents in place** — Excluded because the Weekly Plan lifecycle treats a week as Archived once it ends (feature-dependency-map.md, Weekly Plan lifecycle) and this feature's Primary Flows only support copying a past week into a future week, never modifying the historical record itself (product-features.md, Primary Flows & Alternates).
- **Importing plans or grocery lists from other meal-planning or list apps into history** — Excluded per scope-boundaries.md SC-12: the product supports only per-link recipe import and manual list entry; bulk import from other apps is out of scope for the whole product, including populating or supplementing plan history.
- **User-initiated deletion or purge of individual past weeks** — Excluded per scope-boundaries.md SC-18: history is kept for the life of the household account with no cap on how far back it is browsable, and this retention is a documented trust commitment (paywalling or losing users' own history produced a documented trust backlash); only a full account deletion (FEAT-18) removes history, never a per-week purge.

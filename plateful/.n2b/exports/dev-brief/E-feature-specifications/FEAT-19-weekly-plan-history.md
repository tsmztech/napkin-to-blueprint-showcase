# FEAT-19 — Weekly Plan History

This chapter covers FEAT-19, Weekly Plan History, a Nice-to-Have-tier feature. It contains 4 specifications carrying 49 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-19.SPEC-001 | Weekly Plan History Browse | screen | 12 |
| FEAT-19.SPEC-002 | Past Week Detail View | screen | 14 |
| FEAT-19.SPEC-003 | Past Plan Reuse | automation | 10 |
| FEAT-19.SPEC-004 | History Access & Reuse Authorization | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Weekly Plan History Browse

## Overview

**Name:** Weekly Plan History Browse
**ID:** FEAT-19.SPEC-001
**Type:** Screen
**Purpose:** Household member browses backward through the household's previously archived weekly plans in a chronological list, most recent first, and selects one to open its full detail.
**Parent Feature:** FEAT-19 -- Weekly Plan History

## Scope and Non-Goals

**In Scope:**
- Listing archived Weekly Plan records for the household, most recent first, with no cap on how far back the household may browse (feature-overview.md, Non-Functional Notes; product-features.md, Validation & Limits)
- A chronological list-item summary per past week (week identifier, status, at-a-glance meal count) consistent with the shared "past-week card/list-item" pattern (feature-overview.md, Shared UI Patterns)
- Navigating from a selected week into its full detail (FEAT-19.SPEC-002)
- Empty, loading, error, and offline/degraded presentation of the list itself

**Non-Goals:**
- Showing the full plan or grocery list contents inline in the list -- that is FEAT-19.SPEC-002's (Past Week Detail View) responsibility; this screen shows only the at-a-glance summary, keeping the list scannable as history grows across several years of weekly plans (feature-overview.md, Non-Functional Notes)
- Reusing a past week directly from the list without opening it first -- excluded because the Brief's Internal Dependency Map routes reuse through the detail screen ("SPEC-002 -> [Maya taps 'Reuse this week'] -> SPEC-003"), keeping the safety-sensitive reuse action behind a deliberate open-and-review step rather than a one-tap list action
- Deleting or purging an individual past week from this list -- excluded per scope-boundaries.md SC-18: history is kept for the life of the household account with no per-week purge, and this retention is a documented trust commitment

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry (feature-overview.md) | Household navigates to Weekly Plan History from wherever the product surfaces the history entry point | None -- list loads the household's full archived history |
| FEAT-19.SPEC-002 (Past Week Detail View) | User taps back/close from a past week's detail | Returns to the list at the previously scrolled position |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- entire archived history list | Open any past week into its detail (FEAT-19.SPEC-002) | -- |
| Sam (Other Adult Member) | Full screen -- entire archived history list | Open any past week into its detail (FEAT-19.SPEC-002); no reuse action lives on this screen for any role (governed by FEAT-19.SPEC-004) | -- |
| Jordan (older kid, limited login -- Later) | Full screen -- entire archived history list | Open any past week into its detail (FEAT-19.SPEC-002) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; there is no screen for it to reach -- see Household Setup & Member Profiles (FEAT-01) for how this profile's data is managed on its behalf |
| Riley (Operator, support) | Full screen -- entire archived history list, per Weekly Plan View access (user-persona.md Access Matrix) | View only; no reuse action, consistent with Riley never being able to change household data (feature-dependency-map.md, XBR-14) | -- |
| Unauthenticated | No | No | Redirected to the sign-in screen; no history content is shown before or during redirect |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." appears; the list's current scroll position is not preserved across the redirect, and the household's history reloads fresh from the top after re-authentication |

## Layout and Content

**Header:** Screen title "Plan History" with a back arrow (returns to wherever the household entered from) and no other header actions -- there is no reuse or edit action at this level (governed by FEAT-19.SPEC-004).

**Body:** A single-column, vertically scrolling list of past-week list-items, ordered most recent (top) to oldest (bottom):
- Each list-item is the shared "past-week card/list-item" pattern (feature-overview.md, Shared UI Patterns) and shows: the week identifier (the calendar week the plan covered), the week's status ("Approved" or "Adopted as proposed," carried from the archived Weekly Plan's approval field), and an at-a-glance summary (the count of dinners the week held, e.g., "6 of 7 nights planned")
- No filter or search control -- the Brief names no such capability for this screen, and history is browsed strictly chronologically

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Single-column list as described above, full width; each list-item stacks its week identifier, status, and meal-count summary vertically within the card.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; list-items lay their week identifier, status, and meal-count summary out in a single row instead of stacking. No other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the entry point the household arrived from | Screen closes | Animated transition back |
| Past-week list-item | Tap | Navigate to FEAT-19.SPEC-002 (Past Week Detail View) for the selected week | Screen transitions to detail | Animated transition to detail screen, carrying the selected week's identifier |
| List (scroll) | Swipe/scroll | Loads further-back weeks as the household scrolls toward the bottom | List extends with additional list-items | Brief loading indicator appears at the bottom edge while additional weeks load |

### Accessibility Notes

- **Focus order:** Back arrow -> each past-week list-item in displayed (most-recent-first) order.
- **List announcements:** As additional weeks load while scrolling, the newly loaded items are appended without moving focus; no interruption is announced for a background load that succeeds.
- **Error announcements:** A failed load's error banner (see States) is announced to assistive technology when it appears.
- **Keyboard alternatives:** Every list-item is independently focusable and activatable via keyboard; there are no pointer-only gestures beyond standard scrolling, which has a keyboard equivalent (arrow/page keys) via the platform's standard list navigation.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | "No past weeks yet" message in place of the list, per the feature's shared empty-state language (feature-overview.md, Shared UI Patterns) | Household has zero archived Weekly Plan records (first week not yet completed) | Household's first week is archived, and the screen is reopened or refreshed |
| Loading | A brief inline loading indicator in place of the list, per the feature's couple-second loading budget (feature-overview.md, Non-Functional Notes) | Screen first opens, before the initial page of archived weeks returns | Initial page of archived weeks returns (successfully or with an error) |
| Populated | List of past-week list-items as described in Layout and Content | Initial load (or a later page load) succeeds with at least one archived week | User navigates away, or a further page load begins |
| Error | Error banner "Couldn't load your plan history. Try again." with a Retry button, in place of the list | Initial load or a further-page load fails | User taps Retry and the load succeeds |
| Offline/Degraded | Previously viewed weeks in this list remain visible; a banner "You're offline -- showing previously viewed history" appears at the top; scrolling to weeks not yet loaded in this session shows a "Reconnect to see more" notice instead of a spinner | Connectivity is lost while this screen is open, or the screen is opened while offline | Connectivity is restored -- the banner clears and further scrolling resumes normal loading |

## Validation Rules

N/A -- this screen has no user input fields; it is a read-only list with navigation only.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Wherever the household entered from | -- |
| Past-week list-item tap | FEAT-19.SPEC-002 (Past Week Detail View) | -- |

## Data Model

**Creates:** None.
**Reads:** Weekly Plan (archived records) -- week, status (approval field, rendered as "Approved" or "Adopted as proposed"), and the count of Planned Meals present, for every archived record belonging to the household, ordered most recent first. All fields per the Weekly Plan definition in feature-dependency-map.md.
**Updates:** None.
**Deletes:** None.

## Business Rules

- List visibility for every role is governed by FEAT-19.SPEC-004 (History Access & Reuse Authorization) -- this screen does not re-derive role access independently (feature-overview.md, Shared Validation).
- XBR-01 (feature-dependency-map.md): every meal shown anywhere in the product, including within an archived week's summary, traces to a safety check already performed at the time the week was live; this screen displays archived data as-is and performs no new safety evaluation itself.

## Edge Cases

- **Household has an in-progress (not yet archived) current week** -- The current week never appears in this list; only Archived Weekly Plan records are shown, consistent with the Weekly Plan lifecycle (feature-dependency-map.md: Generated/Started -> Reviewed -> Approved -> Active -> Archived).
- **A past week has zero planned nights (e.g., the household planned nothing that week)** -- The list-item still appears, showing "0 of 7 nights planned," rather than being omitted, so the household's history stays a complete, honest record.
- **User scrolls rapidly through several years of history** -- Further pages continue to load on demand as the user approaches the bottom of the currently loaded list; the household's ability to browse stays equally responsive as history grows into several years of weekly plans, per the product's general heavier-moment responsiveness expectation (feature-overview.md, Non-Functional Notes; assumptions-constraints.md ASMP-23).
- **User taps a list-item twice rapidly** -- The second tap is ignored while the first navigation to FEAT-19.SPEC-002 is already in progress.
- **No concurrent-edit conflict entry applies** -- This screen only reads archived Weekly Plan records, which are never updated in place once archived (feature-dependency-map.md: Weekly Plan is "Archived when the week ends" and this feature's Entity-Lifecycle Coverage Matrix states its Connected Entities are read-only); there is no live write path on this screen for another user's change to race against.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-002 (Past Week Detail View) | Navigation (outbound) | Selecting a past-week list-item opens its full detail |
| FEAT-19.SPEC-004 (History Access & Reuse Authorization) | References (inbound) | Governs which roles may see this list at all |
| FEAT-03.SPEC-* (AI Weekly Dinner Plan Generation) | References (inbound, cross-feature) | Source of the archived Weekly Plan records this screen lists |
| FEAT-06.SPEC-* (Shared Grocery List) | References (inbound, cross-feature) | Archived Grocery List records exist per listed week, opened via FEAT-19.SPEC-002 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_history_opened | household id, count of archived weeks available | Screen is opened and the initial page of archived weeks returns | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); this event is retained as an operational signal so the gap is visible rather than silently dropped, per Phase 2.5 Category 8 |
| plan_history_week_selected | selected week identifier, position in list (e.g., "3rd most recent") | User taps a past-week list-item | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); see plan_history_opened |

## Acceptance Criteria

**FEAT-19.SPEC-001-AC-01:** Given Maya has three archived weekly plans, when she opens Weekly Plan History, then she sees a list of three past-week list-items ordered most recent first, each showing its week identifier, status, and meal count.

**FEAT-19.SPEC-001-AC-02:** Given Maya taps a past-week list-item, when the tap registers, then the screen navigates to FEAT-19.SPEC-002 (Past Week Detail View) for that week.

**FEAT-19.SPEC-001-AC-03:** Given Sam opens Weekly Plan History, when the list loads, then he sees the same full list of archived weeks as Maya, with no reuse control anywhere on this screen.

**FEAT-19.SPEC-001-AC-04:** Given Jordan (older kid, limited login) opens Weekly Plan History, when the list loads, then he sees the same full list of archived weeks, with no reuse control anywhere on this screen.

**FEAT-19.SPEC-001-AC-05:** Given a household with zero archived weekly plans, when any authorized member opens Weekly Plan History, then the screen shows "No past weeks yet" instead of an empty list.

**FEAT-19.SPEC-001-AC-06:** Given Maya opens Weekly Plan History, when the initial page of archived weeks has not yet returned, then a brief inline loading indicator appears in place of the list.

**FEAT-19.SPEC-001-AC-07:** Given the initial load of archived weeks fails, when the error appears, then Maya sees "Couldn't load your plan history. Try again." with a Retry button, and tapping Retry re-attempts the load.

**FEAT-19.SPEC-001-AC-08:** Given Maya loses connectivity while Weekly Plan History is open, when connectivity drops, then previously viewed weeks remain visible with a "You're offline -- showing previously viewed history" banner, and scrolling to unloaded weeks shows a "Reconnect to see more" notice instead of a spinner.

**FEAT-19.SPEC-001-AC-09:** Given Maya scrolls toward the bottom of the currently loaded list, when more archived weeks exist further back, then the list extends with additional list-items after a brief loading indicator at the bottom edge.

**FEAT-19.SPEC-001-AC-10:** Given a past week in which the household planned zero nights, when Maya views the list, then that week's list-item shows "0 of 7 nights planned" rather than being omitted from the list.

**FEAT-19.SPEC-001-AC-11:** Given an unauthenticated visitor attempts to reach Weekly Plan History, when the screen would otherwise load, then they are redirected to the sign-in screen with no history content shown.

**FEAT-19.SPEC-001-AC-12:** Given Maya's session expires while Weekly Plan History is open, when she next interacts with the screen, then a dialog reads "Your session has expired. Sign in to continue." and, after re-authenticating, the list reloads fresh from the top.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (empty, loading, populated, error, offline) | 5 |
| Business Rules | 2 | 2 |
| Edge Cases | 5 | 5 |



# Screen Spec: Past Week Detail View

## Overview

**Name:** Past Week Detail View
**ID:** FEAT-19.SPEC-002
**Type:** Screen
**Purpose:** Household member views a single past week's full archived plan and grocery list, with the option for Maya to reuse the week as a starting point for a future week.
**Parent Feature:** FEAT-19 -- Weekly Plan History

## Scope and Non-Goals

**In Scope:**
- Displaying the full archived Weekly Plan for a selected week: every archived Planned Meal (dinner and leftover-lunch), each with its recipe name, dietary badges, cook time, and rough cost
- Displaying the full archived Grocery List for the same week, aisle-grouped, exactly as it stood when the week was archived
- Opening recipe detail for any archived meal
- Presenting the "Reuse this week" control to Maya only, and launching FEAT-19.SPEC-003 (Past Plan Reuse) when she picks a target future week

**Non-Goals:**
- Editing any field of the archived week's contents -- excluded because the Weekly Plan lifecycle treats a week as Archived once it ends (feature-dependency-map.md, Weekly Plan lifecycle) and this feature's Primary Flows only support copying a past week into a future week, never modifying the historical record itself (product-features.md, Primary Flows & Alternates)
- Ticking or unticking archived Grocery List Items -- excluded for the same reason: the archived Grocery List is a read-only historical record, not a live list; a fresh, tickable list is built only once a reused week is edited in Manual Weekly Planning (FEAT-23), per FEAT-06's ownership of Grocery List recalculation
- Performing the reuse copy itself -- handled entirely by FEAT-19.SPEC-003 (Past Plan Reuse); this screen only launches that automation and hands off once the target week is chosen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-19.SPEC-001 (Weekly Plan History Browse) | User taps a past-week list-item | The selected week's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail; tap "Reuse this week" and pick a target future week, launching FEAT-19.SPEC-003 | -- |
| Sam (Other Adult Member) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail only; the "Reuse this week" control is not shown to this role | -- |
| Jordan (older kid, limited login -- Later) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail only; the "Reuse this week" control is not shown to this role | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; there is no screen for it to reach |
| Riley (Operator, support) | Full screen -- archived plan, archived grocery list, and recipe detail, per Weekly Plan View access (user-persona.md Access Matrix) | View only; the "Reuse this week" control is not shown to this role, consistent with Riley never changing household data (feature-dependency-map.md, XBR-14) | -- |
| Unauthenticated | No | No | Redirected to the sign-in screen; no detail content is shown before or during redirect |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." appears; on re-authentication the user returns to FEAT-19.SPEC-001 (Weekly Plan History Browse) rather than directly back into this detail view |

## Layout and Content

**Header:** Screen title showing the week identifier (e.g., "Week of Mar 3") with a back arrow (returns to FEAT-19.SPEC-001) and, for Maya only, a "Reuse this week" action button (right-aligned).

**Body, top section -- Archived Plan:** A seven-night list, one row per night, each showing: the night's name, the recipe name (tap opens recipe detail), the "checked against allergies" safety badge and "always check labels" disclaimer carried from the archived record, cook time, and rough cost. A night with no meal picked that week shows a "nothing planned" marker consistent with the shared empty/loading/error state language (feature-overview.md, Shared UI Patterns). Any leftover-lunch Planned Meal linked to a dinner appears directly beneath its source dinner's row.

**Body, middle section -- Week Summary:** The week's estimated total cost against the household's budget at the time, carried from the archived Weekly Plan's estimated_total field.

**Body, bottom section -- Archived Grocery List:** The full aisle-grouped list as it stood when archived, read-only: each line shows ingredient name, combined quantity and unit, and aisle, with no tick controls.

**Footer:** None -- "Reuse this week" is in the header.

### Responsive Behavior

- **Compact size class:** Plan, week summary, and grocery list sections stack vertically in the single-column order described above, full width.
- **Medium size class and above:** Plan section and Grocery List section lay out side by side as two columns, with the Week Summary spanning the full width between the header and the two-column area; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-19.SPEC-001 (Weekly Plan History Browse) | Screen closes | Animated transition back to the list |
| Recipe name (any night) | Tap | Open recipe detail (FEAT-08.SPEC-002 Recipe Detail View) for the archived recipe | Screen transitions to recipe detail | Animated transition to recipe detail screen |
| "Reuse this week" button (Maya only) | Tap | Present a target-future-week picker | Picker overlay appears | Picker shows the available future weeks Maya may choose |
| Target-week picker option | Tap (select a future week) | Launch FEAT-19.SPEC-003 (Past Plan Reuse) with this past week as source and the selected week as target | Picker closes; reuse automation begins | Brief inline "Copying this week..." indicator while the automation runs |
| Target-week picker (Cancel) | Tap | Dismiss the picker without launching reuse | Picker closes | Detail screen remains unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> "Reuse this week" button (Maya only) -> each night row in order (Sunday through Saturday, with any linked leftover-lunch row immediately after its source) -> week summary -> each grocery list line in aisle order.
- **Dynamic announcements:** The "Copying this week..." indicator and its eventual success or failure outcome (handed off to FEAT-19.SPEC-003) are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every recipe name and the "Reuse this week" button are independently focusable and activatable via keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- this screen is only ever reached by selecting a real past-week list-item from FEAT-19.SPEC-001, so a source week with archived plan data always exists; there is no zero-data variant of this screen to render | N/A | N/A |
| Loading | Brief inline loading indicator in place of the plan and list sections, per the feature's couple-second loading budget (feature-overview.md, Non-Functional Notes) | Screen first opens, before the selected week's archived plan and list return | Load succeeds or fails |
| Populated | Full archived plan, week summary, and grocery list as described in Layout and Content | Load succeeds | User navigates away |
| Error | Error banner "Couldn't load this week. Try again." with a Retry button, in place of the plan and list sections | Load of the selected week's archived data fails | User taps Retry and the load succeeds |
| Offline/Degraded | If this week was previously viewed in this session, its cached plan and list remain visible with a "You're offline -- showing a previously viewed version" banner; "Reuse this week" is disabled with the inline note "Reusing a week needs a connection" while offline | Connectivity is lost while this screen is open, or the screen is opened offline for a week not previously cached this session | Connectivity is restored -- banner clears and "Reuse this week" re-enables for Maya |
| Recipe removed from household pool | The affected night's recipe name shows "Recipe no longer available" in place of a tappable link, with cook time and cost still shown from the archived record; the safety badge remains as archived | A recipe referenced in this archived week has since been removed from the household's recipe pool (FEAT-10) | N/A -- this is a permanent state for this archived record once the recipe is gone |

## Validation Rules

N/A -- this screen has no user input fields beyond the target-future-week picker selection, whose valid range (available future weeks) is governed by FEAT-19.SPEC-003 (Past Plan Reuse) and FEAT-23 (Manual Weekly Planning)'s planning-horizon rule (up to one week ahead, per product-features.md, Validation & Limits).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-19.SPEC-001 (Weekly Plan History Browse) | -- |
| Recipe name tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 (Recipe Library) |
| "Reuse this week" -> pick target week | FEAT-19.SPEC-003 (Past Plan Reuse), then on completion to Manual Weekly Planning's week view | FEAT-23 (Manual Weekly Planning) |

## Data Model

**Creates:** None on this screen directly -- the reuse action's copy is created by FEAT-19.SPEC-003 once launched from here.
**Reads:** Weekly Plan (archived) -- week, status, estimated_total, and every archived Planned Meal (night, recipe, safety_badge, cook_time, rough_cost, meal_kind, status). Grocery List (archived) -- aisle_grouping and every archived Grocery List Item (ingredient_name, quantity_and_unit, aisle). Recipe -- name, dietary_badges, and current pool-membership status (to detect a since-removed recipe), read within the plan display. All fields per the definitions in feature-dependency-map.md.
**Updates:** None -- this screen never writes to the archived record.
**Deletes:** None.

## Business Rules

- Reuse-control visibility (whether "Reuse this week" appears at all) is governed by FEAT-19.SPEC-004 (History Access & Reuse Authorization) -- this screen does not re-derive the rule independently (feature-overview.md, Shared Validation).
- XBR-01 (feature-dependency-map.md): the safety badge shown on each archived meal reflects the check performed when that meal was live; reusing the week re-runs the check under today's rules via FEAT-19.SPEC-003, not on this display screen.
- A recipe removed from the household's pool since this week was archived is shown per the "Recipe removed from household pool" state above, consistent with this feature's Entity-Lifecycle Coverage Matrix note on FEAT-10's ownership of recipe removal.

## Edge Cases

- **Concurrent-edit conflict** -- N/A -- this screen only reads the archived Weekly Plan and Grocery List, which are never updated in place once archived (feature-dependency-map.md: Weekly Plan is "Archived when the week ends"; this feature's Entity-Lifecycle Coverage Matrix states its Connected Entities are read-only); there is no live write path here for another user's change to race against.
- **Two household members tap "Reuse this week" for the same past week into different target future weeks at the same time** -- N/A for Sam and the older-kid login, since the control is not shown to them; for Maya, only one active session can hold the control, so no concurrent reuse launch from this screen is possible for the same account.
- **Maya taps "Reuse this week" then immediately taps Back before picking a target week** -- The picker is dismissed and no reuse automation launches; the detail screen closes normally.
- **A night's leftover-lunch link references a source dinner that was itself swapped multiple times before archiving** -- The archived record shows only the final swap_history-resolved recipe that was active at archive time, consistent with how the live plan displayed it.
- **User navigates directly to this screen's URL for a week identifier from an unauthorized role or another household** -- Treated as the Unauthorized Experience defined in Access and Visibility: an unauthenticated or wrong-household request is redirected to sign-in with no detail content shown; no week data is ever returned for a household the requester does not belong to.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-001 (Weekly Plan History Browse) | Navigation (inbound) | User arrives here from a selected list-item |
| FEAT-19.SPEC-003 (Past Plan Reuse) | Triggers (outbound) | "Reuse this week" launches the reuse automation with this week as source |
| FEAT-19.SPEC-004 (History Access & Reuse Authorization) | References (inbound) | Governs detail visibility and reuse-control visibility |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Recipe name tap opens recipe detail |
| FEAT-06.SPEC-* (Shared Grocery List) | References (inbound, cross-feature) | Source of the archived Grocery List data displayed here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_history_week_detail_viewed | week identifier, meal count, whether any recipe shows "no longer available" | Screen load succeeds | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); this event is retained as an operational signal so the gap is visible rather than silently dropped, per Phase 2.5 Category 8 |
| past_plan_reused | source week identifier, target week identifier | Maya picks a target future week and the reuse automation (FEAT-19.SPEC-003) is launched | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); see plan_history_week_detail_viewed |

## Acceptance Criteria

**FEAT-19.SPEC-002-AC-01:** Given Maya opens a past week's detail from the history list, when the screen loads, then she sees all seven nights with their recipes, safety badges, cook time, and cost, plus the archived grocery list grouped by aisle.

**FEAT-19.SPEC-002-AC-02:** Given Maya is viewing a past week's detail, when she taps a recipe name, then the screen navigates to that recipe's detail view.

**FEAT-19.SPEC-002-AC-03:** Given Maya is viewing a past week's detail, when she looks at the header, then she sees a "Reuse this week" button.

**FEAT-19.SPEC-002-AC-04:** Given Sam is viewing a past week's detail, when he looks at the header, then no "Reuse this week" button is shown.

**FEAT-19.SPEC-002-AC-05:** Given Jordan (older kid, limited login) is viewing a past week's detail, when he looks at the header, then no "Reuse this week" button is shown.

**FEAT-19.SPEC-002-AC-06:** Given Maya taps "Reuse this week" and picks a target future week, when the selection is made, then FEAT-19.SPEC-003 (Past Plan Reuse) launches with this week as the source and the picked week as the target.

**FEAT-19.SPEC-002-AC-07:** Given Maya taps "Reuse this week" and then taps Cancel in the target-week picker, when Cancel is tapped, then the picker closes and no reuse automation launches.

**FEAT-19.SPEC-002-AC-08:** Given the selected week's archived data has not yet returned, when the screen first opens, then a brief inline loading indicator appears in place of the plan and list sections.

**FEAT-19.SPEC-002-AC-09:** Given the selected week's archived data fails to load, when the error appears, then Maya sees "Couldn't load this week. Try again." with a Retry button.

**FEAT-19.SPEC-002-AC-10:** Given Maya loses connectivity while viewing a previously cached week's detail, when connectivity drops, then the cached plan and list remain visible with an offline banner, and "Reuse this week" is disabled with the note "Reusing a week needs a connection."

**FEAT-19.SPEC-002-AC-11:** Given an archived week references a recipe that has since been removed from the household's pool, when Maya views that night, then it shows "Recipe no longer available" in place of a tappable recipe name, while cook time, cost, and the archived safety badge still display.

**FEAT-19.SPEC-002-AC-12:** Given a night in the archived week had no meal picked that week, when Maya views the plan, then that night shows a "nothing planned" marker.

**FEAT-19.SPEC-002-AC-13:** Given Riley (Operator, support) views a past week's detail during an open support request, when the screen loads, then she sees the full archived plan and list read-only, with no "Reuse this week" control shown.

**FEAT-19.SPEC-002-AC-14:** Given Maya's session expires while viewing a past week's detail, when she next interacts with the screen, then a dialog reads "Your session has expired. Sign in to continue." and, after re-authenticating, she lands back on FEAT-19.SPEC-001 (Weekly Plan History Browse) rather than directly on this detail screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, loading, populated, error, offline, recipe-removed) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: History Access & Reuse Authorization

## Overview

**Name:** History Access & Reuse Authorization
**ID:** FEAT-19.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines which roles may view a household's archived Weekly Plan history and which single role may trigger reusing a past week, enforced consistently across both history screens and the reuse automation.
**Parent Feature:** FEAT-19 -- Weekly Plan History
**Governed Entity:** Weekly Plan (archived)

## Scope and Non-Goals

**In Scope:**
- Who may view the Weekly Plan History Browse list (FEAT-19.SPEC-001) and Past Week Detail View (FEAT-19.SPEC-002) for the household
- Who may see the "Reuse this week" control at all, and who may actually trigger the Past Plan Reuse automation (FEAT-19.SPEC-003)
- What every role and unauthenticated/expired state experiences when denied

**Non-Goals:**
- Authorization for editing the Weekly Plan a reuse copy lands in -- governed by Manual Weekly Planning's own authorization rules (FEAT-23), since once the copy is handed off it is an ordinary future week subject to FEAT-23's Access field (Maya Full, Sam Own-only)
- Field-level validation of Weekly Plan or Grocery List data -- this feature's Connected Entities are read-only (feature-overview.md, Entity-Lifecycle Coverage Matrix: "no full CRUD matrix applies; all entities this feature touches are listed... as Referenced Entities"); no field-level validation rules exist for this feature to govern, since it neither creates nor edits these entities
- The allergy and religious-rule safety re-check performed during reuse -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02) and referenced by FEAT-19.SPEC-003 as XBR-01; this spec governs only who may trigger reuse, not the safety logic reuse depends on

## Governed Entity

**Entity:** Weekly Plan (archived)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date range | The calendar week the archived plan covered -- no validation beyond data type; this spec governs read/reuse access, not the field's format |
| origin | enum | AI-generated or manually built -- no validation beyond data type |
| status | enum | Generated/Started, Reviewed, Approved, Active, Archived; this spec's rules apply only once status is Archived (feature-dependency-map.md, Weekly Plan lifecycle) |
| approval | text/enum | Organiser approval record or auto-adoption note -- no validation beyond data type; displayed read-only |
| estimated_total | number | The archived week's estimated cost -- no validation beyond data type; displayed read-only |
| over_budget_note | text | Shown when the archived week exceeded budget -- no validation beyond data type; displayed read-only |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-19.SPEC-001 | Weekly Plan History Browse | On screen entry -- list visibility gated per role before any archived week data is shown |
| FEAT-19.SPEC-002 | Past Week Detail View | On screen entry -- detail visibility gated per role; "Reuse this week" control visibility gated per role |
| FEAT-19.SPEC-003 | Past Plan Reuse | On trigger -- the automation only fires when the acting role is authorized to reuse |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type | Always | -- | -- | -- |
| origin | No validation beyond data type | Always | -- | -- | -- |
| status | No validation beyond data type | Always | -- | -- | -- |
| approval | No validation beyond data type | Always | -- | -- | -- |
| estimated_total | No validation beyond data type | Always | -- | -- | -- |
| over_budget_note | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs read and reuse-trigger access, not field-level cross-validation; the archived Weekly Plan's fields carry no conditional-requirement relationships this spec must arbitrate.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View history list (FEAT-19.SPEC-001) | Maya (Organiser) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Sam (Other Adult Member) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Jordan (older kid, limited login -- Later) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no reachable screen to deny access to |
| View history list (FEAT-19.SPEC-001) | Riley (Operator, support) | Always, per Weekly Plan View access (user-persona.md Access Matrix) | -- |
| View history list (FEAT-19.SPEC-001) | Unauthenticated / expired session | Never | Redirected to the sign-in screen; no history content shown before or during redirect |
| View past week detail (FEAT-19.SPEC-002) | Maya (Organiser) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Sam (Other Adult Member) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Jordan (older kid, limited login -- Later) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no reachable screen to deny access to |
| View past week detail (FEAT-19.SPEC-002) | Riley (Operator, support) | Always, per Weekly Plan View access (user-persona.md Access Matrix) | -- |
| View past week detail (FEAT-19.SPEC-002) | Unauthenticated / expired session | Never | Redirected to the sign-in screen; no detail content shown before or during redirect |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Maya (Organiser) | Always | -- |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Sam (Other Adult Member) | Never | Control is not rendered on the detail screen for this role -- no button appears |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Jordan (older kid, limited login -- Later) | Never | Control is not rendered on the detail screen for this role -- no button appears |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Riley (Operator, support) | Never | Control is not rendered on the detail screen for this role, consistent with Riley never changing household data (feature-dependency-map.md, XBR-14) |
| Trigger reuse copy (FEAT-19.SPEC-003) | Maya (Organiser) | Always -- the target future week must fall within the one-week-ahead planning horizon (product-features.md, Validation & Limits) | -- |
| Trigger reuse copy (FEAT-19.SPEC-003) | Sam, Jordan (either form), Riley | Never | No launching control is available to attempt the action from; a direct attempt to invoke the automation without the control (e.g., a stale or manipulated request) is refused with "Only the household organiser can reuse a past week" |

## Defaults and Derivations

N/A -- this spec's governed entity is read-only in this feature (Entity-Lifecycle Coverage Matrix, feature-overview.md); it defines no defaulted or derived field values of its own. The Past Plan Reuse automation's own created-week defaults (e.g., origin recorded as manually built, per the existing two-value enum) are specified in FEAT-19.SPEC-003, not here.

## Business Rules

- Access level for viewing history follows the Weekly Plan column of the Access Matrix in user-persona.md exactly: Full for Maya (including reuse), View for Sam and the older-kid login, View for Riley from v1 (feature-overview.md, Access field).
- Reuse is gated to Maya alone regardless of a household's tier or the past week's origin (AI-generated or manually built) -- the same single-role gate applies to every archived week.
- XBR-01 (feature-dependency-map.md): a reused week is still subject to the app-enforced safety check before it is shown as pre-filled; this spec's authorization gate on who may trigger reuse does not substitute for, or weaken, that check.
- An unauthorized visitor (not signed in as a household member) never reaches any Weekly Plan History screen; per user-persona.md's Access Matrix notes, unauthorized visitors see only a sign-in screen, an invitation-acceptance screen, or a household-referral welcome page -- never household data of any kind.
- This spec is the single authoritative source for history view and reuse-trigger access; FEAT-19.SPEC-001, FEAT-19.SPEC-002, and FEAT-19.SPEC-003 each reference it rather than re-deriving the rule (feature-overview.md, Shared Validation).

## Edge Cases

- **Maya's organiser role is handed over to Sam mid-session while Maya has the Past Week Detail View open with "Reuse this week" visible** -- The hand-over (FEAT-09) takes effect immediately; on Maya's next interaction with the control, her session is re-evaluated against the current organiser and the control is hidden for her (she is no longer Maya-the-organiser once the hand-over completes), while Sam's session gains the control on his next screen load.
- **Riley's support access opens against a household with no open Support Request** -- Per XBR-14, Riley's support access opens only for a household with an open Support Request; outside that window, Riley has no access to any Weekly Plan History screen for that household at all, not merely a read-only view.
- **The older-kid limited login (Later) is removed or its login access ends between viewing the list and opening a detail** -- The detail screen's own access check re-evaluates on load; if the login no longer exists, the request is treated as unauthenticated and redirected to sign-in.
- **A direct API-level request attempts to trigger Past Plan Reuse for a household where the requester is Sam** -- Refused with "Only the household organiser can reuse a past week," regardless of whether the request came through the rendered control (which Sam never sees) or an out-of-band request.
- **Two devices are signed in as Maya at once, one of which is mid-reuse-trigger when the household's organiser role is handed over on the other device** -- The reuse trigger already in flight completes under the authorization state captured when it started (per FEAT-19.SPEC-003's own concurrency handling); any new reuse attempt from either device is evaluated against the current organiser at the moment of that new attempt.

## Acceptance Criteria

**FEAT-19.SPEC-004-AC-01:** Given Maya opens Weekly Plan History, when the list loads, then she sees the full archived history.

**FEAT-19.SPEC-004-AC-02:** Given Sam opens Weekly Plan History, when the list loads, then he sees the full archived history, the same as Maya.

**FEAT-19.SPEC-004-AC-03:** Given Jordan (older kid, limited login) opens Weekly Plan History, when the list loads, then he sees the full archived history, the same as Maya.

**FEAT-19.SPEC-004-AC-04:** Given an unauthenticated visitor attempts to reach Weekly Plan History, when the request is made, then they are redirected to the sign-in screen with no history content shown.

**FEAT-19.SPEC-004-AC-05:** Given Riley (Operator, support) has an open Support Request for a household, when she opens that household's Weekly Plan History, then she sees the full archived history read-only.

**FEAT-19.SPEC-004-AC-06:** Given Maya opens a past week's detail, when the screen renders, then she sees the "Reuse this week" control in the header.

**FEAT-19.SPEC-004-AC-07:** Given Sam opens the same past week's detail, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-08:** Given Jordan (older kid, limited login) opens the same past week's detail, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-09:** Given Riley opens the same past week's detail during an open support request, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-10:** Given Maya taps "Reuse this week" and picks a valid target future week, when the trigger fires, then FEAT-19.SPEC-003 (Past Plan Reuse) begins processing.

**FEAT-19.SPEC-004-AC-11:** Given a direct request attempts to trigger the reuse automation as Sam, when the request is evaluated, then it is refused with "Only the household organiser can reuse a past week."

**FEAT-19.SPEC-004-AC-12:** Given Maya hands over the organiser role to Sam while she has the Past Week Detail View open, when she next interacts with the screen, then the "Reuse this week" control is no longer shown to her.

**FEAT-19.SPEC-004-AC-13:** Given Jordan as a young kid profile has no login, when any attempt is made to reach a Weekly Plan History screen under that profile, then no such attempt is possible -- there is no reachable screen for it to load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 17 | 17 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

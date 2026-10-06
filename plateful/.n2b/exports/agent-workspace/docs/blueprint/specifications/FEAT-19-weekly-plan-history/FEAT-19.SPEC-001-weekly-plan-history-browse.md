---
document_type: spec
spec_type: screen
spec_id: FEAT-19.SPEC-001
spec_name: Weekly Plan History Browse
spec_slug: weekly-plan-history-browse
parent_feature: FEAT-19
parent_feature_name: Weekly Plan History
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

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

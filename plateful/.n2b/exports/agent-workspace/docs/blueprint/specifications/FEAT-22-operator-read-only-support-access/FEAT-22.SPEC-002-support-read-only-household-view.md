---
document_type: spec
spec_type: screen
spec_id: FEAT-22.SPEC-002
spec_name: Support Read-Only Household View
spec_slug: support-read-only-household-view
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Support Read-Only Household View

## Overview

**Name:** Support Read-Only Household View
**ID:** FEAT-22.SPEC-002
**Type:** Screen
**Purpose:** Riley views one household's setup, plan, and related data in read-only form, with no edit controls anywhere, to diagnose the open Support Request.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Displaying the household's setup, weekly plan, grocery list, ratings, pantry items, recipes, and plan tier, all read-only, for the household behind the open Support Request
- Suppressing kid profile detail and billing detail per FEAT-22.SPEC-007
- Starting and ending the access session that FEAT-22.SPEC-004 logs
- The "Mark Resolved" action that hands off to FEAT-22.SPEC-005

**Non-Goals:**
- Any edit action on any household data displayed on this screen (household setup, member profiles, weekly plan, grocery list, ratings, pantry items, recipes, subscription tier) -- excluded per BRIEF.md's Target Users & Roles and FEAT-22.SPEC-006: this diagnostic content is read-only with zero edit affordances anywhere in the layout, described as an absence rather than a disabled state. This is distinct from "Mark Resolved," the one permitted lifecycle action, which changes only the Support Request's own status (governed by FEAT-22.SPEC-008), never any household data.
- Viewing a second household at the same time, or any household with no open request -- governed entirely by FEAT-22.SPEC-006
- The access-session log itself and its display to the organiser -- owned by FEAT-22.SPEC-004 (logging) and FEAT-22.SPEC-003 (the organiser's own view of the record)
- Payment details or billing history beyond plan tier, and any kid profile detail beyond the allergy fact tied to an open safety-concern request -- governed entirely by FEAT-22.SPEC-007

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-22.SPEC-001 (Support Request Queue) | Riley selects an open request row, and FEAT-22.SPEC-006's gating allows the open | The selected household and its open Support Request (kind, note, planned meal/recipe for safety concerns) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Riley (Operator, support) | Full screen, subject to FEAT-22.SPEC-007's suppressions | Mark the open request Resolved; close the view | -- |
| Maya (Organiser) | No | No | This screen does not exist within Maya's product surface; her own household screens are the live, editable versions of this data |
| Sam (Other Adult Member) | No | No | This screen does not exist within Sam's product surface |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role |
| Jordan (older kid, limited login -- Later) | No | No | This login's product surface has no path to this operator-only screen |
| Unauthenticated | No | No | Redirected to the operator sign-in screen |
| Expired session | No | No | The open access session is ended (FEAT-22.SPEC-004) and Riley is redirected to the operator sign-in screen; re-authenticating returns Riley to FEAT-22.SPEC-001, not directly back into this view, since re-entry must pass FEAT-22.SPEC-006's gating again |

## Layout and Content

**Header:** The household's name, the open request's kind and note (for safety concerns: the reported meal and recipe name), and a "Mark Resolved" action (right-aligned). No back arrow; a "Close" control (top-left) ends the session and returns to FEAT-22.SPEC-001.

**Body:** A single-column, section-by-section read-only display, in this order:

- **Household setup:** household name, weekly budget, weekly schedule, unit system, currency, aisle names, plan-arrival day and time.
- **Members:** each adult member's display_name and dietary rules in full; each kid member shown per FEAT-22.SPEC-007's suppression (no display_name, age_band, or broader profile -- only the specific allergy fact tied to an open safety-concern request naming that meal, attributed to "a household kid member," never a name).
- **Weekly plan:** the current week's plan, each Planned Meal's night, recipe, cook time, rough cost, and status; for a safety-concern request, the specific reported meal is highlighted.
- **Grocery list:** the household's current list, read-only.
- **Ratings:** the household's recorded meal ratings, read-only.
- **Pantry items:** the household's current pantry list, read-only.
- **Recipes:** the household's imported recipes, read-only, alongside starter-library recipes referenced by the plan.
- **Subscription:** plan tier only (free or paid) -- no billing period, billing state, billing history, or payment detail (FEAT-22.SPEC-007).

No edit controls, input fields, or action buttons appear anywhere within these household-data sections -- every element below the header is display-only. (The header's "Mark Resolved" action is the one exception in the layout, and it acts only on the Support Request's own status, never on any household data shown below it.)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order listed, each collapsible to a summary row that expands on tap; "Mark Resolved" remains in the header.
- **Medium size class and above:** Sections render side by side in two columns (household setup and members in one column; plan, list, and the rest in the other), with no structural change beyond column layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | End the access session (FEAT-22.SPEC-004) and navigate to FEAT-22.SPEC-001 | Screen closes | Standard transition back to the queue |
| Section header (compact only) | Tap | Expand or collapse the section | Section content shows or hides | Chevron icon rotates |
| "Mark Resolved" | Tap | Open the resolution confirmation: for a safety-concern request, present two outcome options ("Recipe is safe" / "Recipe is unsafe"); for a general support request, present a single confirmation | Dialog opens | Dialog title "Mark this request resolved?" |
| Resolution dialog confirm | Tap | Trigger FEAT-22.SPEC-005 (Support Request Resolution) with the request and (for safety concerns) the selected outcome | Button shows a brief loading state | Success: dialog closes, the header updates to show the request as Resolved, and "Mark Resolved" is replaced with a "Resolved" label. Failure: inline error in the dialog with Retry |
| Resolution dialog cancel | Tap | Dismiss the dialog without resolving | Dialog closes | Returns to the household view unchanged |

### Accessibility Notes

- **Focus order:** Close control -> "Mark Resolved" -> household setup section -> members -> weekly plan -> grocery list -> ratings -> pantry items -> recipes -> subscription.
- **Section state announcements:** Expanding or collapsing a section announces its new state ("Household setup, expanded" / "collapsed") to assistive technology.
- **Resolution feedback:** The resolved confirmation and any dialog error are announced when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A brief inline loading indicator in place of the sections | Screen first opens, before household data resolves | Data loads (populated or error) |
| Populated (default) | All sections shown with current household data | Data loads successfully | Riley closes the view or marks the request Resolved |
| Resolving | "Mark Resolved" dialog shows a loading state during submission | Riley confirms resolution | Resolution completes or fails |
| Resolved | Header shows the request as Resolved; "Mark Resolved" replaced with a static "Resolved" label; the rest of the view remains viewable read-only until Riley closes it | Resolution completes successfully | Riley taps Close |
| Error (dialog) | Inline error "Couldn't mark this request resolved. Check your connection and try again." with Retry | Resolution submission fails | Riley taps Retry (returns to Resolving) or Cancel |
| Offline/Degraded | Banner "You're offline -- this household's data needs a connection to load or refresh." Sections already loaded remain visible but cannot refresh; "Mark Resolved" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears, "Mark Resolved" re-enables, and data refreshes |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Access gating is governed by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules). Data visibility and suppression is governed by FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule). Status transition validity for "Mark Resolved" is governed by FEAT-22.SPEC-008 (Support Request Status Transition Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Close control tap | FEAT-22.SPEC-001 (Support Request Queue) | -- |
| Successful resolution, Close tapped afterward | FEAT-22.SPEC-001 (Support Request Queue) | -- |

## Data Model

**Creates:** None directly -- opening this screen triggers FEAT-22.SPEC-004 to create an access-session entry.
**Reads:** Household -- household_name, weekly_budget, weekly_schedule, unit_system, currency, aisle_names, plan_arrival_day_time. Member Profile -- display_name and Dietary Rules for adults; kid fields per FEAT-22.SPEC-007. Weekly Plan and Planned Meal -- current week's dinners and their fields. Grocery List and Grocery List Item -- current list. Rating -- household ratings. Pantry Item -- current pantry list. Recipe -- household's imported and referenced starter recipes. Subscription -- tier only. Support Request -- the open request's kind, note, planned_meal/recipe.
**Updates:** None directly by this screen -- FEAT-22.SPEC-004 writes the access_record on open/close, and FEAT-22.SPEC-005 writes status on resolution.
**Deletes:** None.

## Business Rules

- No household data on this screen is editable -- there is no edit affordance for any diagnostic content (household setup, members, plan, list, ratings, pantry, recipes, subscription) anywhere in the layout, per FEAT-22.SPEC-006. "Mark Resolved" is the one permitted lifecycle action in the layout, and it writes only the Support Request's own status, never any household data.
- Every field shown is subject to FEAT-22.SPEC-007's suppression rules; a field this spec does not explicitly list as suppressed is shown as-is.
- Opening this screen always starts an access session (FEAT-22.SPEC-004); closing it always ends that session.
- The first time a given Support Request's household is opened, FEAT-22.SPEC-008 transitions the request from Raised to Under review, via FEAT-22.SPEC-004.
- XBR-14: Operator support access is strictly read-only, never shows payment details or kid profile data beyond the allergy details in a specific safety report, and closes when the request is resolved.

## Edge Cases

- **The organiser edits the household's data while Riley is viewing it** -- No conflict exists: this screen never writes to any shared entity, so there is nothing for a concurrent household edit to collide with. The screen does not force-refresh mid-view; Riley sees the data as loaded until closing and reopening, since the feature's use is described as rare, on-demand, and not requiring live synchronization.
- **Riley taps "Mark Resolved" twice in rapid succession** -- The second tap is ignored while the first is processing (dialog in loading state).
- **The Support Request is resolved by a concurrent process (e.g., a rare race with another operator session, though FEAT-22.SPEC-006 permits only one) while this view is open** -- The header updates to show Resolved on the next load; Riley may still close the view normally, and no error is shown since resolution is itself the intended outcome.
- **Network failure while loading the household's data** -- Error banner: "Couldn't load this household's data. Check your connection and try again." with Retry; no partial edit state exists to lose, since the screen has never accepted input.
- **Riley closes the view mid-load, before data finishes resolving** -- The access session's end timestamp is recorded at the moment of closing regardless of load state; no partial data is left displayed since the screen unmounts entirely.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Support Request Queue) | Navigation (inbound) | Entry point when Riley selects a request |
| FEAT-22.SPEC-006 (Support Access Scope & Gating Rules) | References (inbound) | Governs whether this screen may open at all |
| FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule) | References (inbound) | Governs every field's visibility on this screen |
| FEAT-22.SPEC-004 (Support Access Session Logging) | Triggers (outbound) | Opening and closing this screen starts and ends the logged session |
| FEAT-22.SPEC-005 (Support Request Resolution) | Triggers (outbound) | "Mark Resolved" hands off to this automation |
| FEAT-22.SPEC-008 (Support Request Status Transition Rules) | References (outbound) | Governs the Raised -> Under review -> Resolved transitions this screen triggers or displays |
| FEAT-01.SPEC-010 (Household Settings Hub) | References (inbound) | This screen's household setup section reads the same household facts (name, budget, schedule) FEAT-01.SPEC-010 maintains, in read-only form |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | This screen's Members section reads the same member profile fields FEAT-01.SPEC-005 maintains, subject to FEAT-22.SPEC-007's kid-data suppression |
| FEAT-03.SPEC-001 (Weekly Plan View) | References (inbound) | This screen's Weekly plan section reads the same current-week Planned Meal data FEAT-03.SPEC-001 displays to the household, in read-only form |
| FEAT-06.SPEC-001 (Grocery List) | References (inbound) | This screen's Grocery list section reads the same current-week Grocery List FEAT-06.SPEC-001 displays to the household, in read-only form |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_household_view_opened | request kind (safety concern / general support) | Screen opens successfully | N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing operator surface's actual use is observable |
| support_household_view_closed | duration | Riley closes the view | N/A -- no success-metrics.md metric traces to this feature |
| support_request_resolved_from_view | kind, outcome (for safety concerns) | Resolution completes successfully | N/A -- no success-metrics.md metric traces to this feature |

## Acceptance Criteria

**FEAT-22.SPEC-002-AC-01:** Given Riley selects an open safety-concern request from the queue, when this screen opens, then the header shows the household name, the request's kind, note, and the reported meal, and the weekly plan section highlights that meal.

**FEAT-22.SPEC-002-AC-02:** Given Riley is viewing a household, when Riley looks anywhere in the household-data sections (setup, members, plan, list, ratings, pantry, recipes, subscription) for an edit control, then none exists in any section -- "Mark Resolved" in the header remains the one exception, acting only on the Support Request's own status.

**FEAT-22.SPEC-002-AC-03:** Given the household has a kid member with an allergy relevant to the open safety-concern request, when Riley views the Members section, then only the allergy fact tied to the reported meal is shown, attributed to "a household kid member," with no name, age, or other kid profile detail.

**FEAT-22.SPEC-002-AC-04:** Given the household is on the paid tier, when Riley views the Subscription section, then only "Paid" is shown, with no billing period, billing state, billing history, or payment detail.

**FEAT-22.SPEC-002-AC-05:** Given Riley opens this screen, then FEAT-22.SPEC-004 records the start of an access session for this household and Support Request.

**FEAT-22.SPEC-002-AC-06:** Given Riley taps Close, then the access session's end is recorded by FEAT-22.SPEC-004 and Riley returns to FEAT-22.SPEC-001.

**FEAT-22.SPEC-002-AC-07:** Given the open request is a safety concern, when Riley taps "Mark Resolved", then the dialog presents "Recipe is safe" and "Recipe is unsafe" as the two outcome options.

**FEAT-22.SPEC-002-AC-08:** Given the open request is a general support contact, when Riley taps "Mark Resolved", then the dialog presents a single confirmation with no outcome options.

**FEAT-22.SPEC-002-AC-09:** Given Riley confirms resolution, when it completes successfully, then the header shows the request as Resolved and "Mark Resolved" is replaced with a static "Resolved" label.

**FEAT-22.SPEC-002-AC-10:** Given resolution submission fails due to a network error, when Riley views the dialog, then the inline error with Retry appears and the request remains open.

**FEAT-22.SPEC-002-AC-11:** Given Riley loses connectivity while viewing an already-loaded household, then the offline banner appears, already-loaded sections remain visible, and "Mark Resolved" is disabled until connectivity returns.

**FEAT-22.SPEC-002-AC-12:** Given household data fails to load due to a network error, when Riley opens this screen, then the error banner with Retry appears in place of the sections.

**FEAT-22.SPEC-002-AC-13:** Given Riley closes the view before data finishes loading, then the access session's end is still recorded and Riley returns to FEAT-22.SPEC-001.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (loading, populated, resolving, resolved, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

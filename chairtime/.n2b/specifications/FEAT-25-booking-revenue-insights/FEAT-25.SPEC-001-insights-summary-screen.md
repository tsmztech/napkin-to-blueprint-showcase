---
document_type: spec
spec_type: screen
spec_id: FEAT-25.SPEC-001
spec_name: Insights Summary Screen
spec_slug: insights-summary-screen
parent_feature: FEAT-25
parent_feature_name: Booking & Revenue Insights
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 24
---

# Screen Spec: Insights Summary Screen

## Overview

**Name:** Insights Summary Screen
**ID:** FEAT-25.SPEC-001
**Type:** Screen
**Purpose:** A read-only period summary of the Pro's own booking volume, deposits collected, the no-show "saved" figure, most-booked services, and amounts received into her payout account, with a period selector and full Empty / Loading / Error / Offline-degraded coverage.
**Parent Feature:** FEAT-25 -- Booking & Revenue Insights

## Scope and Non-Goals

**In Scope:**
- Rendering the period summary: total bookings, deposits collected, the "saved" figure, the ranked most-booked-services list, and the amounts-received figure sourced from FEAT-28's money list (via FEAT-25.SPEC-002), for the selected period
- The period selector (week / month / year) and re-rendering the summary when the period changes
- The Empty, Loading, Error, and Offline-degraded states, including the "not enough data yet" alternate for a Pro with insufficient history
- Presenting the same figures identically to the owning Pro and to Platform Operator (Support) in read-only review

**Non-Goals:**
- Computing the summary figures themselves -- owned by FEAT-25.SPEC-002 (Period Insights Aggregation); this screen only renders what that automation returns
- Defining the period bounds, data-sufficiency threshold, and the "saved"/most-booked derivation formulas -- owned by FEAT-25.SPEC-003 (Insights Derivation, Period & Access Rules); this screen renders their outcomes
- Any input, editing, or annotation on the displayed figures -- excluded per this feature's own Validation & Limits field in product-features.md: "a read-only reporting view with no input validation"; every element on this screen is display-only apart from the period selector
- Exporting the summary or drilling into individual bookings from it -- excluded per the Brief's Non-Goals: no Key Capability, Primary Flow, or journey step names export or drill-down, and the Brief's Data Notes field scopes this feature to "aggregated figures" only
- Comparing this Pro's figures against any other pro's figures, or presenting a cross-pro benchmark -- excluded per SC-03 and the Brief's own Non-Goals; enforced by FEAT-25.SPEC-003

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro taps the Insights navigation affordance | None -- the screen loads with its default period |

<!-- This feature has no standalone entry point of its own (feature-overview.md, Default Entry): FEAT-12 (Pro Daily Schedule Dashboard) is the sole navigation source, per the dependency map's Navigation Connections row "FEAT-12 | navigation | FEAT-25 | insights (v1) | Pro opens insights." -->

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- her own figures only | Change the period selector; retry a failed computation | -- |
| Platform Operator (Support) | Full screen, identical figures to what Talia sees, for the one Pro account under active review | Change the period selector; retry a failed computation (view-only in every other respect -- no export, no annotation exists to withhold) | -- |
| The Client (Riley) | No | No | No navigation path reaches this screen from any Client-facing surface (Access Matrix: Activity Record & Insights = None for Riley); a direct attempt is redirected to the Pro sign-in screen (FEAT-29.SPEC-001) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29.SPEC-001); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29.SPEC-001) on the next data refresh; no unsaved input exists to preserve, since this is a read-only viewing surface with no form state |

Authorization governed by FEAT-25.SPEC-003 (Insights Derivation, Period & Access Rules), Authorization Rules section.

## Layout and Content

**Header:** Screen title "Insights" with a back arrow (returns to FEAT-12.SPEC-001) and, when viewed by Support, a persistent banner naming the Pro account under review (consistent with FEAT-19's read-only support-view pattern).

**Body:** Below the header, a period selector (three options: Week, Month, Year) shown as a segmented control, defaulting to Month. Below the selector, a single-column stack of summary elements, top to bottom:
1. **Total Bookings** -- a labeled figure showing the count of bookings for the selected period.
2. **Deposits Collected** -- a labeled figure showing the total amount collected from client deposits for the selected period, in the Pro's account currency (XBR-25).
3. **Saved from No-Shows** -- a labeled figure showing the amount that would have been lost to no-shows without automatic forfeiture, for the selected period, with a short supporting line ("kept automatically under your cancellation policy").
4. **Most-Booked Services** -- a ranked list (most bookings first) of Service names with their booking count for the selected period, capped at the top 5; a service tied in count with the 5th-ranked service is included, extending the list past 5 only for an exact tie at the cutoff.
5. **Amounts Received** -- a labeled figure showing the net amount actually received for the selected period, in the Pro's account currency, sourced from FEAT-28's money list composition (FEAT-28.SPEC-005) via FEAT-25.SPEC-002; renders whatever that composition returns for a Pro with no connected, Active Payout Account (per FEAT-28.SPEC-004), never a value this screen computes or fabricates on its own.

All five elements reflect the currently selected period and re-render together whenever the period selector changes. No element on this screen accepts text or numeric input.

### Responsive Behavior

- **Compact breakpoint (phone width, including inside the Instagram in-app browser):** Single-column stack as described above, full width; the period selector remains directly under the header at all times without requiring a scroll.
- **Medium size class and above:** The stack remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping, since the product's primary use is phone-first (ASMP: mobile-first for both roles).
- **Most-Booked Services list:** Uniform scaling, no structural change -- the ranked list always renders as a simple vertical list regardless of breakpoint.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Screen closes | Standard navigation transition |
| Period selector -- Week | Tap | Emits insights_period_changed and triggers FEAT-25.SPEC-002 (Period Insights Aggregation) for the week period | Selector shows Week active; summary elements enter Loading | Brief in-place loading indicator over the summary elements |
| Period selector -- Month | Tap | Emits insights_period_changed and triggers FEAT-25.SPEC-002 for the month period | Selector shows Month active; summary elements enter Loading | Brief in-place loading indicator over the summary elements |
| Period selector -- Year | Tap | Emits insights_period_changed and triggers FEAT-25.SPEC-002 for the year period | Selector shows Year active; summary elements enter Loading | Brief in-place loading indicator over the summary elements |
| Retry control (Error state only) | Tap | Re-triggers FEAT-25.SPEC-002 for the currently selected period | Summary elements enter Loading | Brief in-place loading indicator over the summary elements |
| Total Bookings, Deposits Collected, Saved from No-Shows, Most-Booked Services list, Amounts Received | -- | Display-only -- no interaction | None | None |

### Accessibility Notes

- **Focus order:** Back arrow -> period selector (Week -> Month -> Year) -> Total Bookings -> Deposits Collected -> Saved from No-Shows -> Most-Booked Services list -> Amounts Received, in that order.
- **Dynamic-change announcements:** When a period change or retry completes, the updated figures are announced to assistive technology as a group (not field by field), so the Pro or Support hears the new period's result as one update. Entry into the Error, Empty, and Offline-degraded states is announced when it occurs.
- **Keyboard alternatives:** Every action on this screen (period selection, retry) is reachable without a pointer-only gesture.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (sufficient data) | All five summary elements populated as described in Layout and Content, for the selected period | FEAT-25.SPEC-002 returns a computed summary and the Pro's history passes the data-sufficiency threshold (FEAT-25.SPEC-003) | Period selector changed, or data refreshes |
| Not enough data yet | An encouraging message in place of the summary elements: "Check back after a few weeks of bookings -- there's not enough history yet for a meaningful summary." The period selector remains visible but figures are not shown. | The Pro's history does not pass the data-sufficiency threshold (FEAT-25.SPEC-003); this is the product-features.md Empty state | The Pro's history passes the threshold on a later view |
| Loading | A brief in-place indicator over the summary area; the period selector remains interactive | Screen first opens, or the period selector changes, or Retry is tapped | FEAT-25.SPEC-002 returns a result (success or failure) |
| Error | The most recently successfully computed summary is shown, read-only, with a banner: "Couldn't refresh your insights. Showing your last computed summary." and a Retry control | FEAT-25.SPEC-002 reports a computation failure | Retry succeeds, producing a fresh summary |
| Offline/Degraded | Banner "You're offline -- showing your most recently viewed summary." at the top; the most recently viewed summary remains visible read-only; the period selector remains tappable but a period change while offline shows the same banner rather than a fresh computation | Connectivity is lost while this screen is open, or the screen is opened while already offline with a cached summary available | Connectivity is restored -- the banner clears and a fresh computation runs automatically for the currently selected period |

## Validation Rules

N/A -- this screen has no user-entry form fields; the period selector is a fixed three-option choice with no invalid state, per the Brief's own Validation & Limits field ("a read-only reporting view with no input validation").

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | FEAT-12 |

## Data Model

**Creates:** None.
**Reads:** The Period Insights Summary (a derived, read-only composition produced by FEAT-25.SPEC-002 from Booking, Deposit Transaction, and Service records, per FEAT-25.SPEC-003's formulas, plus FEAT-28.SPEC-005's net-amount-received-per-period derivation) -- total bookings count, deposits-collected amount, saved amount, the ranked most-booked-services list, and the amounts-received figure, all scoped to the selected period.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The period selector's three options (Week, Month, Year) and their bounds are defined once by FEAT-25.SPEC-003 -- this screen does not define period math of its own.
- The "not enough data yet" state's threshold is defined once by FEAT-25.SPEC-003 -- this screen only renders the resulting Loaded or Not-enough-data state.
- Own-figures-only and never-compared-across-pros (SC-03) is enforced by FEAT-25.SPEC-003's Authorization Rules; this screen never renders a cross-pro comparison element because no such data is ever returned to it.
- The Most-Booked Services ranking and the "saved" figure use the shared derivation formulas defined once in FEAT-25.SPEC-003 -- this screen never recomputes them.
- The Amounts Received figure is sourced from FEAT-28's money list composition (FEAT-28.SPEC-005) by way of FEAT-25.SPEC-002 -- this screen never computes it, and neither FEAT-25.SPEC-002 nor FEAT-25.SPEC-003 restates FEAT-28.SPEC-005's underlying formula, consistent with this feature's Referenced Entities table.

## Edge Cases

- **Pro changes the period selector twice in rapid succession** -- Only the most recently requested period's result is rendered; an in-flight computation for a superseded period is discarded on arrival rather than overwriting the newer selection.
- **Pro navigates away while a computation is in flight** -- The in-flight computation is discarded; no partial result is rendered later when the Pro returns, and a fresh computation runs on return.
- **Most-Booked Services list has fewer than 5 services with any bookings in the period** -- The list shows only the services that have at least one booking in the period; no placeholder rows for services with zero bookings.
- **A tie at the 5th-ranked service** -- All services tied with the 5th-ranked count are shown, per the Layout and Content cap rule, rather than an arbitrary cut.
- **Support reviews a Pro account with no history at all** -- Support sees the same "Not enough data yet" state Talia would see; no additional Support-only messaging exists, since Support's view is identical to the Pro's (Access and Visibility).
- **No concurrent-edit conflict applies to this screen** -- This screen never writes to Booking, Deposit Transaction, or Service, so the dependency map's Contention notes for those entities (all keyed to writers) do not produce a conflict state here; a figure can change between two views only because new bookings/deposits occurred, which simply produces a different computed result on the next load, not a save conflict.
- **Talia has no connected Payout Account, or hers is not yet Active** -- The Amounts Received figure shows exactly whatever FEAT-25.SPEC-002 returns for that state (per FEAT-28.SPEC-005's own composition rules for a disconnected or ineligible account), never a value this screen substitutes on its own; the other four summary elements are unaffected, since they never depend on Payout Account.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-002 (Period Insights Aggregation) | Triggers (outbound) | Screen open and period-change events trigger the aggregation; the screen renders its success/failure result |
| FEAT-25.SPEC-003 (Insights Derivation, Period & Access Rules) | References (inbound) | Period bounds, data-sufficiency threshold, derivation formulas, and access authorization all governed here |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Sole entry point into this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| insights_viewed | selected period (week/month/year), data-sufficiency outcome (sufficient/not-enough-data) | Screen finishes loading (Loaded or Not-enough-data state reached) | N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; this event exists as product usage instrumentation for a Nice-to-Have feature whose own value is proven through the figures it surfaces about other features, not a success metric of its own |
| insights_period_changed | previous period, new period | Pro or Support selects a different period on the selector | N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; recorded for the same reason as insights_viewed |

## Acceptance Criteria

**FEAT-25.SPEC-001-AC-01:** Given Talia is on the Insights Summary Screen with sufficient history, when the screen finishes loading, then Total Bookings, Deposits Collected, Saved from No-Shows, the Most-Booked Services list, and Amounts Received all render for the default Month period.

**FEAT-25.SPEC-001-AC-02:** Given Talia is on the Insights Summary Screen, when she taps the Week option on the period selector, then insights_period_changed is emitted, the summary elements show a brief loading indicator, and the figures re-render for the week period.

**FEAT-25.SPEC-001-AC-03:** Given Talia is on the Insights Summary Screen, when she taps the Year option, then the figures re-render for the year period.

**FEAT-25.SPEC-001-AC-04:** Given Talia's account has not yet passed the data-sufficiency threshold, when the screen loads, then the "Not enough data yet" state renders instead of the summary elements, with the message "Check back after a few weeks of bookings -- there's not enough history yet for a meaningful summary."

**FEAT-25.SPEC-001-AC-05:** Given Talia is on the Insights Summary Screen, when a computation is in progress, then a brief in-place loading indicator appears over the summary area and the period selector remains tappable.

**FEAT-25.SPEC-001-AC-06:** Given Talia has a previously computed summary and a fresh computation fails, when the failure occurs, then the last successfully computed summary remains visible read-only with the banner "Couldn't refresh your insights. Showing your last computed summary." and a Retry control.

**FEAT-25.SPEC-001-AC-07:** Given Talia is viewing the Error state, when she taps Retry, then a fresh computation runs for the currently selected period and, on success, the summary re-renders without the error banner.

**FEAT-25.SPEC-001-AC-08:** Given Talia loses connectivity while viewing the screen, when connectivity drops, then the banner "You're offline -- showing your most recently viewed summary." appears and the last-viewed summary remains visible read-only.

**FEAT-25.SPEC-001-AC-09:** Given Talia is offline and taps a different period option, when the tap registers, then the same offline banner is shown rather than a fresh computation being attempted.

**FEAT-25.SPEC-001-AC-10:** Given Talia's connectivity is restored while the Offline/Degraded state is showing, when connectivity returns, then the banner clears and a fresh computation runs automatically for the currently selected period.

**FEAT-25.SPEC-001-AC-11:** Given Support opens the Insights Summary Screen for a Pro account under active review, when the screen loads, then Support sees exactly the same figures Talia would see for that Pro, with a banner naming the Pro account under review.

**FEAT-25.SPEC-001-AC-12:** Given Support is on the Insights Summary Screen, when Support looks for any export, annotation, or edit control, then none exists -- Support's screen offers only the period selector and Retry, identical in kind to Talia's.

**FEAT-25.SPEC-001-AC-13:** Given Riley (the Client) attempts to reach this screen by any means, when the attempt is made, then no navigation path exists to it from any Client-facing surface.

**FEAT-25.SPEC-001-AC-14:** Given an unauthenticated visitor attempts to reach this screen directly, when the attempt is made, then they are redirected to the Pro sign-in screen (FEAT-29.SPEC-001) without any indication of whether a Pro account exists.

**FEAT-25.SPEC-001-AC-15:** Given Talia's session expires while this screen is open, when the next data refresh is attempted, then she is redirected to the Pro sign-in screen (FEAT-29.SPEC-001) with no unsaved input lost, since the screen holds no form state.

**FEAT-25.SPEC-001-AC-16:** Given Talia taps the Week option twice in rapid succession before the first request returns, when both requests are in flight, then only the result for the most recently tapped period is rendered.

**FEAT-25.SPEC-001-AC-17:** Given Talia navigates away from the screen while a period-change computation is in flight, when she returns to the screen later, then a fresh computation runs rather than showing a stale in-flight result.

**FEAT-25.SPEC-001-AC-18:** Given the Most-Booked Services ranking for the selected period has only 3 services with bookings, when the list renders, then exactly 3 rows are shown with no placeholder rows for unbooked services.

**FEAT-25.SPEC-001-AC-19:** Given the 5th and 6th ranked services are tied in booking count for the selected period, when the Most-Booked Services list renders, then both tied services are shown, extending the list past 5 rows.

**FEAT-25.SPEC-001-AC-20:** Given Talia is on the screen, when she taps her back arrow, then she is returned to FEAT-12.SPEC-001 (Today's & Upcoming Schedule).

**FEAT-25.SPEC-001-AC-21:** Given Talia is on the screen with the Month period selected, when she looks for any comparison against another pro's figures, then no such element exists anywhere on the screen, per SC-03.

**FEAT-25.SPEC-001-AC-22:** Given Talia is on the screen, when she attempts to tap or edit any of the Total Bookings, Deposits Collected, Saved from No-Shows, or Amounts Received figures, then no interaction occurs -- these elements are display-only.

**FEAT-25.SPEC-001-AC-23:** Given Talia has no connected Payout Account, when the Amounts Received figure renders, then it shows exactly what FEAT-25.SPEC-002 returns for that state, and no other summary element is affected.

**FEAT-25.SPEC-001-AC-24:** Given Support reviews a Pro account under active review, when the screen loads, then Amounts Received renders identically to what the Pro would see, consistent with the Access and Visibility table's identical-figures rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loaded, not-enough-data, loading, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

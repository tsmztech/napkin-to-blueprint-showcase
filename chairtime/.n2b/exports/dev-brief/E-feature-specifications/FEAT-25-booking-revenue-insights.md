# FEAT-25 — Booking & Revenue Insights

This chapter covers Booking & Revenue Insights (FEAT-25), a Nice-to-Have-tier feature. It carries 4 specifications carrying 87 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-25.SPEC-001 | Insights Summary Screen | screen | 24 |
| FEAT-25.SPEC-002 | Period Insights Aggregation | automation | 19 |
| FEAT-25.SPEC-003 | Insights Derivation, Period & Access Rules | logic-rule | 20 |
| FEAT-25.SPEC-004 | Historical Aggregate Maintenance | automation | 24 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Booking & Revenue Insights

## Summary

**Feature:** Booking & Revenue Insights
**ID:** FEAT-25
**Description:** A simple, functional summary for the Pro of their own booking volume, deposits collected, and no-shows recovered over time -- proof of the value the product is delivering, not a full analytics suite.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Success Criteria centers on the Pro noticing outcomes ("I haven't had an unpaid no-show since I switched"); a simple summary makes that outcome visible rather than only felt anecdotally, which supports the brief's referral-driven go-to-market ("most new pros arrive because another pro told them about it"). Not required for the core loop, so phased to v1. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- See total bookings and deposits collected over a selected period
- See how much would have been lost to no-shows without automatic forfeiture (a "saved" figure)
- See which services are booked most often

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-25.SPEC-001 | Insights Summary Screen | Screen | The Pro, Platform Operator (Support) | Read-only period summary view -- bookings, deposits collected, the no-show "saved" figure, and most-booked services, with a period selector and Empty / Loading / Error / Offline-degraded coverage |
| FEAT-25.SPEC-002 | Period Insights Aggregation | Automation | The Pro, Platform Operator (Support) | Computes the requested period's summary figures on view or period change, and retains the most recently successful result for reuse when a fresh computation fails or the Pro is offline |
| FEAT-25.SPEC-003 | Insights Derivation, Period & Access Rules | Logic/Rule | The Pro, Platform Operator (Support) | Defines the period bounds and data-sufficiency threshold, the "saved" and most-booked-service derivation formulas, and the own-figures-only / no-cross-pro-comparison access rule |
| FEAT-25.SPEC-004 | Historical Aggregate Maintenance | Automation | The Pro, Platform Operator (Support) | Maintains rolling per-period aggregates as bookings, deposit outcomes, no-show marks, and payout figures occur, so the summary stays responsive as a Pro's history grows across years |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| See total bookings and deposits collected over a selected period | FEAT-25.SPEC-001, FEAT-25.SPEC-002 | The screen renders the period summary; the automation computes bookings and deposits-collected for the selected period from Booking and Deposit Transaction records | Phase 2 (Explicit) |
| See how much would have been lost to no-shows without automatic forfeiture (a "saved" figure) | FEAT-25.SPEC-001, FEAT-25.SPEC-002, FEAT-25.SPEC-003 | The screen displays the figure; the automation computes it from forfeited-deposit outcomes; the derivation formula itself is defined once in the Logic/Rule spec | Phase 2 (Explicit) |
| See which services are booked most often | FEAT-25.SPEC-001, FEAT-25.SPEC-002 | The screen renders the ranked list; the automation tallies bookings per Service for the selected period | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-25.SPEC-003 | Insights Derivation, Period & Access Rules | Phase 5 (Rule Discovery) | Three interacting rule sets share this feature and none is a trivial inline check: the period-bounds / data-sufficiency threshold that decides the "not enough data yet" alternate (Primary Flows & Alternates), the "saved" figure and most-booked-service derivation formulas (Data Notes: "Derived: entirely"), and the own-figures-only / never-compared-across-pros authorization rule (Access field, SC-03) -- past the inline-validation threshold and shared by both the screen and the aggregation automation |
| FEAT-25.SPEC-004 | Historical Aggregate Maintenance | Phase 4 (Trigger-Response, time-based triggers lens) | ASMP-22 requires the product to "stay equally responsive as pros accumulate history over multiple years" at 20-40 bookings/week; recomputing bookings, deposits, no-show outcomes, and payout figures from full multi-year history on every view would not hold that bar, so a periodic/incremental aggregate-maintenance automation is required alongside the on-demand aggregation |

## Entity-Lifecycle Coverage Matrix

{This feature manages no entity of its own. Its own Data Notes field states this directly: "Derived: entirely -- every figure here is computed from existing Booking, Deposit Transaction, and Service records; nothing is captured directly." Its Connected Entities are all marked (read) in product-features.md. Accordingly, no Create/Read(single)/Update/Delete/State-Transition matrix applies to any entity this feature owns; every entity it touches is listed below as Referenced.}

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Source of booking counts and per-service tallies for the selected period; never created, updated, or deleted by this feature |
| Deposit Transaction | FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Source of deposits-collected totals and of forfeited-outcome records used to compute the "saved" figure; never written by this feature |
| Service | FEAT-25.SPEC-002 | Source of service names for the most-booked-services ranking; never written by this feature |
| Payout Account (via FEAT-28's money list) | FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Source of "amounts received" figures; explicitly not a Connected Entity of this feature in product-features.md -- read only through FEAT-28, per this feature's own Interactions field and the dependency map's context note |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro or Support opens the Insights Summary Screen | Compute (or reuse a cached) period summary and render it | Standalone Automation | FEAT-25.SPEC-002 |
| Pro changes the selected period (insights_period_changed) | Recompute the summary for the newly selected period, bounded to week/month/year per this feature's Validation & Limits field | Standalone Automation | FEAT-25.SPEC-002 |
| A fresh computation fails, or the view is opened offline | Show the most recently successfully computed summary, read-only, with a retry option when online | Standalone Automation (result), rendered by the Screen | FEAT-25.SPEC-002, FEAT-25.SPEC-001 |
| The Pro's history is too shallow for a meaningful summary | Show a plain "not enough data yet" state instead of a single-point chart | Standalone Logic/Rule (threshold), rendered by the Screen | FEAT-25.SPEC-003, FEAT-25.SPEC-001 |
| A booking is completed, a deposit outcome is set, a no-show is marked, or a payout figure is reported (FEAT-07, FEAT-11, FEAT-28) | Update the relevant rolling per-period aggregates | Standalone Automation | FEAT-25.SPEC-004 |
| Screen or aggregation applies the "saved" figure or most-booked-service ranking | Apply the shared derivation formula rather than restating it | Standalone Logic/Rule | FEAT-25.SPEC-003 |
| Any actor other than the owning Pro or read-only Support attempts to view a Pro's figures | Refused -- figures are never shown to another pro, a client, or compared across pros | Standalone Logic/Rule | FEAT-25.SPEC-003 |
| Pro opens insights from the daily schedule dashboard | Navigate into the Insights Summary Screen | Cross-feature -- entry point owned by FEAT-12 | FEAT-12 responsibility |

## Shared Context

**Shared Entities:**
- Booking, Deposit Transaction, Service, Payout Account (via FEAT-28) -- all read-only across this feature; SPEC-002 and SPEC-004 are the only specs that read them, and neither ever writes to them. No entity is created, updated, or deleted by this feature (Data Notes: "Derived: entirely").

**Shared UI Patterns:**
- Single period-summary surface -- SPEC-001 is the one screen for both the Pro's own use and Support's read-only use, matching the Access Matrix ("View" for both); it does not present a second screen for Support, only the same figures with no comparison or editing affordance. Spec Writers should describe both audiences of this one screen consistently, including how the Empty, Loading, Error, and Offline-degraded states each render (States field).

**Shared Validation:**
- SPEC-003 defines the period bounds, data-sufficiency threshold, "saved"/most-booked derivation formulas, and the own-figures-only access rule once. SPEC-001 (rendering the not-enough-data alternate and the period selector's bounds) and SPEC-002 (computing the figures and applying the access rule) both reference it rather than restating the rules.

## Internal Dependency Map

```
SPEC-001 (Insights Summary Screen) -> [Pro or Support opens insights, or changes the period] -> SPEC-002 (Period Insights Aggregation)
SPEC-002 (Period Insights Aggregation) -> [reads pre-maintained rolling aggregates from] -> SPEC-004 (Historical Aggregate Maintenance)
SPEC-002 (Period Insights Aggregation) -> [applies bounds, thresholds, and formulas from] -> SPEC-003 (Insights Derivation, Period & Access Rules)
SPEC-002 (Period Insights Aggregation) -> [computation succeeds] -> SPEC-001 (renders the period summary)
SPEC-002 (Period Insights Aggregation) -> [computation fails, or offline] -> SPEC-001 (renders the last successfully computed summary, read-only, with retry)
SPEC-004 (Historical Aggregate Maintenance) -> [booking completed / deposit outcome set / no-show marked / payout figure reported, from FEAT-07, FEAT-11, FEAT-28] -> updates rolling aggregates read by SPEC-002
SPEC-001 (Insights Summary Screen) -> [governed by] -> SPEC-003 (Insights Derivation, Period & Access Rules)
```

**Default Entry:** SPEC-001 (Insights Summary Screen) -- reached only via navigation from FEAT-12 (Pro Daily Schedule Dashboard, "Pro opens insights"); this feature has no standalone entry point of its own, consistent with the dependency map's Navigation connections row.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-25.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Sole navigation entry point into the insights view | Pro opens insights |
| FEAT-25.SPEC-002 | Inbound | FEAT-07 (Deposit Payment at Booking) | Reads deposit outcomes and amounts to compute deposits-collected for the selected period | Aggregation computed on view or period change |
| FEAT-25.SPEC-002 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Reads forfeiture outcomes to compute the "saved" figure | Aggregation computed on view or period change |
| FEAT-25.SPEC-002 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Reads the money list for "amounts received" figures | Aggregation computed on view or period change |
| FEAT-25.SPEC-004 | Inbound | FEAT-07 (Deposit Payment at Booking) | Deposit outcome events update rolling per-period aggregates | Deposit outcome determined |
| FEAT-25.SPEC-004 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | No-show mark and forfeiture events update rolling per-period aggregates | Booking marked no-show |
| FEAT-25.SPEC-004 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Reported payout figures update rolling per-period aggregates | Payout figure reported |

## Non-Functional Notes

**Data volumes / growth:** A Pro accumulates 20-40 bookings a week across a few hundred clients, with history kept for the life of the account across multiple years (ASMP-22, SC-22); this feature aggregates over that full, growing history, which is why SPEC-004 maintains rolling aggregates rather than relying on full recomputation from raw records at view time.

**Responsiveness:** The States field specifies only "a brief indicator while aggregating" for Loading, implying the summary should otherwise render promptly; ASMP-22's "equally responsive as pros accumulate history over multiple years" is the standard SPEC-004's rolling aggregates are built to meet, so that responsiveness does not degrade as a Pro's history deepens.

**Data sensitivity / privacy:** The view shows only the Pro's own aggregated figures, computed from Booking and Deposit Transaction records that are personal data linked to identifiable clients at the source (dependency map, Data Sensitivity lines); this feature never displays individual client identities, only period totals and per-service tallies, and those totals are visible solely to the owning Pro (View) and Platform Operator Support (View-only), never to any other pro or to any client (Access field, SC-03, ASMP-23).

**Compliance flags:** SC-11 applies indirectly: the figures this feature aggregates never include card data, since Deposit Transaction records already hold only amounts and outcomes, never card details, which the payment processor alone owns.

## Non-Goals

- **A full analytics or business-intelligence suite (custom reports, drill-down, scheduled exports)** -- Excluded per this feature's own Description, which states directly it is "a simple, functional summary ... not a full analytics suite"; no Key Capability names reporting beyond the three period figures and the most-booked-services list.
- **Comparing a Pro's figures against any other pro, or any cross-pro benchmark or leaderboard** -- Excluded per SC-03: BRIEF.md's Constraints state a client's and, by the same principle, a pro's data is "never visible to any other pro or client"; the Access field for this feature is explicit that figures are "never compared against or visible to any other pro," enforced by FEAT-25.SPEC-003.
- **Any input, editing, or annotation on the insights view** -- Excluded per this feature's own Validation & Limits field: "N/A -- a read-only reporting view with no input validation"; the screen never accepts writes of any kind.
- **Exporting the period summary to a spreadsheet or file, or a per-booking drill-down from the summary** -- Not named by any Key Capability, Primary Flow, or journey step (context package section 2 confirms no journey step owns this feature); the feature's Data Notes scope it to "aggregated figures" only, consistent with the Description's "not a full analytics suite" boundary.
- **A dedicated notification or alert tied to insights figures (e.g., a periodic digest message)** -- Excluded per this feature's own Communications field: "N/A -- a self-initiated view with no notification trigger"; the Pro must open the view to see figures, and nothing is pushed to her.



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



# Automation Spec: Period Insights Aggregation

## Overview

**Name:** Period Insights Aggregation
**ID:** FEAT-25.SPEC-002
**Type:** Automation
**Purpose:** Computes the requested period's summary figures (total bookings, deposits collected, the "saved" figure, most-booked services, and amounts received) on view or period change, and retains the most recently successful result for reuse when a fresh computation fails or the Pro is offline.
**Parent Feature:** FEAT-25 -- Booking & Revenue Insights

## Scope and Non-Goals

**In Scope:**
- Computing the four Booking/Deposit-Transaction-derived summary figures for a requested period, by reading FEAT-25.SPEC-004's maintained rolling aggregates rather than the Pro's full raw history
- Reading FEAT-28.SPEC-005's net-amount-received-per-period derivation for the same resolved period bounds, to produce the fifth summary figure -- Amounts Received -- per this feature's own Referenced Entities table ("Payout Account (via FEAT-28's money list)") and Cross-Feature Touchpoints row naming FEAT-28 as the source for "amounts received" figures
- Applying the period bounds, data-sufficiency threshold, and derivation formulas from FEAT-25.SPEC-003
- Retaining the most recently successful result so the screen can fall back to it on a failed recomputation or while offline
- Enforcing the own-figures-only access rule (FEAT-25.SPEC-003) before returning any result

**Non-Goals:**
- Maintaining the underlying rolling per-period aggregates as bookings, deposit outcomes, no-show marks, and payout figures occur -- owned by FEAT-25.SPEC-004; this automation only reads what that one maintains
- Defining the period bounds, data-sufficiency threshold, or the "saved"/most-booked derivation formulas themselves -- owned by FEAT-25.SPEC-003; this automation applies them, it does not define them
- Computing the net-amount-received-per-period figure itself -- owned by FEAT-28.SPEC-005; this automation reads that spec's already-composed net figure for the same period bounds and this Pro's Payout Account, it never recomputes the underlying deposit/refund/processor-fee composition
- Rendering the computed result -- owned by FEAT-25.SPEC-001 (Insights Summary Screen); this automation returns data, not layout
- Recomputing from the Pro's full multi-year raw booking history on every view -- excluded per ASMP-22 (SC-22): the product must "stay equally responsive as pros accumulate history over multiple years," which full recomputation at this volume would not hold; FEAT-25.SPEC-004's rolling aggregates exist specifically to avoid this cost

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Insights screen opened | FEAT-25.SPEC-001 (Insights Summary Screen) | Always, on screen load, for the screen's default period (Month) | Requesting Pro's account reference, default period |
| Period selector changed | FEAT-25.SPEC-001 (Insights Summary Screen) | Fires whenever the Pro or Support selects a different period option (Week, Month, Year) | Requesting Pro's account reference, newly selected period |
| Retry tapped in Error state | FEAT-25.SPEC-001 (Insights Summary Screen) | Fires when the Pro or Support taps Retry after a computation failure | Requesting Pro's account reference, currently selected period |

## Processing Logic

1. Receive the requesting Pro's account reference and the requested period (Week, Month, or Year) from the triggering screen.
2. Verify the requesting actor is authorized to view this Pro's figures, per FEAT-25.SPEC-003's Authorization Rules (the owning Pro, or Platform Operator Support reviewing that Pro's account). If not authorized, halt and return no data (this path is not reachable in practice, since FEAT-25.SPEC-001 is never rendered for an unauthorized actor).
3. Resolve the period's exact start and end bounds in the Pro's account timezone, per FEAT-25.SPEC-003's period-bounds rule.
4. Check whether the Pro's lifetime history passes the data-sufficiency threshold, per FEAT-25.SPEC-003. If it does not, return the "not enough data yet" outcome and stop -- no figures are computed.
5. Read the rolling per-period aggregates maintained by FEAT-25.SPEC-004 that fall within the resolved period bounds.
6. Sum the read aggregates' booking counts into the Total Bookings figure for the period.
7. Sum the read aggregates' deposits-collected amounts into the Deposits Collected figure for the period, using FEAT-25.SPEC-003's deposits-collected derivation.
8. Sum the read aggregates' forfeited-no-show amounts into the Saved from No-Shows figure for the period, using FEAT-25.SPEC-003's "saved" derivation formula.
9. Combine the read aggregates' per-service booking tallies across the period into a single ranked Most-Booked Services list, per FEAT-25.SPEC-003's ranking rule (ties included at the cutoff).
10. Read FEAT-28.SPEC-005's net-amount-received-per-period derivation for this Pro's Payout Account, for the same resolved period bounds, to obtain the Amounts Received figure.
11. Assemble the five figures (Total Bookings, Deposits Collected, Saved from No-Shows, Most-Booked Services, Amounts Received) into the Period Insights Summary result and mark it as the most recently successful result for this Pro and this period.
12. Return the assembled result to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Summary computed | Data-sufficiency threshold passed and both the aggregate read and the FEAT-28.SPEC-005 net-amount-received read succeed | None to source data -- this automation only reads; the most-recently-successful result is retained for reuse | Screen renders the five figures in the Loaded state | FEAT-25.SPEC-001 |
| Not enough data yet | Data-sufficiency threshold not passed | None | Screen renders the "not enough data yet" state | FEAT-25.SPEC-001 |
| Computation failure, prior result available | The FEAT-25.SPEC-004 aggregate read or the FEAT-28.SPEC-005 net-amount-received read fails or is unreachable, and a previously successful result exists for this Pro | None -- the retained prior result is unaffected | Screen renders the Error state: the last successful summary, read-only, with a Retry control | FEAT-25.SPEC-001 |
| Computation failure, no prior result available | The FEAT-25.SPEC-004 aggregate read or the FEAT-28.SPEC-005 net-amount-received read fails on the Pro's very first-ever computation attempt | None | Screen renders the Error state with no prior figures to fall back to -- the same retry banner appears over an empty summary area | FEAT-25.SPEC-001 |
| Offline at trigger time | The triggering screen detects no connectivity before invoking this automation | None -- this automation is not invoked | Screen renders the Offline/Degraded state using its own locally retained last-viewed summary, without calling this automation | FEAT-25.SPEC-001 |

## Data Model

**Reads:** FEAT-25.SPEC-004's rolling per-period aggregates (booking counts, deposits-collected amounts, forfeited-no-show amounts, and per-service booking tallies, bucketed by day) for the resolved period bounds; and FEAT-28.SPEC-005's net-amount-received-per-period derivation, for the same resolved period bounds and this Pro's Payout Account. Booking, Deposit Transaction, Service, and Payout Account are never read directly by this automation -- FEAT-25.SPEC-004 is the sole reader of Booking/Deposit Transaction raw records, and FEAT-28.SPEC-005 is the sole composer of the Payout Account-derived figure, which this automation relies on to keep the read volume bounded regardless of the Pro's total history depth.
**Creates:** None.
**Updates:** None -- this automation is read-only; it marks its own output as "most recently successful" for reuse by FEAT-25.SPEC-001's Error and Offline/Degraded states, but writes no product entity.
**Deletes:** None.

## Business Rules

- Period bounds, the data-sufficiency threshold, and the derivation formulas for the "saved" and most-booked-service figures are defined once by FEAT-25.SPEC-003 and applied here without restatement.
- This automation is read-only and non-blocking to every other feature: a failure here never affects Booking, Deposit Transaction, Service, or Payout Account data, and never blocks any action in FEAT-07, FEAT-11, or FEAT-28.
- The own-figures-only access rule (FEAT-25.SPEC-003) is checked before any computation begins -- no partial result is ever assembled for an unauthorized request.
- The most-recently-successful result is retained per Pro and per period independently -- a successful Month computation does not serve as a fallback for a failed Week computation.
- The Amounts Received figure is read directly from FEAT-28.SPEC-005's net-amount-received-per-period derivation for the same resolved period bounds; this automation performs no separate computation of its own over Deposit Transaction, Balance Payment, or Payout Account data for that figure, consistent with FEAT-25.SPEC-004's own scope (which explicitly does not duplicate payout figures into this feature's aggregates).

## Edge Cases

- **Aggregate maintenance (FEAT-25.SPEC-004) has not yet processed a very recent booking or deposit outcome** -- The summary reflects the aggregates as maintained at read time; a brief lag between an event occurring and its aggregate update is expected and does not constitute a computation failure.
- **The Pro's history exactly meets the data-sufficiency threshold** -- The threshold is treated as inclusive: meeting it exactly produces the Summary computed outcome, not "not enough data yet."
- **A period with zero bookings within an otherwise sufficient history (e.g., a slow Week within a busy year)** -- The Summary computed outcome still applies (the threshold is evaluated against lifetime history, not the selected period); figures render as zero/empty for that period, not as "not enough data yet."
- **Requested period spans a timezone-affecting change (e.g., the Pro's account timezone was changed between two aggregate-bucketed days)** -- The period bounds are resolved using the Pro's current account timezone at request time (XBR-25); historical aggregate buckets are read as originally recorded, consistent with FEAT-25.SPEC-003's period-bounds rule.
- **Concurrent trigger firing (Pro taps Week then immediately taps Year before the first computation returns)** -- Each request runs independently against its own period bounds; the triggering screen (FEAT-25.SPEC-001) discards the result of any request superseded by a later one, so this automation itself does not need to cancel in-flight requests.
- **Trigger fires while a previous run for the same Pro and period is still in flight** -- A duplicate request for the identical Pro and period (e.g., rapid double-tap of Retry) is treated as redundant: the second request's result, once it returns, simply confirms or refreshes the same figures already being awaited; no data corruption or double-counting results, since this automation performs no writes.
- **The Pro has no connected, Active Payout Account for part or all of the selected period (FEAT-28.SPEC-004's eligibility constraints)** -- The Amounts Received figure reflects exactly what FEAT-28.SPEC-005 composes for that state (zero, partial, or however that spec defines it for a disconnected or ineligible account); this automation applies no additional logic of its own and never fabricates a value beyond what FEAT-28.SPEC-005 returns.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Insights Summary Screen) | Triggered by (inbound) | Screen open, period change, and retry all trigger this automation |
| FEAT-25.SPEC-001 (Insights Summary Screen) | Affects (outbound) | Returns the computed summary or failure/fallback result for rendering |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) | References (inbound) | Reads the rolling per-period aggregates this automation maintains |
| FEAT-25.SPEC-003 (Insights Derivation, Period & Access Rules) | References (inbound) | Period bounds, data-sufficiency threshold, derivation formulas, and access authorization all governed here |
| FEAT-28.SPEC-005 (Money List Composition & Net Calculation) | References (inbound) | Reads the net-amount-received-per-period derivation for the resolved period bounds to produce the Amounts Received figure |

## Analytics and Success Signals

- **insights_computation_completed** (period: week/month/year, outcome: sufficient/not-enough-data, aggregate-days-read count) -- N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; recorded for product usage instrumentation on a Nice-to-Have feature whose value is proven through the figures it surfaces about other features
- **insights_computation_failed** (period: week/month/year, had-prior-result: yes/no) -- N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; recorded for the same reason as insights_computation_completed, so failure frequency for this non-blocking, read-only automation can still be observed operationally even though no product success metric depends on it

## Acceptance Criteria

**FEAT-25.SPEC-002-AC-01:** Given Talia opens the Insights Summary Screen for the first time in a session, when the screen loads, then this automation computes the summary for the default Month period.

**FEAT-25.SPEC-002-AC-02:** Given Talia selects the Week period, when the trigger fires, then this automation resolves the week's bounds in her account timezone and reads the corresponding rolling aggregates.

**FEAT-25.SPEC-002-AC-03:** Given Talia's lifetime history has not yet passed the data-sufficiency threshold, when a computation is requested, then the "not enough data yet" outcome is returned and no figures are assembled.

**FEAT-25.SPEC-002-AC-04:** Given Talia's lifetime history exactly meets the data-sufficiency threshold, when a computation is requested, then the Summary computed outcome is returned, not "not enough data yet."

**FEAT-25.SPEC-002-AC-05:** Given Talia's history is sufficient but the selected Week has zero bookings, when the computation completes, then the Summary computed outcome returns with all figures at zero/empty for that week.

**FEAT-25.SPEC-002-AC-06:** Given a fresh computation fails and a previously successful result exists for Talia's Month period, when the failure occurs, then the Computation failure outcome returns the prior result reference for the screen's Error state.

**FEAT-25.SPEC-002-AC-07:** Given a fresh computation fails on Talia's very first-ever computation attempt, when the failure occurs, then the Computation failure outcome returns with no prior result available.

**FEAT-25.SPEC-002-AC-08:** Given Talia is offline when she opens the Insights Summary Screen, when the screen detects no connectivity, then this automation is never invoked and the screen renders its own locally retained last-viewed summary.

**FEAT-25.SPEC-002-AC-09:** Given Support requests a computation for a Pro account under active review, when the request is processed, then the own-figures-only authorization check passes for Support's read-only review access and the same figures Talia would see are returned.

**FEAT-25.SPEC-002-AC-10:** Given a computation succeeds for Talia's Week period, when it completes, then the result is retained as the most recently successful Week result, independent of any Month or Year result retained for her.

**FEAT-25.SPEC-002-AC-11:** Given FEAT-25.SPEC-004's aggregate for a booking completed moments ago has not yet updated, when a computation reads the aggregates, then the summary reflects the aggregates as currently maintained, without this being treated as a failure.

**FEAT-25.SPEC-002-AC-12:** Given Talia's account timezone changed since some of her history was recorded, when a period is resolved, then the bounds use her current account timezone at request time.

**FEAT-25.SPEC-002-AC-13:** Given Talia taps Week and then immediately taps Year before the Week computation returns, when both requests run, then each completes independently against its own period bounds.

**FEAT-25.SPEC-002-AC-14:** Given Talia double-taps Retry in rapid succession, when both requests are processed, then neither produces a duplicated or corrupted result, since this automation performs no writes.

**FEAT-25.SPEC-002-AC-15:** Given a computation succeeds, when the summary is assembled, then Total Bookings, Deposits Collected, and Saved from No-Shows are each derived from the rolling aggregates using FEAT-25.SPEC-003's formulas, not from a fresh scan of raw Booking or Deposit Transaction records.

**FEAT-25.SPEC-002-AC-16:** Given the Most-Booked Services tally has a tie at the ranking cutoff for the requested period, when the ranked list is assembled, then all tied services are included per FEAT-25.SPEC-003's ranking rule.

**FEAT-25.SPEC-002-AC-17:** Given a computation succeeds, when the summary is assembled, then the Amounts Received figure is read from FEAT-28.SPEC-005's net-amount-received-per-period derivation for the same resolved period bounds, not computed independently by this automation.

**FEAT-25.SPEC-002-AC-18:** Given Talia has no connected, Active Payout Account for the selected period, when Amounts Received is read, then this automation applies no logic of its own beyond exactly what FEAT-28.SPEC-005 composes for that account state.

**FEAT-25.SPEC-002-AC-19:** Given FEAT-28.SPEC-005's net-amount-received read fails while the FEAT-25.SPEC-004 aggregate read succeeds, when the computation is evaluated, then the overall computation is treated as a Computation failure, falling back to the prior result exactly as an aggregate-read failure would.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (screen open, period change, retry) | 3 |
| Outcome Paths | 5 (computed, not-enough-data, failure-with-prior, failure-no-prior, offline-at-trigger) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Insights Derivation, Period & Access Rules

## Overview

**Name:** Insights Derivation, Period & Access Rules
**ID:** FEAT-25.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the period bounds and data-sufficiency threshold, the "saved" and most-booked-service derivation formulas, and the own-figures-only / no-cross-pro-comparison access rule, once, for both FEAT-25.SPEC-001 and FEAT-25.SPEC-002 to reference.
**Parent Feature:** FEAT-25 -- Booking & Revenue Insights
**Governed Entity:** Period Insights Summary -- a derived, read-only composition over Booking, Deposit Transaction, and Service records (via FEAT-25.SPEC-004's maintained aggregates), not a stored entity of its own

## Scope and Non-Goals

**In Scope:**
- The exact bounds for each of the three selectable periods (Week, Month, Year)
- The data-sufficiency threshold that decides the "not enough data yet" alternate
- The derivation formula for the "saved from no-shows" figure
- The derivation formula for the deposits-collected figure
- The ranking rule for the most-booked-services list
- Authorization rules for who may view a Pro's figures, including the never-compared-across-pros rule (SC-03)

**Non-Goals:**
- Reading the underlying rolling aggregates or assembling the final summary object -- owned by FEAT-25.SPEC-002; this spec defines the formulas it applies, not the read/compose sequence
- Maintaining the rolling per-period aggregates as source events occur -- owned by FEAT-25.SPEC-004; this spec's formulas operate on whatever those aggregates contain
- Rendering the period selector or any summary element -- owned by FEAT-25.SPEC-001; this spec defines what the numbers mean, not how they display
- Any rule governing Booking, Deposit Transaction, or Service themselves (their own field validation, lifecycle, or authorization) -- owned by their respective owning features (FEAT-01, FEAT-05, FEAT-07, FEAT-09, FEAT-11, FEAT-30); this spec only derives read-only figures from their already-valid data

## Governed Entity

**Entity:** Period Insights Summary -- a derived, read-only view computed at request time from Booking, Deposit Transaction, and Service records (via FEAT-25.SPEC-004's maintained aggregates). No independent stored field list; each field below is a computed output.

**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| period | enum (Week \| Month \| Year) | The selected period type |
| period_start / period_end | derived date | The resolved bounds of the selected period, in the Pro's account timezone |
| total_bookings | derived number | Count of Bookings falling within the period, per this spec's counting rule |
| deposits_collected | derived number | Total amount collected from client deposits within the period, in the Pro's account currency |
| saved_amount | derived number | Total amount that would have been lost to no-shows without automatic forfeiture, within the period |
| most_booked_services | derived ranked list | Service names ranked by booking count within the period, ties included at the cutoff |
| data_sufficient | derived boolean | Whether the Pro's lifetime history passes the data-sufficiency threshold |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-002 | Period Insights Aggregation | Applies every formula and the data-sufficiency check on every computation request |
| FEAT-25.SPEC-001 | Insights Summary Screen | Renders period bounds via the selector and the data-sufficiency outcome; applies no computation of its own |

## Field Validation Rules

Every field on this derived entity is computed from already-valid source data (Booking, Deposit Transaction, and Service records, each validated by their own owning feature); no field here carries entry-time validation of its own, since nothing on this entity is ever entered directly.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| period | Must be one of Week, Month, Year | Always | On computation request | N/A -- the period selector (FEAT-25.SPEC-001) only ever offers these three values; no invalid value can be submitted | No |
| period_start / period_end | No validation beyond data type -- always derived, never entered | Always | -- | -- | -- |
| total_bookings | No validation beyond data type -- always derived | Always | -- | -- | -- |
| deposits_collected | No validation beyond data type -- always derived | Always | -- | -- | -- |
| saved_amount | No validation beyond data type -- always derived | Always | -- | -- | -- |
| most_booked_services | No validation beyond data type -- always derived | Always | -- | -- | -- |
| data_sufficient | No validation beyond data type -- always derived | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Period bounds resolution | period, period_start, period_end | Week = the 7 days ending on and including the current date, in the Pro's account timezone (XBR-25). Month = the current calendar month to date, from its first day through the current date. Year = the current calendar year to date, from January 1 through the current date. All three are rolling "to date" windows, not fixed prior periods, so the summary always reflects the Pro's most recent activity. | N/A -- display composition, not a user-facing error |
| Total bookings counting rule | total_bookings, period_start, period_end | Counts every Booking whose start_time falls within the period bounds and whose state is Completed, No-Show, Confirmed, or Awaiting Outcome. A Booking in state Cancelled by Client, Cancelled by Pro, Rescheduled, or Expired (unpaid) is excluded -- Rescheduled is a terminal, non-counted state (never a real appointment in its own right), and the other three never became a real appointment at all. Two distinct reschedule mechanisms both resolve to "counted once": an in-place (outside-window) reschedule keeps the same Booking record and simply moves which start_time it is counted at, so it is counted once, at its current start_time, never at any superseded prior time; a compound (inside-window, late) reschedule instead excludes the original Booking entirely once it reaches terminal Rescheduled, and counts only the newly created Booking (FEAT-10.SPEC-004), on its own separate Booking-confirmed transition once its own fresh deposit is captured -- the two records are never both counted, and the new record's count is never attributed back to the original's period position. In both mechanisms a single appointment is never double-counted across a reschedule. | N/A |
| Deposits-collected derivation | deposits_collected, period_start, period_end | Sum, over every Deposit Transaction whose relevant timestamp (capture, application, or forfeiture) falls within the period bounds and whose status is Captured, Applied, Refund in Progress, or Forfeited, of (Deposit Transaction.amount minus Deposit Transaction.processor_fee) -- the same net-amount composition FEAT-28.SPEC-005 uses for its deposit rows, applied here per period rather than restated. A Deposit Transaction still Authorized, or one that was fully Refunded with no amount retained, contributes nothing to this figure. | N/A |
| "Saved from no-shows" derivation | saved_amount, period_start, period_end | Sum, over every Deposit Transaction whose status is Forfeited and whose outcome_reason records a no-show mark (FEAT-11) with a forfeiture timestamp falling within the period bounds, of (Deposit Transaction.amount minus Deposit Transaction.processor_fee). A deposit forfeited for a reason other than a no-show mark (there is none in v1 -- forfeiture is exclusively the no-show/inside-window outcome per XBR-09) is not distinguished separately, since the product defines no other forfeiture path; every Forfeited deposit within the period counts toward this figure. | N/A |
| Most-booked-services ranking | most_booked_services, period_start, period_end | Tally the count of Bookings counted under the total-bookings rule above, grouped by Service, within the period bounds. Rank Services by count, descending. Include the top 5 ranks; where the 5th and 6th (or further) ranks are exactly tied in count, include every tied Service rather than truncating arbitrarily. A Service with zero bookings in the period is omitted entirely, never shown as a zero row. | N/A |
| Data-sufficiency check | data_sufficient, total lifetime bookings | data_sufficient is true when the Pro's lifetime count of Bookings in state Completed or No-Show (the only states that represent a booking that actually occurred) is greater than or equal to platform parameter: `insights-minimum-history-threshold`; false otherwise. This check is evaluated against the Pro's entire history, independent of the currently selected period, so a Pro who passes the threshold overall still sees a Loaded (if empty) result for a quiet Week rather than reverting to "not enough data yet." | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View own Period Insights Summary | The Pro (Talia) | Own figures only, always | -- |
| View a Pro's Period Insights Summary | Platform Operator (Support) | Only for the one Pro account under active review; identical figures to what that Pro sees, never a cross-Pro aggregate or comparison | -- |
| View a Pro's Period Insights Summary | The Client (Riley) | Never | No navigation path to FEAT-25.SPEC-001 exists for any Client; the Access Matrix lists Activity Record & Insights = None for Riley |
| View another Pro's Period Insights Summary | The Pro (Talia) | Never | No mechanism exists in the product to select or view any account other than the signed-in Pro's own; the request path itself carries only the signed-in Pro's account reference |
| Compare or benchmark one Pro's figures against another's | Any role | Never, for any role | No such view, control, or figure exists anywhere in the product, per SC-03 and the Brief's own Non-Goals |
| Change the selected period | The Pro (Talia), Platform Operator (Support) | Always, read-only action -- changes only which period's figures are shown, writes nothing | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| period | Month | On screen load, before any period change | Yes -- changeable via the period selector on FEAT-25.SPEC-001 |
| period_start / period_end | See the period bounds resolution cross-field rule above | Recomputed on every computation request | No -- always derived from the selected period and the Pro's current account timezone |
| total_bookings, deposits_collected, saved_amount, most_booked_services | See the respective cross-field rules above | Recomputed on every computation request | No -- always derived, never entered or edited |
| data_sufficient | See the data-sufficiency cross-field rule above | Recomputed on every computation request, against lifetime history | No |

## Business Rules

- SC-03: a Pro's figures are never visible to, or compared against, any other pro or client; this spec's Authorization Rules are the sole gate that makes this true, and no other spec in this feature defines an alternate access path.
- ASMP-22 / SC-22: the formulas above are defined to operate over FEAT-25.SPEC-004's maintained aggregates rather than full raw history, so responsiveness does not degrade as a Pro's history deepens across years.
- XBR-25: every period bound and every displayed amount uses the Pro's account timezone and account currency respectively; this spec never hard-codes either.
- XBR-09: forfeiture is a binary outcome (kept in full, or refunded in full); the "saved" derivation therefore never needs a partial-percentage case, consistent with the product's binary cancellation/no-show rule.
- The data-sufficiency threshold (platform parameter: `insights-minimum-history-threshold`) is evaluated once, against lifetime history, and applies uniformly to every period selection -- it is never re-evaluated per period.

## Edge Cases

- **A Booking's start_time falls exactly on the period boundary (e.g., exactly at period_start)** -- Included: period bounds are inclusive of both period_start and period_end.
- **A Deposit Transaction's forfeiture timestamp falls in a different period than its original capture timestamp** -- Deposits-collected counts it in the period its capture timestamp falls in; the "saved" figure counts it in the period its forfeiture timestamp falls in. The two figures are not required to reconcile to each other within a single period, since they measure different events.
- **A Rescheduled booking's final start_time falls outside the originally requested period while its original start_time fell inside it (in-place, outside-window reschedule)** -- Only the final start_time is counted, per the rescheduled-booking rule; the booking does not appear in a period it was moved out of, even though it once had a start_time there.
- **A Booking undergoes a compound (inside-window, late) reschedule -- the original transitions to terminal Rescheduled and a new Booking is created for the new time (FEAT-10.SPEC-004)** -- The original Booking is excluded from total_bookings for every period from the moment it reaches terminal Rescheduled, in any period whose bounds its original start_time falls in. The new Booking is counted only once it independently reaches Confirmed on its own fresh deposit capture, at its own start_time. If the new Booking's deposit is never captured (it stays unpaid or expires), no Booking from this reschedule is counted at all for that appointment -- the original's exclusion is not conditional on the new one's success.
- **The Pro's lifetime Completed/No-Show count is exactly at platform parameter: `insights-minimum-history-threshold`** -- data_sufficient evaluates to true (the threshold is inclusive, "greater than or equal to").
- **Two Services are tied for the 5th Most-Booked rank, and a third is tied with them** -- All three tied Services are included; the list extends to 7 rows in this case rather than being capped at 5 or 6.
- **A Pro changes account timezone (FEAT-27) between two dates that fall in the same selected period under the old timezone but different periods under the new one** -- Period bounds are resolved using the account's current timezone at the moment of the computation request (per the period bounds resolution rule); a Booking's own recorded start_time is unaffected by a later timezone change.
- **Support reviews a Pro account and the data-sufficiency threshold is not met** -- Support receives the same data_sufficient = false outcome as the Pro would, per the identical-figures authorization rule; no Support-specific override of the threshold exists.

## Acceptance Criteria

**FEAT-25.SPEC-003-AC-01:** Given Talia's account timezone is set, when the Month period is resolved, then period_start is the first day of the current calendar month at 00:00 in her account timezone and period_end is the current date.

**FEAT-25.SPEC-003-AC-02:** Given Talia selects the Week period, when it is resolved, then the bounds cover the 7 days ending on and including the current date, in her account timezone.

**FEAT-25.SPEC-003-AC-03:** Given a Booking is Cancelled by Client within the selected period, when total_bookings is computed, then that Booking is excluded from the count.

**FEAT-25.SPEC-003-AC-04:** Given a Booking was Rescheduled from a time inside the selected period to a time outside it, when total_bookings is computed, then the Booking is not counted in the originally requested period.

**FEAT-25.SPEC-003-AC-05:** Given a Deposit Transaction is Captured within the selected period with a processor_fee, when deposits_collected is computed, then the amount minus the processor_fee is included in the total.

**FEAT-25.SPEC-003-AC-06:** Given a Deposit Transaction is still Authorized (not yet Captured) within the selected period, when deposits_collected is computed, then it contributes nothing to the total.

**FEAT-25.SPEC-003-AC-07:** Given a Deposit Transaction is Forfeited due to a no-show mark within the selected period, when saved_amount is computed, then the amount minus the processor_fee is included in the total.

**FEAT-25.SPEC-003-AC-08:** Given a Deposit Transaction is Refunded in full within the selected period, when saved_amount is computed, then it contributes nothing to the total.

**FEAT-25.SPEC-003-AC-09:** Given 6 Services have bookings within the selected period and the 5th and 6th are tied in count, when most_booked_services is computed, then both the 5th- and 6th-ranked Services are included.

**FEAT-25.SPEC-003-AC-10:** Given a Service has zero bookings within the selected period, when most_booked_services is computed, then that Service is omitted from the list entirely.

**FEAT-25.SPEC-003-AC-11:** Given Talia's lifetime Completed/No-Show booking count is below platform parameter: `insights-minimum-history-threshold`, when data_sufficient is evaluated, then it resolves to false.

**FEAT-25.SPEC-003-AC-12:** Given Talia's lifetime Completed/No-Show booking count exactly equals platform parameter: `insights-minimum-history-threshold`, when data_sufficient is evaluated, then it resolves to true.

**FEAT-25.SPEC-003-AC-13:** Given Talia (the Pro) requests her own Period Insights Summary, when the request is authorized, then it is allowed without restriction.

**FEAT-25.SPEC-003-AC-14:** Given Support requests a Period Insights Summary for the one Pro account under active review, when the request is authorized, then it is allowed and returns the same figures the Pro would see.

**FEAT-25.SPEC-003-AC-15:** Given Riley (the Client) attempts to view any Pro's Period Insights Summary, when the attempt is evaluated, then it is never allowed -- no navigation path exists for her role to originate the request.

**FEAT-25.SPEC-003-AC-16:** Given any role attempts to view a comparison of one Pro's figures against another's, when the attempt is evaluated, then it is never allowed, per SC-03 -- no such view or control exists in the product.

**FEAT-25.SPEC-003-AC-17:** Given Talia or Support changes the selected period, when the action is evaluated, then it is always allowed, since it is a read-only action that writes nothing.

**FEAT-25.SPEC-003-AC-18:** Given a Deposit Transaction's forfeiture timestamp falls in a different period than its capture timestamp, when deposits_collected and saved_amount are each computed for the relevant periods, then each figure counts the event in the period its own relevant timestamp falls in, and the two figures are not forced to reconcile.

**FEAT-25.SPEC-003-AC-19:** Given Talia changes her account timezone between two historical booking dates, when a period is later resolved, then the resolution uses her current account timezone at request time, not the timezone in effect when those bookings were made.

**FEAT-25.SPEC-003-AC-20:** Given a Booking undergoes a compound (inside-window, late) reschedule and its original reaches terminal Rescheduled while the newly created Booking (FEAT-10.SPEC-004) independently reaches Confirmed on its own fresh deposit capture, when total_bookings is computed for the period containing each start_time, then the original Booking is excluded (Rescheduled is not a counted state) and the new Booking is counted once, on its own confirmation, so the appointment is never counted twice and never counted zero times.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 6 | 6 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



# Automation Spec: Historical Aggregate Maintenance

## Overview

**Name:** Historical Aggregate Maintenance
**ID:** FEAT-25.SPEC-004
**Type:** Automation
**Purpose:** Maintains rolling per-period aggregates as bookings, deposit outcomes, no-show marks, and payout figures occur, so the summary stays responsive as a Pro's history grows across years.
**Parent Feature:** FEAT-25 -- Booking & Revenue Insights

## Scope and Non-Goals

**In Scope:**
- Incrementally updating a per-day rolling aggregate, per Pro, whenever a qualifying booking or deposit-outcome event occurs, so FEAT-25.SPEC-002 never needs to scan the Pro's full raw history to answer a period request
- Maintaining booking counts, deposits-collected amounts, forfeited-no-show ("saved") amounts, and per-service booking tallies within each day's aggregate
- Keeping every day's aggregate available for the life of the Pro's account, consistent with SC-22's retention standard

**Non-Goals:**
- Computing or returning a period summary to the Insights Summary Screen -- owned by FEAT-25.SPEC-002; this automation only maintains the aggregates it reads
- Defining the counting, derivation, or ranking formulas applied to the aggregates -- owned by FEAT-25.SPEC-003; this automation tallies raw counts and amounts using those same category definitions (e.g., which Booking states count), but the summary-level formulas themselves live in FEAT-25.SPEC-003
- Writing to Booking, Deposit Transaction, Service, or Payout Account -- this automation only reads those records as they change; it never modifies them, consistent with this feature's Data Notes ("Derived: entirely")
- Real-time push of updated figures to an open Insights Summary Screen -- excluded per this feature's own States field, which specifies only "a brief indicator while aggregating" on load/period-change, not a live-updating view; a screen already open does not need to reflect an aggregate update the instant it happens

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking confirmed (deposit captured) | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires when a Booking atomically transitions from Pending Payment to Confirmed, on a successful deposit capture (dependency map: "Updated by FEAT-07 (pending -> confirmed)") | Booking's Pro Account reference, service reference, start_time |
| Booking completed | FEAT-12.SPEC-006 (Booking Completion Rules) | Fires when a Booking transitions to Completed (manual mark-complete or FEAT-12.SPEC-004's auto-completion sweep) | Booking's Pro Account reference, service reference, start_time |
| Booking rescheduled (start_time changed for a counted booking, in place) | FEAT-10.SPEC-004 (Booking Update Commit), FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Fires when a Confirmed or Awaiting Outcome Booking's start_time is updated in place by an outside-window reschedule. FEAT-30.SPEC-007 Pro reschedules are always in-place and are fully covered by this row -- a Pro reschedule never produces the compound terminal-Rescheduled transition below. | Booking's Pro Account reference, service reference, original start_time, new start_time |
| Booking rescheduled to terminal Rescheduled (compound, inside-window late reschedule -- reversing the original's count) | FEAT-10.SPEC-004 (Booking Update Commit) only | Fires when a Confirmed or Awaiting Outcome Booking transitions to the terminal Rescheduled state as part of an inside-window (late) reschedule commit (FEAT-10.SPEC-004 Processing Logic step 7 / "Reschedule committed, inside window (late reschedule, compound)"). Not applicable to FEAT-30.SPEC-007, whose Pro-initiated reschedules are always in-place (covered by the row above) and never produce this terminal transition. The compound commit's newly created Booking is a separate Booking record that fires its own, independent Booking-confirmed trigger (the first row of this table) once its own fresh deposit is captured -- it is not part of this trigger's payload. | Original Booking's Pro Account reference, service reference, original start_time |
| Booking cancelled (reversing a counted booking) | FEAT-10.SPEC-004 (Booking Update Commit), FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Fires when a Confirmed or Awaiting Outcome Booking transitions to Cancelled by Client or Cancelled by Pro | Booking's Pro Account reference, service reference, start_time |
| No-show marked | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Fires when a Booking is marked No-Show | Booking's Pro Account reference, service reference, start_time |
| No-show mark undone | FEAT-11.SPEC-003 (No-Show Mark Undo) | Fires when a No-Show mark is undone within its grace window | Booking's Pro Account reference, service reference, start_time |
| Deposit outcome set (captured) | FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Fires when a deposit capture succeeds | Deposit Transaction's Booking reference, Pro Account reference, amount, processor_fee, capture timestamp |
| Deposit outcome set (refund in progress, refunded, or forfeited) | FEAT-09.SPEC-005 (automatic refund/forfeit), FEAT-11.SPEC-002 (forfeiture), FEAT-30.SPEC-011 (goodwill refund) | Fires when a Deposit Transaction's status changes to Refund in Progress, Refunded, or Forfeited | Deposit Transaction's Booking reference, Pro Account reference, amount, processor_fee, outcome_reason, outcome timestamp |
| Payout figure reported | FEAT-28.SPEC-006 (Payout Account Connection Verification) | Fires when the connected payout capability reports a new or updated payout figure for a Pro | Pro Account reference, payout amount, payout date |

## Processing Logic

1. Receive the qualifying event (booking confirmed, completed, rescheduled, or cancelled; no-show marked or undone; deposit outcome set; or payout figure reported) with its Pro Account reference and event timestamp.
2. Resolve the calendar day (in the Pro's account timezone) the event's relevant timestamp falls on.
3. Locate that Pro's rolling aggregate for that day, creating a new empty one first if this is the first qualifying event recorded for that Pro on that day.
4. Depending on the event type, update the day's aggregate:
   - **Booking confirmed:** increment the day's booking count by one, attributed to the Booking's start_time day and its Service. This is the single point at which a Booking first becomes counted, per FEAT-25.SPEC-003's total-bookings counting rule (which counts a Booking in state Confirmed, Awaiting Outcome, Completed, or No-Show); no further booking-count increment occurs for the same Booking at any later transition into one of those same-counted states.
   - **Booking completed:** no change to the day's booking-count or per-service aggregates -- the Booking was already counted at its Booking-confirmed transition.
   - **No-show marked:** no change to the day's booking-count aggregate -- the Booking was already counted at its Booking-confirmed transition; see the "Deposit forfeited (no-show outcome)" bullet below for this event's saved-from-no-shows contribution.
   - **Booking rescheduled, in place (outside-window):** move the earlier Booking-confirmed increment with it -- decrement the booking count and per-service tally by one at the original start_time's day, and increment both by one at the new start_time's day, so the Booking remains counted exactly once, at its current start_time, per FEAT-25.SPEC-003's rescheduled-booking rule. This applies only to the in-place mechanism (same Booking record, start_time updated); it never applies to the compound terminal-Rescheduled transition below.
   - **Booking rescheduled to terminal Rescheduled (compound, inside-window late reschedule):** decrement the day's booking count by one and its per-service tally by one, attributed to the original Booking's start_time day, reversing its earlier Booking-confirmed increment -- mirroring the cancellation reversal below, because Rescheduled is a terminal, non-counted state per FEAT-25.SPEC-003. This decrement is independent of the new Booking created by the same compound commit: the new Booking's own eventual Booking-confirmed transition (on its own fresh deposit capture) adds its own increment separately, via the ordinary Booking-confirmed trigger, at its own start_time day -- the two events are never merged into one update, and the new Booking contributes nothing until it is confirmed on its own terms.
   - **Booking cancelled (Cancelled by Client or Cancelled by Pro):** decrement the day's booking count by one and its per-service tally by one, attributed to the Booking's (current) start_time day, reversing the Booking-confirmed increment -- a cancelled Booking is excluded from total_bookings per FEAT-25.SPEC-003 and must not remain counted. A Booking cancelled before ever reaching Confirmed (it never left Pending Payment) has no prior increment to reverse, so this event is a no-op for it.
   - **Deposit captured:** add (amount minus processor_fee) to the day's deposits-collected total, attributed to the day the capture timestamp falls on.
   - **Deposit forfeited (no-show outcome):** add (amount minus processor_fee) to the day's saved-from-no-shows total, attributed to the day the forfeiture timestamp falls on.
   - **Deposit marked Refund in Progress:** no change to the day's deposits-collected total -- FEAT-25.SPEC-003 still counts a Refund-in-Progress deposit toward deposits_collected, so nothing is subtracted until the refund reaches its terminal Refunded status.
   - **Deposit refunded (terminal Refunded status):** subtract the refunded (amount minus processor_fee) from the day's deposits-collected total for the day the original capture occurred, per FEAT-25.SPEC-003's deposits-collected rule. This subtraction applies only on the transition to the terminal Refunded status, never at the earlier Refund in Progress transition.
   - **No-show mark undone:** subtract the previously added (amount minus processor_fee) from the day's saved-from-no-shows total, for the day the original forfeiture was recorded, reversing that forfeiture aggregate entry. The day's booking-count aggregate is unchanged by this event -- the Booking remains in a counted state (returning to Awaiting Outcome) both before and after the undo, per FEAT-25.SPEC-003.
   - **Payout figure reported:** no change to booking, deposits-collected, saved, or per-service aggregates -- payout figures are read directly by FEAT-28's own money list and are not aggregated by this spec for FEAT-25's booking-and-revenue figures (this feature's Interactions field notes FEAT-28 as a source for "amounts received," which FEAT-25.SPEC-002 reads through FEAT-28.SPEC-005 rather than through a duplicate aggregate maintained here).
5. For a booking-count-affecting event (Booking confirmed, Booking rescheduled in place, Booking rescheduled to terminal Rescheduled, or Booking cancelled), the day's per-service tally for the Booking's Service is updated together with the booking count, per the bullets above -- never independently of it.
6. Persist the updated day aggregate so it is available to FEAT-25.SPEC-002 on the next read.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Aggregate updated | A qualifying event is processed successfully | The relevant day's rolling aggregate (booking count, deposits-collected, saved amount, per-service tally) is updated | None directly -- the Pro sees no immediate feedback from this automation; the effect surfaces only the next time FEAT-25.SPEC-002 reads the aggregate | FEAT-25.SPEC-002 |
| No-op (payout figure reported) | A payout figure is reported | None to this feature's aggregates, per the processing logic's payout-figure step | None | None within this feature |
| Aggregate update deferred and retried | The aggregate update cannot complete at event time (e.g., the day's aggregate record is temporarily unreachable) | None persisted yet; the event is retried up to platform parameter: `insights-aggregate-retry-count` times before being logged as a gap | None user-visible -- non-blocking; the triggering event's own success (e.g., the booking completion, the deposit capture) is never delayed or reverted by an aggregate-update failure | FEAT-25.SPEC-002 (may compute a period that is transiently missing one event until the retry succeeds) |
| Aggregate update permanently failed after retries | All retries for a given event are exhausted | The event's contribution is not reflected in the day's aggregate | None user-visible in the moment; a persistently missing contribution would only ever be noticed as a smaller-than-expected figure in FEAT-25.SPEC-001, which carries no error indicator of its own for this case, since the underlying source record (Booking, Deposit Transaction) remains fully correct and unaffected | FEAT-25.SPEC-002 |

## Data Model

**Reads:** Booking (Pro Account reference, service reference, start_time, state) at each qualifying transition; Deposit Transaction (Booking reference, Pro Account reference, amount, processor_fee, status, outcome_reason, timestamps) at each qualifying outcome change. Neither is ever written by this automation.
**Creates:** A new day's rolling aggregate record, for a Pro's first qualifying event on a calendar day that has no aggregate yet. This is an internal maintenance record owned entirely by this feature, not a product entity defined in the Feature Dependency Map.
**Updates:** The relevant day's rolling aggregate -- booking count, deposits-collected total, saved-from-no-shows total, and per-service tally -- per the processing logic above.
**Deletes:** None -- aggregates are retained for the life of the Pro's account, consistent with SC-22's history-retention standard; they are never pruned while the account exists.

## Business Rules

- This automation never blocks or delays the event that triggers it: a booking completion, a deposit outcome, or a no-show mark all succeed on their own terms (FEAT-12, FEAT-07, FEAT-09, FEAT-11) regardless of whether the aggregate update that follows succeeds immediately.
- Aggregates are bucketed by calendar day in the Pro's account timezone (XBR-25) so that FEAT-25.SPEC-002 can assemble any Week, Month, or Year period by summing a bounded number of day-buckets, never by scanning raw history.
- A Booking is counted into the day's booking-count aggregate exactly once, at its Booking-confirmed transition (deposit-capture success, FEAT-07.SPEC-002), and remains counted through every later state FEAT-25.SPEC-003 treats as counted (Awaiting Outcome, Completed, No-Show) without any further increment. A transition to Cancelled by Client, Cancelled by Pro, or terminal Rescheduled (the original Booking's side of a compound inside-window late reschedule, FEAT-10.SPEC-004 only) each reverses that count, since FEAT-25.SPEC-003 excludes all three states from total_bookings; a Booking that never reached Confirmed (e.g., Expired (unpaid) from Pending Payment) was never counted and needs no reversal.
- A compound (inside-window, late) reschedule's original Booking and its newly created replacement Booking are two separate records with two independent aggregate lifecycles: the original's reversal (above) fires the moment it reaches terminal Rescheduled, and the new Booking's own increment fires only when it separately reaches Confirmed on its own fresh deposit capture -- the same ordinary Booking-confirmed trigger every new booking uses. Nothing here treats the pair as one appointment for aggregate purposes; each record's contribution is entirely its own, so the appointment is counted at most once at any given moment and is never left double-counted across the pair.
- A no-show mark that is later undone (FEAT-11.SPEC-003, within its grace window) reverses only the saved-from-no-shows contribution it made; the booking-count contribution is untouched, since the Booking remains in a counted state (Awaiting Outcome) both before and after the undo, per FEAT-25.SPEC-003.
- A deposit's deposits-collected contribution is reversed only on its transition to the terminal Refunded status; a Refund-in-Progress transition changes nothing, since FEAT-25.SPEC-003 still counts a deposit in that status toward deposits_collected.
- Payout figures reported through FEAT-28 are not duplicated into this feature's own aggregates; FEAT-25.SPEC-002 reads "amounts received" figures directly from FEAT-28.SPEC-005's composition instead, avoiding two competing sources of the same number.
- ASMP-22 / SC-22: per-day aggregates are retained for the life of the account, so a Pro's full multi-year history remains summarizable without ever re-deriving it from raw records.

## Edge Cases

- **Concurrent trigger firing (a booking is completed and its deposit is captured at effectively the same moment)** -- Each event updates its own aggregate fields (booking count vs. deposits-collected) independently; both updates apply to the same day's aggregate without conflicting, since they touch different fields within it.
- **Trigger fires while a previous run for the same Pro and day is still in flight** -- Updates to the same day's aggregate for the same Pro are applied one at a time in the order their triggering events occurred; a second event for the same Pro and day waits for the first update to complete before applying, so no update is silently lost or overwritten.
- **A booking is completed for a day whose aggregate does not exist yet (the Pro's very first booking)** -- A new aggregate record for that day is created as part of the same update, per the processing logic's step 3.
- **A deposit is refunded for a booking whose original capture aggregate day is far in the past (multi-year history)** -- The refund's subtraction is applied to that original day's aggregate directly, regardless of how long ago it was recorded, since aggregates are retained for the life of the account and remain individually addressable by day.
- **The aggregate update fails and exhausts its retries** -- The underlying Booking or Deposit Transaction record is entirely unaffected (this automation never writes to them); only this feature's own derived figures may transiently under-report until a subsequent related event on the same booking (if any) prompts a fresh update, or until an operator-level reconciliation outside this spec's scope corrects it.
- **A Pro account is created with no history yet** -- No aggregates exist until the first qualifying event; FEAT-25.SPEC-002's data-sufficiency check (FEAT-25.SPEC-003) independently determines that the Pro sees "not enough data yet" regardless of aggregate presence.
- **A Booking is cancelled after being counted at its Booking-confirmed transition** -- The day's booking count and per-service tally, at the Booking's start_time day, are decremented by one each, so the cancelled Booking no longer contributes to total_bookings, consistent with FEAT-25.SPEC-003; a Booking cancelled while still Pending Payment (never confirmed) triggers no decrement, since it was never counted.
- **A counted Booking (Confirmed or Awaiting Outcome) is rescheduled in place to a start_time on a different calendar day before it completes** -- The booking-count and per-service tally increment moves from the original day's aggregate to the new day's aggregate; the Booking is never counted on both days at once, and is found only at its current start_time day, per FEAT-25.SPEC-003's rescheduled-booking rule.
- **A counted Booking undergoes a compound (inside-window, late) reschedule -- the original transitions to terminal Rescheduled and FEAT-10.SPEC-004 creates a new Booking for the chosen time** -- The original's day and per-service tally are decremented at its (original) start_time day the moment it reaches terminal Rescheduled, exactly as a cancellation reversal; this happens regardless of whether the new Booking ever completes its own deposit. The new Booking contributes its own increment only later, and only if it independently reaches Confirmed -- so the appointment is briefly uncounted between the original's reversal and the new Booking's own confirmation, is never counted twice, and if the new Booking never pays, is never counted at all.
- **The new Booking created by a compound late reschedule fails to reach Confirmed (its deposit is never captured, e.g., the client abandons payment)** -- No increment is ever recorded for it; the original Booking's decrement (already applied at its terminal-Rescheduled transition) stands unreversed, since the original genuinely did not occur as originally scheduled and the replacement never occurred either. This is the same outcome FEAT-25.SPEC-003 defines for any booking that never reaches a counted state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | The Pending Payment -> Confirmed transition fires the booking-count and per-service aggregate increment |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | A client-initiated cancellation, an outside-window (in-place) reschedule, or the original Booking's transition to terminal Rescheduled in a compound inside-window (late) reschedule each fires its corresponding booking-count reversal or day-bucket move; the compound case's newly created Booking separately fires its own Booking-confirmed trigger (FEAT-07.SPEC-002, routed through FEAT-07's ordinary deposit flow) once its own deposit is captured, independent of this trigger |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | A Pro-initiated cancellation or reschedule fires the corresponding booking-count reversal or day-bucket move |
| FEAT-12.SPEC-006 (Booking Completion Rules) | Triggered by (inbound) | Booking completion fires an aggregate update (no-op for booking count, already counted at confirmation) |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) | No-show marking fires a saved-amount aggregate update (no-op for booking count, already counted at confirmation) |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | Triggered by (inbound) | A no-show undo reverses the saved-from-no-shows aggregate contribution only; the booking count is untouched |
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Triggered by (inbound) | A successful deposit capture fires a deposits-collected aggregate update |
| FEAT-09.SPEC-005 (automatic refund/forfeit integration) | Triggered by (inbound) | An automatic refund or forfeit outcome fires the corresponding aggregate update |
| FEAT-30.SPEC-011 (goodwill/bulk refund execution) | Triggered by (inbound) | A Pro-issued goodwill refund fires a deposits-collected aggregate update |
| FEAT-28.SPEC-006 (Payout Account Connection Verification) | Triggered by (inbound) | A reported payout figure is received but produces no aggregate change (see Processing Logic step 4) |
| FEAT-25.SPEC-002 (Period Insights Aggregation) | Affects (outbound) | Reads the maintained rolling aggregates for every computation |

## Analytics and Success Signals

- **insights_aggregate_updated** (event type: booking-confirmed / booking-completed / booking-rescheduled-in-place / booking-rescheduled-terminal / booking-cancelled / no-show-marked / no-show-undone / deposit-captured / deposit-refund-in-progress / deposit-refunded / deposit-forfeited, day bucket) -- N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; recorded as internal maintenance instrumentation so aggregate-update volume can be observed operationally
- **insights_aggregate_update_failed** (event type, retry count exhausted: yes/no) -- N/A -- no metric in success-metrics.md names Booking & Revenue Insights as its Connected Feature; recorded so a persistently failing aggregate path can be noticed operationally even though it never blocks the triggering feature and carries no user-facing error state of its own

## Acceptance Criteria

**FEAT-25.SPEC-004-AC-01:** Given Talia's client's deposit is captured and the Booking transitions from Pending Payment to Confirmed, when the confirmation event fires, then that day's booking-count aggregate for Talia increments by one, attributed to the Service booked.

**FEAT-25.SPEC-004-AC-02:** Given a Booking already counted at its Booking-confirmed transition is later marked No-Show, when the mark event fires, then no additional increment occurs to the day's booking-count aggregate, since it is already counted.

**FEAT-25.SPEC-004-AC-03:** Given a No-Show mark is undone within its grace window and a forfeiture aggregate entry had been recorded for it, when the undo event fires, then the previously added saved-from-no-shows amount is subtracted from that day's aggregate.

**FEAT-25.SPEC-004-AC-04:** Given a deposit is Captured with a processor_fee, when the capture event fires, then (amount minus processor_fee) is added to the capture day's deposits-collected aggregate.

**FEAT-25.SPEC-004-AC-05:** Given a deposit is Forfeited due to a no-show mark, when the forfeiture event fires, then (amount minus processor_fee) is added to the forfeiture day's saved-from-no-shows aggregate.

**FEAT-25.SPEC-004-AC-06:** Given a previously Captured deposit is later Refunded, when the refund event fires, then the refunded (amount minus processor_fee) is subtracted from the original capture day's deposits-collected aggregate.

**FEAT-25.SPEC-004-AC-07:** Given a payout figure is reported through FEAT-28, when the event fires, then no change occurs to this feature's booking, deposits-collected, saved, or per-service aggregates.

**FEAT-25.SPEC-004-AC-08:** Given a Pro's first-ever qualifying event occurs on a day with no existing aggregate, when the event is processed, then a new day aggregate is created as part of that same update.

**FEAT-25.SPEC-004-AC-09:** Given a booking completion and a deposit capture for the same booking occur at effectively the same moment, when both events fire, then each updates its own aggregate field on the same day's aggregate without conflicting with the other.

**FEAT-25.SPEC-004-AC-10:** Given two events for the same Pro and the same day arrive while a prior update for that Pro and day is still being applied, when the second event is processed, then it applies after the first completes, and neither update is lost.

**FEAT-25.SPEC-004-AC-11:** Given an aggregate update fails and exhausts its retries (platform parameter: `insights-aggregate-retry-count`), when the failure is finalized, then the underlying Booking or Deposit Transaction record remains entirely unaffected.

**FEAT-25.SPEC-004-AC-12:** Given a deposit is refunded for a booking whose original capture day is several years in the past, when the refund event fires, then the subtraction is applied to that original day's aggregate directly.

**FEAT-25.SPEC-004-AC-13:** Given a Pro Account has no history yet, when Talia's Insights Summary Screen is opened, then no aggregate exists to read, and FEAT-25.SPEC-003's data-sufficiency check independently returns "not enough data yet" regardless.

**FEAT-25.SPEC-004-AC-14:** Given a booking completion event and a deposit capture event for two different Pros occur at the same moment, when both are processed, then each Pro's aggregate updates independently with no cross-Pro interaction.

**FEAT-25.SPEC-004-AC-15:** Given Talia's account timezone determines the calendar-day bucket for an event, when an event's timestamp is resolved to a day, then the resolution uses her account timezone (XBR-25), not a fixed reference timezone.

**FEAT-25.SPEC-004-AC-16:** Given a previously Captured deposit transitions to Refund in Progress, when the transition event fires, then no change occurs to the day's deposits-collected aggregate, since FEAT-25.SPEC-003 still counts a Refund-in-Progress deposit toward deposits_collected.

**FEAT-25.SPEC-004-AC-17:** Given a Booking already counted at its Booking-confirmed transition is later marked Completed, when the completion event fires, then no additional increment occurs to the day's booking-count aggregate.

**FEAT-25.SPEC-004-AC-18:** Given a counted Booking (Confirmed or Awaiting Outcome) is cancelled by the Client or the Pro, when the cancellation event fires, then that day's booking-count aggregate and per-service tally are each decremented by one, reversing the earlier Booking-confirmed increment.

**FEAT-25.SPEC-004-AC-19:** Given a Booking is cancelled while still Pending Payment (never confirmed), when the cancellation event fires, then no booking-count or per-service decrement occurs, since the Booking was never counted.

**FEAT-25.SPEC-004-AC-20:** Given a counted Booking is rescheduled to a new start_time falling on a different calendar day, when the reschedule event fires, then the booking-count and per-service tally increment moves from the original day's aggregate to the new day's aggregate, so the Booking is counted exactly once.

**FEAT-25.SPEC-004-AC-21:** Given a No-Show mark is undone within its grace window, when the undo event fires, then the day's booking-count aggregate is unchanged, since the Booking remains in a counted state (Awaiting Outcome) both before and after the undo.

**FEAT-25.SPEC-004-AC-22:** Given a counted Booking (Confirmed or Awaiting Outcome) undergoes a compound (inside-window, late) reschedule and its original transitions to terminal Rescheduled (FEAT-10.SPEC-004), when the terminal-Rescheduled event fires, then that day's booking-count aggregate and per-service tally, at the original Booking's start_time day, are each decremented by one, reversing the earlier Booking-confirmed increment -- independent of whether the new Booking created by the same commit ever reaches Confirmed.

**FEAT-25.SPEC-004-AC-23:** Given a compound late reschedule's newly created Booking (FEAT-10.SPEC-004) separately reaches Confirmed on its own fresh deposit capture, when its own Booking-confirmed event fires, then that day's booking-count aggregate and per-service tally, at the new Booking's start_time day, are incremented by one via the ordinary Booking-confirmed trigger -- this increment is never merged with, and never depends on, the original Booking's terminal-Rescheduled decrement, so the appointment is counted exactly once across the pair.

**FEAT-25.SPEC-004-AC-24:** Given a compound late reschedule's original Booking has already reached terminal Rescheduled (and been decremented) and its newly created Booking never reaches Confirmed (its deposit is never captured), when no further event fires for either record, then no booking-count increment is ever recorded for the appointment, and the original's decrement is not reversed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 10 (booking confirmed, booking completed, booking rescheduled in place, booking rescheduled to terminal Rescheduled (compound), booking cancelled, no-show marked, no-show undone, deposit captured, deposit refund-in-progress/refunded/forfeited, payout figure reported) | 10 |
| Outcome Paths | 4 (updated, no-op, deferred-and-retried, permanently failed) | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 10 | 10 |


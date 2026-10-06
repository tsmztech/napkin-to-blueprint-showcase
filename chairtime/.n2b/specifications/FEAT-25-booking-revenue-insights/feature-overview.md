---
document_type: feature-overview
feature_number: FEAT-25
feature_name: Booking & Revenue Insights
feature_slug: booking-revenue-insights
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

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

---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-002
spec_name: Period Insights Aggregation
spec_slug: period-insights-aggregation
parent_feature: FEAT-25
parent_feature_name: Booking & Revenue Insights
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

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

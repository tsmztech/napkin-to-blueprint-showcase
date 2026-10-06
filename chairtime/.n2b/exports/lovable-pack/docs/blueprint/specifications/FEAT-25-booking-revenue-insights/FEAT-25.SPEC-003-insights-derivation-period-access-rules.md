---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-25.SPEC-003
spec_name: Insights Derivation, Period & Access Rules
spec_slug: insights-derivation-period-access-rules
parent_feature: FEAT-25
parent_feature_name: Booking & Revenue Insights
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 28
acceptance_criteria_count: 20
---

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

---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-004
spec_name: Historical Aggregate Maintenance
spec_slug: historical-aggregate-maintenance
parent_feature: FEAT-25
parent_feature_name: Booking & Revenue Insights
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 24
---

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

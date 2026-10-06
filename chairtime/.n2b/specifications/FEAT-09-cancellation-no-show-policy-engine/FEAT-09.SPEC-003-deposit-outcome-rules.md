---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-003
spec_name: Deposit Outcome Rules
spec_slug: deposit-outcome-rules
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 15
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Deposit Outcome Rules

## Overview

**Name:** Deposit Outcome Rules
**ID:** FEAT-09.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the complete, binary, symmetric rule set governing what happens to a booking's deposit for every combination of who acts (client or Pro), what they do (cancel, reschedule, no-show), and when they do it relative to the policy's cutoff -- the single source of truth every evaluation and preview in the product reads instead of re-deriving.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Deposit Transaction (outcome-determination slice), evaluated jointly against Booking (timing and initiator) and Cancellation Policy (bound version's cutoff)

## Scope and Non-Goals

**In Scope:**
- The complete decision table covering client cancellation (outside/inside the window), no-show, Pro-initiated cancellation, and reschedule (outside/inside the window)
- The exact condition that determines "outside" vs. "inside" the window for each action type
- Authorization over who may view a determined outcome, and the explicit statement that no role may override the automatic determination
- The rationale and boundary conditions for every rule, so downstream evaluation (FEAT-09.SPEC-004) never has to interpret intent

**Non-Goals:**
- Actually evaluating a specific cancellation/reschedule/no-show event and writing its outcome to the Deposit Transaction -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation); this spec defines the rule table that evaluation applies, not the evaluation process itself
- Executing the refund through the payment-processing capability -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund)
- Applying the forfeiture flag as a terminal Forfeited state -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture), per the Key Capability's own wording and the dependency map's Updated-by list for Deposit Transaction
- Partial refunds or tiered cancellation schedules -- excluded per SC-18: BRIEF.md's Business Context defines a binary rule; this spec never computes a percentage-based or sliding-scale outcome, however close to the cutoff an action lands
- Automatic detection of a "genuine emergency" exception to any of these rules -- excluded per the Validation & Limits field ("the policy is binary in v1") and SC-17/SC-18; a goodwill override is always a manual Pro decision through FEAT-30, never a rule this spec or FEAT-09.SPEC-004 infers automatically

## Governed Entity

**Entity:** Deposit Transaction (outcome_reason field, and the status transition it sets up for FEAT-09.SPEC-004/FEAT-09.SPEC-005/FEAT-11 to apply), evaluated against Booking and Cancellation Policy
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Deposit Transaction.status | enum | Current status at the moment of evaluation; this spec's rules apply only while status is Captured (an already-Refunded, Forfeited, Refund in Progress, or Disputed transaction is never re-evaluated, per the dependency map's "once per deposit" Contention rule) |
| Deposit Transaction.outcome_reason | text | Which rule below produced the determination; set by FEAT-09.SPEC-004 from this spec's table, not by this spec directly |
| Booking.start_time | date/time | The appointment's scheduled start, in the Pro's timezone |
| Booking's cancellation/reschedule/no-show timestamp | date/time | When the triggering action occurred, read by FEAT-09.SPEC-004 and compared against the booking's cutoff (FEAT-09.SPEC-002) |
| Booking's cancellation/reschedule initiator | enum (Client \| Pro) | Who performed the action; determines which half of the symmetric rule set applies |
| Cancellation Policy's bound version, window_hours | reference / number | The version acknowledged by this specific booking, and its window, both read via FEAT-09.SPEC-002 to compute the cutoff this spec's rules compare against |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Applies this spec's rule table at the moment a cancellation, reschedule, or no-show marking is recorded, to determine the outcome it writes |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Cross-feature | Previews the outcome this spec determines to the client before they confirm a cancellation or reschedule |
| FEAT-30 (Pro Booking Management) | Cross-feature | Applies the Pro-cancellation-always-refunds rule when Talia cancels a booking |

## Field Validation Rules

No input fields exist on this spec's governed slice -- outcome_reason is always system-derived from the rule table below, never entered directly by any role.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction.outcome_reason | No direct validation -- always derived from this spec's rule table by FEAT-09.SPEC-004; never accepts direct input from any role | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Client cancellation, outside the window | timestamp, cutoff, initiator | If initiator = Client and timestamp is at or before the cutoff, outcome = full refund | N/A -- automatic determination, no client-facing error |
| Client cancellation, inside the window | timestamp, cutoff, initiator | If initiator = Client and timestamp is after the cutoff, outcome = deposit kept | N/A -- automatic determination, no client-facing error |
| No-show | Booking marked no-show | A no-show marking always evaluates as deposit kept, regardless of how the computed cutoff compares to the marking time -- a no-show is treated as always inside-window-equivalent (product-features.md, Primary Flows) | N/A -- automatic determination, no client-facing error |
| Pro-initiated cancellation | initiator | If initiator = Pro, outcome = full refund, unconditionally -- the window is never applied to a Pro-initiated cancellation | N/A -- automatic determination, no client-facing error |
| Client reschedule, outside the window | timestamp, cutoff, initiator | If initiator = Client, action = reschedule, and timestamp is at or before the cutoff, the existing deposit carries over to the new appointment time -- no new charge, no outcome change | N/A -- automatic determination, no client-facing error |
| Client reschedule, inside the window | timestamp, cutoff, initiator | If initiator = Client, action = reschedule, and timestamp is after the cutoff, the compound outcome applies: the original deposit is kept (treated as a late cancellation), and the new appointment requires its own fresh deposit, shown to the client before they confirm | N/A -- automatic determination, no client-facing error |
| Pro-initiated reschedule | initiator | If initiator = Pro, action = reschedule, the client is never exposed to the window: the existing deposit always carries over to the new time, regardless of timing | N/A -- automatic determination, no client-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger an outcome determination (by cancelling, rescheduling, or being marked no-show) | The Client (Riley), The Pro (Talia) | Riley: her own booking only, via FEAT-10; Talia: her own bookings, via FEAT-11 (no-show) or FEAT-30 (cancel/reschedule) | -- |
| View a determined outcome | The Pro (Talia), The Client (Riley) | Talia: any of her own bookings; Riley: her own booking only | -- |
| View determined outcomes for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Override or manually set an outcome different from this spec's rule table | Any role | Never -- the determination is always automatic; a goodwill refund overriding a kept deposit is a distinct, explicit Pro action through FEAT-30, not an override of this spec's determination | No control anywhere lets any role directly set outcome_reason; Talia's only path to a different financial result is FEAT-30's explicit goodwill refund action, which is recorded as its own action, not a correction to this spec's rule |
| Adjudicate whether a cancellation or no-show was "justified" | Any role | Never -- excluded per SC-17; the product provides the timestamped record, never a ruling | No adjudication control exists; a client who disputes an outcome contests the charge with their card issuer through the payment processor |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| outcome_reason | Derived from the rule table in Business Rules below, based on initiator, action type, and the timestamp-vs-cutoff comparison | At the moment FEAT-09.SPEC-004 processes a cancellation, reschedule, or no-show marking | No -- always automatic |
| "Outside" vs. "inside" the window | timestamp at or before the cutoff = outside; timestamp strictly after the cutoff = inside | Every timing-dependent rule row above | No |

## Business Rules

The complete outcome table (XBR-09), evaluated in this order of precedence -- initiator and action type first, then timing where the rule is timing-dependent:

| # | Trigger | Timing | Outcome |
|---|---------|--------|---------|
| 1 | Client cancels | Outside the window (at or before cutoff) | Full refund |
| 2 | Client cancels | Inside the window (after cutoff) | Deposit kept |
| 3 | Booking marked no-show | N/A -- always applies regardless of cutoff comparison | Deposit kept |
| 4 | Pro cancels | N/A -- window never applies | Full refund |
| 5 | Client reschedules | Outside the window (at or before cutoff) | Deposit carries over to the new time; no new charge |
| 6 | Client reschedules | Inside the window (after cutoff) | Original deposit kept (treated as a late cancellation per Rule 2); the new appointment requires its own fresh deposit, shown to the client before they confirm |
| 7 | Pro reschedules | N/A -- window never applies to a Pro-made reschedule | Deposit carries over to the new time; no new charge |

- Rules 1-2 and 5-6 are mutually exclusive and exhaustive for a client-initiated action: every client cancellation or reschedule falls into exactly one timing bucket, with no undefined middle case.
- Rule 3 takes precedence over Rules 1-2: once a booking is marked no-show, it is never re-evaluated as a "late cancellation" -- the two are mutually exclusive triggers.
- Rules 4 and 7 are absolute: no timing comparison is ever performed for a Pro-initiated action, so there is no "Pro cancels inside the window" variant.
- The outcome is always binary -- full refund or deposit kept in full -- per SC-18; no rule in this table produces a partial or percentage-based result.
- This rule table is the single authoritative source FEAT-10's pre-confirmation preview and FEAT-30's pro-cancellation flow both read, rather than re-deriving the rule independently (Shared Validation).

## Edge Cases

- **A cancellation timestamp lands exactly at the cutoff, to the minute** -- Rule 1/5 applies (at or before the cutoff = outside): the boundary favors the client, consistent with "at or before" in the rule definitions above.
- **A client reschedules more than once before the appointment** -- Each reschedule is evaluated independently against the (possibly new) appointment's own cutoff at the moment it occurs; an earlier outside-window reschedule that carried the deposit over does not exempt a later reschedule from its own timing check.
- **A no-show is marked, then the Pro later realizes the client did in fact show up** -- Outside this spec's scope: reversing a no-show marking is FEAT-11's undo mechanism (FEAT-11.SPEC-003); if reversed within its window, the outcome this spec determined is reversed by FEAT-11's own undo logic, not re-evaluated by this spec.
- **A Pro reschedules a booking that a client had already rescheduled inside the window (compound event)** -- Rule 7 applies to the Pro's own reschedule action independently; the earlier client-side late-reschedule outcome (Rule 6) on the original booking already resolved and is not revisited by the Pro's subsequent action on the new booking.
- **A client attempts to cancel a booking that has already been marked no-show** -- Not possible: per XBR-12, a no-show or completed booking cannot be cancelled or rescheduled, so Rules 1-2 and 5-6 can never fire against a booking Rule 3 has already resolved.
- **A booking's Deposit Transaction is already Refunded, Forfeited, Refund in Progress, or Disputed when a second triggering event somehow arrives** -- No rule in this table re-evaluates it; per the dependency map's Deposit Transaction Contention note, a terminal outcome is set once per deposit, and FEAT-09.SPEC-004 (not this spec) is responsible for refusing a second determination.
- **A goodwill refund is issued by Talia after a deposit was kept under Rule 2 or Rule 3** -- This is a distinct FEAT-30 action, not a correction of this spec's determination; the original outcome_reason set by this spec's rule table is never rewritten, and the goodwill refund is recorded as its own event.

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Riley cancels her booking at a time at or before the computed cutoff, then the outcome determined is a full refund (Rule 1).

**FEAT-09.SPEC-003-AC-02:** Given Riley cancels her booking at a time after the computed cutoff, then the outcome determined is deposit kept (Rule 2).

**FEAT-09.SPEC-003-AC-03:** Given Riley's booking is marked no-show, then the outcome determined is deposit kept, regardless of how the marking time compares to the cutoff (Rule 3).

**FEAT-09.SPEC-003-AC-04:** Given Talia cancels Riley's booking, then the outcome determined is a full refund, regardless of timing (Rule 4).

**FEAT-09.SPEC-003-AC-05:** Given Riley reschedules her booking at a time at or before the computed cutoff, then the existing deposit carries over to the new appointment time with no new charge and no outcome change (Rule 5).

**FEAT-09.SPEC-003-AC-06:** Given Riley reschedules her booking at a time after the computed cutoff, then the original deposit is kept as a late cancellation and she is shown, before confirming, that the new appointment requires its own fresh deposit (Rule 6).

**FEAT-09.SPEC-003-AC-07:** Given Talia reschedules Riley's booking, then the existing deposit carries over to the new time with no new charge, regardless of timing (Rule 7).

**FEAT-09.SPEC-003-AC-08:** Given a cancellation timestamp lands exactly at the computed cutoff, then it is treated as outside the window (full refund), per the "at or before" boundary rule.

**FEAT-09.SPEC-003-AC-09:** Given Riley's booking has already been marked no-show, when a cancellation attempt is made against it, then no such attempt can occur (XBR-12 blocks it upstream) and Rule 3's outcome stands.

**FEAT-09.SPEC-003-AC-10:** Given Talia looks for a control to set a different outcome than this spec's rule table would produce, then no such control exists anywhere in the product.

**FEAT-09.SPEC-003-AC-11:** Given Riley disputes that her deposit was correctly kept, when she looks for an in-product way to have Chairtime rule on the dispute, then none exists -- she is directed to contest the charge with her card issuer through the payment processor.

**FEAT-09.SPEC-003-AC-12:** Given Talia issues a goodwill refund through FEAT-30 after a deposit was kept under Rule 2, then the original outcome_reason this spec determined is never rewritten -- the goodwill refund is recorded as its own separate action.

**FEAT-09.SPEC-003-AC-13:** Given Platform Operator (Support) views a determined outcome for Talia's account under an active help request, then they see the outcome read-only, with no action to change it.

**FEAT-09.SPEC-003-AC-14:** Given a Deposit Transaction is already Forfeited, when a second triggering event is somehow recorded against the same booking, then this spec's rule table produces no second determination for it.

**FEAT-09.SPEC-003-AC-15:** Given Riley reschedules twice before her appointment, then each reschedule is evaluated independently against its own appointment's cutoff at the time it occurs.

**FEAT-09.SPEC-003-AC-16:** Given FEAT-10 previews a cancellation outcome to Riley before she confirms, then the preview reflects exactly this spec's rule table rather than a separately derived approximation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 7 | 7 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 7 (rule table rows) | 7 |
| Edge Cases | 7 | 7 |

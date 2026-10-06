---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-10.SPEC-005
spec_name: Cancellation Window & Eligibility Rule
spec_slug: cancellation-window-eligibility-rule
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 10
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Cancellation Window & Eligibility Rule

## Overview

**Name:** Cancellation Window & Eligibility Rule
**ID:** FEAT-10.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs whether a booking is currently eligible to be cancelled or rescheduled by its client, and computes the window countdown that determines which deposit-outcome branch applies.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule
**Governed Entity:** Booking (client-eligibility slice only)

## Scope and Non-Goals

**In Scope:**
- Whether a specific booking may currently be cancelled or rescheduled by the client who owns it (the eligibility gate)
- Computing and rendering the plain-language countdown to the cancellation window's cutoff, for display on FEAT-10.SPEC-001 and FEAT-10.SPEC-003
- Authorization over who may check eligibility and who may never override an ineligible result
- Boundary behavior at the exact cutoff moment, consistent with FEAT-09.SPEC-003's inclusive-boundary rule

**Non-Goals:**
- Computing the cutoff time itself from the bound policy version and window_hours -- owned by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this spec consumes that computed cutoff, never re-derives it
- Deciding which deposit-outcome branch applies once the window comparison is made -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec supplies the timing input that rule set consumes
- Evaluating and writing the actual deposit outcome once an action is recorded -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation)
- Pro-side eligibility for the Pro's own cancel/reschedule actions -- owned by Pro Booking Management (FEAT-30); XBR-09 states a Pro-made reschedule never exposes the client to the window, so this spec governs client-initiated eligibility only

## Governed Entity

**Entity:** Booking (client-eligibility slice: state, start_time, policy_version)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) -- this spec reads it to gate eligibility |
| start_time | date/time | The appointment's current start time, in the Pro's timezone; this spec reads it to compute the window countdown |
| policy_version | reference | The bound Cancellation Policy version; this spec reads it only to pass through to FEAT-09.SPEC-002's cutoff computation, never to interpret window_hours itself |

**Referenced (read-only):** Cancellation Policy -- window_hours and the computed cutoff, via FEAT-09.SPEC-002. This spec computes no cutoff of its own; it consumes FEAT-09.SPEC-002's rendered value and applies the eligibility gate around it.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-10.SPEC-001 | Cancel Booking | On screen load, before offering the Cancel action; the countdown is displayed live |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | On screen load, before offering the slot list at all |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | On screen load and again at the moment of confirm, to determine which outcome branch applies and to display the countdown |
| FEAT-10.SPEC-004 | Booking Update Commit | Re-checked at the instant of commit, as the authoritative eligibility gate before any state transition is written |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|------------------------------|---------------|-----------------|-----------|
| state | No validation beyond data type -- this spec reads it to gate eligibility; it is never entered or altered by any role through this spec | Always | -- | -- | -- |
| start_time | No validation beyond data type -- this spec reads it to compute the countdown; it is never entered or altered by any role through this spec | Always | -- | -- | -- |
| policy_version | No validation beyond data type -- this spec passes it through to FEAT-09.SPEC-002; it is never entered or altered by any role through this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|-------------------|-------|------------------|
| Eligibility is state-first, window-second | state, start_time, policy_version | Eligibility is checked first (state must not be Completed or No-Show, nor already Cancelled or Rescheduled); only an eligible booking's start_time and policy_version are then used to compute the window countdown -- an ineligible booking never reaches the countdown computation | "This booking can no longer be cancelled or rescheduled." (shown in place of any countdown) |
| Countdown reflects current start_time | start_time | The countdown is always computed against the booking's *current* start_time; for a booking being evaluated on FEAT-10.SPEC-003 for a prospective reschedule, the comparison uses the *original* (pre-reschedule) start_time, per FEAT-09.SPEC-003's Rule 5/Rule 6 definitions | N/A -- structural guarantee, not user-facing |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|-------------|-----------|------------------------------------------------|
| Check eligibility and view the window countdown for a booking | The Client (Riley) | Own-only -- their own booking only, verified via FEAT-06 before this feature's screens load | -- |
| Check eligibility and view the window countdown | The Pro (Talia) | Never through this spec -- the Pro's own eligibility for cancel/reschedule is governed separately by FEAT-30 | This spec is not reachable from any Pro-facing screen |
| Check eligibility and view the window countdown | Platform Operator (Support) | Never through this spec -- Support never uses or bypasses a client's access link (scope-boundaries SC-05) | No support entry point exists into this spec |
| Override an ineligible result (act on a Completed or No-Show booking anyway) | The Client (Riley) | Never | The Cancel/Reschedule actions are not shown; the Ineligible message is the only content offered in their place |
| Override an ineligible result | The Pro (Talia) | Never through this spec | Not applicable -- the Pro's own actions on a Completed/No-Show booking are governed by XBR-12 within FEAT-30, not by this spec |
| Override an ineligible result | Platform Operator (Support) | Never -- View-only per the Access Matrix | No override control exists for Support |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|------------------------|----------------|---------------------|
| eligible | Derived: true when state is one of Confirmed or Awaiting Outcome; false when state is Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, Expired (unpaid), or Pending Payment | Computed live on every screen load and again at commit | No -- always derived, never directly overridable |
| window_state | Derived: "outside" when the current moment is at or before the cutoff computed by FEAT-09.SPEC-002 against the relevant start_time; "inside" when after it | Computed live on every screen load and again at commit | No -- always derived, never directly overridable |
| countdown_display | Derived: the plain-language time remaining until the cutoff (e.g., "18 hours until the cancellation window closes"), or a plain statement that the window has already closed | Computed live on every screen load | No -- always derived from the cutoff FEAT-09.SPEC-002 renders |

## Business Rules

- XBR-08: every booking is governed by the cancellation policy version shown and acknowledged at booking; this spec reads the bound version but never edits or rebinds it.
- XBR-12: a booking already marked Completed or No-Show cannot be cancelled or rescheduled -- this is the eligibility gate's primary rule.
- The window boundary is inclusive of "outside": an action taken at exactly the cutoff moment (start_time minus window_hours, to the second) counts as outside the window, consistent with FEAT-09.SPEC-003's inclusive-boundary rule and the policy wording's "up to {window_hours} hours before" phrasing.
- A booking already in a terminal state from a prior action (Cancelled by Client, Cancelled by Pro, or Rescheduled) is never re-evaluated as eligible -- once a terminal transition has committed, this spec always returns ineligible for that same booking, consistent with the Booking entity's Contention resolution (reject-with-refresh; the first committed state transition wins).
- A Pending Payment or Expired (unpaid) booking is ineligible -- these states represent a booking that never became a confirmed appointment for the client to cancel or reschedule.

## Edge Cases

- **A booking sits exactly at the cutoff moment when eligibility is checked** -- Treated as outside the window (per the inclusive-boundary rule); the countdown display reads "0 hours" or the equivalent boundary phrasing, and the outside-window outcome branch applies if the client acts in that same instant.
- **A booking's state changes to Completed between FEAT-10.SPEC-001/002/003 loading and the client's confirm tap** -- The eligibility re-check at commit time (FEAT-10.SPEC-004) catches this; the commit is rejected and the client is shown the current, now-ineligible state, per the Booking entity's Contention resolution.
- **A client reschedules, is shown the outcome, and then reloads the same screen before confirming** -- Eligibility and the countdown are recomputed fresh on reload; a countdown that has since crossed the cutoff is reflected accurately rather than showing a stale value.
- **The booking's bound policy version cannot be read (an extremely rare data inconsistency, per FEAT-09.SPEC-004's own edge case)** -- Eligibility can still be determined from state alone, but the countdown cannot be computed; the screen shows the booking as eligible with the countdown display "Cancellation window: pending" until the read succeeds, rather than blocking the whole screen.
- **A booking is in Awaiting Outcome state (its appointment time has passed but it has not yet been marked Completed or No-Show)** -- Still eligible under this spec's rule (only Completed and No-Show gate eligibility per XBR-12); in practice the window has already closed by this point, so the inside-window outcome branch applies.

## Acceptance Criteria

**FEAT-10.SPEC-005-AC-01:** Given Riley's booking is in Confirmed state and the current moment is before the computed cutoff, when eligibility is checked, then the booking is eligible and window_state is "outside."

**FEAT-10.SPEC-005-AC-02:** Given Riley's booking is in Confirmed state and the current moment is after the computed cutoff, when eligibility is checked, then the booking is eligible and window_state is "inside."

**FEAT-10.SPEC-005-AC-03:** Given Riley's booking is in Completed state, when eligibility is checked, then the booking is ineligible and the message "This booking can no longer be cancelled or rescheduled." is shown.

**FEAT-10.SPEC-005-AC-04:** Given Riley's booking is in No-Show state, when eligibility is checked, then the booking is ineligible with the same message.

**FEAT-10.SPEC-005-AC-05:** Given Riley's booking is already Cancelled by Client, Cancelled by Pro, or Rescheduled, when eligibility is checked, then the booking is ineligible.

**FEAT-10.SPEC-005-AC-06:** Given Riley's booking is in Awaiting Outcome state, when eligibility is checked, then the booking is still eligible under this spec's rule, with window_state "inside" in practice since the appointment time has passed.

**FEAT-10.SPEC-005-AC-07:** Given Riley's booking reaches exactly the cutoff moment, when the window comparison runs, then it is treated as outside the window.

**FEAT-10.SPEC-005-AC-08:** Given Talia (the Pro) has no path into this spec, when she looks for a way to check a client's booking eligibility through it, then none exists -- her own eligibility is governed separately by FEAT-30.

**FEAT-10.SPEC-005-AC-09:** Given Support opens a Pro's account in the read-only support view, when Support looks for an entry point into this spec, then none exists, consistent with scope-boundaries SC-05.

**FEAT-10.SPEC-005-AC-10:** Given Riley looks for an override of an ineligible result, when she inspects the Cancel or Reschedule screens, then no override control is shown -- only the ineligibility message.

**FEAT-10.SPEC-005-AC-11:** Given a booking's state transitions to Completed between screen load and Riley's confirm tap, when the commit is attempted, then the re-check at commit time rejects it and the current ineligible state is shown.

**FEAT-10.SPEC-005-AC-12:** Given Riley's booking's bound policy version cannot be read at the moment of eligibility check, when the countdown is computed, then eligibility is still determined from state alone and the countdown shows "Cancellation window: pending" rather than blocking the screen.

**FEAT-10.SPEC-005-AC-13:** Given Riley is evaluating a prospective reschedule on FEAT-10.SPEC-003, when the window comparison runs, then it compares against the booking's *original* start_time, not the newly chosen time.

**FEAT-10.SPEC-005-AC-14:** Given Riley's booking is in Pending Payment or Expired (unpaid) state, when eligibility is checked, then the booking is ineligible, since it never became a confirmed appointment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

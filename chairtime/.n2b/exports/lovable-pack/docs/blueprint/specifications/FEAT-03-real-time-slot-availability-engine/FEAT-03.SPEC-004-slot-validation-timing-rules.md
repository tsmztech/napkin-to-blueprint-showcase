---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-004
spec_name: Slot Validation & Timing Rules
spec_slug: slot-validation-timing-rules
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 11
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Slot Validation & Timing Rules

## Overview

**Name:** Slot Validation & Timing Rules
**ID:** FEAT-03.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines what makes any candidate time slot offerable -- the duration-plus-buffer fit, minimum notice, booking horizon, the Pro-only exception to notice and horizon, and the rule that every slot is always computed and labeled in the Pro's timezone.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine
**Governed Entity:** Candidate Slot -- a derived, non-persisted value computed fresh at every request (never stored) from the Service and Availability Rule entities in the Feature Dependency Map; this spec governs the rules that decide whether one such computed value may ever be offered.

## Scope and Non-Goals

**In Scope:**
- The duration-plus-buffer fit rule that decides whether a candidate start time is wide enough to hold a service
- The minimum-notice and booking-horizon rules that bound how soon or how far ahead a slot may be offered
- The Pro-only exception that lets the Pro book inside notice or beyond horizon when booking a client in or rescheduling at the chair (FEAT-30)
- The rule that every slot is always computed and displayed in the Pro's account timezone, labeled as such, regardless of the client's own timezone
- Authorization for who may see and who may override each of these rules

**Non-Goals:**
- Combining these rules with Booking, Time Block, Recurring Series, and calendar busy-time occupancy into the actual computed list -- owned by FEAT-03.SPEC-001, which applies (never re-derives) these rules
- Setting the actual values of minimum_booking_notice, default_buffer, and booking_horizon -- these are Pro-configured fields on the Availability Rule entity, owned and written by FEAT-02 (Availability & Working Hours Setup); this spec governs how those values are applied to a candidate slot, not where they come from
- Resolving which of two contending clients wins a slot that passes these rules -- owned by FEAT-03.SPEC-005
- Timezone and currency account-level ownership -- owned by FEAT-27 (Pro Profile & Booking Page Settings) per XBR-25; this spec applies the Pro's already-set timezone to slot display, it does not let anyone edit it

## Governed Entity

**Entity:** Candidate Slot (derived), drawing its governing fields from Service and Availability Rule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| service_duration | number (minutes) | From Service.duration -- the length of time the candidate slot must hold |
| buffer_minutes | number (minutes) | From Service.buffer_override if set, otherwise Availability Rule.default_buffer -- the gap required around the service |
| candidate_start_time | date/time | The specific start time being evaluated, always interpreted and displayed in the Pro Account's timezone |
| minimum_booking_notice | number (hours/days, 0 to 7 days) | From Availability Rule.minimum_booking_notice -- how close to the current moment a client-facing candidate may start |
| booking_horizon | number (weeks/months, 1 week to 12 months) | From Availability Rule.booking_horizon -- how far ahead a client-facing candidate may start |
| requesting_actor | enum (Client, Pro) | Derived from which spec is requesting slot validation -- determines whether the notice/horizon exception applies |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-03.SPEC-001 | Slot Availability Computation | On every computation of the open-slot list, for every candidate start time |
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | Re-validated at the instant a Slot Hold is created, before the hold is written |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Re-validated at the instant a Pro-created deposit-request hold is created, applying the Pro-only exception |
| FEAT-05 | Public Booking Page & Booking Flow | Displays only slots that already passed these rules via FEAT-03.SPEC-001 |
| FEAT-30 | Pro Booking Management | Applies the Pro-only notice/horizon exception when the Pro books or reschedules a client at the chair |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| service_duration + buffer_minutes | The full contiguous span (service duration plus buffer) must fit entirely inside one open Availability Rule working window with no conflicting occupancy | Always | On every computation and every hold-creation re-validation | Slot is simply excluded from the list -- no client-facing error, since a non-fitting candidate is never shown as a choice | Yes |
| candidate_start_time | Must be no closer to the current moment than minimum_booking_notice | Requesting actor is Client | On every computation and every hold-creation re-validation | Slot excluded from the client-facing list; no separate error is shown because the client never sees the excluded candidate as an option | Yes |
| candidate_start_time | Must be no farther from the current moment than booking_horizon | Requesting actor is Client | On every computation and every hold-creation re-validation | Slot excluded from the client-facing list, same as above | Yes |
| candidate_start_time | No validation beyond data type when the requesting actor is the Pro booking a client in or rescheduling at the chair (FEAT-30) -- minimum_booking_notice and booking_horizon do not apply | Requesting actor is Pro | On every Pro-side booking or reschedule request | -- | No |
| candidate_start_time (display) | Always computed and labeled in the Pro Account's timezone | Always | On every display of a computed or held slot | -- (the display itself carries the timezone label, e.g., "2:00 PM Pacific Time") | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Notice/horizon exception scoping | requesting_actor, minimum_booking_notice, booking_horizon | The notice and horizon rules apply only when requesting_actor is Client; when requesting_actor is Pro, both rules are skipped entirely for that request | -- (no error; the exception is silent and automatic) |
| Buffer source precedence | service_duration, buffer_minutes | buffer_minutes always resolves from Service.buffer_override when present; only falls back to Availability Rule.default_buffer when no override exists -- the two are never summed | -- |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View a computed slot's time | The Pro (Talia), the Client (Riley) | Always, on their own respective pages | -- |
| Book inside minimum_booking_notice or beyond booking_horizon | The Pro (Talia) | Only when booking a client in or rescheduling at the chair (FEAT-30) | -- |
| Book inside minimum_booking_notice or beyond booking_horizon | The Client (Riley) | Never | The candidate is simply never offered as a choice; no separate denial dialog exists because the client never sees an excluded candidate |
| View the underlying Availability Rule values (minimum_booking_notice, default_buffer, booking_horizon) | The Pro (Talia) | Always, on their own setup screen (FEAT-02) | -- |
| View the underlying Availability Rule values | The Client (Riley) | Never | Clients see only the resulting open times, never the rule itself, per the dependency map's Availability Rule Data Sensitivity note |
| View the underlying Availability Rule values | Platform Operator (Support) | View-only, for troubleshooting a specific Pro's reported conflict | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|----------------------|
| buffer_minutes | Service.buffer_override if set, otherwise Availability Rule.default_buffer | On every slot validation | Yes -- the Pro sets buffer_override per service via FEAT-02's per-service screen; the Client never overrides it |
| candidate_start_time display timezone | Always the Pro Account's account timezone (XBR-25) | On every display | No -- never derived from or overridden by the Client's device timezone |
| minimum_booking_notice enforcement | Skipped when requesting_actor is Pro | On every Pro-side request | No -- this is a structural exception, not a per-request choice |
| booking_horizon enforcement | Skipped when requesting_actor is Pro | On every Pro-side request | No -- same as above |

## Business Rules

- A slot is only offered if the full service duration plus buffer fits entirely within an open working window with no conflicting Booking, Time Block, Recurring Series occurrence, active Slot Hold, or calendar busy time (XBR-01) -- FEAT-03.SPEC-001 applies this rule during computation.
- Minimum booking notice and booking horizon limit every client-facing booking path; the Pro alone may book inside notice or beyond horizon when booking a client in or rescheduling (XBR-03) -- this is a cross-feature rule this spec elaborates as FEAT-02's authority (notice and horizon settings), not this feature's own.
- Slot times are always computed and shown in the Pro's timezone, labeled as such, regardless of the client's own timezone or device settings (XBR-25) -- currency and timezone are per-account settings FEAT-27 owns; this spec applies the timezone, it does not set it.
- The duration+buffer fit, notice, and horizon rules are defined once here and referenced -- never re-derived -- by FEAT-03.SPEC-001 (computation) and FEAT-03.SPEC-002/FEAT-03.SPEC-007 (hold creation re-validation), per this Brief's Shared Validation note.

## Edge Cases

- **Candidate slot's contiguous span fits exactly to the minute (service duration plus buffer equals the remaining open window with zero slack)** -- Passes validation; the fit rule requires the span to fit entirely inside the window, and an exact fit satisfies "entirely inside."
- **Candidate start time falls exactly at the minimum_booking_notice boundary** -- Passes validation; the rule excludes only candidates closer than the notice threshold, so the boundary instant itself is offerable.
- **Candidate start time falls exactly at the booking_horizon boundary** -- Passes validation for the same reason; only candidates strictly beyond the horizon are excluded.
- **The Pro books a client in for a time that would fail the fit rule (duration+buffer does not fit)** -- The fit rule is never exempted for the Pro -- only notice and horizon carry the Pro-only exception; a physically non-fitting slot is refused for the Pro exactly as for a client, since accepting it would create an actual scheduling conflict.
- **A client's device reports a different local time than the Pro's account timezone** -- The displayed slot time is unaffected; it is always computed and labeled in the Pro's timezone, and the client's device timezone plays no role in either computation or display.
- **Availability Rule is edited mid-session (minimum_booking_notice or booking_horizon changes) while a client is viewing an already-computed list** -- The next computation applies the new values; a candidate already shown to the client that no longer passes is excluded from the next refresh, with a refreshed list shown per FEAT-03.SPEC-001, not a silent stale display.

## Acceptance Criteria

**FEAT-03.SPEC-004-AC-01:** Given a candidate start time where the service duration plus buffer fits entirely inside Talia's open working window with no conflicting occupancy, when the fit rule is evaluated, then the candidate passes.

**FEAT-03.SPEC-004-AC-02:** Given a candidate start time where the service duration plus buffer does not fit entirely inside an open window, when the fit rule is evaluated, then the candidate is excluded, with no separate client-facing error shown.

**FEAT-03.SPEC-004-AC-03:** Given Riley requests slots and a candidate falls closer to now than Talia's minimum_booking_notice, when the notice rule is evaluated, then the candidate is excluded from Riley's list.

**FEAT-03.SPEC-004-AC-04:** Given Riley requests slots and a candidate falls beyond Talia's booking_horizon, when the horizon rule is evaluated, then the candidate is excluded from Riley's list.

**FEAT-03.SPEC-004-AC-05:** Given Talia (the Pro) is booking a client in at the chair for a time inside her own minimum_booking_notice, when the notice rule is evaluated for her request, then the candidate is not excluded on notice grounds.

**FEAT-03.SPEC-004-AC-06:** Given Talia is booking a client in for a time beyond her own booking_horizon, when the horizon rule is evaluated for her request, then the candidate is not excluded on horizon grounds.

**FEAT-03.SPEC-004-AC-07:** Given Talia attempts to book a client in for a time where the service duration plus buffer does not physically fit her open hours, when the fit rule is evaluated, then the candidate is still excluded -- the Pro-only exception does not extend to the fit rule.

**FEAT-03.SPEC-004-AC-08:** Given a candidate start time falls exactly at the minimum_booking_notice boundary, when the notice rule is evaluated, then the candidate passes.

**FEAT-03.SPEC-004-AC-09:** Given a candidate start time falls exactly at the booking_horizon boundary, when the horizon rule is evaluated, then the candidate passes.

**FEAT-03.SPEC-004-AC-10:** Given Riley is browsing Talia's booking page from a different timezone than Talia's account timezone, when a slot is displayed, then the time shown is computed and labeled in Talia's account timezone, not Riley's device timezone.

**FEAT-03.SPEC-004-AC-11:** Given a Service has no buffer_override set, when buffer_minutes is resolved for a candidate slot, then Availability Rule's default_buffer applies.

**FEAT-03.SPEC-004-AC-12:** Given a Service has a buffer_override set, when buffer_minutes is resolved for a candidate slot, then the override applies instead of the default_buffer, never both summed.

**FEAT-03.SPEC-004-AC-13:** Given Riley attempts to view the underlying Availability Rule values (working hours, buffer, notice, horizon) directly, when Riley's booking page renders, then only the resulting open times are shown, never the rule itself.

**FEAT-03.SPEC-004-AC-14:** Given Talia edits her minimum_booking_notice mid-session while Riley is viewing an already-computed slot list, when Riley's list next refreshes, then any candidate that no longer passes the new notice value is excluded, and Riley sees the refreshed list rather than a stale one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

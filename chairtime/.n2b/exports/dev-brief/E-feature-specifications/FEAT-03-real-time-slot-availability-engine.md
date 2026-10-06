# FEAT-03 — Real-Time Slot Availability Engine

This chapter covers Real-Time Slot Availability Engine (FEAT-03), a Core-tier feature. It carries 7 specifications carrying 78 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-03.SPEC-001 | Slot Availability Computation | automation | 12 |
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | automation | 10 |
| FEAT-03.SPEC-003 | Slot Hold Expiration | automation | 8 |
| FEAT-03.SPEC-004 | Slot Validation & Timing Rules | logic-rule | 14 |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | logic-rule | 11 |
| FEAT-03.SPEC-006 | Calendar Busy-Time Consumption & Degraded Mode | integration | 11 |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | automation | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Real-Time Slot Availability Engine

## Summary

**Feature:** Real-Time Slot Availability Engine
**ID:** FEAT-03
**Description:** The system that computes, at the moment a client is looking, exactly which time slots are genuinely free -- combining the Pro's working hours, buffer time, existing Chairtime bookings, manual time blocks, and busy times from the Pro's connected personal calendar -- so a client can never select a time that is not truly open.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Vision and Success Criteria are explicit: "a genuinely free time," and "nobody has ever had a double booking." This is the mechanism that makes that promise true; every other feature that touches time depends on it.

**Key Capabilities:**
- Compute the live set of open slots for a given service and date range
- Reserve a slot the instant a client begins paying, preventing a second client from grabbing it mid-checkout
- Release a held-but-unpaid slot automatically after a short timeout if payment is not completed

This is a computation engine, not a user-facing surface in its own right: it produces slot truth and slot holds that other features (chiefly FEAT-05) render and act on. Its Access field, States, Validation & Limits, and Signals fields (product-features.md) are elaborated below into specs; none are re-derived.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-03.SPEC-001 | Slot Availability Computation | Automation | All | Computes the live set of genuinely open slots for a service and date range from working hours, buffer, bookings, blocks, recurring reservations, and calendar busy time |
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | Automation | All | Reserves a slot the instant a client begins paying so no other client can take it mid-checkout |
| FEAT-03.SPEC-003 | Slot Hold Expiration | Automation | All | Automatically releases a held-but-unpaid slot back to public availability after a short fixed timeout |
| FEAT-03.SPEC-004 | Slot Validation & Timing Rules | Logic/Rule | All | Governs what makes a slot offerable: duration-plus-buffer fit, minimum notice, booking horizon, Pro-only exceptions, and Pro-timezone labeling |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | Logic/Rule | All | Governs how a contested slot (two clients, or a client vs. a Pro-side change) resolves: first committed action wins, the loser sees a plain re-pick message |
| FEAT-03.SPEC-006 | Calendar Busy-Time Consumption & Degraded Mode | Integration | All | Consumes busy/free periods from the Pro's connected personal calendar into slot computation and defines the fallback behavior when sync is unavailable |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Automation | All | Reserves a slot the instant the Pro books a client in with a deposit request through FEAT-30, holding it up to 24 hours or until 2 hours before the appointment (whichever comes first), and expiring the booking with Pro notification if the deposit is never paid |

Zero screen and zero notification specs are legal counts here: this feature owns no screen of its own (the slot list is rendered by FEAT-05's booking page) and triggers no messages of its own (Communications field: N/A).

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Compute the live set of open slots for a given service and date range | FEAT-03.SPEC-001 | Core computation combining Availability Rule, Booking, Time Block, Calendar Connection busy time, Recurring Series, and Service duration/buffer | Phase 2 (Explicit) |
| Reserve a slot the instant a client begins paying | FEAT-03.SPEC-002 | Creates a time-limited Slot Hold at the start of checkout, immediately excluding the slot from other clients' computed lists | Phase 2 (Explicit) |
| Release a held-but-unpaid slot after a short timeout | FEAT-03.SPEC-003 | Automatically expires the Slot Hold and returns the slot to FEAT-03.SPEC-001's computed output | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-03.SPEC-004 | Slot Validation & Timing Rules | Phase 5 (Rule-Constraint Discovery) | The feature's Validation & Limits field names four distinct governing rules (duration+buffer fit, minimum notice, booking horizon, Pro-timezone labeling); XBR-03 and XBR-25 apply these rules across five and eight other features respectively, meeting the "rules shared across multiple screens/automations" and "conditional logic" thresholds for a standalone spec |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | Phase 4 (Trigger-Response) / Phase 5 (Rule-Constraint Discovery) | Two Alternates in the feature's own Primary Flows & Alternates field, two journey Failure/Recovery Variants, and cross-feature rule XBR-01 all describe the same conditional-logic pattern (first committed action wins, loser is refreshed); this recurs across FEAT-03, FEAT-05, FEAT-07, FEAT-10, FEAT-20, FEAT-21, and FEAT-30, crossing the standalone threshold |
| FEAT-03.SPEC-006 | Calendar Busy-Time Consumption & Degraded Mode | Phase 4 (External Dependencies lens) | ASMP-33 names calendar-sync as a category-level external dependency this feature relies on; the External Touchpoints row in the dependency map explicitly assigns FEAT-03 (not FEAT-04, which owns the connection) the job of specifying how it consumes busy periods and degrades when sync lapses (XBR-13) |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Phase 5 (Rule-Constraint Discovery) / Phase 6 (Failure-Mode Analysis) | XBR-02 (this feature's own authority rule) names a second, longer-lived hold class -- a Pro-created deposit request holding its slot up to 24 hours or until 2 hours before the appointment -- distinct from the checkout hold SPEC-002/SPEC-003 already model; the journey *Talia's Between-Clients Day* (Owning Persona: Talia; Coverage: Regular), Step 3, walks exactly this path (Talia books the client's next fill and the client pays a scanned deposit link before leaving), and its unpaid-outcome failure mode (the client never pays) has no covering spec without this one |

## Entity-Lifecycle Coverage Matrix

**Entity: Slot Hold**

<!-- Slot Hold is not listed as a Shared Data Entity in the dependency map slice because it is wholly internal to this feature (ephemeral, never read by another feature's spec) -- but the feature's own Key Capabilities describe its full lifecycle (create-on-checkout-start, expire-on-timeout), so Phase 3 requires it be modeled here. -->

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-03.SPEC-002 / FEAT-03.SPEC-007 | Created the instant a client begins paying for a specific slot (SPEC-002), or the instant the Pro books a client in with a deposit request through FEAT-30 (SPEC-007) | Two creation paths, one Slot Hold shape: SPEC-007's hold carries a longer, variable expiry (up to 24 hours, capped at 2 hours before the appointment) instead of SPEC-002's fixed few-minute checkout timeout |
| Read (single) | FEAT-03.SPEC-002 / FEAT-03.SPEC-003 / FEAT-03.SPEC-007 | Checked before creating a new hold on a candidate slot (SPEC-002/SPEC-007) and checked when evaluating a specific hold's timeout (SPEC-003/SPEC-007) | -- |
| Read (list) | FEAT-03.SPEC-001 | Current active holds -- checkout holds and Pro-created deposit-request holds alike -- are read whenever the open-slot list is computed, so held slots are excluded from what clients see | -- |
| Update | FEAT-03.SPEC-003 / FEAT-03.SPEC-007 | Transitioned to Expired when its timeout elapses without completed payment (SPEC-003 for checkout holds, SPEC-007 for Pro-created deposit-request holds) | Also transitioned to Consumed when FEAT-07 completes the client's payment (cross-feature trigger; FEAT-07 owns payment completion) |
| Delete/Archive | FEAT-03.SPEC-003 / FEAT-03.SPEC-007 | Hard delete on expiration (either hold type) or on conversion to a confirmed Booking (cross-feature) -- no restore path (a released hold simply becomes an ordinary open slot again; there is nothing a user restores), no cascade beyond marking the associated Booking Expired and notifying the Pro (SPEC-007), and no retention/purge policy is needed: this is a transient computation artifact with a lifetime of minutes to at most 24 hours, never a historical record -- recorded here as an explicit non-goal, not an omission | -- |
| State Transition | FEAT-03.SPEC-002 / FEAT-03.SPEC-003 / FEAT-03.SPEC-007 | Active (SPEC-002 or SPEC-007) -> Expired (SPEC-003 or SPEC-007) or Active -> Consumed (cross-feature, on payment completion) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-03.SPEC-001 | Existing confirmed/pending bookings occupy time and must be excluded from computed availability |
| Availability Rule | FEAT-03.SPEC-001, FEAT-03.SPEC-004 | Working-hours windows, default/per-service buffer, minimum notice, and booking horizon are read to bound and shape the computed slots |
| Time Block | FEAT-03.SPEC-001 | Manual blocks (including recurring ones) remove availability from the computed slot list |
| Calendar Connection | FEAT-03.SPEC-006 (feeds FEAT-03.SPEC-001) | Busy/free periods from the Pro's connected personal calendar are consumed as an additional availability constraint; connection health drives the degraded-mode behavior |
| Recurring Series | FEAT-03.SPEC-001 | Generated future occurrences reserve their matching slots so they are not offered to other clients |
| Service | FEAT-03.SPEC-001, FEAT-03.SPEC-004 | Service duration and any buffer override determine how much contiguous open time a slot requires |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client selects a service and date range | System computes the live open-slot list from rules, bookings, blocks, calendar busy time, and recurring reservations | Standalone Automation | FEAT-03.SPEC-001 |
| Client begins the payment step for a chosen slot | System creates a Slot Hold, instantly removing that slot from every other client's computed list | Standalone Automation | FEAT-03.SPEC-002 |
| A Slot Hold reaches its fixed timeout without completed payment | System expires and deletes the hold; the slot reappears in the computed list | Standalone Automation | FEAT-03.SPEC-003 |
| Two clients attempt to hold or pay for the same slot concurrently | First to complete payment wins; the other's attempt is refused with a plain "just taken" message and a refreshed live list | Standalone Logic/Rule | FEAT-03.SPEC-005 |
| A candidate slot falls inside the Pro's minimum notice or beyond their booking horizon | Slot is excluded from the client-facing computed list (Pro-side booking is exempt) | Standalone Logic/Rule | FEAT-03.SPEC-004 |
| Any computed or held slot is displayed | Time is always computed and labeled in the Pro's timezone, regardless of the client's own timezone | Standalone Logic/Rule | FEAT-03.SPEC-004 |
| The Pro's connected personal calendar sync becomes unavailable | Engine falls back to Chairtime-only data and flags reduced confidence to the Pro only, never the client | Standalone Integration | FEAT-03.SPEC-006 |
| A Booking is cancelled, a Time Block is removed, or hours are changed | The freed time becomes bookable again the next time the list is computed | Inline in FEAT-03.SPEC-001 (no separate automation: computation is always live, never cached) | FEAT-03.SPEC-001 |
| A freed slot enters the waitlist's 30-minute priority window | Underlying free/not-free slot truth is unchanged; the priority gate itself is owned elsewhere | Cross-feature -- logged in touchpoints | FEAT-20 responsibility (XBR-28, XBR-02): the 30-minute claim window is a timing rule FEAT-20 owns and applies; FEAT-03 supplies the same generic hold/release mechanism FEAT-20 invokes to enforce it |
| Payment completes for a held slot | Hold is converted and a confirmed Booking is created | Cross-feature -- logged in touchpoints | FEAT-05 / FEAT-07 responsibility |
| The Pro books a client in and creates a deposit request (FEAT-30) | System creates a Slot Hold on the chosen slot, instantly removing it from every client's computed list, with an expiry up to 24 hours out or 2 hours before the appointment, whichever comes first | Standalone Automation | FEAT-03.SPEC-007 |
| A Pro-created deposit-request hold reaches its timeout without completed payment | System expires and deletes the hold, marks the associated Booking Expired, and the slot reappears in the computed list; the Pro is notified the deposit was never paid via FEAT-30 (dashboard) and FEAT-08 (message) | Standalone Automation | FEAT-03.SPEC-007 |
| An unpaid recurring occurrence reaches its cancellation cut-off | Underlying free/not-free slot truth is unchanged; the release timing itself is owned elsewhere | Cross-feature -- logged in touchpoints | FEAT-21 responsibility (XBR-02): the cancellation-cut-off release point is a timing rule FEAT-21 owns and applies; FEAT-03 supplies the same generic hold/release mechanism FEAT-21 invokes to enforce it |

## Shared Context

**Shared Entities:**
- Slot Hold -- created by SPEC-002 (checkout) or SPEC-007 (Pro-created deposit request), read by SPEC-001 (to exclude held slots) and SPEC-002/SPEC-003/SPEC-007 (to check a specific slot's hold), updated and deleted by SPEC-003 or SPEC-007 (or, on payment completion, by the cross-feature trigger from FEAT-07). Fields (functional): held slot's service, start time and duration, the client's in-progress checkout or the Pro-created deposit request that owns it, hold-created timestamp, expiry timestamp (fixed few-minute timeout for a checkout hold; up to 24 hours or 2 hours before the appointment, whichever comes first, for a Pro-created deposit-request hold), state (Active | Expired | Consumed).
- Availability Rule, Time Block, Calendar Connection, Recurring Series, Booking, Service -- all read-only inputs to SPEC-001's computation; none are created, updated, or deleted by this feature (see Referenced Entities above).
- XBR-02 elaboration -- this cross-feature rule names four time-limited hold/release points; FEAT-03 (its authority) elaborates two directly as Slot Hold specs (SPEC-002/SPEC-003 for the checkout hold, SPEC-007 for the Pro-created deposit-request hold) and supplies the same generic hold/release mechanism to the other two, whose actual timing threshold is a decision each owning feature makes and applies: the waitlist claim window (30 minutes, owned by FEAT-20) and the unpaid recurring-occurrence release (at its cancellation cut-off, owned by FEAT-21). FEAT-03 does not model those two features' timing rules as its own specs.

**Shared UI Patterns:**
- N/A -- this feature owns no Screen specs. The slot-list rendering, its Empty/Loading/Error/Offline-degraded presentation, and the "just taken" / "hold expired" messaging described in this feature's States and Primary Flows & Alternates fields are contract requirements this engine's output must support; the presentation itself belongs to FEAT-05's Screen specs, which SPEC-001 and SPEC-005 must supply enough state and messaging detail to satisfy.

**Shared Validation:**
- FEAT-03.SPEC-004 defines the timing and fit rules (duration+buffer, notice, horizon, timezone). SPEC-001 and SPEC-002 both reference SPEC-004 rather than re-deriving these rules: SPEC-001 uses them to decide what to include in the computed list, and SPEC-002 re-validates them at the instant a hold is created (a slot computed a moment earlier must still pass the same rules when checkout begins).
- FEAT-03.SPEC-005 defines the contention tie-break rule. SPEC-002 and SPEC-003 both reference SPEC-005 rather than each implementing their own conflict handling.

## Internal Dependency Map

```
SPEC-001 (Slot Availability Computation) -> [reads busy periods from] -> SPEC-006 (Calendar Busy-Time Consumption & Degraded Mode)
SPEC-001 (Slot Availability Computation) -> [validates candidate slots against] -> SPEC-004 (Slot Validation & Timing Rules)
SPEC-001 (Slot Availability Computation) -> [excludes slots held by] -> SPEC-002 (Slot Hold Creation & Checkout Reservation)
SPEC-002 (Slot Hold Creation & Checkout Reservation) -> [client begins payment] -> creates a Slot Hold -> [excluded from next computation by] -> SPEC-001
SPEC-002 (Slot Hold Creation & Checkout Reservation) -> [re-validates against] -> SPEC-004 (Slot Validation & Timing Rules)
SPEC-002 (Slot Hold Creation & Checkout Reservation) -> [on contested slot] -> SPEC-005 (Slot Contention Resolution Rules)
SPEC-003 (Slot Hold Expiration) -> [timeout reached] -> expires/deletes the Slot Hold -> [slot reappears via] -> SPEC-001
SPEC-003 (Slot Hold Expiration) -> [governed by] -> SPEC-005 (Slot Contention Resolution Rules)
SPEC-006 (Calendar Busy-Time Consumption & Degraded Mode) -> [sync unavailable] -> [flags reduced confidence consumed by] -> SPEC-001
SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -> [Pro books a client in via FEAT-30] -> creates a Slot Hold -> [excluded from next computation by] -> SPEC-001
SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -> [re-validates against] -> SPEC-004 (Slot Validation & Timing Rules)
SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -> [on contested slot] -> SPEC-005 (Slot Contention Resolution Rules)
SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -> [timeout reached (24h cap, or 2h before appointment, whichever first)] -> expires/deletes the Slot Hold, marks the Booking Expired -> [slot reappears via] -> SPEC-001; Pro notified via FEAT-30 and FEAT-08
```

**Default Entry:** N/A -- this feature has no navigable entry point of its own. Its functional entry point is invocation by FEAT-05 (Public Booking Page & Booking Flow) whenever a client selects a service; SPEC-001 is the spec that responds to that invocation.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-03.SPEC-001 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Live slot list is rendered on the client-facing booking page | Client picks a service |
| FEAT-03.SPEC-001 | Inbound | FEAT-02 (Availability & Working Hours Setup) | Reads working hours, buffer, minimum notice, and booking horizon settings | Pro edits availability; a new rule version takes effect |
| FEAT-03.SPEC-001 | Inbound | FEAT-17 (Manual Time Blocking) | Reads manual time blocks to remove them from computed availability | Pro creates or edits a time block |
| FEAT-03.SPEC-001 | Inbound | FEAT-21 (Recurring/Standing Appointments) | Reads generated recurring occurrences to reserve their matching future slots | Recurring series generates or changes occurrences |
| FEAT-03.SPEC-006 | Inbound | FEAT-04 (Two-Way Calendar Sync) | Consumes busy/free periods from the Pro's connected personal calendar; the connection itself is owned by FEAT-04 | Calendar sync completes, or health degrades/lapses |
| FEAT-03.SPEC-002 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) | Slot Hold created the instant a client begins paying | Client begins checkout |
| FEAT-03.SPEC-003 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) | Expired hold returns the client to the live slot list with an explanatory message | Hold timeout reached before payment completes |
| FEAT-03.SPEC-001 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | Live slot list surfaced again when the client picks a new time | Client initiates a reschedule |
| FEAT-03.SPEC-005 | Outbound | FEAT-20 (Waitlist for Cancelled Slots) | Supplies the underlying slot free/not-free truth that the waitlist's 30-minute priority window builds on; the 30-minute timing rule itself is FEAT-20's, not this feature's | Slot is freed by a cancellation |
| FEAT-03.SPEC-001 | Outbound | FEAT-30 (Pro Booking Management) | Slot truth (with the Pro's notice/horizon exemption) consumed when the Pro books a client in or reschedules at the chair | Pro creates or reschedules a booking |
| FEAT-03.SPEC-007 | Inbound | FEAT-30 (Pro Booking Management) | Slot Hold created the instant the Pro books a client in with a deposit request at the chair | Pro creates a deposit request for a client booked in |
| FEAT-03.SPEC-007 | Outbound | FEAT-30 (Pro Booking Management) / FEAT-08 (Automated Booking Messaging) | Expired Pro-created deposit-request hold marks the Booking Expired on the Pro's dashboard and notifies the Pro the deposit was never paid | Hold timeout reached (24h cap, or 2h before appointment, whichever first) before payment completes |
| FEAT-03.SPEC-001 / FEAT-03.SPEC-007 | Outbound | FEAT-21 (Recurring/Standing Appointments) | Supplies the underlying slot free/not-free truth and the generic hold/release mechanism that an unpaid recurring occurrence's release at its cancellation cut-off builds on; that cut-off timing rule itself is FEAT-21's, not this feature's | An unpaid recurring occurrence reaches its cancellation cut-off |

## Non-Functional Notes

**Data volumes / growth:** A few hundred pros in year one, each with roughly 100-500 clients and 20-40 bookings a week, with responsiveness held steady as each Pro's booking, block, and recurring-series history accumulates over multiple years (assumptions-constraints.md ASMP-22). Computation must stay equally fast as the volume of historical Bookings it must exclude from availability grows.

**Responsiveness:** Available slots for a chosen service appear within roughly one second of selection, and the list updates within roughly one second of a slot being taken by another client (success-metrics.md, Slot Search Responsiveness; assumptions-constraints.md ASMP-21). This is the product's most frequent and time-sensitive interaction loop, since the under-one-minute booking promise depends on availability never becoming a visible wait.

**Data sensitivity / privacy:** The computation itself reads Booking data, which is personal data linked to an identifiable client (appointment time, service); this feature's output to a client, however, is deliberately stripped to bare availability (open/not-open), never another client's identity or booking detail, and never another Pro's schedule at all (user-persona.md Access Matrix; dependency map, Booking Data Sensitivity line). Availability Rule and Time Block data are low-sensitivity to the Pro but Time Block labels are private and never surfaced through this engine's output.

**Compliance flags:** N/A -- no named compliance regime (health, financial, or otherwise) attaches specifically to slot computation; it inherits the general privacy handling of the Booking data it reads but introduces no new regulated data category of its own (assumptions-constraints.md).

## Non-Goals

- **Multi-staff or multi-chair slot pooling** -- Excluded per scope-boundaries.md (SC-01): BRIEF.md's Constraints state the product is "strictly single-operator for v1... probably forever," so this engine computes availability for exactly one Pro's single calendar; there is no cross-staff pooling, chair assignment, or capacity-splitting logic of any kind.
- **Cross-pro availability views** -- Excluded per scope-boundaries.md (SC-03): BRIEF.md's Constraints state a client's data is "never visible to any other pro or client," so every computation is scoped to exactly one Pro's public page; the engine never produces a merged, comparative, or marketplace-style view of slots across multiple Pros.
- **Predictive or AI-recommended time suggestions** -- Excluded by adjacency analysis: the feature's own Description and Data Notes define this engine as a deterministic, entirely-derived free/busy computation ("nothing here is directly entered by a user"; every slot shown is computed live from existing rules and records) in service of the correctness bar in scope-boundaries.md (SC-21, "never silently double-book"); ranking or recommending "best" times is a different, optimization-oriented capability the product definition does not ask for.
- **Waitlist priority-window management** -- Intentional ownership boundary, not an omission: XBR-28 assigns ownership of the 30-minute post-cancellation priority window to FEAT-20 (Waitlist for Cancelled Slots). This feature supplies only the underlying free/not-free slot truth that the priority window is built on top of; it does not itself gate, notify, or time-box waitlisted clients.



# Automation Spec: Slot Availability Computation

## Overview

**Name:** Slot Availability Computation
**ID:** FEAT-03.SPEC-001
**Type:** Automation
**Purpose:** Computes, live and on demand, the complete set of genuinely open time slots for a chosen service and date range by combining the Pro's working hours and buffer, existing Bookings, manual Time Blocks, Recurring Series occurrences, active Slot Holds, and the Pro's connected-calendar busy time.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Computing the complete open-slot list for one service and one date range, for exactly one Pro's schedule
- Combining every constraint that removes availability: Availability Rule windows and buffer, Booking occupancy, Time Block spans, Recurring Series reserved occurrences, active Slot Holds, and Calendar Connection busy time
- Applying the timing and fit rules defined in FEAT-03.SPEC-004 (duration+buffer fit, minimum notice, booking horizon, Pro-timezone labeling) to every candidate slot
- Recomputing live on every request -- never serving a cached or stale list
- Supplying the reduced-confidence flag from FEAT-03.SPEC-006 to the Pro-facing rendering when calendar sync has lapsed

**Non-Goals:**
- Rendering the slot list on a screen -- owned by FEAT-05 (Public Booking Page & Booking Flow), which is the sole consumer of this computation's output; this spec supplies the data and state contract FEAT-05's screens must satisfy, per this Brief's Shared UI Patterns
- Multi-staff or multi-chair slot pooling -- excluded per scope-boundaries.md (SC-01): the product is "strictly single-operator for v1... probably forever," so this computation always resolves to exactly one Pro's single calendar
- Deciding the calendar-sync degraded-mode fallback logic itself -- owned by FEAT-03.SPEC-006; this spec only consumes its output (the busy periods and the confidence flag)
- Ranking, recommending, or reordering slots by any preference signal -- excluded by adjacency analysis: the feature's Description defines this as a deterministic, entirely-derived free/busy computation, not a recommendation engine

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client selects a service and (implicitly) the visible date range | FEAT-05.SPEC-002 (Slot Selection), reached when the client picks a service on FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, service selection step) | Fires every time a client opens or changes the service selection on the booking page, and every time the visible date window scrolls forward | Service ID, requested date range, the Pro Account whose page this is |
| Client's slot list is showing and a slot elsewhere is taken or freed | FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation), FEAT-03.SPEC-003 (Slot Hold Expiration), FEAT-10 (Client-Initiated Cancel/Reschedule), FEAT-30 (Pro Booking Management) | Fires on a poll/refresh cycle of roughly one second while a client is viewing the list, per the Slot Search Responsiveness target | Same as above, re-evaluated against current data |
| A Pro-side setup change takes effect | FEAT-02 (Availability & Working Hours Setup), FEAT-17 (Manual Time Blocking), FEAT-21 (Recurring/Standing Appointments) | Fires the next time any client requests the list after hours, a block, or a recurring series changes -- there is no separate recompute event because computation is always live | Current Availability Rule version, current Time Blocks, current Recurring Series occurrences |
| Reschedule flow requests a fresh list | FEAT-10.SPEC-002 (Reschedule -- Select New Time) (Client-Initiated Cancel/Reschedule) | Fires when a client picks "reschedule" for an existing booking | Service ID (same as original booking), requested date range, excluding the booking's own currently-held time from being treated as a conflict against itself |

## Processing Logic

1. Receive the requested Service ID and date range from the triggering screen, scoped to exactly one Pro Account.
2. Read the Service's duration and any buffer_override; read the current Availability Rule version's weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon.
3. Read all confirmed and pending Bookings for this Pro Account that fall within the requested date range plus the service duration on either edge.
4. Read all active Time Blocks (including recurring-pattern occurrences) for this Pro Account within the requested date range.
5. Read all future occurrences generated by any active Recurring Series for this Pro Account within the requested date range.
6. Read all active Slot Holds (checkout holds from FEAT-03.SPEC-002 and Pro-created deposit-request holds from FEAT-03.SPEC-007) for this Pro Account.
7. Read the busy periods supplied by FEAT-03.SPEC-006 for this Pro Account's connected personal calendar, and note the current confidence flag (Normal or Reduced).
8. Generate the set of candidate start times within the Availability Rule's weekly working windows for the requested date range, at the Service's duration granularity.
9. For each candidate start time, apply the validation rules from FEAT-03.SPEC-004: the full Service duration plus the applicable buffer (per-service override or default) must fit entirely inside one open working window; the candidate must be no closer than minimum_booking_notice from the current moment and no farther than booking_horizon; the candidate must not fall within a Booking, Time Block, Recurring Series occurrence, active Slot Hold, or Calendar Connection busy period.
10. Exclude every candidate that fails any check in Step 9 from the result set.
11. Label every remaining candidate's start time in the Pro Account's timezone, per FEAT-03.SPEC-004.
12. Return the resulting open-slot list to the triggering screen, tagged with the current calendar-sync confidence flag for Pro-only surfacing (never shown to the client).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Open slots computed | One or more candidates pass all checks | None -- read-only computation | Client sees the live list of open times for the chosen service | FEAT-05 (booking page slot list) |
| Fully booked | Zero candidates pass all checks within the requested range | None | Client sees a plain "fully booked, check back or view other services" message, per this feature's States field, never a blank grid | FEAT-05 |
| Reduced-confidence computation | Calendar sync (FEAT-03.SPEC-006) is currently degraded | None | Pro-only banner reflecting reduced confidence on their own dashboard view of the schedule; the client sees the ordinary computed list with no indication anything is degraded | FEAT-03.SPEC-006 (source of the flag), FEAT-12 (Pro Daily Schedule Dashboard, Pro-visible banner) |
| Computation failure | The computation cannot complete (e.g., a required input cannot be read) | None -- no partial or stale list is ever shown | Client sees a retry prompt, never a stale or incorrect slot list, per this feature's States field: "an incorrect slot is treated as worse than no slot list at all" | FEAT-05 |

## Data Model

**Reads:** Service (duration, buffer_override), Availability Rule (weekly_windows, default_buffer, minimum_booking_notice, booking_horizon, effective_from), Booking (start_time, duration, state -- excluding Cancelled/Expired states from occupancy), Time Block (start, end, recurrence), Recurring Series (generated occurrences), Slot Hold (service, start time, duration, expiry, state -- Active holds only), Calendar Connection (busy_periods, status) via FEAT-03.SPEC-006.
**Creates:** None -- this spec performs no writes.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Computation is always live, never cached: a Booking cancellation, a Time Block removal, or a working-hours change is reflected the very next time the list is computed (XBR-01), with no separate recompute automation needed.
- A slot is only offered if it passes every rule in FEAT-03.SPEC-004 (duration+buffer fit, minimum notice, booking horizon) -- this spec never re-derives those rules, it applies them.
- Every candidate slot excludes time held by an active Slot Hold of either kind (SPEC-002 or SPEC-007), so two clients (or a client and a Pro-side action) never see the same contested slot simultaneously offered as open (XBR-01, XBR-02).
- The computation is scoped to exactly one Pro Account per invocation; it never merges or compares availability across Pro Accounts (scope-boundaries.md SC-03).
- Reduced calendar-sync confidence changes nothing about which slots are computed as open -- it is a Pro-only trust signal, never a reason to withhold or alter the client-facing list (FEAT-03.SPEC-006).

## Edge Cases

- **Requested date range spans an Availability Rule version boundary** -- Each date in the range is evaluated against whichever Availability Rule version was effective for that date; a slot computed under an old version that no longer fits under the new one is simply not offered going forward.
- **Service has no buffer_override** -- The Availability Rule's default_buffer applies uniformly.
- **A Recurring Series occurrence and a one-off Booking would land on the same start time** -- This cannot occur: the Recurring Series occurrence itself reserves the slot at generation time (FEAT-21), so no second Booking can ever be created against it; the computation simply treats the occurrence as occupied.
- **Calendar Connection has never been set up** -- Busy periods contribute nothing (an empty set); computation proceeds using only Chairtime-internal data, with no reduced-confidence flag (that flag applies only to a lapsed existing connection, per FEAT-03.SPEC-006).
- **Concurrent trigger firing (two clients request the same service/date range at effectively the same time)** -- Each computation runs independently against the data visible at that instant; because Slot Holds are created synchronously by FEAT-03.SPEC-002 before either computation can return, the two results can differ only if a hold was created between the two reads, which is exactly the intended exclusion behavior, not a conflict.
- **Trigger fires while a previous computation for the same client is still in flight** -- The client-facing screen is expected to display only the most recently returned result; an in-flight computation whose result is superseded by a newer request is simply discarded when it returns, never merged with the newer one.
- **Requested date range extends beyond the booking_horizon** -- Only the portion of the range within the horizon is computed; dates beyond the horizon return no candidates for that portion, consistent with FEAT-03.SPEC-004.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Every candidate slot is validated against these fit, notice, horizon, and timezone rules |
| FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Affects (inbound) | Active checkout holds are read and excluded from the computed list |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Affects (inbound) | Active Pro-created deposit-request holds are read and excluded from the computed list |
| FEAT-03.SPEC-003 (Slot Hold Expiration) | Affects (inbound) | An expired hold's slot reappears the next time this computation runs |
| FEAT-03.SPEC-006 (Calendar Busy-Time Consumption & Degraded Mode) | Triggered by (inbound) | Supplies busy periods and the sync-confidence flag consumed here |
| FEAT-05 (Public Booking Page & Booking Flow) | Triggered by (inbound) / Affects (outbound) | The booking page invokes this computation on service selection and displays its result |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) / Affects (outbound) | Reschedule flow invokes this computation for a new time |
| FEAT-30 (Pro Booking Management) | Affects (outbound) | Slot truth, with the Pro's notice/horizon exemption, is consumed when the Pro books or reschedules a client at the chair |

## Analytics and Success Signals

- **slot_list_computed** (service_id, date_range, result_count, computation_duration_ms) -- supports success-metrics.md: "Slot Search Responsiveness"
- **slot_list_empty** (service_id, date_range) -- supports success-metrics.md: "Slot Search Responsiveness" (an empty result is still a completed, timely computation)
- **slot_conflict_prevented** (service_id, candidate_start_time) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_computation_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence" (a failed computation must never silently surface an unsafe slot list -- this event measures how often the fail-safe path is exercised)

## Acceptance Criteria

**FEAT-03.SPEC-001-AC-01:** Given Riley opens Talia's booking page and selects a service, when the slot computation runs, then Riley sees only start times where the full service duration plus buffer fits inside Talia's open working hours with no conflicting Booking, Time Block, Recurring Series occurrence, active Slot Hold, or calendar busy time.

**FEAT-03.SPEC-001-AC-02:** Given Talia has zero open time for the selected service across the visible date range, when the computation completes, then Riley sees a plain "fully booked, check back or view other services" message rather than a blank grid.

**FEAT-03.SPEC-001-AC-03:** Given the computation cannot complete due to a read failure, when Riley is viewing the booking page, then Riley sees a retry prompt and no slot list -- never a stale or incorrect one.

**FEAT-03.SPEC-001-AC-04:** Given Talia cancels an upcoming Booking, when Riley next requests the same service's slot list, then the freed time appears as open without any separate recompute action.

**FEAT-03.SPEC-001-AC-05:** Given Talia's calendar sync (FEAT-03.SPEC-006) is currently in degraded mode, when Riley requests the slot list, then Riley sees the ordinary computed list with no indication of reduced confidence, while Talia's own dashboard shows the reduced-confidence banner.

**FEAT-03.SPEC-001-AC-06:** Given a candidate start time falls within Talia's minimum_booking_notice, when the computation runs, then that candidate is excluded from Riley's list (per FEAT-03.SPEC-004).

**FEAT-03.SPEC-001-AC-07:** Given a candidate start time falls beyond Talia's booking_horizon, when the computation runs, then that candidate is excluded from Riley's list (per FEAT-03.SPEC-004).

**FEAT-03.SPEC-001-AC-08:** Given a slot is currently held by another client's in-progress checkout (FEAT-03.SPEC-002), when a second client requests the same service's list at effectively the same moment, then the held slot does not appear in the second client's result.

**FEAT-03.SPEC-001-AC-09:** Given Talia has a Recurring Series generating a future occurrence inside the requested date range, when the computation runs, then that occurrence's exact time is excluded from the client-facing list.

**FEAT-03.SPEC-001-AC-10:** Given Riley is rescheduling an existing booking (FEAT-10), when Riley requests a new time for the same service, then the computation excludes the same conflicts as a fresh booking would, without treating the booking's own current time as a self-conflict.

**FEAT-03.SPEC-001-AC-11:** Given Talia changes her working hours mid-week (FEAT-02), when Riley requests the slot list afterward, then the list reflects the new hours immediately, with no separate propagation delay.

**FEAT-03.SPEC-001-AC-12:** Given the requested date range extends beyond Talia's booking_horizon, when the computation runs, then only the in-horizon portion of the range returns candidates and the out-of-horizon portion returns none.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Slot Hold Creation & Checkout Reservation

## Overview

**Name:** Slot Hold Creation & Checkout Reservation
**ID:** FEAT-03.SPEC-002
**Type:** Automation
**Purpose:** Creates a time-limited Slot Hold the instant a client begins paying for a chosen slot, instantly excluding it from every other client's computed availability so no second client can grab it mid-checkout.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Creating a Slot Hold at the moment a client begins the payment step of checkout
- Re-validating the candidate slot against FEAT-03.SPEC-004's rules at the instant of hold creation (a slot computed a moment earlier must still pass)
- Resolving a contested slot per FEAT-03.SPEC-005 when two clients attempt to hold the same slot concurrently
- Handing the created hold's identity to the payment step so it can be converted to a Booking on payment completion

**Non-Goals:**
- Processing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reserves the time slot the payment is for
- Expiring or releasing the hold -- owned by FEAT-03.SPEC-003, which this spec's created hold is subject to
- The Pro-created deposit-request hold class (longer-lived, up to platform parameter: `deposit-request-hold-max-hours`) -- owned by FEAT-03.SPEC-007, a distinct creation path for a distinct trigger (a Pro booking a client in), per the Entity-Lifecycle Coverage Matrix's two-creation-paths note
- Converting a hold into a confirmed Booking -- a cross-feature outcome owned by FEAT-07 when payment completes (Booking creation is FEAT-07's write, not this automation's)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client begins the payment step for a chosen slot | FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) -- within FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) | Fires the instant a client, having selected a service and slot, advances into the deposit payment step | Service ID, chosen start time and duration, the in-progress Booking's Client reference |
| Client begins the payment step for a reschedule | FEAT-10 (Client-Initiated Cancel/Reschedule) | Fires when a client, rescheduling an existing booking, advances to confirm the new time (no new deposit charge if within policy, but the new time must still be held while the change commits) | Service ID, new start time and duration, the existing Booking reference |

## Processing Logic

1. Receive the candidate Service ID, start time, and duration from the triggering checkout step.
2. Re-validate the candidate slot against FEAT-03.SPEC-004's rules (duration+buffer fit, minimum notice, booking horizon) exactly as at computation time -- a slot that no longer passes is rejected before any hold is attempted.
3. Check for any existing active Slot Hold (of either class) already covering the candidate start time.
4. If no conflicting hold, Booking, Time Block, Recurring Series occurrence, or calendar busy period covers the candidate time, create a new Slot Hold: service, start time, duration, the owning in-progress checkout (or reschedule), a hold-created timestamp, an expiry timestamp set to the fixed checkout-hold timeout, and state Active.
5. If a conflicting hold or occupancy is found, apply the contention resolution rule (FEAT-03.SPEC-005) to determine the outcome for this attempt.
6. Return the created hold's identity to the triggering checkout step so the payment step can proceed against it.
7. The created hold immediately excludes the candidate time from the next slot-list computation (FEAT-03.SPEC-001) for every other client.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold created | Candidate slot passes re-validation and no conflict exists | New Slot Hold created (Active) | Client proceeds to the deposit payment step with the slot reserved | FEAT-05, FEAT-07, FEAT-03.SPEC-001 |
| Slot no longer valid | Re-validation against FEAT-03.SPEC-004 fails (e.g., minimum notice now violated) | None | Client sees a plain message that the slot is no longer available and is returned to a refreshed live slot list | FEAT-05, FEAT-03.SPEC-001 |
| Slot contested | Another hold, Booking, or calendar event already occupies the candidate time | None for the losing attempt | Client sees the plain "just taken" message per FEAT-03.SPEC-005 and a refreshed live list | FEAT-03.SPEC-005, FEAT-05 |
| Hold creation failure | The hold cannot be written (e.g., a processing error) | None | Client sees a retry prompt; no payment step is entered without a confirmed hold | FEAT-05 |

## Data Model

**Reads:** Service (duration, buffer_override), Availability Rule (for re-validation), Booking, Time Block, Recurring Series, existing Slot Hold records, Calendar Connection busy periods (via FEAT-03.SPEC-006).
**Creates:** Slot Hold -- service, start time, duration, owning checkout/deposit-request reference, hold-created timestamp, expiry timestamp (fixed checkout-hold timeout), state Active.
**Updates:** None -- this spec only creates; expiration and conversion are owned elsewhere (FEAT-03.SPEC-003, FEAT-07).
**Deletes:** None.

## Business Rules

- A Slot Hold is created only after the candidate slot re-passes every rule in FEAT-03.SPEC-004 -- a slot computed a moment earlier is never assumed still valid (XBR-01).
- The checkout hold's expiry is a fixed, short timeout: platform parameter: `checkout-hold-timeout-minutes`.
- The first client to successfully create a hold on a given slot wins it; a second, concurrent attempt on the same slot is resolved per FEAT-03.SPEC-005's first-committed-wins rule (XBR-01, XBR-02).
- A reschedule's new-time hold follows the same creation rule as a new booking's hold -- the Brief's Shared Validation note that SPEC-002 re-validates rather than re-deriving FEAT-03.SPEC-004's rules.
- One hold exists per contested slot at a time; no two Active holds can cover the same overlapping time for the same Pro Account.

## Edge Cases

- **Client abandons checkout after a hold is created but before payment starts** -- The hold remains Active until its timeout, then expires per FEAT-03.SPEC-003; no separate abandonment signal is needed.
- **Candidate slot re-validation fails because minimum notice was crossed while the client was choosing** -- The client sees a plain "this time is no longer available" message and a refreshed live list, never a payment error.
- **Concurrent trigger firing (two clients begin checkout for the same slot at effectively the same time)** -- Exactly one hold-creation attempt succeeds; the other is refused per FEAT-03.SPEC-005, with no partial or duplicate hold ever existing.
- **Trigger fires while a previous hold-creation attempt for the same client is still in flight** -- The client's checkout step disables further submission until the in-flight attempt resolves, preventing a duplicate hold request from the same client.
- **Reschedule hold contends with the booking's own original time** -- The booking's own currently-held original time is never treated as a conflict against its own reschedule attempt; only the new candidate time is checked for conflicts.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The created hold immediately excludes the slot from the next computation |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Candidate slot is re-validated against these rules at hold-creation time |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when the candidate slot is contested |
| FEAT-03.SPEC-003 (Slot Hold Expiration) | Affects (outbound) | The created hold is subject to this spec's timeout and release logic |
| FEAT-05 (Public Booking Page & Booking Flow) | Triggered by (inbound) / Affects (outbound) | Checkout's payment step triggers hold creation and receives the outcome |
| FEAT-07 (Deposit Payment at Booking) | Triggered by (inbound) / Affects (outbound) | Payment step begins against the created hold; converts it to a Booking on success |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A reschedule's new-time confirmation also creates a hold through this spec |

## Analytics and Success Signals

- **slot_held** (service_id, hold_type: checkout, start_time) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_creation_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_contested_at_checkout** (service_id, outcome: won / lost) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **checkout_started** (service_id) -- supports success-metrics.md: "Booking Completion Speed" (marks the start of the timed checkout window this metric measures)

## Acceptance Criteria

**FEAT-03.SPEC-002-AC-01:** Given Riley selects a genuinely free slot and taps to begin payment, when the checkout step advances, then a Slot Hold is created instantly and Riley proceeds to the deposit payment step.

**FEAT-03.SPEC-002-AC-02:** Given a Slot Hold has just been created for Riley's chosen slot, when a second client (a different Riley) requests the same service's slot list moments later, then the held slot does not appear as open.

**FEAT-03.SPEC-002-AC-03:** Given Riley's chosen slot no longer passes FEAT-03.SPEC-004's minimum-notice rule by the time checkout begins, when the hold-creation step re-validates it, then Riley sees a plain "this time is no longer available" message and a refreshed live list, never a payment error.

**FEAT-03.SPEC-002-AC-04:** Given two clients begin checkout for the same slot at effectively the same time, when hold creation runs for both, then exactly one succeeds and the other sees the plain "just taken" message per FEAT-03.SPEC-005.

**FEAT-03.SPEC-002-AC-05:** Given Riley abandons checkout after a hold is created but never reaches payment, when the checkout-hold timeout elapses, then the hold expires per FEAT-03.SPEC-003 and the slot reappears.

**FEAT-03.SPEC-002-AC-06:** Given a hold cannot be created due to a processing error, when Riley attempts to begin checkout, then Riley sees a retry prompt and is not advanced into the payment step.

**FEAT-03.SPEC-002-AC-07:** Given Riley is rescheduling an existing booking and picks a new free time, when Riley confirms the new time, then a Slot Hold is created on the new time using the same validation as a new booking, without treating the booking's own current time as a conflict.

**FEAT-03.SPEC-002-AC-08:** Given a Slot Hold already exists on a candidate slot (checkout or Pro-created), when another client attempts to hold the same slot, then the attempt is refused per FEAT-03.SPEC-005 rather than creating a second, overlapping hold.

**FEAT-03.SPEC-002-AC-09:** Given Riley's checkout attempt is still in flight after tapping to begin payment, when Riley taps the same control again before the first attempt resolves, then no duplicate hold-creation request is submitted.

**FEAT-03.SPEC-002-AC-10:** Given a Slot Hold is successfully created for Riley, when Riley completes the deposit payment (FEAT-07), then the hold is converted to a confirmed Booking rather than expiring.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Slot Hold Expiration

## Overview

**Name:** Slot Hold Expiration
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** Automatically expires and deletes a checkout Slot Hold that reaches its fixed timeout without completed payment, returning the slot to public availability.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Detecting when an Active checkout Slot Hold (created by FEAT-03.SPEC-002) reaches its fixed timeout without a completed payment
- Transitioning that hold to Expired and deleting it, per the Entity-Lifecycle Coverage Matrix's Delete/Archive row
- Notifying the client, in-flow, that their held time has expired
- Applying the contention resolution rule (FEAT-03.SPEC-005) when a client's payment and the timeout race

**Non-Goals:**
- Expiring the Pro-created deposit-request hold class -- owned by FEAT-03.SPEC-007, whose longer, variable expiry (up to platform parameter: `deposit-request-hold-max-hours` or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment) is a distinct rule from this spec's fixed few-minute checkout timeout
- Creating the hold in the first place -- owned by FEAT-03.SPEC-002
- Converting a hold to a confirmed Booking -- a cross-feature outcome owned by FEAT-07 when payment completes before the timeout is reached
- Historical retention of expired holds -- excluded per the Entity-Lifecycle Coverage Matrix's explicit non-goal: "this is a transient computation artifact with a lifetime of minutes... never a historical record"

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A checkout Slot Hold's expiry timestamp is reached | FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Fires the moment the hold's expiry timestamp passes with the hold still Active (payment not completed) | The hold's service, start time, duration, owning checkout reference |

## Processing Logic

1. Identify each Active checkout Slot Hold whose expiry timestamp has passed.
2. Confirm the hold has not already transitioned to Consumed by a completed payment (FEAT-07) -- if it has, take no action (the race is resolved in payment's favor per Step 4 below).
3. Transition the hold's state to Expired.
4. Delete the hold record, per the Entity-Lifecycle Coverage Matrix (no restore path; a released hold simply becomes an ordinary open slot again).
5. Signal the owning checkout step (if the client is still on the payment screen) that the hold has expired.
6. The freed time reappears the next time FEAT-03.SPEC-001 computes the open-slot list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Hold expired and released | Timeout reached with no completed payment | Slot Hold transitioned to Expired then deleted | Client, if still on the payment screen, sees a plain message that their held time expired and is returned to a refreshed live slot list (never a payment error) | FEAT-05, FEAT-07, FEAT-03.SPEC-001 |
| Race won by payment | Payment completes at effectively the same moment the timeout is reached | Hold instead transitioned to Consumed by FEAT-07; this automation takes no further action | Client sees their booking confirmed, not an expiry message | FEAT-07 |
| Expiration failure | The expiration processing itself cannot complete (e.g., a processing error) | None -- the hold remains Active until retried | No client-facing message; this is a non-blocking internal failure that is retried | FEAT-03.SPEC-001 (a lingering stale hold would otherwise incorrectly withhold the slot) |

## Data Model

**Reads:** Slot Hold (state, expiry timestamp, owning checkout reference).
**Creates:** None.
**Updates:** Slot Hold -- state transitioned to Expired.
**Deletes:** Slot Hold -- the expired record is removed, per the Entity-Lifecycle Coverage Matrix.

## Business Rules

- The checkout hold's timeout is fixed and short: platform parameter: `checkout-hold-timeout-minutes` (same marker as defined in FEAT-03.SPEC-002 -- reused verbatim).
- The first committed action wins a race between an expiring hold and a completing payment (XBR-01, FEAT-03.SPEC-005): if payment completes before this automation processes the expiry, the hold is Consumed, not Expired.
- An expired hold is deleted, not archived -- there is no restore path and no retention requirement, since it is a transient computation artifact (Entity-Lifecycle Coverage Matrix).
- Expiration is a background process; it is never blocked by, or blocking to, any client's screen state.

## Edge Cases

- **Payment completes in the same instant the hold's timeout elapses** -- The completed-payment transition (to Consumed) takes precedence; this automation detects the already-Consumed state in Step 2 and takes no action, so the client is never shown an expiry message for a booking that actually succeeded.
- **Client's device is offline when their hold expires** -- The hold still expires server-side on schedule; the client sees the expiry message on their next successful interaction (e.g., attempting to submit payment), never a silently-accepted payment against an expired hold.
- **Concurrent trigger firing (two holds for different clients expire at the same moment)** -- Each hold's expiration is processed independently; there is no shared state between unrelated holds, so no contention exists between them.
- **Trigger fires while a previous expiration run for the same hold is still in flight** -- Expiration is idempotent: a hold already transitioned to Expired and deleted is a no-op if the expiration logic is invoked again for it.
- **Expiration processing itself fails** -- The hold remains Active and is retried; it is never left in an ambiguous state, and FEAT-03.SPEC-001's computation continues to correctly treat it as held until the retry succeeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-002 (Slot Hold Creation & Checkout Reservation) | Triggered by (inbound) | The hold this spec expires was created there, with the expiry timestamp it set |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The freed slot reappears in the next computation |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the race outcome between an expiring hold and a completing payment |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout), FEAT-05.SPEC-002 (Slot Selection) -- within FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Expired hold returns the client to the live slot list with an explanatory message |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) | Expired hold returns the client to the live slot list with an explanatory message; a completed payment instead converts the hold |

## Analytics and Success Signals

- **slot_hold_expired** (service_id, hold_type: checkout) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_expired_race_lost_to_payment** (service_id) -- N/A -- no Stage 2 metric measures this internal race outcome specifically; retained so the correctness of the first-committed-wins rule is observable
- **slot_hold_expiration_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence" (a failed expiration must never leave a stale hold silently blocking a slot -- this event measures how often the retry path is exercised)

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Riley's Slot Hold has reached its fixed checkout timeout without completed payment, when the expiration automation runs, then the hold is expired and deleted, and the slot reappears in the next computed list.

**FEAT-03.SPEC-003-AC-02:** Given Riley is still on the payment screen when the hold expires, when expiration completes, then Riley sees a plain message that the held time expired and is returned to a refreshed live slot list, never a payment error.

**FEAT-03.SPEC-003-AC-03:** Given Riley's payment completes at effectively the same moment the hold's timeout is reached, when both processes evaluate, then the hold is transitioned to Consumed by the completed payment, and the expiration automation takes no action.

**FEAT-03.SPEC-003-AC-04:** Given Riley's device loses connection just before the hold expires, when Riley next interacts with the payment screen after reconnecting, then Riley sees the expiry message rather than an ambiguous or silently-accepted payment attempt.

**FEAT-03.SPEC-003-AC-05:** Given two different clients' holds expire at the same moment, when the expiration automation processes both, then each is expired independently with no interference between them.

**FEAT-03.SPEC-003-AC-06:** Given a hold has already been expired and deleted, when the expiration logic is invoked again for the same hold, then nothing changes (no error, no duplicate action).

**FEAT-03.SPEC-003-AC-07:** Given the expiration processing itself encounters a processing error, when the hold's timeout has passed, then the hold remains Active and is retried, and FEAT-03.SPEC-001 continues to correctly exclude it as held until the retry succeeds.

**FEAT-03.SPEC-003-AC-08:** Given a checkout Slot Hold expires, when Riley next requests the same service's slot list, then the freed time appears as open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: Slot Contention Resolution Rules

## Overview

**Name:** Slot Contention Resolution Rules
**ID:** FEAT-03.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs how a contested slot -- two clients attempting the same time, or a client colliding with a Pro-side change -- resolves: the first committed action wins, and every other attempt sees a plain re-pick message, never a payment error.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine
**Governed Entity:** Slot Hold, plus its contention against Booking and Time Block occupancy

## Scope and Non-Goals

**In Scope:**
- The tie-break rule for two clients (or a client and a Pro-created deposit-request hold) contending for the same slot at effectively the same time
- The tie-break rule for a client's in-progress checkout colliding with a Pro-side change (a new Time Block, a Pro-side booking) committed first
- The exact "just taken" experience every losing attempt receives
- Authorization for who can trigger a contention outcome and who is shown what

**Non-Goals:**
- Creating the Slot Hold that becomes contended -- owned by FEAT-03.SPEC-002 (checkout) and FEAT-03.SPEC-007 (Pro-created deposit request); this spec governs only the tie-break outcome, not hold creation itself
- Expiring an uncontested hold that simply times out -- owned by FEAT-03.SPEC-003 and FEAT-03.SPEC-007; this spec applies only when two committed actions actually collide
- The waitlist's 30-minute priority window -- excluded per this Brief's Non-Goals: XBR-28 assigns that timing rule to FEAT-20 (Waitlist for Cancelled Slots); this spec supplies only the underlying free/not-free slot truth and generic tie-break mechanism FEAT-20 builds on
- Deciding deposit refund or forfeiture outcomes for a losing or bumped booking -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec governs only which attempt wins the slot, not the money consequence of losing one

## Governed Entity

**Entity:** Slot Hold, contended against Booking and Time Block occupancy
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| service | reference | The Service the contended hold or booking is for |
| start_time / duration | date/time, number | The specific contested span |
| owning_actor | enum (Client checkout, Pro deposit request, Pro-side change) | Which actor's action is attempting to claim or occupy the span |
| commit_timestamp | date/time | The exact moment the action was committed (hold created, or Time Block/Booking written) |
| state | enum (Active, Expired, Consumed) | Slot Hold's own lifecycle state, per the Entity-Lifecycle Coverage Matrix |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | At the moment a checkout hold-creation attempt finds an existing conflicting hold or occupancy |
| FEAT-03.SPEC-003 | Slot Hold Expiration | When an expiring hold races against a completing payment for the same slot |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | At the moment a Pro-created hold-creation attempt finds an existing conflicting hold or occupancy |
| FEAT-20.SPEC-004 | Waitlist Priority Claim Window Rule | Builds the waitlist priority window on this tie-break: when two notified clients claim the same freed slot, the first to complete payment wins under this rule |
| FEAT-05 | Public Booking Page & Booking Flow | Displays the "just taken" message and refreshed list to a losing client |
| FEAT-10 | Client-Initiated Cancel/Reschedule | Displays the same contention outcome when a reschedule's new-time attempt is contended |
| FEAT-17 | Manual Time Blocking | A Time Block committed first against an in-progress checkout produces the Pro-side-change contention outcome |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| commit_timestamp | The action with the earliest commit_timestamp for a given contested span is the winner; every later action for the same span is refused | Always, whenever two or more actions target the same or overlapping span | At the instant a second (or later) action attempts to commit against an already-committed span | "That time was just taken. Here are the current available times." (client-facing); the Pro sees the conflict on their dashboard if the Pro-side action loses (rare, since a Pro-side change ordinarily wins against a client) | Yes |
| owning_actor | No validation beyond data type -- contention resolution applies identically regardless of which actor type is involved | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| First-committed-wins | commit_timestamp, owning_actor | Whichever action (checkout hold creation, Pro-created hold creation, or a Pro-side Time Block/Booking write) commits first for a given span wins it; every other action targeting the same or overlapping span at that point is refused, regardless of actor type | "That time was just taken. Here are the current available times." |
| Checkout-hold vs. Pro-side-change collision | commit_timestamp (hold), commit_timestamp (Time Block/Booking) | If a Pro-side change commits before the client's payment completes, the client's in-progress checkout is refused even though a hold already exists on the slot, per this Brief's Time Block Contention note ("first committed wins... a block committed first removes the slot and the client's confirmation is refused with a refreshed slot list") | "That time was just taken. Here are the current available times." |
| Hold-expiry-vs-payment-completion race | commit_timestamp (payment completion), expiry timestamp (hold) | If payment completes before the hold's expiry is processed, the hold is Consumed, not Expired -- payment completion counts as the earlier "commit" for this race (FEAT-03.SPEC-003) | -- (no error; this is the winning path) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Trigger a contention check | The Client (Riley), the Pro (Talia) | Whenever either commits an action (hold creation, Time Block, booking) against a span another action already occupies | -- |
| Receive the "just taken" message | The Client (Riley) | Only the losing attempt in a client-vs-client or client-vs-Pro-side-change contention | -- (this is the denied behavior itself) |
| See the conflicting Pro-side change that caused a client's loss | The Pro (Talia) | Always, on her own dashboard, if her own action lost a rare Pro-vs-Pro-created-hold race | Flagged for her explicit choice per XBR-11 (setup changes never silently cancel a confirmed booking) |
| View another client's contention outcome or identity | The Client (Riley) | Never | Riley sees only "that time was just taken," never who took it or any detail about the other client |
| View another client's contention outcome or identity | Platform Operator (Support) | View-only, for troubleshooting a specific Pro's reported conflict; never shown another client's identity beyond what troubleshooting requires | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|----------------------|
| commit_timestamp | Set automatically to the exact system time the action is written (hold created, or Time Block/Booking committed) | On every hold creation, Time Block creation, or Booking write | No -- never client- or Pro-editable |
| winner determination | Derived by comparing commit_timestamp across every action targeting the same or overlapping span; earliest wins | Whenever a contention is detected | No |

## Business Rules

- A time is offered, held, or booked only if it passes the live slot check; the first client to complete payment wins a contested slot and the other sees a plain "just taken" message, never a payment error (XBR-01) -- this is the rule this spec formalizes as the tie-break itself.
- Slot holds are time-limited and release automatically; a race between an expiring hold and a completing payment resolves in the completing payment's favor if payment commits first (XBR-02, FEAT-03.SPEC-003).
- A Time Block committed first removes the slot and refuses a contended client confirmation with a refreshed slot list; a client's confirmation committed first makes the block a conflict the Pro must explicitly decide (cancel, reschedule, or keep as an exception), per this Brief's Time Block Contention note and XBR-11 -- this spec's tie-break rule underlies both directions.
- Contention resolution never merges two conflicting actions into a combined or partial outcome -- exactly one action wins a given span, and every other action for that span is refused outright.
- The losing party is always shown a refreshed live slot list immediately alongside the "just taken" message, never left on a page showing the now-stale slot as still available.

## Edge Cases

- **Two clients' checkout holds are created within the same millisecond for the same slot** -- Exactly one hold-creation write succeeds (the underlying single-Pro-Account data store enforces this); the other is refused as a contention loss even though both attempts appeared simultaneous to their respective clients.
- **A client's payment completes and a Pro-created deposit-request hold is attempted on the same slot at nearly the same time** -- Whichever action's commit_timestamp is earlier wins; if the client's payment committed first, the Pro's attempt to create a deposit-request hold on that slot fails with the Pro seeing the slot is no longer available.
- **A Pro's Time Block is committed for a span where a client's checkout hold already exists (Active, unexpired)** -- The existing hold committed first, so it wins: the Time Block creation is refused or flagged to the Pro as a conflict requiring her explicit choice (FEAT-17, XBR-11), not silently applied over an active hold.
- **A losing client retries the exact same slot immediately after seeing "just taken"** -- The retry is evaluated as a brand-new contention check against current data; if the slot is still occupied, the same refusal recurs; if it has since freed (e.g., the winning hold itself later expired), the retry can succeed.
- **Contention outcome must be determined but the underlying data store cannot confirm which action committed first (a rare consistency failure)** -- The system defaults to refusing both contending actions rather than guessing a winner, and both parties see a refreshed live list; neither is shown as confirmed until a clean, unambiguous single-winner commit succeeds.

## Acceptance Criteria

**FEAT-03.SPEC-005-AC-01:** Given Riley and a second client both attempt to hold the same slot at effectively the same time, when the hold-creation attempts are evaluated, then exactly one succeeds (the earliest commit_timestamp) and the other sees "That time was just taken. Here are the current available times."

**FEAT-03.SPEC-005-AC-02:** Given Riley's checkout hold is Active on a slot, when Talia attempts to place a Time Block over that same span, then the Time Block attempt is refused or flagged to Talia as a conflict requiring her explicit choice, since Riley's hold committed first.

**FEAT-03.SPEC-005-AC-03:** Given Talia commits a Time Block over a span before any client has begun checkout on it, when a client subsequently attempts to hold that span, then the hold-creation attempt is refused with the "just taken" message, since the block committed first.

**FEAT-03.SPEC-005-AC-04:** Given Riley's checkout hold's timeout is about to elapse at the same moment Riley's payment completes, when both are evaluated, then the hold is Consumed by the completed payment, not expired, because payment completion is treated as the earlier commit.

**FEAT-03.SPEC-005-AC-05:** Given a losing client sees the "just taken" message, when the message displays, then Riley also sees a refreshed live slot list immediately, never a stale page still showing the taken slot as available.

**FEAT-03.SPEC-005-AC-06:** Given Riley loses a contention, when Riley checks whether they can see who took the slot, then no other client's identity or detail is ever shown -- only the plain "just taken" message.

**FEAT-03.SPEC-005-AC-07:** Given Talia's own Pro-created deposit-request hold attempt loses a rare race against a client's just-completed payment, when Talia views her dashboard, then she sees the slot is no longer available for that deposit request, with no ambiguity about the outcome.

**FEAT-03.SPEC-005-AC-08:** Given Riley retries the same slot immediately after losing a contention, when the retry is evaluated and the slot is still occupied, then Riley sees the same "just taken" refusal.

**FEAT-03.SPEC-005-AC-09:** Given Riley retries the same slot after the winning hold has since expired, when the retry is evaluated, then Riley's new attempt can succeed since the slot is now free.

**FEAT-03.SPEC-005-AC-10:** Given a rare data-consistency failure prevents determining which of two contending actions committed first, when the contention check runs, then both actions are refused and both parties see a refreshed live list rather than either being shown as confirmed.

**FEAT-03.SPEC-005-AC-11:** Given Riley is rescheduling an existing booking and the new time becomes contended by another client mid-flow, when the contention resolves against Riley, then Riley sees the "just taken" message and a refreshed live list, with the original booking left untouched.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Integration Spec: Calendar Busy-Time Consumption & Degraded Mode

## Overview

**Name:** Calendar Busy-Time Consumption & Degraded Mode
**ID:** FEAT-03.SPEC-006
**Type:** Integration
**Purpose:** Consumes busy/free periods from the Pro's connected personal calendar (the connection itself owned by FEAT-04) as an additional availability constraint, and defines the fallback behavior -- Chairtime-only data with a Pro-only reduced-confidence flag -- when that sync becomes unavailable.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Consuming the busy_periods data FEAT-04's Calendar Connection already reads from the Pro's personal calendar
- Supplying those busy periods to FEAT-03.SPEC-001's slot computation as an additional occupancy constraint
- Detecting when calendar sync health degrades or lapses and falling back to Chairtime-only data
- Surfacing reduced confidence to the Pro only, on their own dashboard, never to the client

**Non-Goals:**
- Establishing, authenticating, or maintaining the Calendar Connection itself (connecting, reconnecting, disconnecting, and reporting sync health) -- owned entirely by FEAT-04 (Two-Way Calendar Sync); this spec only consumes what FEAT-04 already reads
- Writing Chairtime bookings out to the Pro's personal calendar -- also owned by FEAT-04; this spec is inbound-only (busy time in), never outbound booking writes
- Deciding the actual slot computation logic that combines busy time with other occupancy sources -- owned by FEAT-03.SPEC-001, which this spec feeds
- Storing full calendar event details -- excluded per the dependency map's Calendar Connection entity definition: "only busy/free periods are kept, never event titles or details"

## Capability Category

**Category:** Calendar sync
**Dependency Source:** ASMP-33 -- "Calendar-sync capability (reading and writing to a pro's personal calendar) -- required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the 'never double-book' correctness bar." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Calendar sync -- reading busy time from and writing bookings to a Pro's personal calendar (ASMP-33)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-04, FEAT-03, FEAT-15; Integration Specs: FEAT-04.SPEC-003 for the connection handshake and booking writes, FEAT-03.SPEC-006 -- this spec -- for busy-time consumption and degraded mode)
**Vendor Mandate:** None recorded in BRIEF.md for the specific calendar-sync mechanics this spec covers -- BRIEF.md's Ecosystem & Integrations names Google and Apple as the two supported personal-calendar kinds the Pro may connect through FEAT-04, but vendor selection for the underlying sync mechanism is a Stage 4 decision; this spec stays at the category level throughout.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A time slot that conflicts with Talia's personal calendar event is never offered to a client | Compute the live set of open slots for a given service and date range | FEAT-03.SPEC-001 (Slot Availability Computation) |
| Talia sees a reduced-confidence indicator when her calendar sync has lapsed, so she knows Chairtime-only data is currently in effect | Never silently double-book (BRIEF.md, Vision) | FEAT-12 (Pro Daily Schedule Dashboard, Pro-visible banner) |
| A client is never shown any indication that calendar confidence is reduced -- the client always sees a clean, ordinary slot list | Never expose Pro-side operational detail to a client | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Exchanged

**Leaves the product:**

No data leaves the product through this spec. This spec is inbound-only: it consumes busy_periods already read by FEAT-04's own connection handshake (FEAT-04.SPEC-003 owns any outbound data, such as Chairtime booking writes to the personal calendar). Nothing from this spec's own processing -- no Booking, Client, or Service detail -- is sent to the calendar-sync capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|------------------------------|
| Busy/free periods (minimum data needed to block slots -- never full event details) | FEAT-04 completes a sync cycle with the Pro's connected personal calendar | Calendar Connection -- busy_periods, last_successful_sync (read by this spec; written by FEAT-04) |
| Sync health status (Connected / Syncing / Needs Reconnection / Disconnected) | FEAT-04 detects a change in connection health | Calendar Connection -- status (read by this spec; written by FEAT-04) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Busy periods refreshed | FEAT-04 completes a sync cycle successfully | None written by this spec (Calendar Connection is FEAT-04's write) | None directly -- the next slot computation simply reflects the refreshed busy periods | FEAT-03.SPEC-001 (consumes the refreshed data) |
| Sync health degrades (status moves to Needs Reconnection or Disconnected) | FEAT-04 detects the connection has lapsed | None written by this spec; this spec reads the status change and sets its own confidence flag to Reduced for consumption by FEAT-03.SPEC-001 | Talia sees a reduced-confidence banner on her dashboard; Riley sees no change at all | FEAT-03.SPEC-001, FEAT-12 (Pro Daily Schedule Dashboard) |
| Sync health recovers (status returns to Connected) | Talia reconnects or the connection self-heals (FEAT-04) | This spec's confidence flag returns to Normal | Talia's reduced-confidence banner clears | FEAT-03.SPEC-001, FEAT-12 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|---------------------|
| FEAT-05 (Public Booking Page & Booking Flow) | No visible effect -- the client-facing slot list computes from whatever busy-period data is currently available (last successfully synced, or none) with no waiting state tied to calendar sync specifically | Availability falls back to Chairtime-only data (Availability Rule, Booking, Time Block, Recurring Series, Slot Hold); the client sees the ordinary computed list with no indication anything is degraded, never a client-facing error or delay | N/A -- this spec never sends a request the calendar-sync capability can reject; it only consumes data FEAT-04 already reads |
| FEAT-12 (Pro Daily Schedule Dashboard) | A brief "syncing" indication may appear on the connection-health banner (owned by FEAT-04); this spec's own confidence flag stays Normal until sync genuinely lapses | Talia sees a reduced-confidence banner: "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy." The rest of her dashboard remains fully usable | N/A -- same as above; this spec has no request path a capability can reject |

## Consent and Disclosure

- **No new disclosure moment introduced by this spec** -- the consent and disclosure moment for sharing calendar access (what data is read, what a Pro is told) belongs to FEAT-04's connection handshake (FEAT-04.SPEC-003), since this spec introduces no new outbound data of its own; it only reads what FEAT-04 has already disclosed and captured.
- **What is never shared or surfaced** -- full calendar event details (titles, descriptions, attendees) are never read or stored by this spec or by FEAT-04; only the minimum busy/free period data needed to block slots ever enters Chairtime, per the Calendar Connection entity's Data Sensitivity note, and none of it is ever shown to a client.
- **Reduced-confidence disclosure (Pro-facing only)** -- when sync health degrades, Talia's dashboard states plainly: "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy." This is shown to the Pro only; the client-facing booking page carries no equivalent notice, per this feature's own design intent that a client "must never be shown a slot that turns out to be unavailable" while also never being shown Pro-side operational detail.

## Edge Cases

- **Busy-period data refreshes while a client is mid-computation** -- The computation in flight uses whatever busy-period snapshot was current when it started; a newly refreshed set of busy periods is reflected starting with the next computation, never retroactively altering a result already returned.
- **Sync health degrades and then recovers within the same minute** -- The confidence flag follows the most recent status change; a rapid degrade-then-recover sequence simply leaves the flag at Normal once recovery is detected, with no lingering reduced-confidence banner.
- **The Pro has never connected a calendar at all (no Calendar Connection exists)** -- Busy periods contribute nothing (an empty set), and no reduced-confidence flag is raised, since "reduced confidence" applies only to a lapsed existing connection, not to the ordinary case of no connection ever being made.
- **Calendar-sync capability reports busy-period data twice for the same sync cycle** -- The second delivery changes nothing beyond the current busy-period snapshot it already reflects; no duplicate exclusion or double-counting occurs in the next computation.
- **Sync health status arrives out of order (a "recovered" event processed before an earlier "degraded" event)** -- The confidence flag reflects the most recent event by its own event time, not arrival order, consistent with FEAT-04's own health reporting.
- **Degradation strikes mid-way through a client's checkout (busy-period data becomes stale while a hold is already Active)** -- The already-created Slot Hold (FEAT-03.SPEC-002) is unaffected; degraded confidence changes nothing about slots already held or already computed in the current view, since the correctness guarantee for an in-progress checkout rests on the hold itself, not on live calendar freshness.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Connection handshake, busy-time pull, booking write/move/remove -- owned by FEAT-04) | Triggered by (inbound) | Supplies the busy_periods and status this spec reads and consumes |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Busy periods and the confidence flag are consumed as an additional occupancy constraint |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Reduced-confidence banner surfaces here, Pro-only |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Receives only the ordinary computed list; never a degraded-mode indicator |

## Analytics and Success Signals

- **calendar_busy_time_consumed** (pro_account_id, sync_confidence: normal / reduced) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_confidence_degraded** (pro_account_id) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_confidence_recovered** (pro_account_id) -- supports success-metrics.md: "Calendar Sync Reliability"
- **slot_computation_used_degraded_calendar_data** (pro_account_id) -- supports success-metrics.md: "Zero Double-Booking Confidence" (measures how often the fail-safe Chairtime-only fallback is exercised, which is exactly the correctness guarantee this metric validates)

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given Talia's calendar sync is healthy and reports a busy period overlapping a candidate slot, when FEAT-03.SPEC-001 computes the open-slot list, then that candidate is excluded from Riley's list.

**FEAT-03.SPEC-006-AC-02:** Given Talia's calendar sync has just degraded (FEAT-04 reports Needs Reconnection), when Talia opens her dashboard, then she sees "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy."

**FEAT-03.SPEC-006-AC-03:** Given Talia's calendar sync is degraded, when Riley requests the slot list, then Riley sees the ordinary computed list with no indication anything is degraded.

**FEAT-03.SPEC-006-AC-04:** Given Talia's calendar sync recovers after a degraded period, when Talia next opens her dashboard, then the reduced-confidence banner clears.

**FEAT-03.SPEC-006-AC-05:** Given Talia has never connected a personal calendar, when a slot is computed, then no reduced-confidence flag is raised, and busy periods contribute nothing to the computation.

**FEAT-03.SPEC-006-AC-06:** Given calendar sync degrades and recovers within the same minute, when the confidence flag is evaluated, then it reflects Normal, with no lingering banner shown to Talia.

**FEAT-03.SPEC-006-AC-07:** Given busy-period data is delivered twice for the same sync cycle, when the second delivery arrives, then no duplicate exclusion or double-counting occurs in the next computation.

**FEAT-03.SPEC-006-AC-08:** Given a "recovered" health event and an earlier "degraded" event arrive out of order, when the confidence flag is evaluated, then it reflects the most recent event by its own event time, not arrival order.

**FEAT-03.SPEC-006-AC-09:** Given Riley's checkout hold is already Active when calendar sync degrades mid-flow, when the degradation is detected, then Riley's already-created hold is unaffected.

**FEAT-03.SPEC-006-AC-10:** Given a client attempts to view any calendar-sync detail on the booking page, when the page renders, then no calendar-sync status or confidence indicator of any kind is shown to the client.

**FEAT-03.SPEC-006-AC-11:** Given Support (Platform Operator) opens a Pro's account to troubleshoot a reported conflict, when Support views calendar-sync status, then Support sees connection health only (View-only), never full calendar event details.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 4 (2 screens; 1 N/A cell excluded) | 4 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Pro-Created Deposit Request Hold & Expiration

## Overview

**Name:** Pro-Created Deposit Request Hold & Expiration
**ID:** FEAT-03.SPEC-007
**Type:** Automation
**Purpose:** Reserves a slot the instant the Pro books a client in with a deposit request through FEAT-30, holding it up to 24 hours (platform parameter: `deposit-request-hold-max-hours`) or until 2 hours before the appointment (platform parameter: `deposit-request-hold-appointment-cutoff-hours`), whichever comes first, and expires the booking with a Pro notification if the deposit is never paid.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Creating a Slot Hold the instant the Pro books a client in and requests a deposit through FEAT-30
- Setting the hold's expiry to whichever comes first: platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment
- Re-validating the candidate slot against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception applied)
- Resolving a contested slot per FEAT-03.SPEC-005
- Expiring the hold and marking the associated Booking Expired (unpaid) -- this spec is the sole writer of that Booking transition -- and triggering the Pro notification, if the deposit is never paid

**Non-Goals:**
- The checkout hold class created when a client pays through the public booking flow -- owned by FEAT-03.SPEC-002/FEAT-03.SPEC-003, a distinct creation path and a distinct (fixed, few-minute) timeout, per the Entity-Lifecycle Coverage Matrix's "two creation paths, one Slot Hold shape" note
- Generating or sending the deposit-request link itself, or capturing the client's payment against it -- owned by FEAT-30 (Pro Booking Management) and FEAT-07 (Deposit Payment at Booking); this spec only reserves the time slot the request is for
- Composing or delivering the Pro's expiry notice content -- FEAT-08.SPEC-006 (Pro Attention Alert) is the content owner, with FEAT-30.SPEC-013 referencing it for dashboard surfacing; this spec is only the trigger and does not define their wording
- Writing the Booking's Expired (unpaid) state from any other spec -- FEAT-30.SPEC-010 (and FEAT-30.SPEC-013) rely on this spec's transition and never write it themselves (XBR-02 authority: FEAT-03)
- Recurring-occurrence release timing -- excluded per this Brief's Non-Goals: XBR-02 assigns an unpaid recurring occurrence's release at its cancellation cut-off to FEAT-21; this spec supplies only the generic hold/release mechanism FEAT-21 invokes

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro books a client in and creates a deposit request | FEAT-30 (Pro Booking Management) | Fires the instant Talia, at the chair or from her dashboard, books a client into a chosen slot and chooses to request a deposit rather than collect it in person | Service ID, chosen start time and duration, the Client reference, the appointment time the 2-hour cutoff is measured against |
| A Pro-created deposit-request hold reaches its expiry | (self -- schedule-based, derived from the hold's own expiry timestamp) | Fires the moment the hold's expiry timestamp passes with the hold still Active (deposit not paid) | The hold's service, start time, duration, owning Booking reference |

## Processing Logic

1. **Hold creation path:** Receive the candidate Service ID, start time, duration, and Client reference from FEAT-30's booking-in step.
2. Re-validate the candidate slot against FEAT-03.SPEC-004's rules, applying the Pro-only exception to minimum notice and booking horizon (the fit rule still applies unexempted).
3. Check for any existing active Slot Hold, Booking, Time Block, Recurring Series occurrence, or calendar busy period already covering the candidate time.
4. If no conflict exists, create a new Slot Hold: service, start time, duration, the owning Booking (created in Pending Payment state by FEAT-30), hold-created timestamp, and an expiry timestamp computed as the earlier of (creation time + platform parameter: `deposit-request-hold-max-hours`) and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`), state Active.
5. If a conflict exists, apply the contention resolution rule (FEAT-03.SPEC-005).
6. The created hold immediately excludes the candidate time from the next slot-list computation (FEAT-03.SPEC-001) for every client.
7. **Expiration path:** Identify each Active Pro-created deposit-request hold whose computed expiry timestamp has passed.
8. Confirm the hold has not already transitioned to Consumed by a completed deposit payment (FEAT-07) -- if it has, take no action.
9. Transition the hold's state to Expired and delete the hold record.
10. Mark the associated Booking's state as Expired (unpaid). This spec is the sole writer of the Booking -> Expired (unpaid) transition; FEAT-30.SPEC-010 does not repeat it and instead reads the resulting state, and no other automation may set it.
11. Trigger the Pro notification that the deposit was never paid: content is owned by FEAT-08.SPEC-006 (Pro Attention Alert, in-app and message) and surfaced on the dashboard via FEAT-30.SPEC-013; this spec supplies only the trigger and the Booking reference.
12. The freed time reappears the next time FEAT-03.SPEC-001 computes the open-slot list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deposit-request hold created | Candidate slot passes re-validation (with Pro exception) and no conflict exists | New Slot Hold created (Active); Booking created in Pending Payment state | Talia sees the booking on her dashboard as awaiting deposit payment; the client receives the deposit-request link (FEAT-30, FEAT-08) | FEAT-30, FEAT-03.SPEC-001 |
| Slot contested at creation | Another hold, Booking, Time Block, or calendar event already occupies the candidate time | None | Talia sees the slot is no longer available and is returned to a refreshed live view, per FEAT-03.SPEC-005 | FEAT-03.SPEC-005, FEAT-30 |
| Deposit paid before expiry | Client completes the deposit payment while the hold is still Active | Hold transitioned to Consumed; Booking transitioned to Confirmed (FEAT-07) | Talia and the client both see the booking confirmed | FEAT-07, FEAT-30 |
| Hold expired -- deposit never paid | The computed expiry timestamp passes with the hold still Active | Hold transitioned to Expired then deleted; Booking transitioned to Expired (unpaid), written only by this spec | Talia is notified the deposit was never paid, with content owned by FEAT-08.SPEC-006 and surfaced via FEAT-30.SPEC-013 (dashboard); the client receives no further reminder for this booking | FEAT-30.SPEC-010, FEAT-30.SPEC-013, FEAT-08.SPEC-006, FEAT-03.SPEC-001 |
| Hold creation failure | The hold cannot be written (e.g., a processing error) | None | Talia sees a retry prompt on FEAT-30; the deposit-request link is not sent until a confirmed hold exists | FEAT-30 |

## Data Model

**Reads:** Service (duration, buffer_override), Availability Rule (for re-validation, notice/horizon exempted for Pro requests), Booking, Time Block, Recurring Series, existing Slot Hold records, Calendar Connection busy periods (via FEAT-03.SPEC-006).
**Creates:** Slot Hold -- service, start time, duration, owning Booking reference, hold-created timestamp, computed expiry timestamp, state Active.
**Updates:** Slot Hold -- state transitioned to Expired on timeout, or Consumed on payment completion (by FEAT-07). Booking -- state transitioned to Expired on hold expiration.
**Deletes:** Slot Hold -- the expired record is removed, per the Entity-Lifecycle Coverage Matrix (no restore path; no retention).

## Business Rules

- The Pro-created deposit-request hold's expiry is computed as the earlier of two limits: platform parameter: `deposit-request-hold-max-hours` from creation, or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment (XBR-02) -- whichever boundary is reached first governs.
- The Pro-only exception to minimum notice and booking horizon applies to this hold's creation (FEAT-03.SPEC-004), since the triggering action is always a Pro-side booking through FEAT-30; the fit rule (duration+buffer) is never exempted.
- The first committed action wins a race between an expiring deposit-request hold and a completing deposit payment (XBR-01, FEAT-03.SPEC-005): if payment completes before this automation processes the expiry, the hold is Consumed and the Booking Confirmed, not Expired.
- An expired deposit-request hold's associated Booking is marked Expired (unpaid), never silently deleted -- this preserves the record that a booking was attempted and lapsed, distinct from the transient hold itself which is deleted (Entity-Lifecycle Coverage Matrix).
- This spec is the sole writer of Booking -> Expired (unpaid) for a Pro-created deposit request (XBR-02 authority: FEAT-03); FEAT-30.SPEC-010 relies on this transition and does not perform it.
- The Pro is always notified on expiration through two channels (FEAT-30 dashboard, FEAT-08 message) -- never silently, since this is money the Pro was counting on that never arrived. FEAT-08.SPEC-006 owns the notice content; this spec only triggers it.
- The hold-window values (platform parameter: `deposit-request-hold-max-hours`, platform parameter: `deposit-request-hold-appointment-cutoff-hours`) and the Pro-only notice/horizon exception are stated by FEAT-30.SPEC-006 (Pro Booking Action Rules); this spec consumes them and computes and enforces the expiry.

## Edge Cases

- **Deposit payment completes in the same instant the hold's computed expiry elapses** -- The completed-payment transition (to Consumed, Booking Confirmed) takes precedence; this automation detects the already-Consumed state before processing the expiry and takes no further action.
- **The appointment is scheduled less than platform parameter: `deposit-request-hold-appointment-cutoff-hours` away at the moment of creation** -- The 2-hour-before-appointment limit is already closer than the 24-hour cap, so the hold's expiry is set to that earlier boundary immediately; if the appointment is itself less than the cutoff away from the current moment, the hold's effective window is correspondingly short, and Talia is not blocked from creating it (the Pro-only notice exception applies to hold creation, not to how soon the resulting hold itself may then expire).
- **Talia cancels the booking herself before the deposit is paid or the hold expires** -- The cancellation (via FEAT-30) transitions the Booking and deletes the hold directly, outside this automation's own expiration path; no expiration notification fires for a Pro-initiated cancellation.
- **Concurrent trigger firing (Talia books two different clients into two different slots at effectively the same time)** -- Each hold-creation attempt is processed independently against its own distinct candidate slot; no interference occurs since the slots do not overlap.
- **Trigger fires while a previous hold-creation attempt for the same booking-in action is still in flight** -- FEAT-30's booking-in control is disabled during submission, preventing a duplicate hold-creation request for the same client and slot.
- **A second Pro-created deposit request is attempted for the same slot after the first hold already exists** -- The second attempt is refused per FEAT-03.SPEC-005, since the first hold committed first; Talia is shown the slot is already held.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The created hold immediately excludes the slot from the next computation; an expired hold's slot reappears |
| FEAT-03.SPEC-004 (Slot Validation & Timing Rules) | References (outbound) | Candidate slot is re-validated at hold-creation time, with the Pro-only notice/horizon exception applied |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (outbound) | Governs the outcome when the candidate slot is contested |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) / Affects (outbound) | The booking-in-with-deposit-request action triggers hold creation; this spec's expiration path is the sole writer of Booking -> Expired (unpaid), and FEAT-30.SPEC-010 relies on that transition and reflects it on the Pro's dashboard |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) -- within FEAT-30 | References (inbound) | States the hold-window values and the Pro-only notice/horizon exception this spec applies when creating and expiring the hold |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) -- within FEAT-30 | Affects (outbound) | Surfaces the expiry notice on the Pro's dashboard; references FEAT-08.SPEC-006 and defines no duplicate content |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Content owner of the Pro's unpaid-deposit expiry notice; this spec is its trigger when a hold expires unpaid |
| FEAT-07 (Deposit Payment at Booking) | Affects (outbound) / Triggered by (inbound, on payment completion) | Converts the hold to Consumed and the Booking to Confirmed when the deposit is paid |
| FEAT-08 (Automated Booking Messaging) | Triggers (outbound) | An expired, unpaid deposit-request hold triggers the Pro notification message (content per FEAT-08.SPEC-006) |

## Analytics and Success Signals

- **slot_held** (service_id, hold_type: pro_deposit_request, start_time) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **slot_hold_expired** (service_id, hold_type: pro_deposit_request) -- supports success-metrics.md: "Pro Change Correctness" (the target's "at least 70% of deposit requests the pro sends when rebooking at the chair are paid before the hold expires" is directly measured by the paid-vs-expired split of this event alongside the Consumed outcome)
- **deposit_request_hold_creation_failed** (service_id, reason category) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **deposit_request_hold_contested** (service_id, outcome: won / lost) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given Talia books a client in at the chair and requests a deposit, when the hold-creation step runs, then a Slot Hold is created instantly, excluding the slot from every client's computed list.

**FEAT-03.SPEC-007-AC-02:** Given Talia creates a deposit-request hold for an appointment more than 24 hours away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-max-hours` from creation, since that boundary is reached first.

**FEAT-03.SPEC-007-AC-03:** Given Talia creates a deposit-request hold for an appointment less than platform parameter: `deposit-request-hold-max-hours` away, when the hold's expiry is computed, then it is set to platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment, since that boundary is reached first.

**FEAT-03.SPEC-007-AC-04:** Given Talia books a client in for a time that would be inside her own minimum_booking_notice for a client, when the hold-creation step re-validates the candidate, then the candidate is not excluded on notice grounds, per the Pro-only exception.

**FEAT-03.SPEC-007-AC-05:** Given a deposit-request hold's computed expiry passes with the deposit never paid, when the expiration automation runs, then the hold is expired and deleted, the associated Booking is marked Expired (unpaid) by this automation alone (no other spec writes that transition), and Talia is notified via her dashboard (FEAT-30.SPEC-013) and a message whose content is owned by FEAT-08.SPEC-006.

**FEAT-03.SPEC-007-AC-06:** Given the client completes the deposit payment before the hold's expiry, when payment completes, then the hold is transitioned to Consumed and the Booking to Confirmed, and no expiration notification fires.

**FEAT-03.SPEC-007-AC-07:** Given a deposit-request hold's expiry and a completing payment occur at effectively the same moment, when both are evaluated, then the completed payment takes precedence and the hold is Consumed, not Expired.

**FEAT-03.SPEC-007-AC-08:** Given Talia attempts to create a second deposit-request hold on a slot already held by a first deposit request, when the second attempt is evaluated, then it is refused per FEAT-03.SPEC-005 and Talia sees the slot is already held.

**FEAT-03.SPEC-007-AC-09:** Given Talia cancels a booking herself before its deposit-request hold expires, when the cancellation completes, then the hold is deleted directly by that cancellation, and no expiration notification fires.

**FEAT-03.SPEC-007-AC-10:** Given a deposit-request hold cannot be created due to a processing error, when Talia attempts to book a client in with a deposit request, then Talia sees a retry prompt and no deposit-request link is sent.

**FEAT-03.SPEC-007-AC-11:** Given a deposit-request hold expires, when Riley next requests the same service's slot list, then the freed time appears as open.

**FEAT-03.SPEC-007-AC-12:** Given Talia books two different clients into two different, non-overlapping slots at effectively the same time, when both hold-creation attempts run, then each succeeds independently with no interference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


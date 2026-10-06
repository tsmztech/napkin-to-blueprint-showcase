---
document_type: feature-overview
feature_number: FEAT-03
feature_name: Real-Time Slot Availability Engine
feature_slug: real-time-slot-availability-engine
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 0
automation_count: 4
logic_rule_count: 2
integration_count: 1
notification_count: 0
---

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

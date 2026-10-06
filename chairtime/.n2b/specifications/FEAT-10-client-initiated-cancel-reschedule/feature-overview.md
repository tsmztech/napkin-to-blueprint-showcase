---
document_type: feature-overview
feature_number: FEAT-10
feature_name: Client-Initiated Cancel/Reschedule
feature_slug: client-initiated-cancel-reschedule
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 3
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

# Feature Breakdown Brief: Client-Initiated Cancel/Reschedule

## Summary

**Feature:** Client-Initiated Cancel/Reschedule
**ID:** FEAT-10
**Description:** A client can cancel or reschedule their own booking, within the Pro's stated policy, without a phone call or a DM -- from the reminder's one-tap option or the client's own booking-management link.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Target Users & Roles states the Client "reschedules or cancels within the policy window" as a defined capability, and the Vision's reminder flow ("I need to reschedule") depends on it existing. Research-informed market validation adds that self-service client booking meaningfully reduces phone and DM interruptions during the workday (Capterra/SoftwareAdvice-aggregated reviews, MEDIUM confidence).

**Key Capabilities:**
- Cancel an upcoming booking and see the deposit outcome before confirming
- Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy
- See the applicable cancellation window countdown before acting

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-10.SPEC-001 | Cancel Booking | Screen | Client | Client views the cancellation window countdown and deposit outcome for their own upcoming booking and confirms or backs out of cancelling it |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | Screen | Client | Client picks a new, genuinely free time for the same service from the same live slot list a fresh booking would use |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | Screen | Client | Client sees the deposit outcome for the chosen new time (carried-over deposit, or late-reschedule deposit-kept-plus-new-deposit-needed) before confirming |
| FEAT-10.SPEC-004 | Booking Update Commit | Automation | Client, Pro, Support | Commits the client's cancellation or reschedule to the booking, coordinating the deposit outcome, calendar mirroring, activity logging, and freed-slot handoff this triggers in other features |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | Logic/Rule | Client | Governs whether a booking is currently eligible to be cancelled or rescheduled by its client and computes the window countdown that determines which deposit-outcome branch applies |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | Notification | Client, Pro | Sends the client a cancellation or reschedule confirmation and the Pro a change notice once the update commits |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Cancel an upcoming booking and see the deposit outcome before confirming | FEAT-10.SPEC-001 | Primary purpose of the Cancel Booking screen, using SPEC-005's window calculation and FEAT-09's outcome computation for the preview | Phase 2 (Explicit) |
| Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy | FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Select New Time applies the same slot validation as a new booking (FEAT-03); Outcome & Confirm shows the carry-over (outside window) or new-deposit-needed (inside window) result before committing | Phase 2 (Explicit) |
| See the applicable cancellation window countdown before acting | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-005 | Countdown is computed by the eligibility rule and surfaced on both the Cancel screen and the Reschedule outcome screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-10.SPEC-004 | Booking Update Commit | Phase 4 (Trigger-Response) | Confirming a cancel or reschedule triggers cross-entity, cross-feature processing (deposit outcome, calendar mirror, activity log, freed-slot/waitlist handoff, notification) that exceeds a simple inline data write -- the Booking entity's High contention and reject-with-refresh resolution (dependency map) also requires a dedicated processing spec |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | Phase 5 (Rule Discovery) | The Validation & Limits field ("a booking already marked completed or no-show cannot be cancelled or rescheduled") and the States field's window-countdown behavior are rules shared across all three screens -- a rule referenced by multiple specs crosses the standalone threshold |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | Phase 4 (Notification surfacing) | The Communications field names a two-audience message (client confirmation, Pro change notice) sent through FEAT-08's messaging mechanism with real audience and content rules -- not a bare success toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Booking**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-10.SPEC-004 (late reschedule only) | Bookings are otherwise created by FEAT-05, FEAT-21, and FEAT-30. FEAT-10.SPEC-004 creates one new Booking record, only on an inside-window (late) reschedule, for the chosen new time (same service, duration, and client, freshly acknowledged policy version, flagged as requiring its own deposit via FEAT-07); cancellations and outside-window reschedules create nothing | Created atomically with the original's move to Rescheduled; an unpaid new Booking follows FEAT-03's standard hold/expiry rules (XBR-02) |
| Read (single) | FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003 | Each screen loads the one booking the client is acting on (already identified via FEAT-06's access link) to display its current time, service, and policy | -- |
| Read (list) | N/A | Browsing multiple bookings is FEAT-06's My Bookings List; this feature always acts on one already-identified booking | -- |
| Update | FEAT-10.SPEC-004 | Writes the cancellation state (cancel), the new start_time in place on the same record (outside-window reschedule), or the Rescheduled state on the original record (late reschedule), plus the cancellation/reschedule timestamp, once the client confirms | -- |
| Delete/Archive | N/A | Bookings are never deleted by this feature or any feature -- kept for the life of the account per SC-22; a cancelled or rescheduled booking remains as history, never removed. Recorded as an explicit non-goal below rather than a silent gap | -- |
| State Transition | FEAT-10.SPEC-004 | Confirmed/Awaiting Outcome -> Cancelled by Client (cancel); Confirmed/Awaiting Outcome -> Rescheduled (terminal) on the original Booking for an inside-window (late) reschedule, with a new Booking created at the new time; an outside-window reschedule updates start_time in place with the state unchanged. All governed by FEAT-10.SPEC-005's eligibility gate and the Booking entity's reject-with-refresh contention rule (dependency map) | Resolves the earlier flagged discrepancy to match FEAT-10.SPEC-004: the late reschedule is a compound write (original to Rescheduled plus new Booking) that succeeds or fails as one unit. The dependency map's Booking Creators line still omits FEAT-10 (map-level delta carried to Stage 4, SG-01) |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deposit Transaction | FEAT-10.SPEC-001, FEAT-10.SPEC-003 | Read for the deposit outcome preview shown before the client confirms; the outcome itself is derived and applied by FEAT-09, never computed by this feature |
| Cancellation Policy | FEAT-10.SPEC-001, FEAT-10.SPEC-003, FEAT-10.SPEC-005 | Read for the window_hours value (countdown calculation) and the plain-language wording shown to the client; this feature never edits the policy |
| Access Link | FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003 | The client's identity and booking scope are already validated by FEAT-06 before any of this feature's screens load; this feature performs no independent link validation |
| Availability Rule | FEAT-10.SPEC-002 | Read indirectly through FEAT-03's live slot computation when re-checking availability for a reschedule -- this feature never reads the rule directly |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client opens Cancel Booking or Reschedule Outcome & Confirm | Compute the cancellation window countdown and eligibility (booking not Completed/No-Show) | Standalone Logic/Rule | FEAT-10.SPEC-005 |
| Client opens Cancel Booking or Reschedule Outcome & Confirm | Read the current deposit outcome preview from FEAT-09 for display | Cross-feature -- logged in touchpoints | FEAT-09 responsibility |
| Client picks a new time on Select New Time | Re-validate the chosen slot against live availability (full duration plus buffer, no conflicts) | Cross-feature -- logged in touchpoints | FEAT-03 responsibility (XBR-01) |
| Client's first-choice new time disappears before they confirm it | Show a plain "that time was just taken" message and keep the client on the same live slot list | Inline in triggering screen | FEAT-10.SPEC-002 |
| Client confirms a reschedule outside the cancellation window | Deposit carries over to the new time; no new deposit charge | Inline in triggering screen (uses SPEC-005's window result) | FEAT-10.SPEC-003 |
| Client confirms a reschedule inside the cancellation window | Show plainly that the original deposit is kept and a new deposit is needed for the new time, before confirming | Inline in triggering screen, then handed to FEAT-07 for the new deposit charge | FEAT-10.SPEC-003 |
| Client backs out of a late-reschedule confirmation | Nothing changes; client returns to slot selection | Inline in triggering screen | FEAT-10.SPEC-003 |
| Client confirms cancel or reschedule | Commit the state transition on Booking; for a late reschedule, move the original to Rescheduled and create a new Booking for the new time as one atomic write | Standalone Automation | FEAT-10.SPEC-004 |
| Booking Update Commit succeeds | Apply the deposit outcome (refund, forfeiture, or late-reschedule new-deposit determination) | Cross-feature -- logged in touchpoints | FEAT-09 responsibility (XBR-09) |
| Booking Update Commit succeeds | Move or remove the entry on the Pro's personal calendar | Cross-feature -- logged in touchpoints | FEAT-04 responsibility (XBR-13) |
| Booking Update Commit succeeds | Write an append-only activity event | Cross-feature -- logged in touchpoints | FEAT-16 responsibility (XBR-21) |
| Booking Update Commit succeeds (cancellation) | Freed slot becomes publicly bookable immediately; matching waitlisted clients notified first with a 30-minute priority window | Cross-feature -- logged in touchpoints | FEAT-20 responsibility (XBR-28) |
| Booking Update Commit succeeds | Trigger the client confirmation and the Pro's change notice | Standalone Notification | FEAT-10.SPEC-006 |
| Booking Update Commit fails to save | Original booking remains untouched and intact; client sees an error and can retry | Standalone Automation (failure handling) | FEAT-10.SPEC-004 |
| A Pro-side action (FEAT-30) commits a conflicting transition first | Reject-with-refresh: the client is shown the booking's current state and must re-decide; the two transitions are never merged | Standalone Automation (failure handling) | FEAT-10.SPEC-004 |
| Client attempts to act on a booking already Completed or No-Show | Action is blocked; client sees a plain ineligibility message | Standalone Logic/Rule | FEAT-10.SPEC-005 |
| Client attempts any action while offline or connectivity drops | Plain message that connectivity is required; nothing is submitted | Inline in triggering screen | FEAT-10.SPEC-001 / SPEC-002 / SPEC-003 |

## Shared Context

**Shared Entities:**
- Booking -- read by SPEC-001, SPEC-002, and SPEC-003; updated by SPEC-004, which also creates a new Booking on a late reschedule (original goes to Rescheduled). Fields in scope here: state, start_time, policy_version, cancellation/reschedule timestamps.
- Deposit Transaction (read-only) -- read by SPEC-001 and SPEC-003 for the outcome preview; the authoritative outcome value and its application are owned by FEAT-09.
- Cancellation Policy (read-only) -- read by SPEC-001, SPEC-003, and SPEC-005 for window_hours and plain_language_wording.
- Access Link (read-only, consumed not managed) -- the client's identity and booking scope arrive pre-validated from FEAT-06 into every screen in this feature.

**Shared UI Patterns:**
- "See the outcome before confirming" pattern -- SPEC-001 (cancellation) and SPEC-003 (late reschedule) both show the deposit consequence plainly, with an explicit confirm step and a no-penalty way to back out with nothing changed. Spec Writers for both should keep this pattern's plainness and ordering (outcome shown, then confirm) consistent, per the feature's Alternate flows.
- Live slot list reuse -- SPEC-002 presents the identical real-time slot list mechanism a fresh booking uses (FEAT-03), including the "just taken" recovery message, so a client never sees a stale or misleading option.

**Shared Validation:**
- FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) is referenced, not duplicated, by SPEC-001 (cancel eligibility and countdown), SPEC-002 (eligibility gate before offering reschedule), and SPEC-003 (which deposit-outcome branch applies).

## Internal Dependency Map

```
SPEC-001 (Cancel Booking) -> [checks eligibility and countdown using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-001 (Cancel Booking) -> [client confirms cancellation] -> SPEC-004 (Booking Update Commit)
SPEC-002 (Reschedule -- Select New Time) -> [checks eligibility using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-002 (Reschedule -- Select New Time) -> [client picks a new time] -> SPEC-003 (Reschedule -- Outcome & Confirm)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [determines outcome branch using] -> SPEC-005 (Cancellation Window & Eligibility Rule)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [client confirms reschedule] -> SPEC-004 (Booking Update Commit)
SPEC-003 (Reschedule -- Outcome & Confirm) -> [client backs out] -> SPEC-002 (Reschedule -- Select New Time) [nothing changed]
SPEC-004 (Booking Update Commit) -> [commit succeeds] -> SPEC-006 (Cancellation/Reschedule Notification)
SPEC-004 (Booking Update Commit) -> [commit fails or a conflicting transition wins] -> SPEC-001 (Cancel Booking) / SPEC-003 (Reschedule -- Outcome & Confirm) [original or current state re-shown]
```

**Default Entry:** This feature has no single default landing screen -- the client arrives already viewing one specific booking (via FEAT-06.SPEC-004, Booking Detail) and chooses Cancel (routes to SPEC-001) or Reschedule (routes to SPEC-002); a reminder's one-tap "I need to reschedule" option (FEAT-08) routes directly into SPEC-002, bypassing the choice.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-10.SPEC-001 / SPEC-002 | Inbound | FEAT-06 (Client Booking Identity) | Client, already viewing their own booking, chooses to cancel or reschedule it | Client chooses cancel or reschedule on Booking Detail |
| FEAT-10.SPEC-002 | Inbound | FEAT-08 (Automated Booking Messaging) | Client taps the reminder's "I need to reschedule" one-tap option | Client taps reminder action |
| FEAT-10.SPEC-002 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Reschedule requests and re-validates the live slot list for the same service | Client opens Select New Time / confirms a slot |
| FEAT-10.SPEC-001 / SPEC-003 / SPEC-005 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Reads the cancellation policy window and the derived deposit outcome for preview and eligibility | Client opens Cancel or Reschedule Outcome & Confirm |
| FEAT-10.SPEC-004 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | Hands off deposit outcome application (refund, forfeiture, or late-reschedule new-deposit determination) | Booking update commits |
| FEAT-10.SPEC-004 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Commit causes the Pro's personal calendar entry to move or be removed | Booking cancelled or rescheduled |
| FEAT-10.SPEC-004 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Commit writes an append-only activity event | Booking cancelled or rescheduled |
| FEAT-10.SPEC-004 | Outbound | FEAT-20 (Waitlist for Cancelled Slots) | A cancellation frees the slot and triggers waitlist priority notification before general availability | Booking cancelled |
| FEAT-10.SPEC-003 / SPEC-004 | Outbound | FEAT-07 (Deposit Payment at Booking) | A late reschedule (inside the window) creates a new Booking (SPEC-004) that requires a new deposit charge for the new time, handled by the standard deposit-payment mechanism | Client confirms a late reschedule |
| FEAT-10.SPEC-006 | Outbound | FEAT-08 (Automated Booking Messaging) | Notification content is delivered through FEAT-08's transactional text/email messaging capability, which owns the delivery contract | Booking Update Commit succeeds |

## Non-Functional Notes

**Data volumes / growth:** Cancel/reschedule volume is a subset of overall booking traffic -- a few hundred Pros in year one, each with roughly 100-500 clients and 20-40 bookings a week (scope-boundaries SC-19). The Self-Service Reschedule Rate success metric targets at least 85% of client cancellations/reschedules completing entirely through this feature, so it should be expected to carry a meaningful share of weekly booking-adjacent traffic per Pro, not an edge case.

**Responsiveness:** Re-checking live availability for a reschedule follows the product-wide slot-search responsiveness bar -- available slots appear within roughly one second of selection and the list updates within roughly one second of a slot being taken (Slot Search Responsiveness success metric; ASMP-21). A full booking-equivalent action here should complete without a perceptible wait once the client confirms, consistent with ASMP-21's under-one-minute booking benchmark.

**Data sensitivity / privacy:** The booking, deposit outcome, and policy data this feature displays are personal and financial data linked to an identifiable client, visible only to that client and their one Pro (ASMP-23); Platform Operator (Support) has view-only access to the resulting state and never uses or bypasses the client's own access link to reach it.

**Compliance flags:** N/A -- this feature applies no compliance regime of its own; the money movement it triggers (refund, forfeiture, or a new deposit charge) is a category-level payment-processing capability (ASMP-31) whose contract and any related compliance handling belong to FEAT-07 and FEAT-09, not to this feature.

**Signals:** This feature emits booking_cancelled_by_client and booking_rescheduled_by_client on Booking Update Commit (FEAT-10.SPEC-004), and late_reschedule_warning_shown when the Outcome & Confirm screen (FEAT-10.SPEC-003) displays the inside-window, new-deposit-needed warning -- these three signals are the analytics basis for the Self-Service Reschedule Rate success metric.

## Non-Goals

- **Pro-initiated cancellation or reschedule** -- Excluded per the feature entry's own MODIFIED note: the Pro's own cancel/reschedule actions were moved to Pro Booking Management (FEAT-30) so client-side and Pro-side deposit rules are each owned by one feature; this feature covers only the client-initiated path.
- **Partial refunds or tiered cancellation schedules** -- Excluded per scope-boundaries SC-18: BRIEF.md's Business Context defines a binary deposit rule (kept inside the window, refunded outside it); this feature's outcome preview and commit never expose a percentage-based or tiered outcome.
- **Charging a client's card later, or keeping one on file, for a cancellation fee** -- Excluded per scope-boundaries SC-13: the product protects the Pro with a deposit paid up front; when a late reschedule needs a new deposit, that charge goes through the same standard deposit-payment mechanism (FEAT-07) as any other booking, never a stored-card charge made after the fact.
- **Automatic purge or deletion of cancelled/rescheduled booking history** -- Intentional lifecycle decision surfaced by the CRUD matrix: bookings are retained for the life of the account per SC-22, so a cancelled or rescheduled booking is never deleted, only left as history with an updated state.
- **Support acting on a client's or Pro's behalf to cancel or reschedule** -- Excluded per scope-boundaries SC-05 and the Access Matrix: Platform Operator (Support) has view-only access to the outcome and never performs the cancellation or reschedule itself, even to help resolve a support request.

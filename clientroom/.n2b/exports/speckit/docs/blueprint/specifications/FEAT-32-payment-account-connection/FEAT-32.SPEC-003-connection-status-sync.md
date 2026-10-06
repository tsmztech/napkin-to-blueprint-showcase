---
document_type: spec
spec_type: automation
spec_id: FEAT-32.SPEC-003
spec_name: Connection Status Sync
spec_slug: connection-status-sync
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Automation Spec: Connection Status Sync

## Overview

**Name:** Connection Status Sync
**ID:** FEAT-32.SPEC-003
**Type:** Automation
**Purpose:** Applies the processor-reported status (Connected, Needs attention, or a failed/abandoned attempt) and available payment methods to the Payment Account Connection record the instant it is reported.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Applying a readiness confirmation to the Payment Account Connection record
- Applying a restriction/needs-more-information report, including its specific reason
- Treating a hand-off that Nadia abandons through "Start over" (available once it is older than platform parameter: `payment-connect-handoff-timeout`) as an abandoned hand-off
- Leaving an existing record's status unchanged when a Reconnect hand-off fails or is abandoned, and removing the interim Connecting record when a first connect fails or is abandoned
- Treating a readiness report with zero available payment methods as Needs attention with a defined reason
- Resolving events that arrive out of order or twice for the same underlying change

**Non-Goals:**
- Initiating the hand-off itself, or defining what data is exchanged with the payment-processing capability -- owned by FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting); this automation only consumes events that spec already receives.
- Removing the connection reference -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account), which is triggered by Nadia's own explicit action, not by a processor report.
- Displaying the resulting status -- owned by FEAT-32.SPEC-001 (Payment Connection Screen), which reads this automation's output.
- Sending the confirmation or alert email -- owned by FEAT-32.SPEC-006 (Connection Status Notifications), which this automation triggers but does not compose.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection confirmed ready | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the payment-processing capability reports the account is ready, following a Connect or Reconnect hand-off (an empty methods list is handled as a restriction, step 3) | processor_account_reference, list of currently available payment methods (may be empty), event report time |
| Connection restricted / more information requested | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when the capability restricts the account or asks for more information, at any point after a hand-off began, including after a prior Connected state | Specific attention reason (plain-language text), event report time |
| Hand-off fails or is abandoned mid-flow | FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Fires when a Connect or Reconnect attempt does not complete, technically or because Nadia exits before it finishes | Event report time; no reason data required for this outcome |
| Hand-off timed out -- Nadia starts over | FEAT-32.SPEC-001 (Payment Connection Screen) | Fires when Nadia confirms "Start over" while the screen shows Connecting, i.e. no outcome was reported within platform parameter: `payment-connect-handoff-timeout` of `handoff_started_at` | Confirmation time; the record's `handoff_started_at` |

## Processing Logic

1. Receive the reported event and its data from FEAT-32.SPEC-002, including the event's report time.
2. If no Payment Account Connection record exists, discard the event without creating one and stop (Edge Cases: disconnected connection). Otherwise, for a readiness or restriction event, compare its report time against the record's `last_event_reported_at` (set to the hand-off start time when a hand-off begins, and to each applied event's report time):
   - If the event's report time is **older than** `last_event_reported_at`, discard it as stale (Edge Cases: out-of-order arrival) and stop.
   - If the event's report time is **equal to** `last_event_reported_at`, it is an already-applied duplicate: discard it as a no-op and stop -- no field changes and no notification fires (step 6 is never reached). The single exception is a restriction event whose report time equals that of an applied readiness event while `status` is Connected: the restriction is applied (restriction takes precedence over readiness at the same instant, since a needs-attention account cannot be relied on for payment).
   - Failure/abandonment events are not compared by report time; they are applied only while a hand-off is in progress (status Connecting or `handoff_started_at` present), and are a no-op otherwise (step 5).
3. If the event reports readiness: check the reported list of available payment methods.
   - If the list contains at least one method: set `status` to Connected, confirm or set `processor_account_reference`, set `available_payment_methods` to the reported list, clear `handoff_started_at`, and set `last_event_reported_at` to the event's report time.
   - If the list is empty (zero methods): treat the event as a restriction. Set `status` to Needs attention with the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", confirm or set `processor_account_reference`, set `available_payment_methods` to none, clear `handoff_started_at`, and set `last_event_reported_at`. Because this reason is supplied by the product, step 4's missing-reason check does not apply to this path.
4. If the event reports a restriction: check that a specific reason accompanies it. If no reason is present, treat the event as malformed, discard it, and leave the record unchanged (Edge Cases). If a reason is present, set `status` to Needs attention (retaining the reason for display), set `available_payment_methods` to none -- a needs-attention account cannot be relied on for payment -- clear `handoff_started_at`, and set `last_event_reported_at` to the event's report time.
5. If the event reports a failed or abandoned hand-off: if no hand-off is in progress, apply nothing and stop. Otherwise, if the record's `status` is Connecting (a first connect, with no earlier known-good state), delete the interim record so the screen returns to Empty and no half-created record remains; if the record has any other status (a Reconnect), clear only `handoff_started_at` and leave `status`, `processor_account_reference`, and `available_payment_methods` exactly as they were before the attempt.
5a. If the trigger is Nadia's confirmed "Start over" (FEAT-32.SPEC-001): if no hand-off is in progress (an outcome was applied first, or the record no longer exists), apply nothing and stop. If a hand-off is in progress but `handoff_started_at` is **not yet older than** platform parameter: `payment-connect-handoff-timeout` (the threshold was not actually reached), reject the request as a no-op and leave the record unchanged. Otherwise treat it exactly as an abandoned hand-off and apply step 5 (first connect: delete the interim record; Reconnect: clear only `handoff_started_at`). A processor event reported later for the abandoned hand-off is handled by step 2 (discarded for a deleted record; for a Reconnect, ordered by report time against `last_event_reported_at`, and applied only if it reports a genuinely later state).
6. When a change is applied (step 3 or 4, including the zero-methods path), trigger FEAT-32.SPEC-006 (Connection Status Notifications) for the resulting outcome. Steps that discard the event or apply no status change trigger nothing.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Connected | A readiness-confirmed event with at least one available method is applied | `status` set to Connected; `processor_account_reference` confirmed/set; `available_payment_methods` set from the report; `handoff_started_at` cleared; `last_event_reported_at` advanced | FEAT-32.SPEC-001 shows "Ready to accept payments" and the available methods | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 (pay-link availability) |
| Needs attention | A restriction event with a stated reason is applied | `status` set to Needs attention (carrying the reason); `available_payment_methods` cleared to none; `handoff_started_at` cleared; `last_event_reported_at` advanced | FEAT-32.SPEC-001 shows "Needs attention" with the reason verbatim, plus Reconnect, Disconnect, and Contact support | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 and FEAT-10 (pay-link marked unavailable) |
| Needs attention (zero methods) | A readiness-confirmed event reports zero available payment methods | Same data changes as Needs attention, with the product-defined reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect." and `processor_account_reference` confirmed/set | FEAT-32.SPEC-001 shows "Needs attention" with that reason; FEAT-32.SPEC-006's alert email carries the same text as `{attention_reason}` | FEAT-32.SPEC-001, FEAT-32.SPEC-006, FEAT-09 and FEAT-10 |
| Hand-off failed or abandoned | A failure/abandonment event arrives while a hand-off is in progress | First connect (`status` Connecting): the interim record is deleted, returning Nadia to Empty. Reconnect (any other status): only `handoff_started_at` is cleared; `status`, `processor_account_reference`, and `available_payment_methods` retain their prior values. With no hand-off in progress: no change | FEAT-32.SPEC-001 shows its Error state with Retry over the previous state (Empty for a first connect); no false Connected or Needs-attention flash | FEAT-32.SPEC-001 |
| Hand-off timed out, Nadia starts over | Nadia confirms "Start over" on FEAT-32.SPEC-001 and the hand-off is older than platform parameter: `payment-connect-handoff-timeout` with no outcome applied | Same data changes as Hand-off failed or abandoned: first connect deletes the interim record; Reconnect clears only `handoff_started_at`. If no hand-off is in progress, or the threshold has not elapsed: no change | FEAT-32.SPEC-001 shows its Error banner over the previous state (Empty for a first connect); no false Connected or Needs-attention flash. A no-op request leaves the screen on the state the record holds | FEAT-32.SPEC-001 |
| Malformed restriction event discarded | A restriction event arrives with no stated reason | None -- record unchanged | No user-visible feedback from this event; the previous state stands | FEAT-32.SPEC-001 (unaffected) |
| Stale or duplicate event discarded | An event's report time is older than, or equal to (except restriction-over-readiness), the record's `last_event_reported_at` | None -- no field changes and no notification fires | No user-visible feedback | FEAT-32.SPEC-001 (unaffected), FEAT-32.SPEC-006 (not triggered) |
| Automation failure | This automation cannot durably apply a genuinely valid, current event (an internal processing error) | None persisted on this attempt; the event is not silently dropped -- it is retried | FEAT-32.SPEC-001 continues showing the last durably-applied status; no false update is shown | FEAT-32.SPEC-001 |

## Data Model

**Reads:** Payment Account Connection -- the record's current `status`, `handoff_started_at`, and `last_event_reported_at` (the report time of the most recently applied event, or the hand-off start), to resolve ordering and duplicates.
**Creates:** None -- the record is first created with `status` Connecting by FEAT-32.SPEC-002 when Nadia confirms Continue on FEAT-32.SPEC-001's consent notice; this automation never creates the record itself.
**Updates:** Payment Account Connection -- `status`, `processor_account_reference`, `available_payment_methods`, `handoff_started_at`, `last_event_reported_at`, as defined in Outcome Definitions.
**Deletes:** Payment Account Connection -- only the interim Connecting record of a first connect whose hand-off failed or was abandoned (step 5), so no half-created record remains.

## Business Rules

- Processor-reported status is authoritative over any client-visible expectation or Nadia's own prior screen state (feature-dependency-map.md, Entity: Payment Account Connection, Contention).
- A failed or abandoned hand-off never overwrites the last known-good status (product-features.md, States field); the only state it removes is the interim Connecting record of a first connect, which never held a known-good status.
- An event whose report time equals the `last_event_reported_at` already applied is a duplicate and is a no-op (no field change, no email); the only equal-time exception is a restriction event over an applied readiness event, where the restriction wins.
- `available_payment_methods` is fully derived from the capability's latest report and is never set by direct user input.
- Events are applied strictly in the order the capability reports them occurring (event time), never in the order they happen to arrive.
- XBR-19: this automation's output is the sole source FEAT-09 and FEAT-10 read to derive pay-link availability -- this automation itself renders no pay-link copy.

## Edge Cases

- **Concurrent trigger firing (a readiness event and a restriction event both fire for the same connection at effectively the same time)** -- Both are applied in event-time order, not arrival order; whichever genuinely occurred later determines the final state, and the earlier one is superseded rather than lost (it was still validly applied first, then overwritten by the later, true state).
- **Trigger fires while a previous run is still in flight** -- A second event for the same connection record queues behind the first rather than writing concurrently; each is applied in turn, in event-time order, so the record never reflects a half-applied state.
- **A restriction event arrives with no stated reason** -- Treated as malformed and discarded; the record's previous status stands, since the product's Specific-reason display pattern (feature-overview.md, Shared UI Patterns) never falls back to a generic "something went wrong."
- **A processor-reported event arrives for a connection Nadia has since disconnected** -- The record no longer exists (FEAT-32.SPEC-004 hard-deletes it), so the event has nothing to update and is discarded without recreating a record.
- **Reconnect is initiated while a stale event from a prior, superseded attempt is still in flight** -- FEAT-32.SPEC-002 advances `last_event_reported_at` to the new hand-off's start time, so the stale event's older report time fails step 2's ordering check and is discarded; the new hand-off's outcome is unaffected.
- **The same event is delivered twice, or two events carry an identical report time** -- The second delivery has a report time equal to `last_event_reported_at`, so step 2 discards it as an already-applied duplicate: nothing changes and no second confirmation or alert email fires. If the two events are a readiness and a restriction with an identical report time, the restriction is applied and the readiness is discarded.
- **The processor never reports any outcome for a hand-off** -- The record stays Connecting (or Needs attention with `handoff_started_at` set, for a Reconnect) until Nadia confirms "Start over" on FEAT-32.SPEC-001 once the hand-off is older than platform parameter: `payment-connect-handoff-timeout`; step 5a then applies the abandoned-hand-off handling, so no connection can remain stuck indefinitely.
- **Nadia confirms "Start over" while the processor's outcome is being applied** -- Both are serialised behind the in-flight run; if the outcome is applied first, `handoff_started_at` is already cleared and step 5a stops with no change; if "Start over" is applied first, a later readiness or restriction event is handled by step 2 (discarded for a deleted first-connect record).
- **A readiness event reports zero available methods** -- Step 3 applies Needs attention with the product-defined reason rather than Connected, so Nadia sees a specific, actionable reason and never a "Connected" state with nothing payable.
- **This automation's own write fails on first attempt** -- Retried automatically at platform parameter: `connection-status-apply-retry-interval` intervals until it succeeds; there is no maximum retry count, since a permanently unapplied status update could leave Nadia believing she is ready to accept payments when she is not, or the reverse.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting) | Triggered by (inbound) | Every readiness, restriction, and failure/abandonment event fires this automation |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Nadia's confirmed "Start over" on a Connecting hand-off older than the timeout fires the abandoned-hand-off handling |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Displays the applied status, reason, and available methods |
| FEAT-32.SPEC-006 (Connection Status Notifications) | Triggers (outbound) | The Connected and Needs attention outcomes fire this notification |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Derives pay-link availability from this automation's applied status (via FEAT-09.SPEC-009) |
| FEAT-10 (Invoice Payment Processing) | Affects (outbound) | Reads the applied status and available methods before allowing a payment attempt |

## Analytics and Success Signals

- **payment_account_connected** (methods_count) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_needs_attention** (has_prior_connection: yes/no; reason_source: processor / zero_methods) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **connection_status_malformed_event_discarded** (event type: restriction_no_reason) -- N/A -- no Stage 2 metric measures malformed-event frequency; retained so a discarded processor report is observable rather than silently lost.
- **connection_handoff_start_over_applied** (attempt_type: connect/reconnect; result: applied / rejected_not_timed_out / no_handoff) -- N/A -- no Stage 2 metric measures stalled hand-off recovery; retained so timed-out hand-offs are observable.
- **connection_status_apply_failed** (retry_count) -- N/A -- no Stage 2 metric measures this automation's own reliability directly; retained for observability of the correctness-critical retry guarantee.

## Acceptance Criteria

**FEAT-32.SPEC-003-AC-01:** Given Nadia has initiated a Connect hand-off, when the capability reports readiness, then this automation sets `status` to Connected, sets `processor_account_reference`, and sets `available_payment_methods` from the report.

**FEAT-32.SPEC-003-AC-02:** Given a Connected account, when the capability reports a restriction with a stated reason, then this automation sets `status` to Needs attention carrying that reason and clears `available_payment_methods` to none.

**FEAT-32.SPEC-003-AC-03:** Given a Reconnect attempt on a Needs attention record fails or is abandoned, when this automation receives that outcome, then it applies no change to `status`, `processor_account_reference`, or `available_payment_methods` and only clears `handoff_started_at`.

**FEAT-32.SPEC-003-AC-04:** Given a restriction event arrives with no stated reason, when this automation evaluates it, then the event is discarded as malformed and the record's previous status stands.

**FEAT-32.SPEC-003-AC-05:** Given a readiness event and a restriction event arrive out of order, when this automation processes them, then it applies them in event-time order, not arrival order, so the true later state wins.

**FEAT-32.SPEC-003-AC-06:** Given Nadia has already disconnected her account, when a stale status-report event for the removed connection arrives, then this automation discards it without recreating a record.

**FEAT-32.SPEC-003-AC-07:** Given a Connected outcome is applied, when FEAT-32.SPEC-001 next loads, then it shows "Ready to accept payments" and the available methods, and FEAT-32.SPEC-006 fires the confirmation email.

**FEAT-32.SPEC-003-AC-08:** Given a Needs attention outcome is applied, when FEAT-32.SPEC-001 next loads, then it shows the specific reason verbatim, and FEAT-32.SPEC-006 fires the alert email.

**FEAT-32.SPEC-003-AC-09:** Given two events fire for the same connection at effectively the same time, when this automation processes them, then it applies them in event-time order and the record reflects the genuinely later state.

**FEAT-32.SPEC-003-AC-10:** Given an event fires while a previous run for the same connection is still in flight, when the second event arrives, then it queues and is applied afterward in event-time order rather than writing concurrently.

**FEAT-32.SPEC-003-AC-11:** Given the same readiness-confirmed event is delivered twice, when the second delivery arrives with a report time equal to the record's `last_event_reported_at`, then step 2 discards it as an already-applied duplicate, no field changes, and no second confirmation email fires.

**FEAT-32.SPEC-003-AC-12:** Given this automation's own write fails on first attempt, when the retry logic runs, then it retries at platform parameter: `connection-status-apply-retry-interval` intervals until it succeeds, with no maximum retry count.

**FEAT-32.SPEC-003-AC-13:** Given Reconnect is initiated while a stale event from a superseded attempt is still in flight, when the stale event arrives, then it is discarded and the new hand-off's own outcome is unaffected.

**FEAT-32.SPEC-003-AC-14:** Given a readiness-confirmed event reports zero available payment methods, when this automation evaluates it, then it sets `status` to Needs attention with the reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.", sets `available_payment_methods` to none, and triggers FEAT-32.SPEC-006's alert email carrying that reason.

**FEAT-32.SPEC-003-AC-15:** Given Nadia's first Connect hand-off (record status Connecting) fails or is abandoned, when this automation receives that outcome, then it deletes the interim record, FEAT-32.SPEC-001 shows the Error banner over the Empty state, and no record remains.

**FEAT-32.SPEC-003-AC-16:** Given a readiness event and a restriction event carry an identical report time, when both are processed, then the restriction is applied (or remains applied) and the readiness event is discarded.

**FEAT-32.SPEC-003-AC-17:** Given a duplicate event is discarded at step 2, when discarding completes, then FEAT-32.SPEC-006 is not triggered.

**FEAT-32.SPEC-003-AC-18:** Given Nadia's first-connect hand-off (record status Connecting) is older than platform parameter: `payment-connect-handoff-timeout` with no outcome reported, when she confirms "Start over" on FEAT-32.SPEC-001, then this automation deletes the interim record, FEAT-32.SPEC-001 shows the Error banner over the Empty state, and no record remains; and given a Reconnect hand-off in the same condition, then it clears only `handoff_started_at` and the record keeps its Needs attention `status`, `processor_account_reference`, and `available_payment_methods`.

**FEAT-32.SPEC-003-AC-19:** Given a "Start over" request arrives when no hand-off is in progress (an outcome was already applied or the record was removed) or when `handoff_started_at` is not yet older than platform parameter: `payment-connect-handoff-timeout`, when this automation evaluates it at step 5a, then it makes no change to the record and triggers no notification.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 8 | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 10 | 10 |

---
document_type: spec
spec_type: automation
spec_id: FEAT-20.SPEC-005
spec_name: Cancellation-Triggered Waitlist Matching
spec_slug: cancellation-triggered-waitlist-matching
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Cancellation-Triggered Waitlist Matching

## Overview

**Name:** Cancellation-Triggered Waitlist Matching
**ID:** FEAT-20.SPEC-005
**Type:** Automation
**Purpose:** On a freed-slot signal from a cancellation, finds every matching Requested entry, transitions each to Notified, and hands off to the opening notification.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Receiving a cancellation-sourced freed-slot signal from FEAT-10 (client cancellation) or FEAT-30 (Pro cancellation)
- Applying the matching test (FEAT-20.SPEC-004) to find every eligible Requested entry
- Transitioning every matched entry to Notified and setting its claim_deadline
- Handing off to FEAT-20.SPEC-008 for the opening notification

**Non-Goals:**
- Treating a reschedule's vacated original time as a freed slot -- excluded per this Brief's recorded reading of XBR-28's "freed by a cancellation" wording and FEAT-10's own Side-Effect Inventory, which scopes its waitlist hand-off specifically to cancellation; this automation is never triggered by a reschedule
- Defining the matching test itself, the claim window, or contention resolution -- owned by FEAT-20.SPEC-004; this automation applies that spec's rules, it does not define them
- Sending the notification content -- owned by FEAT-20.SPEC-008; this automation only triggers it once entries are Notified

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A client cancellation commits | FEAT-10.SPEC-004 (Booking Update Commit) | Fires only for the Cancellation committed outcome (never a reschedule outcome, per this Brief's recorded reading) | Freed slot's service, date, and start time |
| A Pro single cancellation commits | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Fires only for a cancel outcome (never the reschedule branch) | Freed slot's service, date, and start time |
| A Pro bulk cancellation commits | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Fires once per booking cancelled in the bulk action | Freed slot's service, date, and start time, per affected booking |

## Processing Logic

1. Receive the freed-slot signal (service, date, start time) from the triggering cancellation commit.
2. Apply the matching test (FEAT-20.SPEC-004): find every Waitlist Entry with this Pro in state Requested whose service equals the freed slot's service and whose [start_date, end_date] range includes the freed slot's date.
3. If zero entries match, take no further action -- the slot proceeds directly to general public availability with no priority window (FEAT-20.SPEC-004's zero-match rule).
4. If one or more entries match, transition each matched entry's state to Notified and set its claim_deadline to the current moment plus platform parameter: `waitlist-claim-window-minutes`, simultaneously for every matched entry.
5. Trigger FEAT-20.SPEC-008 (Waitlist Opening Notification) once per matched entry, carrying the freed slot's details and that entry's claim_deadline.
6. Report completion; no data is returned to the triggering cancellation commit beyond acknowledgment, since FEAT-10.SPEC-004 and FEAT-30.SPEC-007/SPEC-008 do not wait on this automation's outcome to complete their own commit.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No match found | Zero Requested entries match the freed slot | None | None -- the slot simply becomes generally available per FEAT-03/FEAT-05's ordinary path | FEAT-03, FEAT-05 |
| One or more entries matched and notified | 1+ Requested entries match | Each matched entry: state -> Notified, claim_deadline set | Each matched client receives the opening notification (FEAT-20.SPEC-008) | FEAT-20.SPEC-008, FEAT-20.SPEC-002 (status updates on next load) |
| Matching runs but the notification hand-off fails for one or more matched entries | The state transition to Notified succeeds but FEAT-20.SPEC-008 cannot be triggered for one or more of the matched entries (a transient failure) | The affected entries remain Notified with claim_deadline already set | The affected client sees no notification arrive, but her entry still shows as Notified with a countdown on FEAT-20.SPEC-002 if she checks that screen directly; the claim link itself is only ever delivered by the notification, so a client who never receives it cannot claim within the window through this path | FEAT-20.SPEC-002, FEAT-20.SPEC-008 |
| Automation failure (the matching step itself cannot run) | A transient failure prevents evaluating the freed-slot signal at all | No entry is transitioned | No client is notified for this opening; the slot proceeds to general availability once FEAT-03's own timing allows it, exactly as if zero entries had matched -- this is a silent miss from the waitlist's perspective, not a blocking failure of the triggering cancellation | FEAT-03, FEAT-05 |

## Data Model

**Reads:** Waitlist Entry -- service, start_date, end_date, state, scoped to this Pro; Booking -- the freed slot's service, date, and start time from the triggering cancellation.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Requested -> Notified) and claim_deadline, for every matched entry.
**Deletes:** None.

## Business Rules

- XBR-28 governs this automation's entire purpose: matching waitlisted clients are notified first, with platform parameter: `waitlist-claim-window-minutes` of priority before the slot returns to general availability.
- This automation is triggered only by a cancellation-sourced freed-slot signal (FEAT-10.SPEC-004's Cancellation committed outcome, or FEAT-30.SPEC-007/SPEC-008's cancel outcomes) -- never by a reschedule, per this Brief's recorded reading of XBR-28 and FEAT-10's own scoping.
- Notification is simultaneous across every matched entry -- there is no queue or sequential notification order (this Brief's Non-Goals).
- This automation's own responsibility ends once matched entries are transitioned to Notified and the notification is triggered; it does not itself decide who wins a subsequently contested claim (FEAT-20.SPEC-006 and FEAT-20.SPEC-004 own that).
- Platform Operator (Support) can view every transition this automation writes (Requested -> Notified, and the freed-slot signal that produced it) through FEAT-19's read-only account view, per XBR-24 -- Support never triggers, delays, or overrides a match.

## Edge Cases

- **A single Pro bulk cancellation frees several slots at once (FEAT-30.SPEC-008)** -- Each freed booking's slot is processed as its own independent freed-slot signal, with its own matching pass, its own set of Notified entries (if any), and its own claim windows; slots freed in the same bulk action never share a claim window or notification batch.
- **Two cancellations for the same service and overlapping dates commit at effectively the same time** -- Each freed-slot signal is processed independently; if both signals could match the same waitlisted entry (a range join covering both freed dates), that entry is Notified once per matching freed slot it actually satisfies, since a Requested entry can only be matched while it remains in Requested state -- the moment it is Notified for the first freed slot, it is no longer eligible to be Notified again for a second freed slot until it resolves (converts, expires, or the client leaves and rejoins).
- **A Requested entry's date range covers a freed slot, but the entry converts or expires between the freed-slot signal arriving and this automation's matching step actually running** -- Not possible in practice: an entry only transitions to Notified through this automation itself, so at the moment matching evaluates state, an entry still in Requested is genuinely eligible; a stale read is avoided by evaluating state fresh at trigger time.
- **The notification hand-off (step 5) fails for one matched entry among several** -- The other matched entries' notifications proceed independently; only the failed entry's client experiences the "Notified but never notified" gap described in the Outcome Definitions above.
- **Concurrent trigger firing (two different bookings, for two different services, are cancelled at effectively the same time)** -- Each freed-slot signal runs its own independent matching pass; neither is delayed by the other.
- **Trigger fires while a previous run is in flight for the same freed slot** -- Not possible in practice: a given Booking can only be cancelled once (its Contention resolution is reject-with-refresh, first committed transition wins), so exactly one freed-slot signal is ever produced per cancelled booking.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | A client cancellation commit fires this automation |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | A Pro single-cancellation commit fires this automation |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each cancelled booking in a Pro bulk cancellation fires this automation |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Supplies the matching test and claim-window computation this automation applies |
| FEAT-20.SPEC-008 (Waitlist Opening Notification) | Triggers (outbound) | Fires once per matched, Notified entry |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The status change (Requested -> Notified) is what that screen's next load reflects |
| FEAT-03 (Real-Time Slot Availability Engine) | References (outbound) | Supplies the freed slot's live truth and governs when it joins general availability |

## Analytics and Success Signals

- **waitlist_notified** (service ID, matched_entry_count) -- N/A -- no success-metrics.md metric directly measures waitlist matching; retained per the Brief's Non-Functional Notes naming waitlist_notified as one of the four Stage 2 signals this feature's transitions must emit.
- **waitlist_match_none_found** (service ID) -- N/A -- no Stage 2 metric measures unmatched openings; retained for operational visibility into how often a freed slot has no waiting client.
- **waitlist_notification_handoff_failed** (service ID) -- N/A -- no Stage 2 metric measures this failure path; retained so a silent notification gap is observable rather than invisible, consistent with the product's correctness-first bar (ASMP-21).

## Acceptance Criteria

**FEAT-20.SPEC-005-AC-01:** Given a client cancellation commits (FEAT-10.SPEC-004) for a service and date matching Riley's Requested entry, when this automation runs, then her entry transitions to Notified with claim_deadline set to platform parameter: `waitlist-claim-window-minutes` from that moment, and FEAT-20.SPEC-008 is triggered.

**FEAT-20.SPEC-005-AC-02:** Given a Pro single cancellation commits (FEAT-30.SPEC-007) for a freed slot with no matching Requested entries, when this automation runs, then no entry is transitioned and the slot proceeds to general availability.

**FEAT-20.SPEC-005-AC-03:** Given a Pro bulk cancellation (FEAT-30.SPEC-008) frees three bookings' slots, when this automation processes them, then each freed slot's matching runs independently with its own set of Notified entries.

**FEAT-20.SPEC-005-AC-04:** Given three Requested entries match the same freed slot, when this automation runs, then all three are transitioned to Notified simultaneously and each triggers its own FEAT-20.SPEC-008 notification.

**FEAT-20.SPEC-005-AC-05:** Given a booking's vacated original time results from a reschedule rather than a cancellation, then this automation is never triggered for that vacated time.

**FEAT-20.SPEC-005-AC-06:** Given this automation transitions an entry to Notified, when the notification hand-off to FEAT-20.SPEC-008 fails, then the entry still shows as Notified with a countdown on FEAT-20.SPEC-002, even though no notification was delivered.

**FEAT-20.SPEC-005-AC-07:** Given the matching step itself fails to run for a freed slot, when the failure occurs, then no entry is transitioned and the slot proceeds to general availability exactly as if zero entries had matched.

**FEAT-20.SPEC-005-AC-08:** Given a client's range-joined entry could match two different freed slots that arrive at the same time, when the first freed-slot signal is processed, then the entry transitions to Notified for that slot and becomes ineligible to be matched again until it resolves.

**FEAT-20.SPEC-005-AC-09:** Given two different bookings for two different services are cancelled at effectively the same time, when this automation processes both, then each freed-slot signal's matching runs independently and neither is delayed by the other.

**FEAT-20.SPEC-005-AC-10:** Given a Booking can only be cancelled once due to its own reject-with-refresh contention resolution, then this automation is never triggered twice for the same cancelled booking.

**FEAT-20.SPEC-005-AC-11:** Given zero entries match a freed slot, when this automation completes, then the slot appears on FEAT-05's general public list without any priority window elapsing.

**FEAT-20.SPEC-005-AC-12:** Given one or more entries are matched and notified, when the claim window is in effect, then the freed slot does not appear on FEAT-05's general public list until the window lapses, per FEAT-20.SPEC-004.

**FEAT-20.SPEC-005-AC-13:** Given a matched entry's claim_deadline is computed, then it is set exactly once, at the moment of transition to Notified, and this automation never recomputes it afterward.

**FEAT-20.SPEC-005-AC-14:** Given this automation reports completion back to the triggering cancellation commit, then that commit (FEAT-10.SPEC-004 or FEAT-30.SPEC-007/SPEC-008) proceeds and completes without waiting on this automation's own outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

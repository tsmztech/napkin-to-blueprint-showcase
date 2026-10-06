---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-002
spec_name: Activity Event Recording
spec_slug: activity-event-recording
parent_feature: FEAT-16
parent_feature_name: Booking & Payment Activity Record
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 23
---

# Automation Spec: Activity Event Recording

## Overview

**Name:** Activity Event Recording
**ID:** FEAT-16.SPEC-002
**Type:** Automation
**Purpose:** Writes one immutable, append-only Activity Event for every qualifying action across Booking, Deposit Payment, Messaging, Cancellation/Reschedule, No-Show, Client Deletion, Payout, Pro-initiated cancel/reschedule, and Support View, so a complete timeline exists for FEAT-16.SPEC-001 to render.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record

## Scope and Non-Goals

**In Scope:**
- Writing a new Activity Event for every qualifying trigger listed in the Trigger Definition below, including the support-view event handed off by FEAT-19.SPEC-002 (written against the Pro Account, actor "a support view")
- Converting this booking's (or Pro Account's) existing Activity Events to de-identified form when a client deletion is processed, per XBR-19
- Guaranteeing that every write is append-only and immutable at the moment of creation (the entry is never revisited by this automation once written)

**Non-Goals:**
- Rendering the timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline); this automation only produces the data that screen displays
- Writing the card-issuer dispute event -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration), which writes through this same append-only mechanism but is triggered by an external event this spec does not itself receive
- Editing or correcting a previously written entry, by any role or process, ever -- excluded per feature-overview.md's Validation & Limits ("append-only and immutable once written") and enforced by FEAT-16.SPEC-005; a mistaken entry is never overwritten, only ever superseded by a later, independent entry describing what actually happened next
- Assembling the support-view event's own content (reason/ticket reference, which timeline was viewed) -- owned by FEAT-19.SPEC-002 (Support View Logging), which hands the assembled event to this automation; this spec only writes it

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking created | FEAT-05.SPEC-005 (Booking Confirmation) | A new Booking reaches a confirmed state after deposit payment | Booking reference, service, appointment time, client reference, source (client link / Pro booked-in / recurring occurrence) |
| Policy shown and acknowledged | FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | The client acknowledges the cancellation policy during booking | Booking reference, Cancellation Policy version, plain-language wording shown, acknowledgment timestamp |
| Deposit attempted / succeeded / failed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Every deposit attempt reaches an outcome | Booking reference, Deposit Transaction reference, outcome (attempted / succeeded / failed), amount, timestamp |
| Message sent, retried, failed, or falls back to email | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-08.SPEC-009 (Automated Booking Messaging specs) | Every message delivery attempt or outcome, including a failed-then-fallback sequence per XBR-17 | Booking reference (or Pro Account reference for Pro notifications), message type, channel, delivery_status, timestamp |
| Client commits a cancel/reschedule | FEAT-10.SPEC-004 (Booking Update Commit) | The client's cancel or reschedule action commits | Booking reference, action (cancelled / rescheduled), new time (if rescheduled), deposit outcome triggered, timestamp |
| No-show marked | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | The Pro marks a booking as a no-show | Booking reference, Deposit Transaction outcome (forfeited), timestamp |
| No-show mark undone | FEAT-11.SPEC-003 (No-Show Mark Undo) | The Pro undoes a no-show mark within the grace window | Booking reference, reversed Deposit Transaction outcome, timestamp |
| Client deletion processed | FEAT-13.SPEC-004 (Client Deletion Execution) | A client deletion request completes | Client reference, list of affected Bookings, de-identification instruction |
| Payout account status changes | FEAT-28.SPEC-003 (Payout Account Status Processing) | The Pro's Payout Account status changes | Pro Account reference, new status, timestamp |
| Pro commits a single cancel/reschedule | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | The Pro's single cancel or reschedule action commits | Booking reference, action, new time (if rescheduled), deposit outcome, timestamp |
| Pro commits a bulk cancel/reschedule | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | The Pro's bulk cancel or reschedule action commits | List of affected Booking references, action, deposit outcome per booking, timestamp |
| Support view logged (session opened, or disputed booking timeline opened within a session) | FEAT-19.SPEC-002 (Support View Logging) | Support opens a help-request-gated session or opens a disputed booking's timeline within it | Pro Account reference, event_type (support_view_opened / support_view_booking_timeline), reason/ticket reference, Booking reference (timeline view only), timestamp |
| Card-issuer dispute recorded (external-event trigger) | FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | FEAT-16.SPEC-003 receives an inbound card-issuer dispute notice | Booking reference, Deposit Transaction reference, dispute timestamp |

## Processing Logic

1. Receive the triggering event's data from the source spec, identified by its trigger type from the table above.
2. Determine the correct event_type for the entry (e.g., "created," "policy_acknowledged," "deposit_attempted" / "deposit_succeeded" / "deposit_failed," "message_sent" / "message_failed" / "message_fallback," "cancelled" / "rescheduled," "no_show_marked" / "no_show_undone," "payout_status_changed," "support_view_opened" / "support_view_booking_timeline," "disputed").
3. Determine the actor for the entry: Client, Pro, "the product automatically," or a support view (when the trigger is FEAT-19.SPEC-002).
4. Assemble the details field from the trigger's available data (e.g., policy version and wording shown, message channel and outcome, deposit outcome and amount, new appointment time).
5. Write one new Activity Event, associated with the Booking (or, for a Pro-Account-level event such as a payout status change or a support view, the Pro Account -- a support-view entry never belongs to the Booking, even when the trigger was a timeline view) referenced by the trigger. The entry's time, actor, event_type, and details are fixed at the moment of this write and never revisited.
6. If the trigger is a bulk action affecting multiple bookings (FEAT-30.SPEC-008), repeat steps 2--5 once per affected booking so each booking's timeline carries its own entry.
7. If the trigger is a client deletion (FEAT-13.SPEC-004), do not write a new event for the deletion itself as a fresh fact on the booking; instead, convert every existing Activity Event belonging to the affected client's bookings to de-identified form: strip contact and note content from each entry's details field while retaining the financial and timeline facts (amounts, outcomes, timestamps, event types) unchanged, per XBR-19.
8. Confirm the write (or, for client deletion, the conversion) completed before returning control to the triggering spec; no triggering spec's own success path depends on waiting for this automation, since Activity Event recording is a side effect, not a precondition of any other spec's completion.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Entry written | The trigger's data is complete and valid | One new, immutable Activity Event created for the referenced Booking or Pro Account | None directly -- the entry becomes visible the next time FEAT-16.SPEC-001 is opened | FEAT-16.SPEC-001 (Booking Activity Timeline) |
| Bulk entries written | A bulk trigger (FEAT-30.SPEC-008) affects multiple bookings | One new Activity Event per affected booking | None directly | FEAT-16.SPEC-001 |
| Entries de-identified | A client deletion completes (FEAT-13.SPEC-004) | All existing Activity Events for the client's bookings have contact and note content stripped from their details field; financial and timeline facts retained | None directly -- Talia sees the de-identified entries the next time she opens an affected timeline | FEAT-16.SPEC-001 |
| No-op (nothing to record) | A trigger fires but its underlying action produced no state genuinely worth recording (this does not occur for any trigger in the table above -- every listed trigger corresponds to a qualifying, recordable action) | None | None | -- |
| Write failure | The automation cannot complete the write against a Booking or Pro Account reference (e.g., the referenced record cannot be found) | No entry is created | Non-blocking to the triggering spec -- the triggering action (e.g., the deposit capture, the message send) completes on its own terms regardless of whether its activity entry succeeded; the gap is retried automatically | The triggering spec proceeds unaffected; the timeline (FEAT-16.SPEC-001) shows a gap until the retry succeeds |

## Data Model

**Reads:** Booking (reference, service, start_time, client reference, state), Deposit Transaction (reference, status, amount, outcome_reason), Message (type, channel, delivery_status), Cancellation Policy (version, plain_language_wording) -- read only to assemble each entry's details field from the triggering spec's own available data, never independently re-queried beyond what the trigger provides.
**Creates:** Activity Event -- event_type, time, actor, details, associated to one Booking or the Pro Account.
**Updates:** Activity Event -- the only update path this spec has is the client-deletion de-identification conversion (details field content stripped of contact/note content); no other field of any entry is ever changed after creation.
**Deletes:** None -- hard deletion never occurs, per XBR-19 and the Entity-Lifecycle Coverage Matrix.

## Business Rules

- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit.
- Every entry is written once, at the moment its qualifying action occurs; no batching or delayed writing that could reorder entries relative to when their actions actually happened.
- A message-delivery gap (failed text, then email fallback) is recorded as its own visible fact, never merged into or replaced by the eventual fallback's success entry, per XBR-17.
- The Disputed overlay's own event (written by FEAT-16.SPEC-003) uses this same append-only mechanism and the same immutability guarantee; this spec's Processing Logic (steps 2--5) applies identically to that trigger.
- Client deletion never removes financial or timeline facts -- only contact and note content is stripped, per XBR-19 and SC-22.
- This automation never blocks or delays the triggering spec's own completion; recording is a side effect that runs alongside, not a gate the triggering action must pass through.

## Edge Cases

- **The triggering spec's own action later needs correction (e.g., a mis-marked no-show is undone)** -- The undo (FEAT-11.SPEC-003) is its own distinct trigger producing its own new entry; the original no-show-marked entry is never edited or removed, so the timeline shows both facts in order.
- **Two triggers fire for the same booking at effectively the same time (e.g., a client reschedule commits at the same moment a reminder message is sent)** -- Each trigger writes its own independent entry; entries are ordered by their own recorded time, and no coordination between the two writes is needed since neither reads or depends on the other's outcome.
- **A trigger fires while a previous recording run for the same booking is still in flight** -- Each write is independent and additive (append-only), so a second write for the same booking never needs to wait for, merge with, or overwrite the first; both entries land in the timeline in their own time order.
- **A client deletion is processed for a client with an in-flight, not-yet-recorded event (e.g., a message send that has not yet reported delivery status)** -- The de-identification conversion applies to entries that already exist at the time of deletion; a still-in-flight event that lands afterward is written and immediately carries no contact or note content for that now-deleted client's record, consistent with the fields already stripped elsewhere.
- **A trigger references a Booking or Pro Account that cannot be found (e.g., a data inconsistency upstream)** -- The write fails per the Write failure outcome above; the triggering spec's own action is unaffected, and the gap is retried automatically without blocking any user-facing flow.
- **A bulk cancel/reschedule (FEAT-30.SPEC-008) affects zero bookings (e.g., the Pro's selection ends up empty)** -- No entries are written; this is not a failure, simply nothing to record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Triggered by (inbound) | Booking creation writes the "created" entry |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | Triggered by (inbound) | Policy acknowledgment writes its own entry |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Deposit outcome writes its own entry |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-08.SPEC-009 | Triggered by (inbound) | Every message delivery attempt or outcome writes an entry |
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | Client cancel/reschedule writes an entry |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) | No-show mark writes an entry |
| FEAT-11.SPEC-003 (No-Show Mark Undo) | Triggered by (inbound) | No-show undo writes an entry |
| FEAT-13.SPEC-004 (Client Deletion Execution) | Triggered by (inbound) | Client deletion triggers de-identification conversion |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Payout status change writes an entry |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | Pro single cancel/reschedule writes an entry |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Pro bulk cancel/reschedule writes one entry per affected booking |
| FEAT-19.SPEC-002 (Support View Logging) | Triggered by (inbound) | Support session open and disputed-timeline view hand off an assembled support-view event; this automation is its sole writer |
| FEAT-16.SPEC-003 (Card-Issuer Dispute Integration) | Triggered by (inbound, external-event) | The dispute event is written through this same mechanism |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Affects (outbound) | Every entry this automation writes is what that screen renders |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only, immutable guarantee this automation upholds |

## Analytics and Success Signals

- **activity_event_recorded** (event_type, actor) -- N/A -- no success-metrics.md metric is connected to FEAT-16; this signal is retained for operational observability of recording volume and coverage across the inbound writer specs.
- **activity_event_recording_failed** (trigger source spec, reason) -- N/A -- no connected success-metrics.md metric; retained to observe how often a gap occurs before automatic retry closes it, since a silent gap would undermine the record's trustworthiness as dispute evidence.
- **activity_events_deidentified** (count of entries converted) -- N/A -- no connected success-metrics.md metric; retained to observe that XBR-19's de-identification obligation is actually being fulfilled on client deletion.

## Acceptance Criteria

**FEAT-16.SPEC-002-AC-01:** Given Riley completes a booking and her deposit is captured, when FEAT-05.SPEC-005 confirms the booking, then a "created" Activity Event is written for that booking with the appointment time, service, and client reference.

**FEAT-16.SPEC-002-AC-02:** Given Riley acknowledges the cancellation policy during booking, when FEAT-09.SPEC-002 records the acknowledgment, then a distinct "policy_acknowledged" Activity Event is written carrying the exact policy version and wording shown and the acknowledgment timestamp.

**FEAT-16.SPEC-002-AC-03:** Given a deposit attempt fails and is then retried and succeeds, when each outcome is determined by FEAT-07.SPEC-002, then two separate Activity Events are written -- one for the failed attempt and one for the succeeded attempt -- neither overwriting the other.

**FEAT-16.SPEC-002-AC-04:** Given a reminder text fails and falls back to email per XBR-17, when FEAT-08.SPEC-009 reports the fallback, then an Activity Event is written showing the failure and a second showing the successful email fallback, both visible on the timeline.

**FEAT-16.SPEC-002-AC-05:** Given Riley commits a reschedule through FEAT-10.SPEC-004, when the commit succeeds, then an Activity Event is written recording the reschedule, the new appointment time, and any resulting deposit outcome.

**FEAT-16.SPEC-002-AC-06:** Given Talia marks a booking as a no-show, when FEAT-11.SPEC-002 forfeits the deposit, then an Activity Event is written recording the no-show mark and the forfeiture outcome.

**FEAT-16.SPEC-002-AC-07:** Given Talia undoes a no-show mark within the grace window, when FEAT-11.SPEC-003 reverses the forfeiture, then a new Activity Event is written recording the undo; the original no-show-marked entry remains unchanged and visible.

**FEAT-16.SPEC-002-AC-08:** Given a client deletion completes via FEAT-13.SPEC-004, when the deletion is processed, then every existing Activity Event for that client's bookings has its contact and note content stripped from the details field while amounts, outcomes, and timestamps remain intact.

**FEAT-16.SPEC-002-AC-09:** Given Talia's Payout Account status changes via FEAT-28.SPEC-003, when the change is processed, then an Activity Event is written against her Pro Account recording the new status.

**FEAT-16.SPEC-002-AC-10:** Given Talia commits a single cancel through FEAT-30.SPEC-007, when the commit succeeds, then an Activity Event is written for that booking recording the cancellation and its deposit outcome.

**FEAT-16.SPEC-002-AC-11:** Given Talia commits a bulk cancellation affecting 5 bookings through FEAT-30.SPEC-008, when the commit succeeds, then 5 separate Activity Events are written, one per affected booking.

**FEAT-16.SPEC-002-AC-12:** Given FEAT-16.SPEC-003 receives an inbound card-issuer dispute notice, when it hands off the dispute event, then this automation writes a "disputed" Activity Event for the affected booking through the same append-only mechanism.

**FEAT-16.SPEC-002-AC-13:** Given any Activity Event has already been written, when any role, including Talia, attempts to change it, then no path exists anywhere in the product to do so -- the write in Processing Logic step 5 is the entry's only ever write.

**FEAT-16.SPEC-002-AC-14:** Given a trigger references a Booking that cannot be found, when the write is attempted, then the write fails without affecting the triggering spec's own success path, and the gap is retried automatically.

**FEAT-16.SPEC-002-AC-15:** Given a client cancel/reschedule commit and a message-delivery event both fire for the same booking at effectively the same time, when both automations run, then each writes its own independent entry and both appear correctly ordered by their own recorded time.

**FEAT-16.SPEC-002-AC-16:** Given a recording run for one booking is still in flight, when a second, unrelated trigger fires for the same booking, then the second write proceeds independently and does not wait for or merge with the first.

**FEAT-16.SPEC-002-AC-17:** Given Talia's bulk cancellation selection ends up affecting zero bookings, when FEAT-30.SPEC-008 completes with no bookings changed, then no Activity Event is written and this is not treated as a failure.

**FEAT-16.SPEC-002-AC-18:** Given a still-in-flight message send for a client completes delivery reporting after that client's deletion has already been processed, when the delivery event is recorded, then the new entry carries no contact or note content for that client, consistent with the client's other de-identified entries.

**FEAT-16.SPEC-002-AC-19:** Given a reschedule and an automatic policy-acknowledgment write occur for two different bookings at the same moment, when both automations run, then neither booking's timeline is affected by the other's write.

**FEAT-16.SPEC-002-AC-20:** Given Talia views a booking's timeline after a payout status change was recorded against her Pro Account, when she looks at her account-level activity (surfaced via the Pro Account's own activity context), then the payout-status entry appears alongside her booking-level entries' shared account context.

**FEAT-16.SPEC-002-AC-21:** Given a deposit outcome and its corresponding Activity Event write are both in progress, when the deposit outcome itself completes, then the deposit's own success or failure path is never blocked or delayed by whether this automation's write has finished.

**FEAT-16.SPEC-002-AC-22:** Given a client is deleted and later books again with the same Pro, when the new booking is created, then a fresh Client record and a fresh set of Activity Events begin -- the de-identified historical entries from the earlier relationship are never resurrected or merged into the new record's timeline.

**FEAT-16.SPEC-002-AC-23:** Given Support opens a session or a disputed booking's timeline and FEAT-19.SPEC-002 hands off the assembled support-view event, when this automation processes it, then one immutable Activity Event with actor "a support view" is written against Talia's Pro Account (not the Booking), and Support's view is not blocked by the write.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 13 | 13 |
| Outcome Paths | 5 | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |

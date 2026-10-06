---
document_type: spec
spec_type: automation
spec_id: FEAT-19.SPEC-002
spec_name: Support View Logging
spec_slug: support-view-logging
parent_feature: FEAT-19
parent_feature_name: Platform Support Read-Only Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Automation Spec: Support View Logging

## Overview

**Name:** Support View Logging
**ID:** FEAT-19.SPEC-002
**Type:** Automation
**Purpose:** On session open, and on each disputed-booking timeline opened within it, assembles the support-view event (actor, time, reason/ticket reference) and hands it to FEAT-16.SPEC-002 to write as the Pro Account's Activity Event -- this feature never writes a second event store.
**Parent Feature:** FEAT-19 -- Platform Support Read-Only Access

## Scope and Non-Goals

**In Scope:**
- Assembling the support-view event's data (actor, time, reason/ticket reference, and, when applicable, which booking's timeline was viewed) from each of its two triggers
- Handing the assembled event off to FEAT-16.SPEC-002, the sole Activity Event writer, for every trigger
- Firing once per qualifying trigger, with no deduplication across repeated views within the same session

**Non-Goals:**
- Writing the Activity Event record itself -- owned by FEAT-16.SPEC-002 (Activity Event Recording), the single append-only writer this feature hands off to and never duplicates, per feature-overview.md's Entity-Lifecycle Coverage Matrix.
- Rendering the resulting log -- owned by FEAT-19.SPEC-003 (Support Access Log), which reads what this automation's hand-offs eventually produce.
- Determining whether a session was opened for a legitimate reason -- excluded per scope-boundaries.md SC-05 and the Requirements Architect's own note that no Help Request entity exists in the Domain Entity Inventory for this automation to check against; this spec logs every session and timeline view that occurs, regardless of the reason's substance.
- Deciding when a session may open at all (the one-account-at-a-time and help-request-precondition rules) -- owned by FEAT-19.SPEC-004 (Support Session Scope & Access Rules); this automation assumes a session has already legitimately opened by the time it fires.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Support session opens | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | A lookup resolves to exactly one Pro Account and the session hub renders | Pro Account reference, reason/ticket reference, timestamp |
| Support opens a disputed booking's timeline within an open session | FEAT-16.SPEC-001 (Booking Activity Timeline), reached via FEAT-19.SPEC-001's hand-off | The timeline view opens while a support session is Active for that Pro Account | Pro Account reference, Booking reference, reason/ticket reference (carried from the session context), timestamp |

## Processing Logic

1. Receive the triggering event's data: the actor (a support view), a Pro Account reference, the reason/ticket reference carried by the current session, a timestamp, and (for the timeline trigger only) a Booking reference.
2. Determine the event_type for the entry: "support_view_opened" for the session-open trigger, or "support_view_booking_timeline" for the timeline-view trigger.
3. Assemble the details field: the reason/ticket reference always, and the Booking reference when the trigger is a timeline view.
4. Set the entry's actor to "a support view" (the fourth actor value in FEAT-16.SPEC-002's Processing Logic vocabulary, alongside Client, Pro, and the product automatically).
5. Hand off the assembled event -- event_type, time, actor, details, and the Pro Account it belongs to -- to FEAT-16.SPEC-002 for writing. This automation performs no write of its own; FEAT-16.SPEC-002 (the recording automation for support-view logging, which accepts this hand-off as an inbound trigger) is the entry's only writer.
6. Confirm the hand-off completed before returning control to the triggering spec; this automation's own completion never blocks or delays Support's view (consistent with FEAT-16.SPEC-002's non-blocking recording design).
7. If the disputed timeline is opened more than once within the same session, repeat steps 1--6 once per view -- no view is deduplicated or merged with a prior one, per feature-overview.md's Key Capability ("every support view is recorded").

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session-open event handed off | A support session opens successfully | One Activity Event is written by FEAT-16.SPEC-002, belonging to the Pro Account | None directly -- the entry becomes visible the next time FEAT-19.SPEC-003 is opened, by the Pro or by Support | FEAT-19.SPEC-003, FEAT-16.SPEC-002 |
| Timeline-view event handed off | A disputed booking's timeline is opened within an open session | One Activity Event is written by FEAT-16.SPEC-002, belonging to the Pro Account (not the Booking, per feature-overview.md's Shared Entities note) | None directly | FEAT-19.SPEC-003, FEAT-16.SPEC-002 |
| Hand-off failure | FEAT-16.SPEC-002's own write cannot complete (e.g., the referenced Pro Account cannot be found) | No entry is created for this occurrence | Non-blocking -- Support's session or timeline view proceeds unaffected; the gap is retried automatically per FEAT-16.SPEC-002's own Write failure outcome | FEAT-19.SPEC-003 (shows a gap until the retry succeeds) |
| No-op (nothing to record) | N/A -- both triggers in the table above always correspond to a qualifying, recordable action; there is no trigger path in this automation that produces nothing worth logging | -- | -- | -- |

## Data Model

**Reads:** Pro Account (reference), Booking (reference, when the trigger is a timeline view) -- read only to identify what the assembled event belongs to and, where applicable, which timeline was viewed; never independently re-queried beyond what the trigger provides.
**Creates:** None directly -- this automation assembles and hands off event data; FEAT-16.SPEC-002 is the sole creator of the resulting Activity Event, per the Entity-Lifecycle Coverage Matrix.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-24: every view is logged in the Pro's visible account activity -- this automation is the mechanism that fulfills that obligation for FEAT-19.
- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit -- this automation hands off into that same mechanism rather than defining a second one.
- Every qualifying trigger produces exactly one hand-off; no batching, delay, or deduplication across repeated views in the same session (a booking's timeline opened three times in one session produces three separate entries).
- This automation never blocks or delays the triggering screen's own completion -- logging is a side effect that runs alongside, not a gate Support's view must pass through, consistent with FEAT-16.SPEC-002's own non-blocking rule.
- This automation assumes the session it is logging has already satisfied FEAT-19.SPEC-004's opening rules; it logs every session that reaches it, without re-validating those rules itself.

## Edge Cases

- **Support opens a session and immediately opens a disputed booking's timeline within it** -- Each trigger produces its own independent hand-off; the second write proceeds without waiting for or merging with the first, per FEAT-16.SPEC-002's own append-only concurrency handling.
- **Concurrent trigger firing (two views logged at effectively the same moment, e.g., session open and an immediate timeline view)** -- Each writes its own independent entry, ordered by its own recorded time; neither write depends on or is blocked by the other.
- **Trigger fires while a previous hand-off for the same session is still in flight** -- The second hand-off proceeds independently and additively; append-only writes never need to wait for, merge with, or overwrite an in-flight one.
- **Support opens the same disputed booking's timeline twice within one session** -- Two separate Activity Events are logged, with no deduplication; the Support Access Log (FEAT-19.SPEC-003) later shows both as distinct rows.
- **The hand-off to FEAT-16.SPEC-002 fails (e.g., the Pro Account reference cannot be resolved at that moment)** -- Per the Hand-off failure outcome, Support's own session or timeline view is unaffected, and the gap is retried automatically without blocking any part of the read-only experience.
- **A session opens and ends again within moments (Support immediately realizes the wrong Pro was looked up)** -- The support_view_opened event handed off at session open was already recorded and is never retracted, consistent with XBR-21's immutability guarantee, even though the session itself was very short.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Triggered by (inbound) | Session open fires this automation's first trigger |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | Triggered by (inbound) | A disputed-timeline view reached from the open session fires this automation's second trigger |
| FEAT-16.SPEC-002 (Activity Event Recording) | Affects (outbound) | The automation that records support-view logging: FEAT-16.SPEC-002 lists FEAT-19.SPEC-002 as a writer source (actor "a support view") and receives every assembled event for the actual write; this automation is the source of that trigger |
| FEAT-19.SPEC-003 (Support Access Log) | Affects (outbound) | The entries this automation produces (via FEAT-16.SPEC-002) are what that screen renders |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | References (inbound) | Governs the append-only immutability of the entries this automation's hand-offs produce |
| FEAT-19.SPEC-004 (Support Session Scope & Access Rules) | References (inbound) | Governs the precondition for a session existing at all; this automation assumes that precondition was already satisfied |

## Analytics and Success Signals

- **support_view_logged** (event_type: support_view_opened / support_view_booking_timeline) -- N/A -- no success-metrics.md metric is connected to FEAT-19; retained to observe how reliably every support view is recorded, since XBR-24's trust guarantee to the Pro depends on this log's completeness.
- **support_view_log_handoff_failed** (trigger source, reason) -- N/A -- no connected success-metrics.md metric; retained to observe how often a hand-off gap occurs before FEAT-16.SPEC-002's automatic retry closes it, since an unrecorded support view would undermine the audit trail feature-overview.md's Key Capabilities promise to the Pro.

## Acceptance Criteria

**FEAT-19.SPEC-002-AC-01:** Given Support submits a valid lookup for Talia's account, when FEAT-19.SPEC-001 opens the session, then a "support_view_opened" event is assembled and handed off with the Pro Account reference, reason/ticket reference, and timestamp.

**FEAT-19.SPEC-002-AC-02:** Given Support's session for Talia's account is open, when Support opens a disputed booking's timeline, then a "support_view_booking_timeline" event is assembled and handed off with the Booking reference in addition to the session's reason/ticket reference.

**FEAT-19.SPEC-002-AC-03:** Given a support-view event is assembled, when it is handed off, then its actor field reads "a support view," never Client, Pro, or "the product automatically."

**FEAT-19.SPEC-002-AC-04:** Given a support-view event's hand-off completes, when FEAT-16.SPEC-002 writes it, then the resulting entry belongs to Talia's Pro Account, never to the specific Booking, even when the trigger was a timeline view of that booking.

**FEAT-19.SPEC-002-AC-05:** Given Support opens the same disputed booking's timeline twice within one session, when both views occur, then two separate Activity Events are logged, with neither view overwriting or merging into the other.

**FEAT-19.SPEC-002-AC-06:** Given Support opens a session and immediately opens a disputed timeline within it, when both triggers fire in close succession, then each produces its own independent hand-off, correctly ordered by its own recorded time.

**FEAT-19.SPEC-002-AC-07:** Given a hand-off to FEAT-16.SPEC-002 cannot complete because the Pro Account reference cannot be resolved, when the failure occurs, then Support's own session or timeline view proceeds unaffected, and the gap is retried automatically.

**FEAT-19.SPEC-002-AC-08:** Given a support session opens and is ended again within moments, when the session closes early, then the originally handed-off support_view_opened event is never retracted or removed.

**FEAT-19.SPEC-002-AC-09:** Given two support-view triggers fire for the same session at effectively the same time, when both automations run, then neither write waits for or is blocked by the other.

**FEAT-19.SPEC-002-AC-10:** Given a hand-off for one trigger is still in flight, when a second, unrelated trigger fires for the same session, then the second hand-off proceeds independently and does not queue behind the first.

**FEAT-19.SPEC-002-AC-11:** Given this automation completes its hand-off, when Talia later views her Support Access Log (FEAT-19.SPEC-003), then the entry appears with the actor, time, and reason/ticket reference this automation assembled.

**FEAT-19.SPEC-002-AC-12:** Given this automation's hand-off is in progress, when Support's own session-open or timeline-view action completes on the screen, then that screen's own completion is never blocked or delayed by whether this automation's hand-off has finished.

**FEAT-19.SPEC-002-AC-13:** Given a support session's reason/ticket reference is carried into a disputed-timeline-view trigger, when the second event is assembled, then it carries the same reason/ticket reference as the session's opening event.

**FEAT-19.SPEC-002-AC-14:** Given FEAT-19.SPEC-004's opening rules were satisfied before this automation's trigger fired, when this automation processes the trigger, then it logs the view without re-validating those opening rules itself.

**FEAT-19.SPEC-002-AC-15:** Given a hand-off failure occurs and is retried automatically, when the retry succeeds, then the resulting Activity Event carries the same event_type, actor, and details that were originally assembled, with only the completion timing delayed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

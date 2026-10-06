---
document_type: spec
spec_type: automation
spec_id: FEAT-22.SPEC-004
spec_name: Support Access Session Logging
spec_slug: support-access-session-logging
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Support Access Session Logging

## Overview

**Name:** Support Access Session Logging
**ID:** FEAT-22.SPEC-004
**Type:** Automation
**Purpose:** Records the start and end of every operator view of a household, and moves a Raised Support Request to Under review on its first view.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Creating an access-session entry (start timestamp) when Riley opens FEAT-22.SPEC-002 for a household
- Recording the session's end timestamp when Riley closes or navigates away from FEAT-22.SPEC-002
- Deferring to FEAT-22.SPEC-006 for whether an open is permitted at all
- Triggering the first-view status transition (FEAT-22.SPEC-008) and the organiser's in-app note (FEAT-22.SPEC-009)

**Non-Goals:**
- Deciding whether an open is permitted -- owned by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules); this automation only records sessions FEAT-22.SPEC-006 has already allowed
- Resolving the Support Request or closing a session as part of resolution -- owned by FEAT-22.SPEC-005 (Support Request Resolution), a distinct trigger source from the screen-close path this automation handles
- Displaying the logged sessions to the organiser -- owned by FEAT-22.SPEC-003 (Household Support Access Record), which reads what this automation writes
- Any household data beyond the session's own start/end timestamps and its Support Request reference -- this automation never reads or writes the household's setup, plan, or other domain data; FEAT-22.SPEC-002 owns that display

## Trigger Definition

| Trigger | Category | Source Spec | Conditions | Available Data |
|---------|----------|------------|------------|----------------|
| Riley opens the household view | User-initiated | FEAT-22.SPEC-002 (Support Read-Only Household View) | Fires only after FEAT-22.SPEC-006 has permitted the open | The household, the selected Support Request, current timestamp |
| Riley closes or navigates away from the household view | User-initiated | FEAT-22.SPEC-002 (Support Read-Only Household View) | Fires whenever the screen unmounts, including mid-load closes and session expiry | The open access-session entry (from the corresponding open trigger), current timestamp |

## Processing Logic

1. On open: receive the household, the selected Support Request, and the current timestamp from FEAT-22.SPEC-002.
2. Confirm with FEAT-22.SPEC-006 that this open is permitted (the request's status is Raised or Under review, and Riley has no other household open); this automation does not itself decide permission -- FEAT-22.SPEC-002 only calls this automation once FEAT-22.SPEC-006 has already allowed the open.
3. Append a new entry to the Support Request's access_record: start timestamp set to the current time, end timestamp unset.
4. Check whether this is the first access-session entry ever recorded for this Support Request.
5. If it is the first entry, trigger FEAT-22.SPEC-008 (Support Request Status Transition Rules) to move the request's status from Raised to Under review.
6. Trigger FEAT-22.SPEC-009 (Support View Recorded Notification) to tell the organiser a session opened, with the current entry's start time and the request's kind.
7. On close: receive the open access-session entry and the current timestamp from FEAT-22.SPEC-002.
8. Set that entry's end timestamp to the current time.
9. Trigger FEAT-22.SPEC-009 again to tell the organiser the session closed, with the entry's end time.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session opened, request advanced | Open permitted and this is the request's first session | New access_record entry with start timestamp; Support Request status Raised -> Under review | Riley sees FEAT-22.SPEC-002 populate; organiser receives the "support viewed" note | FEAT-22.SPEC-002, FEAT-22.SPEC-008, FEAT-22.SPEC-009 |
| Session opened, request already under review | Open permitted and a prior session already exists for this request | New access_record entry with start timestamp; status unchanged (already Under review) | Riley sees FEAT-22.SPEC-002 populate; organiser receives the "support viewed" note | FEAT-22.SPEC-002, FEAT-22.SPEC-009 |
| Session closed | Riley closes or navigates away from an open session | The open access_record entry's end timestamp is set | Organiser receives the "support access ended" note | FEAT-22.SPEC-003, FEAT-22.SPEC-009 |
| Open denied upstream | FEAT-22.SPEC-006 does not permit the open | No access_record entry created; no status change | Riley sees FEAT-22.SPEC-001's gating-denial message; this automation never runs | FEAT-22.SPEC-001 |
| Logging failure | The access_record write cannot be completed | No partial entry persists | Riley sees FEAT-22.SPEC-002's load-error state; the open is treated as not having occurred | FEAT-22.SPEC-002 |

## Data Model

**Reads:** Support Request -- status, access_record (to detect whether this is the first session).
**Creates:** Support Request.access_record entry -- start timestamp, on open.
**Updates:** Support Request.access_record entry -- end timestamp, on close; Support Request.status -- Raised -> Under review, via FEAT-22.SPEC-008, on first view only.
**Deletes:** None -- access_record entries are never removed, per SC-18's retention posture.

## Business Rules

- XBR-14: every operator visit is recorded where the organiser can see it -- this automation is the sole writer of that record.
- Every open that FEAT-22.SPEC-006 permits produces exactly one access_record entry; every close of an open session sets that same entry's end timestamp -- entries are never merged or split.
- The Raised -> Under review transition fires exactly once per Support Request, on its first-ever session, regardless of how many sessions follow.
- This automation runs synchronously with the triggering screen action: FEAT-22.SPEC-002 does not render the household's data until the open is logged, and does not complete its close until the end timestamp is recorded.

## Edge Cases

- **Riley's session expires while a household view is open (no explicit close)** -- The session end is still recorded, using the moment the session is confirmed expired as the end timestamp, since an abandoned session must not remain "in progress" indefinitely in the organiser's record.
- **Riley closes the view before household data has finished loading (FEAT-22.SPEC-002's Loading state)** -- The access-session entry, once created on open, still receives its end timestamp on close; a session that never showed data to Riley is still a session that occurred and is still logged.
- **Two access-session entries would ever be open for the same Support Request at once** -- Cannot occur: FEAT-22.SPEC-006 permits only one household open per operator, and Riley is the sole operator role in the product (Access Matrix), so no second open can be initiated while a session for the same request is unclosed.
- **Concurrent trigger firing (an open and a close arrive at effectively the same time for different Support Requests)** -- Each processes independently against its own Support Request's access_record; there is no shared state between the two runs.
- **Trigger fires while a previous run is in flight (a rapid close-then-reopen of the same household)** -- The close is fully recorded (end timestamp set) before the reopen's permission check runs, since FEAT-22.SPEC-006 evaluates Riley's current open-household state synchronously; a reopen cannot begin logging a new session until the prior one's close has completed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-002 (Support Read-Only Household View) | Triggered by (inbound) | Opening and closing the screen fires this automation |
| FEAT-22.SPEC-006 (Support Access Scope & Gating Rules) | References (inbound) | Determines whether an open is permitted before this automation logs it |
| FEAT-22.SPEC-008 (Support Request Status Transition Rules) | Triggers (outbound) | First-view transition from Raised to Under review |
| FEAT-22.SPEC-009 (Support View Recorded Notification) | Triggers (outbound) | Tells the organiser a session opened or closed |
| FEAT-22.SPEC-003 (Household Support Access Record) | Affects (outbound) | Displays every entry this automation writes |

## Analytics and Success Signals

- **support_access_session_opened** (request kind, first_view: yes/no) -- N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing, low-volume audit mechanism's actual use is observable
- **support_access_session_closed** (duration) -- N/A -- no success-metrics.md metric traces to this feature
- **support_access_first_view_transition** (request kind) -- N/A -- no success-metrics.md metric traces to this feature; retained so the Raised -> Under review handoff is observable rather than silent

## Acceptance Criteria

**FEAT-22.SPEC-004-AC-01:** Given FEAT-22.SPEC-006 permits Riley's open of a household with a Raised request that has never been viewed, when this automation runs, then a new access_record entry is created with a start timestamp and the request transitions to Under review.

**FEAT-22.SPEC-004-AC-02:** Given the request is already Under review from a prior session, when Riley opens the household again, then a new access_record entry is created but the status does not change.

**FEAT-22.SPEC-004-AC-03:** Given Riley has an open session, when Riley taps Close on FEAT-22.SPEC-002, then the open access_record entry's end timestamp is set to the current time.

**FEAT-22.SPEC-004-AC-04:** Given an access session opens, when this automation completes step 6, then the organiser receives FEAT-22.SPEC-009 with the session's start time and the request's kind.

**FEAT-22.SPEC-004-AC-05:** Given an access session closes, when this automation completes step 9, then the organiser receives FEAT-22.SPEC-009 with the session's end time.

**FEAT-22.SPEC-004-AC-06:** Given FEAT-22.SPEC-006 denies an open, then this automation never runs and no access_record entry is created.

**FEAT-22.SPEC-004-AC-07:** Given Riley's session expires while a household view is open, when the expiry is confirmed, then the open access_record entry's end timestamp is set at that moment.

**FEAT-22.SPEC-004-AC-08:** Given Riley closes the household view before its data finishes loading, when the close fires, then the access_record entry still receives an end timestamp.

**FEAT-22.SPEC-004-AC-09:** Given two different Support Requests' sessions open and close at effectively the same time, when both trigger this automation, then each processes independently against its own access_record.

**FEAT-22.SPEC-004-AC-10:** Given Riley closes a household view and immediately reopens the same household, when the reopen's permission check runs, then it evaluates only after the close's end timestamp has been recorded.

**FEAT-22.SPEC-004-AC-11:** Given the access_record write itself fails, when Riley attempts to open a household, then no partial entry is created and FEAT-22.SPEC-002 shows its load-error state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (open, close) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

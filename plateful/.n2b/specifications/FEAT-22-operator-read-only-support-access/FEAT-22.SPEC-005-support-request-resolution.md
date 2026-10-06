---
document_type: spec
spec_type: automation
spec_id: FEAT-22.SPEC-005
spec_name: Support Request Resolution
spec_slug: support-request-resolution
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Support Request Resolution

## Overview

**Name:** Support Request Resolution
**ID:** FEAT-22.SPEC-005
**Type:** Automation
**Purpose:** Applies Riley's "Mark Resolved" decision, closing any open access session and handing off the outcome to the household's own resolution notice.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Transitioning a Support Request from Under review (or Raised, if resolved before any view) to Resolved
- Capturing the safety/unsafe outcome for a safety-concern request and forwarding it to FEAT-02.SPEC-005
- Closing any still-open access session for the request
- Triggering FEAT-22.SPEC-010 for a general-support request's resolution

**Non-Goals:**
- Determining the household-facing safety outcome message, or the recipe's eligibility to return to the candidate pool -- owned by FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) and FEAT-02.SPEC-014 (Safety Concern Resolution Notice), which this automation hands the outcome to for safety-concern requests
- Notifying the organiser of a safety-concern resolution -- excluded per this feature's own design: FEAT-02.SPEC-014 already tells the household the safety outcome once this automation forwards it, and a second, separate FEAT-22 notice for the same event would duplicate and could contradict that message; FEAT-22.SPEC-010 fires only for general-support resolutions
- Removing or restoring any household data -- this automation writes only Support Request status and access-record fields, per the dependency map's "Managed by" line for Support Request ("status and access record only")
- Deciding whether the resolution attempt is permitted -- owned by FEAT-22.SPEC-008 (Support Request Status Transition Rules), which this automation defers to before applying the transition

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Riley confirms "Mark Resolved" | FEAT-22.SPEC-002 (Support Read-Only Household View) | Fires when Riley confirms the resolution dialog; for a safety-concern request, only after Riley selects an outcome | The Support Request (kind, status, planned_meal/recipe), and for safety concerns, the selected outcome ("Recipe is safe" / "Recipe is unsafe") |

## Processing Logic

1. Receive the Support Request and, for a safety-concern kind, the selected outcome, from FEAT-22.SPEC-002.
2. Confirm with FEAT-22.SPEC-008 (Support Request Status Transition Rules) that a transition to Resolved is valid from the request's current status (Raised or Under review).
3. Set the Support Request's status to Resolved via FEAT-22.SPEC-008.
4. Check whether the request has a still-open access session (an access_record entry with no end timestamp).
5. If an open session exists, set its end timestamp to the current time (the same mechanic FEAT-22.SPEC-004 uses on an ordinary close).
6. If the request's kind is safety concern, forward the request and the selected outcome to FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), which determines the recipe's eligibility and triggers the household's outcome notice (FEAT-02.SPEC-014).
7. If the request's kind is general support contact, trigger FEAT-22.SPEC-010 (Support Request Resolved Notification) to tell the organiser.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Safety concern resolved | Kind is safety concern and Riley selects an outcome | Status set to Resolved; open session (if any) closed; outcome forwarded to FEAT-02.SPEC-005 | Riley sees FEAT-22.SPEC-002's Resolved state; household receives FEAT-02.SPEC-014's outcome notice | FEAT-22.SPEC-002, FEAT-22.SPEC-008, FEAT-02.SPEC-005 |
| General support resolved | Kind is general support contact | Status set to Resolved; open session (if any) closed | Riley sees FEAT-22.SPEC-002's Resolved state; organiser receives FEAT-22.SPEC-010 | FEAT-22.SPEC-002, FEAT-22.SPEC-008, FEAT-22.SPEC-010 |
| Resolution rejected (invalid transition) | The request is already Resolved (e.g., resolved elsewhere in a race that FEAT-22.SPEC-006 otherwise prevents) | No status change | Riley sees FEAT-22.SPEC-002's dialog error: "This request has already been resolved." | FEAT-22.SPEC-002 |
| Resolution failure (processing error) | The status write or session close cannot be completed | No partial state -- either the full resolution succeeds or neither the status nor the session changes | Riley sees FEAT-22.SPEC-002's dialog error "Couldn't mark this request resolved. Check your connection and try again." with Retry | FEAT-22.SPEC-002 |

## Data Model

**Reads:** Support Request -- status, kind, planned_meal/recipe, access_record (to find any still-open session).
**Creates:** None.
**Updates:** Support Request -- status (-> Resolved) via FEAT-22.SPEC-008; access_record entry's end timestamp, if a session was still open.
**Deletes:** None.

## Business Rules

- XBR-14: operator support access closes when the request is resolved -- this automation is the mechanism that closes it.
- A safety-concern resolution always requires an outcome selection ("Recipe is safe" / "Recipe is unsafe"); a general-support resolution never requires one, since it has no recipe-eligibility consequence.
- This automation writes only Support Request status and access-record fields -- never household, plan, or any other domain data, per the dependency map's "Managed by" line for Support Request.
- Resolving a request with a still-open session always closes that session as part of the same action -- there is no path to a Resolved request with an "in progress" access_record entry.
- Only Riley (Operator, support) can trigger this automation, per FEAT-22.SPEC-008's Authorization Rules.

## Edge Cases

- **Riley marks a request Resolved while no session is currently open for it (e.g., the request was Raised and Riley resolves it from the queue's context without ever opening the household)** -- Step 4 finds no open session, and step 5 is skipped; resolution proceeds normally on status alone.
- **Riley selects "Recipe is unsafe" for a safety-concern request** -- FEAT-02.SPEC-005 receives that outcome and applies the permanent-exclusion path; this automation's own responsibility ends at forwarding the outcome, per its Non-Goals.
- **The request is already Resolved when Riley attempts to resolve it again** -- FEAT-22.SPEC-008 rejects the transition; Riley sees "This request has already been resolved." and no further processing occurs.
- **Network interruption after the status write but before the session-close write completes** -- The automation treats the status transition and session close as one atomic outcome for the user; if the session close cannot be confirmed, Riley sees the failure state and may retry, and the retry's status check (already Resolved) short-circuits step 3 while still completing step 5 for the still-open session.
- **Concurrent trigger firing (two different requests resolved at effectively the same time)** -- Each processes independently against its own Support Request; there is no shared state between the two runs.
- **Trigger fires while a previous resolution for the same request is still in flight** -- Cannot occur under normal use: FEAT-22.SPEC-002 disables "Mark Resolved" during the Resolving state, so a second resolution attempt for the same request cannot be submitted until the first completes or fails.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-002 (Support Read-Only Household View) | Triggered by (inbound) | "Mark Resolved" confirmation fires this automation |
| FEAT-22.SPEC-008 (Support Request Status Transition Rules) | References (outbound) | Validates and applies the Resolved transition |
| FEAT-22.SPEC-004 (Support Access Session Logging) | References (outbound) | Shares the session-close mechanic for a still-open session |
| FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Triggers (outbound, cross-feature) | Receives the safety outcome for safety-concern requests |
| FEAT-22.SPEC-010 (Support Request Resolved Notification) | Triggers (outbound) | Tells the organiser for general-support resolutions |

## Analytics and Success Signals

- **support_request_resolved** (kind, outcome: safe / unsafe / N/A for general support) -- supports success-metrics.md: "Zero Allergy Incidents" (for safety-concern outcomes only, since this event is the operator-side half of the same trust-critical resolution the metric tracks)
- **support_request_resolution_rejected** (reason: already_resolved) -- N/A -- no success-metrics.md metric traces to this feature; retained so an invalid-transition attempt is observable rather than silent

## Acceptance Criteria

**FEAT-22.SPEC-005-AC-01:** Given Riley confirms resolution of a safety-concern request with outcome "Recipe is safe", when this automation runs, then the request's status is set to Resolved and FEAT-02.SPEC-005 receives the "safe" outcome.

**FEAT-22.SPEC-005-AC-02:** Given Riley confirms resolution of a safety-concern request with outcome "Recipe is unsafe", when this automation runs, then the request's status is set to Resolved and FEAT-02.SPEC-005 receives the "unsafe" outcome.

**FEAT-22.SPEC-005-AC-03:** Given Riley confirms resolution of a general-support request, when this automation runs, then the request's status is set to Resolved and the organiser receives FEAT-22.SPEC-010.

**FEAT-22.SPEC-005-AC-04:** Given Riley resolves a request that currently has an open access session, when this automation runs, then the session's end timestamp is set as part of the same resolution.

**FEAT-22.SPEC-005-AC-05:** Given Riley resolves a Raised request that was never viewed (no open session exists), when this automation runs, then the status transition completes with no session-close step performed.

**FEAT-22.SPEC-005-AC-06:** Given a request is already Resolved, when Riley attempts to resolve it again, then FEAT-22.SPEC-008 rejects the transition and Riley sees "This request has already been resolved."

**FEAT-22.SPEC-005-AC-07:** Given the resolution write fails due to a network error, when Riley views FEAT-22.SPEC-002's dialog, then the failure message with Retry appears and the request's status is unchanged.

**FEAT-22.SPEC-005-AC-08:** Given a general-support resolution completes, then no FEAT-02.SPEC-005 forwarding occurs, since that path is reserved for safety-concern requests only.

**FEAT-22.SPEC-005-AC-09:** Given two unrelated requests are resolved at effectively the same time, when both trigger this automation, then each processes independently.

**FEAT-22.SPEC-005-AC-10:** Given a resolution is already in progress for a request, when FEAT-22.SPEC-002's dialog is in its Resolving state, then a second resolution attempt for the same request cannot be submitted until the first completes or fails.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-008
spec_name: Support Request Status Transition Rules
spec_slug: support-request-status-transition-rules
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Support Request Status Transition Rules

## Overview

**Name:** Support Request Status Transition Rules
**ID:** FEAT-22.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs the Raised -> Under review -> Resolved lifecycle of a Support Request, including who may cause each transition and what happens to an invalid attempt.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access
**Governed Entity:** Support Request

## Scope and Non-Goals

**In Scope:**
- The valid status values and transitions for a Support Request: Raised -> Under review -> Resolved
- Who may cause each transition, and under what condition
- What happens when a transition is attempted out of order or on an already-Resolved request
- Field coverage for every field on the Support Request entity, for completeness, even though most fields are outside this spec's own concern

**Non-Goals:**
- Creating a Support Request -- owned by FEAT-02 (safety concerns) and FEAT-18 (general support contact); this spec governs only the status field once a request exists
- Deciding whether an access session may open against a given status -- owned by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules), which reads this spec's status values as its precondition rather than duplicating the lifecycle
- The safety/unsafe outcome captured alongside a safety-concern resolution -- owned by FEAT-22.SPEC-005 (Support Request Resolution), which this spec's Resolved transition accepts without interpreting the outcome itself
- Deleting or archiving a Support Request -- excluded per the Entity-Lifecycle Coverage Matrix's Delete/Archive: N/A finding; no feature in the dependency map removes a Support Request, and this spec accordingly defines no such transition

## Governed Entity

**Entity:** Support Request
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| kind | enum (safety concern, general support) | Set at creation by FEAT-02 or FEAT-18; not modified by this spec |
| raised_by | reference (Member Profile) | Set at creation; not modified by this spec |
| planned_meal/recipe | reference | Set at creation for safety concerns; not modified by this spec |
| note | text | Set at creation; not modified by this spec |
| status | enum (Raised, Under review, Resolved) | The field this spec governs |
| access_record | list | Written by FEAT-22.SPEC-004 and FEAT-22.SPEC-005; not modified by this spec directly, though this spec's transitions are triggered alongside access_record writes |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-004 | Support Access Session Logging | Applies the Raised -> Under review transition on a request's first view |
| FEAT-22.SPEC-005 | Support Request Resolution | Applies the -> Resolved transition when Riley confirms resolution |
| FEAT-22.SPEC-001 | Support Request Queue | Reads status to decide which requests qualify as "open" for listing |
| FEAT-22.SPEC-006 | Support Access Scope & Gating Rules | Reads status as its open-precondition check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| kind | No validation beyond data type -- set at creation by FEAT-02 or FEAT-18, outside this spec's scope | Always | -- | -- | -- |
| raised_by | No validation beyond data type -- set at creation, outside this spec's scope | Always | -- | -- | -- |
| planned_meal/recipe | No validation beyond data type -- set at creation, outside this spec's scope | Always | -- | -- | -- |
| note | No validation beyond data type -- set at creation, outside this spec's scope | Always | -- | -- | -- |
| status | Must follow the sequence Raised -> Under review -> Resolved; no transition may skip backward, and no transition may be applied to a request already at Resolved | Always | On every transition attempt (FEAT-22.SPEC-004, FEAT-22.SPEC-005) | "This request has already been resolved and can no longer be opened." (attempted open on a Resolved request); "This request has already been resolved." (attempted second resolution) | Yes |
| access_record | No validation beyond data type -- owned and written by FEAT-22.SPEC-004 and FEAT-22.SPEC-005, outside this spec's own writes | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Resolution allowed from either open state | status | A transition to Resolved is valid from either Raised (a request resolved without ever being viewed) or Under review (the ordinary path, after at least one view); it is never valid from Resolved itself | "This request has already been resolved." |
| First-view transition is a system effect, not a user action | status, access_record | The Raised -> Under review transition fires automatically, as a direct consequence of FEAT-22.SPEC-004 logging the first access-session entry -- it is never a separate action Riley takes | N/A -- this is a system-triggered transition, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Transition Raised -> Under review | System, via FEAT-22.SPEC-004 | Automatically, on the request's first logged access session | -- |
| Transition Raised or Under review -> Resolved | Riley (Operator, support), via FEAT-22.SPEC-005 | Only when the current status is Raised or Under review | "This request has already been resolved." if attempted on a Resolved request |
| Transition Resolved -> any other status | No role, ever | Never -- Resolved is terminal | No control anywhere in the product offers to reopen a Resolved request |
| Maya, Sam, or either Jordan role transitioning status directly | No role, ever | Never | No screen available to these roles exposes a status control; their only visibility is FEAT-01.SPEC-010 and FEAT-22.SPEC-003's read-only display of the current status |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| status | Set to Raised by the creating feature (FEAT-02 or FEAT-18) | On Support Request creation | No -- every request begins Raised; this spec governs only transitions after creation |

## Business Rules

- XBR-14: operator support access closes when the request is resolved -- Resolved is this lifecycle's terminal state, and FEAT-22.SPEC-005 always closes any open session as part of reaching it.
- Under review exists specifically to signal that Riley has begun looking into the request; it carries no separate authorization consequence beyond distinguishing "seen" requests from "unseen" ones in FEAT-22.SPEC-001's queue ordering context.
- A request that is resolved directly from Raised (never viewed) is a legitimate path -- Riley is not required to open a household before resolving its request, though doing so without viewing is expected to be rare given the feature's purpose.
- This spec never writes access_record -- it only defines the status field's own transition rules, which FEAT-22.SPEC-004 and FEAT-22.SPEC-005 apply alongside their own access_record writes.

## Edge Cases

- **A request is resolved from Raised, having never been viewed** -- The Under review state is simply skipped; the transition Raised -> Resolved is valid per the Cross-Field Rules, and no access session ever existed to close.
- **Two attempts to transition the same request to Resolved arrive at effectively the same time** -- FEAT-22.SPEC-006 permits only one operator session at a time, and Riley is the sole operator role, so no two independent resolution attempts for the same request can originate; a rapid double-tap on "Mark Resolved" is separately debounced by FEAT-22.SPEC-002's own interaction handling.
- **An attempt to view a household whose request has just transitioned to Resolved (a race between the queue's stale data and a fresh status)** -- The transition rule re-validates at the moment of the attempt and denies it, per FEAT-22.SPEC-006's own re-check.
- **A request sits at Under review indefinitely, with no further sessions and no resolution** -- No automatic timeout or escalation exists in this spec; the request remains Under review until Riley resolves it, consistent with the feature's description as rare, on-demand use with no scheduled follow-up.

## Acceptance Criteria

**FEAT-22.SPEC-008-AC-01:** Given a newly created Support Request, when it is created by FEAT-02 or FEAT-18, then its status is Raised.

**FEAT-22.SPEC-008-AC-02:** Given a Raised request receives its first logged access session, when FEAT-22.SPEC-004 completes that step, then the status transitions to Under review.

**FEAT-22.SPEC-008-AC-03:** Given a request already at Under review receives a second access session, when FEAT-22.SPEC-004 completes that step, then the status remains Under review.

**FEAT-22.SPEC-008-AC-04:** Given a request at Raised, when Riley resolves it without ever opening its household, then the status transitions directly to Resolved.

**FEAT-22.SPEC-008-AC-05:** Given a request at Under review, when Riley resolves it, then the status transitions to Resolved.

**FEAT-22.SPEC-008-AC-06:** Given a request already at Resolved, when Riley attempts to resolve it again, then the attempt is denied with "This request has already been resolved."; and when Riley attempts to open its household, then the attempt is denied with "This request has already been resolved and can no longer be opened."

**FEAT-22.SPEC-008-AC-07:** Given a request at Resolved, when any role looks for a control to reopen it, then none exists anywhere in the product.

**FEAT-22.SPEC-008-AC-08:** Given Maya or Sam views their household's request status, then it is shown read-only with no control to change it, per FEAT-01.SPEC-010 or FEAT-22.SPEC-003.

**FEAT-22.SPEC-008-AC-09:** Given a request's status changes from Raised to Under review, when this transition occurs, then no user-facing message or confirmation is shown, since it is a system effect of viewing, not a user action.

**FEAT-22.SPEC-008-AC-10:** Given a request's status is Resolved while FEAT-22.SPEC-001's queue still shows it as open due to stale data, when Riley attempts to select it, then the screen-entry re-check denies the open.

**FEAT-22.SPEC-008-AC-11:** Given a request sits at Under review with no further action, when time passes with no resolution, then the status remains Under review indefinitely, with no automatic transition applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

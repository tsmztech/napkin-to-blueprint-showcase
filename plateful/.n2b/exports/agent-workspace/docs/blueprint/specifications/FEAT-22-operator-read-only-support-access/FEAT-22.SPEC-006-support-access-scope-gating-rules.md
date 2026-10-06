---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-006
spec_name: Support Access Scope & Gating Rules
spec_slug: support-access-scope-gating-rules
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Support Access Scope & Gating Rules

## Overview

**Name:** Support Access Scope & Gating Rules
**ID:** FEAT-22.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs when Riley's read-only access may open, that it is strictly read-only with no edit actions, and that only one household is open at a time.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access
**Governed Entity:** Support Request (its status field drives the gating window) and the operator's access-session state derived from it

## Scope and Non-Goals

**In Scope:**
- The precondition that must hold for a household to be opened: an open (Raised or Under review) Support Request against it
- The one-household-at-a-time limit on Riley's access
- The blanket no-edit-action rule for household data across every screen this feature reaches -- distinct from "Mark Resolved," the one permitted lifecycle action on the Support Request's own status
- Authorization for opening, viewing within, and closing an access session

**Non-Goals:**
- The status lifecycle itself (Raised -> Under review -> Resolved) -- owned by FEAT-22.SPEC-008 (Support Request Status Transition Rules), which this spec's preconditions reference by status value rather than redefining
- What data is suppressed within an open session (kid profile detail, billing detail) -- owned by FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule); this spec governs whether access opens at all, not what it shows once open
- Logging the session's start and end timestamps -- owned by FEAT-22.SPEC-004 (Support Access Session Logging), which defers to this spec's permission decision before writing anything
- Any household member's own access to their household's screens -- excluded per scope-boundaries.md SC-01: this spec governs only Riley's operator-scoped access, never a household member's ordinary product access, which is unaffected by these rules

## Governed Entity

**Entity:** Support Request (status field) and the operator access-session state it drives
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Raised, Under review, Resolved) | The field this spec's open-precondition rule reads; owned and transitioned by FEAT-22.SPEC-008 |
| access-session state (derived, not a stored field) | derived | Whether Riley currently has any household open, computed from unresolved access_record entries across all of Riley's Support Requests |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Support Request Queue | On every request-row selection, before navigation |
| FEAT-22.SPEC-002 | Support Read-Only Household View | On screen entry (whether the open is honored at all), and continuously by the absence of any edit control in its layout |
| FEAT-22.SPEC-004 | Support Access Session Logging | Before creating any access_record entry |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | Must be Raised or Under review for an open to be permitted | On every open attempt | On selection (FEAT-22.SPEC-001) and on screen entry (FEAT-22.SPEC-002) | "This request has already been resolved and can no longer be opened." | Yes |
| access-session state | No validation beyond data type -- it is a derived, system-computed value, never user-entered | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| One household at a time | status, access-session state | An open is permitted only when the target request's status qualifies (Raised or Under review) AND Riley currently has no other household's session open | "You already have {household name} open. Close it before opening another household." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Open a household's read-only view | Riley (Operator, support) | The target Support Request's status is Raised or Under review, AND Riley has no other household currently open | "This request has already been resolved and can no longer be opened." (status fails) or "You already have {household name} open. Close it before opening another household." (a different household is already open) |
| Open a household with no open Support Request against it | No role, ever | Never -- there is no screen or control that lets Riley select a household with no open request; FEAT-22.SPEC-001 lists only Raised/Under review requests | No such control exists anywhere in the product |
| View household data within an open session | Riley (Operator, support) | Session is open (per the rule above) | -- |
| Perform an edit action on any household data (household setup, member profiles, weekly plan, grocery list, ratings, pantry items, recipes, subscription tier) shown within an open session | No role, ever | Never -- not even Riley | No edit control for any household data exists anywhere in FEAT-22.SPEC-002's layout; there is no denied-attempt path to describe, since no control to attempt exists. (Distinct from "Mark Resolved," the one permitted lifecycle action, which changes only the Support Request's own status per FEAT-22.SPEC-008, never any household data.) |
| Close an open session | Riley (Operator, support) | Session is open | -- |
| Maya, Sam, or either Jordan role opening any FEAT-22 screen | No role, ever | Never | These screens are not part of any household member's product surface at all -- redirected away per each screen's own Access and Visibility table |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| access-session state (open / not open) | Derived as "open" when any of Riley's Support Requests has an access_record entry with a start timestamp and no end timestamp; "not open" otherwise | Evaluated on every open attempt | No -- fully system-derived from FEAT-22.SPEC-004's logged entries |

## Business Rules

- XBR-14: operator support access opens only for one household with an open Support Request, is strictly read-only, and closes when the request is resolved -- this spec is the authoritative source for the open/one-at-a-time/read-only half of that rule; the closes-on-resolution half is governed by FEAT-22.SPEC-005 and FEAT-22.SPEC-008.
- "Read-only" (for household data) is enforced by the absence of edit controls in the screens themselves (FEAT-22.SPEC-002), not by a permission check on an edit action that could otherwise exist -- there is no edit endpoint for household data in this feature to gate. The one exception is "Mark Resolved," which is not household-data editing at all: it is the permitted lifecycle action on the Support Request's own status, governed by FEAT-22.SPEC-008, not by this read-only rule.
- The one-household-at-a-time limit applies to Riley individually; the product defines a single operator role, so this is equivalent to a single global limit on FEAT-22 access at any moment.
- This spec's open-precondition rule is evaluated fresh on every attempt -- a request that qualified a moment ago but has since resolved (e.g., through a race FEAT-22.SPEC-008 otherwise prevents) is re-checked, not assumed still valid.

## Edge Cases

- **Riley has a household open and the underlying Support Request resolves through some other path while the session remains open** -- FEAT-22.SPEC-005 is the only path that resolves a request, and it always closes the open session as part of the same action (per its own Business Rules), so this scenario cannot leave an open session against a Resolved request; there is no separate resolution path this spec must additionally gate against.
- **Riley closes the currently open household and immediately attempts to open a different one** -- The close (FEAT-22.SPEC-004) completes and clears the derived access-session state before the new open's permission check runs, so the second open succeeds if the target request otherwise qualifies.
- **Two open Support Requests exist for the same household Riley already has open** -- The one-household-at-a-time rule is satisfied, since both requests belong to the household already open; selecting the second request from the queue reopens the same household view rather than being denied.
- **A request's status changes to Resolved between FEAT-22.SPEC-001 loading the queue and Riley tapping the row** -- The screen-entry check on FEAT-22.SPEC-002 re-validates status and denies the open with "This request has already been resolved and can no longer be opened.", even though the stale queue row still showed it as open.
- **Riley's browser or client is closed without an explicit Close action** -- The session is treated as still open until FEAT-22.SPEC-004 records its end on the eventual session-expiry path (FEAT-22.SPEC-004's edge cases); until that end is recorded, this spec's derived access-session state continues to read "open," correctly preventing a second household from being opened in the interim.

## Acceptance Criteria

**FEAT-22.SPEC-006-AC-01:** Given a Support Request with status Raised and Riley has no household currently open, when Riley selects it, then the open is permitted.

**FEAT-22.SPEC-006-AC-02:** Given a Support Request with status Under review and Riley has no household currently open, when Riley selects it, then the open is permitted.

**FEAT-22.SPEC-006-AC-03:** Given a Support Request with status Resolved, when Riley attempts to open it, then the open is denied with "This request has already been resolved and can no longer be opened."

**FEAT-22.SPEC-006-AC-04:** Given Riley already has household A open, when Riley attempts to open household B's request, then the open is denied with "You already have {household name} open. Close it before opening another household."

**FEAT-22.SPEC-006-AC-05:** Given Riley already has household A open, when Riley selects a second open request that also belongs to household A, then the open succeeds, since it is the same already-open household.

**FEAT-22.SPEC-006-AC-06:** Given Riley is viewing an open household session, when Riley looks for any edit control on any household data in the layout, then none exists, and there is no denied-attempt experience to trigger since no such control is present -- "Mark Resolved" remains available as the one permitted lifecycle action on the Support Request's own status, distinct from editing household data.

**FEAT-22.SPEC-006-AC-07:** Given Maya, Sam, or either Jordan role attempts to reach any FEAT-22 screen, then access is denied per that screen's own Access and Visibility table, since none of these roles is ever permitted by this spec's Authorization Rules.

**FEAT-22.SPEC-006-AC-08:** Given Riley closes household A, when Riley immediately attempts to open household B's qualifying request, then the open succeeds, since the derived access-session state has cleared.

**FEAT-22.SPEC-006-AC-09:** Given a request qualified when the queue loaded but is resolved before Riley taps it, when Riley taps the row, then the screen-entry check re-validates and denies the open.

**FEAT-22.SPEC-006-AC-10:** Given Riley's session is unexpectedly terminated without an explicit Close, when Riley attempts a second household open before the abandoned session's end is recorded, then the second open is denied under the one-household-at-a-time rule.

**FEAT-22.SPEC-006-AC-11:** Given no Support Request is open for any household, when the derived access-session state is evaluated, then it reads "not open" and any qualifying request may be opened.

**FEAT-22.SPEC-006-AC-12:** Given Riley resolves the household currently open (FEAT-22.SPEC-005), then the session closes as part of that same action, and Riley may open a different household's qualifying request immediately afterward.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

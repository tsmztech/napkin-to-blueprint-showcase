---
document_type: spec
spec_type: screen
spec_id: FEAT-22.SPEC-001
spec_name: Support Request Queue
spec_slug: support-request-queue
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Support Request Queue

## Overview

**Name:** Support Request Queue
**ID:** FEAT-22.SPEC-001
**Type:** Screen
**Purpose:** Riley sees every open Support Request across households and selects one to open the read-only household view against.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Listing every Support Request with status Raised or Under review, across all households
- Showing enough per-request context (kind, household reference, raised date, current status) for Riley to pick the right one
- Selecting a request to open FEAT-22.SPEC-002 (Support Read-Only Household View), subject to FEAT-22.SPEC-006's gating

**Non-Goals:**
- Showing Resolved requests -- excluded per the feature's own Validation & Limits: this is a queue of open work, not a history browser; resolved requests remain retained data (SC-18) but have no operator-facing surface here
- Any household data beyond what identifies the request (household name, request kind, dates) -- the household's actual setup and plan are shown only after selection, in FEAT-22.SPEC-002, per FEAT-22.SPEC-006's one-household-at-a-time gating
- Editing, reassigning, or bulk-acting on requests -- excluded per BRIEF.md's Target Users & Roles ("read-only support access... nothing more") and scope-boundaries.md SC-01: Riley has no product-facing entitlements beyond viewing and resolving one request's household at a time
- Multi-household or bulk browsing -- excluded per the feature's own Validation & Limits ("no bulk or cross-household browsing"): this screen lists requests, it never opens more than one household's data at once (FEAT-22.SPEC-006)

## Entry Points

{This screen is Riley's own landing screen -- the Brief's Default Entry -- reached only by Riley's own sign-in, since Riley is not a household member and reaches no other product surface.}

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Operator sign-in (Riley's own, separate from household member sign-in) | Riley signs in as the operator | None -- queue loads all open requests |
| FEAT-22.SPEC-002 (Support Read-Only Household View) | Riley closes or navigates away from an open household view | None -- returns to the queue with the just-closed request's status reflecting SPEC-004/SPEC-008's transition |
| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Riley taps "Review report" in the operator alert email | The household's open safety-concern Support Request, highlighted in the queue |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Riley (Operator, support) | Full screen | Select an open request to view its household | -- |
| Maya (Organiser) | No | No | This screen is not part of any household member's product surface; Maya's own visibility into support activity is FEAT-22.SPEC-003 (Household Support Access Record), reached from her own settings |
| Sam (Other Adult Member) | No | No | This screen does not exist within Sam's product surface at all |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role, so no screen is reachable |
| Jordan (older kid, limited login -- Later) | No | No | This login's product surface (Grocery List, Dinner Voting) has no path to this operator-only screen |
| Unauthenticated | No | No | Redirected to the operator sign-in screen |
| Expired session | No | No | Redirected to the operator sign-in screen; no in-progress selection survives, since selecting a request opens a session (FEAT-22.SPEC-004) only after a valid, current sign-in |

## Layout and Content

**Header:** Screen title "Support requests" with a count of open requests shown next to the title (e.g., "Support requests (4)").

**Body:** A single list of open Support Requests, most recently raised first. Each row shows:
- Household reference (household name, shown only to identify which household a request belongs to)
- Kind (a "Safety concern" or "General support" label)
- Status (Raised or Under review)
- Raised date

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** The list occupies the full screen width; each row stacks its kind label and status below the household name.
- **Medium size class and above:** The list renders as a table with household name, kind, status, and raised date in separate columns; no structural change beyond column layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Request row | Tap | Attempt to open FEAT-22.SPEC-002 for this request's household, subject to FEAT-22.SPEC-006's gating | Row shows a brief loading state | Success: navigates to FEAT-22.SPEC-002. Denied (another household already open): inline message "You already have {household name} open. Close it before opening another household." with a "Go to open household" action |
| "Go to open household" (shown on denial) | Tap | Navigate to FEAT-22.SPEC-002 for the currently open household | Screen changes | Standard transition to the already-open household view |

### Accessibility Notes

- **Focus order:** Screen title -> request rows in list order.
- **Denial announcement:** The gating-denial message is announced to assistive technology when it appears, since it interrupts an expected navigation.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Populated (default) | List of open requests as described | Screen loads with 1+ open requests | Riley selects a request or all requests resolve |
| Empty | A short message: "No open support requests right now." with no list | Screen loads with zero open requests | A new Support Request reaches Raised status |
| Loading | A brief inline loading indicator in place of the list | Screen first opens, before the list resolves | List loads (populated or empty) |
| Error | Error banner "Couldn't load support requests. Check your connection and try again." with Retry | List fails to load | Riley taps Retry (returns to Loading) |
| Offline/Degraded | Banner "You're offline -- support requests need a connection to load." List area is empty until connectivity returns | Connectivity lost while this screen is open | Connectivity restored -- the list reloads automatically |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Selection gating is governed by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules). See that spec for the exact one-household-at-a-time and open-request-precondition rules.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Request row tap (gating allows) | FEAT-22.SPEC-002 (Support Read-Only Household View) | -- |
| "Go to open household" tap | FEAT-22.SPEC-002 (Support Read-Only Household View) | -- |

## Data Model

**Creates:** None.
**Reads:** Support Request -- kind, status, household reference, created (raised) date, for every request with status Raised or Under review, across all households.
**Updates:** None -- this screen only reads the queue; status transitions happen through FEAT-22.SPEC-004 and FEAT-22.SPEC-005 once a household is opened.
**Deletes:** None.

## Business Rules

- Only requests with status Raised or Under review appear here -- governed by FEAT-22.SPEC-008 (Support Request Status Transition Rules).
- Selecting a request is always subject to FEAT-22.SPEC-006's gating: the request's household must not conflict with a household Riley already has open elsewhere.
- XBR-14: Operator support access opens only for one household with an open Support Request, is strictly read-only, and closes when the request is resolved.

## Edge Cases

- **A request's status changes to Resolved while Riley is viewing the queue (another concurrent access, or an automated resolution path)** -- The row disappears from the list on the next refresh; no action is required from Riley, and there is no conflict to resolve since this screen never writes to the Support Request.
- **Two requests exist for the same household** -- Both rows are shown; selecting either opens the same household view (FEAT-22.SPEC-002), and FEAT-22.SPEC-006's one-household-at-a-time rule is satisfied since both belong to the same already-open household.
- **Riley taps a row while a network request for the previous tap is still resolving** -- The second tap is ignored until the first resolves (row in loading state).
- **All open requests resolve while Riley is signed in with an open household session** -- The queue continues to reflect the live open/closed state; the currently open household's own request appearing as Resolved does not force-close Riley's session, which FEAT-22.SPEC-004 closes only when Riley navigates away or resolves it directly.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-002 (Support Read-Only Household View) | Navigation (outbound) | Selecting a request opens this screen for the request's household |
| FEAT-22.SPEC-006 (Support Access Scope & Gating Rules) | References (outbound) | Governs whether a selection is permitted |
| FEAT-22.SPEC-008 (Support Request Status Transition Rules) | References (outbound) | Governs which requests qualify as "open" for this list |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Navigation (inbound, cross-feature) | A safety-concern report reaches this queue |
| FEAT-18 (Account & Data Management) | Navigation (inbound, cross-feature) | A general support contact reaches this queue |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_queue_viewed | open_request_count | Screen loads | N/A -- no success-metrics.md metric traces to FEAT-22 (Operator Read-Only Support Access is a Nice-to-Have platform capability with no Connected Feature entry in success-metrics.md); retained so this trust-facing, low-volume operator surface remains observable |
| support_queue_request_selected | kind (safety concern / general support) | Riley taps an open request row and selection succeeds | N/A -- no success-metrics.md metric traces to this feature |
| support_queue_selection_denied | reason (household already open) | FEAT-22.SPEC-006 denies a selection | N/A -- no success-metrics.md metric traces to this feature; retained so gating friction is observable rather than silent |

## Acceptance Criteria

**FEAT-22.SPEC-001-AC-01:** Given Riley signs in as the operator and 3 requests are open across 2 households, when the queue loads, then all 3 rows appear, most recently raised first, with kind, status, and raised date shown for each.

**FEAT-22.SPEC-001-AC-02:** Given Riley has no open household session, when Riley taps a request row, then FEAT-22.SPEC-006's gating allows the open and Riley is navigated to FEAT-22.SPEC-002 for that household.

**FEAT-22.SPEC-001-AC-03:** Given Riley already has one household open, when Riley taps a request row for a different household, then the inline message "You already have {household name} open. Close it before opening another household." appears and no navigation occurs.

**FEAT-22.SPEC-001-AC-04:** Given Riley sees the gating-denial message, when Riley taps "Go to open household", then Riley is navigated to FEAT-22.SPEC-002 for the household already open.

**FEAT-22.SPEC-001-AC-05:** Given zero requests are open, when the queue loads, then the message "No open support requests right now." appears with no list.

**FEAT-22.SPEC-001-AC-06:** Given the queue fails to load due to a network error, when Riley views the screen, then the error banner with Retry appears, and tapping Retry reloads the list.

**FEAT-22.SPEC-001-AC-07:** Given Riley loses connectivity while the queue is open, then a banner states a connection is needed and the list area is empty until connectivity returns.

**FEAT-22.SPEC-001-AC-08:** Given a request Riley can see in the queue is resolved by another process while the queue screen is idle, when the list next refreshes, then that row no longer appears.

**FEAT-22.SPEC-001-AC-09:** Given two open requests exist for the same household, when Riley selects either one, then FEAT-22.SPEC-002 opens for that household in both cases.

**FEAT-22.SPEC-001-AC-10:** Given Maya (Organiser) or Sam attempts to reach this screen's URL directly, then they are redirected away, since this screen exists only within Riley's operator sign-in context.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 5 (populated, empty, loading, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |

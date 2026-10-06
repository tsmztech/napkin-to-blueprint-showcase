# FEAT-22 — Operator Read-Only Support Access

This chapter covers FEAT-22, Operator Read-Only Support Access, a Nice-to-Have-tier feature. It contains 10 specifications carrying 107 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-22.SPEC-001 | Support Request Queue | screen | 10 |
| FEAT-22.SPEC-002 | Support Read-Only Household View | screen | 13 |
| FEAT-22.SPEC-003 | Household Support Access Record | screen | 9 |
| FEAT-22.SPEC-004 | Support Access Session Logging | automation | 11 |
| FEAT-22.SPEC-005 | Support Request Resolution | automation | 10 |
| FEAT-22.SPEC-006 | Support Access Scope & Gating Rules | logic-rule | 12 |
| FEAT-22.SPEC-007 | Kid Profile & Billing Data Visibility Rule | logic-rule | 12 |
| FEAT-22.SPEC-008 | Support Request Status Transition Rules | logic-rule | 11 |
| FEAT-22.SPEC-009 | Support View Recorded Notification | notification | 10 |
| FEAT-22.SPEC-010 | Support Request Resolved Notification | notification | 9 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Operator Read-Only Support Access

## Summary

**Feature:** Operator Read-Only Support Access
**ID:** FEAT-22
**Description:** The founder, acting as operator, can view a household's setup and plan in read-only form to help diagnose a reported problem — and nothing more.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** Platform
**Rationale:** The brief names this directly: "the founder, as operator, needs only read-only support access to help a household. Nothing more." (BRIEF.md, Target Users & Roles). Nice-to-Have and phased to v1 because the product can launch and be supported manually at very small scale; a dedicated read-only view becomes worth building once household volume makes ad hoc support impractical.

**Key Capabilities:**
- View household setup — Operator sees a household's setup and plan to diagnose a specific reported issue
- Nothing more — Operator cannot edit any household data, view billing detail beyond plan tier, or access kid profile data beyond what a specific safety report requires
- Open only against a request — Support access opens only for a household with an open Support Request, and closes when the request is resolved
- Leave a visible record — Each time support views a household, the organiser can see when and why in the household's settings

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-22.SPEC-001 | Support Request Queue | Screen | Operator (Support) | Operator sees open Support Requests across households and selects one to act on |
| FEAT-22.SPEC-002 | Support Read-Only Household View | Screen | Operator (Support) | Operator views a specific household's setup, plan, and related data in read-only form to diagnose the open request |
| FEAT-22.SPEC-003 | Household Support Access Record | Screen | Organiser | Organiser views the record of when and why support viewed their household, entered from household settings |
| FEAT-22.SPEC-004 | Support Access Session Logging | Automation | Operator (Support), Organiser | System records the start and end of every operator view of a household, and moves a Raised request to Under review on first view |
| FEAT-22.SPEC-005 | Support Request Resolution | Automation | Operator (Support) | Operator marks an open Support Request Resolved, closing any open access session for it |
| FEAT-22.SPEC-006 | Support Access Scope & Gating Rules | Logic/Rule | Operator (Support) | Governs when access may open, that it is strictly read-only with no edit actions, and that only one household is open at a time |
| FEAT-22.SPEC-007 | Kid Profile & Billing Data Visibility Rule | Logic/Rule | Operator (Support) | Governs what the read-only view suppresses: kid profile detail beyond a specific safety report's allergy facts, and billing detail beyond plan tier |
| FEAT-22.SPEC-008 | Support Request Status Transition Rules | Logic/Rule | Operator (Support) | Governs the Raised → Under review → Resolved lifecycle of a Support Request |
| FEAT-22.SPEC-009 | Support View Recorded Notification | Notification | Organiser | Organiser is shown an in-app note each time support viewed their household |
| FEAT-22.SPEC-010 | Support Request Resolved Notification | Notification | Organiser | Organiser is told when their Support Request is resolved |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View household setup | FEAT-22.SPEC-001, FEAT-22.SPEC-002 | Queue lets the operator select the household with the open request; the household view then shows its setup, plan, and related read-only data | Phase 2 (Explicit) |
| Nothing more | FEAT-22.SPEC-006, FEAT-22.SPEC-007 | Gating rules keep the view free of any edit action; the visibility rule suppresses kid profile detail and billing detail beyond plan tier | Phase 5 (Rule Discovery) |
| Open only against a request | FEAT-22.SPEC-006, FEAT-22.SPEC-004, FEAT-22.SPEC-008 | Gating rule permits access only while a request is Raised or Under review, one household at a time; session logging and status transitions enforce and reflect this | Phase 5 (Rule Discovery) |
| Leave a visible record | FEAT-22.SPEC-004, FEAT-22.SPEC-003, FEAT-22.SPEC-009 | Every session open/close is logged; the organiser's household settings surface the record; an in-app note fires on each view | Phase 4 (Trigger-Response / Notification Surfacing) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-22.SPEC-005 | Support Request Resolution | Phase 3 (Entity-Lifecycle) | The CRUD matrix for Support Request needs an Update operation with a real owner; "closes when the request is resolved" implies an explicit resolution action, not a passive state |
| FEAT-22.SPEC-008 | Support Request Status Transition Rules | Phase 3 (Entity-Lifecycle) | The State Transition cell of the CRUD matrix (Raised → Under review → Resolved) needed an explicit governing rule; the feature description names the endpoints but not the transition trigger |
| FEAT-22.SPEC-010 | Support Request Resolved Notification | Phase 4 (Notification Surfacing) | The Communications field states the organiser "is told when their Support Request is resolved" — a named message with an audience and a trigger, so it needed its own Notification spec rather than staying inline in SPEC-005 |

## Entity-Lifecycle Coverage Matrix

**Entity: Support Request**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Created by Dietary Rules & Allergy Safety Engine (FEAT-02, safety concerns) and Account & Data Management (FEAT-18, general support contact) — not by this feature | This feature only reads and updates status/access-record fields, per the dependency map's "Managed by" line |
| Read (single) | FEAT-22.SPEC-002 | The household view surfaces the open request's detail (kind, note, planned meal/recipe for safety concerns) as the diagnostic context | -- |
| Read (list) | FEAT-22.SPEC-001 | Queue lists open (Raised, Under review) requests across households | -- |
| Update | FEAT-22.SPEC-005 | Operator resolves the request, which sets status and finalizes the access record; the first view also updates status (see State Transition) | Only status and access-record fields are ever written by this feature — never household data (dependency map: "Updated by FEAT-22 (status and access record only)") |
| Delete/Archive | N/A -- non-goal | No feature in the dependency map deletes or archives a Support Request; it is retained as history rather than purged | See Non-Goals: retention is consistent with ASMP-24's account-lifetime history depth |
| State Transition | FEAT-22.SPEC-008 | Raised → Under review (on the operator's first view, via SPEC-004) → Resolved (via SPEC-005) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-22.SPEC-002 | Household setup (name, budget, schedule, unit system, currency, aisle names, plan-arrival timing) shown to diagnose the reported problem |
| Member Profile | FEAT-22.SPEC-002, FEAT-22.SPEC-007 | Adult and kid member details shown subject to the visibility rule's kid-data suppression |
| Weekly Plan | FEAT-22.SPEC-002 | The current plan (and, where relevant, the meal named in a safety-concern request) shown read-only |
| Grocery List | FEAT-22.SPEC-002 | The household's current list shown read-only, per the operator's View access to Grocery List |
| Rating | FEAT-22.SPEC-002 | Household ratings shown read-only to help diagnose a reported problem, per the operator's View access to Ratings |
| Subscription | FEAT-22.SPEC-002, FEAT-22.SPEC-007 | Plan tier only is shown; the visibility rule blocks payment and billing-history detail |
| Pantry Item | FEAT-22.SPEC-002 | Shown read-only per the operator's View access to Pantry Input (Access Matrix) |
| Recipe | FEAT-22.SPEC-002 | Shown read-only per the operator's View access to Recipe Library (Access Matrix) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Operator opens the read-only view against an open Support Request | Validate the request is Raised or Under review for exactly one household; if valid, start an access session and log its start | Standalone Automation | FEAT-22.SPEC-004 |
| Operator's session-opening view is the request's first view | Transition the request from Raised to Under review | Standalone Logic/Rule (state transition), executed by the session automation | FEAT-22.SPEC-008 / FEAT-22.SPEC-004 |
| Operator closes or navigates away from the read-only view | End the access session and log its end timestamp | Standalone Automation | FEAT-22.SPEC-004 |
| Access session opens or closes | Show the organiser an in-app note that support viewed the household, with when and why | Standalone Notification | FEAT-22.SPEC-009 |
| Read-only view renders any household data | Suppress kid profile detail beyond the open safety-concern request's allergy facts, and billing detail beyond plan tier | Standalone Logic/Rule | FEAT-22.SPEC-007 |
| Operator attempts to open a household with no open Support Request, or a second household while one is already open | Deny the attempt; only one open household at a time, only against an open request | Standalone Logic/Rule | FEAT-22.SPEC-006 |
| Operator marks a Support Request Resolved | Update status to Resolved, close any still-open access session for it, and notify the organiser of the outcome | Standalone Automation + Standalone Notification | FEAT-22.SPEC-005, FEAT-22.SPEC-010 |
| Household reports a safety concern about a specific meal | A Support Request reaches the operator's queue | Cross-feature | FEAT-02 responsibility (creates the request; surfaced in FEAT-22.SPEC-001) |
| Household submits a general support contact | A Support Request reaches the operator's queue | Cross-feature | FEAT-18 responsibility (creates the request; surfaced in FEAT-22.SPEC-001) |

## Shared Context

**Shared Entities:**
- Support Request -- read (list) by SPEC-001, read (single) by SPEC-002, updated by SPEC-005, its status transitions governed by SPEC-008, its access-record fields written by SPEC-004 and displayed by SPEC-003. Fields (functional): kind, raised_by, planned_meal/recipe, note, status, access_record.
- Household, Member Profile, Weekly Plan, Grocery List, Rating, Subscription, Pantry Item, Recipe -- all read-only, displayed by SPEC-002 subject to the visibility rule (SPEC-007) and the gating rule (SPEC-006).

**Shared UI Patterns:**
- Read-only display, no edit affordances -- SPEC-002 (household view) and SPEC-003 (access record) both present data with zero edit controls anywhere in the layout; Spec Writers for both should describe this consistently as an absence, not a disabled state, so it is never mistaken for a permissions-driven grey-out.
- Access-scoped entry -- SPEC-001 and SPEC-002 both gate entry through SPEC-006; neither screen should duplicate the gating logic, only reference it.

**Shared Validation:**
- SPEC-006 defines the access-gating rules (open-request precondition, one-household-at-a-time, no edit actions). SPEC-001, SPEC-002, and SPEC-004 all reference SPEC-006 rather than re-deriving gating behavior.
- SPEC-007 defines the kid-data and billing visibility rule. SPEC-002 references SPEC-007 for every field it renders rather than deciding visibility inline.

## Internal Dependency Map

```
SPEC-001 (Support Request Queue) -> [operator selects an open request] -> SPEC-002 (Support Read-Only Household View)
SPEC-001 (Support Request Queue) -> [governed by] -> SPEC-006 (Support Access Scope & Gating Rules)
SPEC-002 (Support Read-Only Household View) -> [governed by] -> SPEC-006 (Support Access Scope & Gating Rules)
SPEC-002 (Support Read-Only Household View) -> [governed by] -> SPEC-007 (Kid Profile & Billing Data Visibility Rule)
SPEC-002 (Support Read-Only Household View) -> [opens] -> SPEC-004 (Support Access Session Logging)
SPEC-002 (Support Read-Only Household View) -> [operator navigates away] -> SPEC-004 (Support Access Session Logging)
SPEC-002 (Support Read-Only Household View) -> [operator marks the request Resolved] -> SPEC-005 (Support Request Resolution)
SPEC-004 (Support Access Session Logging) -> [first view of the request] -> SPEC-008 (Support Request Status Transition Rules)
SPEC-004 (Support Access Session Logging) -> [session opens or closes] -> SPEC-009 (Support View Recorded Notification)
SPEC-004 (Support Access Session Logging) -> [writes access_record] -> SPEC-003 (Household Support Access Record)
SPEC-005 (Support Request Resolution) -> [status change] -> SPEC-008 (Support Request Status Transition Rules)
SPEC-005 (Support Request Resolution) -> [closes any open session] -> SPEC-004 (Support Access Session Logging)
SPEC-005 (Support Request Resolution) -> [request resolved] -> SPEC-010 (Support Request Resolved Notification)
```

**Default Entry:** SPEC-001 (Support Request Queue) -- the screen shown when the operator opens support access; SPEC-003 (Household Support Access Record) is instead entered from FEAT-01's household settings, on the organiser's side.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-22.SPEC-001 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | A safety-concern Support Request reaches the operator's queue | Household reports a safety concern about a specific meal |
| FEAT-22.SPEC-001 | Inbound | FEAT-18 (Account & Data Management) | A general support-contact Support Request reaches the operator's queue | Household sends a support contact message |
| FEAT-22.SPEC-002 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Read-only view displays household setup and member profiles | Operator opens access against an open request |
| FEAT-22.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Read-only view displays the household's weekly plan | Operator opens access against an open request |
| FEAT-22.SPEC-002 | Inbound | FEAT-06 (Shared Grocery List) | Read-only view displays the household's grocery list | Operator opens access against an open request |
| FEAT-22.SPEC-003 | Outbound | FEAT-01 (Household Setup & Member Profiles) | Organiser opens the support access record from household settings | Organiser taps "support access" in household settings |
| FEAT-22.SPEC-005 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Resolving a safety-concern Support Request closes the case FEAT-02 opened, per XBR-08 | Operator marks a safety-concern request Resolved |

## Non-Functional Notes

**Data volumes / growth:** ASMP-24 projects several thousand households in year one, but this feature's own usage stays low regardless of that growth: Behavioral Context describes support access as "rare, on-demand use triggered by a household's support request or safety-concern report; never a routine or scheduled interaction," and Validation & Limits scopes it to one household at a time with no bulk or cross-household browsing.

**Responsiveness:** N/A — the feature's States field marks Loading N/A ("a lightweight read-only view with no heavy computation"), and assumptions-constraints.md's Non-Functional Expectations name no distinct responsiveness target for the operator support surface beyond the product's general expectations.

**Data sensitivity / privacy:** Support Request data may contain children's allergy details and carries children's-privacy-class protection (dependency map, Data Sensitivity); SPEC-007 implements the household-privacy commitment in ASMP-26 by ensuring Riley sees kid profile data only inside a specific safety report, never a kid's broader profile. Household, Weekly Plan, and Grocery List are private household data never sold or used for advertising (ASMP-14, ASMP-26), and Riley's read access to them is gated to the life of one open request (SPEC-006).

**Compliance flags:** ASMP-27's children's-privacy-class protections apply to the allergy details Riley may see inside a safety-concern request; ASMP-14's no-sale, no-advertising constraint applies to every entity this feature reads. No payment-data compliance regime is engaged here, since Riley's Billing view is capped to plan tier and never reaches payment details (dependency map, Subscription Data Sensitivity).

## Non-Goals

- **Any edit capability for the operator** -- Excluded per BRIEF.md's Target Users & Roles ("the founder, as operator, needs only read-only support access... Nothing more") and scope-boundaries.md SC-01 (Operator as a full product role is out of scope): the view is deliberately built with no edit controls at all (SPEC-006), so there is no path by which read-only access could accidentally become a change.
- **Multi-household or bulk browsing by the operator** -- Excluded per the feature's own Validation & Limits field ("Access is scoped to one household at a time... no bulk or cross-household browsing"), reinforced by scope-boundaries.md SC-03's single-household-per-account product model: the gating rule (SPEC-006) permits exactly one open household at a time.
- **Automatic purge of Support Request history** -- Intentional lifecycle decision surfaced by the Entity-Lifecycle Coverage Matrix: no feature in the dependency map deletes or archives a Support Request, so resolved requests and their access records are retained rather than purged, consistent with ASMP-24's "every household's history is kept for the life of the household account."
- **In-product messaging or chat as the support-visit communication channel** -- Excluded per scope-boundaries.md SC-14: coordination and communication in Plateful is delivered through named features (here, in-app notes per SPEC-009 and SPEC-010), not a general messaging layer between the household and the operator.



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



# Screen Spec: Support Read-Only Household View

## Overview

**Name:** Support Read-Only Household View
**ID:** FEAT-22.SPEC-002
**Type:** Screen
**Purpose:** Riley views one household's setup, plan, and related data in read-only form, with no edit controls anywhere, to diagnose the open Support Request.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Displaying the household's setup, weekly plan, grocery list, ratings, pantry items, recipes, and plan tier, all read-only, for the household behind the open Support Request
- Suppressing kid profile detail and billing detail per FEAT-22.SPEC-007
- Starting and ending the access session that FEAT-22.SPEC-004 logs
- The "Mark Resolved" action that hands off to FEAT-22.SPEC-005

**Non-Goals:**
- Any edit action on any household data displayed on this screen (household setup, member profiles, weekly plan, grocery list, ratings, pantry items, recipes, subscription tier) -- excluded per BRIEF.md's Target Users & Roles and FEAT-22.SPEC-006: this diagnostic content is read-only with zero edit affordances anywhere in the layout, described as an absence rather than a disabled state. This is distinct from "Mark Resolved," the one permitted lifecycle action, which changes only the Support Request's own status (governed by FEAT-22.SPEC-008), never any household data.
- Viewing a second household at the same time, or any household with no open request -- governed entirely by FEAT-22.SPEC-006
- The access-session log itself and its display to the organiser -- owned by FEAT-22.SPEC-004 (logging) and FEAT-22.SPEC-003 (the organiser's own view of the record)
- Payment details or billing history beyond plan tier, and any kid profile detail beyond the allergy fact tied to an open safety-concern request -- governed entirely by FEAT-22.SPEC-007

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-22.SPEC-001 (Support Request Queue) | Riley selects an open request row, and FEAT-22.SPEC-006's gating allows the open | The selected household and its open Support Request (kind, note, planned meal/recipe for safety concerns) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Riley (Operator, support) | Full screen, subject to FEAT-22.SPEC-007's suppressions | Mark the open request Resolved; close the view | -- |
| Maya (Organiser) | No | No | This screen does not exist within Maya's product surface; her own household screens are the live, editable versions of this data |
| Sam (Other Adult Member) | No | No | This screen does not exist within Sam's product surface |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role |
| Jordan (older kid, limited login -- Later) | No | No | This login's product surface has no path to this operator-only screen |
| Unauthenticated | No | No | Redirected to the operator sign-in screen |
| Expired session | No | No | The open access session is ended (FEAT-22.SPEC-004) and Riley is redirected to the operator sign-in screen; re-authenticating returns Riley to FEAT-22.SPEC-001, not directly back into this view, since re-entry must pass FEAT-22.SPEC-006's gating again |

## Layout and Content

**Header:** The household's name, the open request's kind and note (for safety concerns: the reported meal and recipe name), and a "Mark Resolved" action (right-aligned). No back arrow; a "Close" control (top-left) ends the session and returns to FEAT-22.SPEC-001.

**Body:** A single-column, section-by-section read-only display, in this order:

- **Household setup:** household name, weekly budget, weekly schedule, unit system, currency, aisle names, plan-arrival day and time.
- **Members:** each adult member's display_name and dietary rules in full; each kid member shown per FEAT-22.SPEC-007's suppression (no display_name, age_band, or broader profile -- only the specific allergy fact tied to an open safety-concern request naming that meal, attributed to "a household kid member," never a name).
- **Weekly plan:** the current week's plan, each Planned Meal's night, recipe, cook time, rough cost, and status; for a safety-concern request, the specific reported meal is highlighted.
- **Grocery list:** the household's current list, read-only.
- **Ratings:** the household's recorded meal ratings, read-only.
- **Pantry items:** the household's current pantry list, read-only.
- **Recipes:** the household's imported recipes, read-only, alongside starter-library recipes referenced by the plan.
- **Subscription:** plan tier only (free or paid) -- no billing period, billing state, billing history, or payment detail (FEAT-22.SPEC-007).

No edit controls, input fields, or action buttons appear anywhere within these household-data sections -- every element below the header is display-only. (The header's "Mark Resolved" action is the one exception in the layout, and it acts only on the Support Request's own status, never on any household data shown below it.)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order listed, each collapsible to a summary row that expands on tap; "Mark Resolved" remains in the header.
- **Medium size class and above:** Sections render side by side in two columns (household setup and members in one column; plan, list, and the rest in the other), with no structural change beyond column layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | End the access session (FEAT-22.SPEC-004) and navigate to FEAT-22.SPEC-001 | Screen closes | Standard transition back to the queue |
| Section header (compact only) | Tap | Expand or collapse the section | Section content shows or hides | Chevron icon rotates |
| "Mark Resolved" | Tap | Open the resolution confirmation: for a safety-concern request, present two outcome options ("Recipe is safe" / "Recipe is unsafe"); for a general support request, present a single confirmation | Dialog opens | Dialog title "Mark this request resolved?" |
| Resolution dialog confirm | Tap | Trigger FEAT-22.SPEC-005 (Support Request Resolution) with the request and (for safety concerns) the selected outcome | Button shows a brief loading state | Success: dialog closes, the header updates to show the request as Resolved, and "Mark Resolved" is replaced with a "Resolved" label. Failure: inline error in the dialog with Retry |
| Resolution dialog cancel | Tap | Dismiss the dialog without resolving | Dialog closes | Returns to the household view unchanged |

### Accessibility Notes

- **Focus order:** Close control -> "Mark Resolved" -> household setup section -> members -> weekly plan -> grocery list -> ratings -> pantry items -> recipes -> subscription.
- **Section state announcements:** Expanding or collapsing a section announces its new state ("Household setup, expanded" / "collapsed") to assistive technology.
- **Resolution feedback:** The resolved confirmation and any dialog error are announced when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A brief inline loading indicator in place of the sections | Screen first opens, before household data resolves | Data loads (populated or error) |
| Populated (default) | All sections shown with current household data | Data loads successfully | Riley closes the view or marks the request Resolved |
| Resolving | "Mark Resolved" dialog shows a loading state during submission | Riley confirms resolution | Resolution completes or fails |
| Resolved | Header shows the request as Resolved; "Mark Resolved" replaced with a static "Resolved" label; the rest of the view remains viewable read-only until Riley closes it | Resolution completes successfully | Riley taps Close |
| Error (dialog) | Inline error "Couldn't mark this request resolved. Check your connection and try again." with Retry | Resolution submission fails | Riley taps Retry (returns to Resolving) or Cancel |
| Offline/Degraded | Banner "You're offline -- this household's data needs a connection to load or refresh." Sections already loaded remain visible but cannot refresh; "Mark Resolved" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears, "Mark Resolved" re-enables, and data refreshes |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Access gating is governed by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules). Data visibility and suppression is governed by FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule). Status transition validity for "Mark Resolved" is governed by FEAT-22.SPEC-008 (Support Request Status Transition Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Close control tap | FEAT-22.SPEC-001 (Support Request Queue) | -- |
| Successful resolution, Close tapped afterward | FEAT-22.SPEC-001 (Support Request Queue) | -- |

## Data Model

**Creates:** None directly -- opening this screen triggers FEAT-22.SPEC-004 to create an access-session entry.
**Reads:** Household -- household_name, weekly_budget, weekly_schedule, unit_system, currency, aisle_names, plan_arrival_day_time. Member Profile -- display_name and Dietary Rules for adults; kid fields per FEAT-22.SPEC-007. Weekly Plan and Planned Meal -- current week's dinners and their fields. Grocery List and Grocery List Item -- current list. Rating -- household ratings. Pantry Item -- current pantry list. Recipe -- household's imported and referenced starter recipes. Subscription -- tier only. Support Request -- the open request's kind, note, planned_meal/recipe.
**Updates:** None directly by this screen -- FEAT-22.SPEC-004 writes the access_record on open/close, and FEAT-22.SPEC-005 writes status on resolution.
**Deletes:** None.

## Business Rules

- No household data on this screen is editable -- there is no edit affordance for any diagnostic content (household setup, members, plan, list, ratings, pantry, recipes, subscription) anywhere in the layout, per FEAT-22.SPEC-006. "Mark Resolved" is the one permitted lifecycle action in the layout, and it writes only the Support Request's own status, never any household data.
- Every field shown is subject to FEAT-22.SPEC-007's suppression rules; a field this spec does not explicitly list as suppressed is shown as-is.
- Opening this screen always starts an access session (FEAT-22.SPEC-004); closing it always ends that session.
- The first time a given Support Request's household is opened, FEAT-22.SPEC-008 transitions the request from Raised to Under review, via FEAT-22.SPEC-004.
- XBR-14: Operator support access is strictly read-only, never shows payment details or kid profile data beyond the allergy details in a specific safety report, and closes when the request is resolved.

## Edge Cases

- **The organiser edits the household's data while Riley is viewing it** -- No conflict exists: this screen never writes to any shared entity, so there is nothing for a concurrent household edit to collide with. The screen does not force-refresh mid-view; Riley sees the data as loaded until closing and reopening, since the feature's use is described as rare, on-demand, and not requiring live synchronization.
- **Riley taps "Mark Resolved" twice in rapid succession** -- The second tap is ignored while the first is processing (dialog in loading state).
- **The Support Request is resolved by a concurrent process (e.g., a rare race with another operator session, though FEAT-22.SPEC-006 permits only one) while this view is open** -- The header updates to show Resolved on the next load; Riley may still close the view normally, and no error is shown since resolution is itself the intended outcome.
- **Network failure while loading the household's data** -- Error banner: "Couldn't load this household's data. Check your connection and try again." with Retry; no partial edit state exists to lose, since the screen has never accepted input.
- **Riley closes the view mid-load, before data finishes resolving** -- The access session's end timestamp is recorded at the moment of closing regardless of load state; no partial data is left displayed since the screen unmounts entirely.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Support Request Queue) | Navigation (inbound) | Entry point when Riley selects a request |
| FEAT-22.SPEC-006 (Support Access Scope & Gating Rules) | References (inbound) | Governs whether this screen may open at all |
| FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule) | References (inbound) | Governs every field's visibility on this screen |
| FEAT-22.SPEC-004 (Support Access Session Logging) | Triggers (outbound) | Opening and closing this screen starts and ends the logged session |
| FEAT-22.SPEC-005 (Support Request Resolution) | Triggers (outbound) | "Mark Resolved" hands off to this automation |
| FEAT-22.SPEC-008 (Support Request Status Transition Rules) | References (outbound) | Governs the Raised -> Under review -> Resolved transitions this screen triggers or displays |
| FEAT-01.SPEC-010 (Household Settings Hub) | References (inbound) | This screen's household setup section reads the same household facts (name, budget, schedule) FEAT-01.SPEC-010 maintains, in read-only form |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | This screen's Members section reads the same member profile fields FEAT-01.SPEC-005 maintains, subject to FEAT-22.SPEC-007's kid-data suppression |
| FEAT-03.SPEC-001 (Weekly Plan View) | References (inbound) | This screen's Weekly plan section reads the same current-week Planned Meal data FEAT-03.SPEC-001 displays to the household, in read-only form |
| FEAT-06.SPEC-001 (Grocery List) | References (inbound) | This screen's Grocery list section reads the same current-week Grocery List FEAT-06.SPEC-001 displays to the household, in read-only form |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_household_view_opened | request kind (safety concern / general support) | Screen opens successfully | N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing operator surface's actual use is observable |
| support_household_view_closed | duration | Riley closes the view | N/A -- no success-metrics.md metric traces to this feature |
| support_request_resolved_from_view | kind, outcome (for safety concerns) | Resolution completes successfully | N/A -- no success-metrics.md metric traces to this feature |

## Acceptance Criteria

**FEAT-22.SPEC-002-AC-01:** Given Riley selects an open safety-concern request from the queue, when this screen opens, then the header shows the household name, the request's kind, note, and the reported meal, and the weekly plan section highlights that meal.

**FEAT-22.SPEC-002-AC-02:** Given Riley is viewing a household, when Riley looks anywhere in the household-data sections (setup, members, plan, list, ratings, pantry, recipes, subscription) for an edit control, then none exists in any section -- "Mark Resolved" in the header remains the one exception, acting only on the Support Request's own status.

**FEAT-22.SPEC-002-AC-03:** Given the household has a kid member with an allergy relevant to the open safety-concern request, when Riley views the Members section, then only the allergy fact tied to the reported meal is shown, attributed to "a household kid member," with no name, age, or other kid profile detail.

**FEAT-22.SPEC-002-AC-04:** Given the household is on the paid tier, when Riley views the Subscription section, then only "Paid" is shown, with no billing period, billing state, billing history, or payment detail.

**FEAT-22.SPEC-002-AC-05:** Given Riley opens this screen, then FEAT-22.SPEC-004 records the start of an access session for this household and Support Request.

**FEAT-22.SPEC-002-AC-06:** Given Riley taps Close, then the access session's end is recorded by FEAT-22.SPEC-004 and Riley returns to FEAT-22.SPEC-001.

**FEAT-22.SPEC-002-AC-07:** Given the open request is a safety concern, when Riley taps "Mark Resolved", then the dialog presents "Recipe is safe" and "Recipe is unsafe" as the two outcome options.

**FEAT-22.SPEC-002-AC-08:** Given the open request is a general support contact, when Riley taps "Mark Resolved", then the dialog presents a single confirmation with no outcome options.

**FEAT-22.SPEC-002-AC-09:** Given Riley confirms resolution, when it completes successfully, then the header shows the request as Resolved and "Mark Resolved" is replaced with a static "Resolved" label.

**FEAT-22.SPEC-002-AC-10:** Given resolution submission fails due to a network error, when Riley views the dialog, then the inline error with Retry appears and the request remains open.

**FEAT-22.SPEC-002-AC-11:** Given Riley loses connectivity while viewing an already-loaded household, then the offline banner appears, already-loaded sections remain visible, and "Mark Resolved" is disabled until connectivity returns.

**FEAT-22.SPEC-002-AC-12:** Given household data fails to load due to a network error, when Riley opens this screen, then the error banner with Retry appears in place of the sections.

**FEAT-22.SPEC-002-AC-13:** Given Riley closes the view before data finishes loading, then the access session's end is still recorded and Riley returns to FEAT-22.SPEC-001.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (loading, populated, resolving, resolved, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Screen Spec: Household Support Access Record

## Overview

**Name:** Household Support Access Record
**ID:** FEAT-22.SPEC-003
**Type:** Screen
**Purpose:** Maya views the record of every time support viewed her household, when and why, entered from her household settings.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- Listing every access session logged by FEAT-22.SPEC-004 for this household, with its start and end time and the Support Request it belongs to
- Showing the reason (the request's kind and, where present, its note) alongside each session
- Read-only display, entered from FEAT-01.SPEC-010 (Household Settings Hub)

**Non-Goals:**
- Any control over support access itself (ending a session early, blocking future access) -- excluded per XBR-14: the organiser's Support View access level is View, not Full; only Riley's read-only view (FEAT-22.SPEC-002) and its own gating (FEAT-22.SPEC-006) govern when access opens and closes
- Sam's access to this record -- excluded per the Access Matrix's Support View column (Sam: None); this screen is Maya's alone
- The underlying household data Riley viewed -- this screen shows only that access occurred and why, never a replay of what Riley saw, since that would recreate the very household-data exposure this feature exists to bound
- Any general support Support Request that never opened an access session (e.g., resolved before Riley opened the household) -- excluded per this feature's own definition: the access record tracks sessions, not the request's status history in general, which is visible instead through the request's own resolution notice (FEAT-22.SPEC-010)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Maya taps "Support access record" | None -- screen loads this household's full access history |
| FEAT-22.SPEC-009 (Support View Recorded Notification) | Maya taps "View record" | None -- screen loads this household's full access history |
| FEAT-22.SPEC-010 (Support Request Resolved Notification) | Maya taps "View record" | None -- screen loads this household's full access history |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | View only -- no actions beyond viewing | -- |
| Sam (Other Adult Member) | No | No | "Support access record" is not shown as an option within Sam's household settings, per the Access Matrix's Support View column (None for Sam) |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role |
| Jordan (older kid, limited login -- Later) | No | No | This login's product surface (Grocery List, Dinner Voting) has no path to household settings |
| Riley (Operator, support) | No | No | This screen is the organiser's own view of the record; Riley's read-only access is a separate screen (FEAT-22.SPEC-002) that never shows this record |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no data was being entered here, so nothing is lost |

## Layout and Content

**Header:** Screen title "Support access record" with a back arrow (returns to FEAT-01.SPEC-010).

**Body:** A single list of access sessions, most recent first. Each entry shows:
- Start and end time (or "In progress" if the session has not yet ended)
- The associated Support Request's kind ("Safety concern" or "General support") and, for safety concerns, the reported meal's name
- The request's current status (Raised, Under review, Resolved)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Entries stack vertically, full width, each showing all fields listed above in a single card.
- **Medium size class and above:** Entries render as a table with start/end time, kind, reported meal (safety concerns only), and status in separate columns; no structural change beyond column layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard transition back |

### Accessibility Notes

- **Focus order:** Back arrow -> record entries in most-recent-first order.
- **In-progress announcement:** An entry whose session has not yet ended announces "In progress" as its end-time value to assistive technology, rather than being left blank.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Populated (default) | List of access sessions as described | Screen loads with 1+ logged sessions | Maya navigates away |
| Empty | Message "Support has never accessed your household." with no list | Screen loads with zero logged sessions | A new access session is logged |
| Loading | Brief inline loading indicator in place of the list | Screen first opens, before the record resolves | Record loads (populated or empty) |
| Error | Error banner "Couldn't load the support access record. Check your connection and try again." with Retry | Record fails to load | Maya taps Retry (returns to Loading) |
| Offline/Degraded | Banner "You're offline -- the support access record needs a connection to load." List area is empty until connectivity returns | Connectivity lost while this screen is open | Connectivity restored -- the record loads automatically |

## Validation Rules

**Option B -- Inline (no user input exists on this read-only screen; no fields to validate).**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|----------------|
| N/A -- read-only screen | This screen accepts no input | -- | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 (Household Setup & Member Profiles) |

## Data Model

**Creates:** None.
**Reads:** Support Request -- access_record (each logged session's start/end timestamp), kind, planned_meal/recipe (for safety concerns), status, all for Support Requests belonging to this household.
**Updates:** None.
**Deletes:** None.

## Business Rules

- This screen shows the organiser's View-level visibility into FEAT-22's access log, per XBR-14 -- Maya sees when and why Riley viewed the household, with no control over the access itself.
- A session's entry is written by FEAT-22.SPEC-004 and never edited by this screen or any other -- this screen only reads what FEAT-22.SPEC-004 has already recorded.
- Access records are retained for the life of the household account, per SC-18 and the Entity-Lifecycle Coverage Matrix's Delete/Archive: N/A finding for Support Request.

## Edge Cases

- **A new access session is logged while Maya is viewing this screen** -- The list does not auto-insert the new entry mid-view; Maya sees it on the next screen load, consistent with this being a rare, on-demand record rather than a live-updating feed.
- **The household has multiple Support Requests, each with its own sessions** -- All sessions across all of the household's Support Requests appear together in one chronological list, not grouped or filtered by request.
- **A session is still in progress (Riley has not yet closed the household view) when Maya opens this screen** -- The entry shows "In progress" as its end time rather than a blank or a stale value.
- **Network failure while loading the record** -- Error banner: "Couldn't load the support access record. Check your connection and try again." with Retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound) | Entry point from household settings |
| FEAT-22.SPEC-004 (Support Access Session Logging) | References (inbound) | Writes every entry this screen displays |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_access_record_viewed | entry_count | Screen loads | N/A -- no success-metrics.md metric traces to FEAT-22; retained so the organiser's actual use of this trust-facing record is observable |

## Acceptance Criteria

**FEAT-22.SPEC-003-AC-01:** Given Maya taps "Support access record" from household settings, when the screen loads and 2 past sessions exist, then both entries appear, most recent first, each showing start/end time, kind, and status.

**FEAT-22.SPEC-003-AC-02:** Given a safety-concern session is logged, when Maya views its entry, then the reported meal's name is shown alongside the kind.

**FEAT-22.SPEC-003-AC-03:** Given no session has ever been logged for Maya's household, when the screen loads, then "Support has never accessed your household." is shown with no list.

**FEAT-22.SPEC-003-AC-04:** Given Riley currently has this household's view open, when Maya loads this screen, then the corresponding entry shows "In progress" as its end time.

**FEAT-22.SPEC-003-AC-05:** Given Sam attempts to reach this screen directly, then it is not accessible to him, since "Support access record" is not shown in his household settings.

**FEAT-22.SPEC-003-AC-06:** Given the record fails to load due to a network error, when Maya views the screen, then the error banner with Retry appears.

**FEAT-22.SPEC-003-AC-07:** Given Maya loses connectivity while viewing this screen, then the offline banner appears and the list area remains empty until connectivity returns.

**FEAT-22.SPEC-003-AC-08:** Given Maya's household has Support Requests of both kinds with their own sessions, when the screen loads, then all sessions from all requests appear together in one chronological list.

**FEAT-22.SPEC-003-AC-09:** Given Maya taps the back arrow, then she returns to FEAT-01.SPEC-010 (Household Settings Hub).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 1 | 1 |
| States | 5 (populated, empty, loading, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



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



# Logic/Rule Spec: Kid Profile & Billing Data Visibility Rule

## Overview

**Name:** Kid Profile & Billing Data Visibility Rule
**ID:** FEAT-22.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs what FEAT-22.SPEC-002 suppresses -- kid profile detail beyond a specific safety report's allergy facts, and billing detail beyond plan tier.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access
**Governed Entity:** Member Profile (kid-type rows) and Subscription

## Scope and Non-Goals

**In Scope:**
- Field-by-field visibility of Member Profile for kid-type rows within FEAT-22.SPEC-002
- Field-by-field visibility of Subscription within FEAT-22.SPEC-002
- The narrow exception that lets a specific allergy fact through when tied to an open safety-concern request
- Authorization for viewing each suppressed and non-suppressed field

**Non-Goals:**
- Visibility of adult Member Profile fields -- not restricted by this spec; adults' data is shown in full within FEAT-22.SPEC-002, since the Access Matrix's Kid Profile Data restriction applies only to kid rows
- Whether access opens at all -- owned by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules); this spec governs only what is shown once access has already been permitted
- The organiser's own visibility into Member Profile or Subscription data -- excluded per the Access Matrix: Maya's Household Setup and Billing access are Full, entirely unaffected by this spec, which governs Riley's read-only view alone
- Payment-processing data itself (card details, transaction records) -- excluded per the dependency map's Subscription Data Sensitivity note: payment details are held by the payment-processing capability and are never modeled as a field this or any other spec's screens display to anyone but Maya

## Governed Entity

**Entity:** Member Profile (kid-type rows: young kid profile, no login -- MVP, and older kid, limited login -- Later) and Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| display_name | text | Kid member's first name or nickname |
| member_type | enum | Organiser, Other Adult Member, young kid profile, older kid limited login |
| sign_in | text | Adults only; not applicable to kid rows |
| age_band | enum | Kid profiles only |
| parental_consent_confirmation | boolean | Kid profiles only, the organiser's consent confirmation |
| notification_preferences | object | Adults only; not applicable to kid rows |
| status | enum | Invited, Active, Left, Removed |
| Dietary Rule (referenced, not a Member Profile field) -- rule_kind, strength, allergen | -- | The specific allergy fact this policy may narrowly admit |
| Subscription.tier | enum (free, paid) | The one Subscription field this policy admits |
| Subscription.billing_period | enum (monthly, yearly) | Suppressed |
| Subscription.billing_state | enum | Suppressed |
| Subscription.billing_history | list | Suppressed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-002 | Support Read-Only Household View | On every render of the Members and Subscription sections |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| display_name (kid rows) | Never rendered to Riley | Always | On render | N/A -- field is omitted, not shown with an error | -- |
| member_type (kid rows) | Never rendered to Riley as an identifying label; the Members section instead groups kid data anonymously under "a household kid member" | Always | On render | N/A -- no field-level error; this is a display-omission rule | -- |
| age_band | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| parental_consent_confirmation | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| notification_preferences (kid rows) | No validation beyond data type -- not applicable to kid rows, and adult rows are outside this policy's scope | Always | -- | -- | -- |
| status (kid rows) | Never rendered to Riley as part of an identifiable kid row | Always | On render | N/A -- field is omitted | -- |
| Dietary Rule (kid member's allergy, rule_kind = allergy) | Rendered only when it is the specific allergy the open safety-concern request's reported meal fails, and only as "a household kid member has an allergy to {allergen}" with no name attached | The open Support Request is kind = safety concern AND this Dietary Rule is the one the reported recipe's ingredient failed | On render | N/A -- field is conditionally omitted, not error-producing | -- |
| Dietary Rule (kid member's non-allergy rules: dislikes, vegetarian settings) | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| Subscription.tier | Always rendered to Riley | Always | On render | N/A -- not a restricted field | -- |
| Subscription.billing_period, billing_state, billing_history | Never rendered to Riley | Always | On render | N/A -- fields are omitted | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Allergy-fact admission is request-scoped | Support Request.kind, Support Request.planned_meal/recipe, Dietary Rule.allergen, Dietary Rule.rule_kind | A kid member's allergy fact is shown only when the open Support Request is a safety concern AND the allergen named matches the specific ingredient that failed the reported meal's safety check -- an allergy unrelated to the reported meal is never shown, even for the same kid, even during the same session | N/A -- this is a display-scoping rule, not a user-facing validation |
| Billing suppression is absolute | Subscription.tier, Subscription.billing_period, Subscription.billing_state, Subscription.billing_history | Only tier passes through; every other Subscription field is suppressed regardless of the open request's kind (safety concern or general support) -- there is no request type that widens billing visibility | N/A -- this is a display-scoping rule |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a kid member's display_name, age_band, parental_consent_confirmation, or status | Riley (Operator, support) | Never | Field is not rendered anywhere in FEAT-22.SPEC-002 -- omitted from the Members section entirely, not shown disabled or blurred |
| View a kid member's allergy fact tied to the open safety-concern request | Riley (Operator, support) | Only while a safety-concern Support Request naming that exact meal/recipe is open, and only as an anonymized fact ("a household kid member has an allergy to {allergen}") | Outside this condition (no safety-concern request open, or the allergy is unrelated to the reported meal): the fact is not rendered |
| View a kid member's non-allergy Dietary Rule (dislike, vegetarian setting) | Riley (Operator, support) | Never | Field is not rendered under any condition |
| View Subscription.tier | Riley (Operator, support) | Always, whenever FEAT-22.SPEC-002 is open | -- |
| View Subscription.billing_period, billing_state, or billing_history | Riley (Operator, support) | Never | Fields are not rendered anywhere in the Subscription section, regardless of the open request's kind |
| View adult Member Profile data (display_name, Dietary Rules, status) | Riley (Operator, support) | Always, whenever FEAT-22.SPEC-002 is open | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| "a household kid member has an allergy to {allergen}" (derived display line) | Derived by matching the open safety-concern Support Request's planned_meal/recipe against the household's kid Dietary Rules for the ingredient that caused the safety check to fail | Computed on every render of FEAT-22.SPEC-002 while a matching safety-concern request is open | No -- fully system-derived; Riley cannot request additional detail |

## Business Rules

- ASMP-26 and ASMP-27: children's-privacy-class protection applies to every kid Member Profile field; Riley may see kid allergy detail only inside a specific safety report, never a kid's broader profile -- this spec is the sole authority for what "inside a specific safety report" admits.
- The Access Matrix's Kid Profile Data column (Riley: None except the allergy details inside a specific safety report) and Billing column (Riley: View, plan tier only) are both implemented entirely by this spec -- no other spec in this feature independently restricts these fields.
- The admitted allergy fact is scoped to the single reported meal's failing ingredient, never the kid's full allergy list, even if the same kid has other allergies unrelated to the report.
- A general-support Support Request never admits any kid profile field, since the narrow allergy exception applies only to safety-concern requests.
- This spec's suppressions apply uniformly regardless of which household is open -- there is no household-specific configuration of what Riley may see.

## Edge Cases

- **The reported meal's safety check failed on more than one kid's allergy (a shared meal with two affected kids)** -- Each affected kid's allergy fact is shown separately, each anonymized as its own "a household kid member has an allergy to {allergen}" line, since the exception is scoped to the meal's failing ingredients, not to a single kid.
- **The household has a kid member whose allergy is unrelated to the reported meal** -- That kid's allergy is never shown, regardless of how the report is worded or how long the session remains open.
- **Riley resolves the safety-concern request while viewing the household** -- The admitted allergy fact remains visible for the remainder of the current session (the request's kind does not change on resolution), but a later session against a new, unrelated request for the same household re-evaluates the admission rule fresh and would not show it unless a new matching safety-concern request is open.
- **The household is on the free tier** -- Subscription.tier still renders as "Free"; the suppression of billing_period, billing_state, and billing_history applies identically regardless of tier.
- **A general support Support Request is open for a household that also has a separate, unrelated open safety-concern request** -- The allergy-fact admission is evaluated against whichever safety-concern request is open for that household, independent of the general-support request; if no safety-concern request is open, no allergy fact is admitted even though a different Support Request exists.
- **The kid's allergy rule is edited by the organiser (allergen changed, or the rule removed) while Riley's session is open** -- The admitted fact reflects the Dietary Rule as currently stored on each render; a rule change is picked up in the same manner as any other field this screen reads, per FEAT-22.SPEC-002's own state model (no live re-fetch mid-session, refreshed on reopen).

## Acceptance Criteria

**FEAT-22.SPEC-007-AC-01:** Given Riley opens a household with a young kid profile, when the Members section renders, then no display_name, age_band, parental_consent_confirmation, or status appears for that kid.

**FEAT-22.SPEC-007-AC-02:** Given a safety-concern request is open for a meal that failed a kid's peanut allergy, when Riley views the Members section, then "a household kid member has an allergy to peanuts" is shown with no name attached.

**FEAT-22.SPEC-007-AC-03:** Given the same kid also has an unrelated dairy allergy not implicated in the reported meal, when Riley views the Members section, then the dairy allergy is not shown.

**FEAT-22.SPEC-007-AC-04:** Given a kid member has a soft dislike recorded, when Riley views the Members section, then the dislike is never shown, regardless of any open request.

**FEAT-22.SPEC-007-AC-05:** Given the open request is a general support contact (not a safety concern), when Riley views the Members section, then no kid allergy fact is shown, since the exception applies only to safety-concern requests.

**FEAT-22.SPEC-007-AC-06:** Given the household is on the paid tier, when Riley views the Subscription section, then "Paid" is shown and no billing_period, billing_state, or billing_history appears.

**FEAT-22.SPEC-007-AC-07:** Given the household is on the free tier, when Riley views the Subscription section, then "Free" is shown with the same suppression of all other billing fields.

**FEAT-22.SPEC-007-AC-08:** Given a shared meal's safety check failed for two different kids' allergies, when Riley views the Members section, then both kids' allergy facts are shown, each anonymized separately.

**FEAT-22.SPEC-007-AC-09:** Given Riley views an adult member's data, when the Members section renders, then the adult's display_name and Dietary Rules are shown in full, since this spec restricts only kid rows.

**FEAT-22.SPEC-007-AC-10:** Given Riley resolves the open safety-concern request mid-session, when Riley continues viewing the same session, then the previously admitted allergy fact remains visible for the remainder of that session.

**FEAT-22.SPEC-007-AC-11:** Given a household has both an open general-support request and a separate open safety-concern request, when Riley views the Members section under either, then the allergy-fact admission is evaluated only against the open safety-concern request, independent of the general-support request.

**FEAT-22.SPEC-007-AC-12:** Given the organiser removes the kid's allergy rule while Riley's session is open, when Riley reopens the household in a later session, then the removed rule no longer appears, since the fact reflects the Dietary Rule as currently stored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



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



# Notification Spec: Support View Recorded Notification

## Overview

**Name:** Support View Recorded Notification
**ID:** FEAT-22.SPEC-009
**Type:** Notification
**Purpose:** Shows Maya an in-app note each time support views her household, so the visible record the Brief promises reaches her at the moment it happens, not only when she goes looking.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when an access session opens, and again when it closes
- Both variants' exact content, tied to the request's kind

**Non-Goals:**
- The persistent, browsable record of every past session -- owned by FEAT-22.SPEC-003 (Household Support Access Record), which this note only announces the arrival of a new entry for
- Any channel beyond in-app -- excluded per this feature's own definition: the dependency map's External Touchpoints table states "Its support-visit note (FEAT-22.SPEC-009) is in-app," and FEAT-22 inventories no Integration spec that could carry an email or push capability for this note
- Notifying Riley -- this notification exists to tell the household support was viewed; Riley is the one doing the viewing and has no reciprocal notification need
- Any household-data content beyond the kind and timing of the visit -- excluded per FEAT-22.SPEC-007's suppression posture: this note never surfaces what Riley actually saw, only that a session occurred and why

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on every session open and every session close | Support access is rare and household-facing trust matters more than urgency; an in-app note that Maya sees on her next visit is proportionate, and the dependency map's External Touchpoints table confirms no other channel is defined for this notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An access session opens | FEAT-22.SPEC-004 (Support Access Session Logging) | Fires every time Riley opens the household view, including a second or later session for the same request | Household reference, Support Request kind, session start time |
| An access session closes | FEAT-22.SPEC-004 (Support Access Session Logging) | Fires every time Riley closes or navigates away from an open session | Household reference, Support Request kind, session end time |

## Audience and Preferences

**Recipients:** Maya (Organiser) only -- the sole role with Support View access to this household's own record (Access Matrix: Maya View, Sam None, both Jordan rows None, Riley is the subject of the note, not its recipient).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this note has no on/off control | -- | Always on | N/A -- excluded per XBR-14: every support visit must be visible to the organiser without exception; a household must never be able to silence the one signal that bounds an otherwise-invisible access capability |

**Quiet Hours:** N/A -- the product defines quiet hours for time-sensitive reminders (e.g., the nightly dinner nudge), not for a trust-and-audit note whose entire purpose is to reach the organiser promptly; this note is delivered in-app only and has no channel a quiet-hours window would meaningfully hold.

## Content Definition

**In-app (session opened):**
- **Title:** Support viewed your household
- **Body:** {support_visit_reason} on {session_start_date_time}
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**In-app (session closed):**
- **Title:** Support access ended
- **Body:** {support_visit_reason} -- access ended on {session_end_date_time}
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {support_visit_reason} | Derived -- Support Request.kind, rendered as "Reviewing a reported safety concern" (kind = safety concern) or "Reviewing a support request" (kind = general support) | Reviewing a reported safety concern | Never empty -- every Support Request has a kind at creation |
| {session_start_date_time} | Support Request.access_record -- the session entry's start timestamp | Sep 27, 2026, 2:14 PM | Never empty -- set by FEAT-22.SPEC-004 at the moment the session opens, before this notification fires |
| {session_end_date_time} | Support Request.access_record -- the session entry's end timestamp | Sep 27, 2026, 2:31 PM | Never empty -- set by FEAT-22.SPEC-004 at the moment the session closes, before this notification fires |

## Delivery Rules

**Batching:** None -- each open and each close is delivered as its own note; sessions are rare enough (per the feature's own Behavioral Context: "rare, on-demand use... never a routine or scheduled interaction") that batching would only delay the visibility this notification exists to provide.
**Deduplication:** Exactly one open note per access-session entry and exactly one close note per access-session entry, since each is fired once by FEAT-22.SPEC-004 at the corresponding step of its processing logic, which itself runs each open and close exactly once per session.
**Retry on failure:** In-app delivery has no retry mechanism of its own: the note is delivered when Maya next opens the product, per standard in-app notification behavior; there is no separate delivery attempt to fail or retry.
**Expiry:** The note itself does not expire in the sense of becoming undeliverable -- it is a record of something that already happened, so it remains available to view until Maya dismisses it; the underlying access-session entry it announces survives indefinitely in FEAT-22.SPEC-003, per SC-18's retention posture.

## Edge Cases

- **The household is deleted between the session and delivery of this note** -- No note is delivered; a deleted household has no organiser left to notify, and FEAT-18's deletion cascade removes the household's data before any pending in-app note could render.
- **Maya has left the organiser role (organiser hand-over, FEAT-09) between the session and delivery** -- The note is delivered to whoever holds the organiser role at the moment of delivery, consistent with Support View being an organiser-role entitlement rather than tied to a specific person.
- **Two sessions open and close in rapid succession for the same household** -- Each open and each close produces its own note in order; they are never merged, since batching is explicitly excluded for this notification.
- **Maya is actively viewing FEAT-22.SPEC-003 when a new session opens** -- The new entry does not auto-insert into that screen mid-view (per FEAT-22.SPEC-003's own edge cases); the in-app note still delivers and is visible the next time Maya checks her notifications, independent of what screen she happens to be on.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-004 (Support Access Session Logging) | Triggered by (inbound) | Both open and close steps fire this notification |
| FEAT-22.SPEC-003 (Household Support Access Record) | Navigation (outbound) | The CTA on both variants deep-links here |

## Analytics and Success Signals

- **support_view_note_delivered** (event: session_opened / session_closed) -- N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing note's actual delivery is observable
- **support_view_note_cta_tapped** (event: session_opened / session_closed) -- N/A -- no success-metrics.md metric traces to this feature; retained to observe whether organisers actually follow through to the full record

## Acceptance Criteria

**FEAT-22.SPEC-009-AC-01:** Given Riley opens a household's view against a safety-concern request, when the session opens, then Maya receives an in-app note titled "Support viewed your household" with the body "Reviewing a reported safety concern on {session_start_date_time}".

**FEAT-22.SPEC-009-AC-02:** Given Riley opens a household's view against a general-support request, when the session opens, then Maya receives the same note with the body "Reviewing a support request on {session_start_date_time}".

**FEAT-22.SPEC-009-AC-03:** Given an open session closes, when FEAT-22.SPEC-004 records its end, then Maya receives an in-app note titled "Support access ended" with the session's end time.

**FEAT-22.SPEC-009-AC-04:** Given Maya taps the CTA on either note, then she is taken to FEAT-22.SPEC-003 (Household Support Access Record).

**FEAT-22.SPEC-009-AC-05:** Given this notification exists, when Maya looks for a way to turn it off, then no preference control exists anywhere in the product.

**FEAT-22.SPEC-009-AC-06:** Given a session opens at 2:00 AM local time, when the notification fires, then it is delivered in-app immediately, since no quiet-hours window applies to this notification.

**FEAT-22.SPEC-009-AC-07:** Given the household is deleted between the session and delivery, then no note is delivered.

**FEAT-22.SPEC-009-AC-08:** Given the organiser role has been handed over between the session and delivery, when the note delivers, then it reaches whoever currently holds the organiser role.

**FEAT-22.SPEC-009-AC-09:** Given two sessions open and close in rapid succession for the same household, when both complete, then four separate notes are delivered (two opens, two closes), never batched together.

**FEAT-22.SPEC-009-AC-10:** Given Sam or either Jordan role is signed in to the same household, when a session opens or closes, then neither receives this notification, since only Maya holds Support View access to the record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (open, close) | 2 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Support Request Resolved Notification

## Overview

**Name:** Support Request Resolved Notification
**ID:** FEAT-22.SPEC-010
**Type:** Notification
**Purpose:** Tells Maya when her general-support Support Request is resolved, so a submitted request never goes silent.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when Riley resolves a general-support Support Request

**Non-Goals:**
- Safety-concern resolutions -- excluded per FEAT-22.SPEC-005's own scope decision: FEAT-02.SPEC-014 (Safety Concern Resolution Notice) already tells the household the safety outcome once FEAT-22.SPEC-005 forwards it to FEAT-02.SPEC-005; a second, separate note here for the same event would duplicate and risk contradicting that outcome-specific message, so this notification fires only for general-support kind requests
- The acknowledgement sent when a general-support request is first submitted -- owned by FEAT-18.SPEC-015 (Support Request Acknowledgement), a distinct earlier communication this notification does not repeat
- Any household-data content about what was discussed or resolved -- excluded per scope-boundaries.md SC-14: coordination happens through named features, not a messaging layer, so this note confirms only that the request is resolved, never a description of the resolution's substance
- Delivery to Riley -- Riley is the one resolving the request, not a recipient of its resolution notice

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on resolution | This feature inventories no Integration spec and the dependency map's External Touchpoints table assigns FEAT-22 no email or push capability; a general support contact is itself a low-frequency, non-urgent interaction, and an in-app confirmation matches the product's existing pattern of confirming Support Request outcomes in-product (FEAT-18.SPEC-005's own same-screen confirmation on submission) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A general-support Support Request is resolved | FEAT-22.SPEC-005 (Support Request Resolution) | Fires only when the resolved request's kind is general support contact | Household reference, the resolved Support Request's note (the household's original description) |

## Audience and Preferences

**Recipients:** Maya (Organiser) -- the raised_by member on a general-support Support Request is the adult who submitted it (Maya or Sam, per FEAT-18.SPEC-005's Access), but this notification's recipient is always the organiser, consistent with FEAT-22.SPEC-009's audience and with Maya's Support View access being the household's own visibility into support outcomes; Sam's own submission is covered by his same-screen confirmation from FEAT-18.SPEC-005 and this in-app note additionally reaches Maya as the accountable organiser.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this note has no on/off control | -- | Always on | N/A -- excluded per the same reasoning as FEAT-22.SPEC-009: a submitted request's outcome must always reach the organiser, since there is no other product surface that tells her a general support request was resolved |

**Quiet Hours:** N/A -- the product's quiet-hours behavior governs time-sensitive reminders, not a one-time resolution confirmation with no urgency to hold back; this note is in-app only, so no channel exists for quiet hours to meaningfully affect.

## Content Definition

**In-app:**
- **Title:** Your support request is resolved
- **Body:** We've resolved your support request: "{request_note_summary}"
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {request_note_summary} | Support Request.note, truncated to the first 100 characters if longer | "The grocery list stopped updating after a swap" | The body renders as "We've resolved your support request." with no quoted text, since FEAT-18.SPEC-010 permits an empty note only when validation allows it; in practice a general-support note is required at submission (FEAT-18.SPEC-005), so this fallback covers the case where the note has since been cleared by a data-management action rather than an ordinary submission gap |

## Delivery Rules

**Batching:** None -- each resolution is delivered as its own note; general-support requests are infrequent enough that batching would only delay a resolution the household is specifically waiting to hear about.
**Deduplication:** Exactly one note per Support Request, since FEAT-22.SPEC-008 (Support Request Status Transition Rules) allows a request to reach Resolved exactly once, and FEAT-22.SPEC-005 triggers this notification exactly once at that transition.
**Retry on failure:** In-app delivery has no retry of its own: the note is delivered when Maya next opens the product, per standard in-app notification behavior.
**Expiry:** The note does not expire -- it remains available to view until Maya dismisses it, and the resolution itself remains visible afterward through the request's Resolved status wherever it is shown (FEAT-01.SPEC-010, FEAT-22.SPEC-003).

## Edge Cases

- **The household is deleted between resolution and delivery** -- No note is delivered; a deleted household has no organiser left to notify.
- **The organiser role changes hands between resolution and delivery** -- The note is delivered to whoever currently holds the organiser role, consistent with this being an organiser-role entitlement rather than tied to a specific person.
- **A safety-concern request is resolved** -- This notification never fires; FEAT-02.SPEC-014 is the sole household-facing resolution message for that kind, per this spec's own Non-Goals.
- **The request's note was longer than 100 characters** -- The body shows the first 100 characters with no truncation indicator beyond the quoted text itself reading as a summary, consistent with this being a confirmation rather than a full replay of the original submission.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-005 (Support Request Resolution) | Triggered by (inbound) | Resolving a general-support request fires this notification |
| FEAT-22.SPEC-003 (Household Support Access Record) | Navigation (outbound) | The CTA deep-links here |
| FEAT-18.SPEC-005 (Contact Support) | References (inbound, cross-feature) | The originating screen for the general-support request this notification resolves |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | References (inbound, cross-feature) | The earlier, submission-time communication this notification does not repeat |

## Analytics and Success Signals

- **support_request_resolved_note_delivered** -- N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing note's actual delivery is observable
- **support_request_resolved_note_cta_tapped** -- N/A -- no success-metrics.md metric traces to this feature; retained to observe whether organisers follow through to the full record

## Acceptance Criteria

**FEAT-22.SPEC-010-AC-01:** Given Riley resolves a general-support Support Request whose note reads "The grocery list stopped updating after a swap", when resolution completes, then Maya receives the in-app note "We've resolved your support request: \"The grocery list stopped updating after a swap\"".

**FEAT-22.SPEC-010-AC-02:** Given Maya taps the CTA on this note, then she is taken to FEAT-22.SPEC-003 (Household Support Access Record).

**FEAT-22.SPEC-010-AC-03:** Given Riley resolves a safety-concern request instead, when resolution completes, then this notification does not fire.

**FEAT-22.SPEC-010-AC-04:** Given Sam submitted the original general-support request, when it is resolved, then Maya (the organiser) still receives this note, in addition to whatever same-screen confirmation Sam saw at submission.

**FEAT-22.SPEC-010-AC-05:** Given this notification exists, when Maya looks for a way to turn it off, then no preference control exists anywhere in the product.

**FEAT-22.SPEC-010-AC-06:** Given a request is resolved at 3:00 AM local time, when the notification fires, then it is delivered in-app immediately, since no quiet-hours window applies.

**FEAT-22.SPEC-010-AC-07:** Given the household is deleted between resolution and delivery, then no note is delivered.

**FEAT-22.SPEC-010-AC-08:** Given the request's note is longer than 100 characters, when the note renders, then only the first 100 characters are quoted in the body.

**FEAT-22.SPEC-010-AC-09:** Given the organiser role changes hands between resolution and delivery, when the note delivers, then it reaches whoever currently holds the organiser role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

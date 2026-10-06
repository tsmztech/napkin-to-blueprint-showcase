---
document_type: feature-overview
feature_number: FEAT-22
feature_name: Operator Read-Only Support Access
feature_slug: operator-read-only-support-access
priority_tier: Nice-to-Have
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 10
screen_count: 3
automation_count: 2
logic_rule_count: 3
integration_count: 0
notification_count: 2
---

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

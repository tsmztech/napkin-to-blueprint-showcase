---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-19.SPEC-004
spec_name: History Access & Reuse Authorization
spec_slug: history-access-reuse-authorization
parent_feature: FEAT-19
parent_feature_name: Weekly Plan History
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 13
---

# Logic/Rule Spec: History Access & Reuse Authorization

## Overview

**Name:** History Access & Reuse Authorization
**ID:** FEAT-19.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines which roles may view a household's archived Weekly Plan history and which single role may trigger reusing a past week, enforced consistently across both history screens and the reuse automation.
**Parent Feature:** FEAT-19 -- Weekly Plan History
**Governed Entity:** Weekly Plan (archived)

## Scope and Non-Goals

**In Scope:**
- Who may view the Weekly Plan History Browse list (FEAT-19.SPEC-001) and Past Week Detail View (FEAT-19.SPEC-002) for the household
- Who may see the "Reuse this week" control at all, and who may actually trigger the Past Plan Reuse automation (FEAT-19.SPEC-003)
- What every role and unauthenticated/expired state experiences when denied

**Non-Goals:**
- Authorization for editing the Weekly Plan a reuse copy lands in -- governed by Manual Weekly Planning's own authorization rules (FEAT-23), since once the copy is handed off it is an ordinary future week subject to FEAT-23's Access field (Maya Full, Sam Own-only)
- Field-level validation of Weekly Plan or Grocery List data -- this feature's Connected Entities are read-only (feature-overview.md, Entity-Lifecycle Coverage Matrix: "no full CRUD matrix applies; all entities this feature touches are listed... as Referenced Entities"); no field-level validation rules exist for this feature to govern, since it neither creates nor edits these entities
- The allergy and religious-rule safety re-check performed during reuse -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02) and referenced by FEAT-19.SPEC-003 as XBR-01; this spec governs only who may trigger reuse, not the safety logic reuse depends on

## Governed Entity

**Entity:** Weekly Plan (archived)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date range | The calendar week the archived plan covered -- no validation beyond data type; this spec governs read/reuse access, not the field's format |
| origin | enum | AI-generated or manually built -- no validation beyond data type |
| status | enum | Generated/Started, Reviewed, Approved, Active, Archived; this spec's rules apply only once status is Archived (feature-dependency-map.md, Weekly Plan lifecycle) |
| approval | text/enum | Organiser approval record or auto-adoption note -- no validation beyond data type; displayed read-only |
| estimated_total | number | The archived week's estimated cost -- no validation beyond data type; displayed read-only |
| over_budget_note | text | Shown when the archived week exceeded budget -- no validation beyond data type; displayed read-only |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-19.SPEC-001 | Weekly Plan History Browse | On screen entry -- list visibility gated per role before any archived week data is shown |
| FEAT-19.SPEC-002 | Past Week Detail View | On screen entry -- detail visibility gated per role; "Reuse this week" control visibility gated per role |
| FEAT-19.SPEC-003 | Past Plan Reuse | On trigger -- the automation only fires when the acting role is authorized to reuse |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type | Always | -- | -- | -- |
| origin | No validation beyond data type | Always | -- | -- | -- |
| status | No validation beyond data type | Always | -- | -- | -- |
| approval | No validation beyond data type | Always | -- | -- | -- |
| estimated_total | No validation beyond data type | Always | -- | -- | -- |
| over_budget_note | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs read and reuse-trigger access, not field-level cross-validation; the archived Weekly Plan's fields carry no conditional-requirement relationships this spec must arbitrate.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View history list (FEAT-19.SPEC-001) | Maya (Organiser) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Sam (Other Adult Member) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Jordan (older kid, limited login -- Later) | Always | -- |
| View history list (FEAT-19.SPEC-001) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no reachable screen to deny access to |
| View history list (FEAT-19.SPEC-001) | Riley (Operator, support) | Always, per Weekly Plan View access (user-persona.md Access Matrix) | -- |
| View history list (FEAT-19.SPEC-001) | Unauthenticated / expired session | Never | Redirected to the sign-in screen; no history content shown before or during redirect |
| View past week detail (FEAT-19.SPEC-002) | Maya (Organiser) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Sam (Other Adult Member) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Jordan (older kid, limited login -- Later) | Always | -- |
| View past week detail (FEAT-19.SPEC-002) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no reachable screen to deny access to |
| View past week detail (FEAT-19.SPEC-002) | Riley (Operator, support) | Always, per Weekly Plan View access (user-persona.md Access Matrix) | -- |
| View past week detail (FEAT-19.SPEC-002) | Unauthenticated / expired session | Never | Redirected to the sign-in screen; no detail content shown before or during redirect |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Maya (Organiser) | Always | -- |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Sam (Other Adult Member) | Never | Control is not rendered on the detail screen for this role -- no button appears |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Jordan (older kid, limited login -- Later) | Never | Control is not rendered on the detail screen for this role -- no button appears |
| See "Reuse this week" control (FEAT-19.SPEC-002) | Riley (Operator, support) | Never | Control is not rendered on the detail screen for this role, consistent with Riley never changing household data (feature-dependency-map.md, XBR-14) |
| Trigger reuse copy (FEAT-19.SPEC-003) | Maya (Organiser) | Always -- the target future week must fall within the one-week-ahead planning horizon (product-features.md, Validation & Limits) | -- |
| Trigger reuse copy (FEAT-19.SPEC-003) | Sam, Jordan (either form), Riley | Never | No launching control is available to attempt the action from; a direct attempt to invoke the automation without the control (e.g., a stale or manipulated request) is refused with "Only the household organiser can reuse a past week" |

## Defaults and Derivations

N/A -- this spec's governed entity is read-only in this feature (Entity-Lifecycle Coverage Matrix, feature-overview.md); it defines no defaulted or derived field values of its own. The Past Plan Reuse automation's own created-week defaults (e.g., origin recorded as manually built, per the existing two-value enum) are specified in FEAT-19.SPEC-003, not here.

## Business Rules

- Access level for viewing history follows the Weekly Plan column of the Access Matrix in user-persona.md exactly: Full for Maya (including reuse), View for Sam and the older-kid login, View for Riley from v1 (feature-overview.md, Access field).
- Reuse is gated to Maya alone regardless of a household's tier or the past week's origin (AI-generated or manually built) -- the same single-role gate applies to every archived week.
- XBR-01 (feature-dependency-map.md): a reused week is still subject to the app-enforced safety check before it is shown as pre-filled; this spec's authorization gate on who may trigger reuse does not substitute for, or weaken, that check.
- An unauthorized visitor (not signed in as a household member) never reaches any Weekly Plan History screen; per user-persona.md's Access Matrix notes, unauthorized visitors see only a sign-in screen, an invitation-acceptance screen, or a household-referral welcome page -- never household data of any kind.
- This spec is the single authoritative source for history view and reuse-trigger access; FEAT-19.SPEC-001, FEAT-19.SPEC-002, and FEAT-19.SPEC-003 each reference it rather than re-deriving the rule (feature-overview.md, Shared Validation).

## Edge Cases

- **Maya's organiser role is handed over to Sam mid-session while Maya has the Past Week Detail View open with "Reuse this week" visible** -- The hand-over (FEAT-09) takes effect immediately; on Maya's next interaction with the control, her session is re-evaluated against the current organiser and the control is hidden for her (she is no longer Maya-the-organiser once the hand-over completes), while Sam's session gains the control on his next screen load.
- **Riley's support access opens against a household with no open Support Request** -- Per XBR-14, Riley's support access opens only for a household with an open Support Request; outside that window, Riley has no access to any Weekly Plan History screen for that household at all, not merely a read-only view.
- **The older-kid limited login (Later) is removed or its login access ends between viewing the list and opening a detail** -- The detail screen's own access check re-evaluates on load; if the login no longer exists, the request is treated as unauthenticated and redirected to sign-in.
- **A direct API-level request attempts to trigger Past Plan Reuse for a household where the requester is Sam** -- Refused with "Only the household organiser can reuse a past week," regardless of whether the request came through the rendered control (which Sam never sees) or an out-of-band request.
- **Two devices are signed in as Maya at once, one of which is mid-reuse-trigger when the household's organiser role is handed over on the other device** -- The reuse trigger already in flight completes under the authorization state captured when it started (per FEAT-19.SPEC-003's own concurrency handling); any new reuse attempt from either device is evaluated against the current organiser at the moment of that new attempt.

## Acceptance Criteria

**FEAT-19.SPEC-004-AC-01:** Given Maya opens Weekly Plan History, when the list loads, then she sees the full archived history.

**FEAT-19.SPEC-004-AC-02:** Given Sam opens Weekly Plan History, when the list loads, then he sees the full archived history, the same as Maya.

**FEAT-19.SPEC-004-AC-03:** Given Jordan (older kid, limited login) opens Weekly Plan History, when the list loads, then he sees the full archived history, the same as Maya.

**FEAT-19.SPEC-004-AC-04:** Given an unauthenticated visitor attempts to reach Weekly Plan History, when the request is made, then they are redirected to the sign-in screen with no history content shown.

**FEAT-19.SPEC-004-AC-05:** Given Riley (Operator, support) has an open Support Request for a household, when she opens that household's Weekly Plan History, then she sees the full archived history read-only.

**FEAT-19.SPEC-004-AC-06:** Given Maya opens a past week's detail, when the screen renders, then she sees the "Reuse this week" control in the header.

**FEAT-19.SPEC-004-AC-07:** Given Sam opens the same past week's detail, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-08:** Given Jordan (older kid, limited login) opens the same past week's detail, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-09:** Given Riley opens the same past week's detail during an open support request, when the screen renders, then no "Reuse this week" control appears anywhere on the screen.

**FEAT-19.SPEC-004-AC-10:** Given Maya taps "Reuse this week" and picks a valid target future week, when the trigger fires, then FEAT-19.SPEC-003 (Past Plan Reuse) begins processing.

**FEAT-19.SPEC-004-AC-11:** Given a direct request attempts to trigger the reuse automation as Sam, when the request is evaluated, then it is refused with "Only the household organiser can reuse a past week."

**FEAT-19.SPEC-004-AC-12:** Given Maya hands over the organiser role to Sam while she has the Past Week Detail View open, when she next interacts with the screen, then the "Reuse this week" control is no longer shown to her.

**FEAT-19.SPEC-004-AC-13:** Given Jordan as a young kid profile has no login, when any attempt is made to reach a Weekly Plan History screen under that profile, then no such attempt is possible -- there is no reachable screen for it to load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 17 | 17 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

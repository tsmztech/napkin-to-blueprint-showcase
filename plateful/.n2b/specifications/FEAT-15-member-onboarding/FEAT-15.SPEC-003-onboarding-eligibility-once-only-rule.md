---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-003
spec_name: Onboarding Eligibility & Once-Only Rule
spec_slug: onboarding-eligibility-once-only-rule
parent_feature: FEAT-15
parent_feature_name: Member Onboarding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 25
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Onboarding Eligibility & Once-Only Rule

## Overview

**Name:** Onboarding Eligibility & Once-Only Rule
**ID:** FEAT-15.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs which role is ever routed through Member Onboarding, and enforces that onboarding fires exactly once per accepted invitation -- including the case where a re-invited former member onboards fresh rather than being restored to prior data.
**Parent Feature:** FEAT-15 -- Member Onboarding
**Governed Entity:** Member Profile (the onboarding-relevant fields consulted for routing), read alongside the paired Invitation's status.

## Scope and Non-Goals

**In Scope:**
- The eligibility rule: only a Member Profile with member_type Other Adult Member is ever routed through onboarding
- The once-only rule: onboarding is shown exactly once per accepted Invitation's transition
- The re-join rule: a previously Removed or Left member who is re-invited and accepts again is eligible and onboards fresh, with no data restoration
- Authorization Rules for reaching (being routed to) and rendering (viewing) the Onboarding Landing, across every role in the Access Matrix

**Non-Goals:**
- Creating or updating the Member Profile or Invitation records themselves -- owned by FEAT-01 and FEAT-09; this spec only reads their status and member_type to decide routing, per the feature's declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix)
- Restoring a re-joined member's dietary rules, ratings, or prior household history -- excluded per XBR-18 and the feature's Non-Goals: a re-join always onboards fresh rather than being silently restored to old data
- Field-level validation of Member Profile or Invitation data unrelated to onboarding routing (e.g., display_name length, sign_in format, contact_detail format) -- owned by FEAT-01.SPEC-014 and FEAT-09.SPEC-010 respectively; this spec addresses only the fields it actually reads for routing decisions

## Governed Entity

**Entity:** Member Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| display_name | text | First name or nickname; read only to personalize the role-explanation banner on FEAT-15.SPEC-001, not a routing input |
| member_type | enum | Organiser, Other Adult Member, young kid profile (no login), or older kid limited login (Later); the sole role-eligibility gate for onboarding |
| sign_in | text | Email and protected sign-in for adults only; outside this spec's read scope |
| age_band | enum | Kid profiles only; never populated for an Other Adult Member and never read by onboarding |
| parental_consent_confirmation | boolean | Kid profiles only; never populated for an Other Adult Member and never read by onboarding |
| notification_preferences | derived | Per-member plan-ready/nudge toggles; outside this spec's read scope |
| status | enum | Invited, Active, Left, or Removed; paired with the Invitation's Accepted transition to determine both eligibility-to-fire-now and re-join status |

A second entity is consulted alongside Member Profile but is not independently governed here: **Invitation.status** (Sent, Accepted, Revoked, Expired), whose own field validation, race resolution, and lifecycle are owned by FEAT-09.SPEC-010. This spec reads only whether the specific Invitation this evaluation concerns has just transitioned to Accepted -- it does not restate Invitation's own rules.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | On the invitation-acceptance trigger, before any routing decision is made |
| FEAT-15.SPEC-001 | Onboarding Landing | On screen entry -- re-confirms authorization at render time, catching any change between routing and render |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| member_type | Must equal Other Adult Member for this Member Profile to ever be eligible for onboarding | Always | On automation trigger (FEAT-15.SPEC-002) and on screen entry (FEAT-15.SPEC-001) | N/A -- this is a routing rule, not a form field; an ineligible member_type produces no error message, only a silent redirect away from onboarding (see Authorization Rules) | Yes (blocking for onboarding routing) |
| status | Must be Active, and that Active status must be the direct result of the specific Invitation transition currently being evaluated (not a stale Active status from long-standing membership), for onboarding to fire now | Always | On automation trigger and on screen entry | N/A -- routing rule, not a form field | Yes |
| display_name | No validation beyond data type -- already validated at profile creation by FEAT-01.SPEC-014/FEAT-09.SPEC-007; onboarding only reads it to personalize the role-explanation banner | Always | -- | -- | No |
| sign_in | No validation beyond data type -- outside this spec's read scope | Always | -- | -- | No |
| age_band | No validation beyond data type -- never populated for an Other Adult Member; onboarding never reads it | Always | -- | -- | No |
| parental_consent_confirmation | No validation beyond data type -- never populated for an Other Adult Member; onboarding never reads it | Always | -- | -- | No |
| notification_preferences | No validation beyond data type -- outside this spec's read scope | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Onboarding eligibility | member_type, status | member_type must equal Other Adult Member AND status must be Active as a direct result of the specific Invitation transition this onboarding run is evaluating | N/A -- no user-facing error message; an ineligible combination routes silently to the ordinary Weekly Plan View (see Authorization Rules) |
| Once-only gate | status, Invitation.status (paired) | Onboarding may fire only on the transition where the paired Invitation's status becomes Accepted for the first time; once FEAT-15.SPEC-001 has rendered for that transition, no later evaluation of the same transition may fire onboarding again | N/A -- silent routing away, not a blocking error |
| Re-join freshness | status (prior value: Removed or Left; current value: Active via a new Invitation) | A Member Profile whose status was Removed or Left immediately before this acceptance, and is now Active via a newly accepted Invitation, is eligible exactly as a first-time member -- no lookup against the prior profile's dietary rules, ratings, or history is performed or permitted | N/A -- data-isolation rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Be routed to Onboarding Landing (fire onboarding) | Sam (Other Adult Member) | Only on the specific Invitation-acceptance transition FEAT-15.SPEC-002 is currently evaluating, and only once per that transition | -- |
| Be routed to Onboarding Landing (fire onboarding) | Maya (Organiser) | Never -- the organiser creates the household and is never shown this flow | Structurally unreachable (organiser accounts are never produced by an invitation acceptance); if ever evaluated, routing goes to the ordinary Weekly Plan View instead, with no error message |
| Be routed to Onboarding Landing (fire onboarding) | Jordan (young kid profile, no login -- MVP) | Never -- young kid profiles never accept invitations | Structurally unreachable; no login exists through which this role could ever receive this routing |
| Be routed to Onboarding Landing (fire onboarding) | Jordan (older kid, limited login -- Later) | Never -- gets its own short introduction under FEAT-17, not this flow | Routing to FEAT-15.SPEC-001 is never evaluated for this role; FEAT-17 owns its own first-use path |
| Be routed to Onboarding Landing (fire onboarding) | Riley (Operator, support) | Never -- Riley's Household Invitations access is None | Structurally unreachable; Riley has no acceptance event that could trigger this rule |
| Render Onboarding Landing (direct or repeat request) | Sam (Other Adult Member) | Only once per accepted Invitation's transition, and only immediately following FEAT-15.SPEC-002's routing for that transition | Any later or direct request (reload, re-entry, deep link) renders the ordinary Weekly Plan View instead -- never a second onboarding render, and never an error message |
| Render Onboarding Landing (direct or repeat request) | Maya (Organiser) | Never | Any request renders the ordinary Weekly Plan View instead; Onboarding Landing itself never renders |
| Render Onboarding Landing (direct or repeat request) | Jordan (young kid profile, no login -- MVP) | Never | No request is possible without a login; Onboarding Landing never renders for this role |
| Render Onboarding Landing (direct or repeat request) | Jordan (older kid, limited login -- Later) | Never | Any request renders that role's own standard landing instead; Onboarding Landing itself never renders |
| Render Onboarding Landing (direct or repeat request) | Riley (Operator, support) | Never | Any request shows no household data; Onboarding Landing itself never renders |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| "onboarding-shown" evaluation | Derived at each check from whether FEAT-15.SPEC-001 has already rendered for the current Invitation's Accepted transition -- read from the Invitation's own one-time Accepted state, not a separate stored flag (feature-overview.md, Entity-Lifecycle Coverage Matrix) | On every routing decision (FEAT-15.SPEC-002) and every render-time re-confirmation (FEAT-15.SPEC-001) | No -- system-derived gate, not user-overridable |
| "is-re-join" flag | Derived by checking whether the Member Profile's status immediately before this Accepted transition was Removed or Left | On every routing decision | No |

## Business Rules

- XBR-18 (owned jointly with FEAT-09): an accepted invitation creates a Member Profile with Other Adult Member access and triggers first-use onboarding exactly once per accepted invitation; a re-invited former member onboards again rather than being restored to old data.
- The once-only gate rides on the Invitation's own one-time Accepted transition (FEAT-09), not on a new persisted flag on Member Profile -- this keeps the feature within its declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- Eligibility, once-only status, and re-join freshness are the single source of truth for FEAT-15.SPEC-001 and FEAT-15.SPEC-002 -- neither spec restates this logic independently (Feature Breakdown Brief, Shared Validation).

## Edge Cases

- **Member Profile's member_type is somehow not Other Adult Member at evaluation time** (e.g., a hypothetical future role change mid-flow) -- Routing is denied for that transition; the member sees the ordinary Weekly Plan View, never an error message, since this is a routing gate, not a form validation.
- **Invitation's Accepted transition is evaluated twice in rapid succession** (e.g., a duplicate event delivery) -- The once-only gate treats the second evaluation as already-shown, since FEAT-15.SPEC-001 has already rendered (or is already rendering) for that same transition; no second onboarding fires.
- **A member is Removed and re-invited multiple times across the product's lifetime** -- Each new accepted Invitation is evaluated independently; every one is a fresh eligible transition, and each onboards exactly once, with no memory carried between onboarding runs beyond the current transition's own eligibility check.
- **Member Profile status reads Active but the specific Invitation this evaluation is tied to is not the one that produced the current Active status** (e.g., a long-since-onboarded member) -- Ineligible for firing again; only the Invitation transition currently being evaluated can ever satisfy the once-only gate, never a historical Active status alone.
- **Authorization boundary crossed mid-flow** -- a member's status changes (e.g., removed) between FEAT-15.SPEC-002's routing decision and FEAT-15.SPEC-001's render -- Render-time re-confirmation (Enforced By) catches this: if status is no longer Active, the screen does not render and the member instead sees whatever a removed member sees elsewhere in the product (owned by FEAT-09/FEAT-18, not restated here).

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Sam's Member Profile has member_type Other Adult Member and status Active from the Invitation acceptance currently being evaluated, when eligibility is checked, then Sam is confirmed eligible and onboarding is permitted to fire.

**FEAT-15.SPEC-003-AC-02:** Given a hypothetical evaluation where the accepted role's member_type is not Other Adult Member, when eligibility is checked, then onboarding is denied and the member is routed to the ordinary Weekly Plan View instead, with no error message shown.

**FEAT-15.SPEC-003-AC-03:** Given Sam is a previously Removed member whose new Invitation has just been accepted, when re-join status is checked, then he is flagged as a re-join and still confirmed eligible, with no lookup performed against his prior profile's data.

**FEAT-15.SPEC-003-AC-04:** Given onboarding has already rendered once for Sam's current accepted Invitation, when the once-only gate is evaluated again (e.g., a duplicate event), then it reports already-shown and onboarding does not fire a second time.

**FEAT-15.SPEC-003-AC-05:** Given Sam has already viewed Onboarding Landing once, when he reloads or re-enters the app later, then render-time re-confirmation denies a second render and he sees the ordinary Weekly Plan View instead.

**FEAT-15.SPEC-003-AC-06:** Given Maya (Organiser) is evaluated against "Be routed to Onboarding Landing," when the check runs, then the action is always denied and she is never routed to this flow.

**FEAT-15.SPEC-003-AC-07:** Given Jordan (young kid profile, no login) is evaluated against "Be routed to Onboarding Landing," when the check runs, then the action is always denied, since young kid profiles never accept invitations and hold no login to be routed through.

**FEAT-15.SPEC-003-AC-08:** Given Jordan (older kid, limited login) is evaluated against "Render Onboarding Landing," when the check runs, then the action is always denied and no path to this screen is ever produced for that role.

**FEAT-15.SPEC-003-AC-09:** Given Riley (Operator, support) attempts any request that would render Onboarding Landing, when the check runs, then the action is denied and Riley sees no household data through this path.

**FEAT-15.SPEC-003-AC-10:** Given an unauthenticated visitor requests the Onboarding Landing screen directly, when the request is evaluated, then rendering is denied and the visitor is redirected to sign-in.

**FEAT-15.SPEC-003-AC-11:** Given a member's status changes to removed between FEAT-15.SPEC-002's routing decision and FEAT-15.SPEC-001's render, when the screen re-confirms authorization at render time, then rendering is denied and the member does not see any plan or list data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |

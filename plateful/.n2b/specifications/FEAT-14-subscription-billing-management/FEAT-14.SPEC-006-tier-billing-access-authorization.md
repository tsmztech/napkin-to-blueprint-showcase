---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-006
spec_name: Tier & Billing Access Authorization
spec_slug: tier-billing-access-authorization
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Tier & Billing Access Authorization

## Overview

**Name:** Tier & Billing Access Authorization
**ID:** FEAT-14.SPEC-006
**Type:** Logic/Rule
**Purpose:** Enforces who can view tier status, who can change billing, and what an unauthorized visitor sees, per the Access Matrix.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- The base per-role authorization for viewing plan tier
- The base per-role authorization for viewing billing history and payment details
- The base per-role authorization for acting on billing (upgrade, manage billing, downgrade/cancel) at all, independent of billing state
- The unauthenticated and expired-session experience for every FEAT-14 screen
- Riley's Support-Request-gated, tier-only view

**Non-Goals:**
- Billing-state-conditioned and timing-conditioned rules layered on top of Maya's own access (e.g., that a period switch is disabled while billing_state is not Active) -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules), which references this spec for the base role check
- Executing any Subscription write -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec only governs who may initiate a request that could lead to one
- Opening or closing Riley's Support Request itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only governs what Riley may see of Subscription data while a request is open, per XBR-14

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The household's current plan tier |
| billing_period | enum (none, monthly, yearly) | The household's billing cadence |
| billing_state | enum (Active, Payment failed, Cancelled, Reverted to free) | The household's current billing status |
| billing_history | list | Chronological record of charges, outcomes, and amounts |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-001 | Plan Tier Overview | On screen entry -- who sees the screen at all, and which sections and actions render |
| FEAT-14.SPEC-002 | Upgrade to Paid | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-003 | Billing & Payment Management | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-004 | Downgrade / Cancel | On screen entry -- reachable only to Maya |
| FEAT-14.SPEC-005 | Billing State & Refund Rules | References this spec for the base role check before layering its own billing-state conditions |
| FEAT-22 | Operator Read-Only Support Access | Governs when Riley's Support-Request-gated access opens or closes; this spec governs what that access may see of Subscription data while open |

## Field Validation Rules

{This spec governs access, not field content; every Subscription field's content rules are defined in FEAT-14.SPEC-005 (Billing State & Refund Rules). No validation beyond data type applies here.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_period | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_state | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |
| billing_history | No validation beyond data type -- content rules owned by FEAT-14.SPEC-005 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Payment-detail fields never surface outside Maya's session | billing_history | Payment method summaries and billing_history entries are rendered only when the requesting role is Maya; every other role's request for this data is refused before any field value is read | "You don't have access to billing for this household." |
| Tier visibility is broader than billing-detail visibility | tier, billing_history | tier alone may be shown to Maya, Sam, and (while an open Support Request exists) Riley; billing_history and payment details are shown to Maya only | N/A -- this is a visibility scope rule, not a single error message; see Authorization Rules |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View plan tier | Maya (Organiser) | Always | -- |
| View plan tier | Sam (Other Adult Member) | Always -- every adult member can see which tier the household is on | -- |
| View plan tier | Jordan (young kid profile, no login -- MVP) | Never | Screen unreachable -- a no-login profile has no sign-in path to any screen |
| View plan tier | Jordan (older kid, limited login -- Later) | Never | No entry point is shown; a direct link resolves to "This isn't part of your household view." |
| View plan tier | Riley (Operator, support -- from v1) | Only while an open Support Request exists for the household (XBR-14) | Outside an open Support Request, the screen is unreachable to Riley; a session's access ends the moment the request is resolved, showing "This support session has ended." |
| View billing history and payment details | Maya (Organiser) | Always | -- |
| View billing history and payment details | Sam (Other Adult Member) | Never | "You don't have access to billing for this household." |
| View billing history and payment details | Both Jordan rows | Never | Screen unreachable (young kid); no entry point shown, direct link denied (older kid) |
| View billing history and payment details | Riley (Operator, support) | Never, under any support scenario | Not rendered even during an open Support Request -- Riley's Billing access is View of plan tier only, never payment details |
| Initiate any billing action (upgrade, manage billing, downgrade, cancel) | Maya (Organiser) | Always (subject to FEAT-14.SPEC-005's billing-state and timing conditions) | -- |
| Initiate any billing action | Sam, both Jordan rows, Riley | Never | "You don't have access to billing for this household." (Sam, older-kid row); screen unreachable (young kid, Riley) |
| Every FEAT-14 screen and action | Unauthenticated or expired-session visitor | Never | Redirected to the sign-in screen (unauthenticated); dialog "Your session has expired. Sign in to continue." (expired session) |

## Defaults and Derivations

{This spec governs access, not derived Subscription values; defaults and derivations for tier, billing_period, billing_state, and billing_history are owned by FEAT-14.SPEC-005.}

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Viewer's access level | Derived from the requesting role in the Access Matrix (user-persona.md) plus, for Riley, whether an open Support Request exists for the household | On every screen entry and every action attempt | No -- access level is never user-settable; it follows the product's role and support-request state |

## Business Rules

- Every adult member (Maya, Sam) can see which tier the household is on, but only Maya can act on billing or see billing history and payment details, per user-persona.md's Access Matrix.
- Riley's Billing access is View of the plan tier only and never extends to payment details under any support scenario, per user-persona.md's Access Matrix notes -- this is an absolute exclusion, not conditional on the reason for the support visit.
- Riley's access opens only against an open Support Request and closes the moment that request resolves, per XBR-14; every visit is recorded where Maya (Support View) can see it, per FEAT-22.
- Neither kid row (young or older) ever reaches any FEAT-14 screen or action -- the Access Matrix sets Billing to None for both rows unconditionally.
- No FEAT-14 screen conditionally reveals Maya-only controls to another role based on entity state (e.g., a "temporary" billing action for Sam) -- the role boundary is unconditional and independent of tier or billing_state.

## Edge Cases

- **Sam attempts to reach a Maya-only billing screen via a direct link while correctly signed in as himself** -- The screen is unreachable; the exact denied experience matches the screen's own Access and Visibility table (e.g., "You don't have access to billing for this household." on FEAT-14.SPEC-002, FEAT-14.SPEC-003, FEAT-14.SPEC-004).
- **Maya hands over the organiser role to Sam mid-session (FEAT-09 organiser hand-over) while Sam has this screen's tier-only view open** -- The moment the hand-over completes, Sam's access upgrades to full Billing access on his next screen load; his in-progress tier-only view does not retroactively grant him mid-session Maya-only actions without a reload.
- **Riley's Support Request resolves while Riley has the tier-only view open** -- Access is revoked immediately; any further action attempt returns "This support session has ended.", consistent with XBR-14's requirement that support access closes when the request is resolved.
- **An older-kid limited login (Later) is added to a household that later upgrades to paid** -- The older-kid row's Billing access remains None regardless of the household's tier; upgrading the household never changes any role's access level.
- **Unauthenticated visitor follows a deep link to billing history while household referral or invitation flows are also active** -- The visitor is redirected to sign-in exactly as for any other unauthenticated attempt; no referral or invitation context grants billing visibility.

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given Maya (Organiser) requests to view plan tier, when the request is made, then it is always allowed.

**FEAT-14.SPEC-006-AC-02:** Given Sam (Other Adult Member) requests to view plan tier, when the request is made, then it is allowed -- he sees the tier but no billing actions.

**FEAT-14.SPEC-006-AC-03:** Given Sam attempts to view billing history or payment details, when the attempt is made, then it is denied with "You don't have access to billing for this household."

**FEAT-14.SPEC-006-AC-04:** Given the young kid profile (no login, MVP) attempts to reach any FEAT-14 screen, when the attempt is made, then the screen is unreachable, since no sign-in path exists.

**FEAT-14.SPEC-006-AC-05:** Given the older-kid limited login (Later) attempts to reach any FEAT-14 screen, when the attempt is made, then no entry point is shown and a direct link resolves to "This isn't part of your household view."

**FEAT-14.SPEC-006-AC-06:** Given Riley has no open Support Request for a household, when Riley attempts to view its plan tier, then the screen is unreachable.

**FEAT-14.SPEC-006-AC-07:** Given Riley has an open Support Request for a household, when Riley views it, then only the plan tier is shown, never billing history or payment details.

**FEAT-14.SPEC-006-AC-08:** Given Riley's open Support Request resolves while Riley is viewing the tier, when the resolution completes, then Riley's access ends immediately and any further action shows "This support session has ended."

**FEAT-14.SPEC-006-AC-09:** Given Maya attempts to initiate an upgrade, manage billing, or a downgrade/cancellation, when the attempt is made, then it is allowed, subject to FEAT-14.SPEC-005's billing-state conditions.

**FEAT-14.SPEC-006-AC-10:** Given Sam attempts to initiate an upgrade, manage billing, or a downgrade/cancellation, when the attempt is made, then it is denied with "You don't have access to billing for this household."

**FEAT-14.SPEC-006-AC-11:** Given an unauthenticated visitor opens a link to any FEAT-14 screen, when the link resolves, then they are redirected to the sign-in screen.

**FEAT-14.SPEC-006-AC-12:** Given a signed-in member's session expires while on any FEAT-14 screen, when they attempt an action, then the dialog "Your session has expired. Sign in to continue." appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

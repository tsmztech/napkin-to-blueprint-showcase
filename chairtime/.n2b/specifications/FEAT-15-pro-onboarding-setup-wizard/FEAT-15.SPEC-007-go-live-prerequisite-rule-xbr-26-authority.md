---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-007
spec_name: Go-Live Prerequisite Rule (XBR-26 Authority)
spec_slug: go-live-prerequisite-rule-xbr-26-authority
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 19
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Go-Live Prerequisite Rule (XBR-26 Authority)

## Overview

**Name:** Go-Live Prerequisite Rule (XBR-26 Authority)
**ID:** FEAT-15.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines and owns the exact set of steps that must be complete before the booking link can go live, per XBR-26; consumed by FEAT-15.SPEC-005 and referenced by other features that gate on go-live status.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard
**Governed Entity:** Pro Account's Go-Live readiness state (the derived "may this booking link be active" flag)

## Scope and Non-Goals

**In Scope:**
- The exact, complete list of conditions required before a booking link may go live, per XBR-26
- The single condition (calendar connection) that is explicitly excluded from the requirement
- Authorization for who may act on go-live status and who may only observe it
- The derivation logic that produces the readiness flag itself
- Being the single authoritative source other features (FEAT-05, FEAT-07) reference rather than re-deriving the condition list

**Non-Goals:**
- Tracking whether each individual condition is currently true -- owned by FEAT-15.SPEC-004 (setup-progress state) and by each condition's owning feature (FEAT-01, FEAT-02, FEAT-09, FEAT-28, FEAT-18, FEAT-27, FEAT-29); this spec only defines which conditions matter and how they combine
- Activating the link once the rule is satisfied -- owned by FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation), which consumes this rule's evaluation
- Defining the step sequence or which step is skippable in the wizard's own navigation -- owned by FEAT-15.SPEC-006; this spec defines the go-live gate, not the wizard's walking order (though the two are deliberately aligned)
- Allowing a partial or Pro-overridable go-live state -- excluded per the feature's own explicit non-goal and Validation & Limits field: the seven required conditions (the eight wizard steps less the optional calendar step) are a hard gate with no partial or Pro-overridable go-live state

## Governed Entity

**Entity:** Pro Account's Go-Live readiness state
**Source:** Feature Dependency Map (XBR-26)

| Field | Data Type | Description |
|-------|-----------|-------------|
| condition_signin | boolean | Sign-in is established (FEAT-29) |
| condition_display_name_location | boolean | Display name and studio location are set (FEAT-27) |
| condition_service | boolean | At least one Service exists (FEAT-01), which by definition includes a deposit rule |
| condition_hours | boolean | Working hours are set (FEAT-02) |
| condition_cancellation_policy | boolean | Cancellation Policy version 1 exists (FEAT-15.SPEC-002) |
| condition_payout_active | boolean | Payout Account status is Active (FEAT-28) |
| condition_subscription_active | boolean | Subscription status is Active (FEAT-18) |
| condition_calendar (excluded) | boolean | Calendar connection status (FEAT-04) -- tracked for display purposes only; never included in the readiness computation |
| is_ready | derived (boolean) | True only when all seven required conditions above (excluding calendar) are true |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | Evaluates is_ready on every relevant trigger and activates the link when it becomes true |
| FEAT-15.SPEC-003 | Go-Live Preview & Booking Link Hand-Over | Displays Live vs. Waiting based on is_ready and, specifically, condition_payout_active when that is the sole remaining gap |
| FEAT-05 | Public Booking Page & Booking Flow | Reads whether the link is live (the outcome of this rule via FEAT-15.SPEC-005) rather than re-deriving the condition list, per XBR-26's authority note |
| FEAT-07 | Deposit Payment at Booking | Reads the same live/not-live outcome (via FEAT-28's payout-active check, which is itself one of this rule's seven conditions) to determine whether a deposit may be taken, per XBR-06 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| condition_signin | Must be true; sourced from FEAT-29 | Always | On every readiness evaluation | N/A -- this is an internal derived condition, not a user-facing input field, so it carries no error message of its own | Yes |
| condition_display_name_location | Must be true; sourced from FEAT-27 | Always | On every readiness evaluation | N/A | Yes |
| condition_service | Must be true; sourced from FEAT-01 (existence of at least one Service, which requires a deposit rule by that entity's own definition) | Always | On every readiness evaluation | N/A | Yes |
| condition_hours | Must be true; sourced from FEAT-02 | Always | On every readiness evaluation | N/A | Yes |
| condition_cancellation_policy | Must be true; sourced from FEAT-15.SPEC-002 | Always | On every readiness evaluation | N/A | Yes |
| condition_payout_active | Must be true; sourced from FEAT-28 (status = Active, not merely Verification Pending) | Always | On every readiness evaluation | N/A | Yes |
| condition_subscription_active | Must be true; sourced from FEAT-18 (status = Active) | Always | On every readiness evaluation | N/A | Yes |
| condition_calendar | No validation applied -- explicitly excluded from the readiness computation regardless of its value | Always excluded | Never checked for readiness purposes (tracked separately for display only, per FEAT-15.SPEC-006) | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Readiness is a strict conjunction | condition_signin, condition_display_name_location, condition_service, condition_hours, condition_cancellation_policy, condition_payout_active, condition_subscription_active | is_ready = true only when every one of these seven is true; any single false condition makes is_ready false -- there is no weighted, partial, or majority-based readiness | N/A -- surfaced to the Pro as the Waiting state (FEAT-15.SPEC-003), not as a field-level error |
| Calendar exclusion | condition_calendar, is_ready | condition_calendar's value never participates in the is_ready computation in any way, regardless of whether it is true, false, or undefined | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own Go-Live readiness state | The Pro (Talia) | Always, for her own account | -- |
| Force or override Go-Live activation ahead of readiness | The Pro (Talia) | Never | No control of any kind exists anywhere in the product to activate a link before all seven conditions are true; the feature's own Validation & Limits field states this is a hard gate with no override |
| View Go-Live readiness state | Platform Operator (Support) | Always, for any Pro Account during an active help request, read-only | -- |
| Force or override Go-Live activation on a Pro's behalf | Platform Operator (Support) | Never | No action control is shown to Support in any progress or readiness view; per SC-05, Support cannot act on a Pro's account |
| View or act on Go-Live readiness for any account | The Client (Riley) | Never | No client-facing surface exposes readiness state directly; a client only ever sees the resulting live-or-not-reachable booking page |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| condition_* fields | Default to false (not yet satisfied) on Pro Account creation | On Pro Account creation | No -- each becomes true only through its own owning feature's completion |
| is_ready | Derived: logical AND of all seven required condition_* fields | Recomputed on every trigger listed in FEAT-15.SPEC-005's Trigger Definition | No -- entirely derived, never directly settable by any role |

## Business Rules

- XBR-26 (the cross-feature business rule this spec is the authority for): "The booking link goes live only when sign-in, display name and studio location, one service, working hours, deposit rule, cancellation policy, an active payout account and an active subscription are all in place; calendar connection is the only optional step." This spec's seven required conditions are the literal enumeration of that rule (deposit rule is folded into condition_service, since a Service cannot exist without one, per FEAT-01's own entity definition).
- FEAT-15 is the sole owner of this rule; other features that gate behavior on go-live status (FEAT-05, FEAT-07 via XBR-06) reference this rule's outcome rather than re-deriving or duplicating the condition list, per the dependency map's Authority column for XBR-26.
- There is no partial, weighted, or Pro-overridable readiness state -- every one of the seven conditions is mandatory and none can be waived, per the feature's own Non-Goals.
- The calendar connection condition is permanently excluded from this computation; it can never become a blocking condition under any configuration, present or future within this feature's scope, since XBR-26 names it explicitly as the one optional step.

## Edge Cases

- **All seven required conditions are true except condition_payout_active, which is Verification Pending rather than outright false** -- Treated identically to any other unmet condition: is_ready is false. This specific gap is surfaced distinctly by FEAT-15.SPEC-003 (the "finish verifying" Waiting state) precisely because it is common enough near the end of setup to warrant its own message, but the underlying rule treats it the same as any other unmet condition.
- **condition_subscription_active later becomes false after the link has already gone live (subscription lapses)** -- Out of scope for this rule's own gating logic: this spec governs the one-time transition to live; an already-live link's subsequent pause on subscription lapse is XBR-14's concern, owned by FEAT-27 and FEAT-18, not a re-evaluation of this go-live gate.
- **A new required condition is proposed in a future version (hypothetical: e.g., requiring an intro photo)** -- Not evaluated by this rule as written; any change to the seven-condition set is a change to XBR-26 itself, made explicitly at that authority, never inferred by another feature adding conditions unilaterally.
- **condition_calendar is true (connected) while every other condition is also true** -- No different outcome than if condition_calendar were false or unset: is_ready is already true from the seven required conditions alone; the calendar's own state adds nothing to and subtracts nothing from readiness.
- **Two required conditions are read at slightly different moments within one evaluation (e.g., a fast payout-status read and a slower subscription-status read)** -- The evaluation in FEAT-15.SPEC-005 reads a consistent snapshot of all seven conditions together before computing is_ready (per that spec's Processing Logic), so this rule itself never operates on a torn read across two different points in time.
- **A downstream feature (FEAT-07) needs to know if a deposit may be taken, which depends on payout status alone (XBR-06), not the full go-live rule** -- FEAT-07 reads condition_payout_active (via FEAT-28) directly for its own XBR-06 check; it does not need is_ready as a whole, since a deposit's precondition and a link's go-live precondition are related but distinct questions this spec keeps separately readable.

## Acceptance Criteria

**FEAT-15.SPEC-007-AC-01:** Given all seven required conditions are true, when is_ready is computed, then it evaluates to true.

**FEAT-15.SPEC-007-AC-02:** Given exactly one required condition (payout active) is false and the other six are true, when is_ready is computed, then it evaluates to false.

**FEAT-15.SPEC-007-AC-03:** Given condition_calendar is false (never connected, never skipped -- a theoretical unset state), when is_ready is computed with all seven required conditions true, then it still evaluates to true, since condition_calendar never participates in the computation.

**FEAT-15.SPEC-007-AC-04:** Given condition_calendar is true (connected) and all seven required conditions are also true, when is_ready is computed, then the outcome is the same as if condition_calendar were false -- true either way.

**FEAT-15.SPEC-007-AC-05:** Given Talia's account has a Service record, when condition_service is evaluated, then it is true, and the deposit-rule requirement is satisfied by the same check, since FEAT-01's Service entity cannot exist without one.

**FEAT-15.SPEC-007-AC-06:** Given Talia (the Pro) looks for any way to force her link live before all seven conditions are met, when she searches the product, then no such control exists anywhere.

**FEAT-15.SPEC-007-AC-07:** Given Platform Operator (Support) views a Pro's readiness state during a help request, when they look for an override action, then none is available to them.

**FEAT-15.SPEC-007-AC-08:** Given a client (Riley) has no reachable surface for readiness state, when any client-facing screen is inspected, then readiness state is never exposed directly, only its outcome (the booking page being reachable or not).

**FEAT-15.SPEC-007-AC-09:** Given Talia's subscription lapses after her link is already live, when this rule is consulted, then it is not re-evaluated to deactivate the link -- that behavior is owned by XBR-14 via FEAT-27 and FEAT-18, outside this spec's one-time gating scope.

**FEAT-15.SPEC-007-AC-10:** Given FEAT-05 needs to know whether a Pro's link is live, when it checks, then it reads the outcome of this rule (via FEAT-15.SPEC-005's activation state) rather than re-deriving any condition itself.

**FEAT-15.SPEC-007-AC-11:** Given FEAT-07 needs to know whether a deposit may be taken, when it checks per XBR-06, then it reads condition_payout_active directly rather than requiring the full seven-condition is_ready to be true.

**FEAT-15.SPEC-007-AC-12:** Given all seven required conditions are true except condition_payout_active, which shows Verification Pending, when FEAT-15.SPEC-003 renders, then it shows the "finish verifying" Waiting state, consistent with this rule treating that gap the same as any other unmet condition.

**FEAT-15.SPEC-007-AC-13:** Given an evaluation reads payout status and subscription status at slightly different instants within one run, when is_ready is computed, then the computation uses a single consistent snapshot of all seven conditions rather than a mix of stale and fresh values.

**FEAT-15.SPEC-007-AC-14:** Given all seven required conditions are true and condition_calendar has just transitioned from incomplete to complete-as-skipped, when is_ready is recomputed, then the transition itself has no effect on the already-true is_ready outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

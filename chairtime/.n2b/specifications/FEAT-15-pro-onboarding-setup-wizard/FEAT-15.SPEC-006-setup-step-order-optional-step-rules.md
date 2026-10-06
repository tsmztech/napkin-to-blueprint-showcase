---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-006
spec_name: Setup Step Order & Optional-Step Rules
spec_slug: setup-step-order-optional-step-rules
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 22
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Setup Step Order & Optional-Step Rules

## Overview

**Name:** Setup Step Order & Optional-Step Rules
**ID:** FEAT-15.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines the fixed step sequence, which single step (calendar connection) is optional and resumable from settings later, and how a skipped step is represented in progress.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard
**Governed Entity:** Pro Account's setup-progress state (the per-step completion sub-structure of the Pro Account entity)

## Scope and Non-Goals

**In Scope:**
- The fixed order of the 8 setup steps
- Which step is optional (calendar connection) and how a skip is recorded and later resolved
- Authorization for who may view or advance setup progress
- Defaults and derivations for the resume-point computation
- The cross-step "offer a default, let the Pro accept or change it" pattern this feature establishes for every hand-off step

**Non-Goals:**
- Determining Go-Live readiness -- owned by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule, XBR-26); this spec governs sequencing and optionality, not the separate question of when the link may activate
- Writing or reading the progress record -- owned by FEAT-15.SPEC-004 (Setup Progress Tracking & Resume); this spec defines the rules that automation enforces, not the mechanics of persistence
- Field-level validation within any individual step's own form (service fields, hours, payout details) -- each step's owning feature governs its own field rules (e.g., FEAT-01.SPEC-004 for service fields); this spec governs only the sequence and optionality of steps as a whole
- Multi-staff or role-based setup paths -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so exactly one linear sequence exists for one Pro

## Governed Entity

**Entity:** Pro Account (setup-progress state)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| step_account_signin | enum (complete) | Always complete once the wizard is entered, since sign-in (FEAT-29) must succeed first |
| step_profile | enum (incomplete, complete) | Whether display name and studio location have been captured (FEAT-27) |
| step_service | enum (incomplete, complete) | Whether at least one Service exists (FEAT-01) |
| step_hours | enum (incomplete, complete) | Whether working hours have been set (FEAT-02) |
| step_cancellation_policy | enum (incomplete, complete) | Whether Cancellation Policy version 1 exists (FEAT-15.SPEC-002) |
| step_payout | enum (incomplete, complete) | Whether a Payout Account exists in at least Verification Pending status (FEAT-28) |
| step_calendar | enum (incomplete, complete-as-connected, complete-as-skipped) | The one step this spec marks optional (FEAT-04) |
| step_subscription | enum (incomplete, complete) | Whether the Subscription is at least started (FEAT-18) |
| resume_point | derived | The first step in fixed order not yet complete (complete-as-skipped counts as complete) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-001 | Setup Wizard Shell, Step Navigation & Guidance | On every screen load, to determine which steps are tappable, dimmed, or current, and to display resume_point |
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | On every step-completion signal, to determine which step advances and how a skip is recorded |
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | On every readiness evaluation, to determine which steps count toward the seven required conditions (excluding the optional calendar step) |
| FEAT-04.SPEC-001 | Calendar Connection Setup | When Talia is offered the "skip and connect later" choice |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| step_account_signin | No validation beyond data type -- always set to complete on wizard entry | Always | -- | -- | -- |
| step_profile | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_service | Must transition only from incomplete to complete; never reverts once at least one Service has ever existed | Always | On completion signal | -- | -- |
| step_hours | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_cancellation_policy | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_payout | Must transition only from incomplete to complete; never reverts once a Payout Account has ever reached at least Verification Pending | Always | On completion signal | -- | -- |
| step_calendar | May transition from incomplete to complete-as-skipped or complete-as-connected, and from complete-as-skipped to complete-as-connected; never from complete-as-connected back to incomplete or skipped | Always | On completion or skip signal | -- | -- |
| step_subscription | Must transition only from incomplete to complete; never reverts within this feature's scope (a later cancellation is FEAT-18's own concern, not a reversion of this setup step) | Always | On completion signal | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Fixed sequence display | step_account_signin, step_profile, step_service, step_hours, step_cancellation_policy, step_payout, step_calendar, step_subscription | The wizard shell (FEAT-15.SPEC-001) shows steps in this exact order; an upcoming step (one after the first incomplete step) is never made tappable regardless of any individual field's own state | N/A -- this is a display/navigation constraint, not a user-facing validation error |
| Calendar step is excluded from advancement blocking | step_calendar, resume_point | resume_point calculation treats step_calendar as satisfied whenever it is complete-as-skipped OR complete-as-connected, so an incomplete calendar step never becomes the resume_point once explicitly skipped | N/A |
| Resume point derivation | All eight step_* fields | resume_point = the first field in the fixed order (per Field Validation Rules) whose value is still "incomplete"; if all are satisfied, resume_point is null and setup is complete | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own setup progress | The Pro (Talia) | Always, for her own account only | -- |
| Advance a step (via its owning screen) | The Pro (Talia) | Only the step at resume_point may be advanced through the wizard shell's Continue action; a completed step may be re-opened for review through the header list without being able to un-complete it | Attempting to advance a step ahead of resume_point is not exposed as a possible action -- the step-list item for any step after resume_point is rendered non-interactive (dimmed) rather than producing an error |
| Skip the calendar step | The Pro (Talia) | Only the calendar step, only while it is incomplete | No other step exposes a skip choice; the choice control appears only on FEAT-04.SPEC-001's own screen for this one step |
| View setup progress | Platform Operator (Support) | Always, for any Pro Account during an active help request, read-only | -- |
| Advance, skip, or otherwise modify a step on the Pro's behalf | Platform Operator (Support) | Never | No action control of any kind is shown to Support in the progress view surfaced by FEAT-19; per SC-05 and XBR-24, a stuck Pro must resolve their own step |
| View or act on any Pro Account's setup progress | The Client (Riley) | Never | No client-facing surface of any kind exposes setup progress; the concept does not exist in any client-reachable screen |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| step_account_signin | Defaults to complete | On Pro Account creation (first wizard entry) | No -- it is a fact of having reached the wizard at all |
| All other step_* fields | Default to incomplete | On Pro Account creation | No -- they become complete only through their own owning screen's flow |
| resume_point | Derived: first incomplete step in fixed order | Recomputed on every progress change and every wizard shell load | No -- it is entirely derived, never directly settable |
| step_calendar | Defaults to incomplete; the Pro chooses complete-as-connected (by connecting) or complete-as-skipped (by explicit skip) | On reaching the calendar step in the fixed order | Yes -- the Pro may later convert complete-as-skipped to complete-as-connected by connecting from settings (FEAT-04), at any time |

## Business Rules

- The fixed step order is: (1) account & sign-in, (2) profile & studio location, (3) at least one service (including its deposit rule), (4) working hours, (5) cancellation policy, (6) payout account, (7) calendar connection (optional), (8) subscription -- matching the Feature Breakdown Brief's Key Capabilities and the dependency map's Navigation Connections table exactly.
- The calendar-connection step is the only step in the sequence that can be marked complete without its underlying capability having actually been used, per the Brief's Side-Effect Inventory ("Pro reaches the calendar-connection step -> offer the skip choice -> mark the step complete-as-skipped without blocking progress").
- A skipped calendar step never counts against Go-Live readiness (XBR-26; enforced by FEAT-15.SPEC-007) -- this spec only defines that the step is marked satisfied either way; FEAT-15.SPEC-007 is the authority on which steps that satisfaction actually gates.
- The "offer a default, let the Pro accept or change it" pattern established concretely by FEAT-15.SPEC-002 (the cancellation window) is the general pattern every hand-off step's owning feature should also apply to its own defaults (e.g., a suggested buffer time in FEAT-02, a suggested deposit percentage in FEAT-01), so the wizard reads as one consistent experience end to end, per the Brief's Shared Context.
- No step may be advanced out of order through the wizard shell; the shell's own enforcement (FEAT-15.SPEC-001) makes upcoming steps non-interactive rather than presenting them as actionable and then rejecting the action.

## Edge Cases

- **Talia reaches the calendar step, skips it, and never returns to it** -- step_calendar remains complete-as-skipped indefinitely; this has no expiry and never blocks resume_point from advancing past it, since it is treated as satisfied.
- **Talia completes the subscription step (step 8) before every earlier step, by navigating a deep link outside the wizard** -- Not possible through the wizard shell's own navigation (steps are gated in order), but if an owning feature's screen is reached directly and its data is saved (for example, Talia subscribes via a settings-area link before finishing profile setup), FEAT-15.SPEC-004 still records step_subscription as complete out of order; resume_point calculation is unaffected, since it always evaluates the fixed order from the start regardless of which steps happen to already be complete.
- **Two of the eight step fields are updated by two completion signals arriving in the same instant** -- Each field is independent (per Field Validation Rules, no field's transition depends on another field's simultaneous value), so both updates apply cleanly with no conflict.
- **step_calendar is at exactly the boundary between complete-as-skipped and complete-as-connected (Talia connects immediately after skipping, before advancing past the step)** -- The transition from complete-as-skipped to complete-as-connected is explicitly allowed (see Field Validation Rules); the reverse (complete-as-connected back to skipped or incomplete) is never allowed, since disconnecting a calendar afterward is FEAT-04's own disconnect action, not a reversion of this setup step.
- **All eight fields become complete at once (a rare but possible burst, e.g., a batch of upstream statuses resolving together)** -- resume_point correctly computes to null; this spec makes no further claim about what happens next, since Go-Live activation itself is FEAT-15.SPEC-007's and FEAT-15.SPEC-005's authority.
- **Support attempts to view progress for a Pro Account with no help request context** -- Not a rule this spec governs directly; FEAT-19 owns the authorization gate on when Support may open any Pro's account at all. This spec's Authorization Rules apply once Support is validly viewing an account per FEAT-19's own rules.

## Acceptance Criteria

**FEAT-15.SPEC-006-AC-01:** Given Talia has just entered the wizard for the first time, when the progress record is created, then step_account_signin is already complete and all other step_* fields are incomplete.

**FEAT-15.SPEC-006-AC-02:** Given Talia has completed steps 1 through 4, when resume_point is computed, then it evaluates to step 5 (cancellation policy).

**FEAT-15.SPEC-006-AC-03:** Given Talia is at the calendar step, when she chooses "skip and connect later," then step_calendar transitions to complete-as-skipped and resume_point advances to step 8 (subscription).

**FEAT-15.SPEC-006-AC-04:** Given Talia skipped the calendar step, when she later connects a calendar from settings, then step_calendar transitions from complete-as-skipped to complete-as-connected.

**FEAT-15.SPEC-006-AC-05:** Given Talia has already connected her calendar (complete-as-connected), when any process attempts to mark it skipped, then the transition is rejected -- a connected calendar step can never revert to skipped or incomplete.

**FEAT-15.SPEC-006-AC-06:** Given Talia (the Pro) is on the wizard shell, when she looks at a step after resume_point, then it is rendered non-interactive (dimmed), and no action is available to advance it out of order.

**FEAT-15.SPEC-006-AC-07:** Given Talia (the Pro) taps a completed step's header entry, when she views it, then she can review it but no control exists to un-complete it.

**FEAT-15.SPEC-006-AC-08:** Given Platform Operator (Support) opens Talia's account during a help request, when they view her setup progress, then they see the same fields read-only with no action control shown.

**FEAT-15.SPEC-006-AC-09:** Given Platform Operator (Support) is viewing a Pro's setup progress, when they look for any way to advance, skip, or modify a step, then no such control exists anywhere in that view.

**FEAT-15.SPEC-006-AC-10:** Given a client (Riley) attempts to reach any setup-progress surface, when the attempt is made, then no such surface is reachable by a client under any circumstance.

**FEAT-15.SPEC-006-AC-11:** Given every one of the eight step_* fields is satisfied at once, when resume_point is recomputed, then it evaluates to null.

**FEAT-15.SPEC-006-AC-12:** Given Talia's step_service field is already complete, when a duplicate completion signal for the services step arrives, then the field remains complete and does not toggle or reset.

**FEAT-15.SPEC-006-AC-13:** Given the calendar step is skipped, when FEAT-15.SPEC-007 evaluates Go-Live readiness, then the skipped calendar step never counts against it.

**FEAT-15.SPEC-006-AC-14:** Given Talia subscribes via a settings-area link before completing her profile step, when FEAT-15.SPEC-004 records the subscription completion, then resume_point still correctly reflects the profile step as the next incomplete step in fixed order.

**FEAT-15.SPEC-006-AC-15:** Given the wizard shell displays the eight steps, when Talia views the header list, then the order shown exactly matches: account & sign-in, profile, service, hours, cancellation policy, payout, calendar (optional), subscription.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

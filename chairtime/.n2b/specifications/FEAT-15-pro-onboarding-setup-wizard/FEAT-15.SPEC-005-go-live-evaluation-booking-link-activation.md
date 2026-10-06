---
document_type: spec
spec_type: automation
spec_id: FEAT-15.SPEC-005
spec_name: Go-Live Evaluation & Booking Link Activation
spec_slug: go-live-evaluation-booking-link-activation
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Go-Live Evaluation & Booking Link Activation

## Overview

**Name:** Go-Live Evaluation & Booking Link Activation
**ID:** FEAT-15.SPEC-005
**Type:** Automation
**Purpose:** Re-evaluates readiness against the Go-Live Prerequisite Rule after every relevant step completion or upstream status change, and activates Talia's booking link the moment it is satisfied.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Re-evaluating Go-Live readiness whenever a required step's completion state changes (FEAT-15.SPEC-004) or an upstream status this rule depends on changes (payout account status, subscription status)
- Activating the booking link (making it publicly reachable at FEAT-05) the instant readiness is reached
- Emitting onboarding_completed and the state that makes FEAT-15.SPEC-008's welcome confirmation eligible to fire
- Reporting readiness (or its absence, with the specific gap) to FEAT-15.SPEC-003 for display

**Non-Goals:**
- Defining which steps are required and which is optional -- owned entirely by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule, XBR-26 authority); this automation only consumes that rule's evaluation
- Tracking step completion itself -- owned by FEAT-15.SPEC-004; this automation is notified of changes, it does not compute them
- Rendering the live link or the waiting state to Talia -- owned by FEAT-15.SPEC-003; this automation only supplies the activation event that screen displays
- Deactivating a link once live (for example, on subscription lapse or account pause) -- owned by FEAT-27 (pause state, XBR-14) and FEAT-18 (subscription lapse); this automation's activation is a one-time, one-directional transition from not-live to live, never the reverse

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A required step's completion state changes | FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Fires every time any of the eight steps transitions between complete, complete-as-skipped, and incomplete | Current progress record for all 8 steps |
| Payout account status changes | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when the Payout Account's status changes to or from Active | Payout Account status |
| Subscription status changes | FEAT-18.SPEC-006 (FEAT-18's subscription-billing automation; external-event trigger source, per the External Touchpoints table) | Fires when the Subscription's status changes to or from Active | Subscription status |

## Processing Logic

1. On any trigger, read the current state of all seven prerequisites defined by FEAT-15.SPEC-007's Go-Live Prerequisite Rule: sign-in complete, display name and studio location set, at least one Service exists (which by definition includes a deposit rule, per FEAT-01's Service entity), working hours set, the Cancellation Policy version 1 exists, Payout Account status is Active, and Subscription status is Active.
2. Evaluate FEAT-15.SPEC-007's rule against that state: all seven required conditions (the calendar step is explicitly excluded from this evaluation, per XBR-26) must be true.
3. If the booking link is already active, and the rule is still satisfied, take no action (idempotent -- this automation never re-activates an already-live link or emits a duplicate completion event).
4. If the booking link is not yet active and the rule is now satisfied: activate the link (make FEAT-05's public booking page reachable at Talia's booking_link_name), record the activation timestamp, and emit onboarding_completed.
5. If the booking link is not yet active and the rule is not yet satisfied: take no activation action; compute which specific condition(s) are still unmet (for display purposes) and make that gap available to FEAT-15.SPEC-003.
6. If the booking link is already active and a later trigger reports one of the seven conditions has become false (for example, a payout account regresses from Active to Action Required after the link is already live) -- this automation does not deactivate the link: deactivation on an already-live link is out of scope (see Non-Goals) and owned by the account-pause/subscription-lapse mechanisms in FEAT-27 and FEAT-18, not by this one-directional activation automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link activated | All seven required conditions are satisfied for the first time | Booking link marked active; activation timestamp recorded | Talia sees the Live state on FEAT-15.SPEC-003; the welcome confirmation (FEAT-15.SPEC-008) becomes eligible and fires | FEAT-15.SPEC-003, FEAT-15.SPEC-008, FEAT-05 |
| Readiness still pending | One or more required conditions remain unmet | No change | Talia sees the Waiting state on FEAT-15.SPEC-003 if the payout condition specifically is the gap, or remains inside the wizard shell (FEAT-15.SPEC-001) for any other unmet condition, since the shell does not hand off to FEAT-15.SPEC-003 until FEAT-15.SPEC-001's own final-step completion signal fires | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |
| No-op (already active, still satisfied) | Link is already active and re-evaluation confirms the rule remains satisfied | None | No visible change | -- |
| No-op (already active, a condition regresses) | Link is already active and a later trigger reports a required condition now false | None -- this automation never deactivates a live link | No visible change from this automation; any resulting Pro-facing banner or notification is owned by FEAT-27/FEAT-18's own attention mechanisms, not this spec | FEAT-27, FEAT-18 (out of this spec's scope) |
| Automation failure | Processing error during evaluation | No partial activation state is ever persisted -- the link is either fully active or not active, never partially | Talia sees the wizard shell or FEAT-15.SPEC-003 in its previous known state; a retry occurs on the next trigger (e.g., the next step completion or a manual refresh of FEAT-15.SPEC-003) | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |

## Data Model

**Reads:** Pro Account setup-progress state (FEAT-15.SPEC-004), Payout Account status (FEAT-28), Subscription status (FEAT-18) -- all read-only, per the dependency map's Referenced Entities table for FEAT-15.
**Creates:** None.
**Updates:** The booking link's active/not-active state and activation timestamp on the Pro Account -- owned exclusively by this automation; no other spec activates the link.
**Deletes:** None.

## Business Rules

- FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) determines which steps count toward the seven required conditions on every evaluation; the optional calendar step never counts, whether it is complete, skipped, or incomplete.
- FEAT-15.SPEC-007 (XBR-26) is the sole authority for which conditions gate go-live; this automation never adds, removes, or reinterprets a condition on its own.
- Activation is one-directional: once a link is active, this automation never deactivates it. A regression in an underlying condition (payout action-required, subscription lapse) is handled entirely by the owning feature's own pause/lapse mechanism (FEAT-27, FEAT-18), consistent with XBR-11's "setup changes never silently cancel a confirmed booking" and XBR-14's pause behavior, which apply to an already-live account, not to this feature's one-time go-live transition.
- Evaluation is idempotent -- re-running it against an unchanged state never re-emits onboarding_completed or re-activates an already-active link.
- Re-evaluation is triggered by every relevant upstream change, not on a fixed schedule, so activation happens "the moment" readiness is reached, per the Key Capability's stated behavior, never on a delay.

## Edge Cases

- **Two required conditions become true at nearly the same moment (for example, Talia's subscription payment succeeds at the same instant her payout account finishes verification)** -- Concurrent trigger firing: each trigger independently re-reads the full current state of all seven conditions (step 1 of Processing Logic), so whichever trigger's evaluation runs second sees both conditions already true and activates the link; the first to run either activates it (if it also sees both true) or finds one still pending and takes no action. Exactly one activation ever occurs, never two, because step 3's idempotency check prevents a duplicate activation regardless of which trigger's evaluation "wins."
- **Trigger fires while a previous evaluation run is in flight** -- Because each evaluation run reads a fresh snapshot of all seven conditions independently rather than relying on a value carried over from a prior run, an overlapping second run produces the same correct outcome as if it had waited; both runs converge on the same activation decision, and step 3's idempotency guard prevents any duplicate activation or event.
- **Talia's payout account regresses from Active to Verification Pending again after the link is already live (an unusual processor-side event)** -- Per the Non-Goals and Processing Logic step 6, this automation takes no deactivation action; the link remains live, consistent with "setup changes never silently cancel" behavior owning to other features once an account is live.
- **All seven conditions are satisfied except the deposit rule, because Talia added a service with a name and duration but has not yet set its deposit rule** -- Not possible as a standalone gap: per FEAT-01's Service entity, a deposit rule is a required field of every Service record at creation, so a Service cannot exist without one; "at least one Service exists" and "a deposit rule exists" are therefore always satisfied together, never independently.
- **Talia skips the calendar step and every other required condition is satisfied** -- The link activates normally: the calendar step is explicitly excluded from the seven required conditions per XBR-26, so a skip never blocks or delays activation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Triggered by (inbound) | Every progress change re-triggers this automation's evaluation |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Enforcing side of its Enforced-By entry: on every readiness evaluation this automation applies FEAT-15.SPEC-006 to decide which steps count toward the seven required conditions (the optional calendar step is excluded) |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (inbound) | Supplies the exact condition set this automation evaluates |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Affects (outbound) | Readiness confirmation is what causes the shell to hand off to FEAT-15.SPEC-003 |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Affects (outbound) | This automation's activation state is exactly what that screen displays as Live or Waiting |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | Affects (outbound) | Link activation is the trigger event for the welcome confirmation |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Payout status changes re-trigger evaluation |
| FEAT-18 (subscription-billing automation, FEAT-18.SPEC-006) | Triggered by (inbound) | Subscription status changes re-trigger evaluation |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Activation is what makes FEAT-05's public page reachable at Talia's booking link |

## Analytics and Success Signals

- **onboarding_completed** (elapsed time since onboarding_started, count of steps completed vs. skipped) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **golive_evaluation_run** (outcome: activated / still_pending / no_op; gap conditions if still pending) -- N/A -- no Stage 2 metric directly measures individual evaluation runs; retained internally so a stalled Pro's specific blocking condition is observable to Support (via FEAT-19) rather than invisible.

## Acceptance Criteria

**FEAT-15.SPEC-005-AC-01:** Given Talia has completed every required step except her subscription, when she completes the subscription step, then this automation re-evaluates and, finding all seven conditions satisfied, activates her booking link immediately.

**FEAT-15.SPEC-005-AC-02:** Given Talia's booking link has just activated, when the activation completes, then onboarding_completed is emitted and FEAT-15.SPEC-008's welcome confirmation becomes eligible to send.

**FEAT-15.SPEC-005-AC-03:** Given Talia has completed every required step except her payout account is still Verification Pending, when this automation evaluates readiness, then it does not activate the link, and FEAT-15.SPEC-003 shows the Waiting state.

**FEAT-15.SPEC-005-AC-04:** Given Talia's payout account later becomes Active, when FEAT-28.SPEC-003 reports the status change, then this automation re-evaluates and activates the link without requiring Talia to take any wizard action.

**FEAT-15.SPEC-005-AC-05:** Given Talia's link is already active and a later trigger fires reporting no change in any condition, when this automation re-evaluates, then no duplicate onboarding_completed event is emitted and no re-activation occurs.

**FEAT-15.SPEC-005-AC-06:** Given Talia's payout account regresses from Active to Action Required after her link is already live, when this automation is triggered by that status change, then the link remains live and this automation takes no deactivation action.

**FEAT-15.SPEC-005-AC-07:** Given Talia skips the calendar-connection step and every other required condition is satisfied, when this automation evaluates readiness, then the link activates normally, since the calendar step is excluded from the seven required conditions.

**FEAT-15.SPEC-005-AC-08:** Given two required conditions (subscription and payout) become true at nearly the same instant, when both triggers fire, then exactly one activation occurs and exactly one onboarding_completed event is emitted.

**FEAT-15.SPEC-005-AC-09:** Given a second evaluation run starts while a first run for the same Pro Account is still in flight, when both complete, then they converge on the same activation decision with no duplicate activation.

**FEAT-15.SPEC-005-AC-10:** Given Talia's account has a Service record, when this automation checks the deposit-rule condition, then it is always found satisfied together with the service-exists condition, since a Service cannot exist without a deposit rule.

**FEAT-15.SPEC-005-AC-11:** Given a processing error occurs during evaluation, when the error is detected, then no partial activation state is persisted, and the next relevant trigger re-evaluates from a clean, fully-read state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

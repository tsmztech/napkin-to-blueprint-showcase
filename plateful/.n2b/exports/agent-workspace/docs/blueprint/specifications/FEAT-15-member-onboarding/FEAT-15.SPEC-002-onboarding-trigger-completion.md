---
document_type: spec
spec_type: automation
spec_id: FEAT-15.SPEC-002
spec_name: Onboarding Trigger & Completion
spec_slug: onboarding-trigger-completion
parent_feature: FEAT-15
parent_feature_name: Member Onboarding
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Onboarding Trigger & Completion

## Overview

**Name:** Onboarding Trigger & Completion
**ID:** FEAT-15.SPEC-002
**Type:** Automation
**Purpose:** Fires when an invited adult's invitation acceptance succeeds, confirms eligibility and once-only status through FEAT-15.SPEC-003, routes the new member to the Onboarding Landing exactly once, and emits the feature's start and completion signals.
**Parent Feature:** FEAT-15 -- Member Onboarding

## Scope and Non-Goals

**In Scope:**
- Receiving the acceptance outcome from FEAT-09.SPEC-007 (Invitation Acceptance Processing)
- Checking eligibility and once-only status via FEAT-15.SPEC-003 before routing
- Routing the new member to FEAT-15.SPEC-001 (Onboarding Landing) when eligible and not yet shown
- Falling back to the ordinary Weekly Plan View when onboarding does not fire
- Emitting member_onboarding_started and member_onboarding_completed

**Non-Goals:**
- Creating the Member Profile or transitioning the Invitation to Accepted -- owned by FEAT-09.SPEC-007; this automation only reacts to that outcome, it does not perform or duplicate it
- Determining who is eligible or whether onboarding has already fired for a given invitation -- owned by FEAT-15.SPEC-003; this automation calls that rule rather than restating its logic
- Rendering the plan and grocery list -- owned by FEAT-15.SPEC-001, FEAT-03.SPEC-001/FEAT-23.SPEC-001, and FEAT-06.SPEC-001
- Sending any notification or email for this moment -- excluded per the feature's Communications field: onboarding has no message of its own; the invitation itself (FEAT-09) is the communication (feature-overview.md, Non-Goals)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitation acceptance succeeds | FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Fires on that automation's "Acceptance succeeds" outcome (new Member Profile created, Invitation.status set to Accepted) | The new Member Profile reference (member_type, status, display_name) and the just-accepted Invitation reference (status, and whether its contact detail maps to a previously Removed or Left Member Profile of this household) |

This is the feature's only trigger path -- an event-driven trigger sourced from a specific outcome of a spec in another feature (FEAT-09), consistent with the Internal Dependency Map's statement that landing "does not happen on its own."

## Processing Logic

1. Receive the newly created Member Profile reference and the just-accepted Invitation reference from FEAT-09.SPEC-007's "Acceptance succeeds" outcome.
2. Emit member_onboarding_started, recording the trigger moment.
3. Evaluate eligibility per FEAT-15.SPEC-003: confirm the Member Profile's member_type is Other Adult Member.
4. Evaluate once-only status per FEAT-15.SPEC-003: confirm onboarding has not already been shown for this accepted Invitation's transition.
5. Evaluate re-join status per FEAT-15.SPEC-003: determine whether the Member Profile's status immediately before this acceptance was Removed or Left.
6. If eligible and not yet shown, route the member to FEAT-15.SPEC-001 (Onboarding Landing), passing the Member Profile reference. This applies identically whether or not step 5 flagged a re-join -- a re-join always onboards fresh, with no prior data read or applied.
7. If ineligible, or if onboarding has already been shown for this transition, do not route to Onboarding Landing; instead route the member to the ordinary Weekly Plan View (FEAT-03.SPEC-001 or FEAT-23.SPEC-001, per the household's tier) as their first screen.
8. When FEAT-15.SPEC-001 successfully completes its first render for this member (whether the populated or the explained empty-state variant), emit member_onboarding_completed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Onboarding fires -- eligible, first time | FEAT-15.SPEC-003 confirms Other Adult Member and no prior onboarding shown for this Invitation's transition | None persisted by this automation itself (FEAT-15 owns no entity of its own) | Member is routed to and lands on FEAT-15.SPEC-001 | FEAT-15.SPEC-001 |
| Onboarding fires -- eligible re-join | Same as above, and FEAT-15.SPEC-003 additionally flags this as a re-join (prior status Removed or Left) | None; no prior data restored | Member is routed to FEAT-15.SPEC-001, landing exactly as a first-time member would | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |
| No onboarding -- ineligible role | FEAT-15.SPEC-003 determines the accepted role is not Other Adult Member (evaluated defensively; FEAT-09.SPEC-007 always creates Other Adult Member profiles on acceptance, so this path is not expected to occur in practice) | None | Member is routed to the ordinary Weekly Plan View instead | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |
| No onboarding -- already shown | FEAT-15.SPEC-003 determines onboarding was already shown for this accepted Invitation's transition (e.g., a duplicate event delivery, or a reload reaching this automation again) | None | Member is routed to the ordinary Weekly Plan View | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |
| Automation failure | A processing error occurs between receiving the acceptance outcome and completing the eligibility check | None -- the completed Member Profile and Invitation Accepted state from FEAT-09.SPEC-007 are unaffected | Member is routed to the ordinary Weekly Plan View as a safe, non-blocking default; onboarding is simply not shown this one time | FEAT-03.SPEC-001, FEAT-23.SPEC-001 |

## Data Model

**Reads:** Member Profile -- member_type, status, display_name (FEAT-15.SPEC-003's eligibility and re-join inputs). Invitation -- status, and whether the invited contact detail maps to a previously Removed or Left Member Profile of this household (re-join detection).
**Creates:** None.
**Updates:** None -- once-only tracking rides on the Invitation's own one-time Accepted transition (owned by FEAT-09), not on a new field this automation writes, keeping the feature within its declared read-only access to both entities (feature-overview.md, Entity-Lifecycle Coverage Matrix).
**Deletes:** None.

## Business Rules

- XBR-18: an accepted invitation triggers first-use onboarding exactly once per accepted invitation; this automation fires at most once per Invitation's Accepted transition, never on a reload or re-entry into an already-processed transition.
- Eligibility and once-only status are governed entirely by FEAT-15.SPEC-003 -- this automation calls that rule rather than re-implementing it.
- A re-invited former member is always routed through onboarding again rather than being silently restored to old data (XBR-18, feature-overview.md Non-Goals).
- This automation runs synchronously as part of the acceptance flow -- the member is routed before FEAT-09.SPEC-002's Accept & Join screen is considered fully resolved.

## Edge Cases

- **Member reloads or re-enters mid-viewing of Onboarding Landing** -- Per FEAT-15.SPEC-003, onboarding was already shown for this transition; this automation is not re-triggered by a reload (its only trigger is FEAT-09.SPEC-007's acceptance outcome, which fires once), and FEAT-15.SPEC-001 itself re-confirms once-only status at render time.
- **Household has no plan yet at routing time** -- Routing still proceeds to FEAT-15.SPEC-001, which renders its own explained empty state; this automation does not branch on plan existence.
- **FEAT-09.SPEC-007's acceptance outcome is a race-lost rejection** (invitation already Accepted, Revoked, or Expired) -- This automation never fires, since its only trigger is specifically FEAT-09.SPEC-007's "Acceptance succeeds" outcome.
- **Concurrent trigger firing** (two different invited adults' acceptances resolve at effectively the same time) -- Each acceptance is a distinct Invitation and Member Profile; this automation runs once per Invitation independently, and neither run reads or affects the other's Member Profile or routing decision.
- **Trigger fires while a previous run is still in flight** -- Cannot occur under ordinary use: FEAT-09.SPEC-010's race resolution ensures only the first acceptance for a given Invitation ever reaches FEAT-09.SPEC-007's "Acceptance succeeds" outcome, so this automation is never started twice for the same transition; a second acceptance attempt for the same Invitation is rejected upstream and never reaches this automation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Triggered by (inbound) | Fires on that automation's "Acceptance succeeds" outcome |
| FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule) | References (outbound) | Eligibility, once-only, and re-join status checked before routing |
| FEAT-15.SPEC-001 (Onboarding Landing) | Affects (outbound) | Routes the eligible, not-yet-shown member here |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Fallback destination when onboarding does not fire, paid tier |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Affects (outbound) | Fallback destination when onboarding does not fire, free tier |

## Analytics and Success Signals

- **member_onboarding_started** (trigger source: invitation acceptance) -- N/A -- no success-metrics.md metric carries `Connected Feature: Member Onboarding`; retained per product-features.md's Signals field as the feature's own funnel-start count.
- **member_onboarding_completed** (elapsed time from started to completed) -- N/A -- same reason; retained as the feature's own funnel-completion count.

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Sam's invitation acceptance succeeds via FEAT-09.SPEC-007, when this automation receives the outcome, then it emits member_onboarding_started and evaluates his eligibility via FEAT-15.SPEC-003.

**FEAT-15.SPEC-002-AC-02:** Given FEAT-15.SPEC-003 confirms Sam is an eligible Other Adult Member with no prior onboarding shown for this transition, when evaluation completes, then Sam is routed to FEAT-15.SPEC-001 (Onboarding Landing).

**FEAT-15.SPEC-002-AC-03:** Given Sam is a previously removed member who has just been re-invited and accepted, when this automation processes his acceptance, then FEAT-15.SPEC-003 flags the re-join and Sam is still routed to FEAT-15.SPEC-001 with no prior data restored.

**FEAT-15.SPEC-002-AC-04:** Given FEAT-15.SPEC-001 successfully renders to Sam for the first time, when the landing's first paint completes, then this automation emits member_onboarding_completed.

**FEAT-15.SPEC-002-AC-05:** Given Sam reloads or re-enters the app after onboarding has already been shown once for his acceptance, when this is evaluated, then FEAT-15.SPEC-003 reports already-shown and Sam is routed to the ordinary Weekly Plan View instead of Onboarding Landing again.

**FEAT-15.SPEC-002-AC-06:** Given the household has no plan yet at the moment Sam is routed, when routing completes, then Sam still lands on FEAT-15.SPEC-001, which renders its own explained empty state.

**FEAT-15.SPEC-002-AC-07:** Given a processing error occurs between receiving the acceptance outcome and completing the eligibility check, when the failure occurs, then Sam is routed to the ordinary Weekly Plan View as a safe default and onboarding is not shown this one time.

**FEAT-15.SPEC-002-AC-08:** Given two different invited adults' acceptances resolve at effectively the same time, when both reach this automation, then each is processed independently against its own Invitation and Member Profile with no interference between the two runs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (eligible first-time, eligible re-join, ineligible role, already shown, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

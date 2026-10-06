---
document_type: spec
spec_type: automation
spec_id: FEAT-07.SPEC-001
spec_name: Plan-Ready Notification Trigger
spec_slug: plan-ready-notification-trigger
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Plan-Ready Notification Trigger

## Overview

**Name:** Plan-Ready Notification Trigger
**ID:** FEAT-07.SPEC-001
**Type:** Automation
**Purpose:** On generation completion, determines who is eligible, resolves each eligible member's delivery channel, and fires the plan-ready message exactly once per household per week.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- Receiving the generation-completion signal from either the recurring scheduled generation or the household's first-plan generation
- Enforcing the once-per-household-per-week firing cap
- Determining each household member's eligibility and resolving their delivery channel
- Dispatching the plan-ready message (FEAT-07.SPEC-002) for every eligible member

**Non-Goals:**
- Generating the plan itself, or deciding when generation runs -- owned by AI Weekly Dinner Plan Generation (FEAT-03.SPEC-003, FEAT-03.SPEC-004); this automation begins only once generation signals completion.
- Defining who is eligible, the once-per-week rule, and the channel-fallback logic in detail -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules), which this automation calls rather than re-implementing.
- The exact message content, audience wording, and tap-through destination -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message); this automation only decides who receives it and on which channel.
- Delivering the message once dispatched -- owned by FEAT-07.SPEC-005 (Device-Notification Delivery Integration) and FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration); this automation hands off the dispatch instruction and does not manage delivery mechanics itself.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Recurring weekly generation completes | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Fires when that automation's "Generation succeeded" or "Generation succeeded, over budget" outcome completes | Household id, new week identifier, generation kind (recurring) |
| First-plan generation on upgrade completes | FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Fires when that automation's "First-plan generation succeeded" outcome completes | Household id, new week identifier, generation kind (first-plan) |

## Processing Logic

1. Receive the generation-completion signal, carrying the household id, the week identifier, and whether this is a recurring or first-plan generation.
2. Check the once-per-household-per-week firing cap via FEAT-07.SPEC-003: has a plan-ready dispatch already been recorded for this household and this week identifier? If yes, stop -- no-action outcome (Edge Cases covers why this can occur).
3. Read the household's Member Profile list and, for each Active adult member (Maya, Sam), read their notification_preferences.plan_ready value via FEAT-07.SPEC-003's eligibility rule. Young-kid and older-kid Member Profile rows are excluded outright per FEAT-07.SPEC-003, never evaluated further.
4. Build the eligible-member list: every Active adult member whose plan_ready preference is on.
5. If the eligible-member list is empty, stop -- no-action outcome; record the firing cap for this household-week regardless, since the generation-completion event itself has now been processed once.
6. For each eligible member, resolve their delivery channel via FEAT-07.SPEC-003's channel-fallback rule: device notification (FEAT-07.SPEC-005) where available and enabled for that member, otherwise email (FEAT-07.SPEC-006).
7. Dispatch the plan-ready message (FEAT-07.SPEC-002) to each eligible member on their resolved channel, carrying the generation kind so the message can apply first-plan or recurring framing.
8. Record the firing cap for this household-week so any later or duplicate completion signal for the same week is treated as no-action (step 2).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Notification dispatched (recurring) | FEAT-03.SPEC-003 signals completion and at least one member is eligible | Firing cap recorded for this household-week | Each eligible member receives FEAT-07.SPEC-002's recurring-framed message on their resolved channel | FEAT-07.SPEC-002, FEAT-07.SPEC-003, FEAT-07.SPEC-005, FEAT-07.SPEC-006 |
| Notification dispatched (first-plan) | FEAT-03.SPEC-004 signals completion and at least one member is eligible | Firing cap recorded for this household-week | Each eligible member receives FEAT-07.SPEC-002's first-plan-framed message on their resolved channel | FEAT-07.SPEC-002, FEAT-07.SPEC-003, FEAT-07.SPEC-005, FEAT-07.SPEC-006 |
| No eligible members | Every Active adult member has plan_ready turned off | Firing cap recorded for this household-week; no dispatch occurs | Nothing is sent; each member simply sees the plan the next time they open the app | FEAT-07.SPEC-003 |
| Duplicate signal suppressed (no-action) | A generation-completion signal arrives for a household-week whose firing cap is already recorded | None | Nothing is sent; the household already received this week's message | FEAT-07.SPEC-003 |
| Dispatch failure | Channel resolution or hand-off to FEAT-07.SPEC-005/006 fails for one or more members after eligibility was determined | No firing-cap change for members whose dispatch could not even be attempted; already-dispatched members are unaffected | No error is shown to the household; the plan remains available in-app immediately, unaffected by the notification path, per XBR-12 | FEAT-07.SPEC-005, FEAT-07.SPEC-006 |

## Data Model

**Reads:** Household -- id (to scope the run); Member Profile -- status, member_type, notification_preferences (plan_ready), for every member of the household; Weekly Plan -- week identifier and generation identity, to key the once-per-week firing cap.
**Creates:** None -- this automation creates no new dependency-map entity; it records a firing-cap marker scoped to this automation's own once-per-week enforcement (FEAT-07.SPEC-003), not a new entity.
**Updates:** None on Household, Member Profile, or Weekly Plan.
**Deletes:** None.

## Business Rules

- XBR-12: The plan-ready message fires at most once per household per week, only on generation completion, at the organiser-chosen arrival day and time; the plan's in-app availability never depends on notification delivery; email is the fallback where device notifications are unavailable.
- XBR-13: Each member controls their own plan-ready preference; neither kid row ever receives this notification.
- This automation never determines eligibility, the once-per-week rule, or the channel-fallback logic itself -- it calls FEAT-07.SPEC-003 for every one of those decisions.
- Because both trigger sources (FEAT-03.SPEC-003, FEAT-03.SPEC-004) fire only on their own generation-succeeded outcomes, this automation never runs against a household with no new Weekly Plan for the week in question.
- The organiser-chosen plan-arrival day and time (FEAT-07.SPEC-004) is not separately checked here: because the recurring trigger source itself fires at that day/time (FEAT-03.SPEC-003), this automation's own firing is already timed correctly by construction.

## Edge Cases

- **Concurrent trigger firing (two generation-completion signals for the same household and week arrive at effectively the same time)** -- The once-per-household-per-week firing cap (step 2) ensures only the first signal processed results in a dispatch; the second is treated as the duplicate-signal-suppressed outcome, and no member ever receives two plan-ready messages for the same week.
- **Trigger fires while a previous run is in flight** -- A second completion signal for the same household arriving while step 3-8 is still processing the first is held until the in-flight run finishes recording its firing cap, then evaluated against that cap (step 2) and suppressed as a duplicate; runs for different households proceed independently and never queue behind each other.
- **A member turns their plan_ready preference on or off between generation completion and this automation's dispatch step** -- Eligibility is evaluated at dispatch time (step 3), not at generation time, so the member's state at the moment this automation actually runs is what governs; a preference flip completed before dispatch takes effect for that week's message.
- **A member is removed from the household between generation completion and dispatch** -- That member is no longer read at step 3 (Member Profile status is no longer Active for the household), so they are silently excluded from the eligible-member list; a former member never receives a plan-ready message.
- **Channel resolution succeeds for some members and fails for others in the same run** -- Each member's dispatch proceeds independently; a failure for one member's channel resolution does not block or delay dispatch to another eligible member in the same household.
- **The scheduled trigger (FEAT-03.SPEC-003) retries after an earlier generation failure and later succeeds** -- The retry's own completion signal is the first successful completion for that week, so it fires this automation normally; the earlier failed attempt never reached this automation, since FEAT-03.SPEC-003's failure outcome does not list this spec among its affected specs.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Recurring weekly generation's completion signal fires this automation |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | First-plan generation's completion signal fires this automation |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | Triggers (outbound) | Eligibility, once-per-week cap, and channel-fallback decisions are all read from this rule |
| FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule) | References (inbound) | The recurring trigger's own timing is already governed by this rule via FEAT-03.SPEC-003 |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Triggers (outbound) | Dispatches the message to every eligible member |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | Triggers (outbound) | Hands off dispatch for members resolved to the device-notification channel |
| FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration) | Triggers (outbound) | Hands off dispatch for members resolved to the email-fallback channel |

## Analytics and Success Signals

- **plan_ready_trigger_processed** (household id, generation_kind: recurring / first_plan, eligible_member_count) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_trigger_skipped_no_eligible_members** (household id, generation_kind) -- N/A -- no Stage 2 metric measures the zero-eligible-household case specifically; retained so a household that never receives this notification is observable rather than silent.
- **plan_ready_trigger_duplicate_suppressed** (household id, week identifier) -- N/A -- no Stage 2 metric tracks duplicate-signal suppression; retained to observe how often the once-per-week guard is exercised.

## Acceptance Criteria

**FEAT-07.SPEC-001-AC-01:** Given Maya's and Sam's household completes its recurring weekly generation (FEAT-03.SPEC-003) and both have plan-ready notifications on, when generation signals completion, then both receive FEAT-07.SPEC-002's recurring-framed message on their resolved channels.

**FEAT-07.SPEC-001-AC-02:** Given a household upgrades and its first-plan generation (FEAT-03.SPEC-004) completes, when completion is signaled, then every eligible member receives FEAT-07.SPEC-002's first-plan-framed message.

**FEAT-07.SPEC-001-AC-03:** Given Sam has turned his plan-ready preference off and Maya has hers on, when generation completes, then only Maya receives the message.

**FEAT-07.SPEC-001-AC-04:** Given both Maya and Sam have turned their plan-ready preference off, when generation completes, then no message is sent to either, and both simply see the new plan the next time they open the app.

**FEAT-07.SPEC-001-AC-05:** Given a duplicate completion signal arrives for a household-week that has already fired its plan-ready message, when the duplicate is processed, then no second message is sent to anyone.

**FEAT-07.SPEC-001-AC-06:** Given Maya has device notifications available and enabled and Sam does not, when generation completes, then Maya's message is dispatched by device notification and Sam's by email, in the same run.

**FEAT-07.SPEC-001-AC-07:** Given Sam turns his plan-ready preference on moments before this automation's dispatch step runs (after generation completed), when dispatch evaluates eligibility, then Sam is included as eligible for that week's message.

**FEAT-07.SPEC-001-AC-08:** Given Sam is removed from the household after generation completes but before this automation dispatches, when dispatch runs, then Sam is not evaluated and receives no message.

**FEAT-07.SPEC-001-AC-09:** Given channel resolution fails for Sam but succeeds for Maya in the same run, when dispatch completes, then Maya still receives her message and the plan remains fully available in-app for both, with no error shown to the household.

**FEAT-07.SPEC-001-AC-10:** Given two completion signals for the same household and week arrive at effectively the same time, when both are processed, then only the first results in a dispatch and the second is suppressed as a duplicate.

**FEAT-07.SPEC-001-AC-11:** Given a second completion signal for the same household arrives while an in-flight run for that household is still dispatching, when the in-flight run finishes recording its firing cap, then the second signal is evaluated against that cap and suppressed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (recurring, first-plan) | 2 |
| Outcome Paths | 5 (dispatched recurring, dispatched first-plan, no eligible members, duplicate suppressed, dispatch failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

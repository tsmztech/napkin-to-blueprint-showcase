---
document_type: spec
spec_type: automation
spec_id: FEAT-13.SPEC-001
spec_name: Tonight's Nudge Trigger
spec_slug: tonights-nudge-trigger
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Tonight's Nudge Trigger

## Overview

**Name:** Tonight's Nudge Trigger
**ID:** FEAT-13.SPEC-001
**Type:** Automation
**Purpose:** At a sensible pre-dinner time each day, determines whether tonight's dinner exists, resolves eligible household members, and fires the "Tonight: ..." nudge exactly once per household per day.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- Firing once per household per day, at a single platform-set pre-dinner time (platform parameter: `nightly-nudge-send-time`)
- Checking whether a Planned Meal exists for the household's current night before doing anything else
- Resolving the eligible-member list and dispatching the nudge to each eligible member
- Recording which members actually received today's dispatch, so a same-day correction (FEAT-13.SPEC-003) knows who is owed one

**Non-Goals:**
- Defining eligibility, the once-per-day firing cap, the channel-fallback rule, and the delivery-failure-never-blocks guarantee in detail -- owned by FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules), which this automation calls rather than re-implementing.
- Deriving the prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this automation only reads its output.
- The exact message content, channels, and delivery mechanics -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) and the shared device-notification boundary (FEAT-07.SPEC-005); this automation only decides who receives the nudge and hands off the dispatch instruction.
- Deciding when a same-day swap requires a follow-up -- owned by FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger), a separate automation that reads this spec's dispatched-members record rather than being triggered by this spec directly.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Daily nudge time reached | System (daily schedule) | Fires once per household per day at platform parameter: `nightly-nudge-send-time` | Household id, today's date |

## Processing Logic

1. At platform parameter: `nightly-nudge-send-time` each day, evaluate every household independently.
2. Check the once-per-household-per-day firing cap via FEAT-13.SPEC-005: has a nudge dispatch already been recorded for this household and today's date? If yes, stop -- no-action outcome.
3. Read whether a Planned Meal exists for tonight (today's date, meal_kind: dinner) whose status is not Removed -- the meal comes from either AI Weekly Dinner Plan Generation (FEAT-03) or Manual Weekly Planning (FEAT-23), whichever produced the household's current Weekly Plan.
4. If no such Planned Meal exists, stop -- no-action outcome; record the firing cap for this household-date regardless, since today's evaluation has now been processed once.
5. Read the eligible-member list via FEAT-13.SPEC-005: every Active adult member (Maya, Sam) whose notification_preferences.nightly_nudge is on.
6. If the eligible-member list is empty, stop -- no-action outcome; record the firing cap for this household-date.
7. Read the prep-reminder text (or its confirmed absence) for tonight's Planned Meal via FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule).
8. For each eligible member, dispatch the nudge message (FEAT-13.SPEC-002) carrying the meal name and the prep-reminder text (or its absence), on the channel FEAT-13.SPEC-005 resolves for that member.
9. Record the firing cap for this household-date, together with exactly which members received today's dispatch -- this dispatched-members record is what FEAT-13.SPEC-003's correction trigger reads to decide whether a same-day swap owes a correction, and to whom.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Nudge dispatched | A Planned Meal exists for tonight and at least one member is eligible | Firing cap recorded for this household-date; the set of members who received today's dispatch is recorded | Each eligible member receives FEAT-13.SPEC-002's message on their resolved surface | FEAT-13.SPEC-002, FEAT-13.SPEC-005 |
| No dinner planned tonight | No Planned Meal exists for tonight | Firing cap recorded for this household-date; no members recorded as dispatched | Nothing is sent -- a nudge only fires when a dinner exists for that day | FEAT-13.SPEC-005 |
| No eligible members | A Planned Meal exists but every adult member's nightly_nudge preference is off | Firing cap recorded; no members recorded as dispatched | Nothing is sent to anyone | FEAT-13.SPEC-005 |
| Duplicate signal suppressed (no-action) | The firing cap for this household-date is already recorded | None | Nothing changes; the household already received (or was evaluated for) today's nudge | FEAT-13.SPEC-005 |
| Dispatch failure | Dispatch to one or more eligible members fails after eligibility was determined | The firing cap and dispatched-members record reflect only members whose dispatch was actually attempted; already-dispatched members are unaffected | No error is shown; the Planned Meal remains visible in-app immediately, unaffected by the nudge path | FEAT-13.SPEC-002 |

## Data Model

**Reads:** Household -- id (to scope the run); Planned Meal -- night, meal_kind, status, recipe, for the household's current day; Member Profile -- status, member_type, notification_preferences.nightly_nudge, for every member of the household; Recipe -- prep_requirements, via FEAT-13.SPEC-006, for tonight's Planned Meal's recipe.
**Creates:** None -- this automation creates no new dependency-map entity; it records a firing-cap and dispatched-members marker scoped to its own once-per-day enforcement (read by FEAT-13.SPEC-005 and FEAT-13.SPEC-003), not a new entity.
**Updates:** None on Household, Planned Meal, Member Profile, or Recipe.
**Deletes:** None.

## Business Rules

- At most one nudge per household per day, tied to that day's Planned Meal (Brief, Validation & Limits).
- This automation never determines eligibility, the once-per-day cap, or the channel-fallback logic itself -- it calls FEAT-13.SPEC-005 for every one of those decisions.
- This automation never derives the prep-reminder text itself -- it calls FEAT-13.SPEC-006.
- A failed dispatch never blocks anything else: the Planned Meal's in-app visibility is set entirely by FEAT-03 or FEAT-23 and is never gated by this automation's outcome (FEAT-13.SPEC-005).
- The dispatched-members record this automation writes each day is exactly what FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) reads to decide whether a same-day swap owes a correction, and to whom.

## Edge Cases

- **Concurrent trigger firing (two daily-schedule evaluations for the same household and date arrive at effectively the same time)** -- The once-per-household-per-day firing cap ensures only the first evaluation processed results in a dispatch; the second is treated as the duplicate-signal-suppressed outcome, and no member ever receives two nudges for the same night.
- **Trigger fires while a previous run is in flight** -- A second evaluation for the same household arriving while an earlier one is still processing (steps 3-9) is held until that in-flight run finishes recording its firing cap, then evaluated against that cap and suppressed as a duplicate; runs for different households proceed independently and never queue behind each other.
- **Planned Meal is removed (safety concern, FEAT-02) between the existence check (step 3) and dispatch (step 8)** -- The dispatch is cancelled for that household-date; the run is treated as the "No dinner planned tonight" outcome, and no stale nudge naming a removed meal is ever sent.
- **A household's members span no more than one time zone** -- Out of scope for reconciliation: the product defines one household per account (ASMP-16) and one shared "today," so no cross-time-zone household exists whose members could disagree on when "tonight" is.
- **A new adult member is added to the household after today's firing cap is already recorded** -- That member is not evaluated for today, since today's dispatched-members record is not reopened once recorded; they are included starting with tomorrow's cycle.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (outbound) | Eligibility, once-per-day cap, and channel-fallback decisions are all read from this rule |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (outbound) | Prep-reminder text for tonight's Planned Meal is read from this rule |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Triggers (outbound) | Dispatches the message to every eligible member |
| FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) | Affects (outbound) | The dispatched-members record this automation writes is what that trigger reads to decide correction eligibility |
| FEAT-03 (AI Weekly Dinner Plan Generation) | References (outbound) | Reads whether an AI-generated Planned Meal exists for tonight |
| FEAT-23 (Manual Weekly Planning) | References (outbound) | Reads whether a manually picked Planned Meal exists for tonight |
| FEAT-01 (Household Setup & Member Profiles) | References (outbound) | Reads each adult's Member Profile and nightly-nudge preference |

## Analytics and Success Signals

- **dinner_nudge_sent** (household id, eligible_member_count, dispatched_member_count) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field for this feature (dinner_nudge_sent) so the nudge's reach is observable even without a Stage 2 metric attached. This feature's real contribution -- nudged dinners actually getting cooked -- is measured indirectly by the Reported Food Waste and Spend Reduction metric (Connected Feature: Weekly Waste & Spend Check-In), which this feature cannot cite directly since it does not own that measurement.
- **dinner_nudge_skipped_no_dinner** (household id) -- N/A -- same reason; observes how often a household has nothing planned at nudge time.
- **dinner_nudge_skipped_no_eligible_members** (household id) -- N/A -- same reason; observes how often every adult has the nudge turned off.
- **dinner_nudge_dispatch_failed** (household id, affected_member_count) -- N/A -- same reason; retained to observe how often the non-blocking delivery-failure guarantee is exercised.

## Acceptance Criteria

**FEAT-13.SPEC-001-AC-01:** Given Maya's and Sam's household has a Planned Meal for tonight and both have their nightly nudge on, when the daily nudge time is reached, then both receive FEAT-13.SPEC-002's message on their resolved surfaces.

**FEAT-13.SPEC-001-AC-02:** Given the household has no Planned Meal for tonight, when the daily nudge time is reached, then no nudge is dispatched to anyone.

**FEAT-13.SPEC-001-AC-03:** Given Sam has turned his nightly nudge off and Maya has hers on, when the nudge time is reached, then only Maya receives the message.

**FEAT-13.SPEC-001-AC-04:** Given both Maya and Sam have turned their nightly nudge off, when the nudge time is reached, then no message is sent to either, and both simply see tonight's dinner the next time they open the app.

**FEAT-13.SPEC-001-AC-05:** Given a duplicate daily-schedule evaluation arrives for a household-date whose firing cap is already recorded, when it is processed, then no second nudge is sent to anyone.

**FEAT-13.SPEC-001-AC-06:** Given dispatch fails for Sam but succeeds for Maya in the same run, when the run completes, then Maya still receives her nudge and tonight's dinner remains fully visible in-app for both, with no error shown to the household.

**FEAT-13.SPEC-001-AC-07:** Given two daily-schedule evaluations for the same household and date arrive at effectively the same time, when both are processed, then only the first results in a dispatch and the second is suppressed as a duplicate.

**FEAT-13.SPEC-001-AC-08:** Given a safety-concern removal (FEAT-02) takes tonight's Planned Meal off the plan between the existence check and dispatch, when this automation reaches its dispatch step, then no nudge is sent and the run is treated as "no dinner planned tonight."

**FEAT-13.SPEC-001-AC-09:** Given a new adult member joins the household after today's firing cap has already been recorded, when today's cap is checked again, then that member is not evaluated today and is included starting with tomorrow's cycle.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (daily schedule) | 1 |
| Outcome Paths | 5 (dispatched, no dinner, no eligible members, duplicate suppressed, dispatch failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |

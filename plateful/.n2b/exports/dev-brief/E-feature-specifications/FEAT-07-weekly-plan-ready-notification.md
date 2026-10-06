# FEAT-07 — Weekly Plan Ready Notification

This chapter covers FEAT-07, Weekly Plan Ready Notification, a Core-tier feature. It contains 6 specifications carrying 71 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-07.SPEC-001 | Plan-Ready Notification Trigger | automation | 11 |
| FEAT-07.SPEC-002 | Plan-Ready Notification Message | notification | 13 |
| FEAT-07.SPEC-003 | Plan-Ready Delivery & Eligibility Rules | logic-rule | 14 |
| FEAT-07.SPEC-004 | Plan-Arrival Day & Time Setting Rule | logic-rule | 10 |
| FEAT-07.SPEC-005 | Device-Notification Delivery Integration | integration | 12 |
| FEAT-07.SPEC-006 | Plan-Ready Email Fallback Integration | integration | 11 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Weekly Plan Ready Notification

## Summary

**Feature:** Weekly Plan Ready Notification
**ID:** FEAT-07
**Description:** The household is told, on a predictable schedule, that next week's plan is ready to review.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** The brief's own description of the core experience begins here: "It's Sunday evening and a notification arrives: 'Next week's plan is ready.'" (BRIEF.md, The Experience). Without this, the weekly plan is invisible until someone happens to check.

**Key Capabilities:**
- Notify when the plan is ready — Household members with notifications enabled are told as soon as generation completes
- Control who gets notified — Each household member can enable or disable this notification for themselves
- Choose when the plan arrives — The organiser picks the day and rough time the weekly plan arrives (Sunday evening by default)

This feature is entirely a background/system-message feature: it defines no screens of its own. Its two configurable settings (the per-member on/off preference and the organiser's plan-arrival day/time) are edited through screens that Household Setup & Member Profiles (FEAT-01) owns; this Brief defines the rules, automation, message, and delivery integrations that make the notification happen, and cross-references FEAT-01's screens rather than duplicating them.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-07.SPEC-001 | Plan-Ready Notification Trigger | Automation | Maya, Sam | On generation completion, determines who is eligible, resolves each eligible member's delivery channel, and fires the plan-ready message exactly once per household per week |
| FEAT-07.SPEC-002 | Plan-Ready Notification Message | Notification | Maya, Sam | The "Next week's plan is ready" message itself — content, audience, and tap-through behavior, delivered by device notification or, per member, by email |
| FEAT-07.SPEC-003 | Plan-Ready Delivery & Eligibility Rules | Logic/Rule | All | Governs who is ever eligible to receive the message, the once-per-week constraint, the device-vs-email channel fallback, and the guarantee that delivery never blocks in-app plan availability |
| FEAT-07.SPEC-004 | Plan-Arrival Day & Time Setting Rule | Logic/Rule | Maya | Governs the allowed values, default, and storage of the organiser-chosen plan-arrival day and time |
| FEAT-07.SPEC-005 | Device-Notification Delivery Integration | Integration | Maya, Sam | Product boundary to the device-notification delivery capability, including offline queuing and redelivery on reconnect |
| FEAT-07.SPEC-006 | Plan-Ready Email Fallback Integration | Integration | Maya, Sam | Product boundary to the transactional email capability for the plan-ready fallback route, used per member when device notifications are unavailable or not enabled |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Notify when the plan is ready | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-005, FEAT-07.SPEC-006 | Trigger fires on generation completion; the message is delivered by device notification or its email fallback | Phase 2 (Explicit) |
| Control who gets notified | FEAT-07.SPEC-003; screen: FEAT-01.SPEC-005 (Member Profile Detail) | Each member's own preference (stored on their Member Profile, edited in FEAT-01's screens) is read by the eligibility rule at trigger time | Phase 2 (Explicit) |
| Choose when the plan arrives | FEAT-07.SPEC-004; screen: FEAT-01.SPEC-010 (Household Settings Hub) | The organiser sets the arrival day/time in FEAT-01's settings screen; this feature's rule defines the allowed values, default, and how SPEC-001 reads it | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-07.SPEC-003 | Plan-Ready Delivery & Eligibility Rules | Phase 5 (Rule-Constraint Discovery — authorization + conditional logic) | Five or more interacting conditions govern who receives the message and by which channel (per-member preference, device-notification availability, email opt-out, kid/Riley/unauthorized exclusion, the once-per-week cap) — well past the standalone-spec threshold, and shared by SPEC-001 and SPEC-002 rather than duplicated |
| FEAT-07.SPEC-005 | Device-Notification Delivery Integration | Phase 4 (External Dependencies lens) | assumptions-constraints.md's Dependencies section (ASMP-31) names device-notification delivery as required for this feature; the External Touchpoints slice lists this row as pending on FEAT-07's own validated Brief, since FEAT-04/FEAT-13/FEAT-23 rely on the capability only through this feature |
| FEAT-07.SPEC-006 | Plan-Ready Email Fallback Integration | Phase 4 (External Dependencies lens + Notification surfacing) | The Communications field states the message "goes by transactional email instead" when device notifications are unavailable; assumptions-constraints.md's Dependencies section (ASMP-32) names transactional email as required here, distinct from FEAT-01.SPEC-017's account/recovery email route |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity through the full create/read/update/delete lifecycle. Its one Connected Entity, Weekly Plan, is read-only for this feature (product-features.md Connected Entities: "Weekly Plan (read)") — its creation, update, and deletion/archival are owned entirely by other features (FEAT-03, FEAT-23, FEAT-04, FEAT-18). Per Phase 3's rule for read-only Connected Entities, Weekly Plan is carried below as a Referenced Entity rather than given a full CRUD matrix. Household and Member Profile carry fields this feature's rules govern the *values* of (plan-arrival day/time; notification preference) but whose create/update screens live in FEAT-01 — they are also listed as Referenced Entities, with a note on which side owns the screen versus the rule.

**Referenced Entities (read-only for this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-07.SPEC-001 | The trigger reads the plan's generation-completion event (week + generation identity) to fire the message and to enforce the once-per-household-per-week cap; it does not read or display plan contents |
| Household | FEAT-07.SPEC-001, FEAT-07.SPEC-004 | Reads `plan_arrival_day_time` to know when to fire the message; the value itself is written through FEAT-01.SPEC-010 (Household Settings Hub), but FEAT-07.SPEC-004 owns the rule for its allowed values and default |
| Member Profile | FEAT-07.SPEC-001, FEAT-07.SPEC-003 | Reads each member's `notification_preferences` (plan-ready on/off) to determine eligibility; the value itself is written through FEAT-01.SPEC-005 (Member Profile Detail), per XBR-13 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| AI Weekly Dinner Plan Generation completes (FEAT-03) | Determine eligible members, resolve each one's channel, and fire the plan-ready message at the organiser-chosen arrival day/time | Standalone Automation | FEAT-07.SPEC-001 |
| Plan-ready trigger fires for an eligible member | Send "Next week's plan is ready" by device notification, or by email where device notifications are unavailable or disabled | Standalone Notification | FEAT-07.SPEC-002 |
| Plan-ready message needs to reach a device | Deliver through the device-notification delivery capability; queue and redeliver once an offline device reconnects | Standalone Integration | FEAT-07.SPEC-005 |
| Plan-ready message needs to reach a member without device notifications enabled | Deliver through the transactional email capability as the fallback route | Standalone Integration | FEAT-07.SPEC-006 |
| Household member taps the plan-ready notification | Open directly on the new week's plan | Cross-feature — owned by AI Weekly Dinner Plan Generation | FEAT-03.SPEC-001 responsibility, referenced by FEAT-07.SPEC-002 |
| Device-notification or email delivery fails or is delayed | Plan remains available in-app immediately upon generation, unaffected by the notification path | Standalone Logic/Rule | FEAT-07.SPEC-003 |
| A household member has disabled the plan-ready preference | No notification is sent to them; they see the plan the next time they open the app | Standalone Logic/Rule | FEAT-07.SPEC-003 |
| Organiser changes the plan-arrival day or time | Household's `plan_arrival_day_time` is updated; the next trigger fires at the new day/time | Inline in FEAT-01.SPEC-010 (Household Settings Hub), governed by | FEAT-07.SPEC-004 |

## Shared Context

**Shared Entities:**
- Weekly Plan — read only, by FEAT-07.SPEC-001, to detect generation completion and enforce the once-per-week cap. No fields are displayed or derived by this feature.
- Household — `plan_arrival_day_time` field read by FEAT-07.SPEC-001 and defined (allowed values, default) by FEAT-07.SPEC-004; written through FEAT-01.SPEC-010.
- Member Profile — `notification_preferences` field (plan-ready on/off) read by FEAT-07.SPEC-001 and FEAT-07.SPEC-003; written through FEAT-01.SPEC-005, per XBR-13.

**Shared UI Patterns:**
- N/A — this feature defines no screens of its own. Both of its configurable settings are edited entirely within FEAT-01's existing screens (Member Profile Detail for the per-member preference, Household Settings Hub for the arrival day/time); this Brief cross-references those screens rather than duplicating their UI.

**Shared Validation:**
- FEAT-07.SPEC-003 defines the eligibility, once-per-week, and channel-fallback rules; FEAT-07.SPEC-001 (the trigger) and FEAT-07.SPEC-002 (the message) both reference SPEC-003 rather than restating the logic.
- FEAT-07.SPEC-004 defines the arrival-day/time value rules; FEAT-07.SPEC-001 references it to know when to fire.

## Internal Dependency Map

```
FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) -> [reads eligibility from] -> FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules)
FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) -> [reads arrival day/time from] -> FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule)
FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) -> [fires] -> FEAT-07.SPEC-002 (Plan-Ready Notification Message)
FEAT-07.SPEC-002 (Plan-Ready Notification Message) -> [device channel] -> FEAT-07.SPEC-005 (Device-Notification Delivery Integration)
FEAT-07.SPEC-002 (Plan-Ready Notification Message) -> [email fallback channel] -> FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration)
FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) -> [selects channel per member, governs] -> FEAT-07.SPEC-005 (Device-Notification Delivery Integration)
FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) -> [selects channel per member, governs] -> FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration)
```

**Default Entry:** N/A — this feature has no screen a household member navigates to. Its only user-visible surface is the message itself (FEAT-07.SPEC-002), which delivers the household directly into FEAT-03.SPEC-001 (Weekly Plan View) when tapped or opened.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-07.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Generation completion is the sole trigger for the plan-ready automation | AI Weekly Dinner Plan Generation finishes successfully |
| FEAT-07.SPEC-002 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Tapping or opening the message lands the member directly on the new week's plan (FEAT-03.SPEC-001) | Household member taps the notification or opens the fallback email |
| FEAT-07.SPEC-001, FEAT-07.SPEC-003 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Reads each member's plan-ready notification preference from their Member Profile | Trigger runs at generation completion |
| FEAT-07.SPEC-004 | Inbound | FEAT-01 (Household Setup & Member Profiles) | The plan-arrival day/time is set through FEAT-01.SPEC-010 (Household Settings Hub); FEAT-01.SPEC-005 hosts the per-member preference toggle | Organiser opens notification settings from the Settings Hub |
| FEAT-07.SPEC-005 | Outbound | FEAT-04 (One-Tap Meal Swap), FEAT-13 (Tonight's Dinner Nudge), FEAT-23 (Manual Weekly Planning) | These features' own notifications (swap-suggestion alerts, the nightly nudge, manual-planning prompts) rely on the same device-notification delivery capability this Integration spec owns | Each feature's own notification-worthy event fires |
| FEAT-07.SPEC-006 | Cross-reference | FEAT-01 (Household Setup & Member Profiles, FEAT-01.SPEC-017) | FEAT-01.SPEC-017 owns account-creation and sign-in-recovery email on the same transactional email capability; FEAT-07.SPEC-006 owns only the plan-ready fallback route and does not duplicate account/recovery email behavior | N/A — a standing capability-ownership boundary, not an event |

## Non-Functional Notes

**Data volumes / growth:** At most one plan-ready message per household per week, tied to generation completion rather than user action (Validation & Limits). Volume scales with household count (several thousand households in the first year, ASMP-24) and each household's 2-6 enabled members, not with any per-feature data growth of its own — this feature stores no growing dataset.

**Responsiveness:** At least 80% of enabled plan-ready messages (device notification, or email where device notifications are unavailable) are delivered within one minute of plan generation completing, and at least half are opened within the same day (success-metrics.md, Weekly Plan Ready Notification Reach).

**Data sensitivity / privacy:** The message itself carries no meal, dietary, or budget content — only "Next week's plan is ready" — so it is low-sensitivity compared with the household data it points to. It is never sent to a kid profile of either row, to Riley, or to an unauthorized visitor (Access field); this keeps children's data out of this feature entirely, consistent with ASMP-26's privacy posture. Delivery is never sold or used for advertising (ASMP-14, ASMP-26).

**Compliance flags:** N/A — this feature carries no health or financial data, and its Access rules exclude both kid rows outright, so no children's-privacy-class data ever reaches this feature's delivery path (ASMP-26).

## Non-Goals

- **Independent kid access to the plan-ready notification** — Excluded per scope-boundaries.md (SC-02): v1's default is parent-managed profiles with no login for young kids, and the Access Matrix gives both the young-kid (no-login) and older-kid (Later, limited login) rows `None` on Notification Prefs. Neither kid row will ever receive this notification, in v1 or in the Later-phase login.
- **A native mobile push channel** — Excluded per scope-boundaries.md (SC-05): the platform is a responsive web app for v1 with no native apps and no app stores, so device-notification delivery (FEAT-07.SPEC-005) is scoped to what a web app can deliver, not a native-app push channel.
- **Configurable notification frequency or custom message content** — Excluded per the feature's own Validation & Limits field, which caps this feature at exactly one message per household per week, tied to generation completion and not to user action; the product does not offer additional reminder cadences or an editable message beyond "Next week's plan is ready" in v1.
- **In-product messaging as the delivery channel** — Excluded per scope-boundaries.md (SC-14): the brief names group chat as the coordination problem this product replaces, not a channel to rebuild; the plan-ready message travels by device notification and email fallback only, never by an in-app chat or messaging surface.



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



# Notification Spec: Plan-Ready Notification Message

## Overview

**Name:** Plan-Ready Notification Message
**ID:** FEAT-07.SPEC-002
**Type:** Notification
**Purpose:** Tells each eligible household member that next week's plan is ready, by device notification or email, so the weekly plan is never invisible until someone happens to check.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- The recurring-week and first-plan variants of the "plan is ready" message, on both its channels (device notification and email)
- Preference, deduplication, retry, and expiry behavior for this message
- The tap-through/CTA behavior that lands the member on the new week's plan

**Non-Goals:**
- Deciding who is eligible, the once-per-week cap, and which channel a given member resolves to -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec begins once FEAT-07.SPEC-001 hands it a resolved recipient and channel.
- The mechanics of delivering a device notification or a transactional email -- owned by FEAT-07.SPEC-005 (Device-Notification Delivery Integration) and FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration); this spec defines the message content and delivery rules those integrations carry out.
- Configurable notification frequency or an editable message beyond "Next week's plan is ready" -- excluded per product-features.md's Validation & Limits field for this feature, which caps this feature at exactly one message per household per week tied to generation completion, not user action.
- In-product messaging or chat as a delivery route -- excluded per scope-boundaries.md SC-14: this message travels by device notification and email fallback only, never through an in-app chat or messaging surface.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | The eligible member has device notifications available and enabled, per FEAT-07.SPEC-003's channel-fallback rule | Maya's and Sam's main touchpoint for this feature is exactly this moment -- a device notification arriving so they can open the plan from wherever they are (user-persona.md, Behavioral Context) |
| Email | The eligible member does not have device notifications available or enabled, or the device-notification delivery capability itself is unavailable that week, per FEAT-07.SPEC-003 | Device notifications from a responsive web app are not available on every phone; email keeps the Sunday rhythm the product promises reaching the member anyway (product-features.md, Communications) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Eligible member resolved for dispatch | FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Fires once per eligible member, per household, per week, after eligibility and channel resolution complete | Recipient Member Profile (display_name, sign_in email), household id, week identifier, generation kind (recurring / first-plan), resolved channel |

## Audience and Preferences

**Recipients:** Maya (Organiser) and Sam (Other Adult Member) -- the two roles with a Notification Prefs entry in the Access Matrix. Neither kid row ever receives this message: the young-kid profile has no login at all, and the Later-phase older-kid login carries no Notification Prefs entitlement (Access Matrix, Notification Prefs column: None for both kid rows). Riley (Operator, support) has no Notification Prefs access and never receives this or any other notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-ready notifications | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail) -- each adult controls only their own toggle |

**Quiet Hours:** N/A -- Plateful defines no quiet-hours window for any notification (no ASMP, XBR, or Communications field establishes one anywhere in the product). This message additionally fires at most once per week, at the organiser-chosen plan-arrival day and time (FEAT-07.SPEC-004) -- a moment the organiser has already deliberately chosen as convenient, so layering a further quiet-hours hold on top of a time the household picked for itself would work against that choice.

## Content Definition

**Push -- recurring week:**
- **Title:** Next week's plan is ready
- **Body:** Tap to see next week's dinners.
- **CTA:** Opens the new week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Push -- first-plan (household's first-ever generated plan):**
- **Title:** Your first week's plan is ready
- **Body:** Tap to see your first week of dinners.
- **CTA:** Opens the new week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Email -- recurring week:**
- **Subject:** Next week's plan is ready
- **Body:**
  Hi {member_first_name},

  Next week's plan is ready to review.

  Open Plateful to see the week's dinners.
- **CTA (button):** View this week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Email -- first-plan:**
- **Subject:** Your first week's plan is ready
- **Body:**
  Hi {member_first_name},

  Your household's first week of dinners is ready to review.

  Open Plateful to see the plan.
- **CTA (button):** View your plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {member_first_name} | Member Profile -- display_name | Sam | Greeting renders as "Hi there," -- display_name is required for every Member Profile (FEAT-01), so this fallback is never expected to trigger in practice, but is defined for completeness |

## Delivery Rules

**Batching:** N/A -- at most one plan-ready message ever exists per household per week (XBR-12), and each eligible member receives exactly one instance of it; no scenario produces two pending instances for the same member to collapse together.
**Deduplication:** At most one message per member per household-week, guaranteed by FEAT-07.SPEC-001's once-per-household-per-week firing cap (enforced via FEAT-07.SPEC-003). A duplicate or retried generation-completion signal for the same week never produces a second dispatch to any member.
**Retry on failure:** Push -- delivery to a device that is offline is queued and redelivered automatically once the device reconnects, per FEAT-07.SPEC-005's delivery contract; this is queuing, not a failure retry, since XBR-12 requires the plan's in-app availability to never depend on it. If the resolved channel cannot reach the member at all for a reason other than being temporarily offline (device unregistered, permission revoked, or the device-notification delivery capability itself down), no push retry is attempted -- FEAT-07.SPEC-003's channel-fallback rule already routes that member to Email instead. Email -- delivery failure is retried up to 3 times over 6 hours (FEAT-07.SPEC-006's transactional email delivery contract). After the final email failure, no further channel is attempted for that member that week; the household still sees the plan in-app immediately regardless (XBR-12), and the failure is never surfaced to the household as an error.
**Expiry:** A push notification queued for an offline device is delivered whenever the device reconnects, with no separate time cutoff, up until the following week's plan-ready message becomes due; if the device has not reconnected by then, the stale prior-week instance is discarded rather than delivered alongside the new week's message, since only the current week's message is ever meaningful (deduplication takes precedence). Email carries no separate expiry beyond its retry window -- a send that eventually succeeds within the 6-hour retry window still delivers an accurate message, since "next week's plan is ready" remains true for the whole week it names.

## Edge Cases

- **The device-notification delivery capability is fully unavailable when this message dispatches** -- Per ASMP-31, delivery degrades to email: FEAT-07.SPEC-003 treats every affected member as "device notifications unavailable" for that week, and Email carries the message for them instead of Push.
- **A member turns their plan-ready preference off after FEAT-07.SPEC-001 has already resolved them as eligible but before this message actually sends** -- FEAT-07.SPEC-001's eligibility check runs at dispatch time, not at generation time, so a preference turned off before dispatch means no message reaches that member this week; a preference turned off after dispatch has already sent has no effect on the message already delivered.
- **The household is deleted between generation completion and this message's dispatch** -- The dispatch is cancelled silently on every channel for every member; a household that no longer exists is never notified about a plan.
- **Both Push and Email fail for the same member in the same week** -- The plan remains fully visible in-app immediately, unaffected by the notification path (XBR-12); no further channel is attempted and no error is shown to the household, since this notification's own failure must never surface as a product error.
- **A member's device reconnects only after the following week's plan-ready message has already been dispatched** -- Per Expiry, the stale queued prior-week push is discarded rather than delivered; the member instead receives (or has already received) the current week's message through its own dispatch.
- **A generation retry (FEAT-03.SPEC-003's "Retry succeeds" outcome) completes for the same week that already fired this message** -- Deduplication (via FEAT-07.SPEC-001's firing cap) ensures the retry's completion signal never triggers a second message for that week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggered by (inbound) | Resolves the eligible recipient list and each member's channel, then dispatches this message |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Eligibility, once-per-week cap, and channel-fallback rules this message's dispatch depends on |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Hosts the plan-ready preference toggle this message's audience depends on |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | References (outbound) | Carries out Push delivery, offline queuing, and reconnect redelivery for this message |
| FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration) | References (outbound) | Carries out Email delivery and retry for this message |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | Every CTA on every variant deep-links here |
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | References (inbound) | Its completion, via FEAT-07.SPEC-001, is the ultimate source of the recurring-week variant |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | References (inbound) | Its completion, via FEAT-07.SPEC-001, is the ultimate source of the first-plan variant |

## Analytics and Success Signals

- **plan_ready_notification_sent** (channel: push / email; generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_notification_opened** (channel: push / email) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_email_sent** (generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach" (the email-channel breakdown of plan_ready_notification_sent, tracked separately per product-features.md's own Signals field for this feature)
- **plan_ready_notification_delivery_degraded** (channel; reason: capability_down / device_unreachable) -- N/A -- no Stage 2 metric measures delivery degradation frequency specifically; retained so silent delivery loss on this milestone message is observable rather than invisible.

## Acceptance Criteria

**FEAT-07.SPEC-002-AC-01:** Given Maya has device notifications enabled and default preferences, when FEAT-07.SPEC-001 resolves her as eligible for the recurring week, then she receives a Push notification titled "Next week's plan is ready."

**FEAT-07.SPEC-002-AC-02:** Given Sam does not have device notifications available, when FEAT-07.SPEC-001 resolves him as eligible for the recurring week, then he receives an email with the subject "Next week's plan is ready" instead of a push notification.

**FEAT-07.SPEC-002-AC-03:** Given a household's first-ever plan generation completes, when eligible members are dispatched, then they receive the first-plan-framed variant ("Your first week's plan is ready"), not the recurring variant.

**FEAT-07.SPEC-002-AC-04:** Given Maya taps the Push notification, when it opens, then she lands directly on the new week's plan (FEAT-03.SPEC-001).

**FEAT-07.SPEC-002-AC-05:** Given Sam taps the email's "View this week's plan" button, when it opens, then he lands directly on the new week's plan (FEAT-03.SPEC-001).

**FEAT-07.SPEC-002-AC-06:** Given Sam has turned his plan-ready preference off, when generation completes, then he receives no message on any channel, per FEAT-07.SPEC-001's eligibility check.

**FEAT-07.SPEC-002-AC-07:** Given the device-notification delivery capability is unavailable this week, when Maya would otherwise receive a Push message, then she receives the Email variant instead.

**FEAT-07.SPEC-002-AC-08:** Given a duplicate generation-completion signal arrives for a household-week that already dispatched this message, when the duplicate is processed, then no second message reaches any member.

**FEAT-07.SPEC-002-AC-09:** Given Maya's device is offline when the Push message is queued, when her device reconnects later the same week, then the queued message is delivered at that point.

**FEAT-07.SPEC-002-AC-10:** Given an email send to Sam fails transiently, when it is retried within the 6-hour window and the retry succeeds, then only one email reaches him, and no error appears to the household in the meantime.

**FEAT-07.SPEC-002-AC-11:** Given both Push and Email fail for Sam in the same week, when the final failure occurs, then no error is shown to the household and the plan remains fully visible to Sam in-app.

**FEAT-07.SPEC-002-AC-12:** Given Maya's device has not reconnected by the time the following week's plan-ready message becomes due, when that following week's message dispatches, then the stale prior-week queued push is discarded rather than delivered alongside it.

**FEAT-07.SPEC-002-AC-13:** Given Maya or Sam looks for a quiet-hours setting for this message, when they check their notification preferences (FEAT-01.SPEC-005), then no such control exists, consistent with the product defining no quiet hours for any notification.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, email) | 2 |
| Trigger Paths | 1 (resolved eligible member from FEAT-07.SPEC-001, with recurring and first-plan variants) | 1 |
| Preference States | 3 (on/push, on/email-fallback, off) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Plan-Ready Delivery & Eligibility Rules

## Overview

**Name:** Plan-Ready Delivery & Eligibility Rules
**ID:** FEAT-07.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs who is ever eligible to receive the plan-ready message, the once-per-week firing constraint, the device-vs-email channel fallback, and the guarantee that delivery never blocks in-app plan availability.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification
**Governed Entity:** Member Profile (plan-ready delivery-eligibility state), with a household-week firing cap tied to Weekly Plan generation identity

## Scope and Non-Goals

**In Scope:**
- Eligibility rules: which roles and member states can ever receive this message
- The once-per-household-per-week firing cap
- The device-vs-email channel-fallback decision for each eligible member
- The guarantee that notification delivery outcomes never affect the plan's in-app availability

**Non-Goals:**
- The plan-arrival day and time itself -- owned by FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule); this spec only reads that value's existence as context for FEAT-07.SPEC-001's timing, it does not define its allowed values.
- Firing the notification and calling this rule -- owned by FEAT-07.SPEC-001 (Plan-Ready Notification Trigger), which enforces these rules rather than duplicating them.
- The exact message content and delivery mechanics per channel -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message), FEAT-07.SPEC-005 (Device-Notification Delivery Integration), and FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration).
- Editing the per-member preference toggle -- owned by FEAT-01.SPEC-005 (Member Profile Detail); this spec only reads the stored preference value.

## Governed Entity

**Entity:** Member Profile (plan-ready delivery-eligibility state), plus a household-week firing cap derived from Weekly Plan's generation identity
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_preferences.plan_ready | boolean | Per-member on/off toggle for this notification (Member Profile) |
| member_type | enum | Organiser, Other Adult Member, young-kid profile (no login), or older-kid limited login (Later) -- determines outright exclusion for both kid rows |
| status | enum | Active, Invited, Left, or Removed -- only Active members are ever eligible |
| device-notification availability (derived, not a Member Profile field) | derived | Whether that member currently has a working, permitted device-notification channel, as reported by FEAT-07.SPEC-005; not stored on Member Profile itself |
| Weekly Plan generation identity (derived, not a field this spec governs) | reference | The week and generation event used to key the once-per-household-per-week firing cap; owned and created by FEAT-03 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Plan-Ready Notification Trigger | At dispatch time, immediately after each generation-completion signal, before any message is sent |
| FEAT-07.SPEC-002 | Plan-Ready Notification Message | Reads this rule's channel resolution and deduplication guarantee to decide what it sends and to whom |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_preferences.plan_ready | Must be true for the member to be eligible this week | Always | At dispatch time (FEAT-07.SPEC-001), never at generation time | N/A -- this is an eligibility gate, not a user-facing form field; an ineligible member simply receives nothing, with no error surfaced anywhere | No |
| member_type | Must be Organiser or Other Adult Member | Always | At dispatch time | N/A -- both kid rows are excluded outright; neither kid row has a login through which an error could even be shown | No |
| status | Must be Active | Always | At dispatch time | N/A -- Invited, Left, or Removed members are simply not evaluated further | No |
| device-notification availability | No validation beyond its derived true/false state; used only to select the delivery channel, never to block eligibility | Always | At dispatch time, only for members who already pass the three rules above | N/A -- unavailability changes the channel (email), never eligibility itself | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Once-per-household-per-week firing cap | Weekly Plan generation identity, household id | At most one plan-ready dispatch cycle runs per household per week, keyed to the week's generation identity; any completion signal arriving after that week's cap is already recorded is treated as no-action, regardless of how many times generation itself signaled completion | N/A |
| Device-vs-email channel fallback | notification_preferences.plan_ready, device-notification availability | For a member who passes eligibility, deliver by Push if device-notification availability is true; otherwise deliver by Email. If the device-notification delivery capability itself is unavailable system-wide when dispatch runs, every affected member's channel resolves to Email for that week regardless of their individual device state | N/A |
| Delivery-never-blocks-plan guarantee | notification_preferences.plan_ready, Weekly Plan status | A Weekly Plan's status and in-app visibility are set entirely by FEAT-03's generation process and never read or gated by this spec's eligibility, cap, or channel outcomes; a household always sees its plan in-app immediately on generation, independent of whether any notification is ever sent or delivered | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Toggle own plan-ready preference | Maya (Organiser) | Always, on her own Member Profile | -- |
| Toggle own plan-ready preference | Sam (Other Adult Member) | Always, on his own Member Profile only | Attempting to change another member's plan-ready preference has no control to act on -- FEAT-01.SPEC-005 exposes each adult's toggle only on their own profile |
| Toggle own plan-ready preference | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile type, so no control is ever reachable |
| Toggle own plan-ready preference | Jordan (older kid, limited login -- Later) | Never | Notification Prefs is None for this row (Access Matrix); no toggle is shown for this notification |
| Toggle another member's plan-ready preference | Maya (Organiser) | Never -- Household Setup Full does not extend to another adult's own notification choice | The preference control on any other member's profile is not editable by Maya; only that member controls it themselves |
| Trigger the eligibility check and firing cap | System (invoked by FEAT-07.SPEC-001) | Always, once per generation-completion signal | N/A -- invoked internally, not a user-facing action |
| Receive the plan-ready message | Maya, Sam | Only when Active, an adult member type, and their own plan_ready preference is on | The member simply receives nothing that week; no error or placeholder appears anywhere in the product |
| Receive the plan-ready message | Jordan (young kid profile, no login -- MVP) | Never | No login exists; there is no surface on which a message could ever appear to this profile |
| Receive the plan-ready message | Jordan (older kid, limited login -- Later) | Never | Notification Prefs is None for this row; this message is never sent regardless of the household's other settings |
| Receive the plan-ready message | Riley (Operator, support) | Never | Riley's Notification Prefs access is None; support access is read-only and carries no notification channel of its own |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_preferences.plan_ready | Defaults to on for every newly created adult Member Profile | On member creation (FEAT-01.SPEC-005) | Yes -- each adult controls only their own toggle thereafter |
| Effective delivery channel per eligible member | Derived: Push when device-notification availability is true and the capability is up system-wide; Email otherwise | Evaluated fresh at each week's dispatch (FEAT-07.SPEC-001) | No -- a member does not directly choose their channel; it follows from their device state and the capability's own availability |
| Household-week firing cap | Derived: set the first time a generation-completion signal for a given household and week identifier is processed by FEAT-07.SPEC-001 | Set once, at first successful processing for that household-week | No -- the cap cannot be cleared by any user action; only a new week's generation produces a new, distinct firing opportunity |

## Business Rules

- XBR-12: The plan-ready message is sent at most once per household per week, only on generation completion, at the organiser-chosen arrival day and time; the plan's in-app availability never depends on notification delivery; email is the fallback where device notifications are unavailable.
- XBR-13: Each member controls their own plan-ready preference, held on their Member Profile and set within household settings; neither kid row receives notifications.
- This rule is the single source of truth for FEAT-07.SPEC-001's eligibility, once-per-week, and channel-fallback logic; FEAT-07.SPEC-001 calls this rule rather than re-implementing any part of it.
- A member's ineligibility (preference off, wrong member type, or non-Active status) is never surfaced as a product error anywhere -- it is a silent, expected state consistent with product-features.md's "Notification disabled" alternate flow.
- The once-per-household-per-week cap is keyed to the week's generation identity, not to wall-clock time alone, so two households whose generation completes at different moments in the same calendar week are governed entirely independently.

## Edge Cases

- **A member re-enables their plan-ready preference the same day the week's message has already been sent to other members** -- No retroactive send occurs for that member this week; the once-per-household-per-week cap has already been recorded, and the member is included only in the following week's dispatch cycle.
- **A member's plan-ready preference and their household removal race each other (they toggle their preference and are removed by Maya within moments of the same dispatch)** -- FEAT-01.SPEC-005's own removal-vs-edit resolution (reject-with-refresh, per the dependency map's Member Profile Contention note) governs which change lands first; this spec's dispatch-time read simply reflects whichever state won that race.
- **Both Maya and Sam have their plan-ready preference off in a given week** -- Eligibility yields zero members; no dispatch occurs, and both simply see the plan the next time they open the app, per the Business Rules' silent-ineligibility principle.
- **A member's Member Profile is still Invited (has not yet accepted) when generation completes** -- Not Active, so not eligible; once the invitation is accepted and the profile becomes Active, that member is evaluated starting with the next week's generation cycle, never retroactively for a week that already dispatched.
- **The device-notification delivery capability is unavailable for the entire household's dispatch window** -- Every otherwise-Push-eligible member in every household resolves to Email that week, per the channel-fallback cross-field rule; eligibility itself is unaffected.
- **A member's device-notification availability flips from unavailable to available between two different weeks' dispatches** -- Each week's dispatch re-evaluates the channel fresh; no channel choice persists across weeks as a stored preference of its own.

## Acceptance Criteria

**FEAT-07.SPEC-003-AC-01:** Given Maya has her plan-ready preference on and is an Active adult member, when eligibility is evaluated, then she is included in the eligible-member list.

**FEAT-07.SPEC-003-AC-02:** Given Sam has turned his plan-ready preference off, when eligibility is evaluated, then he is excluded from the eligible-member list and receives no message.

**FEAT-07.SPEC-003-AC-03:** Given Jordan is a young-kid profile with no login, when eligibility is evaluated, then Jordan is excluded outright regardless of any preference value, since no such value can even exist for this profile type.

**FEAT-07.SPEC-003-AC-04:** Given Jordan is an older-kid limited login (Later phase), when eligibility is evaluated, then Jordan is excluded, since Notification Prefs is None for this row.

**FEAT-07.SPEC-003-AC-05:** Given a household-week's firing cap is already recorded, when a second generation-completion signal for that same household and week arrives, then eligibility evaluation is skipped entirely and no message is dispatched.

**FEAT-07.SPEC-003-AC-06:** Given Maya has device-notification availability and Sam does not, when channel resolution runs for both, then Maya resolves to Push and Sam resolves to Email in the same run.

**FEAT-07.SPEC-003-AC-07:** Given the device-notification delivery capability is unavailable system-wide when dispatch runs, when channel resolution runs for every eligible member, then all of them resolve to Email that week, regardless of their individual device state.

**FEAT-07.SPEC-003-AC-08:** Given every channel resolution and dispatch for a household's week fails entirely, when the household opens the app, then the new week's plan is fully visible in-app, unaffected by any notification outcome.

**FEAT-07.SPEC-003-AC-09:** Given Maya attempts to change Sam's plan-ready preference from her own Member Profile screen, when she looks for a control to do so, then none exists -- only Sam's own profile exposes his toggle.

**FEAT-07.SPEC-003-AC-10:** Given a new adult Member Profile is created, when it is saved, then its plan-ready preference defaults to on.

**FEAT-07.SPEC-003-AC-11:** Given Sam re-enables his plan-ready preference after this week's message has already dispatched to Maya, when the change is saved, then Sam receives no retroactive message for the current week.

**FEAT-07.SPEC-003-AC-12:** Given a Member Profile is still Invited (not yet Active) when generation completes, when eligibility is evaluated, then that profile is excluded from this week's dispatch.

**FEAT-07.SPEC-003-AC-13:** Given Riley (Operator) has no Notification Prefs access, when eligibility is evaluated for any household, then Riley is never included as a recipient, regardless of any support access currently open.

**FEAT-07.SPEC-003-AC-14:** Given both Maya and Sam have their plan-ready preference off, when generation completes, then eligibility yields zero members and no dispatch occurs, with no error shown anywhere.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Plan-Arrival Day & Time Setting Rule

## Overview

**Name:** Plan-Arrival Day & Time Setting Rule
**ID:** FEAT-07.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs the allowed values, default, and storage of the organiser-chosen day and time the weekly plan arrives.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification
**Governed Entity:** Household (plan_arrival_day_time field)

## Scope and Non-Goals

**In Scope:**
- The allowed day-of-week and time-slot values for plan_arrival_day_time
- The default value applied at household creation
- Who may view and who may change this setting

**Non-Goals:**
- The screen this setting is edited on -- owned by FEAT-01.SPEC-010 (Household Settings Hub); this spec defines only the value rules that screen enforces when saving.
- Using this value to time the actual generation run -- owned by FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), which reads this field as its own trigger condition rather than this spec re-implementing scheduling.
- The per-member on/off preference for receiving the resulting message -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec governs only when the household-wide arrival moment is, not who is notified at that moment.

## Governed Entity

**Entity:** Household
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| plan_arrival_day_time | reference (day-of-week + time-slot pair) | Day of week and a slot from a small set of evening and morning time-slot options (platform parameter: `plan-arrival-time-slots`; Sunday, the set's default Evening slot, applies by default) that the weekly plan arrives |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-010 | Household Settings Hub | On save, when the organiser expands the inline plan-arrival picker on the hub and submits a new day/time pair |
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | Read at the start of every generation cycle to determine the household's own firing moment |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| plan_arrival_day_time (day component) | Must be one of the seven days of the week | Always | On save | "Choose a day of the week." | Yes |
| plan_arrival_day_time (time-slot component) | Must be one of the small set of morning and evening time-slot options in platform parameter: `plan-arrival-time-slots` (multiple selectable slots within each of the Morning and Evening periods, not a single fixed clock time per period) | Always | On save | "Choose a time from the available morning or evening slots." | Yes |
| plan_arrival_day_time (pair) | Both a day and a time-slot must be present together -- neither can be saved alone | Always | On save | "Choose both a day and a time for your plan to arrive." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Day and time-slot are set as one pair | plan_arrival_day_time (day, time-slot) | The two components are always read and written together as a single value; there is no state where one component has a value and the other does not | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change plan_arrival_day_time | Maya (Organiser) | Always | -- |
| Change plan_arrival_day_time | Sam (Other Adult Member) | Never | The setting is shown to Sam as a read-only household fact on FEAT-01.SPEC-010 (a plain summary row with no chevron); no picker control is present. A direct navigation attempt to the edit path shows "Only the organiser can change this." |
| Change plan_arrival_day_time | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile type |
| Change plan_arrival_day_time | Jordan (older kid, limited login -- Later) | Never | Household Setup is None for this row (Access Matrix); no path to this setting exists |
| Change plan_arrival_day_time | Riley (Operator, support) | Never | Riley's Household Setup access is View only, through FEAT-22, and never includes edit controls |
| View plan_arrival_day_time | Maya, Sam | Always | -- |
| View plan_arrival_day_time | Jordan (either row), Riley (outside an open Support Request) | Never | Not shown -- no login (young kid), no Household Setup access (older kid), or no open Support Request to view through (Riley) |
| View plan_arrival_day_time | Riley (Operator, support) | Only while a Support Request for the household is open | Outside an open Support Request, no access to any household fact, including this one |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| plan_arrival_day_time | Sunday, the designated default slot within the Evening period of platform parameter: `plan-arrival-time-slots` | On household creation (FEAT-01), before the organiser makes any explicit choice | Yes -- Maya can change it at any time via FEAT-01.SPEC-010 |

## Business Rules

- XBR-12: The plan-ready message is sent at most once per household per week, only on generation completion, at the organiser-chosen arrival day and time; this rule's value is what makes that arrival day and time organiser-chosen rather than fixed.
- The default of Sunday evening reflects the product's own description of the core weekly rhythm (BRIEF.md, The Experience: "It's Sunday evening and a notification arrives"), while remaining a default the organiser can change, not a rule every household must live with (product-features.md, Key Capabilities: "Choose when the plan arrives"). The specific slots making up the Morning and Evening periods -- and which slot is the Evening period's default -- are platform-decided values (platform parameter: `plan-arrival-time-slots`), not this spec's invention.
- A change to plan_arrival_day_time takes effect starting with the following week's generation cycle; it never retroactively re-fires generation or a plan-ready message for a week that has already generated (FEAT-03.SPEC-003 Edge Cases).
- This rule is the single source of truth for the allowed values and default of plan_arrival_day_time; FEAT-01.SPEC-010's inline plan-arrival picker and FEAT-03.SPEC-003's schedule read both defer to it rather than duplicating the value rules.

## Edge Cases

- **Maya changes the plan-arrival day/time after this week's generation has already run** -- Per FEAT-03.SPEC-003's own Edge Cases, the change applies to the following week's schedule only; this week's already-completed generation and its plan-ready message are unaffected.
- **A household is created before this feature's default was in place (a re-run or migration scenario)** -- The Sunday-evening default applies retroactively as the household's value until the organiser explicitly changes it, since plan_arrival_day_time is never left unset.
- **Maya selects a day but the time-slot selection fails to register (a partial in-progress edit)** -- The cross-field rule blocks the save with "Choose both a day and a time for your plan to arrive."; the household's previously saved value remains active until a complete pair is submitted.
- **The household is on the free tier, where AI generation does not run** -- The plan_arrival_day_time value is still stored and editable per this rule, but FEAT-03.SPEC-003 never fires for a free-tier household (FEAT-03.SPEC-009's tier gate), so no plan-ready message ever results from it while the household stays on the free tier; the setting simply takes effect once the household upgrades.
- **Maya is editing this setting on one device while it is also being read by an in-progress generation cycle on the household's configured schedule** -- Per the dependency map's low-contention profile for Household, the in-flight generation cycle uses the value it read at its own start; a concurrent edit takes effect only for the next cycle, never interrupting or altering a cycle already underway.

## Acceptance Criteria

**FEAT-07.SPEC-004-AC-01:** Given Maya expands the inline plan-arrival picker on FEAT-01.SPEC-010 with no prior change made, when the picker opens, then it shows Sunday and the Evening period's default slot (platform parameter: `plan-arrival-time-slots`) as the current value.

**FEAT-07.SPEC-004-AC-02:** Given Maya selects Wednesday and one of the available Morning slots and saves, when the save completes, then the household's plan_arrival_day_time is updated to Wednesday paired with that selected Morning slot.

**FEAT-07.SPEC-004-AC-03:** Given Maya attempts to save a time-slot selection without a day selected, when she taps save, then she sees "Choose both a day and a time for your plan to arrive." and the previous value remains active.

**FEAT-07.SPEC-004-AC-04:** Given Sam views the household settings hub, when he looks for an edit control on the plan-arrival setting, then none is present -- he sees the current value as read-only.

**FEAT-07.SPEC-004-AC-05:** Given Sam attempts to navigate directly to the plan-arrival edit path, then he sees "Only the organiser can change this."

**FEAT-07.SPEC-004-AC-06:** Given Riley has an open Support Request for a household, when Riley views that household's settings through FEAT-22, then the plan-arrival day/time is visible as a read-only fact.

**FEAT-07.SPEC-004-AC-07:** Given Riley has no open Support Request for a household, when Riley attempts to view any household fact, then the plan-arrival day/time is not accessible.

**FEAT-07.SPEC-004-AC-08:** Given Maya changes the plan-arrival time from Evening to Morning after this week's generation has already completed, when the change is saved, then this week's already-generated plan and its plan-ready message are unaffected, and the new time applies starting next week.

**FEAT-07.SPEC-004-AC-09:** Given a new household is created, when setup completes, then its plan_arrival_day_time defaults to Sunday paired with the Evening period's designated default slot (platform parameter: `plan-arrival-time-slots`) without the organiser making an explicit choice.

**FEAT-07.SPEC-004-AC-10:** Given a household is on the free tier with a plan_arrival_day_time value stored, when that day/time occurs, then no generation fires and no plan-ready message results, since FEAT-03.SPEC-009's tier gate blocks generation regardless of this setting.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Integration Spec: Device-Notification Delivery Integration

## Overview

**Name:** Device-Notification Delivery Integration
**ID:** FEAT-07.SPEC-005
**Type:** Integration
**Purpose:** The product delivers the plan-ready message to a member's device through an external device-notification delivery capability, including queuing and redelivery when that device is offline.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- Dispatching the plan-ready message's Push variant to an eligible member's device
- Offline queuing and automatic redelivery once a device reconnects
- User-facing behavior when this capability is slow, unavailable, or reports a delivery problem
- Disclosure of what data is shared with this capability

**Non-Goals:**
- Choosing the device-notification vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Deciding which members are eligible or which channel they resolve to -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec only carries out delivery once a member has already been resolved to the Push channel.
- The exact notification content -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message); this spec transports that content, it does not author it.
- Native mobile push infrastructure or app-store-distributed push -- excluded per scope-boundaries.md SC-05: the platform is a responsive web app for v1 with no native apps, so this capability is scoped to what a web app can deliver.
- The swap-suggestion, nightly-nudge, and manual-planning-pick notifications that also rely on this same capability -- those are owned by One-Tap Meal Swap (FEAT-04), Tonight's Dinner Reminder (FEAT-13), and Manual Weekly Planning (FEAT-23) respectively; this spec is the shared delivery boundary those features' own notifications reference, per the Feature Dependency Map's External Touchpoints table, but this spec's own Product Behaviors Enabled and Data Exchanged sections describe only this feature's plan-ready use of it.

## Capability Category

**Category:** Device-notification delivery
**Dependency Source:** ASMP-31 -- "Device-notification delivery capability -- Required for the weekly 'plan ready' notification, the daily 'tonight's dinner' nudge, and swap-suggestion alerts; without it, these degrade to email (plan ready) or in-app-only discovery, weakening the product's proactive rhythm." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Device-notification delivery (ASMP-31)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-13, FEAT-04, FEAT-23; this spec is the shared delivery-boundary Integration spec)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| An eligible household member receives "next week's plan is ready" as a device notification | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |
| A plan-ready notification sent while a member's device is offline still reaches them once they reconnect | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Notification content (exact title and body text) | FEAT-07.SPEC-002's content templates -- no household or entity fields beyond the fixed text | Each dispatch to a member resolved to the Push channel | The capability needs the exact text to display on the device |
| Recipient device/channel reference | The capability's own registration for that member's device, established when the member's device previously granted permission -- not a Member Profile field the product stores or exposes further | Each dispatch | Routes the notification to the correct device |

Household plan contents, meal names, budget figures, dietary rules, and every Member Profile field beyond the destination device reference never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / undelivered -- device unregistered or permission revoked) | The capability reports the result of a dispatch attempt | No dependency-map entity is updated; the outcome feeds only this spec's own delivery-tracking, since a household's Weekly Plan visibility never depends on it (XBR-12) |
| Device-reconnected signal | The capability reports that a previously offline device has come back online | No dependency-map entity is updated; the signal triggers redelivery of any notification still queued for that device |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery confirmed | The capability confirms the notification reached the device | None | None beyond the notification itself now being visible to the member | FEAT-07.SPEC-002 |
| Delivery failed (device unreachable at all -- unregistered or permission revoked) | The capability reports it cannot reach the device by any means, not merely that it is temporarily offline | None -- FEAT-07.SPEC-003's channel resolution for that member is not retroactively changed for this week's already-attempted dispatch | No user-facing error; the plan remains available in-app immediately regardless (XBR-12) | FEAT-07.SPEC-002 |
| Device reconnected | A device with a queued plan-ready notification comes back online | The queued notification is delivered | The member sees the notification as if newly arrived, at the moment they reconnect | FEAT-07.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | The dispatch attempt is queued and delivered once the capability responds; the household's plan remains fully and immediately visible in-app throughout, unaffected (XBR-12). No delay message is ever shown, since this is a background delivery path with no in-app waiting screen of its own. | No push notification is dispatched for the affected member(s) this week; FEAT-07.SPEC-003's channel-fallback rule treats them as "device notifications unavailable," and Email (FEAT-07.SPEC-006) delivers the message to them instead. | N/A -- this capability accepts any well-formed dispatch of fixed notification text to a registered device; it has no concept of rejecting a plan-ready dispatch the way a payment or content-review capability might reject a request. |

## Consent and Disclosure

- **Device-notification permission** -- Before any Push message can ever reach a member, that member's own device or browser prompts them, through the platform's standard permission mechanism, to allow notifications. The product does not layer a separate in-product disclosure on top of that device-level prompt, since the permission decision is already the member's own, made at the device level. Declining simply means the member is treated as "device notifications unavailable" and receives the Email fallback instead (FEAT-07.SPEC-003).
- **What is shared** -- Only the fixed notification text (FEAT-07.SPEC-002's content templates) and the destination device reference cross this boundary. No meal, budget, dietary, or other household content is ever included, a boundary each adult can see restated wherever this feature's preference is set (FEAT-01.SPEC-005).

## Edge Cases

- **The same delivery-confirmed event is delivered twice for one dispatch** -- No user-visible duplicate results, since the notification itself was sent once; a second delivery-confirmed report changes nothing.
- **A delivery-failed event arrives for a member already removed from the household** -- The event is discarded on arrival; a former member's device is never notified regardless of when the capability's report arrives.
- **A device reconnects after its queued notification has already expired per FEAT-07.SPEC-002's Delivery Rules** -- No delivery occurs on that stale reconnect; expiry takes precedence over a late reconnect signal.
- **The capability goes down mid-dispatch, after some household members' pushes were already confirmed but before others'** -- Each member's outcome is independent; already-confirmed deliveries are unaffected, and the remaining undelivered members fall back to Email for that week.
- **Two reconnect events for the same device arrive out of order or are both delivered** -- The queued notification is delivered at most once regardless of how many reconnect signals arrive, per FEAT-07.SPEC-002's deduplication guarantee.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Triggered by (inbound) | Hands off Push-channel dispatches to this integration |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Affects (outbound) | Delivery, queuing, and reconnect outcomes govern what that spec's Delivery Rules describe |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Reads this integration's availability to resolve each member's channel |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | References (outbound) | Shares this same delivery boundary for its own Push variant |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Shares this same delivery boundary for its nightly nudge and same-day correction |
| FEAT-23 (Manual Weekly Planning) | References (outbound) | Shares this same delivery boundary for its pick-suggestion alerts |

## Analytics and Success Signals

- **device_notification_dispatched** (household id, generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **device_notification_delivery_confirmed** (latency_bucket: under_1_min / 1_to_5_min / over_5_min) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach" (the metric's within-one-minute delivery target)
- **device_notification_capability_down** (affected_household_count) -- N/A -- no Stage 2 metric measures capability outage duration or breadth directly; retained to observe how often the email-fallback path is exercised system-wide.

## Acceptance Criteria

**FEAT-07.SPEC-005-AC-01:** Given Maya is resolved to the Push channel for this week's message, when the dispatch is sent and her device is online, then the notification is delivered and confirmed.

**FEAT-07.SPEC-005-AC-02:** Given Sam's device is offline when his Push dispatch is sent, when his device reconnects later, then the queued notification is delivered at that point.

**FEAT-07.SPEC-005-AC-03:** Given a device reports it is unregistered when a dispatch is attempted, when that report is received, then FEAT-07.SPEC-002 shows no error to the household and the plan remains visible in-app.

**FEAT-07.SPEC-005-AC-04:** Given the capability is slow to respond, when a dispatch is attempted, then the attempt is queued and the household's plan remains fully visible in-app throughout, with no in-app delay indicator for this background path.

**FEAT-07.SPEC-005-AC-05:** Given the capability is down for the entire dispatch window, when eligible Push-resolved members are dispatched, then they receive the Email variant instead, per FEAT-07.SPEC-003's fallback rule.

**FEAT-07.SPEC-005-AC-06:** Given Maya has never received a Push notification from Plateful before, when the capability first attempts to reach her device, then her device's own permission prompt governs whether it can, with no separate in-product disclosure required.

**FEAT-07.SPEC-005-AC-07:** Given Maya declines her device's notification permission, when this week's dispatch runs, then she is treated as "device notifications unavailable" and receives the Email fallback.

**FEAT-07.SPEC-005-AC-08:** Given a delivery-confirmed event for Maya's dispatch is delivered twice, when the second copy arrives, then nothing changes and no duplicate notification appears to her.

**FEAT-07.SPEC-005-AC-09:** Given Sam is removed from the household after his dispatch was queued but before a delivery-failed event for it arrives, when that event arrives, then it is discarded and no action is taken on his behalf.

**FEAT-07.SPEC-005-AC-10:** Given Sam's device reconnects only after his queued notification has expired per FEAT-07.SPEC-002's Delivery Rules, when the reconnect signal arrives, then no delivery occurs.

**FEAT-07.SPEC-005-AC-11:** Given the capability goes down partway through dispatching to a household with two eligible Push-resolved members, when one member's push was already confirmed, then that confirmation stands and only the remaining member falls back to Email.

**FEAT-07.SPEC-005-AC-12:** Given only the plan-ready message's fixed text and a device reference are ever sent to this capability, when any dispatch occurs, then no meal, budget, or dietary data is included in what leaves the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 2 (slow, down; reject cell is N/A and excluded) | 2 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



# Integration Spec: Plan-Ready Email Fallback Integration

## Overview

**Name:** Plan-Ready Email Fallback Integration
**ID:** FEAT-07.SPEC-006
**Type:** Integration
**Purpose:** The product delivers the plan-ready message by transactional email to a member whose device notifications are unavailable or disabled, so the weekly rhythm still reaches them.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- Sending the plan-ready message's Email variant to a member resolved to the Email channel
- Retry behavior for a transient send failure
- User-facing behavior when this capability is slow, unavailable, or a send is permanently rejected (bounced or invalid address)
- Disclosure of what data is shared with this capability for this route

**Non-Goals:**
- Choosing the transactional email vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Deciding which members resolve to the Email channel -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec only carries out delivery once a member has already been resolved to Email.
- The exact email subject and body content -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message); this spec transports that content, it does not author it.
- Account sign-up, sign-in recovery, billing, data-export, deletion, or safety-report email -- those routes on the same transactional email capability are owned by FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-14.SPEC-012, and FEAT-18.SPEC-012 respectively; this spec owns only the plan-ready fallback route and does not duplicate any of those other routes' behavior, per the Feature Dependency Map's Cross-Feature Touchpoints entry distinguishing this spec from FEAT-01.SPEC-017.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for account sign-up and sign-in recovery, the plan-ready email fallback, billing and grace-period notices, data-export and deletion confirmations, safety-concern reports reaching the operator, and support acknowledgements; without it, several account, billing, and safety messages would have no reliable route." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this feature's own route: FEAT-07.SPEC-006, plan-ready email fallback)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A member without device notifications available or enabled still receives "next week's plan is ready" by email | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |
| Every eligible member in a household still receives the plan-ready message the week the device-notification capability itself is unavailable | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient email address | Member Profile -- sign_in (email) | Each dispatch to a member resolved to the Email channel | The capability needs an address to deliver to |
| Recipient display name | Member Profile -- display_name | Each such dispatch | Personalizes the greeting per FEAT-07.SPEC-002's content template |
| Fixed message content (subject and body text) | FEAT-07.SPEC-002's content templates -- no other entity fields | Each such dispatch | The capability needs the exact text to send |

Household plan contents, meal names, budget figures, dietary rules, and every Member Profile field beyond email and display name never leave the product through this route.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the result of a send attempt | No dependency-map entity is updated; the outcome feeds only this spec's own retry logic, since a household's Weekly Plan visibility never depends on it (XBR-12) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Send succeeded | The capability confirms the email was delivered | None | None beyond the email itself now being in the member's inbox | FEAT-07.SPEC-002 |
| Send failed (transient) | The capability reports a temporary failure | None -- retry scheduled per FEAT-07.SPEC-002's Delivery Rules (up to 3 retries over 6 hours) | No user feedback during retry; the plan is already visible in-app | FEAT-07.SPEC-002 |
| Send failed (permanent -- invalid or hard-bounced address) | The capability reports the address cannot receive mail at all | None -- no further retry is attempted for this week's dispatch | No user-facing error that week; the plan remains visible in-app regardless (XBR-12), and the member sees it the next time they open the app | FEAT-07.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | The send is queued and retried per FEAT-07.SPEC-002's Delivery Rules; the household's plan remains fully visible in-app throughout, with no in-app waiting indicator for this background path. | No fallback email is sent to the affected member(s) that week; the plan remains fully visible in-app immediately (XBR-12), and no error is shown to the household -- the affected member simply sees the plan the next time they open the app. | A hard bounce or invalid-address rejection ends that week's dispatch attempt with no further retry; the plan remains visible in-app, and there is no separate user-facing error, since a member without a working address for this week's send already reaches the plan by opening the app directly. |

## Consent and Disclosure

- **Email established as the account's contact channel** -- Each adult's sign-in email is provided during account creation (FEAT-01), at which point it is established as the account's contact channel for the product's transactional messages. This spec adds no separate disclosure moment beyond that: sending "next week's plan is ready" to an address the member already provided for their own account, when their plan-ready preference is on, stays within that address's existing purpose.
- **What is never shared** -- Household plan contents, meal names, budget figures, and dietary information are never included; only the fixed subject and body naming that the plan is ready, addressed with the member's own display name, ever leaves the product through this route.

## Edge Cases

- **A duplicate send event fires for the same household-week** -- FEAT-07.SPEC-002's deduplication guarantee (at most one message per member per household-week) prevents a second email from reaching the member even if the underlying send call is retried by the capability.
- **A send event arrives for a member removed from the household before delivery completes** -- The event is discarded on arrival; a former member's address is never used to deliver a household's plan-ready message.
- **The capability goes down mid-send while dispatching to multiple email-resolved members in the same household** -- Each recipient's send is independent; a failure for one member does not affect delivery to another.
- **Events arrive out of order (a failure notice arrives after a success notice for the same dispatch)** -- The most recent event by event time governs; a stale failure notice arriving after a confirmed success changes nothing, and no duplicate or corrective email is sent.
- **A member's email address is changed on their Member Profile between dispatch and delivery** -- The dispatch already in flight uses the address read at send time; a mid-flight address change takes effect starting with the following week's dispatch, not retroactively for the one already sent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Triggered by (inbound) | Hands off Email-channel dispatches to this integration |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Affects (outbound) | Send, retry, and failure outcomes govern what that spec's Delivery Rules describe |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Reads this integration's role as the fallback channel when resolving each member's channel |
| FEAT-01.SPEC-017 (Account Sign-Up & Sign-In Recovery Email) | Cross-reference | Shares the same transactional email capability for a distinct route (account/recovery); this spec owns only the plan-ready fallback route |

## Analytics and Success Signals

- **plan_ready_email_sent** (generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_email_delivery_failed_permanent** (reason: bounced / invalid_address) -- N/A -- no Stage 2 metric tracks permanent email-delivery failure specifically for this route; retained so a member who silently stops receiving this message by email is observable rather than invisible.

## Acceptance Criteria

**FEAT-07.SPEC-006-AC-01:** Given Sam is resolved to the Email channel for this week's message, when the send completes successfully, then he receives the email with the subject "Next week's plan is ready."

**FEAT-07.SPEC-006-AC-02:** Given a send to Sam fails transiently, when it is retried within the 6-hour window and the retry succeeds, then he receives exactly one email.

**FEAT-07.SPEC-006-AC-03:** Given a send to Sam fails on every retry within the 6-hour window, when the final retry fails, then no further attempt is made that week, and Sam sees the plan the next time he opens the app with no error shown.

**FEAT-07.SPEC-006-AC-04:** Given the capability reports Sam's address as permanently invalid, when that report is received, then no further retry occurs and no user-facing error appears anywhere in the product.

**FEAT-07.SPEC-006-AC-05:** Given the capability is slow to respond, when a send is attempted, then it is queued and retried per FEAT-07.SPEC-002's Delivery Rules while the household's plan remains fully visible in-app.

**FEAT-07.SPEC-006-AC-06:** Given the capability is down for the entire dispatch window, when email-resolved members are due to be sent this week's message, then no email is sent to them that week and no error is shown to the household.

**FEAT-07.SPEC-006-AC-07:** Given Sam's sign-in email was already provided during his own account setup, when this route sends him the plan-ready email, then no separate disclosure prompt is required beyond that original account setup.

**FEAT-07.SPEC-006-AC-08:** Given a duplicate send event fires for a household-week that has already delivered its message, when the duplicate is processed, then no second email reaches the member.

**FEAT-07.SPEC-006-AC-09:** Given Sam is removed from the household after his dispatch was queued but before a send event for it arrives, when that event arrives, then it is discarded.

**FEAT-07.SPEC-006-AC-10:** Given the capability goes down partway through sending to a household with two email-resolved members, when one member's send already succeeded, then that success stands and only the remaining member's send is affected.

**FEAT-07.SPEC-006-AC-11:** Given only the plan-ready message's fixed text, recipient email, and display name are ever sent to this capability, when any dispatch occurs, then no meal, budget, or dietary data is included in what leaves the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (slow, down, rejects) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |

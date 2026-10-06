# FEAT-13 — Tonight's Dinner Reminder

This chapter covers FEAT-13, Tonight's Dinner Reminder, a Important-tier feature. It contains 6 specifications carrying 60 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | automation | 9 |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | notification | 10 |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | automation | 9 |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | notification | 8 |
| FEAT-13.SPEC-005 | Nudge Delivery & Eligibility Rules | logic-rule | 15 |
| FEAT-13.SPEC-006 | Prep-Reminder Derivation Rule | logic-rule | 9 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Tonight's Dinner Reminder

## Summary

**Feature:** Tonight's Dinner Reminder
**ID:** FEAT-13
**Description:** On the day of a planned dinner, the household gets a brief nudge with what's cooking and any prep reminder it needs (e.g., taking something out of the freezer).
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** The brief depicts this directly: "On Wednesday at 5pm a nudge arrives: 'Tonight: 20-minute pasta — take the chicken out of the freezer.'" (BRIEF.md, The Experience). Included at MVP because it is core to closing the "what's for dinner?" problem the brief opens with, not a later refinement.

**Key Capabilities:**
- Send the day's nudge — Household members with notifications enabled are told what's for dinner at a sensible time before cooking
- Include a prep reminder — If a dinner needs early prep (e.g., defrosting), the nudge calls it out
- Control who gets nudged — Each household member can enable or disable this nudge for themselves

This feature is entirely a background/system-message feature: it defines no screens of its own. Its one configurable setting (each adult's own nightly-nudge preference) is edited through the screens that Household Setup & Member Profiles (FEAT-01) owns; this Brief defines the rules, automations, and messages that make the nudge happen, and cross-references FEAT-01's screens rather than duplicating them.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | Automation | Maya, Sam | At a sensible pre-dinner time, determines whether tonight's dinner exists, resolves eligible members, and fires the nudge exactly once per household per day |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | Notification | Maya, Sam | The "Tonight: {meal} — {prep step}" message itself — content, audience, and tap-through behavior, delivered by device notification with an in-app "Tonight" card fallback |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | Automation | Maya, Sam | When a same-day swap changes tonight's dinner after the original nudge was already sent, fires at most one follow-up correction |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Notification | Maya, Sam | The brief follow-up message naming the new dinner, so no household member cooks the wrong thing |
| FEAT-13.SPEC-005 | Nudge Delivery & Eligibility Rules | Logic/Rule | All | Governs who is ever eligible for the nudge, the once-per-household-per-day cap, the at-most-one-correction cap, the no-invented-prep-step disposition, and the guarantee that a failed delivery never blocks the meal's in-app visibility |
| FEAT-13.SPEC-006 | Prep-Reminder Derivation Rule | Logic/Rule | Maya, Sam | Derives the prep-reminder text (or its absence) from the Planned Meal's recipe's early-prep requirements |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Send the day's nudge | FEAT-13.SPEC-001, FEAT-13.SPEC-002 | Trigger fires once per household per day when a Planned Meal exists for that night; the message is delivered by device notification or its in-app card fallback | Phase 2 (Explicit) |
| Include a prep reminder | FEAT-13.SPEC-002, FEAT-13.SPEC-006 | The derivation rule computes the prep-reminder text from the recipe's early-prep requirements; the message includes it when present and names the meal only when absent | Phase 2 (Explicit) |
| Control who gets nudged | FEAT-13.SPEC-005; screen: FEAT-01.SPEC-005 (Member Profile Detail) | Each adult's own nightly-nudge preference (stored on their Member Profile, edited in FEAT-01's screens, per XBR-13) is read by the eligibility rule at trigger time | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | Phase 4 (Trigger-Response, cross-feature effect) | The Primary Flows & Alternates field's "meal swapped same-day" alternate and XBR-09 both require a distinct trigger, listening to FEAT-04's Apply Meal Swap, that fires only when the original nudge already went out |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Phase 4 (Notification surfacing) | The Communications field and the journey's Failure/Recovery Variant both name a real, audience-and-content-bearing follow-up message, not a same-screen toast |
| FEAT-13.SPEC-005 | Nudge Delivery & Eligibility Rules | Phase 5 (Rule-Constraint Discovery — authorization + conditional logic) | Five or more interacting conditions govern who receives the nudge and when (per-adult preference, kid/Riley/unauthorized exclusion, the once-per-day cap, the at-most-one-correction cap, delivery-failure non-blocking) — past the standalone-spec threshold, and shared by SPEC-001 through SPEC-004 rather than duplicated |
| FEAT-13.SPEC-006 | Prep-Reminder Derivation Rule | Phase 5 (Rule-Constraint Discovery — derivation) | The Data Notes field names a derived field ("the prep-reminder text, computed from the Planned Meal's recipe requirements") with a non-trivial source lookup and a defined no-prep disposition — this crosses the standalone-spec threshold rather than staying inline in the message spec |

**No Integration spec for device-notification delivery:** assumptions-constraints.md's Dependencies section (ASMP-31) names device-notification delivery as required by this feature, and the External Touchpoints slice lists this row as "pending — awaiting validated Brief for FEAT-13." FEAT-07.SPEC-005 (Device-Notification Delivery Integration) already specifies the delivery boundary shared by FEAT-04, FEAT-13, and FEAT-23 notifications, including offline queuing and redelivery on reconnect — exactly the behavior this feature's States field (Offline-degraded) requires. This feature needs no behavior beyond that boundary, so it inventories no Integration spec of its own; SPEC-002 and SPEC-004 reference FEAT-07.SPEC-005 directly (see Cross-Feature Touchpoints), resolving the pending row.

## Entity-Lifecycle Coverage Matrix

This feature manages no entity through the create/read/update/delete lifecycle. Its sole Connected Entity, Planned Meal, is read-only for this feature (product-features.md Connected Entities: "Planned Meal (read)") — its creation, update, and deletion/archival are owned entirely by other features (FEAT-03, FEAT-23, FEAT-04, FEAT-02, FEAT-11). Per Phase 3's rule for read-only Connected Entities, Planned Meal is carried below as a Referenced Entity rather than given a full CRUD matrix. Recipe (for prep requirements) and Member Profile (for the nightly-nudge preference) are also read-only for this feature and listed the same way.

**Referenced Entities (read-only for this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Planned Meal | FEAT-13.SPEC-001, FEAT-13.SPEC-003 | The trigger reads whether a Planned Meal exists for the household's current night to decide whether to fire, and to enforce the once-per-household-per-day cap; the correction trigger reads the slot's post-swap state to know what to name in the follow-up |
| Recipe | FEAT-13.SPEC-006 | Reads the Planned Meal's recipe's `prep_requirements` field to derive the prep-reminder text, or its absence |
| Member Profile | FEAT-13.SPEC-001, FEAT-13.SPEC-005 | Reads each adult's `notification_preferences` (nightly nudge on/off) to determine eligibility; the value itself is written through FEAT-01.SPEC-005 (Member Profile Detail), per XBR-13 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| The household's current day has a Planned Meal for dinner | Determine eligible members and fire the "Tonight: …" nudge at a sensible pre-dinner time | Standalone Automation | FEAT-13.SPEC-001 |
| Nudge trigger fires for an eligible member | Deliver "Tonight: {meal} — {prep step}" by device notification, with an in-app "Tonight" card shown at the top of the plan as the fallback where device notifications are not enabled | Standalone Notification | FEAT-13.SPEC-002 |
| A same-day swap (FEAT-04.SPEC-004) changes tonight's dinner after the original nudge was already sent | Fire at most one follow-up correction for that household and day | Standalone Automation | FEAT-13.SPEC-003 |
| Same-day swap correction trigger fires | Deliver a brief correction message naming the new dinner | Standalone Notification | FEAT-13.SPEC-004 |
| Tonight's recipe has an early-prep requirement (e.g., defrosting) | Compute the prep-reminder text from the recipe's `prep_requirements` | Standalone Logic/Rule | FEAT-13.SPEC-006 |
| Tonight's recipe has no early-prep requirement | Nudge names the meal only; no prep step is invented | Standalone Logic/Rule | FEAT-13.SPEC-006 |
| Nudge delivery fails for a member | The Planned Meal remains visible in-app regardless; the failure does not block anything else | Standalone Logic/Rule | FEAT-13.SPEC-005 |
| An adult has disabled their nightly-nudge preference | No nudge or correction is ever sent to them | Standalone Logic/Rule | FEAT-13.SPEC-005 |
| Household member taps the nudge | Opens tonight's dinner within the plan | Cross-feature — owned by AI Weekly Dinner Plan Generation / Manual Weekly Planning | FEAT-03.SPEC-001 / FEAT-23.SPEC-001 responsibility, referenced by FEAT-13.SPEC-002 |
| Device notifications are not enabled for a member | The day's dinner and prep step show as a "Tonight" card at the top of the plan instead | Cross-feature — card surface owned by AI Weekly Dinner Plan Generation / Manual Weekly Planning | FEAT-03.SPEC-001 / FEAT-23.SPEC-001, governed by FEAT-13.SPEC-002 |

## Shared Context

**Shared Entities:**
- Planned Meal — read only, by FEAT-13.SPEC-001 (existence + once-per-day cap) and FEAT-13.SPEC-003 (post-swap state for the correction). No fields are created or updated by this feature.
- Recipe — `prep_requirements` field read by FEAT-13.SPEC-006 to derive the prep-reminder text.
- Member Profile — `notification_preferences` field (nightly nudge on/off) read by FEAT-13.SPEC-001 and FEAT-13.SPEC-005; written through FEAT-01.SPEC-005, per XBR-13.

**Shared UI Patterns:**
- N/A — this feature defines no screens of its own. Its one configurable setting (the nightly-nudge preference) is edited entirely within FEAT-01's existing screen (Member Profile Detail); this Brief cross-references that screen rather than duplicating its UI. The "Tonight" card fallback is content this feature's rules govern but that lives inside FEAT-03's and FEAT-23's own Weekly Plan screens.

**Shared Validation:**
- FEAT-13.SPEC-005 defines the eligibility, once-per-day, at-most-one-correction, and delivery-failure rules; FEAT-13.SPEC-001 through FEAT-13.SPEC-004 all reference SPEC-005 rather than restating the logic.
- FEAT-13.SPEC-006 defines the prep-reminder derivation and no-prep disposition; FEAT-13.SPEC-002 references it to build the nudge's content rather than deriving the text itself.

## Internal Dependency Map

```
FEAT-13.SPEC-001 (Tonight's Nudge Trigger) -> [reads eligibility & once-per-day cap from] -> FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules)
FEAT-13.SPEC-001 (Tonight's Nudge Trigger) -> [reads prep-reminder text from] -> FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule)
FEAT-13.SPEC-001 (Tonight's Nudge Trigger) -> [fires] -> FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message)
FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) -> [reads at-most-one-correction cap from] -> FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules)
FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) -> [fires] -> FEAT-13.SPEC-004 (Same-Day Swap Correction Message)
```

**Default Entry:** N/A — this feature has no screen a household member navigates to. Its only user-visible surfaces are its two messages (FEAT-13.SPEC-002, FEAT-13.SPEC-004), which deep-link the household directly into FEAT-03.SPEC-001 or FEAT-23.SPEC-001 (Weekly Plan View, tonight's dinner) when tapped, or show as the in-app "Tonight" card at the top of those same screens when device notifications are not enabled.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-13.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Reads whether an AI-generated Planned Meal exists for tonight | Trigger evaluates at the nudge time each day |
| FEAT-13.SPEC-001 | Inbound | FEAT-23 (Manual Weekly Planning) | Reads whether a manually picked Planned Meal exists for tonight | Trigger evaluates at the nudge time each day |
| FEAT-13.SPEC-003 | Inbound | FEAT-04 (One-Tap Meal Swap) | A same-day swap (FEAT-04.SPEC-004, Apply Meal Swap) after the original nudge is the sole trigger for the correction (XBR-09) | Swap completes same-day, after the day's nudge was already sent |
| FEAT-13.SPEC-002, FEAT-13.SPEC-004 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Tapping either message lands the member directly on tonight's dinner within the plan | Household member taps the nudge or its correction |
| FEAT-13.SPEC-002, FEAT-13.SPEC-004 | Outbound | FEAT-23 (Manual Weekly Planning) | Tapping either message lands the member directly on tonight's dinner within a manually built week | Household member taps the nudge or its correction |
| FEAT-13.SPEC-002 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) / FEAT-23 (Manual Weekly Planning) | Where device notifications are not enabled, the day's dinner and prep step show as a "Tonight" card at the top of the Weekly Plan screen (FEAT-03.SPEC-001 / FEAT-23.SPEC-001) | Member without device notifications enabled opens the plan on nudge day |
| FEAT-13.SPEC-001, FEAT-13.SPEC-005 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Reads each adult's nightly-nudge preference from their Member Profile; the preference toggle itself is set through FEAT-01.SPEC-005 (Member Profile Detail) and opened from FEAT-01.SPEC-010 (Household Settings Hub) | Trigger runs at nudge time each day |
| FEAT-13.SPEC-002, FEAT-13.SPEC-004 | Outbound (dependency) | FEAT-07 (Weekly Plan Ready Notification, FEAT-07.SPEC-005) | Both messages are delivered through the device-notification delivery capability that FEAT-07.SPEC-005 owns, including offline queuing and redelivery on reconnect — this feature inventories no Integration spec of its own for the same capability | Every nudge or correction send |

## Non-Functional Notes

**Data volumes / growth:** At most one nudge, and at most one follow-up correction, per household per day, tied to that day's Planned Meal (Validation & Limits) — not to user action. Volume scales with household count (several thousand households in the first year, ASMP-24) and each household's 2-6 enabled adult members, not with any per-feature data growth of its own; this feature stores no growing dataset.

**Responsiveness:** Delivery is near-instant relative to the chosen pre-dinner time (States field: Loading N/A — "delivery is near-instant"). No success metric names FEAT-13 as its Connected Feature directly, but the Reported Food Waste and Spend Reduction metric depends on planned dinners actually getting cooked, which is this feature's contribution to closing the "what's for dinner?" gap (success-metrics.md; FEAT-13 Rationale).

**Data sensitivity / privacy:** The nudge and its correction carry only the meal name and, when applicable, a prep step — household personal data about what the family eats (ASMP-14, ASMP-26), never sold or used for advertising. Neither kid row (young no-login profile or Later-phase older kid) nor Riley (Operator, support) ever receives it (Access field); unauthorized visitors receive nothing. This keeps children's data out of this feature entirely, consistent with ASMP-26's privacy posture.

**Compliance flags:** N/A — this feature carries no health or financial data, and its Access rules exclude both kid rows and Riley outright, so no children's-privacy-class data ever reaches this feature's delivery path (ASMP-26).

## Non-Goals

- **Independent kid access to the nudge** — Excluded per scope-boundaries.md (SC-02): v1's default is parent-managed profiles with no login for young kids, and the Access Matrix gives both the young-kid (no-login) and older-kid (Later, limited login) rows `None` on Notification Prefs. Neither kid row will ever receive the nudge or its correction, in v1 or in the Later-phase login.
- **A native mobile push channel** — Excluded per scope-boundaries.md (SC-05): the platform is a responsive web app for v1 with no native apps and no app stores, so device-notification delivery (via FEAT-07.SPEC-005) is scoped to what a web app can deliver, not a native-app push channel.
- **Email as the nudge or correction channel** — Excluded per the feature's own Communications field and the dependency map's External Touchpoints note, which states directly that "email is deliberately NOT used for the nudge — the fallback is the in-app 'Tonight' card." This keeps email reserved for the weekly plan-ready message and account messages, consistent with FEAT-07's separate email fallback design for that different notification.
- **In-product messaging as the delivery channel** — Excluded per scope-boundaries.md (SC-14): the brief names group chat as the coordination problem this product replaces, not a channel to rebuild; the nudge and its correction travel by device notification with an in-app card fallback only, never by an in-app chat or messaging surface.



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



# Notification Spec: Tonight's Dinner Nudge Message

## Overview

**Name:** Tonight's Dinner Nudge Message
**ID:** FEAT-13.SPEC-002
**Type:** Notification
**Purpose:** Tells each eligible household member what's for dinner tonight and any prep step it needs, by device notification with an in-app "Tonight" card fallback.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- The nudge delivered once per household per day when FEAT-13.SPEC-001 dispatches it
- Both surfaces this nudge can appear on: push (device notification) and the in-app "Tonight" card fallback
- Content, placeholders, and delivery rules (deduplication, retry, expiry) for this nudge

**Non-Goals:**
- Deciding whether tonight's nudge fires at all, who is eligible, and which channel each member resolves to -- owned by FEAT-13.SPEC-001 (Tonight's Nudge Trigger) and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec begins once a dispatch has already been decided.
- Deriving the prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this spec only renders that rule's output.
- The follow-up sent when a same-day swap changes tonight's dinner after this nudge already went out -- excluded per the feature's own Communications field, which treats the correction as a distinct message; owned by FEAT-13.SPEC-004 (Same-Day Swap Correction Message).
- Using email as a channel for this nudge -- excluded per the Feature Dependency Map's External Touchpoints note, which states directly that "email is deliberately NOT used for the nudge -- the fallback is the in-app 'Tonight' card," keeping email reserved for the weekly plan-ready message and account messages.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | When the member's resolved channel (FEAT-13.SPEC-005) is device notification | Maya's and Sam's day includes stretches away from the app before dinner starts; a push reaches them at the moment they need to act (start cooking, take something out of the freezer), matching the Brief's own "Tonight: ..." example |
| In-app "Tonight" card | When the member's resolved channel is not device notification (no working push channel for that member) | The Brief and the Feature Dependency Map deliberately exclude email as this nudge's fallback; a persistent card at the top of the plan (FEAT-03.SPEC-001, FEAT-23.SPEC-001) still tells that member what's cooking the moment they open the app |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nudge dispatch for an eligible member | FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | Fires when that automation's "Nudge dispatched" outcome completes, once per eligible member | Member id, meal_name (tonight's Planned Meal's recipe name), prep_reminder_text or its confirmed absence (FEAT-13.SPEC-006) |

## Audience and Preferences

**Recipients:** Maya and Sam -- only when Active and their own notification_preferences.nightly_nudge is on (FEAT-13.SPEC-005). Neither kid row (Access Matrix: Notification Prefs None for both the young-kid, no-login row and the older-kid, Later-phase login row) ever receives it; Riley (Operator, support) has no Notification Prefs access and never receives it; an unauthorized visitor receives nothing, since this is a background message with no screen of its own to reach.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Nightly nudge | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail), opened from FEAT-01.SPEC-010 (Household Settings Hub) |

**Quiet Hours:** N/A -- the nudge fires once per day at a single platform-set pre-dinner time (platform parameter: `nightly-nudge-send-time`) already chosen to fall within normal waking hours before dinner; the product defines no separate quiet-hours window for this notification.

## Content Definition

**Push (device notification):**
- **Title (prep step present):** Tonight: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Tonight: {meal_name}
- **Body:** -- none; the title carries the complete message for this brief nudge, matching the Brief's own example ("Tonight: 20-minute pasta — take the chicken out of the freezer.")
- **CTA:** Open tonight's dinner -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View) or FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)), whichever produced the household's current Weekly Plan, at tonight's slot

**In-app "Tonight" card:**
- **Title (prep step present):** Tonight: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Tonight: {meal_name}
- **Body:** -- none; the same single-line content as the push title, shown at the top of the plan
- **CTA:** Tap the card -- opens tonight's dinner within the same screen (FEAT-03.SPEC-001 or FEAT-23.SPEC-001); no separate navigation destination, since the card already sits on that screen

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {meal_name} | Recipe -- name, via tonight's Planned Meal.recipe | 20-minute pasta | Never empty -- Recipe.name is required (Feature Dependency Map, Recipe fields) and Planned Meal.recipe is required, so a dispatched nudge always has a meal name |
| {prep_reminder_text} | Derived -- FEAT-13.SPEC-006's output from Recipe.prep_requirements | take the chicken out of the freezer | When no prep is required, this placeholder and its preceding " — " are omitted entirely; the title reads "Tonight: {meal_name}" alone, per FEAT-13.SPEC-006's no-invented-prep-step disposition |

## Delivery Rules

**Batching:** N/A -- FEAT-13.SPEC-005's once-per-household-per-day firing cap guarantees at most one Planned-Meal-triggered instance per member per day, so no same-day duplicate instances ever exist to collapse into a batch.
**Deduplication:** FEAT-13.SPEC-005's once-per-household-per-day firing cap guarantees at most one dispatch attempt per member per day; a duplicate trigger for the same day is suppressed inside FEAT-13.SPEC-001 before this spec is ever invoked.
**Retry on failure:** Push delivery failure and offline queuing/redelivery are handled entirely by FEAT-07.SPEC-005 (Device-Notification Delivery Integration), the shared delivery boundary this notification travels on; this spec defines no retry of its own. When that capability reports the device unreachable by any means, no further attempt is made and no error is shown -- the in-app "Tonight" card (already the surface for that member per FEAT-13.SPEC-005's channel resolution, or simply the plan itself if the member is not otherwise eligible) stands as the surviving signal.
**Expiry:** A push instance not delivered by the end of tonight (local time) expires undelivered rather than arriving the next morning about last night's dinner, which would confuse the household. The in-app card is not itself time-limited by this rule -- it is simply superseded once tomorrow's Planned Meal becomes "tonight" the next day.

## Edge Cases

- **Planned Meal removed (safety concern) between trigger and delivery** -- The nudge is cancelled silently on every surface; a nudge about a dinner no longer on the plan is never delivered, matching FEAT-13.SPEC-001's "no dinner planned tonight" disposition for the underlying trigger cancellation.
- **Nightly-nudge preference turned off between trigger and delivery** -- Per FEAT-13.SPEC-005, eligibility is evaluated at dispatch time inside FEAT-13.SPEC-001, not at this spec's delivery step, so a member already dispatched to is not retroactively un-sent; a member who disables the preference before FEAT-13.SPEC-001's dispatch step runs is excluded there instead, before this spec ever fires for them.
- **Quiet hours colliding with expiry** -- Cannot occur: this notification defines no quiet-hours window (Audience and Preferences), so there is no hold to collide with the nightly expiry cutoff.
- **Device notifications unavailable for a member who does have their preference on** -- The in-app card is the delivery of record for that member for that day; no push is attempted or retried beyond FEAT-07.SPEC-005's own handling, and no error of any kind is shown.
- **A same-day swap changes tonight's dinner after this nudge was already delivered** -- This spec's already-delivered content is never edited or re-sent; FEAT-13.SPEC-004 (Same-Day Swap Correction Message) delivers a separate, brief follow-up instead (XBR-09), rather than this spec attempting to update its own past delivery.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | Triggered by (inbound) | Its "Nudge dispatched" outcome fires this notification |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (inbound) | Eligibility, cap, and channel resolution govern who receives this and on which surface |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (inbound) | Supplies the prep_reminder_text content |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Where each adult sets their own nightly-nudge preference |
| FEAT-01.SPEC-010 (Household Settings Hub) | References (inbound) | Entry point into the preference toggle |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | Triggers (outbound) | Carries out push delivery, offline queuing, and redelivery |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | CTA destination and the surface for the "Tonight" card fallback |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Navigation (outbound) | CTA destination and the surface for the "Tonight" card fallback |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | References (outbound) | The separate follow-up that handles a same-day swap, rather than this spec re-sending |

## Analytics and Success Signals

- **dinner_nudge_delivered** (channel: push / in_app_card) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_sent) so delivery reach is observable per member and channel even without a Stage 2 metric attached.
- **dinner_nudge_opened** (channel) -- N/A -- same reason; retained per product-features.md's Signals field (dinner_nudge_opened).
- **dinner_nudge_cta_tapped** (channel; destination: FEAT-03.SPEC-001 / FEAT-23.SPEC-001) -- N/A -- same reason; this is the closest observable signal this feature can emit toward whether nudged dinners actually get cooked, which the Reported Food Waste and Spend Reduction metric (Connected Feature: Weekly Waste & Spend Check-In) ultimately depends on but does not measure through this feature's own events.
- **dinner_nudge_disabled** (member id) -- N/A -- same reason; retained per product-features.md's Signals field (dinner_nudge_disabled), observed here as the state a member is in when this notification's audience is resolved, even though the toggle interaction itself is FEAT-01.SPEC-005's screen action.

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Maya has device notifications available and her nightly nudge on, and tonight's dinner is "20-minute pasta" with a prep requirement of "take the chicken out of the freezer," when FEAT-13.SPEC-001 dispatches her nudge, then she receives a push titled "Tonight: 20-minute pasta — take the chicken out of the freezer."

**FEAT-13.SPEC-002-AC-02:** Given Sam's tonight's dinner has no early-prep requirement, when his nudge is dispatched, then he receives a push titled "Tonight: {meal_name}" with no trailing prep text and no invented prep step.

**FEAT-13.SPEC-002-AC-03:** Given Maya taps her push nudge, when the tap is registered, then she lands on tonight's dinner within FEAT-03.SPEC-001 or FEAT-23.SPEC-001, whichever holds the household's current Weekly Plan.

**FEAT-13.SPEC-002-AC-04:** Given Sam has no working device-notification channel and his nightly nudge is on, when his nudge is dispatched, then he receives no push and instead sees the "Tonight" card at the top of the plan the next time he opens it.

**FEAT-13.SPEC-002-AC-05:** Given Maya has turned her nightly nudge off, when tonight's dispatch runs, then she receives no push and no "Tonight" card.

**FEAT-13.SPEC-002-AC-06:** Given a device reports it cannot be reached at all when Sam's push is attempted, when that report arrives, then no error is shown anywhere and the "Tonight" card remains his surviving signal for tonight's dinner.

**FEAT-13.SPEC-002-AC-07:** Given Maya's push for tonight's dinner has not been delivered by midnight, when the expiry cutoff passes, then the push is not delivered the next morning, and the plan itself (not a stale push) is what she sees.

**FEAT-13.SPEC-002-AC-08:** Given tonight's dinner is swapped after Maya's nudge already arrived, when the swap completes, then this spec's already-delivered nudge is left unchanged and FEAT-13.SPEC-004 delivers a separate correction instead.

**FEAT-13.SPEC-002-AC-09:** Given the underlying Planned Meal is removed by a safety-concern report between dispatch and delivery, when delivery would otherwise occur, then no nudge is delivered on any surface.

**FEAT-13.SPEC-002-AC-10:** Given both Maya and Sam turn their nightly nudge off before FEAT-13.SPEC-001's dispatch step runs, when tonight's evaluation completes, then neither receives a push nor a "Tonight" card, and both simply see tonight's dinner the next time they open the plan.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, in-app card) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 3 (on with push, on without push, off) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Same-Day Swap Correction Trigger

## Overview

**Name:** Same-Day Swap Correction Trigger
**ID:** FEAT-13.SPEC-003
**Type:** Automation
**Purpose:** When a same-day swap changes tonight's dinner after the original nudge was already sent, fires at most one follow-up correction naming the new dinner.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- Detecting that a completed swap affects tonight's slot, after today's original nudge (FEAT-13.SPEC-001) has already been dispatched to at least one member
- Enforcing the at-most-one-correction-per-household-per-day cap
- Resolving the correction audience to exactly the members who received today's original nudge
- Dispatching the correction message (FEAT-13.SPEC-004) to that audience

**Non-Goals:**
- Applying the swap itself, re-checking safety, or updating the Planned Meal's recipe -- owned entirely by FEAT-04.SPEC-004 (Apply Meal Swap); this automation only reacts to that automation's completed outcome.
- Defining the correction audience, the once-per-day correction cap, and the channel-fallback logic in detail -- owned by FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules), which this automation calls rather than re-implementing.
- Deriving the new prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this automation only reads its output for the post-swap recipe.
- The exact correction content and delivery mechanics -- owned by FEAT-13.SPEC-004 (Same-Day Swap Correction Message); this automation only decides that a correction is owed, to whom, and hands off the dispatch instruction.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A same-day swap completes for tonight's slot | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires on that automation's "Swap applied (direct)" or "Swap applied (accepted suggestion)" outcome, for a Planned Meal whose night is today (XBR-09) | Household id, Planned Meal reference, new recipe, previous recipe, swap completion time |

## Processing Logic

1. Receive the swap-applied signal from FEAT-04.SPEC-004, carrying the affected Planned Meal reference and its night.
2. Check whether the swapped Planned Meal's night is today. If not, stop -- no-action outcome; a swap for a future night has no bearing on tonight's nudge.
3. Check via FEAT-13.SPEC-005 whether today's original nudge (FEAT-13.SPEC-001) was ever dispatched to at least one member for this household -- read the dispatched-members record FEAT-13.SPEC-001 wrote for today. If no original nudge was ever dispatched today, stop -- no-action outcome; there is nothing to correct.
4. Check the at-most-one-correction-per-household-per-day cap via FEAT-13.SPEC-005 -- has a correction already been recorded for this household today? If yes, stop -- no-action outcome.
5. Read the correction audience via FEAT-13.SPEC-005: exactly the members recorded as having received today's original nudge -- never a freshly re-evaluated eligible list.
6. Read the prep-reminder text (or its confirmed absence) for the new recipe via FEAT-13.SPEC-006.
7. Dispatch the correction message (FEAT-13.SPEC-004) to each member from step 5, naming the new dinner.
8. Record the correction firing cap for this household-date.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Correction dispatched | Swap completes for tonight, an original nudge was dispatched today, and the correction cap is not yet recorded | Correction firing cap recorded for this household-date | Each member who received today's original nudge receives FEAT-13.SPEC-004's follow-up naming the new dinner | FEAT-13.SPEC-004, FEAT-13.SPEC-005 |
| No-action: swap not for tonight | The swapped Planned Meal's night is not today | None | Nothing sent; a swap for a future night follows its own normal path | -- |
| No-action: no original nudge sent today | Today's original nudge was never dispatched (no dinner existed at nudge time, no eligible members, or today's evaluation has not yet occurred) | None | Nothing sent; there is no delivered original nudge for anyone to have been left with a stale version of | FEAT-13.SPEC-005 |
| No-action: correction already sent today | The at-most-one-correction cap for this household-date is already recorded | None | Nothing sent; a second same-day swap after the first correction is not separately announced | FEAT-13.SPEC-005 |
| Dispatch failure | Dispatch to one or more of today's original-nudge recipients fails | The correction firing cap is not recorded for members whose dispatch could not be attempted | No error is shown; the Planned Meal remains visible in-app with its new dinner, unaffected | FEAT-13.SPEC-004 |

## Data Model

**Reads:** Planned Meal -- night, recipe (new), swap_history (previous recipe), status; Recipe -- prep_requirements for the new recipe, via FEAT-13.SPEC-006; Member Profile -- the dispatched-members record from today's original nudge (FEAT-13.SPEC-001), via FEAT-13.SPEC-005.
**Creates:** None.
**Updates:** None on Planned Meal, Recipe, or Member Profile -- this automation records only its own correction firing-cap marker (read by FEAT-13.SPEC-005), not a dependency-map entity.
**Deletes:** None.

## Business Rules

- XBR-09: A same-day swap after the nightly nudge was sent triggers at most one follow-up correction naming the new dinner.
- This automation is the sole trigger source for FEAT-13.SPEC-004 -- no other event ever fires the correction message.
- The correction audience is exactly today's original-nudge recipients, never a freshly re-evaluated eligible list -- a member who never received today's original nudge is never owed a correction for it.
- At most one correction per household per day, regardless of how many further swaps occur the same night after the first correction (Brief, Validation & Limits).
- This automation never determines the correction audience or the once-per-day correction cap itself -- it calls FEAT-13.SPEC-005 for both.

## Edge Cases

- **Concurrent trigger firing (two swaps for the same slot complete at effectively the same time)** -- FEAT-04.SPEC-009's concurrency lock, referenced by FEAT-04.SPEC-004, ensures only one swap actually completes for a given slot at a time, so this automation never receives two concurrent swap-applied signals for the same Planned Meal.
- **Trigger fires while a previous correction run is in flight** -- A second swap-applied signal for the same household arriving while an earlier correction dispatch is still recording its cap is held until that run finishes, then evaluated against the now-recorded cap and suppressed as a duplicate.
- **A second same-day swap happens after the first correction was already sent** -- The at-most-one-correction cap suppresses a second correction; the household's Weekly Plan view already reflects the latest dinner regardless (FEAT-03.SPEC-001, FEAT-23.SPEC-001), so only the notification is capped, not the plan's accuracy.
- **The swap completes for tonight while today's original nudge dispatch is itself still in flight** -- Step 3's check reads whatever dispatched-members state exists at the moment this automation runs; if the original nudge has not yet recorded any dispatched members, this run treats it as "no original nudge sent today" and takes no action, since there is nothing yet to correct.
- **The swap is undone by a second swap back toward the original recipe before any correction fires** -- Both swaps are separate "Swap applied" outcomes from FEAT-04.SPEC-004; the at-most-one-correction cap means only the first eligible swap's correction can ever fire, and a second corrective message announcing the reversal is not separately sent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | Its "Swap applied" outcomes fire this automation for tonight's slot |
| FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | References (inbound) | Reads that automation's dispatched-members record for today to determine whether a correction is owed |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (outbound) | Correction audience and once-per-day correction cap are read from this rule |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (outbound) | Prep-reminder text for the new recipe is read from this rule |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Triggers (outbound) | Dispatches the correction to the resolved audience |

## Analytics and Success Signals

- **dinner_nudge_correction_dispatched** (household id, recipient_count) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_correction_sent) so correction reach is observable even without a Stage 2 metric attached.
- **dinner_nudge_correction_skipped_no_original_sent** (household id) -- N/A -- same reason; observes how often a same-day swap has no original nudge to correct.
- **dinner_nudge_correction_skipped_cap_reached** (household id) -- N/A -- same reason; observes how often the at-most-one-correction cap suppresses a further same-day swap's correction.

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Maya applies a direct swap to tonight's dinner after today's original nudge already reached both her and Sam, when FEAT-04.SPEC-004 reports the swap applied, then both Maya and Sam receive FEAT-13.SPEC-004's correction naming the new dinner.

**FEAT-13.SPEC-003-AC-02:** Given a swap completes for a night that is not tonight, when FEAT-04.SPEC-004 reports it, then no correction is dispatched.

**FEAT-13.SPEC-003-AC-03:** Given today's original nudge was never dispatched (no dinner existed at nudge time), when a same-day swap later adds a dinner to tonight's slot, then no correction is dispatched, since there was no original nudge to correct.

**FEAT-13.SPEC-003-AC-04:** Given a correction has already been sent for tonight, when a second same-day swap changes tonight's dinner again, then no second correction is sent to anyone.

**FEAT-13.SPEC-003-AC-05:** Given Sam turned his nightly nudge off and so never received today's original nudge, when a same-day swap triggers a correction for Maya, then Sam does not receive the correction.

**FEAT-13.SPEC-003-AC-06:** Given two swaps for the same slot attempt to complete at effectively the same time, when FEAT-04.SPEC-009's concurrency lock resolves them, then this automation only ever processes one completed swap-applied signal for that slot.

**FEAT-13.SPEC-003-AC-07:** Given a second swap-applied signal for the same household arrives while an earlier correction run is still recording its cap, when the in-flight run finishes, then the second signal is evaluated against the now-recorded cap and suppressed.

**FEAT-13.SPEC-003-AC-08:** Given the original nudge dispatch for today is itself still in flight when a swap completes, when this automation checks for dispatched members, then it finds none yet recorded and takes no action.

**FEAT-13.SPEC-003-AC-09:** Given a swap is reversed by a second same-day swap before any correction fires, when the reversal completes, then only the first eligible swap's correction can ever be attempted, and no separate message announces the reversal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (swap-applied outcome) | 1 |
| Outcome Paths | 5 (dispatched, not-tonight, no-original-sent, cap-reached, dispatch failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Notification Spec: Same-Day Swap Correction Message

## Overview

**Name:** Same-Day Swap Correction Message
**ID:** FEAT-13.SPEC-004
**Type:** Notification
**Purpose:** Delivers a brief follow-up naming tonight's new dinner when a same-day swap changes it after the original nudge already went out, so no household member cooks the wrong thing.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- The correction delivered when FEAT-13.SPEC-003 decides a same-day swap owes one
- Both surfaces this correction can appear on: push (device notification) and the same in-app "Tonight" card FEAT-13.SPEC-002 already shows, refreshed to the new dinner
- Content, placeholders, and delivery rules for this correction

**Non-Goals:**
- Deciding whether a correction is owed, to whom, and enforcing the at-most-one-correction-per-day cap -- owned by FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec begins once that decision has already been made.
- Deriving the new prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this spec only renders that rule's output for the post-swap recipe.
- The original nudge this corrects -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message); this spec is a distinct, separate message, not an edit to that one's already-delivered content.
- Repeated corrections for further same-day changes -- excluded per the feature's own Validation & Limits (at most one follow-up correction per household per day); a second same-day swap after a correction has already been sent produces no further message (FEAT-13.SPEC-003).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | When the recipient's channel, resolved fresh at correction-dispatch time via FEAT-13.SPEC-005, is device notification | The correction exists precisely so a member who already saw the original "Tonight: ..." push is not left cooking the wrong thing; reaching them the same way is the most direct correction |
| In-app "Tonight" card | When the recipient's resolved channel is not device notification | The same card FEAT-13.SPEC-002 already shows is refreshed to the new dinner rather than a second card being added, so the member simply sees the current, correct dinner whenever they next open the plan |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Same-day swap correction fires | FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) | Fires when that automation's "Correction dispatched" outcome completes, once per recipient | Member id, new meal_name (via the post-swap Planned Meal.recipe), new prep_reminder_text or its confirmed absence (FEAT-13.SPEC-006) |

## Audience and Preferences

**Recipients:** Exactly the members who received today's original nudge (FEAT-13.SPEC-002), as resolved by FEAT-13.SPEC-003 via FEAT-13.SPEC-005 -- never a freshly re-evaluated eligible list. Neither kid row nor Riley is ever included, for the same reasons as the original nudge (Access Matrix: Notification Prefs None); an unauthorized visitor receives nothing.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Nightly nudge | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail) -- this spec has no preference control of its own; receiving today's original nudge is what puts a member in scope for its correction |

**Quiet Hours:** N/A -- like the original nudge, this correction is a single immediate follow-up sent right after a same-day swap completes, not a scheduled window; the product defines no quiet-hours window for either message.

## Content Definition

**Push (device notification):**
- **Title (prep step present):** Dinner update: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Dinner update: {meal_name}
- **Body:** -- none; the same single-line discipline as the original nudge
- **CTA:** Open tonight's dinner -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View) or FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)), whichever holds the household's current Weekly Plan

**In-app "Tonight" card:**
- **Behavior:** The existing "Tonight" card (FEAT-13.SPEC-002) is refreshed in place to the new {meal_name} and {prep_reminder_text} -- this spec never adds a second card. A member who has not opened the app since the swap simply sees the current, correct dinner the next time they do.
- **CTA:** Tap the card -- opens tonight's dinner within the same screen (FEAT-03.SPEC-001 or FEAT-23.SPEC-001)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {meal_name} | Recipe -- name, via the post-swap Planned Meal.recipe | slow-cooker chili | Never empty -- Recipe.name is required and Planned Meal.recipe is required, so a dispatched correction always has a new meal name |
| {prep_reminder_text} | Derived -- FEAT-13.SPEC-006's output from the new Recipe.prep_requirements | -- (no prep needed) | When no prep is required, this placeholder and its preceding " — " are omitted entirely; the title reads "Dinner update: {meal_name}" alone |

## Delivery Rules

**Batching:** N/A -- FEAT-13.SPEC-005's at-most-one-correction-per-household-per-day cap guarantees at most one correction instance per member per day, so no same-day duplicate correction instances ever exist to batch.
**Deduplication:** The at-most-one-correction cap, enforced by FEAT-13.SPEC-003 before this spec is invoked, guarantees no member ever receives two corrections the same day even if multiple swaps occur.
**Retry on failure:** Identical to FEAT-13.SPEC-002 -- delivered through FEAT-07.SPEC-005 with its own queuing and redelivery; this spec defines no separate retry. Failure is silent and non-blocking, with the refreshed "Tonight" card as the surviving signal.
**Expiry:** A correction not delivered by the end of tonight (local time) expires undelivered, for the same reason as the original nudge -- a stale correction about a dinner from a previous night would confuse rather than help. The in-app card remains accurate regardless, since it reflects the Planned Meal's current state whenever the member next opens it.

## Edge Cases

- **The swap is itself reversed by a second same-day swap before the correction is delivered** -- Per FEAT-13.SPEC-003's at-most-one-correction cap, only the first eligible swap's correction can ever be attempted; if it has not yet been delivered when the reversal happens, it still delivers naming whichever recipe was current when FEAT-13.SPEC-003 read it, and the in-app card (always reflecting current state) is the accurate fallback in the moment of any lag.
- **A member's nightly-nudge preference is turned off between the original nudge and the correction** -- The correction still reaches them: eligibility for the correction is "received today's original nudge," a state already fixed at nudge time, not re-evaluated against the current preference; disabling the preference going forward affects only future nudges and corrections, not the one already owed for tonight.
- **The underlying Planned Meal is removed (safety concern) between the swap and the correction's delivery** -- The correction is cancelled silently; a correction about a dinner no longer on the plan is never delivered, the same disposition as the original nudge's own removed-Planned-Meal edge case.
- **Quiet hours colliding with expiry** -- Cannot occur, since this notification defines no quiet-hours window (Audience and Preferences).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) | Triggered by (inbound) | Its "Correction dispatched" outcome fires this notification |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (inbound) | Correction audience, cap, and channel resolution govern who receives this and where |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (inbound) | Supplies the prep_reminder_text content for the new recipe |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | References (inbound) | The original nudge this corrects, and the shared "Tonight" card it refreshes rather than duplicates |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | Triggers (outbound) | Carries out push delivery, offline queuing, and redelivery |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | CTA destination and the surface for the refreshed "Tonight" card |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Navigation (outbound) | CTA destination and the surface for the refreshed "Tonight" card |

## Analytics and Success Signals

- **dinner_nudge_correction_delivered** (channel: push / in_app_card) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_correction_sent) so correction delivery is observable per member and channel.
- **dinner_nudge_correction_opened** (channel) -- N/A -- same reason; observes whether the correction actually reaches the member before dinner.

## Acceptance Criteria

**FEAT-13.SPEC-004-AC-01:** Given Maya received today's original nudge for "20-minute pasta" and Sam swaps tonight's dinner to "slow-cooker chili," when FEAT-13.SPEC-003 dispatches the correction, then Maya receives a push titled "Dinner update: slow-cooker chili" (with any new prep step included).

**FEAT-13.SPEC-004-AC-02:** Given the new recipe has no early-prep requirement, when the correction is dispatched, then its title reads "Dinner update: {meal_name}" alone, with no invented prep step.

**FEAT-13.SPEC-004-AC-03:** Given Sam's channel is device notification, when his correction is dispatched, then he receives a push and no duplicate "Tonight" card is added.

**FEAT-13.SPEC-004-AC-04:** Given Maya has no working device-notification channel, when her correction is dispatched, then her existing "Tonight" card is refreshed in place to the new dinner rather than a second card appearing.

**FEAT-13.SPEC-004-AC-05:** Given Sam turns his nightly-nudge preference off after receiving today's original nudge but before a same-day swap, when the correction is dispatched, then he still receives it, since eligibility for the correction was already fixed at the time he received the original nudge.

**FEAT-13.SPEC-004-AC-06:** Given the underlying Planned Meal is removed by a safety-concern report between the swap and delivery, when delivery would otherwise occur, then no correction is delivered to anyone.

**FEAT-13.SPEC-004-AC-07:** Given a second same-day swap reverses tonight's dinner before the first correction is delivered, when delivery proceeds, then it names whichever recipe was current at the moment FEAT-13.SPEC-003 read it, and the refreshed "Tonight" card shows the accurate current dinner regardless of any lag.

**FEAT-13.SPEC-004-AC-08:** Given a correction is not delivered by midnight, when the expiry cutoff passes, then it is not delivered the next morning, and the "Tonight" card (reflecting the Planned Meal's current state) remains the accurate signal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, in-app card) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (received original nudge; preference later turned off) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Nudge Delivery & Eligibility Rules

## Overview

**Name:** Nudge Delivery & Eligibility Rules
**ID:** FEAT-13.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who is ever eligible for the nightly nudge and its same-day correction, the once-per-household-per-day nudge cap, the at-most-one-correction-per-day cap, the channel-fallback decision, and the guarantee that a failed delivery never blocks the meal's in-app visibility.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder
**Governed Entity:** Member Profile (nightly-nudge delivery-eligibility state), with household-day firing caps tied to Planned Meal

## Scope and Non-Goals

**In Scope:**
- Eligibility rules: which roles and member states can ever receive the nudge or its correction
- The once-per-household-per-day nudge firing cap
- The at-most-one-correction-per-household-per-day cap, and the rule that the correction audience is exactly today's original-nudge recipients
- The device-notification-vs-in-app-card channel-fallback decision for each eligible member
- The guarantee that notification delivery outcomes never affect the Planned Meal's in-app availability

**Non-Goals:**
- Firing the nudge or the correction and calling this rule -- owned by FEAT-13.SPEC-001 (Tonight's Nudge Trigger) and FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger), which enforce these rules rather than duplicating them.
- The exact message content, and delivery mechanics per channel -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message), FEAT-13.SPEC-004 (Same-Day Swap Correction Message), and FEAT-07.SPEC-005 (Device-Notification Delivery Integration).
- Deriving the prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this spec governs only who receives a nudge or correction, not what it says.
- Editing the per-member preference toggle -- owned by FEAT-01.SPEC-005 (Member Profile Detail); this spec only reads the stored preference value.

## Governed Entity

**Entity:** Member Profile (nightly-nudge delivery-eligibility state), plus household-day firing caps derived from Planned Meal's existence for tonight
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_preferences.nightly_nudge | boolean | Per-adult on/off toggle for this nudge and its correction (Member Profile) |
| member_type | enum | Organiser, Other Adult Member, young-kid profile (no login), or older-kid limited login (Later) -- determines outright exclusion for both kid rows |
| status | enum | Active, Invited, Left, or Removed -- only Active members are ever eligible |
| device-notification availability (derived, not a Member Profile field) | derived | Whether that member currently has a working, permitted device-notification channel, as reported by FEAT-07.SPEC-005; not stored on Member Profile itself |
| Planned Meal existence for tonight (derived, not a field this spec governs) | reference | Whether a Planned Meal exists for the household's current night; owned and created by FEAT-03 or FEAT-23, keys the once-per-day nudge cap |
| Today's dispatched-members record (derived, not a field this spec governs) | reference | The set of members who actually received today's original nudge, written by FEAT-13.SPEC-001; keys the correction audience and the at-most-one-correction cap |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | At dispatch time, immediately after the daily nudge-time signal, before any nudge is sent |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | At dispatch time, immediately after a same-day swap-applied signal, before any correction is sent |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | Reads this rule's channel resolution and deduplication guarantee to decide what it sends and to whom |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Reads this rule's correction audience and channel resolution |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_preferences.nightly_nudge | Must be true for the member to be eligible for today's nudge | Always | At dispatch time (FEAT-13.SPEC-001), never at Planned Meal creation time | N/A -- this is an eligibility gate, not a user-facing form field; an ineligible member simply receives nothing, with no error surfaced anywhere | No |
| member_type | Must be Organiser or Other Adult Member | Always | At dispatch time | N/A -- both kid rows are excluded outright; neither kid row has a login through which an error could even be shown | No |
| status | Must be Active | Always | At dispatch time | N/A -- Invited, Left, or Removed members are simply not evaluated further | No |
| device-notification availability | No validation beyond its derived true/false state; used only to select the delivery surface, never to block eligibility | Always | At dispatch time, only for members who already pass the three rules above | N/A -- unavailability changes the surface (in-app card), never eligibility itself | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Once-per-household-per-day nudge firing cap | Planned Meal existence for tonight, household id | At most one nudge dispatch cycle runs per household per day, keyed to tonight's date; any daily-schedule evaluation arriving after that day's cap is already recorded is treated as no-action | N/A |
| At-most-one-correction-per-household-per-day cap | Today's dispatched-members record, household id | At most one correction dispatch cycle runs per household per day; a second same-day swap after a correction is already recorded produces no further message | N/A |
| Correction audience derivation | Today's dispatched-members record, notification_preferences.nightly_nudge | A member's eligibility for today's correction is exactly "recorded as having received today's original nudge" -- fixed at nudge-dispatch time, never re-evaluated against the member's current preference or a fresh eligibility pass at correction time | N/A |
| Device-vs-in-app-card channel fallback | notification_preferences.nightly_nudge, device-notification availability | For a member who passes eligibility, deliver by push if device-notification availability is true; otherwise the in-app "Tonight" card is the delivery surface. Email is never used for this notification or its correction, by product decision (Feature Dependency Map, External Touchpoints) | N/A |
| Delivery-never-blocks-plan guarantee | notification_preferences.nightly_nudge, Planned Meal status | A Planned Meal's status and in-app visibility are set entirely by FEAT-03, FEAT-23, FEAT-04, or FEAT-02's processes and never read or gated by this spec's eligibility, cap, or channel outcomes; a household always sees tonight's dinner in-app immediately, independent of whether any nudge or correction is ever sent or delivered | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Toggle own nightly-nudge preference | Maya (Organiser) | Always, on her own Member Profile | -- |
| Toggle own nightly-nudge preference | Sam (Other Adult Member) | Always, on his own Member Profile only | -- |
| Toggle own nightly-nudge preference | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile type, so no control is ever reachable |
| Toggle own nightly-nudge preference | Jordan (older kid, limited login -- Later) | Never | Notification Prefs is None for this row (Access Matrix); no toggle is shown for this notification |
| Toggle another member's nightly-nudge preference | Maya (Organiser) | Never -- Household Setup Full does not extend to another adult's own notification choice | The preference control on any other member's profile is not editable by Maya; only that member controls it themselves (FEAT-01.SPEC-005) |
| Trigger the eligibility check and firing caps | System (invoked by FEAT-13.SPEC-001, FEAT-13.SPEC-003) | Always, once per daily-schedule or swap-applied signal | N/A -- invoked internally, not a user-facing action |
| Receive the nightly nudge | Maya, Sam | Only when Active, an adult member type, and their own nightly_nudge preference is on | The member simply receives nothing that day; no error or placeholder appears anywhere in the product |
| Receive the nightly nudge | Jordan (young kid profile, no login -- MVP) | Never | No login exists; no surface on which a message could ever appear to this profile |
| Receive the nightly nudge | Jordan (older kid, limited login -- Later) | Never | Notification Prefs is None for this row; never sent regardless of the household's other settings |
| Receive the nightly nudge | Riley (Operator, support) | Never | Riley's Notification Prefs access is None; support access is read-only and carries no notification channel of its own |
| Receive the same-day correction | Maya, Sam | Only when recorded as having received today's original nudge | The member simply receives no correction; a member who never received today's original nudge is never owed one |
| Receive the same-day correction | Jordan (both rows), Riley | Never | Same exclusions as the nightly nudge itself -- neither kid row nor Riley is ever a recipient of any message this feature sends |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_preferences.nightly_nudge | Defaults to on for every newly created adult Member Profile | On member creation (FEAT-01.SPEC-005) | Yes -- each adult controls only their own toggle thereafter |
| Effective delivery surface per eligible member | Derived: push when device-notification availability is true and FEAT-07.SPEC-005's capability is up system-wide; the in-app "Tonight" card otherwise | Evaluated fresh at each dispatch (FEAT-13.SPEC-001 for the nudge; FEAT-13.SPEC-003 for the correction) | No -- a member does not directly choose their surface; it follows from their device state and the capability's own availability |
| Household-day nudge firing cap | Derived: set the first time a daily-schedule evaluation for a given household and date is processed by FEAT-13.SPEC-001 | Set once, at first successful processing for that household-date | No -- the cap cannot be cleared by any user action; only the next day's evaluation produces a new, distinct firing opportunity |
| Household-day correction firing cap | Derived: set the first time a swap-applied signal for a given household and date results in a dispatched correction (FEAT-13.SPEC-003) | Set once, at first successful correction dispatch for that household-date | No -- cannot be cleared by any user action; only the next day's nudge cycle produces a new, distinct correction opportunity |

## Business Rules

- At most one nudge per household per day, tied to that day's Planned Meal (Brief, Validation & Limits).
- A same-day swap triggers at most one follow-up correction (Brief, Validation & Limits; XBR-09).
- XBR-13: Each member controls their own nightly-nudge preference, held on their Member Profile and set within household settings; neither kid row receives notifications.
- This rule is the single source of truth for FEAT-13.SPEC-001's and FEAT-13.SPEC-003's eligibility, cap, and channel-fallback logic; those automations call this rule rather than re-implementing any part of it.
- A member's ineligibility (preference off, wrong member type, or non-Active status) is never surfaced as a product error anywhere -- it is a silent, expected state, consistent with product-features.md's "Notification disabled" framing.
- Email is never used as a channel for this nudge or its correction, by product decision (Feature Dependency Map, External Touchpoints: "email is deliberately NOT used for the nudge -- the fallback is the in-app 'Tonight' card").
- A failed nudge or correction delivery never blocks the meal from being visible in-app (Brief, States: Error).

## Edge Cases

- **A member re-enables their nightly-nudge preference the same day the nudge has already dispatched to other members** -- No retroactive send occurs for that member today; the once-per-household-per-day cap has already been recorded, and the member is included only starting with tomorrow's cycle.
- **A member's preference and their household removal race each other (they toggle their preference and are removed by Maya within moments of the same dispatch)** -- FEAT-01.SPEC-005's own removal-vs-edit resolution (reject-with-refresh, per the dependency map's Member Profile Contention note) governs which change lands first; this spec's dispatch-time read simply reflects whichever state won that race.
- **Both Maya and Sam have their nightly-nudge preference off in a given day** -- Eligibility yields zero members; no nudge is dispatched, and both simply see tonight's dinner the next time they open the app, per the Business Rules' silent-ineligibility principle.
- **A Member Profile is still Invited (has not yet accepted) when the nudge time is reached** -- Not Active, so not eligible; once the invitation is accepted and the profile becomes Active, that member is evaluated starting with the next day's cycle, never retroactively for a day that already dispatched.
- **The device-notification delivery capability is unavailable for a household's entire dispatch window** -- Every otherwise-push-eligible member in that household resolves to the in-app "Tonight" card that day, per the channel-fallback cross-field rule; eligibility itself is unaffected, and no email fallback is ever substituted.
- **A member re-enables their preference after receiving no nudge today, then a same-day swap occurs** -- They are still not owed a correction: correction eligibility is "recorded as having received today's original nudge," which this member does not satisfy regardless of their current preference state.

## Acceptance Criteria

**FEAT-13.SPEC-005-AC-01:** Given Maya has her nightly-nudge preference on and is an Active adult member, when eligibility is evaluated for tonight, then she is included in the eligible-member list.

**FEAT-13.SPEC-005-AC-02:** Given Sam has turned his nightly-nudge preference off, when eligibility is evaluated, then he is excluded from the eligible-member list and receives no nudge.

**FEAT-13.SPEC-005-AC-03:** Given Jordan is a young-kid profile with no login, when eligibility is evaluated, then Jordan is excluded outright regardless of any preference value, since no such value can even exist for this profile type.

**FEAT-13.SPEC-005-AC-04:** Given Jordan is an older-kid limited login (Later phase), when eligibility is evaluated, then Jordan is excluded, since Notification Prefs is None for this row.

**FEAT-13.SPEC-005-AC-05:** Given a household-day's nudge firing cap is already recorded, when a second daily-schedule evaluation for that same household and date arrives, then eligibility evaluation is skipped entirely and no nudge is dispatched.

**FEAT-13.SPEC-005-AC-06:** Given Maya has device-notification availability and Sam does not, when channel resolution runs for both, then Maya resolves to push and Sam resolves to the in-app "Tonight" card in the same run.

**FEAT-13.SPEC-005-AC-07:** Given the device-notification delivery capability is unavailable system-wide when dispatch runs, when channel resolution runs for every eligible member, then all of them resolve to the in-app "Tonight" card that day, and none falls back to email.

**FEAT-13.SPEC-005-AC-08:** Given every channel resolution and dispatch for a household's day fails entirely, when the household opens the app, then tonight's Planned Meal is fully visible in-app, unaffected by any notification outcome.

**FEAT-13.SPEC-005-AC-09:** Given Maya attempts to change Sam's nightly-nudge preference from her own Member Profile screen, when she looks for a control to do so, then none exists -- only Sam's own profile exposes his toggle.

**FEAT-13.SPEC-005-AC-10:** Given a new adult Member Profile is created, when it is saved, then its nightly-nudge preference defaults to on.

**FEAT-13.SPEC-005-AC-11:** Given Sam re-enables his nightly-nudge preference after today's nudge has already dispatched to Maya, when the change is saved, then Sam receives no retroactive nudge for today.

**FEAT-13.SPEC-005-AC-12:** Given a Member Profile is still Invited (not yet Active) when the nudge time is reached, when eligibility is evaluated, then that profile is excluded from today's dispatch.

**FEAT-13.SPEC-005-AC-13:** Given Riley (Operator) has no Notification Prefs access, when eligibility is evaluated for any household, then Riley is never included as a recipient of the nudge or its correction.

**FEAT-13.SPEC-005-AC-14:** Given both Maya and Sam have their nightly-nudge preference off, when the nudge time is reached, then eligibility yields zero members and no dispatch occurs, with no error shown anywhere.

**FEAT-13.SPEC-005-AC-15:** Given Sam never received today's original nudge because his preference was off at nudge time, when he re-enables his preference before a same-day swap occurs, then he is still not owed the swap's correction, since correction eligibility is fixed to who actually received today's original nudge.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Prep-Reminder Derivation Rule

## Overview

**Name:** Prep-Reminder Derivation Rule
**ID:** FEAT-13.SPEC-006
**Type:** Logic/Rule
**Purpose:** Derives the prep-reminder text (or its confirmed absence) for tonight's dinner from the Planned Meal's recipe's early-prep requirements, so the nudge and its correction never invent a prep step that isn't there.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder
**Governed Entity:** Recipe (prep_requirements field), read through tonight's Planned Meal's recipe reference

## Scope and Non-Goals

**In Scope:**
- Reading a recipe's prep_requirements field and rendering it as the prep-reminder text
- The no-invented-prep-step disposition when prep_requirements is empty
- Who may view the derived text (via the nudge or the "Tonight" card) and who may edit its source field

**Non-Goals:**
- Editing a recipe's prep_requirements field -- owned by Recipe Library (Starter Recipes) (FEAT-08, content seeding) and Recipe Import from Web Link (FEAT-10, household edits to imported recipes); this spec only reads the field's current value at dispatch time.
- Deciding whether a nudge or correction fires at all, and who receives it -- owned by FEAT-13.SPEC-001 (Tonight's Nudge Trigger), FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger), and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec only supplies content, never audience.
- The exact message wording that wraps this derived text -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) and FEAT-13.SPEC-004 (Same-Day Swap Correction Message); this spec produces the {prep_reminder_text} placeholder value, not the surrounding sentence.

## Governed Entity

**Entity:** Recipe (prep_requirements field), read via tonight's Planned Meal.recipe reference
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| prep_requirements | text | Early-prep needs such as defrosting, entered on the recipe (starter library content or a household's imported recipe); used for the nightly nudge |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | Reads this derivation once per dispatch, to build tonight's nudge content |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | Reads this derivation once per dispatched correction, for the new recipe |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | Renders the derived text (or its absence) into the nudge's content |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Renders the derived text (or its absence) into the correction's content |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| prep_requirements | No validation beyond data type -- editorial content owned and validated by FEAT-08 (starter content) and FEAT-10 (imported-recipe edits, including the safety re-check FEAT-10 already requires); this spec applies no additional rule beyond reading its current value | Always | At each nudge or correction dispatch (read-only) | N/A -- this spec never rejects or flags a recipe's prep_requirements content; it only reads whatever value already exists | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Prep-reminder text derivation | Planned Meal.recipe, Recipe.prep_requirements | Resolve tonight's Planned Meal's recipe reference, then read that recipe's prep_requirements. If non-empty (after trimming leading/trailing whitespace), render it verbatim as the prep-reminder text. If empty or whitespace-only, the derivation yields no prep step at all -- never an invented placeholder | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the derived prep-reminder text (via the nudge, its correction, or the "Tonight" card) | Maya, Sam | Only when they are eligible recipients per FEAT-13.SPEC-005 | The member simply receives no prep-reminder content, along with no nudge or correction at all |
| View the derived prep-reminder text | Jordan (both rows), Riley | Never | Neither receives any surface this feature produces (FEAT-13.SPEC-005's Authorization Rules) |
| Edit a recipe's prep_requirements (source field) | Maya, Sam (Recipe Library: Full) | Only for imported recipes, through FEAT-10; starter-library recipes are read-only for households | Attempting to edit a starter recipe's prep_requirements has no control to act on -- starter content is read-only per the Feature Dependency Map's Recipe Contention note |
| Edit a recipe's prep_requirements | Jordan (both rows) | Never | No Recipe Library edit access exists for either kid row (Access Matrix: Recipe Library None or View) |
| Edit a recipe's prep_requirements | Riley (Operator, support) | Never | Riley's Recipe Library access is View only (Access Matrix) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| prep_reminder_text (derived, not stored) | If tonight's Planned Meal's recipe's prep_requirements is non-empty (after trimming), render it verbatim; otherwise the derivation yields no prep step and no text is invented | Computed fresh at each nudge or correction dispatch (FEAT-13.SPEC-001, FEAT-13.SPEC-003) | No -- this value is never directly editable; changing the outcome requires editing the recipe's prep_requirements itself, through FEAT-10 |

## Business Rules

- No prep step is ever invented: when prep_requirements is empty, the nudge and its correction name the meal only (product-features.md, States: "No prep needed" alternate).
- The derivation is read-only and non-destructive: it never writes back to Recipe or Planned Meal.
- prep_requirements editing follows FEAT-10's edit path for imported recipes and FEAT-08's content-seeding path for starter recipes, both external to this feature; this spec only consumes the field's current value at dispatch time.
- A mid-day edit to prep_requirements alone (with no accompanying swap) is not itself a trigger for any correction -- only a swap that changes the Planned Meal's recipe (FEAT-04.SPEC-004) triggers FEAT-13.SPEC-003; an edited prep step on the same, unswapped recipe stands as already dispatched for that day.

## Edge Cases

- **prep_requirements is edited on the recipe after tonight's nudge already dispatched, with no accompanying swap** -- No correction is issued for a same-recipe content edit alone; only a swap that changes the Planned Meal's recipe (FEAT-04.SPEC-004) triggers FEAT-13.SPEC-003. The originally dispatched prep text stands for that night.
- **prep_requirements is whitespace-only** -- Treated identically to empty: no prep step is rendered, and the derivation fails safe to the "no prep needed" disposition rather than showing blank or malformed text.
- **prep_requirements is present but exceptionally long** -- Rendered verbatim as authored; this spec applies no additional length rule of its own, since content length is governed by FEAT-08's and FEAT-10's own recipe-content rules, not by this derivation.
- **Tonight's Planned Meal's recipe reference no longer resolves (the recipe itself was removed)** -- Out of scope for this spec: FEAT-13.SPEC-001's own "no dinner planned tonight" disposition already covers a Planned Meal that cannot produce a valid nudge, since a Planned Meal's recipe field is required and its removal path (FEAT-02, FEAT-10) also removes or replaces the Planned Meal itself.

## Acceptance Criteria

**FEAT-13.SPEC-006-AC-01:** Given tonight's Planned Meal's recipe has prep_requirements "take the chicken out of the freezer," when this derivation runs, then the prep-reminder text renders as "take the chicken out of the freezer" verbatim.

**FEAT-13.SPEC-006-AC-02:** Given tonight's Planned Meal's recipe has no prep_requirements, when this derivation runs, then no prep-reminder text is produced and none is invented.

**FEAT-13.SPEC-006-AC-03:** Given tonight's Planned Meal's recipe's prep_requirements is whitespace-only, when this derivation runs, then it is treated as empty and no prep step is rendered.

**FEAT-13.SPEC-006-AC-04:** Given Maya is an eligible recipient of tonight's nudge, when she views it, then she sees whatever prep-reminder text (or its absence) this derivation produced for tonight's recipe.

**FEAT-13.SPEC-006-AC-05:** Given Jordan (young-kid profile) has no notification access at all, when the derivation's output is dispatched, then Jordan never sees it on any surface.

**FEAT-13.SPEC-006-AC-06:** Given Sam attempts to edit an imported recipe's prep_requirements through FEAT-10, when he saves the change, then the edit is accepted, since Sam has Recipe Library Full access.

**FEAT-13.SPEC-006-AC-07:** Given Maya attempts to edit a starter-library recipe's prep_requirements, when she looks for an edit control, then none exists, since starter content is read-only.

**FEAT-13.SPEC-006-AC-08:** Given a recipe's prep_requirements is edited the same day after tonight's nudge already dispatched, with no swap occurring, when the edit is saved, then no correction is issued and tonight's already-dispatched prep text stands.

**FEAT-13.SPEC-006-AC-09:** Given a same-day swap changes tonight's Planned Meal to a new recipe, when FEAT-13.SPEC-003 reads this derivation for the new recipe, then the correction's prep-reminder text reflects the new recipe's prep_requirements, independent of the original recipe's value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

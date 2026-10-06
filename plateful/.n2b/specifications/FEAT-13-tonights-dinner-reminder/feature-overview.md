---
document_type: feature-overview
feature_number: FEAT-13
feature_name: Tonight's Dinner Reminder
feature_slug: tonights-dinner-reminder
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 0
automation_count: 2
logic_rule_count: 2
integration_count: 0
notification_count: 2
---

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

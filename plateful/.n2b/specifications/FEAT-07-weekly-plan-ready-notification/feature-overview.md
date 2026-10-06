---
document_type: feature-overview
feature_number: FEAT-07
feature_name: Weekly Plan Ready Notification
feature_slug: weekly-plan-ready-notification
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 0
automation_count: 1
logic_rule_count: 2
integration_count: 2
notification_count: 1
---

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

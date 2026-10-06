---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-004
spec_name: Plan-Arrival Day & Time Setting Rule
spec_slug: plan-arrival-day-time-setting-rule
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 10
---

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

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-003
spec_name: Plan-Ready Delivery & Eligibility Rules
spec_slug: plan-ready-delivery-eligibility-rules
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 14
---

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

---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-008
spec_name: Notification Preference Rules
spec_slug: notification-preference-rules
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 6
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Notification Preference Rules

## Overview

**Name:** Notification Preference Rules
**ID:** FEAT-21.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces that only optional notifications can be switched off and that transactional record emails always send (XBR-30).
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (notification_preferences field), read against the Notification entity's type classification

## Scope and Non-Goals

**In Scope:**
- The optional/transactional classification lookup that determines which notification types can be toggled
- The toggle-write rule enforced whenever Nadia changes a preference on FEAT-21.SPEC-002
- Authorization for reading and changing notification preferences

**Non-Goals:**
- Which specific notification types exist and their content -- owned entirely by Notifications (Email) (FEAT-14); this spec only consumes FEAT-14's optional/transactional classification, it does not define it
- Delivery timing, batching, retry, or channel behavior for any notification -- owned by FEAT-14 and by each notification's own spec (e.g., FEAT-21.SPEC-011)
- Field validation for name, business details, and payment terms -- owned by FEAT-21.SPEC-007, a separate rule set for a separate part of the Freelancer Account

## Governed Entity

**Entity:** Freelancer Account (notification_preferences field)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_preferences | derived (a set of per-type on/off values) | Optional notifications only; transactional record emails cannot be disabled (dependency map, Freelancer Account fields) |

**Referenced (read-only):** Notification -- `notification_type` and its optional-vs-transactional classification (dependency map, Referenced Entities: "the preferences screen reads the set of notification types ... to build the toggle list").

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-002 | Notification Preferences | On render (determines which rows show a toggle vs. a locked "Always sent" indicator) and on every toggle, before saving |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | -- (referenced, not enforcing) | Consults the saved preference at send time for optional types only; transactional types are never checked against this preference set at all |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_preferences (per optional type) | Value must be a boolean on/off | Always | On toggle | "Could not save. Try again." (generic save failure -- there is no invalid-value case reachable through the toggle UI) | Yes |
| notification_preferences (per transactional type) | No write path exists -- rejected before any save attempt | Always | On toggle attempt | Not applicable -- FEAT-21.SPEC-002 renders no interactive toggle for a transactional type, so no submission is ever produced for it | Yes (structurally, by omission of the control) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Transactional types are never disableable | notification_preferences, Notification.notification_type classification | A toggle write is accepted only when the targeted notification_type's classification (read from Notification) is Optional; a write targeting a Transactional type is rejected regardless of source | "Transactional emails cannot be turned off." (shown only if a write somehow targets a transactional type outside the normal screen path; the screen itself never exposes this control) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View notification preferences | Nadia (Freelancer) | Always, her own account only | -- |
| Toggle an optional notification type | Nadia (Freelancer) | Always, her own account only | -- |
| Toggle a transactional notification type | Nadia (Freelancer) | Never | No toggle control is rendered for transactional types; the row shows a locked "Always sent" indicator instead |
| View notification preferences | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Toggle any notification type | Dana (Support Operator) | Never | Every toggle -- transactional or optional -- renders as a static, disabled indicator; a direct attempt is refused with "Support sessions are read-only." |
| View or toggle any notification type | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings |
| View or toggle any notification type | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_preferences (each optional type) | Defaults to On at account creation (FEAT-20) and for any new optional type FEAT-14 introduces afterward | On create, and when a new optional type is introduced | Yes -- Nadia can turn any optional type off or back on at any time on FEAT-21.SPEC-002 |
| notification_preferences (each transactional type) | Fixed On -- not a stored preference, since it cannot vary | Always | No |

## Business Rules

- XBR-30 is the authority for this spec's central rule: "Notification preferences can switch off only optional emails; transactional emails core to the record ... always send; delivery failures are surfaced to the freelancer as warnings on the affected project" -- the delivery-failure-warning half of XBR-30 is FEAT-14's responsibility, not this spec's.
- A saved preference change is evaluated by FEAT-14 at the moment each individual notification would be sent, not at the moment the preference was changed -- a toggle change never affects a notification already queued or sent before the toggle was saved.
- This spec's classification lookup (Optional vs. Transactional) is authoritative for FEAT-21.SPEC-002's rendering; FEAT-21.SPEC-002 never invents its own classification.
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules.

## Edge Cases

- **A notification type FEAT-14 previously classified as Optional is later reclassified as Transactional** -- Any existing off preference for that type is discarded (transactional types have no off state); the row moves from the Optional group to the locked Transactional group on FEAT-21.SPEC-002's next load.
- **A notification type FEAT-14 previously classified as Transactional is later reclassified as Optional** -- The type gains a toggle, defaulting to On, on FEAT-21.SPEC-002's next load, per the Defaults and Derivations row above.
- **Nadia toggles an optional preference in one open session while a second of her sessions has the same screen open** -- The second session reflects the new value on its next refresh; each toggle is its own field-level write, so no merge conflict arises (per FEAT-21.SPEC-002's own Edge Cases).
- **A malformed or forged toggle request targets a transactional type directly (bypassing the screen's UI)** -- Rejected per the Cross-Field Rule above: "Transactional emails cannot be turned off."; the account's transactional types remain On.
- **Nadia turns an optional type off, and a notification of that type was already queued for delivery before the toggle saved** -- The already-queued notification is unaffected (FEAT-14 evaluates the preference at send time for notifications not yet queued); only notifications queued after the toggle saves respect the new Off state.

## Acceptance Criteria

**FEAT-21.SPEC-008-AC-01:** Given Nadia views the Notification Preferences screen, when she looks at a Transactional notification type, then it shows locked-on with no toggle control.

**FEAT-21.SPEC-008-AC-02:** Given Nadia toggles an Optional notification type off, when the toggle saves, then FEAT-14 will not send that type to her going forward until she turns it back on.

**FEAT-21.SPEC-008-AC-03:** Given Nadia toggles an Optional notification type back on, when the toggle saves, then FEAT-14 resumes sending that type to her.

**FEAT-21.SPEC-008-AC-04:** Given a forged or malformed request attempts to toggle a Transactional type off, when this rule evaluates the request, then it is rejected with "Transactional emails cannot be turned off." and the type remains On.

**FEAT-21.SPEC-008-AC-05:** Given FEAT-14 reclassifies a previously Optional type as Transactional, when Nadia's screen next loads, then that type appears in the locked Transactional group with no toggle, regardless of its prior off/on preference.

**FEAT-21.SPEC-008-AC-06:** Given FEAT-14 introduces a new Optional type, when Nadia's screen next loads, then the new type appears with a toggle defaulted to On.

**FEAT-21.SPEC-008-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views notification preferences, then every toggle -- Transactional or Optional -- is a static, disabled indicator, and a direct change attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-008-AC-08:** Given Owen (Client Primary Contact) attempts to reach the Notification Preferences screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-008-AC-09:** Given a notification of an Optional type was already queued before Nadia turns that type off, when the toggle saves, then the already-queued notification is still delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

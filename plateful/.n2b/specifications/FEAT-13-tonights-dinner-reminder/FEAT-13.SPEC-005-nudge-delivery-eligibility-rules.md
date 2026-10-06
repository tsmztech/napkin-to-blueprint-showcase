---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-005
spec_name: Nudge Delivery & Eligibility Rules
spec_slug: nudge-delivery-eligibility-rules
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 25
acceptance_criteria_count: 15
---

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

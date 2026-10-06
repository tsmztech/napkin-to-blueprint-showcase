---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-24.SPEC-006
spec_name: Household Referral Rules
spec_slug: household-referral-rules
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Household Referral Rules

## Overview

**Name:** Household Referral Rules
**ID:** FEAT-24.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day counting window, the already-has-a-household disposition, and who may see or act on Household Referral data.
**Parent Feature:** FEAT-24 -- Invite Another Household
**Governed Entity:** Household Referral

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every field on the Household Referral record
- The one reusable personal link per adult member limit
- Single-attribution (a new household attributed to at most one referring household, ever) and no-self-referral
- The 30-day counting window between a link's most recent open and the new household's creation
- The already-has-a-household disposition for a visitor who opens a link while already a member of any household
- Authorization for every action the product defines on the Household Referral record and the personal referral link, per role

**Non-Goals:**
- The step-by-step processing that creates the Household Referral record -- owned by FEAT-24.SPEC-004 (Household Referral Recording), which enforces the rules defined here
- The step-by-step processing that updates the `upgraded` field -- owned by FEAT-24.SPEC-005 (Referral Upgrade Tracking), which enforces the rule defined here
- Rewards, credits, or discounts tied to a referral -- excluded per scope-boundaries.md SC-10: this feature records referrals without paying for them
- General household-membership rules (invitations, organiser hand-over, leaving) -- owned by FEAT-09 (Household Invitations & Membership); this spec governs only household-to-household referral, a distinct mechanism per the Feature Breakdown Brief's Shared Context

## Governed Entity

**Entity:** Household Referral
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| referring_household | reference (Household) | The household whose member's link was used |
| referring_member_link | reference (Personal Referral Link) | The specific adult member's personal link that was opened |
| new_household | reference (Household) | The household created from the link |
| created_date | date | The date the new household completed setup |
| upgraded | boolean | Whether the new household's Subscription has gone on to become paid |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-001 | Invite Another Household Screen | Authorization on screen entry (Household Referrals column); one-link-per-member limit relied on for display |
| FEAT-24.SPEC-002 | Referral Welcome Screen | Already-has-a-household disposition on load; link-open moment recorded for the counting window |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | One-link-per-member limit enforced at creation time; authorization on who may trigger provisioning |
| FEAT-24.SPEC-004 | Household Referral Recording | Single-attribution, no-self-referral, and 30-day counting window enforced at record-creation time |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | `upgraded` field derivation enforced as this automation's exclusive write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| referring_household | Required; must reference an existing Household | Always | On record creation (FEAT-24.SPEC-004) | N/A -- system-only field, no user-facing form exists for this record; creation is refused silently and no referral is recorded (see Cross-Field Rules) | Yes |
| referring_member_link | Required; must reference an existing personal referral link owned by an adult member of referring_household | Always | On record creation | N/A -- system-only field; see above | Yes |
| new_household | Required; must reference an existing Household; no validation beyond that reference and the cross-field rules below | Always | On record creation | N/A -- system-only field; see above | Yes |
| created_date | Required; set automatically to the date the new household completed setup; no user input | Always | On record creation | N/A -- system-set field, never entered by any role | Yes |
| upgraded | Required; boolean; defaults to false | Always | On record creation, and on update by FEAT-24.SPEC-005 | N/A -- system-set field, never entered by any role | Yes |

This entity is created and updated entirely by automations (FEAT-24.SPEC-004, FEAT-24.SPEC-005); it exposes no user-facing entry form, so every field's "error message" is N/A in the sense of user-visible text -- an invalid combination instead results in no record being created, per the Cross-Field Rules and Business Rules below.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| No self-referral | referring_household, new_household | new_household must not equal referring_household | N/A -- enforced silently by FEAT-24.SPEC-004 stopping before record creation; the visitor sees the already-has-a-household disposition (FEAT-24.SPEC-002) rather than an error, since they never reach a state where self-referral could otherwise occur |
| Single attribution | new_household | A given new_household value may appear on at most one Household Referral record, ever, across the product's lifetime | N/A -- enforced silently by FEAT-24.SPEC-004; a second candidate referral for an already-attributed household is simply not recorded |
| 30-day counting window | created_date, referring_member_link (via its most-recent-open moment) | created_date must fall within 30 days (inclusive) of the referring_member_link's most recent open moment | N/A -- enforced silently by FEAT-24.SPEC-004; a household created outside the window is simply not recorded, with no error surfaced to the new household's own setup |
| One link per member | referring_member_link (via the owning Member Profile) | A given adult Member Profile owns at most one personal referral link, ever | N/A -- enforced at link-creation time (FEAT-24.SPEC-003), which returns the existing link rather than creating a second one; no error state exists because a second creation attempt is never a failure, only a no-op |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Provision (create) own personal referral link | Maya (Organiser), Sam (Other Adult Member) | Always, for the requesting adult member's own link only | Control (the Invite Another Household Screen) is not shown at all to Jordan (young kid profile, no login), Jordan (older kid, limited login), or Riley (Operator) -- the Household Referrals column of the Access Matrix gives each of these rows None |
| View own personal referral link and joined-families count | Maya (Organiser), Sam (Other Adult Member) | Always, for their own household's link and count | Same as above -- screen not shown to Jordan (either row) or Riley |
| View an individual Household Referral record's detail | No role | Never -- no such capability exists in the product | No screen or control anywhere exposes a single referral record's detail; only the aggregate count and the derived paying share are shown (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix, Read (single): N/A) |
| Create a Household Referral record directly | No role | Never -- creation happens only through FEAT-24.SPEC-004's automated attribution at household completion | No user-facing action exists to create a referral record; the record is written once by the system when a new household's setup completes (dependency map, Household Referral Contention) |
| Edit any field on an existing Household Referral record | No role | Never -- the record is written once and never edited by any role beyond the system's own `upgraded` update (FEAT-24.SPEC-005) | No screen or control exposes any edit path; the dependency map's Household Referral Contention note states the record is never edited by any role |
| Delete or archive a Household Referral record | No role | Never -- no deletion or purge mechanism is defined for this record (Feature Breakdown Brief, Non-Goals) | No screen or control exposes a delete path; retention is indefinite, as the product's only record of referral-driven growth |
| View the referral welcome page (personal referral link, unauthenticated) | Unauthorized visitor | Only for a link addressed to an existing personal referral link; no household session active | -- (this is the intended, unrestricted entry point for the feature) |
| Start household setup from a referral welcome page | Unauthorized visitor | Only when the visitor is not already signed in as a member of any household | A visitor already signed in as a member of any household sees "You already have a household on Plateful." instead of a start-setup option, and the setup path is not offered |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| created_date | The date the new household's setup completes (FEAT-01.SPEC-003 success) | On record creation only | No |
| upgraded | false | On record creation | No (only ever changed by FEAT-24.SPEC-005's system write, never by any role) |
| referring_member_link's most-recent-open moment | The moment the referral welcome page (FEAT-24.SPEC-002) most recently resolved that link for a given visit | Recalculated on every fresh open of a still-valid link | No (system-derived; not user-editable) |

## Business Rules

- XBR-20: a new household set up from an invite link within 30 days is attributed to at most one referring household, never itself; someone who already has a household is told so and nothing is recorded.
- The counting window is 30 days, inclusive of day 30, measured from the referring link's most recent open to the new household's creation date -- a household created on day 31 or later falls outside the window.
- One reusable personal link exists per adult member for the life of their account; the link is never expired or rotated by this feature (product-features.md, Validation & Limits).
- The joined-families count and the derived paying share (shown on FEAT-24.SPEC-001) are computed from Household Referral records where referring_household matches the viewing member's household -- not filtered to the viewing member's own referring_member_link alone, since the Access Matrix grants both adult roles Full visibility of the household's referral results as a whole.
- No money, reward, or credit is exchanged for a referral (scope-boundaries.md SC-10); this spec governs attribution and visibility only.

## Edge Cases

- **A household's only adult member leaves and a different adult later becomes the sole adult** -- The departing member's personal referral link is unaffected by this spec; it belongs to that Member Profile, not to the household, and Household Referral records already attributed against that member's link remain unchanged (they attribute to referring_household, which persists independent of individual membership changes).
- **Two adult members of the same household each have their own personal referral link, and both are used to refer different new households in the same week** -- Each new household's Household Referral record independently records its own referring_member_link (Sam's or Maya's); the household's joined-families count on FEAT-24.SPEC-001 sums both, since the count aggregates by referring_household.
- **A visitor's session carries stale referral context after navigating away from the welcome page and back through browser history** -- The 30-day window is measured from the most recent resolved open of the link (FEAT-24.SPEC-002's own re-open behavior), so returning through history re-triggers a fresh resolution rather than reusing a stale timestamp.
- **The referring household is deleted after a referral has already been recorded (upgraded or not)** -- The existing Household Referral record is unaffected by this spec; cascade behavior for a referenced household's deletion is FEAT-18's responsibility, flagged in the Feature Breakdown Brief's Cross-Feature Touchpoints and not resolved here.
- **A household is created exactly at the 30-day boundary while the referring link was opened at a different time of day** -- The window compares calendar dates (created_date vs. the link's open date), inclusive of day 30, so time-of-day differences within the same calendar dates do not affect eligibility.
- **An adult member requests link creation twice in immediate succession before the first request has completed** -- The one-link-per-member limit's check-then-create step (enforced in FEAT-24.SPEC-003) ensures the second request finds the first request's link already persisted, rather than creating a duplicate.

## Acceptance Criteria

**FEAT-24.SPEC-006-AC-01:** Given a new household completes setup exactly 30 days after its referral link was opened, when FEAT-24.SPEC-004 evaluates it, then the referral is recorded, since the window is inclusive of day 30.

**FEAT-24.SPEC-006-AC-02:** Given a new household completes setup 31 days after its referral link was opened, when FEAT-24.SPEC-004 evaluates it, then no referral is recorded.

**FEAT-24.SPEC-006-AC-03:** Given a visitor opens their own household's own personal link, when they view FEAT-24.SPEC-002, then they see "You already have a household on Plateful." and no start-setup option, and no self-referral is ever recorded.

**FEAT-24.SPEC-006-AC-04:** Given a new household already carries a Household Referral record, when a second referral context reaches FEAT-24.SPEC-004 for the same new household, then no second record is created.

**FEAT-24.SPEC-006-AC-05:** Given Sam already has a personal referral link, when FEAT-24.SPEC-003 is triggered again for him, then his existing link is returned and no second link is created.

**FEAT-24.SPEC-006-AC-06:** Given Maya (Organiser) is signed in, when she looks for the Invite Another Household Screen, then it is available to her, per the Household Referrals column of the Access Matrix.

**FEAT-24.SPEC-006-AC-07:** Given Jordan (young kid profile, no login) has no path to sign in, when any attempt is made to reach the Invite Another Household Screen, then no such attempt is possible, since this role has no login at all.

**FEAT-24.SPEC-006-AC-08:** Given Riley (Operator, support) is signed in for a support session, when Riley looks for the Invite Another Household Screen or any referral data, then none is shown, per the Household Referrals column of the Access Matrix giving Riley None.

**FEAT-24.SPEC-006-AC-09:** Given Maya wants to see a single referral record's detail, when she looks at the Invite Another Household Screen, then no such detail view exists anywhere in the product -- only the aggregate count and derived paying share are shown.

**FEAT-24.SPEC-006-AC-10:** Given a Household Referral record exists, when any role attempts to edit or delete it directly, then no control for doing so exists anywhere in the product.

**FEAT-24.SPEC-006-AC-11:** Given a referred household's Subscription becomes paid, when FEAT-24.SPEC-005 updates the matching record, then `upgraded` is set to true, and no role can set this field directly themselves.

**FEAT-24.SPEC-006-AC-12:** Given Sam's household has referrals from both Maya's link and Sam's own link, when the joined-families count is computed for FEAT-24.SPEC-001, then it sums referrals from both links, since the count aggregates by referring_household.

**FEAT-24.SPEC-006-AC-13:** Given a household is deleted after having referred another household, when the existing Household Referral record is inspected, then it remains unchanged by this spec, since cascade behavior on deletion is FEAT-18's responsibility.

**FEAT-24.SPEC-006-AC-14:** Given two adult members of the same household each hold their own personal referral link, when both links are used to refer different new households, then two separate Household Referral records are created, one per new household, both attributing to the same referring household.

**FEAT-24.SPEC-006-AC-15:** Given an unauthorized visitor (not signed in, no household) opens a valid personal referral link, when the welcome page loads, then they are able to proceed to start-setup, since this role and state combination is the intended, unrestricted entry point for the feature.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

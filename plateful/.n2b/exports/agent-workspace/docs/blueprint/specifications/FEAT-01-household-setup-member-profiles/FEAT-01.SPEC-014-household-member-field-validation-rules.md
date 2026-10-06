---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-014
spec_name: Household & Member Field Validation Rules
spec_slug: household-member-field-validation-rules
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 16
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Household & Member Field Validation Rules

## Overview

**Name:** Household & Member Field Validation Rules
**ID:** FEAT-01.SPEC-014
**Type:** Logic/Rule
**Purpose:** Governs household name, member cap, budget amount, and kid-profile data-minimality validation across every setup screen, plus account-level sign-in field rules.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Household and Member Profile

## Scope and Non-Goals

**In Scope:**
- Field-level validation for household_name, weekly_budget, and the account sign-in fields (email, password) captured during account creation
- The 12-member household cap
- Member Profile field validation: display_name, age_band, and the kid-profile data-minimality constraint
- Authorization rules for creating and editing Household and Member Profile records

**Non-Goals:**
- Dietary Rule validation (allergen selection, strength, removal confirmation gate) -- governed entirely by FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules)
- Locale fields (unit_system, currency, aisle_names) and plan_arrival_day_time -- these Household fields are validated and owned by FEAT-16 and FEAT-07 respectively, per the dependency map's note that FEAT-01 does not update them
- Weekly schedule field validation -- the weekly_schedule field carries no validation beyond data type (an empty schedule is a valid, deliberate statement), as established in FEAT-01.SPEC-008

## Governed Entity

**Entity:** Household and Member Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| household_name | text | The household's display name |
| organiser | reference | The one Member Profile holding the organiser role |
| weekly_budget | number | Rough weekly food budget, a positive amount in the household's currency |
| weekly_schedule | derived | Which nights are short on time and their time limit |
| unit_system | enum | Governed by FEAT-16, not this spec |
| currency | enum | Governed by FEAT-16, not this spec |
| aisle_names | text | Governed by FEAT-16, not this spec |
| plan_arrival_day_time | derived | Governed by FEAT-07, not this spec |
| status | enum | Active or Closed/Deleted; set only by FEAT-18, not this spec |
| display_name | text | First name or nickname (Member Profile) |
| member_type | enum | Organiser, Other Adult Member, young kid profile, or older kid limited login |
| sign_in | credential | Email and protected sign-in for adults only |
| age_band | enum | Kid profiles only |
| parental_consent_confirmation | boolean | Kid profiles only; required to create a kid profile |
| notification_preferences | derived | Per member: plan-ready on/off, nightly nudge on/off (adults only) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-001 | Account Sign-Up & Sign-In | On field blur and form submit for email and password; authorization is implicit (pre-authentication screen) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On field blur and form submit for household_name; authorization on screen entry (organiser only) |
| FEAT-01.SPEC-004 | Member List & Add Member | On "Add" action attempt, for the 12-member cap; authorization on screen entry (Add/Invite hidden from Sam) |
| FEAT-01.SPEC-005 | Member Profile Detail | On field blur and form submit for display_name, age_band; authorization on screen entry and save |
| FEAT-01.SPEC-007 | Parental Consent Confirmation | On "Continue" tap, for parental_consent_confirmation |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On field blur and form submit for weekly_budget |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Account email (FEAT-01.SPEC-001) | Required, valid email format | Always | On blur and on submit | "Enter a valid email address" | Yes |
| Account password (FEAT-01.SPEC-001) | Required, must meet a minimum protection strength (not trivially guessable) | On account creation only | On blur and on submit | "Choose a password that is harder to guess" | Yes |
| household_name | Required, 1-60 characters | Always | On blur and on submit | "Household name must be between 1 and 60 characters" | Yes |
| weekly_budget | If provided, must be a positive number greater than zero | Optional during partial setup; the rule applies only when a value is entered | On blur and on submit | "Enter an amount greater than zero" | Yes |
| display_name (Member Profile) | Required, 1-60 characters | Always | On blur and on submit | "Enter a name (up to 60 characters)" | Yes |
| member_type | Required, set once at creation and immutable afterward | Always | On creation only | N/A -- no editable field, no error state reachable after creation | Yes (at creation) |
| age_band | Required for kid profiles only | member_type is a kid profile | On blur and on submit | "Select an age band" | Yes |
| age_band | No validation beyond data type | member_type is an adult (Organiser or Other Adult Member) | -- | -- | -- |
| parental_consent_confirmation | Must be explicitly confirmed (checkbox checked) before a kid profile can be created | member_type is a kid profile | On the "Continue" action in FEAT-01.SPEC-007 | N/A -- the disabled "Continue" button prevents an invalid submission rather than surfacing an error | Yes |
| notification_preferences | No validation beyond data type -- any combination of on/off is valid | Always | -- | -- | No |
| organiser | Exactly one Active Member Profile must hold this role at all times | Always | Enforced structurally: creation always assigns it to the account holder; changing it is owned entirely by FEAT-09 | N/A -- this spec never presents an organiser-selection control; XBR-15 governs the hand-over path | Yes (structurally) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| 12-member household cap | Member Profile (count), Household | The total count of Active Member Profiles for a household cannot exceed 12 | "Your household already has 12 members, the most Plateful supports right now." |
| Kid-profile data minimality | member_type, display_name, age_band, parental_consent_confirmation | When member_type is a kid profile, no field beyond display_name, age_band, dietary rules, and parental_consent_confirmation may be captured or stored -- no surname, birth date, photo, or contact detail field exists for this member_type at all | N/A -- enforced by omission: no such field is ever presented or accepted for a kid profile, so no error state is reachable |
| Kid profile requires prior consent | member_type, parental_consent_confirmation | A Member Profile cannot be created with member_type set to a kid profile unless parental_consent_confirmation is already true, set by FEAT-01.SPEC-007 before this screen is reached | "A kid profile needs parent or guardian confirmation before it can be saved" -- shown only in the defensive case of a kid-profile creation attempted without having passed through FEAT-01.SPEC-007 |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create account | Maya, Sam (any adult, pre-authentication) | Always | -- |
| Create household | Maya (Organiser) | Only once per account (scope-boundaries.md SC-03); only for an account with no existing household | An account that already has a household is routed directly to FEAT-01.SPEC-010 rather than shown this screen at all |
| Edit household name | Maya (Organiser) | Always | Edit entry point not shown to Sam; a direct attempt shows "Only the organiser can change this" (FEAT-01.SPEC-010) |
| View household name | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| View household name | Riley (Operator, support) | Only through the separate read-only support view (FEAT-22), never directly | Riley never reaches this screen |
| View household name | Jordan (young kid, no login), Jordan (older kid, Later) | Never | N/A -- no login exists (young kid) or household setup is outside the older-kid login's entitlements |
| Add member (adult or kid) | Maya (Organiser) | Household has fewer than 12 Active members | Add actions remain visible but the create screen shows the 12-member-cap message when the cap is reached; Sam never sees Add controls at all |
| View member list | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Edit a member's own display name/age band/type details | Maya (Organiser) | Any member | -- |
| Edit a member's own display name/age band/type details | Sam (Other Adult Member) | Never | Edit controls not shown; a direct attempt shows "Only the organiser can change member details" |
| Edit own notification preferences | Maya, Sam (each, their own) | Always, own record only | Attempting to edit another member's preference is not possible -- the control is not exposed for any profile but the signed-in adult's own |
| Set weekly_budget | Maya (Organiser) | Always | Screen not shown to Sam |
| View weekly_budget | Maya (Organiser), Sam (Other Adult Member) | Always | -- |
| Confirm parental consent | Maya (Organiser) | Always, and required before any kid profile creation | This confirmation cannot be granted by any other role; Sam has no entry point to FEAT-01.SPEC-007 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Household.organiser | The account holder creating the household | On household creation | No -- organiser status changes only through FEAT-09's hand-over |
| Household.status | Active | On creation | No |
| Member Profile.status | Active | On creation | No |
| Member Profile.member_type | Organiser, for the household's first (creating) member | On household creation | No |
| Household.weekly_budget | Unset (no value) until the organiser enters one | On creation, and until FEAT-01.SPEC-008 is completed | Yes -- the organiser sets or changes it at will |

## Business Rules

- The 12-member cap exists "comfortably above the brief's 2-6 people" (this feature's Validation & Limits), so it is intentionally generous rather than a tight operational constraint.
- Kid-profile data minimality is a compliance requirement, not a product preference: it enforces the children's-privacy-class protection described in assumptions-constraints.md ASMP-26/ASMP-27, and no downstream feature may add a field to the kid-profile data set that this spec does not already list.
- An allergy or religious rule, once entered on any member, is governed by FEAT-01.SPEC-015's removal confirmation gate, not by this spec -- this spec covers only the fields listed above.
- Field validation always runs before any cross-feature side effect (e.g., FEAT-01.SPEC-011's Subscription provisioning) -- a Household record is never created with an invalid household_name.

## Edge Cases

- **Household name entered as exactly 60 characters** -- Passes validation; 61 characters shows the length error.
- **Household name entered as only whitespace** -- Treated as empty; the "1-60 characters" required error applies, since whitespace-only input carries no meaningful household name.
- **Weekly budget entered as a very large number (e.g., far beyond any realistic grocery spend)** -- Passes validation; this spec sets no upper bound, since a rough guide has no meaningful ceiling worth blocking on.
- **Household at exactly 12 members, organiser attempts to add a 13th** -- Blocked with the exact cap message; the household's existing 12 members are unaffected.
- **A member profile is removed (FEAT-18), bringing the count below 12, then a new member is added** -- The 12-member cap is evaluated fresh at each add attempt against the current Active count, not any historical high-water mark.
- **Organiser attempts to set an age band on an adult profile via a manipulated request (defensive case)** -- age_band is not a field the interface ever presents for adult member_types; if such a value were somehow submitted, it is ignored rather than stored, since the field has no meaning outside kid profiles.
- **Kid-profile creation attempted with parental_consent_confirmation not yet true (defensive case, e.g., a stale or bypassed flow)** -- Blocked with "A kid profile needs parent or guardian confirmation before it can be saved"; the profile is not created.

## Acceptance Criteria

**FEAT-01.SPEC-014-AC-01:** Given Maya enters "T" as her household name (1 character), when she blurs the field, then no error appears, since 1 character satisfies the minimum.

**FEAT-01.SPEC-014-AC-02:** Given Maya enters a household name of 61 characters, when she blurs the field, then the error "Household name must be between 1 and 60 characters" appears.

**FEAT-01.SPEC-014-AC-03:** Given Maya enters a household name of only spaces, when she blurs the field, then the same length/required error appears, since whitespace-only input is treated as empty.

**FEAT-01.SPEC-014-AC-04:** Given Maya enters "0" for weekly_budget, when she blurs the field, then the error "Enter an amount greater than zero" appears.

**FEAT-01.SPEC-014-AC-05:** Given Maya leaves weekly_budget empty, when she proceeds, then no error appears, since the field is optional during partial setup.

**FEAT-01.SPEC-014-AC-06:** Given Maya's household has 12 Active members, when she attempts to add a 13th, then the error "Your household already has 12 members, the most Plateful supports right now." appears and no new member is created.

**FEAT-01.SPEC-014-AC-07:** Given a member is removed via FEAT-18, bringing the household to 11 members, when Maya adds a new member, then it succeeds, since the cap is evaluated against the current count.

**FEAT-01.SPEC-014-AC-08:** Given Maya is creating a kid profile, when she looks for a surname, birth date, photo, or contact field, then none exists on the screen at all.

**FEAT-01.SPEC-014-AC-09:** Given Maya attempts to save a kid profile without having completed FEAT-01.SPEC-007's confirmation (defensive case), then the save is blocked with "A kid profile needs parent or guardian confirmation before it can be saved."

**FEAT-01.SPEC-014-AC-10:** Given Sam (Other Adult Member) views a member's profile, when he looks for edit controls on display_name or age_band, then none are shown, and a direct edit attempt shows "Only the organiser can change member details."

**FEAT-01.SPEC-014-AC-11:** Given Maya (Organiser) edits any member's display name, when she saves, then the change is accepted regardless of which member it is.

**FEAT-01.SPEC-014-AC-12:** Given Sam edits his own notification preference toggle, when he saves, then only his own preference changes and Maya's is unaffected.

**FEAT-01.SPEC-014-AC-13:** Given a visitor enters an invalid email format when creating an account, when they blur the field, then the error "Enter a valid email address" appears.

**FEAT-01.SPEC-014-AC-14:** Given Maya attempts to create a second household on an account that already has one, then she is routed directly to FEAT-01.SPEC-010 and this creation screen is never shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 11 | 11 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

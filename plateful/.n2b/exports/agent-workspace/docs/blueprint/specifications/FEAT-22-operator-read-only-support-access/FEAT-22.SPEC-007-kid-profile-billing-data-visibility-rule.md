---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-007
spec_name: Kid Profile & Billing Data Visibility Rule
spec_slug: kid-profile-billing-data-visibility-rule
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 10
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Kid Profile & Billing Data Visibility Rule

## Overview

**Name:** Kid Profile & Billing Data Visibility Rule
**ID:** FEAT-22.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs what FEAT-22.SPEC-002 suppresses -- kid profile detail beyond a specific safety report's allergy facts, and billing detail beyond plan tier.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access
**Governed Entity:** Member Profile (kid-type rows) and Subscription

## Scope and Non-Goals

**In Scope:**
- Field-by-field visibility of Member Profile for kid-type rows within FEAT-22.SPEC-002
- Field-by-field visibility of Subscription within FEAT-22.SPEC-002
- The narrow exception that lets a specific allergy fact through when tied to an open safety-concern request
- Authorization for viewing each suppressed and non-suppressed field

**Non-Goals:**
- Visibility of adult Member Profile fields -- not restricted by this spec; adults' data is shown in full within FEAT-22.SPEC-002, since the Access Matrix's Kid Profile Data restriction applies only to kid rows
- Whether access opens at all -- owned by FEAT-22.SPEC-006 (Support Access Scope & Gating Rules); this spec governs only what is shown once access has already been permitted
- The organiser's own visibility into Member Profile or Subscription data -- excluded per the Access Matrix: Maya's Household Setup and Billing access are Full, entirely unaffected by this spec, which governs Riley's read-only view alone
- Payment-processing data itself (card details, transaction records) -- excluded per the dependency map's Subscription Data Sensitivity note: payment details are held by the payment-processing capability and are never modeled as a field this or any other spec's screens display to anyone but Maya

## Governed Entity

**Entity:** Member Profile (kid-type rows: young kid profile, no login -- MVP, and older kid, limited login -- Later) and Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| display_name | text | Kid member's first name or nickname |
| member_type | enum | Organiser, Other Adult Member, young kid profile, older kid limited login |
| sign_in | text | Adults only; not applicable to kid rows |
| age_band | enum | Kid profiles only |
| parental_consent_confirmation | boolean | Kid profiles only, the organiser's consent confirmation |
| notification_preferences | object | Adults only; not applicable to kid rows |
| status | enum | Invited, Active, Left, Removed |
| Dietary Rule (referenced, not a Member Profile field) -- rule_kind, strength, allergen | -- | The specific allergy fact this policy may narrowly admit |
| Subscription.tier | enum (free, paid) | The one Subscription field this policy admits |
| Subscription.billing_period | enum (monthly, yearly) | Suppressed |
| Subscription.billing_state | enum | Suppressed |
| Subscription.billing_history | list | Suppressed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-002 | Support Read-Only Household View | On every render of the Members and Subscription sections |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| display_name (kid rows) | Never rendered to Riley | Always | On render | N/A -- field is omitted, not shown with an error | -- |
| member_type (kid rows) | Never rendered to Riley as an identifying label; the Members section instead groups kid data anonymously under "a household kid member" | Always | On render | N/A -- no field-level error; this is a display-omission rule | -- |
| age_band | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| parental_consent_confirmation | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| notification_preferences (kid rows) | No validation beyond data type -- not applicable to kid rows, and adult rows are outside this policy's scope | Always | -- | -- | -- |
| status (kid rows) | Never rendered to Riley as part of an identifiable kid row | Always | On render | N/A -- field is omitted | -- |
| Dietary Rule (kid member's allergy, rule_kind = allergy) | Rendered only when it is the specific allergy the open safety-concern request's reported meal fails, and only as "a household kid member has an allergy to {allergen}" with no name attached | The open Support Request is kind = safety concern AND this Dietary Rule is the one the reported recipe's ingredient failed | On render | N/A -- field is conditionally omitted, not error-producing | -- |
| Dietary Rule (kid member's non-allergy rules: dislikes, vegetarian settings) | Never rendered to Riley | Always | On render | N/A -- field is omitted | -- |
| Subscription.tier | Always rendered to Riley | Always | On render | N/A -- not a restricted field | -- |
| Subscription.billing_period, billing_state, billing_history | Never rendered to Riley | Always | On render | N/A -- fields are omitted | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Allergy-fact admission is request-scoped | Support Request.kind, Support Request.planned_meal/recipe, Dietary Rule.allergen, Dietary Rule.rule_kind | A kid member's allergy fact is shown only when the open Support Request is a safety concern AND the allergen named matches the specific ingredient that failed the reported meal's safety check -- an allergy unrelated to the reported meal is never shown, even for the same kid, even during the same session | N/A -- this is a display-scoping rule, not a user-facing validation |
| Billing suppression is absolute | Subscription.tier, Subscription.billing_period, Subscription.billing_state, Subscription.billing_history | Only tier passes through; every other Subscription field is suppressed regardless of the open request's kind (safety concern or general support) -- there is no request type that widens billing visibility | N/A -- this is a display-scoping rule |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a kid member's display_name, age_band, parental_consent_confirmation, or status | Riley (Operator, support) | Never | Field is not rendered anywhere in FEAT-22.SPEC-002 -- omitted from the Members section entirely, not shown disabled or blurred |
| View a kid member's allergy fact tied to the open safety-concern request | Riley (Operator, support) | Only while a safety-concern Support Request naming that exact meal/recipe is open, and only as an anonymized fact ("a household kid member has an allergy to {allergen}") | Outside this condition (no safety-concern request open, or the allergy is unrelated to the reported meal): the fact is not rendered |
| View a kid member's non-allergy Dietary Rule (dislike, vegetarian setting) | Riley (Operator, support) | Never | Field is not rendered under any condition |
| View Subscription.tier | Riley (Operator, support) | Always, whenever FEAT-22.SPEC-002 is open | -- |
| View Subscription.billing_period, billing_state, or billing_history | Riley (Operator, support) | Never | Fields are not rendered anywhere in the Subscription section, regardless of the open request's kind |
| View adult Member Profile data (display_name, Dietary Rules, status) | Riley (Operator, support) | Always, whenever FEAT-22.SPEC-002 is open | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| "a household kid member has an allergy to {allergen}" (derived display line) | Derived by matching the open safety-concern Support Request's planned_meal/recipe against the household's kid Dietary Rules for the ingredient that caused the safety check to fail | Computed on every render of FEAT-22.SPEC-002 while a matching safety-concern request is open | No -- fully system-derived; Riley cannot request additional detail |

## Business Rules

- ASMP-26 and ASMP-27: children's-privacy-class protection applies to every kid Member Profile field; Riley may see kid allergy detail only inside a specific safety report, never a kid's broader profile -- this spec is the sole authority for what "inside a specific safety report" admits.
- The Access Matrix's Kid Profile Data column (Riley: None except the allergy details inside a specific safety report) and Billing column (Riley: View, plan tier only) are both implemented entirely by this spec -- no other spec in this feature independently restricts these fields.
- The admitted allergy fact is scoped to the single reported meal's failing ingredient, never the kid's full allergy list, even if the same kid has other allergies unrelated to the report.
- A general-support Support Request never admits any kid profile field, since the narrow allergy exception applies only to safety-concern requests.
- This spec's suppressions apply uniformly regardless of which household is open -- there is no household-specific configuration of what Riley may see.

## Edge Cases

- **The reported meal's safety check failed on more than one kid's allergy (a shared meal with two affected kids)** -- Each affected kid's allergy fact is shown separately, each anonymized as its own "a household kid member has an allergy to {allergen}" line, since the exception is scoped to the meal's failing ingredients, not to a single kid.
- **The household has a kid member whose allergy is unrelated to the reported meal** -- That kid's allergy is never shown, regardless of how the report is worded or how long the session remains open.
- **Riley resolves the safety-concern request while viewing the household** -- The admitted allergy fact remains visible for the remainder of the current session (the request's kind does not change on resolution), but a later session against a new, unrelated request for the same household re-evaluates the admission rule fresh and would not show it unless a new matching safety-concern request is open.
- **The household is on the free tier** -- Subscription.tier still renders as "Free"; the suppression of billing_period, billing_state, and billing_history applies identically regardless of tier.
- **A general support Support Request is open for a household that also has a separate, unrelated open safety-concern request** -- The allergy-fact admission is evaluated against whichever safety-concern request is open for that household, independent of the general-support request; if no safety-concern request is open, no allergy fact is admitted even though a different Support Request exists.
- **The kid's allergy rule is edited by the organiser (allergen changed, or the rule removed) while Riley's session is open** -- The admitted fact reflects the Dietary Rule as currently stored on each render; a rule change is picked up in the same manner as any other field this screen reads, per FEAT-22.SPEC-002's own state model (no live re-fetch mid-session, refreshed on reopen).

## Acceptance Criteria

**FEAT-22.SPEC-007-AC-01:** Given Riley opens a household with a young kid profile, when the Members section renders, then no display_name, age_band, parental_consent_confirmation, or status appears for that kid.

**FEAT-22.SPEC-007-AC-02:** Given a safety-concern request is open for a meal that failed a kid's peanut allergy, when Riley views the Members section, then "a household kid member has an allergy to peanuts" is shown with no name attached.

**FEAT-22.SPEC-007-AC-03:** Given the same kid also has an unrelated dairy allergy not implicated in the reported meal, when Riley views the Members section, then the dairy allergy is not shown.

**FEAT-22.SPEC-007-AC-04:** Given a kid member has a soft dislike recorded, when Riley views the Members section, then the dislike is never shown, regardless of any open request.

**FEAT-22.SPEC-007-AC-05:** Given the open request is a general support contact (not a safety concern), when Riley views the Members section, then no kid allergy fact is shown, since the exception applies only to safety-concern requests.

**FEAT-22.SPEC-007-AC-06:** Given the household is on the paid tier, when Riley views the Subscription section, then "Paid" is shown and no billing_period, billing_state, or billing_history appears.

**FEAT-22.SPEC-007-AC-07:** Given the household is on the free tier, when Riley views the Subscription section, then "Free" is shown with the same suppression of all other billing fields.

**FEAT-22.SPEC-007-AC-08:** Given a shared meal's safety check failed for two different kids' allergies, when Riley views the Members section, then both kids' allergy facts are shown, each anonymized separately.

**FEAT-22.SPEC-007-AC-09:** Given Riley views an adult member's data, when the Members section renders, then the adult's display_name and Dietary Rules are shown in full, since this spec restricts only kid rows.

**FEAT-22.SPEC-007-AC-10:** Given Riley resolves the open safety-concern request mid-session, when Riley continues viewing the same session, then the previously admitted allergy fact remains visible for the remainder of that session.

**FEAT-22.SPEC-007-AC-11:** Given a household has both an open general-support request and a separate open safety-concern request, when Riley views the Members section under either, then the allergy-fact admission is evaluated only against the open safety-concern request, independent of the general-support request.

**FEAT-22.SPEC-007-AC-12:** Given the organiser removes the kid's allergy rule while Riley's session is open, when Riley reopens the household in a later session, then the removed rule no longer appears, since the fact reflects the Dietary Rule as currently stored.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

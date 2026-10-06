---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-25.SPEC-005
spec_name: Check-In Validation & Access Rules
spec_slug: check-in-validation-access-rules
parent_feature: FEAT-25
parent_feature_name: Weekly Waste & Spend Check-In
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 36
acceptance_criteria_count: 22
---

# Logic/Rule Spec: Check-In Validation & Access Rules

## Overview

**Name:** Check-In Validation & Access Rules
**ID:** FEAT-25.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the one-answer-per-week/latest-wins rule, spend validation, the editable window, and who may view or answer the check-in.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In
**Governed Entity:** Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for every field of the Waste & Spend Check-In entity
- The one-answer-per-week, latest-write-wins resolution when more than one adult submits for the same week
- The editable-until-next-week-opens window and what happens to a submission after that window closes
- Authorization: who may view or answer the check-in, per role, and the exact denied experience for every role that cannot
- Default values and derivations for the entity's own stored fields (status, locked)

**Non-Goals:**
- Computing change_against_starting_point or change_against_weekly_budget -- owned by FEAT-25.SPEC-004 (Check-In Trend Calculation); this spec governs the raw stored fields only
- Deciding when a new week opens or when the previous week is marked Skipped or locked -- owned by FEAT-25.SPEC-003 (Weekly Check-In Cycle); this spec defines the rule that a locked week cannot be edited, while that automation is the one that sets the lock
- Deletion or archival of any check-in record -- excluded per scope-boundaries.md SC-18: every past check-in answer is kept for the life of the household account and remains available after a downgrade to the free tier; this spec defines no delete action because the product defines none

## Governed Entity

**Entity:** Waste & Spend Check-In
**Source:** Feature Dependency Map (feature-overview.md's Shared Context)

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The Household this record belongs to |
| week | date/period | The calendar week this record covers |
| waste_amount | enum (none \| a little \| a lot) | The household's answer for how much food was thrown away that week |
| spend | number, optional | The household's rough grocery spend that week, in its configured currency |
| starting_point_waste | enum (none \| a little \| a lot) | The household's one-time baseline: how much it typically threw away before Plateful |
| starting_point_spend | number, optional | The household's one-time baseline: what it typically spent on groceries before Plateful |
| status | enum (Offered \| Answered \| Skipped) | The week's lifecycle state |
| locked | boolean | Whether the week's record can still be edited |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-001 | Weekly Check-In Card | On field blur (spend, typical spend) and on save (starting-point form); authorization on screen entry and on every submission |
| FEAT-25.SPEC-002 | Check-In Trend View | Authorization on screen entry |
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | Status and locked-flag transitions during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| household | No validation beyond data type -- system-assigned from the signed-in adult's household membership, never user input | Always | -- | -- | -- |
| week | No validation beyond data type -- system-assigned by FEAT-25.SPEC-003 when the week's record opens, never user input | Always | -- | -- | -- |
| waste_amount | Must be one of "none," "a little," or "a lot" | Required only if the adult chooses to answer this week's question -- the question itself remains optional per week | On submit (tapping an option saves it immediately) | "Choose an option for how much was thrown away this week." | Yes |
| spend | Must be a positive number (greater than zero) in the household's configured currency; no fixed upper bound is defined by the product, consistent with Household.weekly_budget's own unlimited-positive-amount rule (FEAT-01.SPEC-008) | Optional -- may be left blank regardless of whether waste_amount is answered | On blur | "Enter an amount greater than zero, or leave it blank." | Yes (only when a non-blank value fails the rule; blank is always valid) |
| starting_point_waste | Must be one of "none," "a little," or "a lot" | Required only at the household's first-ever answered check-in (no starting point exists yet for this household) | On submit of the first-use form | "Choose an option for how much your household typically threw away before Plateful." | Yes |
| starting_point_spend | Must be a positive number (greater than zero) in the household's configured currency | Optional, even during starting-point capture -- same optionality as the recurring week's spend field | On blur (first-use form) | "Enter an amount greater than zero, or leave it blank." | Yes (only when a non-blank value fails the rule) |
| status | No validation beyond data type -- system-derived by FEAT-25.SPEC-003 (Offered/Answered/Skipped transitions), never directly settable by a user | Always | -- | -- | -- |
| locked | No validation beyond data type -- system-derived by FEAT-25.SPEC-003, never directly settable by a user | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Starting point captured once | starting_point_waste, starting_point_spend | Once a household's Waste & Spend Check-In history includes a record with a starting point set, no later submission ever includes or alters starting_point_waste or starting_point_spend | N/A -- the fields are simply not presented after the starting point exists; this is not a rejected submission, since FEAT-25.SPEC-001 never offers the starting-point form again |
| Submission locked out after cutover | waste_amount, spend, locked | A submission for a given week is accepted only while that week's record has locked = false; once FEAT-25.SPEC-003 sets locked = true, no further submission for that week is accepted, regardless of who attempts it | "This week's check-in has closed." |
| Latest submission replaces the whole week's answer | waste_amount, spend | When a second adult submits for the same open week, the latest submission replaces waste_amount and spend together as a single unit -- never merged field by field with the earlier submission | N/A -- not an error; this is the intended one-answer-per-week/latest-wins resolution, silently applied |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the check-in card and trend content | Maya (Organiser) | Always | -- |
| View the check-in card and trend content | Sam (Other Adult Member) | Always -- views the same shared household record Maya sees | -- |
| View the check-in card and trend content | Jordan (young kid profile, no login -- MVP) | Never | Card and trend content are not shown; a no-login profile has no access to any screen in the product |
| View the check-in card and trend content | Jordan (older kid, limited login -- Later) | Never | Card and trend content are not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| View the check-in card and trend content | Riley (Operator, support -- from v1) | Never | Card and trend content are not shown, even during an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley, keeping this personal household data outside operator visibility |
| Submit or resubmit the week's answer (waste_amount, spend) | Maya (Organiser) | Always, while the week's record has locked = false | Rejected with "This week's check-in has closed." once locked = true |
| Submit or resubmit the week's answer (waste_amount, spend) | Sam (Other Adult Member) | Always, while the week's record has locked = false -- per the Access Matrix's Full entitlement, matching Maya's, Sam's submission is accepted outright against the shared weekly record; it is not gated behind Maya's approval, and if Maya also submits the same week, whichever submission lands last is the one kept (Cross-Field Rules, above) | Rejected with "This week's check-in has closed." once locked = true |
| Submit or resubmit the week's answer | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; the action is unreachable |
| Submit or resubmit the week's answer | Jordan (older kid, limited login -- Later) | Never | Not offered in this role's app; the Access Matrix's Waste Check-In column is None for this role |
| Submit or resubmit the week's answer | Riley (Operator, support -- from v1) | Never | The read-only support role has no create or update entitlement on this entity anywhere in the product; the action is not offered at all |
| Set the household's one-time starting point (starting_point_waste, starting_point_spend) | Maya (Organiser) | Only at the household's first-ever answered check-in (no starting point set yet) | Not offered again once a starting point exists (Cross-Field Rules, above) |
| Set the household's one-time starting point | Sam (Other Adult Member) | Only at the household's first-ever answered check-in (no starting point set yet) -- Full entitlement, same basis as the weekly answer above | Not offered again once a starting point exists |
| Set the household's one-time starting point | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; the action is unreachable |
| Set the household's one-time starting point | Jordan (older kid, limited login -- Later) | Never | Not offered in this role's app |
| Set the household's one-time starting point | Riley (Operator, support -- from v1) | Never | The read-only support role has no create or update entitlement on this entity |
| Delete or archive a check-in record | All roles | Never -- no role may delete or archive any Waste & Spend Check-In record | The action does not exist in the product; every past record is retained for the life of the household account per scope-boundaries.md SC-18 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| household | Derived from the signed-in adult's own household membership | On create | No |
| week | Derived as the calendar week the record covers, assigned when FEAT-25.SPEC-003 opens the record | On create | No |
| status | Defaults to "Offered" when FEAT-25.SPEC-003 creates the record; becomes "Answered" the moment an adult submits waste_amount; becomes "Skipped" if FEAT-25.SPEC-003's cutover finds it still Offered | On create (Offered); on submit (Answered); on automation cutover (Skipped) | No direct override -- status is always system-derived from the actions above, never directly settable |
| locked | Defaults to false when the record is created (Offered) | On create; set to true by FEAT-25.SPEC-003 at the following week's cutover | No |

## Business Rules

- One answer per household per week: the latest submission from any adult with access to the entity replaces the entire prior submission for that week (waste_amount and spend together), never merged field by field.
- A week's record remains editable only until the following week's check-in opens (FEAT-25.SPEC-003); once locked, no further submission is accepted for that week.
- The starting point (starting_point_waste, starting_point_spend) is captured exactly once, at the household's first answered check-in, and is never altered afterward by any rule in this spec -- no editing path exists for it, per feature-overview.md's Key Capabilities and Non-Goals.
- No delete or archive action exists for any Waste & Spend Check-In record; every past record is retained for the life of the household account and remains visible after a downgrade to the free tier (scope-boundaries.md SC-18).
- Spend figures are entered and displayed in the household's currently configured currency (FEAT-16.SPEC-001); a later currency change converts historical spend figures for display (XBR-11, owned by FEAT-16) without this spec re-validating already-saved amounts.

## Edge Cases

- **Household's very first check-in is left unanswered** -- No starting point is set; the following week's card again renders the first-use, starting-point form (FEAT-25.SPEC-001), since starting_point_waste is still required and no reminder is sent (feature-overview.md's Non-Goals).
- **Two adults submit different answers for the same week within moments of each other** -- The submission that reaches the record last is kept in full; the adult whose submission was superseded sees no error, only the other's answer the next time the card loads, per the Cross-Field Rules' latest-wins resolution.
- **An adult attempts to submit after the week's record has already been locked** -- The submission is rejected with "This week's check-in has closed."; the card shows the locked answer read-only (FEAT-25.SPEC-001's States).
- **Spend entered as exactly the smallest currency unit above zero** -- Passes validation; any amount strictly greater than zero is valid, with no minimum beyond that.
- **Spend field left blank while waste_amount is answered** -- Valid; spend is optional independently of whether waste_amount was answered, and the record saves with spend absent.
- **Starting point's spend field left blank while starting point's waste is answered** -- Valid, same optionality as the recurring week's spend field.
- **Sam submits an answer, then Maya submits a different answer for the same still-open week** -- Maya's submission (the later one) is kept in full; this is not a denial of Sam's access, since his Full entitlement permits the submission itself -- it is simply superseded by a later submission, consistent with the one-answer-per-week/latest-wins rule applying identically regardless of which adult submits first or last.

## Acceptance Criteria

**FEAT-25.SPEC-005-AC-01:** Given Maya selects "A little" for this week's waste, when she saves, then waste_amount is recorded as "a little" with no error.

**FEAT-25.SPEC-005-AC-02:** Given Sam enters a spend amount of zero and blurs the field, then the field shows "Enter an amount greater than zero, or leave it blank." and the value is not saved.

**FEAT-25.SPEC-005-AC-03:** Given Sam leaves the spend field blank and saves the waste answer, then the record saves successfully with spend absent.

**FEAT-25.SPEC-005-AC-04:** Given Maya is on her household's first-ever check-in and selects "A lot" for typical waste but leaves typical spend blank, when she saves, then starting_point_waste is recorded as "a lot," starting_point_spend remains unset, and no error is shown.

**FEAT-25.SPEC-005-AC-05:** Given Maya is on her household's first-ever check-in and taps Save without selecting a typical-waste option, then the error "Choose an option for how much your household typically threw away before Plateful." appears and the save does not proceed.

**FEAT-25.SPEC-005-AC-06:** Given a household already has a starting point set, when Sam opens the check-in card, then no starting-point form is offered to him.

**FEAT-25.SPEC-005-AC-07:** Given Maya submitted this week's answer and Sam later submits a different answer for the same still-open week, when the record is next read, then Sam's submission (the later one) is the one shown, with no error to either adult.

**FEAT-25.SPEC-005-AC-08:** Given a week's record has locked = true, when Maya attempts to submit an answer for it, then the submission is rejected with "This week's check-in has closed."

**FEAT-25.SPEC-005-AC-09:** Given a week's record has locked = false and status Answered, when Sam resubmits a different waste_amount, then the resubmission succeeds and replaces the prior answer.

**FEAT-25.SPEC-005-AC-10:** Given Maya (Organiser) opens the week's plan, when the check-in card renders, then she can both view and act on it with no restriction.

**FEAT-25.SPEC-005-AC-11:** Given Sam (Other Adult Member) opens the week's plan, when the check-in card renders, then he can view the full shared record and submit or resubmit an answer, per his Full entitlement.

**FEAT-25.SPEC-005-AC-12:** Given Jordan is a young kid profile with no login, then no view of or action on this entity is reachable from any device of Jordan's.

**FEAT-25.SPEC-005-AC-13:** Given the older-kid limited login (Later) opens the week's plan, when they look for the check-in card, then it is not shown and no submission action is offered to that role.

**FEAT-25.SPEC-005-AC-14:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no check-in content is shown and no submission action exists for Riley anywhere in the product.

**FEAT-25.SPEC-005-AC-15:** Given any role attempts to find a delete or archive action for a check-in record, then no such action exists anywhere in the product for any role.

**FEAT-25.SPEC-005-AC-16:** Given a household's first check-in is left unanswered, when the following week's card renders, then it still shows the first-use, starting-point form, since starting_point_waste remains unset.

**FEAT-25.SPEC-005-AC-17:** Given Maya's household record has status Offered, when she submits a waste answer, then status becomes Answered.

**FEAT-25.SPEC-005-AC-18:** Given a household's record has status Offered and the following week's cycle runs with no answer ever submitted, then status becomes Skipped, per FEAT-25.SPEC-003.

**FEAT-25.SPEC-005-AC-19:** Given a household's record has status Answered and the following week's cycle runs, then locked becomes true and waste_amount and spend remain exactly as last submitted.

**FEAT-25.SPEC-005-AC-20:** Given Maya enters a spend amount at the smallest unit above zero, when she blurs the field, then the value passes validation and saves.

**FEAT-25.SPEC-005-AC-21:** Given Sam is on the starting-point form and leaves typical spend blank but selects a typical-waste option, when he saves, then the submission succeeds with starting_point_spend unset.

**FEAT-25.SPEC-005-AC-22:** Given a household's record already carries a starting point, when any adult opens a later week's check-in, then the submission form never re-offers the starting-point fields.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

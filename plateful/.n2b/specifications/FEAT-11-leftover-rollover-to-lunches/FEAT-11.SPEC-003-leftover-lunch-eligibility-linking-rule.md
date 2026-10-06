---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-11.SPEC-003
spec_name: Leftover Lunch Eligibility & Linking Rule
spec_slug: leftover-lunch-eligibility-linking-rule
parent_feature: FEAT-11
parent_feature_name: Leftover Rollover to Lunches
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 29
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Leftover Lunch Eligibility & Linking Rule

## Overview

**Name:** Leftover Lunch Eligibility & Linking Rule
**ID:** FEAT-11.SPEC-003
**Type:** Logic/Rule
**Purpose:** Determines which dinners produce leftover-worthy portions, which following day (no more than two days later) to suggest, and enforces the one-source/one-day link and every authorization rule on the leftover-lunch Planned Meal.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches
**Governed Entity:** Planned Meal (leftover-lunch sub-type)

## Scope and Non-Goals

**In Scope:**
- Field validation rules for the leftover-lunch Planned Meal's fields (meal_kind, linked source dinner, night, status)
- The eligibility determination that classifies a dinner as leftover-producing
- The following-day computation, including its two-day ceiling and collision fallback
- Cross-field rules enforcing the one-source/one-day link
- Authorization rules for every action on the leftover-lunch Planned Meal, per role
- Default values and derivations for every field

**Non-Goals:**
- Creating the leftover-lunch Planned Meal record -- owned by FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation), which calls this spec's rules but performs the actual write
- Re-evaluating an existing leftover lunch when its source dinner changes -- owned by FEAT-11.SPEC-004 (Leftover Lunch Withdrawal on Source Change), which calls this spec's eligibility and day rules but owns the re-link/withdraw decision itself
- Detailed leftover quantity or expiry tracking -- excluded per scope-boundaries.md SC-11: eligibility here is a simple yes/no classification, never a tracked quantity or expiry estimate
- Displaying the Confirm/Skip controls this spec authorizes -- owned by FEAT-11.SPEC-001 (Leftover Lunch Card), which enforces these authorization rules on screen but does not define them

## Governed Entity

**Entity:** Planned Meal (leftover-lunch sub-type)
**Source:** Feature Dependency Map (Planned Meal entity; leftover-lunch fields per the Brief's Shared Context)

| Field | Data Type | Description |
|-------|-----------|-------------|
| meal_kind | enum | Fixed to "leftover lunch" for every record this spec governs, distinguishing it from a dinner Planned Meal |
| linked source dinner | reference | The one dinner Planned Meal this leftover lunch rolls over from |
| night | date | The following day the leftover lunch is attached to, computed as no more than two days after the source dinner's night |
| status | enum | Suggested, Eaten, or Skipped |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-002 | Leftover Lunch Suggestion Generation | On creation, during each AI plan-generation cycle: calls the eligibility determination and following-day computation for every dinner |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | On re-evaluation, when the linked source dinner is swapped or changed: calls the eligibility determination and following-day computation again, and applies the re-link/withdraw authorization rules |
| FEAT-11.SPEC-001 | Leftover Lunch Card | On screen entry and on action attempt: enforces the Confirm/Skip authorization rules for the viewing role |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| meal_kind | Always set to "leftover lunch"; system-derived, never entered or edited by any role | Always | On create | N/A -- not a user-entered field; no manual-creation path exists for this record (Non-Goals) | Yes |
| linked source dinner | Must reference exactly one dinner-type Planned Meal in the same household's plan; must never be null while status is Suggested | Always | On create (FEAT-11.SPEC-002) and on re-link (FEAT-11.SPEC-004) | N/A -- system-computed, never entered manually; a dinner that cannot be resolved simply results in no leftover-lunch record being created or in the existing one being withdrawn (FEAT-11.SPEC-004) | Yes |
| night | Must be strictly after the linked source dinner's night, and no more than two days after it (XBR-10) | Always | On create and on re-link | N/A -- system-computed; when no day within the ceiling is free, no record is created (FEAT-11.SPEC-002) or the existing record is withdrawn (FEAT-11.SPEC-004) rather than placing it outside the ceiling | Yes |
| status | Must be one of Suggested, Eaten, Skipped; transitions only Suggested -> Eaten or Suggested -> Skipped, each a one-way, terminal transition | Always | On create (defaults to Suggested) and on update (FEAT-11.SPEC-001's Confirm/Skip) | N/A -- enforced structurally: Confirm and Skip are only ever offered while status is Suggested (FEAT-11.SPEC-001's Access and Visibility), so no invalid transition is ever presented as an option | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| One active link per source dinner | linked source dinner, status | At most one leftover-lunch record with status Suggested, Eaten, or Skipped may reference the same source dinner at a time; a source-dinner change re-links or withdraws the existing record (FEAT-11.SPEC-004) rather than ever creating a second link to the same dinner | N/A -- structurally enforced by FEAT-11.SPEC-002 and FEAT-11.SPEC-004, which always update or replace the existing link instead of creating a duplicate |
| Following-day ceiling | night, linked source dinner (its night) | night must fall strictly after the source dinner's night and no more than two calendar days after it (XBR-10) | N/A -- system-computed; a placement that would exceed the ceiling is never made (see Defaults and Derivations, Following-Day Computation) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a leftover-lunch suggestion | No household role -- system-automation only (FEAT-11.SPEC-002) | Always | Not offered as a manual action to any role; product-features.md's Validation & Limits and this feature's Non-Goals establish no manual-creation path |
| View a leftover-lunch card | Maya (Organiser) | Always | -- |
| View a leftover-lunch card | Sam (Other Adult Member) | Always | -- |
| View a leftover-lunch card | Jordan (older kid, limited login -- Later) | Always (his View access to the Weekly Plan includes it) | -- |
| View a leftover-lunch card | Riley (Operator, support) | Always, within an open support request's read-only view | -- |
| View a leftover-lunch card | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Confirm (mark Eaten) | Maya (Organiser) | Only while the record's status is Suggested | -- |
| Confirm (mark Eaten) | Sam (Other Adult Member) | Only while the record's status is Suggested; his Weekly Plan access is View, but a leftover-lunch status update is treated as a status update on the plan, not a change to it (Access Matrix notes) | -- |
| Confirm (mark Eaten) | Jordan (older kid, limited login -- Later) | Never | Confirm control is hidden; his Weekly Plan access is View-only |
| Confirm (mark Eaten) | Riley (Operator, support) | Never | Confirm control is hidden in the read-only support view (XBR-14) |
| Confirm (mark Eaten) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Skip | Maya (Organiser) | Only while the record's status is Suggested | -- |
| Skip | Sam (Other Adult Member) | Only while the record's status is Suggested; same reasoning as Confirm above | -- |
| Skip | Jordan (older kid, limited login -- Later) | Never | Skip control is hidden; his Weekly Plan access is View-only |
| Skip | Riley (Operator, support) | Never | Skip control is hidden in the read-only support view (XBR-14) |
| Skip | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Re-link to a new source dinner or day | No household role -- system-automation only (FEAT-11.SPEC-004) | Fires only when the linked source dinner's recipe changes or the slot is otherwise altered | Not offered as a manual action to any role; no reschedule path is modeled anywhere in the product (Non-Goals) |
| Withdraw (remove) a Suggested leftover-lunch record | No household role -- system-automation only (FEAT-11.SPEC-004) | Fires only when the source dinner is swapped or cleared and no eligible replacement takes the slot | Not offered as a manual action to any role |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| meal_kind | Always "leftover lunch" for a record this spec governs | On create | No |
| status | Defaults to Suggested | On create only | No -- it changes only through the Confirm/Skip actions in FEAT-11.SPEC-001, never through direct edit |
| linked source dinner | Set to the dinner Planned Meal FEAT-11.SPEC-002 identified as leftover-producing (on create), or the dinner's new recipe reference (on re-link, FEAT-11.SPEC-004) | On create and on re-link | No -- no manual re-link or reschedule path is modeled |
| night | Set by the Following-Day Computation below (on create and on re-link) | On create and on re-link | No |
| **Eligibility Determination (derivation owned by this spec, not a stored Planned Meal or Recipe field)** | A dinner is classified leftover-producing when the source Recipe's existing `name` field (feature-dependency-map.md, Recipe entity) matches, case-insensitively as a whole word, one of this spec's fixed set of batch-style dish-type terms: casserole, roast, stew, bake, chili, chilli, curry, lasagna, lasagne, soup, pot pie, batch. This spec owns and maintains that closed term list; no new field is added to Recipe or Planned Meal to store the classification -- the match runs fresh against the recipe's own `name` text every time this determination is evaluated, using only data the Recipe entity already carries. This is a fixed yes/no outcome per recipe and does not vary by which household cooks it or by household size -- household size affects how a dinner's portions are sized (Planned Meal's cook_time and rough_cost, "carried from the recipe, sized for the household," per the dependency map), but plays no part in this classification itself, which is a content-level read of the recipe's name alone. | Evaluated once per dinner, at each plan-generation cycle (FEAT-11.SPEC-002) and at each source-change re-evaluation (FEAT-11.SPEC-004) | No -- no household role can mark a dinner leftover-producing or not; the classification is read from the recipe's own `name` field each time |
| **Following-Day Computation (derivation, produces the night value above)** | Default: the day immediately following the source dinner's night. Fallback: if that day already holds another Suggested, Eaten, or Skipped leftover lunch from the same evaluation context (the same weekly plan on create, or the same slot's prior link on re-link), the day two days after the source dinner's night is used instead. If both the one-day and two-day days are already occupied, no placement is made -- the two-day ceiling (XBR-10) is never exceeded to find a free day. | Evaluated once per dinner alongside the Eligibility Determination | No |

## Business Rules

- XBR-10: a leftover lunch links to exactly one source dinner and a following day no more than two days later; swapping or removing the source dinner updates or withdraws the leftover suggestion (enforced by FEAT-11.SPEC-004, which calls this spec's rules).
- Eligibility is derived fresh each time from the recipe's existing `name` field against this spec's fixed dish-type term list (Defaults and Derivations, Eligibility Determination) -- it is never stored as a Recipe or Planned Meal field, and it is a fixed recipe-level outcome, not a per-week or per-household variation, keeping the "simple yes/no suggestion" model scope-boundaries.md SC-11 establishes.
- Confirm and Skip are each one-way, terminal transitions -- once Eaten or Skipped, a record's status never changes again through this feature (scope-boundaries.md SC-18 retains the outcome as permanent plan history).
- The following-day computation never exceeds the two-day ceiling to resolve a collision -- an eligible dinner that cannot be placed within the ceiling simply receives no suggestion that week (FEAT-11.SPEC-002) or has its existing suggestion withdrawn rather than moved beyond the ceiling (FEAT-11.SPEC-004).
- No household role has a create, reschedule, or manual-link action on this entity -- every write path is system-automation only (FEAT-11.SPEC-002, FEAT-11.SPEC-004), consistent with the feature's Non-Goals.

## Edge Cases

- **Following day computed at exactly two days after the source dinner** -- Passes the ceiling; a placement three days after the source dinner is never made under any fallback.
- **Both the one-day and two-day following days are already occupied by other leftover lunches from the same evaluation context** -- No placement is made; FEAT-11.SPEC-002 creates no record for that dinner, or FEAT-11.SPEC-004 withdraws the existing one rather than placing it further out.
- **A source dinner falls on the last night of the week** -- The following day may fall in the next calendar week; the ceiling is measured in elapsed days, not week boundaries, so the placement still proceeds normally.
- **A recipe's leftover-producing classification is revised in the Recipe Library after a leftover lunch has already been created from it** -- The already-created record keeps its existing link and night; classification is only re-evaluated when FEAT-11.SPEC-004 re-runs it because the source dinner itself changed, not because the recipe's own content was edited independently.
- **Sam attempts to Confirm a leftover lunch that has already been marked Skipped** -- Denied: the Confirm control is not shown once status is no longer Suggested, per the Field Validation Rules' terminal-transition rule.
- **Jordan (older kid, limited login) attempts to reach the Confirm action directly (e.g., a stale link)** -- Denied: the action is refused and the leftover-lunch card renders in its View-only presentation, consistent with his Authorization Rules row.
- **The linked source dinner is swapped to a recipe that is also leftover-producing** -- Handled by FEAT-11.SPEC-004 as a re-link (the existing record's linked source dinner reference updates; the night is only recomputed if the slot's own night changed, which a swap never does).

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given a dinner recipe whose `name` field contains a batch-style dish-type term from this spec's fixed list (e.g., "Tuesday's Chicken Casserole"), when FEAT-11.SPEC-002 evaluates it during plan generation, then it is classified leftover-producing and a Suggested leftover lunch is created for it.

**FEAT-11.SPEC-003-AC-02:** Given a dinner recipe whose `name` field matches none of this spec's batch-style dish-type terms, when FEAT-11.SPEC-002 evaluates it, then it is not classified leftover-producing and no leftover-lunch record is created for it.

**FEAT-11.SPEC-003-AC-03:** Given an eligible dinner on Tuesday, when the following-day computation runs and Wednesday is free, then the leftover lunch's night is set to Wednesday.

**FEAT-11.SPEC-003-AC-04:** Given an eligible dinner on Tuesday and Wednesday already holds another leftover lunch from the same week, when the following-day computation runs, then the leftover lunch's night falls back to Thursday.

**FEAT-11.SPEC-003-AC-05:** Given an eligible dinner whose Wednesday and Thursday following days are both already occupied, when the following-day computation runs, then no leftover lunch is created for that dinner and no day beyond the two-day ceiling is used.

**FEAT-11.SPEC-003-AC-06:** Given a leftover lunch already exists Suggested for a source dinner, when that same dinner is evaluated again in a later cycle, then no second leftover-lunch record is created for it (one active link per source dinner).

**FEAT-11.SPEC-003-AC-07:** Given Maya views a Suggested leftover lunch, when she taps Confirm, then the status transitions to Eaten, and this transition cannot later be reversed through this feature.

**FEAT-11.SPEC-003-AC-08:** Given Sam views a Suggested leftover lunch while his Weekly Plan access is View, when he taps Skip, then the status transitions to Skipped, since this is a status update rather than a plan change.

**FEAT-11.SPEC-003-AC-09:** Given Jordan (older kid, limited login) views a Suggested leftover lunch, when he looks for Confirm or Skip, then neither is shown, per his View-only authorization.

**FEAT-11.SPEC-003-AC-10:** Given Riley (Operator, support) views a household's plan during an open support request, when a leftover-lunch card is present, then Riley can view it but has no Confirm or Skip control, per XBR-14.

**FEAT-11.SPEC-003-AC-11:** Given Jordan (young kid profile, no login) has no path into the product, when this rule's Authorization Rules are evaluated for this role, then every action is Never, consistent with having no login.

**FEAT-11.SPEC-003-AC-12:** Given no household role has a create action on this entity, when any role looks for a way to manually add a leftover lunch, then no such control exists anywhere in the product.

**FEAT-11.SPEC-003-AC-13:** Given a source dinner is swapped to a still-eligible recipe, when FEAT-11.SPEC-004 calls this spec's rules, then the existing Suggested record's linked source dinner is updated (re-linked) rather than a second record being created.

**FEAT-11.SPEC-003-AC-14:** Given a source dinner is swapped to a no-longer-eligible recipe, when FEAT-11.SPEC-004 calls this spec's eligibility rule, then the eligibility determination returns not-eligible and FEAT-11.SPEC-004 withdraws the existing Suggested record.

**FEAT-11.SPEC-003-AC-15:** Given a leftover lunch is already marked Eaten, when its source dinner is later swapped, then this spec's rules are never invoked to re-link or withdraw it, since only Suggested records are re-evaluated.

**FEAT-11.SPEC-003-AC-16:** Given a source dinner falls on the last night of the week, when the following-day computation places its leftover lunch in the next calendar week, then the placement still succeeds, since the two-day ceiling is measured in elapsed days rather than week boundaries.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

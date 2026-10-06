# FEAT-16 — Units, Currency & Locale Configuration

This chapter covers FEAT-16, Units, Currency & Locale Configuration, a Important-tier feature. It contains 4 specifications carrying 50 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | screen | 11 |
| FEAT-16.SPEC-002 | Aisle Name Customization | screen | 11 |
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | logic-rule | 15 |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Units, Currency & Locale Configuration

## Summary

**Feature:** Units, Currency & Locale Configuration
**ID:** FEAT-16
**Description:** The household can set the measurement units, currency, and supermarket aisle names that fit where they live, so the plan and grocery list read naturally for US and UK households alike.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** The brief states directly that "units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded" for the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations). Included at MVP because both markets are targeted from launch, not added later.

**Key Capabilities:**
- Choose measurement units -- Household sets cups/oz or grams/ml as its default
- Choose currency -- Household sets its currency for budget and cost display
- Customize aisle names -- Household adjusts the aisle groupings on its grocery list to match how its local store is laid out

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | Screen | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Maya sets or changes the household's default measurement unit system and currency; Sam and Riley see the current settings without changing them |
| FEAT-16.SPEC-002 | Aisle Name Customization | Screen | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Maya renames or reorders the household's supermarket aisle groupings to match their local store; Sam and Riley see the current groupings without changing them |
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | Logic/Rule | Maya (Organiser) | Governs the supported unit and currency sets, the aisle-name length limit, and the default unit system, currency, and aisle groupings a new household starts with |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | Logic/Rule | All | Governs how stored quantities and costs are converted into the household's configured unit system and currency wherever they are displayed, and ensures a locale change updates existing figures for display rather than leaving them inconsistent |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Choose measurement units | FEAT-16.SPEC-001 | Primary purpose of the Units & Currency Settings screen | Phase 2 (Explicit) |
| Choose currency | FEAT-16.SPEC-001 | Primary purpose of the Units & Currency Settings screen | Phase 2 (Explicit) |
| Customize aisle names | FEAT-16.SPEC-002 | Primary purpose of the Aisle Name Customization screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field names a supported unit set, a supported currency set (covering at least USD and GBP), and an aisle-name length limit; the States field additionally implies every household has a default configuration from creation, based on setup-time input -- a derivation rule with no home in a single screen, since both settings screens read and enforce it |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | Phase 4 (Trigger-Response Analysis, cross-feature effects) + Phase 5 (Derivation rules) | XBR-11 requires that a locale change convert existing budget figures, recipe quantities, grocery-list quantities, and check-in spend for display rather than leaving them inconsistent, across five other features (FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25); this cross-feature derivation formula needed its own Logic/Rule spec rather than living inside either settings screen |

## Entity-Lifecycle Coverage Matrix

**Entity: Household (locale fields: unit_system, currency, aisle_names)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | The Household record itself is created by FEAT-01 (Household Setup & Member Profiles); this feature does not create the entity, only its locale fields' initial values, which are populated by FEAT-16.SPEC-003's default-derivation rule at the moment FEAT-01 creates the household | Ownership boundary with FEAT-01 |
| Read (single) | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | Both settings screens load and display the household's current unit_system/currency and aisle_names on entry | -- |
| Read (list) | N/A | Each household has exactly one locale configuration; there is no list of locale configurations to browse | One record per household (ASMP -- one household per account, SC-03) |
| Update | FEAT-16.SPEC-001 (unit_system, currency), FEAT-16.SPEC-002 (aisle_names) | Maya edits and saves; FEAT-16.SPEC-004 governs how the updated values are reflected across consuming features' displays | Only Maya can write; Sam and Riley are view-only per the Access Matrix |
| Delete/Archive | N/A -- owned by FEAT-18 | Locale fields have no independent delete/archive path; they are deleted only as part of the whole Household record's deletion, which FEAT-18 (Account & Data Management) owns end to end, including its retention/purge window | Recorded as a non-goal below to avoid a silent omission |
| State Transition | N/A | The product definition gives locale settings no lifecycle states beyond their current saved value -- there is no draft, pending, or archived state for a unit system, currency, or aisle-name set (product-features.md, States: "Empty: N/A ... Loading: N/A ... changes save instantly") | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household (weekly_budget, other fields) | FEAT-16.SPEC-001 | Currency setting determines how the household's existing budget figure is labeled and displayed, without SPEC-001 itself managing the budget field |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya changes unit system or currency and saves | Validate the new value against the supported set; on success, apply the change and make future displays reflect it via the conversion rule | Standalone Logic/Rule (validation) feeding a Standalone Logic/Rule (conversion) | FEAT-16.SPEC-003, FEAT-16.SPEC-004 |
| Maya renames or reorders an aisle grouping and saves | Validate the new name against the length limit; apply the change to future grocery lists only | Standalone Logic/Rule (validation); inline confirmation in triggering screen | FEAT-16.SPEC-003; FEAT-16.SPEC-002 |
| Household is created (FEAT-01) | Populate unit_system, currency, and aisle_names with locale-appropriate defaults based on setup-time input | Standalone Logic/Rule | FEAT-16.SPEC-003 |
| A locale setting (unit system or currency) changes after setup | Existing budget figures, recipe quantities, grocery-list quantities, and check-in spend are converted for display in the newly configured unit/currency, rather than left showing the prior locale's values | Standalone Logic/Rule, cross-feature | FEAT-16.SPEC-004 (consumed by FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25) |
| Maya's save to either settings screen fails (e.g., lost connectivity mid-save) | The prior setting remains active; the user sees the save did not take effect rather than a mixed or inconsistent state | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| Sam or Riley opens either settings screen | Display current values in a view-only mode; no edit controls rendered | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| An unauthorized visitor attempts to reach either settings screen | Access denied -- no locale data is shown | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| Either settings screen is opened while offline | Current values are viewable; edit controls are present but saving is disabled until connectivity returns | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |

**On zero Automation specs:** This feature manages a single, low-contention configuration record with no cascading creates, no scheduled processing, and no asynchronous background work of its own -- every discovered side-effect is either a synchronous validation/derivation rule (SPEC-003, SPEC-004) or a simple inline consequence of a screen save. No trigger-response pair in this feature crosses the "processing logic with its own lifecycle and failure modes" threshold that would justify a standalone Automation spec.

## Shared Context

**Shared Entities:**
- Household locale fields (unit_system, currency, aisle_names) -- read and updated by FEAT-16.SPEC-001 (unit_system, currency) and FEAT-16.SPEC-002 (aisle_names); validated and defaulted by FEAT-16.SPEC-003; consumed for display conversion by FEAT-16.SPEC-004 and, through it, by FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, and FEAT-25.

**Shared UI Patterns:**
- Role-gated settings display -- both FEAT-16.SPEC-001 and FEAT-16.SPEC-002 render the same record in two modes: an editable form for Maya, and a read-only display of the same fields for Sam and Riley. Spec Writers for both screens should describe the view-only mode consistently (same layout, controls simply removed/disabled, not hidden behind a different screen).
- Save-failure recovery -- both screens keep the prior value visible and active on a failed save, per the feature's States field; Spec Writers should describe this identically rather than inventing screen-specific error copy.

**Shared Validation:**
- FEAT-16.SPEC-003 defines the supported unit set, the supported currency set (at least USD and GBP), the aisle-name length limit, and the default-value derivation. FEAT-16.SPEC-001 and FEAT-16.SPEC-002 both reference SPEC-003 for their field-level validation and initial values rather than duplicating the rules.

## Internal Dependency Map

```
SPEC-001 (Units & Currency Settings) -> [Maya saves a change] -> SPEC-003 (Locale Configuration Validation & Defaults) -> [valid] -> SPEC-004 (Cross-Feature Value Conversion Rule)
SPEC-002 (Aisle Name Customization) -> [Maya saves a change] -> SPEC-003 (Locale Configuration Validation & Defaults) -> [valid] -> applies to future grocery lists (FEAT-06)
SPEC-003 (Locale Configuration Validation & Defaults) -> [household created by FEAT-01] -> populates default unit_system, currency, aisle_names on SPEC-001 / SPEC-002
SPEC-004 (Cross-Feature Value Conversion Rule) -> [locale setting changes] -> existing displayed figures reflow in FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25
```

**Default Entry:** SPEC-001 (Units & Currency Settings) -- the screen shown when the user navigates to this feature's settings area from Household Setup; SPEC-002 (Aisle Name Customization) is reached from within the same settings area as a distinct sub-section.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-16.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Maya reaches the units and currency step from the guided setup flow | Setup step "Set units, currency and aisle layout" (First Household Setup, step 4) |
| FEAT-16.SPEC-002 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Maya reaches the default aisle-groupings review from the guided setup flow | Setup step "Set units, currency and aisle layout" (First Household Setup, step 4) |
| FEAT-16.SPEC-003 | Outbound | FEAT-01 (Household Setup & Member Profiles) | Default unit system, currency, and aisle groupings are populated onto the new Household record at creation | Household creation |
| FEAT-16.SPEC-004 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Weekly plan cost and total display in the household's configured currency | Plan displayed or locale setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-06 (Shared Grocery List) | Grocery-list quantities display in the household's configured unit system; aisle grouping reflects FEAT-16.SPEC-002's current names/order | List generated, item displayed, or locale/aisle setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-08 (Recipe Library & Starter Recipes) | Recipe ingredient quantities display in the household's configured unit system | Recipe viewed or unit setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-23 (Manual Weekly Planning) | Manual plan's budget figures display in the household's configured currency | Manual plan viewed or currency setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-25 (Weekly Waste & Spend Check-In) | Check-in spend figures display in the household's configured currency | Check-in viewed or currency setting changed |

## Non-Functional Notes

**Data volumes / growth:** Negligible -- a single locale configuration record per household (unit_system, currency, aisle_names), unchanged in shape as the household base grows to several thousand households (BRIEF.md, Scale & Non-Functional Expectations, Order of magnitude). Aisle-name customization holds a small, bounded list of aisle groupings per household, not an unbounded collection.

**Responsiveness:** Settings changes save instantly with no perceptible wait, consistent with the feature's own States field ("Loading: N/A -- changes save instantly"); the resulting display conversion must appear immediately wherever the plan, recipes, or grocery list are next shown, not as a delayed batch update.

**Data sensitivity / privacy:** Household locale settings (unit system, currency, aisle names) are personal household configuration data -- private to household members, never sold or used for advertising (ASMP-26); they carry no children's-data sensitivity, since no kid-profile field is stored here. They are exportable and deletable under general personal-data rights as part of the whole Household record (ASMP-27, FEAT-18).

**Compliance flags:** ASMP-28 (Localization) governs this feature directly: measurement units, currency, and aisle names must be configurable to support both US and UK households from launch, and the product remains English-only (SC-13) -- this feature delivers locale-format configurability, not translation. No health, financial, or children's-privacy regime applies to locale settings themselves.

## Non-Goals

- **Languages other than English** -- Excluded per scope-boundaries.md (SC-13): the brief targets the US and UK first and requires configurable units, currency, and aisle names (FEAT-16), but explicitly does not require translated content for the two launch markets.
- **Automatic purge of locale settings** -- Intentional lifecycle decision surfaced by the CRUD matrix: locale fields have no independent delete/archive or retention policy because they are not an independent entity -- they are deleted only as part of the whole Household record's deletion, which FEAT-18 (Account & Data Management) owns end to end, including that feature's 30-day retention window (ASMP-27).
- **Support for currencies or unit systems beyond the supported set** -- Excluded per the feature's own Validation & Limits field, which scopes units to a supported set (e.g., cups/oz, grams/ml) and currency to a supported set covering at least USD and GBP at launch; broader locale support is not a stated MVP need since the brief targets only the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations, Geography).
- **Automatic locale detection from device or browser settings** -- Adjacency exclusion: the feature's Data Notes name the Source as "organiser input," and the journey describes Maya confirming, not merely accepting, locale defaults during guided setup (First Household Setup, step 4); this feature derives sensible starting defaults (FEAT-16.SPEC-003) but does not silently auto-apply device-detected locale without the organiser's confirmation.



# Screen Spec: Units & Currency Settings

## Overview

**Name:** Units & Currency Settings
**ID:** FEAT-16.SPEC-001
**Type:** Screen
**Purpose:** Maya sets or changes the household's default measurement unit system and currency; Sam and Riley view the current settings without changing them.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration

## Scope and Non-Goals

**In Scope:**
- Displaying and editing the household's unit_system (cups/oz or grams/ml) and currency (from the supported set)
- Read-only display of these same fields for roles without edit access
- Saving a change, including the failed-save recovery experience
- Linking to the Aisle Name Customization screen (FEAT-16.SPEC-002) as the settings area's other sub-section

**Non-Goals:**
- Editing aisle names or their order -- handled by FEAT-16.SPEC-002 (Aisle Name Customization)
- Defining the supported unit set, the supported currency set, or the default values a new household starts with -- governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults); this screen only reads and enforces those rules
- Converting existing budget, recipe, plan, or check-in figures once a setting changes -- governed by FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule); this screen only triggers the save that FEAT-16.SPEC-004 reacts to
- Automatic locale detection from the device or browser -- excluded per feature-overview.md's Non-Goals: the organiser confirms locale defaults during guided setup rather than having them silently auto-applied
- Editing the household's weekly_budget amount -- owned by Household Setup & Member Profiles (FEAT-01.SPEC-008); this screen only reads weekly_budget to show it labeled in the currently configured currency

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Guided setup reaches its units-and-currency step (wizard step 6 of 8; First Household Setup journey step 4, "Set units, currency and aisle layout"), immediately after the organiser confirms the weekly budget and schedule | Household id, with unit_system/currency pre-filled by FEAT-16.SPEC-003's default-derivation rule for the organiser to confirm or change |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps the "Units, currency & aisles" row | Household id; screen loads the household's current saved unit_system and currency |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Change unit_system and currency, save | -- |
| Sam (Other Adult Member) | Full screen, view-only mode | None -- no edit controls rendered | Attempting to reach the screen with edit intent (e.g., a direct link) still opens the screen but shows every field disabled with no Save control; no separate denial message is needed since nothing is hidden, only non-interactive |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role, so the screen is never reachable; there is no sign-in path that leads here |
| Jordan (older kid, limited login -- Later) | No | No | This screen is not part of the older-kid login's Household Setup access; no navigation entry point is offered, and a direct link redirects to the older-kid login's home screen (the week's plan) |
| Riley (Operator, support) | Full screen, view-only mode | None -- no edit controls rendered | Reachable only through Operator Read-Only Support Access (FEAT-22) against a household with an open Support Request; outside that context the screen is not reachable at all |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Household Settings Hub (FEAT-01.SPEC-010), not directly on this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress unsaved change on this screen is discarded, since changes here save instantly and there is no draft state to preserve |

## Layout and Content

**Header:** Screen title "Units & Currency" with a back arrow (returns to the Household Settings Hub, FEAT-01.SPEC-010) and, in guided setup, a "Continue" action instead of a back arrow, since this is a linear setup step.

**Body:** A single-column settings form with two grouped sections in order:
- **Measurement units** -- a two-option selector: "Cups & ounces (US)" and "Grams & millilitres (metric)". The currently active option is visually indicated. Below the selector, one line of helper text: "This sets how ingredient quantities and grocery list amounts are shown."
- **Currency** -- a selection input listing the supported currency set defined by FEAT-16.SPEC-003 (at minimum "US Dollar ($)" and "British Pound (£)"), showing the currency's name and symbol together. Below the selector, one line of helper text: "This sets how your weekly budget and meal costs are shown." Directly under the helper text, a read-only preview line: "Your weekly budget: {weekly_budget value} {currency symbol}" reflecting the household's existing weekly_budget (Household Setup & Member Profiles, FEAT-01) under whichever currency is currently selected in this form (not yet saved).

**Footer:** A single "Save" action button (Maya only; not rendered for view-only roles). Below the footer, a text link "Customize aisle names" that navigates to FEAT-16.SPEC-002.

For Sam and Riley (view-only mode), the same layout renders with both the unit selector and the currency selector shown as static, non-interactive display rows carrying the current values, no Save button, and the same "Customize aisle names" link (which itself opens FEAT-16.SPEC-002 in that screen's own view-only mode).

### Responsive Behavior

- **Compact size class:** The two settings groups stack vertically as described above, full width; Save remains in the footer, always visible without scrolling past it.
- **Medium size class and above:** The form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Weekly budget preview line:** Wraps to a second line at compact widths if the formatted amount and currency symbol do not fit on one line; never truncates the amount itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (header, outside guided setup) | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard transition |
| Continue (header, guided setup) | Tap | 1. Validate the pending unit_system and currency values against the supported sets defined by FEAT-16.SPEC-003 -- the same validate step Save performs. 2. If valid, save both fields to the Household record -- the same save step Save performs. 3. Trigger FEAT-16.SPEC-004 so that budget, plan, recipe, and grocery-list displays reflect the new setting the next time they are shown. 4. Advance to FEAT-16.SPEC-002 (Aisle Name Customization). Continue behaves as Save followed by navigation, not as navigation alone. | Screen closes on success | Success: standard transition to FEAT-16.SPEC-002. Failure: the same inline error banner Save produces (see States/Error below); guided setup remains on this screen, pending values retained, until the save succeeds |
| Measurement units selector (Maya only) | Tap an option | Selects that option as the pending unit_system value; does not save yet | Selected option shows the active indicator; the other option shows inactive | Immediate visual selection change, no save confirmation yet |
| Currency selector (Maya only) | Select an option | Selects that option as the pending currency value; updates the weekly budget preview line immediately to the newly selected currency's symbol | Selector shows chosen currency; preview line re-renders | Preview line updates instantly with no separate loading state |
| Save button (Maya only) | Tap | 1. Validate the pending unit_system and currency values against the supported sets defined by FEAT-16.SPEC-003. 2. If valid, save both fields to the Household record. 3. Trigger FEAT-16.SPEC-004 so that budget, plan, recipe, and grocery-list displays reflect the new setting the next time they are shown. | Save button shows an inline saving confirmation, not a full-page spinner | Success: inline confirmation "Saved" appears next to the button and the screen's read state updates to the new values. Failure: inline error banner, described in States/Error below; the prior saved values remain shown and active. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in its inline saving state |
| "Customize aisle names" link | Tap | Navigate to FEAT-16.SPEC-002 | Screen changes | Standard transition |
| Unit selector, currency selector (Sam, Riley) | Tap | No action -- controls are disabled, not interactive | None | No feedback; controls render in a visibly non-interactive style |

### Accessibility Notes

- **Focus order (Maya, edit mode):** Back arrow/Continue -> Measurement units selector -> Currency selector -> Save button -> "Customize aisle names" link.
- **Focus order (Sam, Riley, view-only mode):** Back arrow -> Measurement units display row -> Currency display row -> "Customize aisle names" link (Save is absent from the order since it is not rendered).
- **Selection announcements:** When a unit or currency option is selected, the change is announced to assistive technology, including the updated weekly budget preview text.
- **Save feedback:** The "Saved" confirmation is announced on success; on validation or save failure, focus moves to the inline error banner and its message is announced.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures. Disabled controls in view-only mode are marked non-interactive to assistive technology rather than silently unresponsive.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Current unit_system and currency shown, selected/pre-filled per the household's saved values (or FEAT-16.SPEC-003's derived defaults on first entry from guided setup) | Screen opens | User changes a selection or navigates away |
| Editing (Maya) | Pending selections differ from the last saved values; Save button enabled | Maya taps a different unit or currency option | Maya taps Save, or navigates away (see Edge Cases) |
| Saving | Save button shows an inline saving confirmation; selectors remain interactive but a second Save tap is ignored | Maya taps Save | Save completes (success or failure) |
| Error | Inline error banner above the Save button: "Couldn't save your changes. Check your connection and try again." with a Retry action; the prior saved values remain the active, displayed values | Save operation fails, whether triggered by the Save button or by Continue during guided setup | Maya taps Retry and the save succeeds, or Maya navigates away (pending change is discarded) |
| Offline/Degraded | Banner "You're offline -- changes here will save once you're back online." at the top of the form; both selectors remain visible and interactive for Maya, but the Save button is disabled with the same banner explaining why | Connectivity lost while the screen is open | Connectivity restored -- Save button re-enables; nothing was queued for automatic submission since the settings save instantly rather than in the background |

## Validation Rules

Validation governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults). See that spec for the supported unit set and supported currency set. This screen applies validation on selection (the user can only choose from the supported set presented in each selector, so an invalid value cannot be entered) and re-confirms it on Save before writing to the Household record.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 (Household Setup & Member Profiles) |
| Continue tap (guided setup), after validate/save/trigger FEAT-16.SPEC-004 succeeds | FEAT-16.SPEC-002 (Aisle Name Customization) | -- |
| "Customize aisle names" link tap | FEAT-16.SPEC-002 (Aisle Name Customization) | -- |
| Successful save (outside guided setup) | Screen remains on FEAT-16.SPEC-001 with updated values shown | -- |

## Data Model

**Creates:** None.
**Reads:** Household -- unit_system, currency (current saved values, or FEAT-16.SPEC-003's derived defaults on first entry from guided setup); Household -- weekly_budget (read-only, for the budget preview line; owned and updated by FEAT-01, not this screen).
**Updates:** Household -- unit_system, currency (Maya only).
**Deletes:** None -- locale fields have no independent delete path; they are removed only as part of the whole Household record's deletion, owned end to end by FEAT-18 (Account & Data Management).

## Business Rules

- Field-level validation for unit_system and currency is governed by FEAT-16.SPEC-003 -- this screen enforces those rules but does not define them.
- A successful save triggers FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) so that the household's budget, plan, recipe, and grocery-list displays reflect the new setting the next time each is shown.
- XBR-11: the household's units and currency apply consistently everywhere they appear (plan cost and weekly total, recipe quantities, grocery-list quantities, budget, check-in spend); this screen is one of the two places (with FEAT-16.SPEC-002) where those settings are changed.
- Only Maya may change unit_system or currency; Sam and Riley may only view them, per FEAT-16.SPEC-003's Authorization Rules.

## Edge Cases

- **Maya navigates away with an unsaved pending selection** -- No confirmation dialog is shown; the pending selection is simply discarded and the screen reverts to the last saved values on next entry, consistent with the feature's own States field ("changes save instantly," meaning there is no draft state to protect).
- **Maya taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in its inline saving state); no duplicate save is submitted.
- **Household's unit_system or currency changed by Maya on a second device between this screen's load and save** -- Save is accepted and applied last-write-wins per the dependency map's Contention note for the Household entity: whichever save reaches the server last becomes the active setting, and a failed save keeps the prior value active rather than leaving a mixed state. No blocking conflict dialog is shown, since the resolution is last-write-wins rather than reject-with-refresh.
- **Save fails partway (unit_system written, currency write fails, or vice versa)** -- The save is atomic: either both fields are updated together or neither is, so the household never shows a unit_system from one save and a currency from another.
- **Sam or Riley opens this screen while Maya is mid-edit on another device** -- Sam and Riley always see the last successfully saved values; they never see Maya's unsaved pending selection, since pending selections are local to Maya's own screen session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Navigation (inbound); References (inbound) | Guided setup hands off to this screen as wizard step 6 of 8 (journey step 4), immediately after budget and schedule are confirmed; this screen also reads weekly_budget (owned by FEAT-01.SPEC-008) to show it labeled in the current currency |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point outside guided setup; back arrow returns here |
| FEAT-16.SPEC-002 (Aisle Name Customization) | Navigation (outbound) | "Customize aisle names" link and guided-setup Continue |
| FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults) | References (outbound) | Supported unit set, supported currency set, and default-value derivation |
| FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) | Triggers (outbound) | A successful save triggers the conversion rule that reflows existing displayed figures |

## Analytics and Success Signals

{No success-metrics.md metric names Units, Currency & Locale Configuration as its Connected Feature; the events below are recorded per product-features.md's Signals field for this feature, but each cites N/A since this configuration screen's downstream effect on the household's experience is measured through the metrics of the features that consume the setting (e.g., Weekly Planning Time for AI Weekly Dinner Plan Generation, Reported Food Waste and Spend Reduction for the check-in), not through a metric of its own.}

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| locale_units_set | selected unit system, entry context (guided setup / settings hub) | Save succeeds with a changed unit_system value | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |
| locale_currency_set | selected currency, entry context (guided setup / settings hub) | Save succeeds with a changed currency value | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |
| locale_settings_save_failed | which field(s) were pending | Save fails and the error state is shown | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |

## Acceptance Criteria

**FEAT-16.SPEC-001-AC-01:** Given Maya is on the Units & Currency Settings screen, when she selects "Grams & millilitres (metric)" and taps Save, then the household's unit_system is saved as metric, an inline "Saved" confirmation appears, and FEAT-16.SPEC-004 is triggered so future recipe and grocery-list displays show metric quantities.

**FEAT-16.SPEC-001-AC-02:** Given Maya is on the Units & Currency Settings screen with the household's currency currently set to US Dollar, when she selects "British Pound (£)", then the weekly budget preview line updates immediately to show the same budget figure with the £ symbol, before Save is tapped.

**FEAT-16.SPEC-001-AC-03:** Given Maya taps Save and the save fails due to a lost connection, then the inline error banner "Couldn't save your changes. Check your connection and try again." appears with a Retry action, and the household's previously saved unit_system and currency remain the active, displayed values.

**FEAT-16.SPEC-001-AC-04:** Given Maya loses connectivity while this screen is open, when she attempts to change the currency selection, then the selection still updates on screen, but the Save button is disabled and the banner "You're offline -- changes here will save once you're back online." is shown.

**FEAT-16.SPEC-001-AC-05:** Given Sam (Other Adult Member) opens this screen, when it loads, then he sees the household's current unit_system and currency in a non-interactive display, no Save button is rendered, and the "Customize aisle names" link is present.

**FEAT-16.SPEC-001-AC-06:** Given Riley (Operator, support) is viewing this screen through an open Support Request, when the screen loads, then Riley sees the same non-interactive display Sam sees, with no edit controls.

**FEAT-16.SPEC-001-AC-07:** Given Jordan as a young kid profile has no login, when any attempt is made to reach this screen, then no such path exists, since young kid profiles have no sign-in at all.

**FEAT-16.SPEC-001-AC-08:** Given an unauthenticated visitor requests this screen's URL directly, then they are redirected to the sign-in screen, and after signing in they land on the Household Settings Hub (FEAT-01.SPEC-010) rather than directly on this screen.

**FEAT-16.SPEC-001-AC-09:** Given Maya's session expires while she has a pending, unsaved unit selection on this screen, when the session-expired dialog appears and she signs in again, then the pending selection is not restored, since locale settings save instantly and carry no draft state.

**FEAT-16.SPEC-001-AC-10:** Given Maya is signed in on two devices and saves a currency change on Device A while an older currency selection is still pending on Device B, when she then saves from Device B, then Device B's save becomes the active currency (last-write-wins), consistent with the Household entity's Contention resolution in the Feature Dependency Map.

**FEAT-16.SPEC-001-AC-11:** Given Maya reaches this screen from guided setup (FEAT-01.SPEC-008) immediately after confirming the weekly budget and schedule, when the screen first loads, then the unit and currency selectors are pre-filled with FEAT-16.SPEC-003's derived defaults rather than appearing blank, and Maya must tap Continue (which behaves as Save) to confirm them before advancing to FEAT-16.SPEC-002.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (loaded, editing, saving, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Aisle Name Customization

## Overview

**Name:** Aisle Name Customization
**ID:** FEAT-16.SPEC-002
**Type:** Screen
**Purpose:** Maya renames or reorders the household's supermarket aisle groupings to match their local store; Sam and Riley see the current groupings without changing them.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration

## Scope and Non-Goals

**In Scope:**
- Displaying the household's current aisle_names list in its saved order
- Renaming an existing aisle grouping
- Reordering aisle groupings (drag-to-reorder or equivalent up/down controls)
- Adding a new aisle grouping and removing one, within the aisle-name length and list rules FEAT-16.SPEC-003 defines
- Read-only display of the same list for roles without edit access

**Non-Goals:**
- Editing unit_system or currency -- handled by FEAT-16.SPEC-001 (Units & Currency Settings)
- Defining the aisle-name length limit or the default aisle groupings a new household starts with -- governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults); this screen only reads and enforces those rules
- Assigning individual grocery-list items to an aisle -- that mapping is Shared Grocery List's (FEAT-06) own ingredient-to-aisle logic; this screen only manages the names and order of the aisle groupings themselves
- Retroactively re-grouping an already-generated grocery list's items when aisle names change -- per the feature's Side-Effect Inventory, a saved aisle-name change applies to future grocery lists only, not to a list already generated before the change

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-16.SPEC-001 (Units & Currency Settings) | Organiser taps "Customize aisle names," or guided setup advances via Continue after confirming units and currency (wizard step 7 of 8; journey step 4 continuation) | Household id; on first entry from guided setup, the aisle_names list is pre-filled with FEAT-16.SPEC-003's default aisle groupings for the organiser to review |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens the hub's "Units, currency & aisles" row, then taps "Customize aisle names" on FEAT-16.SPEC-001 | Household id; screen loads the household's current saved aisle_names list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Rename, reorder, add, and remove aisle groupings; save | -- |
| Sam (Other Adult Member) | Full screen, view-only mode | None -- no edit, reorder, add, or remove controls rendered | Attempting to reach the screen with edit intent still opens it, but every row shows no rename/remove affordance and no drag handle; nothing is hidden, only non-interactive |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role, so the screen is never reachable |
| Jordan (older kid, limited login -- Later) | No | No | Not part of the older-kid login's access; no navigation entry point is offered, and a direct link redirects to the older-kid login's home screen (the week's plan) |
| Riley (Operator, support) | Full screen, view-only mode | None -- no edit, reorder, add, or remove controls rendered | Reachable only through Operator Read-Only Support Access (FEAT-22) against a household with an open Support Request; outside that context the screen is not reachable at all |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Household Settings Hub (FEAT-01.SPEC-010), not directly on this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress unsaved rename, reorder, add, or remove is discarded |

## Layout and Content

**Header:** Screen title "Aisle Names" with a back arrow (returns to FEAT-16.SPEC-001) and, in guided setup, a "Finish" action instead of a back arrow, since this is the last configuration step of guided setup (step 7 of 8); Finish hands off to FEAT-01.SPEC-009, the Step 8 of 8 completion screen.

**Body:** A vertically ordered list of the household's aisle groupings, one row per aisle, in the household's currently saved order:
- Each row shows a drag handle (Maya only), the aisle name as an editable text field (Maya only; static text for view-only roles), and a remove control (Maya only).
- Below the list, an "Add aisle" action (Maya only) that appends a new, empty-named row at the end of the list.
- One line of helper text above the list: "These names group items on your shared grocery list. Drag to reorder, or tap a name to rename it."

For Sam and Riley (view-only mode), the same list renders with static aisle names in their saved order, no drag handles, no remove controls, and no "Add aisle" action.

**Footer:** A single "Save" action button (Maya only; not rendered for view-only roles).

### Responsive Behavior

- **Compact size class:** The aisle list stacks as full-width rows as described above; drag handles remain touch-sized; Save stays in the footer, always visible without scrolling past it.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Long aisle lists:** The list scrolls within the body region rather than growing the page indefinitely; the header and footer remain fixed.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (header, outside guided setup) | Tap | Navigate to FEAT-16.SPEC-001 | Screen closes | Standard transition |
| Finish (header, guided setup) | Tap | 1. Validate every pending aisle name and the pending order against FEAT-16.SPEC-003's rules -- the same validate step Save performs. 2. If valid, save the full aisle_names list (names and order) to the Household record -- the same save step Save performs. 3. Trigger FEAT-16.SPEC-004 so future grocery lists group items under the new names. 4. Complete setup and navigate to FEAT-01.SPEC-009 (Setup Complete & Next Steps). Finish behaves as Save followed by navigation, not as navigation alone. | Screen closes on success | Success: standard transition to FEAT-01.SPEC-009. Failure: the same inline error banner Save produces (see States/Error below), including the "Add at least one aisle before saving" error if the pending list is empty; guided setup remains on this screen, pending changes retained, until the save succeeds |
| Aisle name field (Maya only) | Type | Captures the new name as a pending edit for that row; does not save yet | Field shows entered text | Standard input focus state |
| Aisle name field (Maya only) | Blur | Validates the entered name against FEAT-16.SPEC-003's aisle-name length rule | Error state on the row if invalid | Inline error message below the row if the name is empty or exceeds the length limit |
| Drag handle (Maya only) | Drag and drop | Reorders the pending aisle list to the dropped position | List re-renders in the new order | Row visually lifts during drag and settles into its new position on drop |
| Remove control (Maya only) | Tap | Removes that row from the pending aisle list | Row disappears from the list | Brief inline confirmation "Aisle removed" with an "Undo" action available until Save is tapped |
| "Add aisle" action (Maya only) | Tap | Appends a new, empty-named row at the end of the pending list, focused for immediate typing | New row appears | Focus moves to the new row's name field |
| Save button (Maya only) | Tap | 1. Validate every pending aisle name and the pending order against FEAT-16.SPEC-003's rules. 2. If valid, save the full aisle_names list (names and order) to the Household record. 3. Trigger FEAT-16.SPEC-004 so future grocery lists group items under the new names. | Save button shows an inline saving confirmation | Success: inline confirmation "Saved" appears. Failure: inline error banner, described in States/Error below; the prior saved list remains active. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in its inline saving state |
| Aisle name field, drag handle, remove control (Sam, Riley) | Tap / drag attempt | No action -- controls are not rendered for these roles | None | No feedback; the row shows static text only |

### Accessibility Notes

- **Focus order (Maya, edit mode):** Back arrow/Finish -> each aisle row's name field, in list order, followed by that row's remove control -> "Add aisle" action -> Save button.
- **Focus order (Sam, Riley, view-only mode):** Back arrow -> each aisle row's static name, in list order (Save is absent from the order since it is not rendered).
- **Reorder announcements:** A keyboard-based reorder alternative (e.g., "Move up" / "Move down" actions per row) is available alongside drag-and-drop, and each move is announced to assistive technology with the row's new position ("Produce moved to position 1 of 6").
- **Validation announcements:** When a name field enters an error state, its error message is announced and programmatically associated with the field.
- **Save feedback:** The "Saved" confirmation is announced on success; on failure, focus moves to the inline error banner.
- **Keyboard alternatives:** Every action on this screen, including reordering, is reachable by keyboard; drag-and-drop is never the only way to reorder.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Current aisle_names shown in saved order (or FEAT-16.SPEC-003's derived default groupings on first entry from guided setup) | Screen opens | Maya edits, reorders, adds, or removes a row |
| Editing (Maya) | Pending list differs from the last saved list; Save button enabled | Maya changes any name, order, addition, or removal | Maya taps Save, or navigates away (see Edge Cases) |
| Saving | Save button shows an inline saving confirmation; list remains interactive but a second Save tap is ignored | Maya taps Save | Save completes (success or failure) |
| Error | Inline error banner above the Save button: "Couldn't save your changes. Check your connection and try again." with a Retry action; the prior saved aisle_names list remains the active, displayed list | Save operation fails, whether triggered by the Save button or by Finish during guided setup | Maya taps Retry and the save succeeds, or Maya navigates away (pending changes are discarded) |
| Offline/Degraded | Banner "You're offline -- changes here will save once you're back online." at the top of the list; renaming, reordering, adding, and removing remain available to Maya, but the Save button is disabled with the same banner explaining why | Connectivity lost while the screen is open | Connectivity restored -- Save button re-enables; nothing was queued for automatic submission since aisle-name changes save instantly rather than in the background |

## Validation Rules

Validation governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults). See that spec for the aisle-name length limit and any list-level rules (minimum aisle count, duplicate-name handling). This screen applies the length rule on field blur and re-confirms all pending names and the pending order on Save before writing to the Household record.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-16.SPEC-001 (Units & Currency Settings) | -- |
| Finish tap (guided setup), after validate/save/trigger FEAT-16.SPEC-004 succeeds | FEAT-01.SPEC-009 (Setup Complete & Next Steps) | FEAT-01 (Household Setup & Member Profiles) |
| Successful save (outside guided setup) | Screen remains on FEAT-16.SPEC-002 with the updated list shown | -- |

## Data Model

**Creates:** None.
**Reads:** Household -- aisle_names (current saved list and order, or FEAT-16.SPEC-003's derived defaults on first entry from guided setup).
**Updates:** Household -- aisle_names (Maya only; the full list of names and their order is replaced on each save).
**Deletes:** None at the Household level -- an individual aisle row can be removed from the pending list before Save, but the aisle_names field itself has no independent delete path; it is removed only as part of the whole Household record's deletion, owned end to end by FEAT-18 (Account & Data Management).

## Business Rules

- Field-level validation for aisle names is governed by FEAT-16.SPEC-003 -- this screen enforces those rules but does not define them.
- A successful save triggers FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) so that future grocery lists group items under the new aisle names; an already-generated grocery list keeps its existing grouping until the next list is built (Shared Grocery List, FEAT-06).
- XBR-11: the household's aisle names apply consistently everywhere they appear on the grocery list; this screen is one of the two places (with FEAT-16.SPEC-001) where locale settings are changed.
- Only Maya may rename, reorder, add, or remove aisle groupings; Sam and Riley may only view them, per FEAT-16.SPEC-003's Authorization Rules.

## Edge Cases

- **Maya navigates away with unsaved renames, reorders, additions, or removals** -- No confirmation dialog is shown; the pending changes are simply discarded and the screen reverts to the last saved list on next entry, consistent with the feature's own States field ("changes save instantly," meaning there is no draft state to protect).
- **Maya taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in its inline saving state); no duplicate save is submitted.
- **Household's aisle_names changed by Maya on a second device between this screen's load and save** -- Save is accepted and applied last-write-wins per the dependency map's Contention note for the Household entity: whichever save reaches the server last becomes the active list, and a failed save keeps the prior list active rather than leaving a mixed state.
- **Maya renames an aisle to a name already used by another row** -- The save proceeds; duplicate aisle names are permitted (e.g., a household might intentionally use the same label for two physical sections), since the list is a free-text grouping name, not a unique key.
- **Maya removes every aisle row, leaving the list empty** -- Save is blocked with the inline error "Add at least one aisle before saving," since a grocery list with no aisle groupings has nowhere to place items; the same block applies when Maya taps Finish during guided setup with an empty pending list, since Finish runs the identical validate step.
- **Sam or Riley opens this screen while Maya is mid-edit on another device** -- Sam and Riley always see the last successfully saved list; they never see Maya's unsaved pending edits, since pending edits are local to Maya's own screen session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-001 (Units & Currency Settings) | Navigation (inbound/outbound) | Entry point and back-arrow destination; shared settings-area journey |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Navigation (outbound) | Guided setup's "Finish" completes the locale-configuration step and hands off here |
| FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults) | References (outbound) | Aisle-name length limit, list rules, and default-value derivation |
| FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) | Triggers (outbound) | A successful save triggers the rule that applies the new grouping to future grocery lists |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | References (outbound) | Consumes the saved aisle_names to group future grocery-list items |

## Analytics and Success Signals

{No success-metrics.md metric names Units, Currency & Locale Configuration as its Connected Feature; the event below is recorded per product-features.md's Signals field for this feature, but it cites N/A since this configuration screen's downstream effect is measured through the metrics of consuming features (e.g., Shared Grocery List's own Grocery List Live-Update Trust metric), not through a metric of its own.}

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| aisle_name_customized | count of aisles renamed, reordered, added, and removed in this save | Save succeeds with any change to the aisle_names list | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |
| aisle_customization_save_failed | count of pending changes at the time of failure | Save fails and the error state is shown | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |

## Acceptance Criteria

**FEAT-16.SPEC-002-AC-01:** Given Maya is on the Aisle Name Customization screen, when she renames "Frozen" to "Frozen Foods" and taps Save, then the household's aisle_names list is saved with the new name, an inline "Saved" confirmation appears, and FEAT-16.SPEC-004 is triggered so future grocery lists use the new name.

**FEAT-16.SPEC-002-AC-02:** Given Maya drags the "Bakery" row above the "Produce" row and taps Save, then the saved aisle_names order reflects Bakery before Produce.

**FEAT-16.SPEC-002-AC-03:** Given Maya taps "Add aisle," when a new empty row appears, then it is focused for typing, and Save is blocked with an inline error if she attempts to save while that row's name is still empty.

**FEAT-16.SPEC-002-AC-04:** Given Maya removes every aisle row from the pending list, when she taps Save, then the inline error "Add at least one aisle before saving" appears and no save occurs.

**FEAT-16.SPEC-002-AC-05:** Given Maya taps Save and the save fails due to a lost connection, then the inline error banner "Couldn't save your changes. Check your connection and try again." appears with a Retry action, and the household's previously saved aisle_names list remains the active, displayed list.

**FEAT-16.SPEC-002-AC-06:** Given Maya loses connectivity while this screen is open, when she renames an aisle, then the rename still applies to the pending list on screen, but the Save button is disabled and the offline banner is shown.

**FEAT-16.SPEC-002-AC-07:** Given Sam (Other Adult Member) opens this screen, when it loads, then he sees the household's aisle_names in their saved order as static text, with no drag handles, remove controls, "Add aisle" action, or Save button.

**FEAT-16.SPEC-002-AC-08:** Given Riley (Operator, support) is viewing this screen through an open Support Request, when the screen loads, then Riley sees the same non-interactive display Sam sees.

**FEAT-16.SPEC-002-AC-09:** Given Jordan as a young kid profile has no login, when any attempt is made to reach this screen, then no such path exists.

**FEAT-16.SPEC-002-AC-10:** Given an unauthenticated visitor requests this screen's URL directly, then they are redirected to the sign-in screen, and after signing in they land on the Household Settings Hub (FEAT-01.SPEC-010) rather than directly on this screen.

**FEAT-16.SPEC-002-AC-11:** Given Maya reaches this screen from guided setup immediately after confirming units and currency, when the screen first loads, then the aisle list is pre-filled with FEAT-16.SPEC-003's default aisle groupings rather than appearing blank, and tapping Finish saves that list (as confirmed or as edited) and completes setup by navigating to FEAT-01.SPEC-009.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (loaded, editing, saving, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Locale Configuration Validation & Defaults

## Overview

**Name:** Locale Configuration Validation & Defaults
**ID:** FEAT-16.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the supported unit and currency sets, the aisle-name and aisle-list rules, the authorization rules for changing locale settings, and the default unit system, currency, and aisle groupings a new household starts with.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration
**Governed Entity:** Household -- locale fields (unit_system, currency, aisle_names)

## Scope and Non-Goals

**In Scope:**
- The supported unit_system set and its validation
- The supported currency set and its validation
- The aisle_names list's per-item length rule and list-level rules (minimum count, duplicate handling)
- Authorization rules for viewing and changing every locale field
- Default unit_system, currency, and aisle_names values applied when a new Household record is created
- Error messages for every locale-field validation failure

**Non-Goals:**
- The screens' layout, interactions, and save-failure recovery experience -- owned by FEAT-16.SPEC-001 (Units & Currency Settings) and FEAT-16.SPEC-002 (Aisle Name Customization), which enforce these rules but do not define them
- How a saved locale-setting change is reflected across other features' displayed figures -- governed by FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule)
- Validation of any other Household field (household_name, weekly_budget, weekly_schedule, plan_arrival_day_time, status, organiser) -- excluded per the Feature Dependency Map's Household entity definition, which assigns those fields to FEAT-01, FEAT-07, and FEAT-14 respectively; this spec addresses only the three locale fields named in its Governed Entity
- Automatic locale detection from the device or browser as a source for the default values -- excluded per feature-overview.md's Non-Goals: defaults are derived from setup-time organiser input and confirmed on screen, never silently auto-applied from device signals

## Governed Entity

**Entity:** Household -- locale fields only
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| unit_system | enum | The household's default measurement unit system: cups/oz (US customary) or grams/ml (metric) |
| currency | enum | The household's currency for budget and cost display, from the supported currency set |
| aisle_names | ordered list of text | The household's aisle groupings and their display order, used to group the shared grocery list |
| aisle_names[].name | text | One aisle grouping's display name (e.g., "Produce," "Dairy & Eggs") |
| aisle_names[].order | derived (position) | The aisle grouping's position in the list, determined by its index |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | On selection (only supported-set values are offered) and on Save (re-confirmed before writing to the Household record); authorization on screen entry (edit controls rendered only for Maya) |
| FEAT-16.SPEC-002 | Aisle Name Customization | On field blur (per-name length rule) and on Save (full list re-validated, including the minimum-count rule); authorization on screen entry (edit controls rendered only for Maya) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On Household record creation -- this spec's default-derivation rules populate unit_system, currency, and aisle_names at the moment the Household record is created, before FEAT-16.SPEC-001 is first shown |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| unit_system | Must be one of the supported unit systems: "cups/oz" (US customary) or "grams/ml" (metric) | Always | On selection and on submit | "Choose a measurement unit system to continue." | Yes |
| currency | Must be one of the supported currencies: at minimum "USD" (US Dollar) and "GBP" (British Pound) at launch | Always | On selection and on submit | "Choose a currency to continue." | Yes |
| aisle_names[].name | Required, non-empty after trimming whitespace, 1-30 characters | Always, per row | On blur and on submit | "Give this aisle a name." (empty) / "Aisle names must be 30 characters or fewer." (too long) | Yes |
| aisle_names (list) | The list must contain at least one aisle | Always | On submit | "Add at least one aisle before saving." | Yes |
| aisle_names[].order | No validation beyond data type -- position is derived automatically from the row's index in the saved list, never entered directly | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Duplicate aisle names permitted | aisle_names[].name | No uniqueness constraint across rows -- two aisles may share the same display name, since aisle_names entries are free-text grouping labels, not unique keys | N/A -- no error; duplicates are allowed |
| Currency independent of unit_system | unit_system, currency | The two fields validate and save independently; no combination of unit_system and currency is disallowed (e.g., "grams/ml" with "USD" is valid, matching a household that has moved between the US and UK) | N/A -- no combination is invalid |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View unit_system, currency | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Always (Riley: only against a household with an open Support Request, per FEAT-22) | -- |
| View unit_system, currency | Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later) | Never | No login exists for the young-kid row, so the screen is unreachable; the older-kid login has no navigation entry point to it and a direct link redirects to that login's home screen |
| Change unit_system, currency | Maya (Organiser) | Always | -- |
| Change unit_system, currency | Sam (Other Adult Member) | Never | Edit controls are not rendered on FEAT-16.SPEC-001 for Sam; the screen shows the current values as static, non-interactive text |
| Change unit_system, currency | Riley (Operator, support) | Never | Edit controls are not rendered on FEAT-16.SPEC-001 for Riley; Operator Read-Only Support Access (FEAT-22) grants view only |
| View aisle_names | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Always (Riley: only against a household with an open Support Request, per FEAT-22) | -- |
| View aisle_names | Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later) | Never | Same as above -- no reachable path to FEAT-16.SPEC-002 |
| Rename, reorder, add, or remove an aisle | Maya (Organiser) | Always | -- |
| Rename, reorder, add, or remove an aisle | Sam (Other Adult Member) | Never | Edit controls (drag handle, remove control, "Add aisle" action) are not rendered on FEAT-16.SPEC-002 for Sam; the screen shows the current list as static text |
| Rename, reorder, add, or remove an aisle | Riley (Operator, support) | Never | Edit controls are not rendered on FEAT-16.SPEC-002 for Riley; Operator Read-Only Support Access (FEAT-22) grants view only |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| unit_system | Defaults to "cups/oz" (US customary) as the initial pre-filled value shown on FEAT-16.SPEC-001 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya confirms or changes it on FEAT-16.SPEC-001 before it is saved; this is a starting point for her explicit confirmation, not a silent auto-apply |
| currency | Defaults to "USD" (US Dollar) as the initial pre-filled value shown on FEAT-16.SPEC-001 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya confirms or changes it on FEAT-16.SPEC-001 before it is saved |
| aisle_names | Defaults to a standard seven-grouping list, in this order: "Produce," "Meat & Seafood," "Dairy & Eggs," "Bakery," "Pantry & Dry Goods," "Frozen," "Household & Other" -- a generic grouping common to both US and UK supermarkets, shown pre-filled on FEAT-16.SPEC-002 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya reviews, renames, reorders, adds to, or removes from this list on FEAT-16.SPEC-002 before it is saved |

## Business Rules

- Every locale field has a value from the moment the Household record is created -- there is no "unset" or null state for unit_system, currency, or aisle_names, consistent with product-features.md's States field for this feature ("Empty: N/A -- every household has a default locale configuration from creation").
- XBR-11: the supported unit and currency sets, and the aisle-name rules defined here, apply consistently everywhere locale settings are read (plan cost and weekly total, recipe quantities, grocery-list aisle grouping and quantities, budget, check-in spend) -- FEAT-16.SPEC-004 governs the display conversion itself, but the values it converts to or from must always come from this spec's supported sets.
- The default-derivation rule runs exactly once, at Household creation -- it is never re-applied afterward, even if a household later clears all its locale settings through some future capability; there is no such capability in this product definition, so this scenario does not currently arise.
- The supported currency set may be extended beyond USD and GBP in a future release without changing this spec's structure -- product-features.md's Validation & Limits field states the set covers "at least USD and GBP at launch," leaving room for growth, but this spec's rules apply to whatever the current supported set is at any time.

## Edge Cases

- **Aisle name at exactly 30 characters** -- Passes validation. 31 characters shows the length error.
- **Aisle name that is only whitespace** -- Treated as empty after trimming; shows "Give this aisle a name."
- **Maya removes all aisle rows in one edit session, then adds one back before saving** -- The minimum-count rule is evaluated only at submit time against the final pending list, so this sequence saves successfully once at least one aisle remains.
- **A household created before this spec's default aisle list changes (a future product change)** -- Not applicable to this run: the default-derivation rule runs once at creation and existing households' saved aisle_names are never overwritten by a later change to the default list; only new households receive the updated defaults.
- **Maya selects "grams/ml" for unit_system while currency remains "USD"** -- Both save successfully; no cross-field rule disallows this combination (a household that relocated from the UK to the US, for example, may want metric units with US dollars).
- **Two aisle rows are given the exact same name** -- Both save successfully; duplicate names are permitted per the Cross-Field Rules above.
- **Sam or Riley attempts to submit a direct edit request to a locale field by bypassing the screen's rendered controls** -- The authorization check runs independently of what the screen displays; the change is rejected and the field's value is unchanged, since Change actions are Never for these roles regardless of how the attempt reaches the system.

## Acceptance Criteria

**FEAT-16.SPEC-003-AC-01:** Given Maya is choosing a unit system on FEAT-16.SPEC-001, when only "cups/oz" and "grams/ml" are offered, then she cannot select or submit any other value.

**FEAT-16.SPEC-003-AC-02:** Given Maya is choosing a currency on FEAT-16.SPEC-001, when the selector lists the supported currencies, then "US Dollar" and "British Pound" both appear among the options.

**FEAT-16.SPEC-003-AC-03:** Given Maya leaves an aisle name empty on FEAT-16.SPEC-002 and moves focus away from the field, then the field shows the error "Give this aisle a name."

**FEAT-16.SPEC-003-AC-04:** Given Maya enters an aisle name of exactly 30 characters, then no length error is shown; given she enters 31 characters, then the error "Aisle names must be 30 characters or fewer." appears.

**FEAT-16.SPEC-003-AC-05:** Given Maya removes every aisle row and taps Save on FEAT-16.SPEC-002, then the error "Add at least one aisle before saving." appears and no save occurs.

**FEAT-16.SPEC-003-AC-06:** Given Maya renames two different aisles to the same name and taps Save, then the save succeeds without any duplicate-name error.

**FEAT-16.SPEC-003-AC-07:** Given Maya (Organiser) is on FEAT-16.SPEC-001, when she changes and saves the unit_system, then the change is applied, since Change is allowed for Maya always.

**FEAT-16.SPEC-003-AC-08:** Given Sam (Other Adult Member) is on FEAT-16.SPEC-001, when he looks for an edit control on the unit_system or currency selector, then none is shown, since Change is never allowed for Sam.

**FEAT-16.SPEC-003-AC-09:** Given Riley (Operator, support) is viewing FEAT-16.SPEC-002 through an open Support Request, when Riley looks for a drag handle or remove control on any aisle row, then none is shown, since Change is never allowed for Riley.

**FEAT-16.SPEC-003-AC-10:** Given Jordan as a young kid profile has no login, when any attempt is made to view or change locale settings, then no such path exists, since View and Change are both Never for this role.

**FEAT-16.SPEC-003-AC-11:** Given a new Household record is created through FEAT-01.SPEC-003, when the default-derivation rule runs, then unit_system is set to "cups/oz," currency is set to "USD," and aisle_names is set to the standard seven-grouping default list, all before Maya reaches FEAT-16.SPEC-001.

**FEAT-16.SPEC-003-AC-12:** Given Maya reaches FEAT-16.SPEC-001 immediately after household creation, when she changes the pre-filled unit_system default from "cups/oz" to "grams/ml" and confirms, then her chosen value overrides the default and is what saves.

**FEAT-16.SPEC-003-AC-13:** Given Maya selects "grams/ml" for unit_system and leaves currency as "USD," when she saves, then both values save successfully, since no cross-field rule links the two.

**FEAT-16.SPEC-003-AC-14:** Given Maya enters an aisle name of only spaces, when she moves focus away from the field, then it is treated as empty and shows "Give this aisle a name."

**FEAT-16.SPEC-003-AC-15:** Given a request to change Sam's currency-editing permission bypasses the FEAT-16.SPEC-001 screen entirely, when the change is evaluated against this spec's Authorization Rules, then it is rejected and the household's currency remains unchanged, since Change is Never for Sam regardless of how the request arrives.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Cross-Feature Value Conversion Rule

## Overview

**Name:** Cross-Feature Value Conversion Rule
**ID:** FEAT-16.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines how stored quantities and costs are converted, at the moment they are displayed, into the household's currently configured unit system and currency, so that a locale change is reflected immediately and consistently everywhere those values appear, without leaving any displayed figure inconsistent with the household's current settings.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration
**Governed Entity:** Displayed quantities and costs across the Household, Weekly Plan, Planned Meal, Recipe, Grocery List Item, and Waste & Spend Check-In entities (a cross-feature derivation rule, not a single entity's own CRUD lifecycle)

## Scope and Non-Goals

**In Scope:**
- The conversion logic applied to any stored quantity (e.g., a recipe ingredient amount) when the household's unit_system differs from the unit system the value was originally recorded in
- The conversion logic applied to any stored monetary figure (budget, plan totals, meal costs, check-in spend) for display under the household's currently configured currency
- The rule that a locale-setting change takes effect for display immediately, on every screen that shows an affected figure, without a delayed or batch update
- Rounding and formatting behavior for converted values
- Which entities' fields this rule touches, by exact field name, across the six affected entities

**Non-Goals:**
- Real-time currency exchange-rate conversion between USD, GBP, or any other supported currency -- excluded because no currency-exchange capability appears in the Feature Dependency Map's External Touchpoints table, and product-features.md's Validation & Limits field frames currency as a display/labeling setting ("currency setting determines how the household's existing budget figure is labeled and displayed"), not a financial conversion; a currency change relabels the same stored numeric figure under the new currency's symbol (see Defaults and Derivations)
- Defining the supported unit and currency sets, or the aisle-name rules -- governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults); this spec only converts values that are already valid under that spec's supported sets
- The screens where unit_system, currency, or aisle_names are changed -- owned by FEAT-16.SPEC-001 and FEAT-16.SPEC-002, which trigger this rule on save but do not define its conversion logic
- Re-grouping an already-generated grocery list's aisle assignments when aisle names change -- per feature-overview.md's Side-Effect Inventory, an aisle-name change applies to future grocery lists only; this spec's conversion behavior for quantities and costs is immediate, but aisle re-grouping is explicitly out of scope for an already-generated list

## Governed Entity

**Entity:** Displayed quantities and costs (cross-feature; each row below names its owning entity per the Feature Dependency Map)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Household.unit_system | enum | The household's target unit system for all quantity conversions (read from FEAT-16.SPEC-003's supported set) |
| Household.currency | enum | The household's target currency for all monetary display (read from FEAT-16.SPEC-003's supported set) |
| Household.weekly_budget | derived (display) | The household's budget figure, displayed under Household.currency |
| Weekly Plan.estimated_total | derived (display) | The week's estimated cost, displayed under Household.currency |
| Planned Meal.rough_cost | derived (display) | A single dinner's estimated cost, displayed under Household.currency |
| Recipe.ingredients[].quantity_and_unit | derived (display) | Each ingredient's amount, displayed under Household.unit_system |
| Grocery List Item.quantity_and_unit | derived (display) | Each combined grocery-list line's amount, displayed under Household.unit_system |
| Waste & Spend Check-In (spend figures) | derived (display) | The household's self-reported weekly spend answer, displayed under Household.currency |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | Triggers this rule on a successful save of unit_system or currency |
| FEAT-16.SPEC-002 | Aisle Name Customization | Triggers this rule on a successful save of aisle_names (for the aisle-grouping consequence only; quantities and costs are unaffected by an aisle-name change) |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | Applies this rule wherever weekly_budget is shown, per the household's currency |
| FEAT-03.SPEC-001 | Weekly Plan View | Applies this rule to Weekly Plan.estimated_total and each Planned Meal.rough_cost wherever the plan is shown |
| FEAT-03.SPEC-006 | Budget-Fit / Estimated Total Rule | Applies this rule's currency display to the estimated_total figure this rule computes |
| FEAT-06.SPEC-001 | Grocery List | Applies this rule to Grocery List Item.quantity_and_unit wherever the list is shown |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | Applies this rule to the combined quantity this spec derives, before display |
| FEAT-08.SPEC-002 | Recipe Detail View | Applies this rule to Recipe.ingredients[].quantity_and_unit wherever a recipe is viewed |
| FEAT-23.SPEC-001 | Weekly Plan Manual Week Builder | Applies this rule to the manually built week's cost figures, the same way FEAT-03.SPEC-001 does |
| FEAT-25 (Weekly Waste & Spend Check-In) | Check-in spend figure display | Applies this rule to the check-in's self-reported spend answer wherever it is shown -- referenced at feature level since FEAT-25's specs are not yet produced as of this run |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Household.unit_system, Household.currency | No validation beyond data type -- these are read, not written, by this spec; their own validation is governed by FEAT-16.SPEC-003 | Always | -- | -- | -- |
| Household.weekly_budget, Weekly Plan.estimated_total, Planned Meal.rough_cost, Waste & Spend Check-In spend figures | No validation beyond data type -- these are derived display values; this spec only formats them for display, it does not validate their underlying accuracy | Always | -- | -- | -- |
| Recipe.ingredients[].quantity_and_unit, Grocery List Item.quantity_and_unit | No validation beyond data type -- this spec only converts these values for display; their underlying accuracy is governed by the entity that authors them (Recipe by FEAT-08/FEAT-10, Grocery List Item by FEAT-06) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Unit conversion applies only when the source and target unit systems differ | Household.unit_system, Recipe.ingredients[].quantity_and_unit, Grocery List Item.quantity_and_unit | If the value's originally recorded unit system already matches Household.unit_system, it is displayed unconverted; otherwise the conversion table in Defaults and Derivations applies | N/A -- no error; this is a display computation, not a validation |
| Currency relabeling is independent of unit conversion | Household.currency, Household.unit_system | A change to one field never triggers or blocks conversion of the other -- unit_system changes affect only quantity display, currency changes affect only monetary display | N/A -- no error |
| A locale change never rewrites stored values | Household.unit_system, Household.currency, all governed display fields | Conversion happens at display time from the stored, originally recorded value -- the underlying stored quantity_and_unit or monetary figure is never overwritten by a locale change, so reverting unit_system or currency back to a prior value restores the original display exactly | N/A -- no error; this is what keeps the conversion reversible and lossless |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a locale-converted quantity or cost | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later, where that role can view the underlying screen, e.g., the Grocery List), Riley (Operator, support, where FEAT-22 grants view) | Per the viewing role's own access rules on the underlying screen (FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-16.SPEC-001, FEAT-23, FEAT-25); this spec adds no restriction of its own | -- |
| View a locale-converted quantity or cost | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role, so no screen showing a converted value is reachable |
| Change which unit system or currency values convert to | Maya (Organiser) | Always -- via FEAT-16.SPEC-001, not this spec directly | -- |
| Change which unit system or currency values convert to | Sam (Other Adult Member), Riley (Operator, support), both Jordan rows | Never | No control exists on any screen for these roles to change unit_system or currency, per FEAT-16.SPEC-003's Authorization Rules |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Recipe.ingredients[].quantity_and_unit (displayed) | If the ingredient's originally recorded unit belongs to the household's currently configured unit_system, display it unconverted. Otherwise, apply the fixed conversion factor for that unit: 1 cup -> 240 ml; 1 tablespoon -> 15 ml; 1 teaspoon -> 5 ml; 1 fluid ounce -> 30 ml; 1 ounce (weight) -> 28 grams; 1 pound -> 454 grams -- and the inverse factors when converting from metric back to US customary. Metric results round to the nearest whole gram or millilitre; US-customary results round to the nearest common cooking fraction (1/4, 1/3, 1/2, 2/3, 3/4, or whole unit). | Every time the ingredient is displayed (plan, recipe detail, grocery list) | No -- the household's unit_system is the single control; there is no per-view override |
| Grocery List Item.quantity_and_unit (displayed) | Same fixed conversion factors as Recipe ingredients above, applied to the combined quantity after Shared Grocery List (FEAT-06) has already summed the ingredient across dinners | Every time the grocery list is displayed | No |
| Household.weekly_budget, Weekly Plan.estimated_total, Planned Meal.rough_cost, Waste & Spend Check-In spend figures (displayed) | Displayed with the household's currently configured currency's symbol and standard formatting for that currency, applied to the stored numeric figure exactly as recorded -- no exchange-rate multiplication or division is applied; a currency change relabels the same number under the new symbol | Every time the figure is displayed | No |
| Any governed field, on a locale-setting change | Re-computed for display the next time each affected screen is shown -- no background batch job re-renders every past screen at once, and no stored value is rewritten; the conversion is applied fresh at each display | Immediately after FEAT-16.SPEC-001 or FEAT-16.SPEC-002 saves a change | No |

## Business Rules

- XBR-11: the household's units, currency, and aisle names apply consistently everywhere they appear -- plan cost and weekly total, recipe quantities, grocery-list aisle grouping and quantities, budget, and check-in spend -- and a later change converts existing figures for display rather than leaving them inconsistent. This spec is the mechanism that fulfills XBR-11 for quantities and monetary figures; FEAT-16.SPEC-001 and FEAT-16.SPEC-002 fulfill the aisle-grouping half directly.
- Conversion is a pure display computation: it never writes to Recipe, Grocery List Item, Weekly Plan, Planned Meal, or Waste & Spend Check-In records. Only the Household record's own unit_system and currency fields are ever written, and only by FEAT-16.SPEC-001 and FEAT-16.SPEC-002.
- Per the feature's Non-Functional Notes, a converted display must appear immediately wherever the plan, recipes, or grocery list are next shown -- there is no acceptable delay between a locale-setting save and the next screen reflecting it, since conversion happens at display time rather than through a background reflow.
- A recipe or grocery-list item authored in one unit system (e.g., a starter recipe written in cups/oz) is never edited to permanently change its authored units; this rule only affects what is shown to the currently viewing household, and two households with different unit_system settings can view the same starter recipe converted differently at the same time.

## Edge Cases

- **Maya changes unit_system, then immediately opens the current week's plan** -- Every ingredient quantity shown in the plan and its linked recipes reflects the new unit_system on that very screen load; there is no stale-display window, since conversion is computed fresh at display time.
- **Maya changes currency, then reviews a past archived Weekly Plan (Weekly Plan History, FEAT-19, v1)** -- The archived plan's estimated_total displays under the newly configured currency's symbol on the same stored figure; archiving does not freeze the display currency to whatever it was when the plan was active.
- **An ingredient quantity has no clean conversion result (e.g., 1/3 cup converts to 79 ml)** -- The metric result rounds to the nearest whole millilitre (79 ml, not a fraction); this is display rounding only and never alters the stored authored quantity.
- **Maya changes unit_system and currency in the same save session (both fields changed on FEAT-16.SPEC-001 at once)** -- Both conversions apply independently and simultaneously on the next display of any affected screen; there is no ordering dependency between the two.
- **Maya reverts unit_system from metric back to US customary after having changed it earlier** -- Displayed quantities return exactly to their originally authored US-customary values, since the underlying stored quantity_and_unit was never overwritten by the earlier conversion.
- **A household's currency is changed while its Weekly Plan.estimated_total is mid-calculation (a plan is still generating)** -- The in-progress calculation completes and stores its result as before; the newly configured currency's symbol is applied only when the completed total is displayed, not to the calculation itself.
- **Sam views the grocery list on his phone at the same moment Maya changes unit_system on hers** -- Sam sees the list re-render in the new unit_system the next time his screen loads or refreshes the list; per the Grocery List's own live-update behavior (FEAT-06), this is consistent with values never being shown "without its matching list" stated in XBR-03, extended here to unit display.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Maya changes the household's unit_system from cups/oz to grams/ml and saves on FEAT-16.SPEC-001, when she next opens a recipe originally authored in cups, then its ingredient quantities display converted to grams/ml using the fixed conversion factors, with metric results rounded to the nearest whole gram or millilitre.

**FEAT-16.SPEC-004-AC-02:** Given a household's unit_system is already grams/ml, when Maya opens a recipe originally authored in grams/ml, then its quantities display unconverted, since the source and target unit systems match.

**FEAT-16.SPEC-004-AC-03:** Given Maya changes the household's currency from USD to GBP and saves, when she next opens the current week's plan, then the plan's estimated_total and each meal's rough_cost display with the £ symbol applied to the same stored numeric figures, with no exchange-rate multiplication applied.

**FEAT-16.SPEC-004-AC-04:** Given Maya saves a currency change, when Sam opens the Household Setup screen showing weekly_budget, then the same stored budget number displays under the newly configured currency's symbol.

**FEAT-16.SPEC-004-AC-05:** Given Maya changes unit_system, when Sam opens the shared Grocery List, then its combined quantities display converted to the new unit_system using the same fixed conversion factors applied to the summed amount.

**FEAT-16.SPEC-004-AC-06:** Given a stored ingredient quantity of 1/3 cup, when the household's unit_system is grams/ml, then the displayed value rounds to the nearest whole millilitre rather than showing a fractional millilitre amount.

**FEAT-16.SPEC-004-AC-07:** Given Maya changes both unit_system and currency in the same save on FEAT-16.SPEC-001, when she next opens the plan, then both the displayed quantities and the displayed costs reflect their respective new settings on that same load.

**FEAT-16.SPEC-004-AC-08:** Given Maya changed unit_system to metric last week and now changes it back to cups/oz, when she opens a recipe originally authored in cups, then its quantities display exactly as originally authored, since the stored value was never overwritten by the earlier conversion.

**FEAT-16.SPEC-004-AC-09:** Given Maya reviews an archived past Weekly Plan after changing the household's currency, then the archived plan's estimated_total displays under the newly configured currency, not the currency that was active when the plan was archived.

**FEAT-16.SPEC-004-AC-10:** Given the household's currency is USD, when Maya answers the Weekly Waste & Spend Check-In's spend question, then the answer displays with the $ symbol, and if she later changes currency to GBP, the same recorded answer displays with the £ symbol on next view.

**FEAT-16.SPEC-004-AC-11:** Given Jordan is a young kid profile with no login, when any attempt is made to view a locale-converted value, then no such path exists, since no screen is reachable for this role.

**FEAT-16.SPEC-004-AC-12:** Given Riley (Operator, support) views a household's plan through an open Support Request, when the plan's costs are shown, then they display converted per the household's own currency, the same as any other viewer, since this spec adds no role-specific restriction beyond the underlying screen's own access rules.

**FEAT-16.SPEC-004-AC-13:** Given a starter recipe authored in cups/oz is viewed by two different households, one configured for cups/oz and one configured for grams/ml, when both view the same recipe at the same time, then the first sees the authored cups/oz quantities unconverted and the second sees the grams/ml converted equivalents, without the underlying Recipe record being modified for either household.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

---
document_type: spec
spec_type: screen
spec_id: FEAT-16.SPEC-001
spec_name: Units & Currency Settings
spec_slug: units-currency-settings
parent_feature: FEAT-16
parent_feature_name: Units, Currency & Locale Configuration
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

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

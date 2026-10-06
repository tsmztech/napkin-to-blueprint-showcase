---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-002
spec_name: Add Service
spec_slug: add-service
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 10
---

# Screen Spec: Add Service

## Overview

**Name:** Add Service
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** The Pro creates a new bookable service by entering its name, price, duration, and deposit rule, with an inline preview of how it will appear on the client-facing booking page.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Capturing name, price, duration, and deposit rule (fixed amount or percentage) for a new service
- Inline field validation and a live client-facing preview as the Pro types
- Saving the new service as Active, appended to the end of the Pro's display order
- The entry path used both from the Service List and from first-time setup (FEAT-15)

**Non-Goals:**
- Editing an existing service, or archiving one -- handled by FEAT-01.SPEC-003 (Edit Service)
- Setting the per-service buffer override -- owned entirely by FEAT-02 (Availability & Working Hours Setup); this screen never shows or captures that field
- Handling or storing card data as part of the deposit rule -- excluded per scope-boundaries.md SC-11: this screen only defines the deposit rule (fixed amount or percentage); actual card capture belongs to the payment-processing capability behind FEAT-07
- Dynamic, demand-based, or time-of-day pricing -- excluded per scope-boundaries.md SC-14: each service carries exactly one fixed price

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Service List) | Pro taps "Add Service" | None -- form starts empty |
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) | Pro reaches the services step of first-time setup | None -- form starts empty; on save, the Pro continues into the next onboarding step (working hours) instead of returning to FEAT-01.SPEC-001 |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Fill and save the form | -- |
| Platform Operator (Support) | No | No | Support's View access to Service & Availability Setup applies to existing service records (FEAT-01.SPEC-001, FEAT-01.SPEC-003) for troubleshooting; there is nothing yet to troubleshoot on a creation form, so this screen is never opened in a support context and no control is rendered for it |
| The Client (Riley) | No | No | Service & Pricing Management screens are never reached through any Client-facing path |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Service" with a back arrow (returns to the entry source) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Price (numeric input, required, in the Pro Account's currency)
- Duration (numeric input in minutes, required, with quick-select common durations)
- Deposit Rule (a two-way toggle between "Fixed amount" and "Percentage," followed by the corresponding numeric input)

Below the form, a live preview panel shows exactly what a client will see on the booking page for this service: name, price, duration, and the deposit amount that results from the current price and deposit rule, recalculated as the Pro types.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; the preview panel appears below the form, stacked.
- **Medium size class and above:** Form and preview panel appear side by side (form on the left, live preview on the right), the whole layout capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry source (FEAT-01.SPEC-001, or the next-earlier onboarding step if entered from FEAT-15) | Screen closes | Standard back transition |
| Name input | Type | Captures text input | Field shows entered text; preview panel's name updates live | Standard input focus state |
| Name input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Price input | Type | Captures numeric input | Field shows entered value; preview panel's price and computed deposit update live | Standard input focus state |
| Price input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Duration input | Type or quick-select | Captures duration in minutes | Field shows entered value; preview panel's duration updates live | Standard input focus state |
| Duration input | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Deposit Rule toggle (Fixed/Percentage) | Tap | Switches the deposit input's mode; clears the previously entered deposit value | Deposit input relabels and resets | Preview panel's computed deposit updates to reflect the cleared value |
| Deposit value input | Type | Captures the fixed amount or percentage, per the toggle's mode | Field shows entered value; preview panel's computed deposit updates live | Standard input focus state |
| Deposit value input | Blur | Triggers field and cross-field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Save button | Tap | 1. Validate all fields and the deposit-rule cross-field rules via FEAT-01.SPEC-004. 2. If valid, create the Service record. | Button shows loading state during save | Success: toast "{Service name} added -- it's live on your booking page" and navigate per Entry Points context. Failure: inline error messages, entered values preserved. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Price -> Duration -> Deposit Rule toggle -> Deposit value -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Preview updates:** The live preview panel is not part of the primary focus order and its updates are not separately announced on every keystroke, to avoid announcement noise; its final state is reachable on demand via a labeled "Preview" landmark.
- **Save feedback:** The success toast is announced on save; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen, including the Deposit Rule toggle, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All form fields empty, preview panel shows a placeholder card, Save enabled | Screen first opens | Pro begins typing in any field |
| Filling | Form fields contain Pro input, preview panel reflects current values, Save enabled | Pro types in any field | Pro taps Save or navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails on blur or submit | Pro corrects the field and re-triggers validation |
| Saving | Save button shows a loading state, form fields disabled | Validation passes | Save completes or fails |
| Error | Error banner at the top of the form: "Couldn't save this service. Try again." with a Retry action | Save operation fails | Pro taps Retry and the save succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow (product-features.md, States field) |

## Validation Rules

Validation governed by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation). See that spec for all field-level and cross-field rules, including the minimum chargeable deposit. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Service List) | -- |
| Back arrow tap (entered from onboarding) | Previous onboarding step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Successful save (entered from list) | FEAT-01.SPEC-001 (Service List) | -- |
| Successful save (entered from onboarding) | Next onboarding step (working hours) | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Cancel (if unsaved changes) | Entry source, after confirmation | -- |

## Data Model

**Creates:** Service record -- name, price, duration, and deposit_rule set from form input; status set to Active automatically; display_order appended to the end of the Pro's current Active list automatically; buffer_override left unset (owned by FEAT-02).
**Reads:** None -- this is a creation screen; no existing data loaded.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation and the minimum chargeable deposit are enforced by FEAT-01.SPEC-004 -- the Pro cannot save with invalid or unchargeable values.
- XBR-25: price is entered and displayed in the Pro Account's currency (owned by FEAT-27); this screen never lets the Pro choose a different currency per service.
- Once saved, the service is immediately visible on the public booking page (FEAT-05) -- there is no separate publish step.
- A service created here is, from that moment, subject to FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time): any future edit or archive of this service will never retroactively change a booking confirmed against it.

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network or save failure** -- Error banner: "Couldn't save this service. Try again." with a Retry button. All entered values are preserved.
- **Pro switches the Deposit Rule toggle after entering a value** -- The previously entered deposit value is cleared (not silently reinterpreted under the new mode), and the preview panel's computed deposit reflects the cleared value until the Pro re-enters one.
- **Two identically named services** -- No uniqueness rule applies to the service name (FEAT-01.SPEC-004 defines no name-uniqueness rule); the save proceeds and the Pro's list simply shows two rows with the same name, distinguished by their other fields.
- **Another service is created or edited by the Pro from a second device while this form is open** -- No live conflict on this creation screen: no existing record is loaded, and this screen's own save only ever creates a new record, so it cannot collide with any other in-flight change to a different or the same service.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | References (inbound) | Validation and the minimum-chargeable-deposit rule applied to form fields |
| FEAT-01.SPEC-001 (Service List) | Navigation (inbound/outbound) | Pro arrives from the list and returns to it on save or back |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard, Setup Wizard step) | Navigation (inbound/outbound) | Pro's first service is added here during setup, then continues into the next onboarding step |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (outbound) | The service created here immediately becomes subject to the lock guarantee for any future booking |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_created | entry source (service list / onboarding), deposit rule form (fixed / percentage) | Successful save completes | supports success-metrics.md: "Service Setup Confidence" |
| service_create_validation_failed | field(s) in error, error type(s) | Save attempt is blocked by validation | supports success-metrics.md: "Service Setup Confidence" |
| service_create_abandoned | last field focused, count of fields filled | Pro discards unsaved changes via the back arrow | supports success-metrics.md: "Service Setup Confidence" |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Talia is on the Add Service screen, when she fills in "Classic Lash Set" as name, a positive price, a duration in 5-minute steps, and a valid deposit rule, and taps Save, then the service is created as Active, appended to the end of her service list, and she sees the toast "Classic Lash Set added -- it's live on your booking page" before returning to FEAT-01.SPEC-001.

**FEAT-01.SPEC-002-AC-02:** Given Talia is on the Add Service screen with the name field empty, when she taps Save, then the name field shows an error state and the save does not proceed.

**FEAT-01.SPEC-002-AC-03:** Given Talia enters a price and switches the Deposit Rule toggle from Fixed to Percentage, when the toggle switches, then any previously entered fixed-amount value is cleared and the preview panel's computed deposit updates to reflect the cleared value.

**FEAT-01.SPEC-002-AC-04:** Given Talia enters a price and a percentage deposit rule whose resulting deposit falls below the minimum chargeable amount, when she blurs the deposit field, then FEAT-01.SPEC-004's validation error appears and Save is blocked until she adjusts the price or the rule.

**FEAT-01.SPEC-002-AC-05:** Given Talia is on the Add Service screen with unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-01.SPEC-002-AC-06:** Given Talia types values into the form, when each keystroke registers, then the live preview panel updates to show the name, price, duration, and computed deposit exactly as a client would see them.

**FEAT-01.SPEC-002-AC-07:** Given Talia reaches this screen from the services step of first-time setup (FEAT-15), when she saves a valid service, then she continues into the next onboarding step (working hours) rather than returning to FEAT-01.SPEC-001.

**FEAT-01.SPEC-002-AC-08:** Given Talia's save fails due to a save error, when the failure occurs, then an error banner reads "Couldn't save this service. Try again." with a Retry action, and all entered field values remain on screen.

**FEAT-01.SPEC-002-AC-09:** Given Talia's session expires while she has partially filled the form, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears, and her entered values are restored after she signs back in.

**FEAT-01.SPEC-002-AC-10:** Given Talia saves a second service with the same name as an existing one, when she taps Save, then the save proceeds without any duplicate-name warning, since no name-uniqueness rule exists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 6 (empty, filling, validation error, saving, error, offline/degraded N/A) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

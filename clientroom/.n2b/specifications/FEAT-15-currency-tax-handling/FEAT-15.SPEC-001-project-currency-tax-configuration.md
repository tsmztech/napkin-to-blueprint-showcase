---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-001
spec_name: Project Currency & Tax Configuration
spec_slug: project-currency-tax-configuration
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Project Currency & Tax Configuration

## Overview

**Name:** Project Currency & Tax Configuration
**ID:** FEAT-15.SPEC-001
**Type:** Screen
**Purpose:** Nadia sets a project's billing currency and tax label/rate during billing setup, and views the locked, read-only version of that configuration once the project's first invoice has been sent; Dana views the same configuration read-only inside a logged support session.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Displaying and capturing a project's currency, tax label, and tax rate (or no tax) during billing setup
- Showing the owning client's name for context while Nadia sets the values
- Replacing the editable fields with a locked, read-only display once the project's first invoice has been sent
- Surfacing the empty-configuration prompt that blocks invoice generation until currency/tax is set
- Surfacing validation and lock errors inline, sourced from FEAT-15.SPEC-003 and FEAT-15.SPEC-004
- Dana's read-only view of this same configuration inside a support session

**Non-Goals:**
- Deciding which currencies and tax shapes are valid -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this screen only surfaces that spec's outcome
- Deciding when the configuration becomes locked -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this screen only renders the locked state
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): the tax line is a freelancer-configured label and rate only, never a computed jurisdictional rule
- Creating, renaming, or archiving the Project record itself -- owned entirely by Client & Project Management (FEAT-01); this screen only sets two of that record's fields
- Currency conversion of any kind -- excluded per XBR-18 and this feature's own Non-Goals: the product never converts between currencies, so no conversion control ever appears here

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management, project billing setup) | Nadia opens a project's billing setup for the first time or to review it | Project reference, owning client reference |
| FEAT-31 (Operator Support Access) | Dana opens a project's billing setup inside a logged, read-only support session | Project reference, read-only session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen for any of her own projects | Set or change currency and tax label/rate while unlocked (FEAT-15.SPEC-003 validates every save); view-only once locked (FEAT-15.SPEC-004) | -- |
| Owen (Client Primary Contact) | None -- this screen is entirely absent from his navigation | None | A direct link to this screen redirects to his portal home with no partial content ever rendered, per FEAT-15.SPEC-005 |
| Priya (Client Reviewer Contact) | None -- this screen is entirely absent from her navigation | None | A direct link to this screen redirects to her portal home with no partial content ever rendered, per FEAT-15.SPEC-005 |
| Dana (Support Operator) | Full content, read-only, inside a logged support session (FEAT-31) | View only -- no save control is ever rendered | The Save control and every editable input are not rendered; a direct attempt to submit a change (e.g., a replayed request) is refused and the screen re-renders in its read-only form, per FEAT-15.SPEC-005 |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Redirected to sign-in; any unsaved currency/tax entry is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Currency & Tax" with the owning client's name shown directly beneath it for context, and a back control returning to the project's billing setup area (FEAT-01).

**Body, unlocked state (before the first invoice is sent):**
- **Currency selector** (selection input, required): a searchable list of recognized world currencies; no value is preselected.
- **Tax treatment group:**
  - "No tax line" toggle -- when on, the tax label and rate inputs below are hidden and no tax line is applied.
  - Tax label (text input, required unless "No tax line" is on) -- freelancer-entered text such as "VAT," "GST," or "Sales Tax."
  - Tax rate (numeric percentage input, required unless "No tax line" is on).
- **Save button**, at the bottom of the form.

**Body, locked state (once the project's first invoice has been sent):**
- The currency, tax label, and tax rate fields render as a plain, non-editable display (the same visual treatment FEAT-09 uses for "not edited after sending") with a short explanation directly beneath: "Currency and tax are fixed after your first invoice."
- No Save button is rendered.

**Body, empty-configuration prompt (surfaced when Nadia or the system reaches invoice generation before this screen has ever been completed):**
- A banner above the form: "Set a currency and tax line before you can generate an invoice for this project," present until the first successful save.

**Body, Dana's read-only view:**
- The same locked-state layout regardless of whether the project's first invoice has been sent, with a small "Viewing read-only in a support session" label beneath the header and no Save button.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains directly below the last field.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-01's project billing setup | Screen closes | Standard transition |
| Currency selector | Select | Captures the chosen currency | Field shows chosen currency | Standard selection state |
| "No tax line" toggle | Toggle on | Hides tax label and rate inputs; clears any entered values | Tax label/rate inputs disappear | Fields collapse from the form |
| "No tax line" toggle | Toggle off | Shows tax label and rate inputs, empty | Tax label/rate inputs reappear | Fields expand into the form |
| Tax label input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Tax rate input | Type | Captures numeric input | Field shows entered value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-15.SPEC-003. 2. If valid, save the project's currency and tax fields. | Button shows loading state during save | Success: toast "Currency and tax saved" and the screen re-renders with the saved values. Failure: inline field-level errors per FEAT-15.SPEC-003. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back control -> empty-configuration banner (when present) -> currency selector -> "No tax line" toggle -> tax label input (when visible) -> tax rate input (when visible) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Lock announcement:** When the screen renders in its locked state, the "currency and tax are fixed after your first invoice" explanation is announced on load.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | N/A -- per product-features.md's States field for FEAT-15 ("Loading: N/A -- configuration is local"), this screen has no distinct loading state of its own: currency, tax label, and tax rate are fields on the Project record already fetched as part of opening the project's billing setup (FEAT-01), so the form renders immediately into Empty, Configured, or Locked with no separate fetch-in-progress appearance | N/A | N/A |
| Empty (unlocked, never configured) | Empty-configuration banner shown above an empty form; Save enabled once required fields are filled | Screen first opens on a project with no currency/tax ever saved | Nadia begins filling the form |
| Filling | Form fields contain user input, Save button enabled | Nadia types or selects in any field | Nadia taps Save or navigates away |
| Saving | Save button shows loading spinner, form fields disabled | Nadia taps Save | Save completes or fails |
| Validation Error | Failed fields highlighted with error messages below them, per FEAT-15.SPEC-003 | Validation fails on save | Nadia corrects the field and re-saves |
| Configured (unlocked) | Form fields show the saved currency, tax label, and tax rate; Save remains enabled for further edits | Save succeeds and the project has not yet had a first invoice sent | Nadia edits and re-saves, or the project's first invoice is sent (transitions to Locked) |
| Locked | Plain, non-editable display of currency/tax with the "fixed after your first invoice" explanation; no Save control | The project's first invoice is sent (FEAT-15.SPEC-004) | Never -- the lock is permanent for this project |
| Error | Error banner "Couldn't save your currency and tax settings. Check your connection and try again." with Retry | Save operation fails for a reason other than validation (e.g., connectivity) | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- currency and tax changes can't be saved right now." at top; form remains viewable but Save is disabled | Connectivity is lost while the screen is open | Connectivity returns and Save re-enables |

## Validation Rules

Validation governed by FEAT-15.SPEC-003 (Currency & Tax Validation Rules). See that spec for all field-level rules on currency and tax label/rate. This screen applies validation on Save; the lock check (FEAT-15.SPEC-004) is applied before the form even renders as editable.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | Project billing setup | FEAT-01 |
| Successful save | This screen, re-rendered with the saved values | -- |

## Data Model

**Creates:** None -- the Project record itself is created by FEAT-01; this screen never creates a Project.
**Reads:** Project -- `currency`, `tax_label`, `tax_rate`, `client` (for the owning client's name shown in the header). Client -- `client_name`, read only for header context.
**Updates:** Project -- `currency`, `tax_label`, `tax_rate`, only while the project is unlocked (FEAT-15.SPEC-004) and only with values that pass FEAT-15.SPEC-003.
**Deletes:** None.

## Business Rules

- Currency and tax field validation (FEAT-15.SPEC-003) is enforced on every save -- Nadia cannot save an unrecognized currency or an invalid tax shape.
- The lock rule (FEAT-15.SPEC-004) governs whether this screen renders editable or locked; once the project's first invoice is sent, this screen never shows an editable field for that project again.
- Access to this screen is governed entirely by FEAT-15.SPEC-005 -- Owen and Priya never reach it, and Dana reaches it only read-only inside a logged support session.
- XBR-17: this screen is the sole place a project's currency and tax line are set, and every downstream invoice (FEAT-09) and proposal (FEAT-02) for this project uses the value saved here.
- An invoice cannot be generated for a project with no currency/tax configured yet -- the empty-configuration banner on this screen is the correction path, enforced by FEAT-15.SPEC-003.
- Two concurrent unlocked edits to the same project's currency/tax resolve last-write-wins, consistent with the dependency map's Project Contention note treating currency/tax as a non-lifecycle field (like renames); this is superseded by the lock the moment the project's first invoice is sent, which always wins over a stale unlocked save.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved currency and tax changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **The project's first invoice is sent by an automation (FEAT-09) while Nadia has this screen open, unlocked** -- The screen is rejected-with-refresh: any in-progress edit is not saved, a message appears ("This project's first invoice was just sent -- currency and tax are now fixed."), and the screen re-renders in its Locked state. Resolution: reject-with-refresh, since a locked configuration can never be overwritten by a stale unlocked edit.
- **Nadia attempts to change currency/tax on a project whose first invoice has already been sent** -- The screen renders no editable fields at all for a locked project, so this is structurally prevented; a direct, out-of-band change attempt (e.g., a replayed request) is refused with the same "fixed after your first invoice" explanation.
- **Nadia toggles "No tax line" on after entering a tax label and rate** -- The entered tax label and rate are discarded from the form; if she toggles it off again, both fields are empty and must be re-entered.
- **Dana opens this screen inside a support session for a project that is not yet locked** -- She sees the current (possibly unconfigured) values read-only; if the empty-configuration state applies, she sees the same "not yet configured" banner but with no ability to act on it.
- **Nadia has this screen open in two sessions at once, and both save a currency/tax change to the same unlocked project before either reloads** -- Resolution: last-write-wins, consistent with the dependency map's Project Contention note (currency/tax are non-lifecycle Project fields, resolved the same way as renames -- the reject-with-refresh treatment there is reserved for the complete/cancel/archive lifecycle transitions only). Both saves succeed individually; the later save's values are what persists on the Project record, and the earlier session's form reflects the newer values the next time it loads or re-saves (it is not live-pushed mid-edit). If the project's first invoice is sent between the two saves, the lock (FEAT-15.SPEC-004) takes precedence: the second save is rejected-with-refresh into the Locked state per the "first invoice sent while open" edge case above, rather than being applied as a last-write-wins update to a now-locked project.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-003 (Currency & Tax Validation Rules) | References (inbound) | Governs every field-level validation rule this screen enforces on save |
| FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice) | References (inbound) | Governs whether this screen renders editable or locked |
| FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules) | References (inbound) | Governs the Access and Visibility table |
| FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application) | References (outbound) | The values saved here are the ones this automation stamps onto every generated invoice |
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Sole entry point into this screen, from project billing setup |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support-session entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| currency_set | project reference, currency code | Nadia's save succeeds with a currency value (first save or a change while unlocked) | supports success-metrics.md: "Invoice Currency and Tax Correctness" |
| tax_treatment_set | project reference, tax treatment (label + rate, or none) | Nadia's save succeeds with a tax treatment | supports success-metrics.md: "Invoice Currency and Tax Correctness" |
| currency_tax_config_blocked_invoice | project reference | Nadia or the system reaches invoice generation for this project while the empty-configuration banner is showing | N/A -- no success-metrics.md metric measures blocked-generation attempts directly; retained since product-features.md's States field for FEAT-15 names this as the Empty-state behavior this screen must surface |

## Acceptance Criteria

**FEAT-15.SPEC-001-AC-01:** Given Nadia opens billing setup for a project with no currency/tax ever saved, when the screen loads, then she sees the empty-configuration banner above an empty form.

**FEAT-15.SPEC-001-AC-02:** Given Nadia selects a recognized currency and enters a tax label "VAT" and rate "20", when she taps Save, then the configuration is validated via FEAT-15.SPEC-003, saved, and she sees a "Currency and tax saved" toast.

**FEAT-15.SPEC-001-AC-03:** Given Nadia toggles "No tax line" on, when she taps Save with only a currency selected, then the project saves with no tax label or rate and no error is shown.

**FEAT-15.SPEC-001-AC-04:** Given Nadia enters an unrecognized currency, when she taps Save, then the currency field shows the exact error message defined by FEAT-15.SPEC-003 and the save does not proceed.

**FEAT-15.SPEC-001-AC-05:** Given Nadia is viewing a project whose first invoice has already been sent, when the screen loads, then currency and tax render as a plain, non-editable display with the explanation "currency and tax are fixed after your first invoice," and no Save button is shown.

**FEAT-15.SPEC-001-AC-06:** Given Owen attempts to reach this screen directly, when the link resolves, then he is redirected to his portal home with no currency/tax content ever rendered.

**FEAT-15.SPEC-001-AC-07:** Given Priya attempts to reach this screen directly, when the link resolves, then she is redirected to her portal home with no currency/tax content ever rendered.

**FEAT-15.SPEC-001-AC-08:** Given Dana opens this screen inside a support session, when the screen loads, then she sees the full current configuration read-only, with no Save control rendered.

**FEAT-15.SPEC-001-AC-09:** Given Nadia has unsaved changes on this screen, when she taps the back control, then a confirmation dialog appears asking "You have unsaved currency and tax changes. Discard?"

**FEAT-15.SPEC-001-AC-10:** Given Nadia has this screen open and unlocked, when the project's first invoice is sent by FEAT-09 while she is mid-edit, then her edit is not saved, a message explains the lock just took effect, and the screen re-renders Locked.

**FEAT-15.SPEC-001-AC-11:** Given Nadia loses connectivity while filling the form, when she looks at the Save button, then it is disabled and the banner "You're offline -- currency and tax changes can't be saved right now." is shown.

**FEAT-15.SPEC-001-AC-12:** Given Nadia taps Save and the save operation fails for a reason other than validation (e.g., connectivity), when the failure occurs, then she sees "Couldn't save your currency and tax settings. Check your connection and try again." with a Retry action, and the form fields retain her entered values. This screen has no separate load-failure state: per product-features.md's States field for FEAT-15, loading is N/A because currency/tax are fields on the already-fetched Project record.

**FEAT-15.SPEC-001-AC-13:** Given Nadia toggles "No tax line" on after entering a tax label and rate, when she toggles it off again, then both fields are shown empty rather than restoring the discarded values.

**FEAT-15.SPEC-001-AC-14:** Given a project has never had currency/tax configured, when Nadia or an automation reaches invoice generation for it, then generation is blocked and this screen's empty-configuration banner is the correction path.

**FEAT-15.SPEC-001-AC-15:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-15.SPEC-001-AC-16:** Given Nadia successfully saves a valid configuration, when the save completes, then the currency_set and tax_treatment_set events are emitted with the project reference and the values saved.

**FEAT-15.SPEC-001-AC-17:** Given Nadia has this screen open in two sessions on the same unlocked project, when she saves a currency/tax change in each session before either reloads, then both saves succeed and the later save's values persist on the Project record (last-write-wins), unless the project's first invoice was sent between the two saves, in which case the second save is rejected-with-refresh into the Locked state instead.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 9 (loading (N/A), empty, filling, saving, validation error, configured, locked, error, offline) | 9 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |

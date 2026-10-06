---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-004
spec_name: Business Details & Payment Terms
spec_slug: business-details-payment-terms
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Business Details & Payment Terms

## Overview

**Name:** Business Details & Payment Terms
**ID:** FEAT-21.SPEC-004
**Type:** Screen
**Purpose:** Nadia sets her business name, address, tax ID, and default payment terms that print on every invoice; Dana views the same values read-only inside a logged support session.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying and editing business name, business address, tax ID, and default payment terms
- Showing whether business details are complete enough to send a first invoice (per FEAT-21.SPEC-009)
- Saving edits immediately with retry-preserving error handling
- Read-only rendering of the identical fields inside a logged Dana support session (FEAT-31)

**Non-Goals:**
- Deciding whether an invoice send is blocked -- owned by FEAT-21.SPEC-009 (Business Details Completeness Gate), which FEAT-09 (Invoice Generation & Sending) checks at send time; this screen only displays completeness status and lets Nadia fill the gap
- Currency and tax-rate configuration -- owned by Currency & Tax Handling (FEAT-15) at the project level; the tax_id field here is the freelancer's own tax identifier printed on invoices, a distinct field from the project's tax label/rate
- Per-project payment schedule or milestone pricing -- owned by Milestone & Payment Schedule Setup (FEAT-04); "default payment terms" here is the account-wide due-date default, not a project-specific schedule
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Business Details & Payment Terms" from the Settings navigation shell | None |
| FEAT-21.SPEC-009 (Business Details Completeness Gate) | Nadia is prompted to complete business details after a blocked first-invoice send attempt (FEAT-09) | A "complete your business details" prompt context, highlighting the incomplete fields |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Business Details & Payment Terms | Read-only support session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Edit business name, address, tax ID, default payment terms | -- |
| Dana (Support Operator) | Full screen, read-only, inside a logged FEAT-31 support session | None -- no save controls (FEAT-21.SPEC-010) | Editing controls are not rendered at all; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- unsaved field edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Business Details & Payment Terms" selected. When entered from FEAT-21.SPEC-009's incomplete-details prompt, a banner appears at the top: "Complete your business details before sending your first invoice."

**Body:** A single-column form with the following fields in order:
- Business name (text input, required before first invoice)
- Business address (multi-line text input, required before first invoice)
- Tax ID (text input, optional)
- Default payment terms (selection input: "Due on receipt" or "Net {N} days", required before first invoice)

Below the form, a completeness indicator line reading either "Business details are complete." or "Business details are incomplete -- required before your first invoice can be sent." (per FEAT-21.SPEC-009), followed by a "Save" action button.

**Footer:** None -- Save is directly below the form.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width, stacked in the order listed; the completeness indicator and Save button remain below the form.
- **Medium size class and above:** Form remains single-column, capped at the platform-wide form width alongside the navigation shell; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Business name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Business name input | Blur (empty) | Triggers the required-before-invoicing check via FEAT-21.SPEC-007 (advisory, non-blocking) | Advisory notice on field; Save stays enabled | "Business name is required before invoicing" below field, per FEAT-21.SPEC-007 |
| Business address input | Type / Blur | Captures text input; validates via FEAT-21.SPEC-007 (empty is advisory, over-length is blocking) | Field shows entered text, advisory notice (empty), or error state (over 500 characters) | Standard input state, "Business address is required before invoicing" (empty), or "Business address must be 500 characters or fewer" |
| Tax ID input | Type | Captures text input | Field shows entered text | Standard input focus state -- no required-field error, since tax ID is optional |
| Default payment terms selector | Select | Captures the chosen option; validates via FEAT-21.SPEC-007 | Selector shows chosen value | Standard selection feedback |
| Save button | Tap | Validate all fields via FEAT-21.SPEC-007; only blocking rules (length limits) stop the save. Empty required-before-invoicing fields do not block: whatever is filled is saved (partial saves allowed), then completeness is re-checked via FEAT-21.SPEC-009 | Button shows loading state during save | Success: toast "Business details updated" and the completeness indicator updates (complete or incomplete). Blocking validation failure: error state on the offending field, save does not proceed. Server failure: inline error banner, retry option, entered fields preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Navigation shell -> Business name -> Business address -> Tax ID -> Default payment terms selector -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Completeness announcements:** A change in the completeness indicator's text (from incomplete to complete, or the reverse) is announced on save.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with current values (empty for any field never set), Save button enabled | Screen opens | User begins editing |
| Editing | Fields show in-progress input, Save button enabled | User types or selects in any field | User taps Save or navigates away |
| Saving | Save button shows a loading spinner, fields disabled | User taps Save with valid input | Save completes or fails |
| Validation Error | Failed fields highlighted with error messages below them; Save does not proceed | A field exceeds its length limit (business name 200, address 500, tax ID 50) on blur or submit | User corrects the field and re-triggers validation |
| Incomplete notice | Empty required-before-invoicing fields show their advisory message below the field; Save remains enabled | A required-before-invoicing field (business name, address, default payment terms) is empty on blur or after save | User fills the field |
| Error | Error banner at top of form: "Could not save your business details. Check your connection and try again." with a Retry button | Save operation fails server-side | User taps Retry; entered fields are preserved and resubmitted |
| Read-only (Dana) | Form shows current values with no Save button | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); the form remains visible with current values but Save is disabled and a banner reads "You're offline. Reconnect to save changes." | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; no queued submission occurs |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules). See that spec for the required-before-invoicing rules on business name, business address, and default payment terms (advisory and non-blocking: they never stop a save), for the blocking length limits, and for tax ID's optional length rule. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |
| Successful save | Stays on FEAT-21.SPEC-004 with the confirmation toast and updated completeness indicator | -- |

## Data Model

**Creates:** None.
**Reads:** Freelancer Account -- `business_name`, `business_address`, `tax_id`, `default_payment_terms`.
**Updates:** Freelancer Account -- `business_name`, `business_address`, `tax_id`, `default_payment_terms`.
**Deletes:** None.

## Business Rules

- Blocking field validation (FEAT-21.SPEC-007 length limits) is enforced before any save completes. Required-before-invoicing fields are non-blocking: Nadia can save with any of business name, address, or default payment terms empty (including a form where only tax ID is filled); the save succeeds and completeness stays incomplete until all three are filled (FEAT-21.SPEC-009).
- Completeness (FEAT-21.SPEC-009) is re-evaluated on every successful save and reflected in the indicator line; the gate itself is enforced at invoice-send time by FEAT-09, not by this screen (XBR-16).
- Access and read-only scope (FEAT-21.SPEC-010) governs what Dana sees and cannot act on.
- Two open sessions of Nadia's editing these fields resolve last-write-wins per field, per the dependency map's Contention note for the Freelancer Account entity.

## Edge Cases

- **Nadia navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails server-side** -- Error banner: "Could not save your business details. Check your connection and try again." with a Retry button; entered fields are preserved and retried without being discarded (product-features.md, States field).
- **Business details changed by Nadia in a second open session while this screen is open** -- Save is last-write-wins per the dependency map's Contention note for the Freelancer Account entity.
- **Nadia completes the last missing required field and saves while a first-invoice send from another screen is in progress against the previously incomplete state** -- FEAT-21.SPEC-009's completeness check runs at the moment FEAT-09 attempts the send, not at the moment this screen loaded, so a send that starts after this save's completeness update proceeds; a send already in flight when this save commits uses the state it read at its own check point (FEAT-21.SPEC-009's own Edge Cases govern the exact race).
- **Nadia clears a previously filled required field, leaving business details incomplete again** -- The save is not blocked; it succeeds with the toast "Business details updated", the completeness indicator switches back to "incomplete", and any future first-invoice send is blocked again per FEAT-21.SPEC-009.
- **Nadia fills only some fields (e.g., only tax ID) and saves** -- The partial values are saved, the toast "Business details updated" appears, the empty required fields keep their advisory messages, and the indicator reads "Business details are incomplete -- required before your first invoice can be sent."

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | Field-level validation for all fields on this screen |
| FEAT-21.SPEC-009 (Business Details Completeness Gate) | Triggers (outbound) | Re-checks completeness on every save; navigates here when a blocked invoice send prompts Nadia to finish |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | Business details and default payment terms print on every invoice; sending is blocked until they exist (XBR-16) |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| business_details_updated | fields_changed (list), completeness_state: complete/incomplete | Nadia's business details save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("business_details_updated") so the save is observable |
| settings_save_failed | section: business_details | Save fails server-side | N/A -- no connected success-metrics.md metric; retained to make retry-preserving save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-004-AC-01:** Given Nadia is on the Business Details & Payment Terms screen with business name empty, when she taps Save, then the business name field shows the advisory "Business name is required before invoicing", the save still proceeds for the other fields with the toast "Business details updated", and the completeness indicator reads "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-02:** Given Nadia fills in business name, business address, and default payment terms, when she taps Save, then the details save, a "Business details updated" toast appears, and the completeness indicator reads "Business details are complete."

**FEAT-21.SPEC-004-AC-03:** Given Nadia leaves the tax ID field empty, when she saves with the other required fields complete, then the save succeeds and the completeness indicator reads "Business details are complete." (tax ID is optional).

**FEAT-21.SPEC-004-AC-04:** Given Nadia arrives from FEAT-21.SPEC-009's incomplete-details prompt, when the screen loads, then the banner "Complete your business details before sending your first invoice." appears above the form.

**FEAT-21.SPEC-004-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the fields, then she sees the current values with no Save button.

**FEAT-21.SPEC-004-AC-06:** Given Dana (Support Operator) is viewing business details read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-004-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-004-AC-08:** Given Nadia's business details save fails server-side, when the failure occurs, then the error banner "Could not save your business details. Check your connection and try again." appears with a Retry button, and her entered fields remain populated.

**FEAT-21.SPEC-004-AC-09:** Given Nadia has an unsaved edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-004-AC-10:** Given Nadia clears a previously filled business address, leaving it empty, when she saves, then the save succeeds with the toast "Business details updated" and the completeness indicator switches to "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-11:** Given Nadia loses connectivity on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-004-AC-12:** Given Nadia's expired session is detected while she has unsaved field edits, when she signs in again, then her unsaved edits are restored on this screen.

**FEAT-21.SPEC-004-AC-13:** Given Nadia fills only the tax ID and leaves business name, business address, and default payment terms empty, when she taps Save, then the tax ID is saved, the toast "Business details updated" appears, each empty required field shows its advisory message, and the indicator reads "Business details are incomplete -- required before your first invoice can be sent."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loaded, editing, saving, validation error, incomplete notice, error, read-only (Dana), offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

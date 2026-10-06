---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-001
spec_name: Proposal Draft Editor
spec_slug: proposal-draft-editor
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Proposal Draft Editor

## Overview

**Name:** Proposal Draft Editor
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** Nadia writes scope, price, and currency for a project's proposal from a blank form, from a copied earlier proposal, or by revising a Sent-but-unaccepted proposal.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- Blank-draft creation for a project with no existing proposal
- Pre-filled draft creation when opened from a copy (FEAT-02.SPEC-008)
- Edit mode for a Sent-but-unaccepted proposal, culminating in a void-and-resend rather than a silent update
- Inline field validation via FEAT-02.SPEC-010
- Local draft persistence and offline-tolerant drafting
- Navigating to the branded preview before sending
- Discarding an unsent draft

**Non-Goals:**
- Sending the proposal -- owned by FEAT-02.SPEC-005 (Proposal Send); this screen only prepares content and hands off to Send from the Preview screen.
- Browsing earlier proposals to copy from -- owned by FEAT-02.SPEC-004 (Reuse Proposal Picker); this screen only receives already-selected copied content from FEAT-02.SPEC-008.
- A configurable proposal template or workflow builder -- excluded per scope-boundaries.md (SC-11): the category's 15-25+ hour setup burden comes from configurable builders, so this product ships one fixed form instead.
- A library of legal contract clauses -- excluded per scope-boundaries.md (SC-13): freelancers write their own scope text or reuse an earlier proposal (FEAT-02.SPEC-004/008) rather than draw on a maintained legal template library.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-003 (Proposal Detail) | Nadia taps "Edit" on a Draft, or on a Sent-but-unaccepted proposal | Existing proposal id, current scope/price/currency, mode flag (Draft edit vs. Sent-but-unaccepted edit) |
| FEAT-02.SPEC-003 (Proposal Detail) | Empty-state prompt when the project has no proposal yet | Project reference; blank form; project currency (FEAT-15) |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Automation finishes copying an earlier proposal and opens the editor | New Draft's scope/price/currency pre-filled from the source; copied_from reference |
| FEAT-01 (Client & Project Management), project view proposal area | Nadia opens the proposal area of a project with no existing proposal | Project reference; blank form |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Draft the first proposal" | New project reference; blank form |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Create, edit, save draft, preview, save & resend, discard -- all actions | -- |
| Owen (Client Primary Contact) | No | No | This screen exists only in the freelancer's own workspace; no route into it exists from the client portal, so Owen never sees it. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal route to this screen exists. |
| Dana (Support Operator) | Full screen, read-only (all field values visible) | None -- no Save, Preview, Save & Resend, or Discard controls appear | Reached only inside a logged, read-only support session (FEAT-31); every input renders disabled, so there is nothing to attempt beyond viewing. |
| Unauthenticated | No | No | Redirected to freelancer sign-in; no proposal content renders before authentication succeeds. |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved field values are preserved locally and restored automatically once re-authentication succeeds. |

## Layout and Content

**Header:** Screen title -- "New Proposal" (blank or copied-draft entry) or "Edit Proposal" (Sent-but-unaccepted edit entry) -- with a back arrow (returns to FEAT-02.SPEC-003) at the left. In Draft mode, a secondary "Save Draft" text action sits at the top-right.

**Body:** A single-column form:
- **Copied-from banner** (shown only when the editor opened from FEAT-02.SPEC-008): "Started from a copy of {source project name}'s proposal, {source proposal's sent or accepted date}." Display-only.
- **Scope Description** -- multi-line text input, required
- **Price** -- number input, required, positive amount
- **Currency** -- read-only display field showing the project's currency (FEAT-15); never editable on this screen, since XBR-17 fixes it once the first invoice is sent and it is set for the project elsewhere
- **Payment Schedule** -- a read-only summary card showing the project's Payment Schedule (FEAT-04) structure if one exists, or the text "No payment schedule set yet" with a link into FEAT-04 if none exists

**Footer:** No footer region. A bottom action bar carries the primary actions: in Draft mode, "Discard Draft" (destructive text button, left) and "Preview" (primary button, right); in Sent-but-unaccepted edit mode, "Save & Resend" (primary button, right) replaces "Preview" and "Discard Draft" is not shown (discard applies only to unsent drafts, per FEAT-02.SPEC-010).

### Responsive Behavior

- **Compact:** Single-column form, full width; header shows only the title and back arrow, with "Save Draft" moving into an overflow menu if space is constrained.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the bottom action bar remains fixed and visible without a structural change. Scope Description grows from 4 visible lines (compact) to 8 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-003 (Proposal Detail); if unsaved changes exist, confirm first | Dialog if unsaved changes exist | "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" |
| Scope Description input | Type | Captures text | Field shows entered text | Standard input focus state |
| Scope Description input | Blur (empty) | Triggers validation via FEAT-02.SPEC-010 | Error state on field | "Scope description is required" |
| Price input | Type | Captures numeric value | Field shows entered value | Standard input focus state |
| Price input | Blur (empty or non-positive) | Triggers validation via FEAT-02.SPEC-010 | Error state on field | Exact message per FEAT-02.SPEC-010 |
| Payment Schedule card | Tap | Navigate to FEAT-04 (Milestone & Payment Schedule Setup) | Screen transitions | -- |
| Save Draft (Draft mode only) | Tap | Validates required-field rules via FEAT-02.SPEC-010; persists content as Draft | Button shows brief saved confirmation | Toast "Draft saved" |
| Preview (Draft mode only) | Tap | Validates via FEAT-02.SPEC-010; if valid, saves current content as Draft and navigates to FEAT-02.SPEC-002 (Proposal Preview); if invalid, blocks navigation | Button shows loading state briefly | Success: navigates to Preview. Failure: inline field errors shown, focus moves to first error |
| Save & Resend (edit mode only) | Tap | Validates via FEAT-02.SPEC-010; if valid, triggers FEAT-02.SPEC-006 (Void & Resend) | Button shows loading state during the operation | Success: navigates to FEAT-02.SPEC-003 showing the updated Sent state. Failure: error banner, edits preserved |
| Discard Draft (Draft mode only) | Tap | Confirmation dialog, then triggers FEAT-02.SPEC-009 (Discard Draft) | Confirmation dialog appears | Dialog: "Discard this draft? This cannot be undone." with "Discard" and "Keep Editing" |

### Accessibility Notes

- **Focus order:** Back arrow -> Save Draft (if shown) -> Scope Description -> Price -> Currency (read-only, announced but not editable) -> Payment Schedule card -> Discard Draft (if shown) -> Preview/Save & Resend.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field; on Preview/Save & Resend failure, focus moves to the first field in error.
- **Save feedback:** The "Draft saved" toast is announced on success.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All fields empty except the read-only Currency (populated from the project) | Screen opens for a new blank proposal | Nadia begins typing in any field |
| Prefilled | Scope Description, Price, and Currency populated from a copy or from the existing Draft/Sent-but-unaccepted proposal | Screen opens from FEAT-02.SPEC-008 or in edit mode | Nadia edits a field |
| Filling | Form fields contain user input | Nadia types in any field | Nadia navigates away or completes an action |
| Validating | Preview/Save Draft/Save & Resend button shows a brief loading indicator | Nadia taps Preview, Save Draft, or Save & Resend | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Nadia corrects the field and re-triggers validation |
| Saving | Active button shows a loading spinner, form fields disabled | Validation passes | Save (or Save & Resend) completes or fails |
| Error | Error banner at the top of the form with a retry option | Save, Preview-save, or Save & Resend fails | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner: "You're offline -- your changes are kept on this device until you reconnect." Drafting continues locally; Preview, Save Draft, and Save & Resend are disabled until connectivity returns (sending requires connectivity per the feature definition) | Connectivity is lost while the screen is open | Connectivity returns; disabled actions re-enable |

## Validation Rules

Validation governed by FEAT-02.SPEC-010 (Proposal Validation & Business Rules). See that spec for all field-level rules (required scope description, positive price) and for the immutability rule that blocks a Save & Resend attempt on an already-accepted proposal. This screen applies validation on field blur and on Preview/Save Draft/Save & Resend.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Preview tap (validation passes) | FEAT-02.SPEC-002 (Proposal Preview) | -- |
| Save & Resend success | FEAT-02.SPEC-003 (Proposal Detail) | -- |
| Discard Draft confirmed | FEAT-02.SPEC-003 (Proposal Detail), empty state | -- |
| Payment Schedule card tap | Milestone & Payment Schedule Setup | FEAT-04 |

## Data Model

**Creates:** Proposal record (Draft) -- scope_description, price, currency (read from the owning Project), payment_schedule_reference (read from the Project's Payment Schedule), status set to Draft, copied_from set when opened from FEAT-02.SPEC-008.
**Reads:** Project -- currency and project_name (FEAT-15). Payment Schedule -- structure summary (FEAT-04). The existing Proposal record's scope_description, price, and status when opened in edit mode.
**Updates:** Proposal record's scope_description and price when Save Draft is used on an existing Draft; a Sent-but-unaccepted proposal's content is never updated directly here -- Save & Resend hands the new content to FEAT-02.SPEC-006, which creates the new version.
**Deletes:** None directly -- the Discard Draft action hands off to FEAT-02.SPEC-009, which performs the deletion.

## Business Rules

- Field requirements and positive-price validation are governed by FEAT-02.SPEC-010 -- this screen never saves a Draft or triggers a send with an invalid scope description or price.
- Currency is never editable on this screen; it always reflects the owning project's currency and is fixed once the project's first invoice is sent (XBR-17).
- Editing a Sent-but-unaccepted proposal never updates the proposal in place -- Save & Resend triggers FEAT-02.SPEC-006, which voids the prior version and creates and sends the new one (XBR-06).
- Discard is available only on unsent Drafts, per FEAT-02.SPEC-010 -- the action is not shown once a proposal has been sent.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Preview or Save & Resend twice rapidly** -- The second tap is ignored while the first operation is in progress (button in loading state).
- **Network failure during Save Draft** -- Error banner: "Could not save your draft. Check your connection and try again." with a Retry button; form content is preserved.
- **Nadia edits a Sent-but-unaccepted proposal in two open sessions and saves from both** -- Reject-with-refresh, per the dependency map's Contention note for the Proposal entity: the second Save & Resend is checked against the proposal's current version at commit; if the first session's edit already voided and re-sent it, the second save is refused with "This proposal was already edited and resent. Review the current version." and the screen reloads the now-current version's content.
- **Nadia attempts to edit a proposal that Owen accepted while the editor was open** -- Save & Resend is refused per XBR-04 with the dialog "This proposal has already been accepted and can no longer be edited." and a "View Current Status" option that navigates to FEAT-02.SPEC-003.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-002 (Proposal Preview) | Navigation (outbound) | Preview tap, after validation, opens the branded preview |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (inbound/outbound) | Entry point for Edit and the empty-state prompt; return destination after save, resend, or discard |
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | References (inbound, indirect) | Reached only through FEAT-02.SPEC-008 after a source proposal is selected |
| FEAT-02.SPEC-006 (Void & Resend) | Triggers (outbound) | Save & Resend, after validation, triggers the void-and-resend automation |
| FEAT-02.SPEC-008 (Create Draft From Copy) | Navigation (inbound) | Opens this screen pre-filled once the copy automation completes |
| FEAT-02.SPEC-009 (Discard Draft) | Triggers (outbound) | Discard Draft, after confirmation, triggers the hard-delete automation |
| FEAT-02.SPEC-010 (Proposal Validation & Business Rules) | References (inbound) | All field validation and the edit-eligibility rule applied on this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| proposal_drafted | entry mode (blank / from copy), time from screen open to first save (seconds) | Save Draft completes successfully on a new Draft | supports success-metrics.md: "Proposal Send Speed" |
| proposal_draft_save_failed | failure reason (validation / connectivity) | Save Draft or Preview-triggered save fails | N/A -- no Stage 2 metric measures failed saves; retained so drafting friction is observable rather than invisible |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Nadia opens the proposal area of a project with no existing proposal, when the screen loads, then the form shows empty Scope Description and Price fields with Currency pre-filled from the project.

**FEAT-02.SPEC-001-AC-02:** Given Nadia fills in a scope description and a positive price and taps "Save Draft", then the system saves the Proposal as Draft and shows the toast "Draft saved".

**FEAT-02.SPEC-001-AC-03:** Given Nadia leaves the Scope Description field empty and moves focus away, then the field shows the error "Scope description is required" per FEAT-02.SPEC-010.

**FEAT-02.SPEC-001-AC-04:** Given Nadia has filled in valid scope and price and taps "Preview", then validation passes, the content saves as Draft, and she is navigated to FEAT-02.SPEC-002 (Proposal Preview).

**FEAT-02.SPEC-001-AC-05:** Given Nadia opens the editor from FEAT-02.SPEC-008 after selecting a source proposal, when the screen loads, then Scope Description, Price, and Currency are pre-filled from the source and the copied-from banner shows the source project's name.

**FEAT-02.SPEC-001-AC-06:** Given Nadia opens a Sent-but-unaccepted proposal in edit mode, when she revises the price and taps "Save & Resend", then FEAT-02.SPEC-006 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 showing the updated Sent state.

**FEAT-02.SPEC-001-AC-07:** Given Nadia is on a Draft and taps "Discard Draft", when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and she is navigated to FEAT-02.SPEC-003 showing the empty state.

**FEAT-02.SPEC-001-AC-08:** Given Nadia has unsaved changes on the form, when she taps the back arrow, then the dialog "You have unsaved changes. Discard?" appears with "Discard" and "Keep Editing".

**FEAT-02.SPEC-001-AC-09:** Given Nadia loses connectivity while filling the form, then the banner "You're offline -- your changes are kept on this device until you reconnect." appears and Preview, Save Draft, and Save & Resend are disabled until connectivity returns.

**FEAT-02.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views the form, then all field values render read-only with no Save, Preview, Save & Resend, or Discard controls.

**FEAT-02.SPEC-001-AC-11:** Given Nadia edited and saved-and-resent a Sent-but-unaccepted proposal from a second open session while this session's edit was also pending, when this session's "Save & Resend" completes its check, then the save is refused with "This proposal was already edited and resent. Review the current version." and the screen reloads the current version's content.

**FEAT-02.SPEC-001-AC-12:** Given Owen accepted the proposal while Nadia's edit screen was open, when Nadia taps "Save & Resend", then the save is refused with the dialog "This proposal has already been accepted and can no longer be edited." and a "View Current Status" option to FEAT-02.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (empty, prefilled, filling, validating, validation error, saving, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |

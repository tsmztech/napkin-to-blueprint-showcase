---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-002
spec_name: Cancellation Policy Default & First-Version Setup Step
spec_slug: cancellation-policy-default-first-version-setup-step
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Cancellation Policy Default & First-Version Setup Step

## Overview

**Name:** Cancellation Policy Default & First-Version Setup Step
**ID:** FEAT-15.SPEC-002
**Type:** Screen
**Purpose:** Talia accepts or adjusts a recommended cancellation window and saves it, creating the first Cancellation Policy version for her account.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Presenting the recommended default cancellation window and plain-language wording
- Letting Talia accept the default or adjust the window within the product's allowed range
- Saving the first Cancellation Policy version (version 1) by applying FEAT-09.SPEC-001's validation and FEAT-09.SPEC-002's versioning rules
- Reporting step completion back to the wizard shell (FEAT-15.SPEC-001) once saved

**Non-Goals:**
- Editing the policy after this first version is saved -- every subsequent edit creates a new version and is owned entirely by FEAT-09 (FEAT-09.SPEC-002); this screen's involvement ends once version 1 is saved
- Defining the field-level validation and versioning rules themselves -- owned by FEAT-09.SPEC-001 (Cancellation Policy Setup) and FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this screen applies those rules rather than restating them
- Partial refunds or tiered cancellation schedules -- excluded per scope-boundaries.md SC-18: the product's cancellation rule is binary (full refund outside the window, deposit kept inside it or on a no-show), so no tiered percentage option is offered here
- Multi-language policy wording -- excluded per scope-boundaries.md SC-10: the near-term geography is entirely English-language, so no translated wording variant is built

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Talia taps "Continue" on the cancellation-policy step, or taps this step in the header list after already completing it | If completing for the first time: none, the form starts with the recommended default pre-filled. If revisiting a completed step: N/A -- this screen has no edit path after version 1 is saved (see Non-Goals); tapping the completed step instead shows the saved version 1 wording read-only with a note that changes are made from Cancellation & No-Show settings later (FEAT-09) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Accept or adjust the recommended window and save version 1 | -- |
| The Client (Riley) | No | No | Never reaches this setup screen; a client sees only the resulting plain-language policy on the public booking page (FEAT-05) once it exists |
| Platform Operator (Support) | No | No | Support has View-only access to the Cancellation & No-Show Handling capability group generally, but this specific setup-step screen is never opened by Support; Support views the saved policy through FEAT-09's own read surfaces if needed for a help request |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); the wizard shell re-enters at this step on return if it was the resume point |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any adjustment Talia made to the window before saving is preserved locally and restored after re-authentication succeeds, since nothing is persisted until Save |

## Layout and Content

**Header:** Step title "Cancellation Policy" with the wizard shell's persistent step-progress indicator above it (owned by FEAT-15.SPEC-001).

**Body:**
- A short explanation in plain language: what the cancellation window controls (when a client can still get their deposit back) and why it protects Talia from last-minute cancellations
- A recommended-window display: "We recommend platform parameter: `cancellation-window-default-hours` hours before the appointment" shown as the pre-filled value in an editable numeric field labeled "Cancellation window (hours before appointment)"
- A live preview of the exact plain-language wording a client will see at booking, updating as Talia adjusts the number (for example: "Cancel or reschedule at least {window} hours before your appointment for a full refund. Cancelling later, or not showing up, means your deposit is kept.")
- A "Save and continue" button

**Footer:** None -- Save is in the body, directly below the preview.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described; the live wording preview sits directly beneath the numeric field, both full width.
- **Medium size class and above:** The numeric field and its live preview render side by side in two columns instead of stacked; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Cancellation window field | Type or use stepper controls | Captures the numeric hour value | Live wording preview updates to reflect the new value | Preview text updates in place as the field changes |
| Cancellation window field | Blur with an out-of-range value | Triggers validation via FEAT-09.SPEC-001 | Field shows error state | Exact error message from FEAT-09.SPEC-001 (window must be a whole number of hours from 1 to 168) |
| "Save and continue" button | Tap | 1. Validate the window via FEAT-09.SPEC-001. 2. If valid, create Cancellation Policy version 1 via FEAT-09.SPEC-002's versioning rules (version = 1, effective_from = now). 3. Report step completion to FEAT-15.SPEC-004. | Button shows a brief saving state | Success: step marked complete, wizard shell (FEAT-15.SPEC-001) advances to the next step. Failure: inline error, entered value preserved |
| "Save and continue" (while saving) | Tap | No action -- debounced | None | Button remains in its saving state |

### Accessibility Notes

- **Focus order:** Explanation text -> cancellation window field -> live wording preview (read-only, announced as a live region) -> "Save and continue" button.
- **Dynamic-change announcements:** When the field's value changes, the updated wording preview is announced to assistive technology as a polite live-region update, not an interruption. A validation error is announced immediately and associated with the field.
- **Keyboard alternatives:** The numeric field's stepper controls (increment/decrement) are reachable and operable by keyboard (arrow keys), with direct typing as an equivalent alternative.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (recommended) | Field pre-filled with platform parameter: `cancellation-window-default-hours`; preview shows the resulting wording; Save enabled | Screen first opens | Talia edits the field, or taps Save with the default unchanged |
| Adjusted | Field shows Talia's entered value; preview reflects it; Save enabled if valid | Talia types a new value | Talia saves, or clears the field back toward a valid state |
| Validation Error | Field shows error state and message from FEAT-09.SPEC-001 | Entered value fails validation (not 1-168 whole hours) | Talia corrects the value |
| Saving | Save button shows a loading indicator; field disabled | Talia taps Save with a valid value | Save completes or fails |
| Error | Error banner "Could not save your cancellation policy. Check your connection and try again." with a Retry action; entered value preserved | Save operation fails | Talia taps Retry and the save succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not an in-the-moment mobile flow (feature's own States field); saving requires connectivity | -- | -- |

## Validation Rules

Validation governed by FEAT-09.SPEC-001 (Cancellation Policy Setup). See that spec for the exact window bounds (1-168 whole hours) and error messaging. This screen checks the field on blur and again on Save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Successful save | FEAT-15.SPEC-001 (Setup Wizard Shell) advances to the next step (payout account) | -- |

## Data Model

**Creates:** Cancellation Policy record -- version 1, with window_hours (the value Talia accepted or adjusted), inside_window_outcome (deposit kept, fixed at binary v1), outside_window_outcome (full refund, fixed), plain_language_wording (rendered from the template shown in the live preview), effective_from (the moment of save). All field names and the entity's versioning behavior match the Cancellation Policy entity in the Feature Dependency Map exactly, applying FEAT-09.SPEC-002's versioning rules.
**Reads:** The recommended default window value (platform parameter: `cancellation-window-default-hours`), pre-filled on load.
**Updates:** None -- this screen's involvement ends at the creation of version 1; every later edit is owned by FEAT-09.SPEC-002.
**Deletes:** None.

## Business Rules

- The window must be a whole number of hours from 1 to 168 (7 days), per FEAT-09.SPEC-001.
- The policy created here is always version 1 with effective_from set to the moment of save, per FEAT-09.SPEC-002's versioning rule -- no earlier version can exist for a new Pro Account.
- The deposit outcome is binary in v1 (full refund outside the window, deposit kept inside it or on a no-show) per BRIEF.md's Business Context; this screen offers no tiered or partial-refund option.
- This is the one Connected Entity FEAT-15 creates outright rather than merely handing off to another feature's screen, per the Brief's Shared Context.

## Edge Cases

- **Talia saves the recommended default without changing it** -- Version 1 is created with window_hours equal to platform parameter: `cancellation-window-default-hours`, identical in every respect to a version she had explicitly typed that same number for.
- **Talia enters 0 hours** -- Rejected by FEAT-09.SPEC-001's validation (minimum 1 hour); the field shows the exact error message from that spec and Save does not proceed.
- **Talia enters 200 hours** -- Rejected by FEAT-09.SPEC-001's validation (maximum 168 hours); same treatment as above.
- **Talia saves, then immediately re-enters this step from the wizard shell's header list** -- Because this screen has no edit path after version 1 is saved (see Non-Goals), she instead sees the saved wording read-only with a note directing further changes to Cancellation & No-Show settings (FEAT-09), never a second "create version 1" form.
- **Network failure during save** -- Error banner appears with a Retry action; the entered window value is preserved exactly as typed, so Talia never has to re-enter it.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in its saving state); only one Cancellation Policy version 1 is ever created.
- **Concurrent-edit conflict** -- Not applicable: the Cancellation Policy entity is created here, not updated, and this is the only screen in the entire product that can create version 1 for a given Pro Account; no other actor can be mid-edit on a record that does not yet exist. Once version 1 exists, every subsequent edit is exclusively owned by FEAT-09.SPEC-002's own screen, which carries its own concurrency handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Navigation (inbound and outbound) | Launches this screen on Continue; receives control back on successful save |
| FEAT-09.SPEC-001 (Cancellation Policy Setup) | References (outbound) | Field-level validation (1-168 whole hours) applied by this screen |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (outbound) | Versioning rule (version 1, effective_from) applied when this screen saves |
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Triggers (outbound) | Successful save reports this step complete |
| FEAT-05 (Public Booking Page & Booking Flow) | References (outbound, indirect) | The plain-language wording created here is later displayed on the public booking page once the link goes live |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_cancellation_policy_saved | window_hours saved, whether the recommended default was accepted unchanged or adjusted | Save completes successfully | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_cancellation_policy_validation_failed | attempted value, reason (below minimum / above maximum) | Save attempted with an out-of-range value | N/A -- no Stage 2 metric measures setup-form validation failures directly; retained so this step's friction is observable rather than invisible |

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Talia reaches the cancellation-policy step for the first time, when the screen loads, then the window field is pre-filled with platform parameter: `cancellation-window-default-hours` and the wording preview reflects that value.

**FEAT-15.SPEC-002-AC-02:** Given Talia is on this screen with the default value unchanged, when she taps "Save and continue", then a Cancellation Policy version 1 is created with that window and she advances to the payout step.

**FEAT-15.SPEC-002-AC-03:** Given Talia types "48" into the window field, when the field updates, then the live wording preview immediately reflects "48 hours" in its text.

**FEAT-15.SPEC-002-AC-04:** Given Talia enters "0" into the window field and moves focus away, when validation runs (FEAT-09.SPEC-001), then the field shows an error state with that spec's exact minimum-hours error message and Save does not proceed.

**FEAT-15.SPEC-002-AC-05:** Given Talia enters "200" into the window field, when validation runs, then the field shows FEAT-09.SPEC-001's exact maximum-hours error message.

**FEAT-15.SPEC-002-AC-06:** Given Talia adjusts the window to "72" and taps Save, when the save completes, then the created Cancellation Policy version 1 has window_hours equal to 72 and effective_from set to the moment of save.

**FEAT-15.SPEC-002-AC-07:** Given a network failure occurs during save, when the failure is detected, then an error banner reads "Could not save your cancellation policy. Check your connection and try again." with a Retry action, and the entered window value remains in the field.

**FEAT-15.SPEC-002-AC-08:** Given Talia taps "Save and continue" twice in rapid succession, when the second tap registers, then it is ignored while the first save is in progress, and exactly one Cancellation Policy version 1 is created.

**FEAT-15.SPEC-002-AC-09:** Given Talia has already saved version 1 and returns to this step from the wizard shell's completed-step list, when the screen loads, then it shows the saved wording read-only with a note that further changes are made from Cancellation & No-Show settings, and no new version is created.

**FEAT-15.SPEC-002-AC-10:** Given a client (Riley) attempts to reach this screen directly, when the request is made, then it is refused and Riley is shown no wizard content of any kind.

**FEAT-15.SPEC-002-AC-11:** Given an unauthenticated visitor reaches this screen's URL, when the screen would load, then they are redirected to the Pro sign-in screen (FEAT-29) instead.

**FEAT-15.SPEC-002-AC-12:** Given Talia's session expires while she has adjusted the window but not yet saved, when she next interacts with the screen, then the expiry dialog appears, and after she signs back in, her adjusted (unsaved) value is restored in the field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (default, adjusted, validation error, saving, error, offline/degraded) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

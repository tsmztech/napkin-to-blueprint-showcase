---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-001
spec_name: Cancellation Policy Setup
spec_slug: cancellation-policy-setup
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Cancellation Policy Setup

## Overview

**Name:** Cancellation Policy Setup
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** Talia sets or edits her cancellation/reschedule window in whole hours and reviews the resulting plain-language wording before saving, so every client sees exactly what will happen to their deposit.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Capturing and editing the single configurable value of the Cancellation Policy: window_hours (1-168 whole hours)
- Rendering the resulting plain-language wording live as Talia adjusts the window, before she saves
- Saving the edit, which hands off to FEAT-09.SPEC-002 to create a new policy version
- Serving both the first-time setup path (reached from FEAT-15's onboarding) and any later edit, as one shared surface

**Non-Goals:**
- Choosing between a refund or a forfeiture outcome inside vs. outside the window -- excluded per SC-18: the binary outcome (full refund outside, deposit kept inside or on a no-show) is fixed by the product definition and is never a configurable choice on this screen
- Composing or free-typing the plain-language wording -- the wording is always system-derived from window_hours (FEAT-09.SPEC-002); Talia never authors policy text herself, which keeps the wording a client reads at booking and at cancellation always consistent
- Viewing a history of past policy versions -- excluded per the Entity-Lifecycle Coverage Matrix: this feature exposes no list of historical versions; a dispute timeline's read of which version was acknowledged belongs to FEAT-30/FEAT-16
- Creating the very first policy version during onboarding -- per the dependency map's Cancellation Policy lifecycle line, Create is owned by FEAT-15 (the default proposed during onboarding); this screen governs the edit path, reached from FEAT-15's setup step and from Talia's own settings for every edit thereafter

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard), setup step: policy | Talia continues the onboarding wizard to the cancellation-window step | A recommended default window (platform parameter: `cancellation-window-default-hours`) is pre-filled; no existing Cancellation Policy version exists yet |
| FEAT-27 (Pro Profile & Booking Page Settings) | Talia opens her booking-page settings and selects the cancellation policy item | The current Cancellation Policy version's window_hours is loaded for editing |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Set the window and save (creating a new version) | -- |
| Platform Operator (Support) | Full screen, read-only, for the Pro account under an active help request | None -- the window control and Save button are not shown | Support's attempt to act is impossible because the controls are not rendered; there is no separate denial message because nothing actionable is ever offered |
| The Client (Riley) | No | No | This setup screen is never reached through any Client-facing path; Riley sees only the resulting plain-language wording rendered by FEAT-09.SPEC-002 on the booking page (FEAT-05) and at cancellation (FEAT-10), never this screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29); after signing in, Talia lands on her Daily Schedule Dashboard (FEAT-12), not directly back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress window change that was not yet saved is discarded (the last saved version stands); after re-authentication Talia returns to her Daily Schedule Dashboard (FEAT-12) |

## Layout and Content

**Header:** Screen title "Cancellation Policy" with a back arrow (returns to FEAT-15's next setup step during onboarding, or to FEAT-27 during a later edit) and a "Save" action button (right-aligned, disabled until the window value changes from the currently saved version).

**Body:** A single-column form with two elements, in order:
- **Cancellation window** (numeric stepper input, required): a whole-number-of-hours value, from 1 to 168. Labeled "Clients can cancel or reschedule free of charge up until this many hours before their appointment."
- **Policy preview** (read-only text block, below the window input): the plain-language wording FEAT-09.SPEC-002 derives from the current window value, updating live as Talia adjusts the stepper -- for example, "Clients cancelling or rescheduling less than {window_hours} hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full." The preview always reflects the value currently shown in the stepper, not yet the saved value.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Policy preview block:** Wraps to as many lines as its text requires at any width; never truncated or scrollable within itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate away without saving any unsaved change | Screen closes | Animated transition to the calling context (FEAT-15 or FEAT-27) |
| Cancellation window stepper | Increment/decrement or type a value | Updates the local window value; re-renders the policy preview from the new value; validates via FEAT-09.SPEC-002 | Preview text updates immediately; Save button becomes enabled if the value differs from the saved version | Stepper shows the new value; preview text changes in place |
| Cancellation window stepper | Blur with an out-of-range or non-whole value | Triggers field validation via FEAT-09.SPEC-002 | Error state on the field | "Enter a whole number of hours between 1 and 168." below the field; Save remains disabled |
| Save button | Tap | 1. Validate the window value via FEAT-09.SPEC-002. 2. If valid, trigger FEAT-09.SPEC-002's versioning logic to create a new policy version effective immediately. | Button shows a loading state during save | Success: toast "Cancellation policy updated" and navigate to the calling context (FEAT-15's next step, or back to FEAT-27). Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Cancellation window stepper -> Save.
- **Validation announcements:** When the stepper enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Live preview announcements:** The policy preview text is announced to assistive technology when it changes, so a screen-reader user hears the exact wording a client would see before saving.
- **Keyboard alternatives:** The stepper's increment/decrement is reachable by keyboard (arrow keys or direct numeric entry); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (first use, onboarding) | Stepper pre-filled with the recommended default (platform parameter: `cancellation-window-default-hours`); preview shows the wording for that default; Save enabled | No Cancellation Policy version exists yet for this Pro Account, reached via FEAT-15 | Talia changes the value or taps Save |
| Default (editing) | Stepper pre-filled with the current version's window_hours; preview shows the current wording; Save disabled until a change is made | Screen opens via FEAT-27 with an existing policy version | Talia changes the value |
| Editing | Stepper shows Talia's in-progress value; preview updates live; Save enabled | Talia changes the stepper value | Talia taps Save or navigates away |
| Validation Error | Stepper shows an error state with the message below it; Save disabled | The value is out of range or not a whole number | Talia corrects the value |
| Saving | Save button shows a loading spinner; stepper disabled | Talia taps Save with a valid value | Save completes or fails |
| Error | Error banner at the top of the form with a Retry option; the entered value is preserved | The save operation fails | Talia taps Retry or navigates away |
| Offline/Degraded | N/A -- this screen requires connectivity, consistent with every other Pro setup screen in the product; the Pro is shown the standard Error state if a save is attempted without connectivity, exactly as any other network failure | -- | -- |

## Validation Rules

Validation governed by FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering). See that spec for the window_hours field rule and the exact error message. This screen applies validation on field blur and on Save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (onboarding entry) | FEAT-15's next setup step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Back arrow tap (settings entry) | Booking-page settings | FEAT-27 (Pro Profile & Booking Page Settings) |
| Successful save (onboarding entry) | FEAT-15's next setup step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Successful save (settings entry) | Booking-page settings | FEAT-27 (Pro Profile & Booking Page Settings) |

## Data Model

**Creates:** None directly -- Save triggers FEAT-09.SPEC-002's versioning logic, which is what actually creates the new Cancellation Policy version record.
**Reads:** Cancellation Policy -- the current version's window_hours (editing path) or the recommended default (first-use path, platform parameter: `cancellation-window-default-hours`).
**Updates:** None directly on this screen -- see Creates; every "edit" is a new version, never an overwrite of the existing record, per FEAT-09.SPEC-002.
**Deletes:** None -- no delete path exists for this entity (Entity-Lifecycle Coverage Matrix).

## Business Rules

- Every save creates a new policy version effective immediately (FEAT-09.SPEC-002); this screen never overwrites the current version in place.
- The plain-language wording is always derived from window_hours by FEAT-09.SPEC-002 -- Talia cannot enter custom wording.
- XBR-08: existing bookings keep the policy version they were created against; a save on this screen never changes what a client already booked was told.
- XBR-26: an active cancellation policy is one of the preconditions for the booking link going live; the recommended default proposed during onboarding (FEAT-15) satisfies this precondition until Talia changes it here.

## Edge Cases

- **Talia navigates away with an unsaved change** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your cancellation policy. Check your connection and try again." with a Retry button. The entered value is preserved.
- **Talia enters exactly 1 or exactly 168 hours** -- Both are valid boundary values and save normally.
- **Talia edits her policy while a client is mid-checkout on the current version (concurrent-edit conflict)** -- Per the dependency map's Cancellation Policy Contention note, Talia's save always succeeds and creates a new version immediately; there is no conflict on her side. The client's in-progress checkout is the side affected: if their payment capture completes before Talia's save, they keep the version they acknowledged; if Talia's save commits first and the client has not yet captured payment, the client's payment attempt is refused with refresh and they must re-acknowledge the new wording (FEAT-09.SPEC-002's contention rule) -- resolution is reject-with-refresh on the client's side, never a conflict Talia sees on this screen.
- **Talia reopens this screen immediately after saving** -- The stepper and preview reflect the just-saved version, not a stale cached value.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | Triggers (outbound) | Save action triggers the creation of a new policy version and supplies the field-level validation rule |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound/outbound) | Onboarding's policy step reaches this screen with a recommended default; Save returns to the wizard's next step |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (inbound/outbound) | Talia reaches this screen from her settings for any later edit; Save returns her there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancellation_policy_updated | entry source (onboarding / settings edit), new window_hours value | Save completes successfully | supports success-metrics.md: "Policy Clarity at Booking" |
| cancellation_policy_save_failed | entry source, attempted window_hours value | Save operation fails | N/A -- no Stage 2 metric measures save failures; retained so save reliability for this setup step is observable |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Talia is on the Cancellation Policy Setup screen during onboarding with no existing policy, when the screen loads, then the stepper is pre-filled with the recommended default and the preview shows the wording for that default.

**FEAT-09.SPEC-001-AC-02:** Given Talia is on the Cancellation Policy Setup screen with an existing policy of 24 hours, when the screen loads via FEAT-27, then the stepper shows 24 and the preview shows the current wording, with Save disabled until she changes the value.

**FEAT-09.SPEC-001-AC-03:** Given Talia adjusts the stepper to 48 hours, when the value changes, then the preview text updates immediately to reflect a 48-hour window and Save becomes enabled.

**FEAT-09.SPEC-001-AC-04:** Given Talia enters 0 hours and moves focus away from the stepper, then the field shows the error "Enter a whole number of hours between 1 and 168." and Save remains disabled.

**FEAT-09.SPEC-001-AC-05:** Given Talia enters 169 hours and moves focus away from the stepper, then the field shows the error "Enter a whole number of hours between 1 and 168." and Save remains disabled.

**FEAT-09.SPEC-001-AC-06:** Given Talia sets the window to 72 hours and taps Save, when the save succeeds, then a new policy version is created effective immediately, she sees the toast "Cancellation policy updated," and she returns to the calling context.

**FEAT-09.SPEC-001-AC-07:** Given Talia is on this screen with unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-09.SPEC-001-AC-08:** Given Talia taps Save and the operation fails due to a network error, then an error banner reads "Could not save your cancellation policy. Check your connection and try again." with a Retry button, and her entered value is preserved.

**FEAT-09.SPEC-001-AC-09:** Given Platform Operator (Support) opens this screen for Talia's account under an active help request, when they view it, then the window stepper and Save button are not shown, and the screen is otherwise fully visible read-only.

**FEAT-09.SPEC-001-AC-10:** Given an unauthenticated visitor attempts to reach this screen directly, then they are redirected to the Pro sign-in screen, and after signing in they land on the Daily Schedule Dashboard rather than back on this screen.

**FEAT-09.SPEC-001-AC-11:** Given Talia's session expires while she has an unsaved change on this screen, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the unsaved change is discarded, with the last saved version standing.

**FEAT-09.SPEC-001-AC-12:** Given Talia saves a new window while a client's checkout for the current version is not yet captured, when the client attempts to complete payment, then their attempt is refused with refresh per FEAT-09.SPEC-002's contention rule, while Talia's own save on this screen completed without any conflict shown to her.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (default-onboarding, default-editing, editing, validation error, saving, error) plus offline N/A | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

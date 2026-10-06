---
document_type: spec
spec_type: screen
spec_id: FEAT-02.SPEC-002
spec_name: Per-Service Buffer Override
spec_slug: per-service-buffer-override
parent_feature: FEAT-02
parent_feature_name: Availability & Working Hours Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 15
---

# Screen Spec: Per-Service Buffer Override

## Overview

**Name:** Per-Service Buffer Override
**ID:** FEAT-02.SPEC-002
**Type:** Screen
**Purpose:** Talia (the Pro) sets a buffer-time override for an individual service that genuinely needs more or less gap than her default buffer.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Listing the Pro's active services with their current buffer state (using the default, or overridden)
- Setting, changing, or clearing a buffer override for an individual service
- Validating the override value against the same bounds as the default buffer

**Non-Goals:**
- Editing a service's name, price, duration, or deposit rule -- owned entirely by FEAT-01 (Service & Pricing Management); this screen touches only the buffer_override field
- Setting the account-wide default buffer, minimum booking notice, or booking horizon -- handled on FEAT-02.SPEC-001, which this screen is reached from
- Archiving or reordering services -- owned by FEAT-01; this screen only lists a Pro's currently active services for the purpose of setting their buffer override
- Field-level and cross-field validation logic -- owned by FEAT-02.SPEC-005 (Availability Setup Validation & Limits); this screen only displays the outcome

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Pro taps "Manage" under per-service overrides | None -- screen loads the Pro's current active service list and each service's current buffer_override, if any |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions -- set, change, or clear a per-service buffer override; save | -- |
| The Client (Riley) | No | No | Clients have no access to Service & Availability Setup (Access Matrix, user-persona.md); there is no client-facing navigation path into this screen at all |
| Platform Operator (Support) | Full screen, read-only | No -- all edit and save controls are hidden | A direct save attempt is not reachable from the UI (controls are hidden, not merely disabled); if attempted through a stale or replayed request, the response is "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved edits on the form are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Buffer Overrides by Service" with a back arrow (returns to FEAT-02.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A list, one row per active service in the Pro's current display order (FEAT-01), each row showing:
- Service name
- A numeric buffer-override field, pre-filled with the service's current override value if one exists, otherwise shown empty with placeholder text "Using default ({default_buffer} min)"
- A "Clear override" control, shown only on rows that currently have an override set, which resets the row to use the account default

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Service rows stack full width, one per row, with the service name above its buffer field.
- **Medium size class and above:** Service rows render as a single table with the service name and buffer field side by side, capped at a consistent platform-wide form width; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-02.SPEC-001 | Screen closes | Standard transition; unsaved edits prompt the discard dialog first if any exist |
| Buffer-override field (per service) | Type | Captures the override value in minutes for that service | Field shows entered value | Validated via FEAT-02.SPEC-005 |
| "Clear override" control (per service) | Tap | Clears the override so the service falls back to the account default | Field returns to its empty, placeholder state | Row shows "Using default ({default_buffer} min)" |
| Save button | Tap | 1. Validate every entered override via FEAT-02.SPEC-005. 2. If valid, trigger FEAT-02.SPEC-003 (Availability Rule Versioning), which in turn triggers FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging). 3. On success, show confirmation and remain on screen. | Button shows loading state during save | Success: "Overrides saved" confirmation banner. Failure: inline field errors (validation) or a retry banner (save failure). |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> each service row in display order (buffer field, then clear-override control when present) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Overrides saved" confirmation is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen, including clearing an override, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no active services) | A guided message: "Add a service first to set buffer overrides" with a link to FEAT-01 | The Pro has zero active services | The Pro adds at least one active service and returns here |
| Filling | Service list shown with any entered edits; Save enabled | Pro edits any override field | Pro taps Save or navigates away |
| Validating | Save button shows a loading spinner | Pro taps Save | Validation (FEAT-02.SPEC-005) completes (pass or fail) |
| Validation Error | Failed field(s) highlighted with their error messages shown inline | Validation fails | Pro corrects the field(s) and re-triggers validation |
| Saving | Save button shows a loading spinner, form fields disabled | Validation passes | FEAT-02.SPEC-003 completes or fails |
| Success | Confirmation banner "Overrides saved"; remains on this screen | Save completes successfully | Banner dismisses after a few seconds or on next edit |
| Error | Error banner "Could not save your overrides. Check your connection and try again." with a Retry action; all entered values remain on screen | The save operation (FEAT-02.SPEC-003) fails | Pro taps Retry or navigates away |
| Offline/Degraded | N/A -- this is a setup screen used between clients on a stable connection, not an in-the-moment mobile flow, consistent with FEAT-02.SPEC-001's own stance | -- | -- |

## Validation Rules

Validation governed by FEAT-02.SPEC-005 (Availability Setup Validation & Limits). See that spec for the buffer-override bounds (the same bounds as the account default buffer). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | -- |
| "Add a service" link (Empty state) | Service list / add-service screen | FEAT-01 (Service & Pricing Management) |
| Successful save | Remains on this screen with the Success state shown | -- |

## Data Model

**Creates:** None -- this screen never creates a Service record.
**Reads:** Service -- name, display_order, status (Active only) for the list; existing buffer_override values to pre-fill each row. Availability Rule -- default_buffer, to show as the placeholder fallback value on rows with no override.
**Updates:** Service -- buffer_override field only, per service, on save. (Flagged discrepancy, per feature-overview.md: the dependency map's Service lifecycle line lists only FEAT-01 as an updater of Service; this Brief records buffer_override as written by this screen and flags the line for the Requirements Architect to reconcile.)
**Deletes:** None.

## Business Rules

- Every entered override is validated against FEAT-02.SPEC-005's buffer bounds before FEAT-02.SPEC-003 is triggered -- the Pro cannot save an out-of-bounds override.
- A passing save creates a new dated Availability Rule version through FEAT-02.SPEC-003, exactly as a save on FEAT-02.SPEC-001 does, even though this screen's own writes land on the Service entity.
- Every new version triggers a check of existing confirmed bookings against the new rule (FEAT-02.SPEC-004); a conflicting booking is never silently cancelled -- it is flagged for the Pro's attention on Pro Booking Management (XBR-11).
- A service with no override uses the account's default_buffer, read from the current Availability Rule.
- Clearing an override removes it entirely rather than setting it to zero -- the row reverts to following the account default, including any future default changes.
- Only the Pro's currently active services are listed; an archived service's prior override is retained on its own record but not shown or editable here, consistent with FEAT-01's archive behavior.
- Every navigation to this screen requires a signed-in Pro (XBR-29).

## Edge Cases

- **Pro navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your overrides. Check your connection and try again." with a Retry button; all entered values remain on screen.
- **Pro enters an override equal to the account default** -- Accepted; the override is still stored explicitly and takes precedence even if the account default later changes.
- **A service is archived by the Pro (via FEAT-01) while this screen is open with unsaved edits for it** -- That service's row is removed on the next load; any unsaved edit for it is discarded silently since the service is no longer bookable.
- **Two Pro sessions save different overrides for the same service at nearly the same time** -- Last-write-wins between the Pro's own sessions, consistent with the dependency map's Contention note for Service: the two screens (FEAT-01 and this one) write disjoint fields, so a concurrent FEAT-01 edit to name/price/duration never conflicts with this screen's buffer_override write.
- **Pro's account has services created after this screen was last opened** -- The list reflects the Pro's current active service set on every fresh load; a newly added service appears with no override (using the default) the next time this screen opens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Navigation (inbound), Navigation (outbound) | Pro arrives from and returns to that screen |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | Buffer-override bounds applied on save |
| FEAT-02.SPEC-003 (Availability Rule Versioning) | Triggers (outbound) | A passing save triggers creation of a new dated Availability Rule version |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound, indirect via FEAT-02.SPEC-003) | Every new version saved through this screen triggers the conflict check |
| FEAT-01 (Service & Pricing Management) | References (inbound), Navigation (outbound) | Reads the Pro's active service list and display order; the Empty state links there to add a first service |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | References (inbound) | Requires a signed-in Pro for any access |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_buffer_override_set | service reference, override value (minutes) | A save completes successfully with a new or changed override | supports success-metrics.md: "Availability Setup Accuracy" |
| service_buffer_override_cleared | service reference, prior override value | A save completes successfully clearing an existing override | supports success-metrics.md: "Availability Setup Accuracy" |
| service_buffer_override_validation_failed | service reference, entered value | Save is blocked by a validation failure on an override field | supports success-metrics.md: "Availability Setup Accuracy" |

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Talia has three active services and none has an override set, then this screen lists all three with each buffer field showing the "Using default ({default_buffer} min)" placeholder.

**FEAT-02.SPEC-002-AC-02:** Given Talia enters 30 minutes as the override for her "Full Set" service, when she taps Save and validation passes, then FEAT-02.SPEC-003 creates a new Availability Rule version reflecting that service's buffer_override as 30.

**FEAT-02.SPEC-002-AC-03:** Given Talia has an existing override on a service, when she taps "Clear override" and saves, then that service's row shows the default-buffer placeholder again and its buffer_override field is cleared.

**FEAT-02.SPEC-002-AC-04:** Given Talia enters an override value outside the bounds defined by FEAT-02.SPEC-005, when the field loses focus, then an inline error appears and Save does not proceed until it is corrected.

**FEAT-02.SPEC-002-AC-05:** Given Talia has zero active services, then this screen shows the guided message "Add a service first to set buffer overrides" with a link into FEAT-01.

**FEAT-02.SPEC-002-AC-06:** Given Talia taps Save with valid overrides entered, then the Save button shows a loading state, FEAT-02.SPEC-003 runs, and on success an "Overrides saved" banner appears while she remains on this screen.

**FEAT-02.SPEC-002-AC-07:** Given Talia taps Save while a prior save is still in progress, when she taps Save a second time, then the second tap is ignored and the button remains in its loading state.

**FEAT-02.SPEC-002-AC-08:** Given Talia's save fails due to a connectivity error, then an error banner reading "Could not save your overrides. Check your connection and try again." appears with a Retry action, and all entered values remain on screen.

**FEAT-02.SPEC-002-AC-09:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-02.SPEC-002-AC-10:** Given Talia archives a service (via FEAT-01) while this screen has an unsaved override for it, when the screen next loads, then that service's row no longer appears and the discarded edit has no effect.

**FEAT-02.SPEC-002-AC-11:** Given Riley (the Client) has no navigational path to this screen, then no client-facing entry point into Per-Service Buffer Override exists anywhere in the product.

**FEAT-02.SPEC-002-AC-12:** Given Platform Operator (Support) opens Talia's account for troubleshooting, when Support views this screen, then all fields are shown read-only and no Save control is visible.

**FEAT-02.SPEC-002-AC-13:** Given a visitor who is not signed in as a Pro attempts to reach this screen, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-002-AC-14:** Given Talia's session expires while she has unsaved edits on this screen, when the expiry is detected, then a dialog reading "Your session has expired. Sign in to continue." appears, and her unsaved edits are restored after she signs back in.

**FEAT-02.SPEC-002-AC-15:** Given Talia saves an override that, combined with a booking's fixed duration, leaves an existing confirmed booking outside the new rule's fit, then this screen still shows its own "Overrides saved" success state, and the conflict is surfaced separately on Pro Booking Management via FEAT-02.SPEC-004, never as an error here.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (empty, filling, validating, validation error, saving, success, error, offline/degraded) | 8 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |

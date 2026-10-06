---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-003
spec_name: Edit Service
spec_slug: edit-service
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 11
---

# Screen Spec: Edit Service

## Overview

**Name:** Edit Service
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** The Pro updates an existing service's fields or archives it; Platform Operator (Support) views the same service details read-only for troubleshooting.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Loading and displaying an existing service's fields, pre-populated
- Editing name, price, duration, or deposit rule and saving the change
- Initiating the Archive action, including the impact-check hand-off
- Support's read-only view of the same fields for troubleshooting

**Non-Goals:**
- Creating a new service -- handled by FEAT-01.SPEC-002 (Add Service), which shares this screen's form layout and validation
- Reordering services or reactivating an archived one -- both handled by FEAT-01.SPEC-001 (Service List)
- Determining whether upcoming bookings exist before an archive completes -- that check and its warning are owned by FEAT-01.SPEC-006 (Archive Impact Check); this screen only triggers it and reflects its result
- Retroactively changing the price, duration, or deposit already agreed on a confirmed booking -- explicitly prevented by FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time); no control on this screen can do this

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Service List) | Pro or Support taps a service row | The selected service's identifier; the screen loads that service's current fields |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Services entry during an active support session | The Pro account under review; screen renders read-only for Support with no action controls |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit any field, save, and Archive | -- |
| Platform Operator (Support) | Full screen, all fields, read-only | None -- no field is editable and no Save or Archive control is rendered; a persistent banner reads "Support view -- no changes can be made here" | A direct attempt to interact with a field is impossible because every input renders as static text in this role; there is no separate denial dialog because nothing actionable is ever offered |
| The Client (Riley) | No | No | Service & Pricing Management screens are never reached through any Client-facing path |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered but unsaved edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title showing the service's name, a back arrow (returns to FEAT-01.SPEC-001), and a "Save" action button (right-aligned, Pro only). Support sees no Save button; in its place, the "Support view -- no changes can be made here" banner spans the header's width.

**Body:** The same single-column form layout as FEAT-01.SPEC-002 (Add Service), pre-populated with the service's current values:
- Name (text input, required)
- Price (numeric input, required)
- Duration (numeric input in minutes, required)
- Deposit Rule (toggle between "Fixed amount" and "Percentage," with the corresponding value pre-filled)

Below the form, the same live client-facing preview panel as Add Service, reflecting the current (or in-progress edited) values.

For the Pro only, an "Archive this service" action appears below the preview panel, visually separated from the editable fields.

**Footer:** None -- Save is in the header; Archive is in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; preview panel and Archive action stacked below it.
- **Medium size class and above:** Form and preview panel appear side by side, matching FEAT-01.SPEC-002's layout; the Archive action remains full-width below both, horizontally centered within the capped content width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-001 (Service List) | Screen closes | Standard back transition |
| Name/Price/Duration/Deposit Rule fields (Pro only) | Type / toggle | Same behavior as FEAT-01.SPEC-002 -- captures input, updates the preview panel live | Field shows entered value; preview updates | Standard input focus state |
| Any field (Pro only) | Blur | Triggers field validation via FEAT-01.SPEC-004 | Error state on field if invalid | Field-level error message if invalid |
| Save button (Pro only) | Tap | 1. Validate all fields via FEAT-01.SPEC-004. 2. If valid, save the changes; existing confirmed bookings are governed by FEAT-01.SPEC-005 and are never recalculated. | Button shows loading state during save | Success: toast "{Service name} updated" and navigate to FEAT-01.SPEC-001. Failure: inline error messages, entered values preserved. |
| Archive this service (Pro only) | Tap | Triggers FEAT-01.SPEC-006 (Archive Impact Check) | Screen shows a loading indicator on the Archive action while the check runs | If no upcoming bookings: proceeds directly to the archive confirmation step below. If upcoming bookings exist: an impact warning modal appears with the count and Confirm/Cancel |
| Impact warning modal -- Confirm (Pro only) | Tap | Sets the service's status to Archived per FEAT-01.SPEC-005's lock guarantee | Screen closes | Toast "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected." Navigate to FEAT-01.SPEC-001 (Archived filter) |
| Impact warning modal -- Cancel (Pro only) | Tap | No action | Modal closes | Edit Service screen remains open, service remains Active |
| Fields (Support) | -- | Rendered as static text, not interactive | -- | -- |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Price -> Duration -> Deposit Rule toggle -> Deposit value -> Archive this service -> Save. For Support, focus order skips directly from the banner to the back arrow, since no field or action is interactive.
- **Validation announcements:** Identical to FEAT-01.SPEC-002 -- error messages are announced to assistive technology and programmatically associated with their field.
- **Archive confirmation:** The impact warning modal, when shown, receives focus immediately and its full text (including the upcoming-booking count) is announced.
- **Read-only announcement:** For Support, the "Support view -- no changes can be made here" banner is announced when the screen first renders.
- **Keyboard alternatives:** Every action, including Archive and the impact-warning modal's Confirm/Cancel, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton form while the service's current fields load | Screen first opens | Fields finish loading |
| Loaded/Filling (Pro) | Form pre-populated, editable, Save enabled | Load completes (Pro) | Pro edits, saves, archives, or navigates away |
| Read-only (Support) | Form pre-populated, all fields static text, banner visible | Load completes (Support) | Support navigates away |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails on blur or submit | Pro corrects the field and re-triggers validation |
| Saving | Save button shows a loading state, form fields disabled | Validation passes | Save completes or fails |
| Archiving | Archive action shows a loading state while the impact check runs | Pro taps Archive this service | Impact check returns (no-warning path or warning modal shown) |
| Error | Error banner: "Couldn't save this service. Try again." (save) or "Couldn't check upcoming bookings for this service. Try again." (archive), each with Retry | Save operation or archive impact check fails | Pro taps Retry and the operation succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow (product-features.md, States field) |

## Validation Rules

Validation governed by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation). See that spec for all field-level and cross-field rules, including the minimum chargeable deposit. This screen applies validation on field blur and on form submit, identically to FEAT-01.SPEC-002.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-001 (Service List) | -- |
| Successful save | FEAT-01.SPEC-001 (Service List) | -- |
| Archive confirmed | FEAT-01.SPEC-001 (Service List, Archived filter) | -- |
| Cancel (if unsaved changes, Pro) | FEAT-01.SPEC-001 (Service List), after confirmation | -- |

## Data Model

**Creates:** None.
**Reads:** Service -- all fields (name, price, duration, deposit_rule, buffer_override, display_order, status). buffer_override is displayed nowhere on this screen (it is owned and shown by FEAT-02) -- the field is simply not part of this screen's data surface.
**Updates:** Service -- name, price, duration, deposit_rule (Pro only, subject to FEAT-01.SPEC-004 validation); status (Pro only, Active -> Archived via the Archive action, governed by FEAT-01.SPEC-005 and FEAT-01.SPEC-006).
**Deletes:** None -- archiving is a soft-delete (status change), never a hard delete (dependency map, Service Lifecycle).

## Business Rules

- Field validation is enforced by FEAT-01.SPEC-004 -- the Pro cannot save with invalid or unchargeable values.
- XBR-04: editing price, duration, or deposit rule here applies to future bookings only; any booking already Confirmed (or later in its lifecycle) keeps the price, duration, and deposit it was given at booking time, per FEAT-01.SPEC-005.
- Archiving always runs FEAT-01.SPEC-006 (Archive Impact Check) first -- the Pro cannot skip straight to archiving without the check completing.
- XBR-11: archiving a service with upcoming bookings never silently cancels them; those bookings are honored as-is, and the resulting conflict is flagged on the Pro's booking management surface (FEAT-30) for an explicit Pro decision -- this screen never auto-resolves it.
- Support's read-only access covers every field on this screen with no exception -- Service carries no personal data, so there is no field Support is additionally barred from (unlike, for example, a Pro's private client notes).

## Edge Cases

- **Service changed by the Pro from another device between load and save** -- Per the dependency map's Contention note for Service: resolution is last-write-wins between the Pro's own sessions (not reject-with-refresh); the save that completes last is the version that stands, and the earlier device's screen silently reflects the new values on its next load or reopen.
- **Pro archives a service with zero upcoming bookings** -- FEAT-01.SPEC-006 finds no upcoming bookings and the archive proceeds immediately with no warning modal; the Pro sees only the success toast.
- **Pro archives a service with upcoming bookings, then cancels the warning** -- The service remains Active and unarchived; no field on this screen changes.
- **Pro navigates away with unsaved field edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Support opens a service that the Pro archives moments later from another device** -- Support's screen is a snapshot at load time; it does not live-update, so it continues showing the values as loaded until Support reopens the screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | References (inbound) | Validation and the minimum-chargeable-deposit rule applied to form fields |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (inbound) | Governs that saved edits and confirmed archives never retroactively change an existing booking |
| FEAT-01.SPEC-006 (Archive Impact Check) | Triggers (outbound) | Archive action fires the impact check before any archive is confirmed |
| FEAT-01.SPEC-001 (Service List) | Navigation (inbound/outbound) | Pro or Support arrives from the list and returns to it on back, save, or confirmed archive |
| FEAT-30 (Pro Booking Management) | References (outbound) | The conflict from archiving a service with upcoming bookings is surfaced on that feature's booking management surface, per XBR-11 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| service_updated | field(s) changed | Successful save completes | supports success-metrics.md: "Service Setup Confidence" |
| service_update_validation_failed | field(s) in error, error type(s) | Save attempt is blocked by validation | supports success-metrics.md: "Service Setup Confidence" |
| service_archived | had upcoming bookings at time of archive (yes/no) | Archive is confirmed | N/A -- success-metrics.md's "Service Setup Confidence" measures the add/edit attempt itself, not archive; no other metric connected to this feature covers archive behavior |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Talia opens an existing service, when the screen loads, then all fields are pre-populated with the service's current name, price, duration, and deposit rule.

**FEAT-01.SPEC-003-AC-02:** Given Talia edits the price of an existing service and taps Save, when the save succeeds, then the change is applied for future bookings, a toast confirms "{Service name} updated," and she returns to FEAT-01.SPEC-001.

**FEAT-01.SPEC-003-AC-03:** Given Talia edits a service that has an existing Confirmed booking, when she saves the change, then that booking's own price, duration, and deposit remain exactly as they were at booking time, per FEAT-01.SPEC-005.

**FEAT-01.SPEC-003-AC-04:** Given Talia taps "Archive this service" on a service with zero upcoming bookings, when FEAT-01.SPEC-006's check completes, then the archive proceeds immediately with no warning modal, and a toast confirms "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected."

**FEAT-01.SPEC-003-AC-05:** Given Talia taps "Archive this service" on a service with two upcoming bookings, when FEAT-01.SPEC-006's check completes, then an impact warning modal appears naming the count, with Confirm and Cancel options, and the archive does not proceed until she taps Confirm.

**FEAT-01.SPEC-003-AC-06:** Given Talia sees the impact warning modal, when she taps Cancel, then the modal closes, the service remains Active, and no field on the screen has changed.

**FEAT-01.SPEC-003-AC-07:** Given Platform Operator (Support) opens a Pro's service during a help request, when the screen renders, then every field displays as static text, the banner "Support view -- no changes can be made here" is visible, and no Save or Archive control appears anywhere.

**FEAT-01.SPEC-003-AC-08:** Given Talia is editing a service on her phone while the same service is also open on her tablet, when she saves from the phone and then, moments later, saves a different change from the tablet, then the tablet's save is the version that stands, per the dependency map's last-write-wins resolution for Service.

**FEAT-01.SPEC-003-AC-09:** Given Talia is on the Edit Service screen with unsaved field edits, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-01.SPEC-003-AC-10:** Given Talia's archive impact check fails to complete, when the failure occurs, then an error banner reads "Couldn't check upcoming bookings for this service. Try again." with a Retry action, and the service remains Active.

**FEAT-01.SPEC-003-AC-11:** Given Talia enters a deposit rule whose resulting deposit falls below the minimum chargeable amount and taps Save, then FEAT-01.SPEC-004's validation error appears and the save does not proceed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (loading, loaded/filling, read-only, validation error, saving, archiving, error, offline/degraded N/A) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |

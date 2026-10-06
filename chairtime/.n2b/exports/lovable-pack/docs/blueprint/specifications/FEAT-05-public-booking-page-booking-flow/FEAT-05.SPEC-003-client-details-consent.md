---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-003
spec_name: Client Details & Consent
spec_slug: client-details-consent
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Client Details & Consent

## Overview

**Name:** Client Details & Consent
**ID:** FEAT-05.SPEC-003
**Type:** Screen
**Purpose:** Client enters their name and phone, opts into text messages, provides an email if declining texts, and adds an optional note for the Pro.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Capturing name, phone, texting opt-in, conditionally-required email, and an optional note
- Recognizing a returning client by phone number, in place on this screen, and pre-filling their name (no navigation to FEAT-06 screens)
- Creating or matching the Client record and creating the Messaging Consent record on continue
- Advancing to the policy acknowledgment and checkout step

**Non-Goals:**
- Field-level validation rules (format, length, conditional requirements) -- defined in full by FEAT-05.SPEC-007 (Booking Details Field Validation); this screen enforces those rules but does not define them
- Updating a returning client's own email or consent after this booking -- owned by FEAT-06 (Client Booking Identity), which the returning client reaches through a manage link, not through this screen on a later visit
- Collecting or displaying the deposit and cancellation policy -- handled entirely by FEAT-05.SPEC-004, the next step
- Health or medical intake -- excluded per scope-boundaries.md SC-08; the optional note is hinted not to contain medical information and no structured intake question is offered

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-002 (Slot Selection) | Client picks a time that passes the live re-check | Chosen service, chosen time (not yet held) |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Client navigates back before paying | Same service and chosen time; previously entered name, phone, opt-in, email, and note preserved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Enter all fields and continue | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering | Fill and continue exactly as a client would; no real Client or Messaging Consent record is created against a real client (preview data is discarded, per Business Rules) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View field layout only -- never a real client's entered data mid-flow, since support only reaches this rendering outside a live session | Form fields are not editable; consistent with XBR-24 |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire on this public flow; no checkout hold exists yet on this step; the hold (platform parameter: `checkout-hold-timeout-minutes`) starts only when the client advances into the payment step | N/A | N/A |

## Layout and Content

**Header:** Back arrow (returns to FEAT-05.SPEC-002) with the chosen service and chosen time shown as persistent context.

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Phone (text input, required)
- Text message opt-in (checkbox, never pre-checked, with the exact consent wording shown beside it)
- Email (text input, required only when the opt-in checkbox is unchecked; optional otherwise)
- Note to the Pro (multi-line text input, optional, with a visible hint: "e.g., \"first full set\" -- please don't include medical information")

**Footer:** "Continue" action button, full width, advances to FEAT-05.SPEC-004.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width, Continue in the footer.
- **Medium size class and above:** Form remains single-column, capped at a comfortable form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-002 (Slot Selection), entries preserved | Screen closes | Standard backward transition |
| Phone input | Blur, matching an existing Client record for this Pro | Look up the returning client in place, applying the same phone-based identity-match rule FEAT-06 (Client Booking Identity) uses; no navigation occurs | Name field auto-fills with the matched client's name | Name field shows the pre-filled value, editable by the client |
| Name input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 on invalid blur |
| Phone input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 on invalid blur |
| Text opt-in checkbox | Tap | Toggles opt-in state; when checked, the Email field becomes optional; when unchecked, Email becomes required | Email field's required indicator updates immediately | Checkbox shows checked/unchecked state with the exact consent wording remaining visible |
| Email input | Type / blur | Captures text input; validated per FEAT-05.SPEC-007 (conditionally required) | Field shows entered text or error state | Standard input feedback; error message per FEAT-05.SPEC-007 |
| Note input | Type | Captures text input; length-checked per FEAT-05.SPEC-007 | Field shows entered text and remaining character count | Standard input feedback |
| Continue button | Tap | 1. Validate all fields via FEAT-05.SPEC-007. 2. If valid, create or match the Client record and create the Messaging Consent record (consent capture is handled by FEAT-14.SPEC-003). 3. Navigate to FEAT-05.SPEC-004. | Button shows loading state during processing | Success: navigate to FEAT-05.SPEC-004. Failure: inline field-level error messages, focus moves to the first invalid field |
| Continue button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Phone -> Text opt-in checkbox -> Email -> Note -> Continue.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Consent wording:** The exact texting consent wording is always presented adjacent to, and read together with, the opt-in checkbox by assistive technology -- it is never conveyed by a separate disconnected label.
- **Keyboard alternatives:** Every action on this screen, including the opt-in checkbox, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | All fields empty except any auto-filled name from a phone match; Continue enabled | Screen first opens for this booking attempt | Client begins typing or continues |
| Filling | Form fields contain client input, Continue enabled | Client types in any field | Client taps Continue or navigates away |
| Validating | Continue button shows a loading indicator | Client taps Continue | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them, per FEAT-05.SPEC-007 | Validation fails | Client corrects the field(s) and re-triggers validation |
| Processing | Continue button shows a loading indicator, form disabled | Validation passes, Client/Consent creation begins | Processing completes or fails |
| Error | Error banner: "Something went wrong saving your details. Try again." with a Retry option | Client or Consent creation fails after validation passes | Client taps Retry, or navigates away with entered data preserved |
| Offline/Degraded | Banner: "Check your connection and try again." at the top; form remains fillable but Continue is disabled until connectivity returns; nothing is queued, since this step requires a live connection (per the product-wide offline stance) | Connectivity lost while the screen is open | Connectivity restored -- Continue re-enables |

## Validation Rules

Validation governed by FEAT-05.SPEC-007 (Booking Details Field Validation). See that spec for all field-level and cross-field rules (name, phone, conditionally required email, actively-checked opt-in, note length/hint). This screen applies validation on field blur and on Continue.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-002 (Slot Selection) | -- |
| Successful Continue | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | -- |

## Data Model

**Creates:** Client record -- name, phone, email (if provided), booking_notes (the optional note) set from form input, scoped to this Pro. Messaging Consent record -- channel: text, state: Granted, timestamp, and the exact consent wording shown, created only if the opt-in checkbox is checked.
**Reads:** An existing Client record, via FEAT-06's phone-based lookup, to pre-fill name for a returning client.
**Updates:** None -- a returning client's own later updates to email or consent are owned by FEAT-06, not this screen.
**Deletes:** None.

## Business Rules

- A phone number matching an existing Client record within this Pro resolves to that single record rather than creating a duplicate (dependency map's Client contention note); the matched client's name is pre-filled and editable. The lookup runs in place on this screen and applies the same phone-based identity-match rule FEAT-06 uses (including for FEAT-06.SPEC-001's access-link requests); it does not navigate to any FEAT-06 screen.
- The Booking Page Availability Gate (FEAT-05.SPEC-008) is evaluated on every load of this screen, so a paused or unavailable page never shows this form (XBR-06, XBR-14, XBR-27).
- Creating the Messaging Consent record on Continue triggers FEAT-14.SPEC-003 (Consent Capture at Booking), which records the consent state, exact wording, and timestamp under FEAT-14's rules (XBR-15).
- If the client declines texts (opt-in left unchecked), no Messaging Consent record is created for the text channel at all; the client must instead supply an email so confirmations and reminders can arrive by the email fallback (XBR-15).
- The texting opt-in is never pre-checked -- it must be an explicit, affirmative action, with the exact wording shown kept as evidence (US SMS-consent rules, ASMP-24).
- Every value entered on this screen persists if the client navigates back to FEAT-05.SPEC-002 or forward to FEAT-05.SPEC-004 and returns, or if a failed step is retried, per the feature's Shared UI Pattern.
- In preview mode, the Pro's entries on this screen are discarded on completion and never create a real Client or Messaging Consent record.

## Edge Cases

- **Client navigates away and returns with unsaved changes** -- No confirmation dialog is needed; entered values are preserved automatically per the feature's persistence rule, so nothing is at risk of being discarded.
- **Client taps Continue twice rapidly** -- The second tap is ignored while the first Continue action is in progress (button in loading state).
- **A second, unrelated booking attempt with the same phone number arrives at effectively the same time (two first bookings for the same phone close together)** -- Both attempts resolve to a single merged Client record rather than creating a duplicate; field values from whichever Continue action completes second are last-write-wins on the shared Client record's contact fields, per the dependency map's Client contention note.
- **Client enters a phone number, then changes it before continuing** -- Any name pre-fill from the original phone's match is cleared, and a fresh lookup runs against the new number on blur.
- **All optional fields (email when texting is opted in, note) left empty** -- Continue proceeds; email and note are stored as empty.
- **Network failure during Client/Consent creation, after validation passes** -- Error banner: "Something went wrong saving your details. Try again." with Retry; entered form data is preserved.
- **Client loses connectivity while filling the form** -- The Offline/Degraded state renders; Continue is disabled until connectivity returns, since this step requires a live connection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (inbound) | Client arrives here after picking a time that passes the live re-check |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (outbound) | Successful Continue advances the client here |
| FEAT-05.SPEC-007 (Booking Details Field Validation) | References (inbound) | Validation and consent rules applied to form fields |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | Governs whether this screen renders on every load |
| FEAT-06 (Client Booking Identity) | References (outbound) | Source of the phone-based identity-match rule applied by the in-place lookup; no navigation to any FEAT-06 screen (FEAT-06.SPEC-001 is not an entry from this screen) |
| FEAT-14.SPEC-003 (Consent Capture at Booking) -- within FEAT-14 (Messaging Consent Management) | Triggers (outbound) | Continue's creation of the Messaging Consent record triggers consent capture |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| client_details_submitted | returning client (yes/no), texting opted in (yes/no), note provided (yes/no) | Continue succeeds and the client advances | supports success-metrics.md: "Booking Completion Speed" |
| returning_client_recognized | -- | Phone lookup matches an existing Client record and pre-fills the name | supports success-metrics.md: "Self-Service Access Success" |
| client_details_validation_failed | field(s) that failed | Continue is blocked by a validation error | supports success-metrics.md: "Booking Completion Speed" |

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Riley has just picked a time on FEAT-05.SPEC-002, when the Client Details & Consent screen loads, then Riley sees empty Name, Phone, opt-in, Email, and Note fields with the chosen service and time shown as context.

**FEAT-05.SPEC-003-AC-02:** Given Riley enters a phone number matching an existing Client record with this Pro, when the phone field loses focus, then the Name field auto-fills with the matched client's name in place, with no navigation away from this screen.

**FEAT-05.SPEC-003-AC-03:** Given Riley fills in valid Name and Phone, checks the texting opt-in, and taps Continue, then the Messaging Consent record is created with state Granted and the exact wording shown (triggering FEAT-14.SPEC-003 consent capture), and Riley advances to FEAT-05.SPEC-004.

**FEAT-05.SPEC-003-AC-04:** Given Riley leaves the texting opt-in unchecked, when Riley taps Continue without an email entered, then the Email field shows a required-field error and Continue does not proceed.

**FEAT-05.SPEC-003-AC-05:** Given Riley leaves the texting opt-in unchecked and provides a valid email, when Riley taps Continue, then no Messaging Consent record is created for the text channel, and Riley advances with the email stored for the fallback channel.

**FEAT-05.SPEC-003-AC-06:** Given Riley is on this screen, when Riley looks at the opt-in checkbox on first load, then it is unchecked by default -- never pre-checked.

**FEAT-05.SPEC-003-AC-07:** Given Riley enters a note exceeding the length limit defined by FEAT-05.SPEC-007, when the note field loses focus, then the field shows the corresponding error message from that spec.

**FEAT-05.SPEC-003-AC-08:** Given Riley taps Continue twice in rapid succession, when the first tap has already begun processing, then the second tap has no additional effect.

**FEAT-05.SPEC-003-AC-09:** Given Riley loses connectivity while filling the form, when the Offline/Degraded state renders, then Continue is disabled and Riley sees "Check your connection and try again."

**FEAT-05.SPEC-003-AC-10:** Given Riley navigates back to FEAT-05.SPEC-002 from this screen and returns, when the screen reloads, then all previously entered field values are preserved.

**FEAT-05.SPEC-003-AC-11:** Given two first-time bookings with the same phone number arrive close together for the same Pro, when both complete Continue, then the Client record is merged rather than duplicated, per the dependency map's Client contention note.

**FEAT-05.SPEC-003-AC-12:** Given Talia previews her own booking page and reaches this screen, when she fills in the form and taps Continue, then the identical flow advances her with no real Client or Messaging Consent record created.

**FEAT-05.SPEC-003-AC-13:** Given Riley's Client/Consent creation fails due to a processing error after validation passes, when the failure occurs, then Riley sees "Something went wrong saving your details. Try again." with entered data preserved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 7 (empty, filling, validating, validation error, processing, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |

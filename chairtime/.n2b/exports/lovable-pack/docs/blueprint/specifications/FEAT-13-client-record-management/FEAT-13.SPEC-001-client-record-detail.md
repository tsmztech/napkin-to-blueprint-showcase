---
document_type: spec
spec_type: screen
spec_id: FEAT-13.SPEC-001
spec_name: Client Record Detail
spec_slug: client-record-detail
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Client Record Detail

## Overview

**Name:** Client Record Detail
**ID:** FEAT-13.SPEC-001
**Type:** Screen
**Purpose:** The Pro views a client's contact details, private note, and full booking history with this Pro, and edits the private note directly on this screen.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Displaying the client identity header (name, phone, email)
- Displaying the private note field, editable inline, with save and error handling
- Displaying the client's full booking history with this Pro (past and upcoming bookings)
- Navigation to Client Contact Edit (FEAT-13.SPEC-002) and Client Deletion Confirmation (FEAT-13.SPEC-003)
- Offline/degraded behavior consistent with the Brief's Non-Functional Notes

**Non-Goals:**
- Editing the client's name, phone, or email -- handled by FEAT-13.SPEC-002 (Client Contact Edit); this screen only edits the private note inline
- A client-facing view of this screen -- excluded per the Access Matrix (user-persona.md), which gives the Client role "None" on Client Records; a Client's implicit view of their own contact details runs entirely through FEAT-06 (Client Booking Identity), never through this screen
- A list or search view across a Pro's clients -- owned by FEAT-24 (Client List Search & Filter, v1) and summarized on FEAT-12 (Pro Daily Schedule Dashboard); this screen has no client-list entry point of its own, per the Brief's Entity-Lifecycle Coverage Matrix
- Exporting the client's data -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle), per the dependency map's Client entity Lifecycle line

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro taps a client's name on a booking row | Client ID |
| FEAT-24.SPEC-001 (Client Search & Filter) (Client List Search & Filter, v1) | Pro taps a client found via search | Client ID |
| FEAT-13.SPEC-002 (Client Contact Edit) | Pro taps Save with valid contact data | Updated contact fields reflected on return |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Pro cancels the deletion flow before confirming | None -- client record unchanged |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All documented actions (edit private note, navigate to contact edit, navigate to deletion) | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; no in-product navigation ever routes a Client here -- a Client's own view of their contact details is delivered entirely by FEAT-06 (Client Booking Identity) |
| Platform Operator (Support) | No -- Support's equivalent read-only view of this same Client entity (excluding the private_note field) is delivered by FEAT-19 (Platform Support Read-Only Access) reading this entity directly, never by this screen | No | If Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 -- this is a Pro-facing screen, not the support view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress, unsaved private-note text is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title showing the client's name, with a back arrow (returns to the screen the Pro arrived from -- FEAT-12 or FEAT-24) and an overflow menu containing "Edit contact" (navigates to FEAT-13.SPEC-002) and "Delete client" (navigates to FEAT-13.SPEC-003).

**Client Identity Header:** Below the screen header -- the client's name, phone number, and email address, displayed read-only (the same identity-summary display shared with FEAT-13.SPEC-002 and FEAT-13.SPEC-003, per the Brief's Shared UI Patterns). Email is shown as "-- (not on file)" when the client has no email address.

**Private Note Section:** A labeled multi-line text field ("Private note"), pre-filled with the note's current text (or empty if none exists yet). Directly below the field, a save action button, initially inactive until the text changes. A helper caption reads "Only you can see this note" (reflecting private_note's Pro-only visibility, governed by FEAT-13.SPEC-005).

**Booking History Section:** Below the private note, a heading "Booking history with [client name]" followed by a reverse-chronological list of this client's bookings with this Pro, each row showing: date and time, service name, and outcome status (Confirmed, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled). Each row is display-only on this screen -- it does not navigate elsewhere; booking-level actions belong to FEAT-12 and FEAT-30, not to this feature.

### Responsive Behavior

- **Compact breakpoint:** Single-column stack in the order described above (header, identity header, private note, booking history), full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.
- **Booking history list:** Each row remains a single line (compact) or expands to show slightly more spacing between elements (medium and above) -- no structural change to what information each row shows.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the screen the Pro arrived from (FEAT-12 or FEAT-24) | Screen closes | Standard navigation transition |
| Overflow menu -- "Edit contact" | Tap | Navigate to FEAT-13.SPEC-002 (Client Contact Edit) | Screen transitions | Standard navigation transition |
| Overflow menu -- "Delete client" | Tap | Navigate to FEAT-13.SPEC-003 (Client Deletion Confirmation) | Screen transitions | Standard navigation transition |
| Private note field | Type | Captures text input, up to the 1,000-character limit governed by FEAT-13.SPEC-005 | Save button becomes active; a character count appears once within 100 characters of the limit | Standard input focus state; count shown as "912/1,000" style text |
| Private note field | Type past 1,000 characters | Further input is blocked at the field level | Field stops accepting characters | Caption below the field: "Private note can be up to 1,000 characters." |
| Save button (private note) | Tap | Validates the note via FEAT-13.SPEC-005, then persists the note; logs client_note_added | Button shows a brief loading state | Success: caption changes to "Saved" and fades after a moment. Failure: inline error banner above the field, entered text preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in its loading state |
| Booking history row | Tap | None -- display-only | No state change | None; no navigation or expansion occurs from this row |

### Accessibility Notes

- **Focus order:** Back arrow -> overflow menu -> client identity header (name, phone, email, read-only) -> private note field -> save button -> booking history heading -> each booking history row in date order.
- **Validation and save announcements:** A private-note validation error is announced to assistive technology and programmatically associated with the field; the "Saved" confirmation is announced once when it appears.
- **Keyboard alternatives:** Every action on this screen (navigation, note editing, saving) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Client identity header, private note (or an empty note field with placeholder "No note yet"), and booking history all populated | Screen opens and the client record loads successfully | User navigates away or edits the note |
| Note Editing | Private note field shows in-progress text; Save button active | User types in the private note field | User taps Save, or discards changes by navigating away |
| Saving Note | Save button shows a loading indicator; field remains editable but a second save is debounced | User taps Save on the private note | Save completes (success or failure) |
| Note Save Error | Inline error banner above the private note field; entered text preserved, Save button re-enabled for retry | The note save fails | User taps Save again and it succeeds, or navigates away (text remains unsaved locally only for this session) |
| Empty Booking History | Booking history section shows a plain message: "No bookings with this client yet" instead of a blank list | The client has zero bookings with this Pro (data inconsistency scenario -- FEAT-13.SPEC-001 typically only exists for a client created by a first booking, so this is rare but handled) | N/A -- this state persists until a booking exists |
| Offline/Degraded | Banner "You're offline -- reconnect to view or edit this client's record." at the top; the most recently loaded contact details, note, and booking history remain visible read-only; the private note field is disabled for editing | Connectivity is lost while the screen is open, or the screen is opened without connectivity | Connectivity is restored -- the banner clears and the private note field re-enables |

## Validation Rules

Validation governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules). See that spec for the private note's 1,000-character limit and the private-note field's Pro-only visibility rule. This screen checks the character limit on input (blocking further typing at the limit) and re-validates on save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | The screen the Pro arrived from | FEAT-12 or FEAT-24 |
| "Edit contact" tap | FEAT-13.SPEC-002 (Client Contact Edit) | -- |
| "Delete client" tap | FEAT-13.SPEC-003 (Client Deletion Confirmation) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email, private_note fields; Booking (read-only, per the dependency map's Referenced Entities) -- the derived booking_history list of this client's bookings with this Pro (service, start_time, state).
**Updates:** Client record -- private_note field only (up to 1,000 characters), per FEAT-13.SPEC-005.
**Deletes:** None -- deletion is handled by FEAT-13.SPEC-003 and FEAT-13.SPEC-004, not this screen.

## Business Rules

- Private note validation and the note field's Pro-only visibility are governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules) -- this screen enforces but does not restate those rules.
- Post-deletion concurrent-edit refusal (a save against a deleted client is refused with "This client's record no longer exists. It may have been deleted.") is governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) -- this screen enforces but does not restate that rule.
- Only the private_note field is editable here; name, phone, and email are read-only on this screen and are corrected only through FEAT-13.SPEC-002.
- This screen requires a live connection to load or save (per the Brief's Non-Functional Notes); it does not queue an offline note save for later submission.

## Edge Cases

- **Pro navigates away with an unsaved private note edit** -- No confirmation dialog is shown; the in-progress text is simply discarded, consistent with this being a low-stakes inline field rather than a multi-field form.
- **Pro taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during note save** -- Inline error banner: "Could not save note. Check your connection and try again." with the entered text preserved in the field.
- **Client record was deleted by the Pro from another device or session while this screen is open** -- Per the dependency map's Contention note for the Client entity ("once deleted any concurrent edit is refused with refresh"), a subsequent Save attempt is rejected with the message "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from after acknowledging.
- **Client's phone number is changed via FEAT-13.SPEC-002 while this screen is open in another tab or device** -- Per the dependency map's Contention note for the Client entity ("field edits are last-write-wins"), this screen's identity header reflects the change on next load; an in-progress but unsaved private-note edit on the stale screen is unaffected, since name/phone/email and private_note are independent fields.
- **A concurrent booking is added to this client's history while this screen is open** -- The booking history list is a snapshot as of load; it refreshes on the Pro's next visit to this screen, not live, consistent with this feature's non-live-updating design for a small per-pro client volume.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Pro arrives here by tapping a client's name on a booking row |
| FEAT-24 (Client List Search & Filter) | Navigation (inbound) | Pro arrives here by tapping a client found via search (v1) |
| FEAT-13.SPEC-002 (Client Contact Edit) | Navigation (outbound) | "Edit contact" tap navigates here; returns here on successful save |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | Navigation (outbound) | "Delete client" tap navigates here |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Private note validation and Pro-only visibility rules applied to the note field |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Post-deletion concurrent-edit refusal rule applied to any private-note save attempted after the client record is gone |
| FEAT-19 (Platform Support Read-Only Access) | References (inbound) | Support's equivalent read-only view of this Client entity (excluding private_note) is delivered by that feature's own screen, not this one |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_record_viewed | entry source (dashboard / search), booking_history_count | Screen opens and the client record loads successfully | N/A -- no success-metrics.md metric names this feature directly; this event is the mechanism behind the "Check a client note" moment in Talia's Between-Clients Day journey, which feeds FEAT-12's "Daily Dashboard Glance Speed" metric rather than a metric of this feature's own |
| client_note_added | note_length, was_previously_empty | Private note save completes successfully | N/A -- same rationale as client_record_viewed: this feature contributes to FEAT-12's glance-speed experience but has no success-metrics.md entry of its own |

## Acceptance Criteria

**FEAT-13.SPEC-001-AC-01:** Given Talia taps a client's name on a booking row on her dashboard, when the Client Record Detail screen opens, then she sees the client's name, phone, email, current private note (or an empty field if none exists), and full booking history with her.

**FEAT-13.SPEC-001-AC-02:** Given Talia is on the Client Record Detail screen with an empty private note, when she types "Prefers a lighter volume" and taps Save, then the note is validated by FEAT-13.SPEC-005, persisted, the client_note_added event is logged, and the caption briefly shows "Saved."

**FEAT-13.SPEC-001-AC-03:** Given Talia types a private note approaching 1,000 characters, when she reaches the limit, then further typing is blocked and the caption reads "Private note can be up to 1,000 characters."

**FEAT-13.SPEC-001-AC-04:** Given Talia taps the overflow menu's "Edit contact" option, when the tap registers, then she is navigated to FEAT-13.SPEC-002 (Client Contact Edit).

**FEAT-13.SPEC-001-AC-05:** Given Talia taps the overflow menu's "Delete client" option, when the tap registers, then she is navigated to FEAT-13.SPEC-003 (Client Deletion Confirmation).

**FEAT-13.SPEC-001-AC-06:** Given Talia is on the Client Record Detail screen for a client with three past bookings and one upcoming booking, when she views the booking history section, then all four bookings appear in reverse-chronological order with date, time, service, and outcome status.

**FEAT-13.SPEC-001-AC-07:** Given Talia's note save fails due to a network error, when the failure occurs, then an inline error banner reads "Could not save note. Check your connection and try again." and her entered text remains in the field.

**FEAT-13.SPEC-001-AC-08:** Given Talia loses connectivity while viewing a client's record, when the connection drops, then the banner "You're offline -- reconnect to view or edit this client's record." appears, the previously loaded data remains visible, and the private note field becomes disabled.

**FEAT-13.SPEC-001-AC-09:** Given a client record with zero bookings is somehow reached, when Talia views the booking history section, then it reads "No bookings with this client yet" instead of a blank area.

**FEAT-13.SPEC-001-AC-10:** Given Talia taps Save on the private note twice in rapid succession, when the second tap registers, then it is ignored while the first save is still in progress.

**FEAT-13.SPEC-001-AC-11:** Given the client's record was deleted from another device while Talia has this screen open, when she attempts to save a private note edit, then the save is rejected with "This client's record no longer exists. It may have been deleted." and she is returned to the screen she arrived from.

**FEAT-13.SPEC-001-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen and no client data is exposed.

**FEAT-13.SPEC-001-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly during a support session, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support's equivalent read-only view is only ever reached through FEAT-19's own screen.

**FEAT-13.SPEC-001-AC-14:** Given Talia's session expires while she has unsaved private-note text entered, when she re-authenticates, then the dialog "Your session has expired. Sign in to continue." appeared beforehand and her unsaved note text is restored after sign-in succeeds.

**FEAT-13.SPEC-001-AC-15:** Given an unauthenticated visitor attempts to open this screen's route directly, when the access check runs, then they are redirected to the Pro sign-in screen (FEAT-29) with no client data exposed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loaded, note editing, saving, error, empty history, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |

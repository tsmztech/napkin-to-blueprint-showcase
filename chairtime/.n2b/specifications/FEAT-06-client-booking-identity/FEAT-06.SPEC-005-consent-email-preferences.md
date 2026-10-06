---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-005
spec_name: Consent & Email Preferences
spec_slug: consent-email-preferences
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Consent & Email Preferences

## Overview

**Name:** Consent & Email Preferences
**ID:** FEAT-06.SPEC-005
**Type:** Screen
**Purpose:** Client updates their own texting consent (including re-granting it after opting out) and their email address for this Pro.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Displaying the client's current texting consent state for this Pro
- Re-granting texting consent from a Revoked state
- Editing the client's own email address on file for this Pro
- Being the single client-facing Preferences screen: it hosts the consent section defined by FEAT-14.SPEC-001 (the consent display and re-grant control below are that section)
- A "Message channel" element (SMS or WhatsApp) linking to FEAT-26.SPEC-001, which writes the client's preferred_message_channel

**Non-Goals:**
- Revoking texting consent (STOP reply or opt-out link) -- owned by FEAT-14 (Messaging Consent Management); this screen only offers re-granting, never revocation
- Editing the client's name, phone number, or private notes -- phone is the identity key (read-only to this feature) and name/notes remain Pro-editable only via FEAT-13
- Setting texting preferences for the Pro's own account -- out of scope; this screen is the Client's own settings only, unrelated to FEAT-27 (Pro Profile & Booking Page Settings)
- Writing the channel choice itself -- FEAT-26.SPEC-001 owns the channel selection and writes Client.preferred_message_channel; this screen only shows the current value and links out

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Client taps "Preferences" | None -- current consent and email load fresh |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps "Preferences" | None -- current consent and email load fresh |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | Client taps the back arrow or completes a successful Save there | None -- current consent, channel, and email load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own texting consent state and email for this one Pro | Re-grant texting consent (when Revoked); edit their own email | -- |
| The Pro (Talia) | No | No | The Pro can see but never override a client's texting consent (Access Matrix, Messaging & Consent); that read-only view belongs to FEAT-13's client record screen, not this client-facing screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); this screen has no support entry point |
| Unauthenticated | No | No | Reachable only from FEAT-06.SPEC-003 or FEAT-06.SPEC-004 within an already-authenticated viewing session; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; returning after the underlying access link has transitioned to Used is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Preferences" with a back arrow returning to the screen the client arrived from.

**Body:**
- Texting consent section (this is the FEAT-14.SPEC-001 consent section, hosted here; consent-control behavior is governed by FEAT-14 under XBR-15): current state shown as plain text ("Texting: on" or "Texting: off"); when off, a "Turn texting back on" button is shown
- Message channel section: one row "Message channel" showing the current value of Client.preferred_message_channel ("SMS" or "WhatsApp"; "SMS" when never set), with a chevron that navigates to FEAT-26.SPEC-001
- Email section: a single email input pre-filled with the current email on file (or empty if none), with a "Save" action

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically, full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the screen the client arrived from; if the email input's current text differs from the last-loaded/last-saved email value, a confirmation dialog appears first instead (see Edge Cases) | Screen closes (no unsaved edit), or confirmation dialog opens (unsaved edit present) | Standard navigation transition, or the dialog's copy when shown |
| "Turn texting back on" button (shown only when consent is Revoked) | Tap | Writes the client's texting consent to Re-granted for this Pro; precedence against any concurrent revocation follows FEAT-14's rule (XBR-15) | Button shows a brief loading state, then the consent line updates to "Texting: on" | Confirmation message "Texting turned back on" |
| "Message channel" row | Tap | Navigate to FEAT-26.SPEC-001 (WhatsApp Channel Preference); if the email input differs from the last-saved value, the unsaved-changes dialog appears first, as for the back arrow | Screen transitions (or dialog opens) | Standard navigation transition |
| Email input | Type | Captures the entered text | Field shows entered text | Standard input focus state |
| Email input | Blur (non-empty, malformed) | Triggers field-level format validation | Error state on field | "Enter a valid email address" below the field |
| "Save" button (email) | Tap | Validates the email format, then updates the Client's email field for this Pro | Button shows a brief loading state during save | Success: toast "Preferences saved." Failure: inline error message. |
| Unsaved-changes dialog -- "Discard" | Tap | Discards the unsaved email edit and navigates back to the screen the client arrived from | Dialog closes, screen closes, the typed email value is not persisted | Standard navigation transition |
| Unsaved-changes dialog -- "Keep Editing" | Tap | Dismisses the dialog; the client remains on this screen with the unsaved email text still in the field | Dialog closes, screen remains open | Focus returns to the email input |

### Accessibility Notes

- **Focus order:** Back arrow -> texting consent line and its "Turn texting back on" button (when shown) -> "Message channel" row -> email input -> "Save" button.
- **Validation and confirmation announcements:** The email field's error state and both confirmation messages ("Texting turned back on", "Preferences saved.") are announced to assistive technology as they appear.
- **Unsaved-changes dialog:** The dialog traps focus between "Discard" and "Keep Editing" until dismissed; its text is announced to assistive technology when it opens.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the consent state and email field will appear | Screen first opens | Data finishes loading |
| Populated (Consent On) | "Texting: on" shown, no re-grant button; email field pre-filled | Data loads with active consent | Client navigates away, or consent is revoked elsewhere (FEAT-14) and this screen is reloaded |
| Populated (Consent Off) | "Texting: off" shown with the "Turn texting back on" button; email field pre-filled | Data loads with revoked consent | Client taps "Turn texting back on" and it succeeds |
| Saving | The acted-on control (re-grant button or Save) shows a loading state | Client taps "Turn texting back on" or "Save" | The write completes or fails |
| Error | Error banner "We couldn't load your preferences. Try again." with a retry action, in place of the form | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline. Reconnect to update your preferences." at the top; the form remains visible but the re-grant button and Save are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and the controls re-enable |
| Unsaved-Changes Confirmation | Modal dialog "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options, form inert behind it | Client taps the back arrow while the email input's text differs from the last-loaded/last-saved value | Client taps "Discard" (navigates away, edit not saved) or "Keep Editing" (dialog closes, form remains with the edit intact) |

## Validation Rules

**Option B -- Inline (simple validation, not shared beyond this screen):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Email | Valid email format (when non-empty) | On blur and on Save | "Enter a valid email address" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (no unsaved edit) | The screen the client arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | -- |
| Back arrow tap (unsaved edit, "Discard" confirmed) | The screen the client arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | -- |
| "Message channel" row tap (no unsaved edit, or "Discard" confirmed) | FEAT-26.SPEC-001 (WhatsApp Channel Preference) | FEAT-26 (WhatsApp Reminders) |

## Data Model

**Creates:** None.
**Reads:** Messaging Consent -- channel, state, timestamp for this Client and Pro, matched via FEAT-06.SPEC-008. Client -- email field and preferred_message_channel (SMS | WhatsApp, default SMS), for this Pro.
**Updates:** Messaging Consent -- state transitioned from Revoked to Re-granted (FEAT-06.SPEC-005 is this feature's only write path for consent; precedence rules are owned by FEAT-14). Client -- email field only; no other Client field is writable from this screen. Client.preferred_message_channel is displayed here but written only by FEAT-26.SPEC-001, and it is not part of Messaging Consent (consent stays governed by FEAT-14).
**Deletes:** None.

## Business Rules

- Only the client's own email and their own texting consent state are ever shown or editable here -- name, phone, private notes, and booking notes stay entirely out of this screen's reach.
- This screen is the single client-facing Preferences screen; FEAT-14.SPEC-001 is the consent section hosted inside it, so there is no second Preferences screen to reach from FEAT-06.SPEC-003 or FEAT-06.SPEC-004.
- The Message channel value is stored as Client.preferred_message_channel (SMS | WhatsApp, default SMS), written by FEAT-26.SPEC-001 and kept separate from Messaging Consent.
- XBR-15: no text is sent without active texting consent; a revoke is honored on the very next message; this screen offers only re-granting, never revocation.
- The re-grant written here follows FEAT-14's most-recent-explicit-action rule against any concurrent STOP reply -- the timestamp of whichever explicit action is later wins.
- Client email and consent updates here trace to FEAT-06.SPEC-008 for phone-to-Client matching -- the write always targets the one Client record already scoped by the viewing session.

## Edge Cases

- **Client taps "Turn texting back on" at effectively the same moment a STOP reply arrives for the same phone number (FEAT-14)** -- Per FEAT-14's rule (XBR-15), the most recent explicit action by timestamp wins; if the STOP reply's timestamp is later, the consent state ends as Revoked despite the client's tap succeeding as a write, and the client sees "Texting: off" on their next view of this screen rather than the "on" state their tap requested.
- **Client edits their email while offline, then regains connectivity** -- The Save button is disabled while offline (per the Offline/Degraded state), so no queued write exists; the client must tap Save again once connectivity returns.
- **Client saves an email identical to the one already on file** -- The save proceeds and shows the same "Preferences saved." confirmation; no error is raised for an unchanged value.
- **Client clears the email field entirely and taps Save while texting consent is Off** -- Per product-features.md's Client entity rule, email is required when texts are declined; the Save is rejected with "An email is required while texting is off." and the field remains in edit state.
- **Client double-taps "Turn texting back on"** -- The second tap is ignored while the first write is in flight.
- **Talia edits this same client's contact details in FEAT-13 (e.g., correcting a typo elsewhere on the record) while Riley is saving her own email here** -- Per the dependency map's Contention note for the Client entity, the two writes touch different purposes but the same record; resolution is last-write-wins between the two saves. Riley's email save completes normally and is not rejected by Talia's concurrent edit; if Talia's edit happens to also touch the email field at the same moment, whichever save commits last is the value that persists, and Riley sees her own value reflected only if her save was the later of the two on her next visit to this screen.
- **Client edits the email field and taps the back arrow without tapping Save** -- The edit is never discarded silently: a confirmation dialog appears -- "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options. Tapping "Discard" navigates back to the screen the client arrived from and the typed email value is not persisted (the field reverts to the last-saved value on any later visit). Tapping "Keep Editing" dismisses the dialog and leaves the client on this screen with the typed value still in the field, unsaved. This mirrors FEAT-13.SPEC-002's unsaved-changes pattern for the Pro-side contact edit screen. The texting-consent re-grant control is unaffected by this edge case -- it writes immediately on tap and holds no draft state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | The "Preferences" link there opens this screen |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound) | The "Preferences" link there opens this screen |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes the consent and email shown and written to the matched Client with this Pro |
| FEAT-14.SPEC-001 (Consent & Preferences) | Hosts (outbound) | The consent section on this screen; FEAT-14.SPEC-001 lists this screen in its Entry Points |
| FEAT-14 (Messaging Consent Management) | References (outbound) | Owns consent state precedence, revocation, and the no-text fallback rule this screen's re-grant write is subject to |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | Navigation (outbound/inbound) | The Message channel row opens it; its back arrow and successful Save return here. It writes Client.preferred_message_channel |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| texting_consent_regranted | -- | Client's re-grant write succeeds | supports success-metrics.md: "Self-Service Access Success" |
| client_email_updated | had_previous_value (yes / no) | Client's email save succeeds | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-005-AC-01:** Given Riley's texting consent for Talia is currently Revoked, when she opens this screen, then it shows "Texting: off" with a "Turn texting back on" button.

**FEAT-06.SPEC-005-AC-02:** Given Riley taps "Turn texting back on", when the write succeeds, then the consent line updates to "Texting: on" and she sees "Texting turned back on".

**FEAT-06.SPEC-005-AC-03:** Given Riley's texting consent is already active, when she opens this screen, then it shows "Texting: on" with no re-grant button shown.

**FEAT-06.SPEC-005-AC-04:** Given Riley enters an invalid email format and blurs the field, when validation runs, then the field shows "Enter a valid email address" and Save is not enabled.

**FEAT-06.SPEC-005-AC-05:** Given Riley enters a valid new email and taps Save, when the save completes, then she sees the toast "Preferences saved." and the field reflects the new email on a later visit.

**FEAT-06.SPEC-005-AC-06:** Given Riley's texting consent is Off and she clears the email field entirely, when she taps Save, then the save is rejected with "An email is required while texting is off." and the field stays editable.

**FEAT-06.SPEC-005-AC-07:** Given Riley taps "Turn texting back on" at the same time a STOP reply for her phone number is processed by FEAT-14 with a later timestamp, when precedence is resolved, then her consent state ends as Revoked and shows "Texting: off" on her next view.

**FEAT-06.SPEC-005-AC-08:** Given the initial load of Riley's preferences fails, when the failure occurs, then an error banner "We couldn't load your preferences. Try again." appears with a retry action.

**FEAT-06.SPEC-005-AC-09:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then a banner explains she is offline and both the re-grant button and Save become disabled.

**FEAT-06.SPEC-005-AC-10:** Given Riley taps "Turn texting back on" twice rapidly, when the second tap registers, then it is ignored while the first write is still in progress.

**FEAT-06.SPEC-005-AC-11:** Given Riley has typed a new email value into the field but has not tapped Save, when she taps the back arrow, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-06.SPEC-005-AC-12:** Given the unsaved-changes dialog is shown after Riley's back-arrow tap, when she taps "Discard", then she is returned to the screen she arrived from and her typed email edit is not saved.

**FEAT-06.SPEC-005-AC-13:** Given the unsaved-changes dialog is shown after Riley's back-arrow tap, when she taps "Keep Editing", then the dialog closes, she remains on this screen, and her typed email value is still in the field, unsaved.

**FEAT-06.SPEC-005-AC-14:** Given Riley has never chosen a channel, when she opens this screen, then the Message channel row shows "SMS".

**FEAT-06.SPEC-005-AC-15:** Given Riley taps the Message channel row with no unsaved email edit, when the tap registers, then she is taken to FEAT-26.SPEC-001, and on returning here the row shows the value FEAT-26.SPEC-001 saved.

**FEAT-06.SPEC-005-AC-16:** Given Riley has an unsaved email edit and taps the Message channel row, when the tap registers, then the "You have unsaved changes. Discard?" dialog appears before any navigation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (consent off, saving, error, offline, unsaved-changes confirmation) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |

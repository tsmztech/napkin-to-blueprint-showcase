---
document_type: spec
spec_type: screen
spec_id: FEAT-26.SPEC-001
spec_name: WhatsApp Channel Preference
spec_slug: whatsapp-channel-preference
parent_feature: FEAT-26
parent_feature_name: WhatsApp Reminders
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: WhatsApp Channel Preference

## Overview

**Name:** WhatsApp Channel Preference
**ID:** FEAT-26.SPEC-001
**Type:** Screen
**Purpose:** Riley (the Client) opts for WhatsApp as her delivery channel for confirmations, reminders and change notices from her manage link, or switches back to text.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders

## Scope and Non-Goals

**In Scope:**
- Displaying the client's current message-channel preference for this Pro (text or WhatsApp) where WhatsApp Reminders is live
- Letting the client choose WhatsApp as their channel, or switch back to text
- Writing the client's `preferred_message_channel` (SMS | WhatsApp, default SMS) on the Client record
- Initiating the channel-scoped consent hand-off to FEAT-06/FEAT-14's existing consent-capture mechanism when the client opts in (the consent grant itself is written by that mechanism, never by this screen)

**Non-Goals:**
- Capturing texting consent for the first time -- owned by FEAT-05 (Public Booking Page & Booking Flow), which writes the client's initial Messaging Consent record at booking; this screen only ever changes an already-established client relationship's channel preference.
- Revoking messaging consent outright (a STOP reply or opt-out link) -- owned by FEAT-14 (Messaging Consent Management); this screen offers a channel choice between text and WhatsApp, never a way to turn all messaging off (that remains FEAT-06.SPEC-005 and FEAT-14's territory).
- Writing the Messaging Consent record -- this screen never writes Messaging Consent (SG-14 resolution). The channel preference is stored as `preferred_message_channel` on the Client record and is not part of Messaging Consent; the Messaging Consent entity's Update column names only FEAT-14 and FEAT-06 as writers, and the channel-scoped consent grant is captured by FEAT-06's existing consent-capture mechanism, the same mechanism FEAT-06.SPEC-005 uses to re-grant texting consent.
- Editing the client's email address or texting-only consent state -- unchanged territory of FEAT-06.SPEC-005 (Consent & Email Preferences); this screen adds the WhatsApp choice alongside it, not in place of it.
- Any Pro-facing view of channel or delivery status -- owned by FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-16 (Booking & Payment Activity Record) via FEAT-26.SPEC-002's delivery-status reporting; this screen is the client's own setting only.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Client taps the "Message channel" element (SMS / WhatsApp) hosted in FEAT-06.SPEC-005's Layout and Content and Interactions; the element is shown only while FEAT-26 is live. | None -- current channel preference and consent state load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| The Client (Riley) | Their own message-channel preference for this one Pro | Choose WhatsApp as their channel, or switch back to text | -- |
| The Pro (Talia) | No | No | The Pro sees the resulting delivery channel/status on each message (FEAT-12, via FEAT-16), never a control to view or set the client's channel preference (Access Matrix, Messaging & Consent: "the Pro can see but never override a client's texting consent" -- the same boundary applies to channel choice) |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); this screen has no support entry point |
| Unauthenticated | No | No | Reachable only from FEAT-06.SPEC-005 within an already-authenticated viewing session; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 (Access Link Request) |
| Expired session | No | No | The viewing session lasts only for the current page; returning after the underlying access link has transitioned to Used is treated as unauthenticated and redirected to FEAT-06.SPEC-001 (Access Link Request) with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Message channel" with a back arrow returning to FEAT-06.SPEC-005 (Consent & Email Preferences).

**Body:**
- Explanation line: plain-language text stating that confirmations, reminders and change notices can be sent by text or by WhatsApp, and that WhatsApp is used only when it can be reached -- a message otherwise arrives by text or email as usual.
- Channel choice: two selectable options, "Text" and "WhatsApp," shown as a single-select control with the client's current choice indicated (defaults to "Text" when no WhatsApp preference has ever been set).
- Below the choice, when "WhatsApp" is selected but not yet saved: a short consent line naming what is shared (phone number and booking-related message content) and confirming the choice takes effect on the client's Save.
- A single "Save" action below the choice.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Explanation, channel choice and Save stack vertically, full width.
- **Medium size class and above:** Same vertical order, content column capped at a consistent platform-wide form width and horizontally centered -- no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to FEAT-06.SPEC-005 (Consent & Email Preferences) | Screen closes | Standard navigation transition |
| "Text" option | Tap | Selects Text as the pending channel choice | Text option shows selected state; WhatsApp option deselects | Standard selection state |
| "WhatsApp" option | Tap | Selects WhatsApp as the pending channel choice | WhatsApp option shows selected state; the consent line appears below the choice | Standard selection state; consent line fades in |
| "Save" button | Tap | If WhatsApp is the pending choice: writes Client.preferred_message_channel = WhatsApp, then initiates the channel-scoped consent hand-off to FEAT-06's consent-capture mechanism (which records WhatsApp consent as Granted or Re-granted; this screen does not write Messaging Consent). If Text is the pending choice and WhatsApp was previously active: writes Client.preferred_message_channel = SMS (the default); no consent write is made. | Button shows a brief loading state during the write | Success: toast "Message channel updated." Failure: inline error message below the choice control. |
| "Save" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> "Text" option -> "WhatsApp" option -> consent line (when shown) -> "Save" button.
- **Selection and confirmation announcements:** A change in the selected channel option is announced to assistive technology; the "Message channel updated." toast and any inline error are announced as they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A brief in-place loading indicator where the channel choice will appear | Screen first opens | Data finishes loading |
| Populated (Text active) | "Text" option shown selected; no consent line shown | Data loads with no WhatsApp preference on file, or the client previously chose Text | Client selects WhatsApp |
| Populated (WhatsApp active) | "WhatsApp" option shown selected; no consent line shown (already granted) | Data loads with an active WhatsApp channel preference | Client selects Text |
| Pending Selection | The newly tapped option shows selected; if WhatsApp was just tapped, the consent line appears; Save is enabled | Client taps an option different from the currently saved one | Client taps Save, or navigates away (pending selection discarded) |
| Saving | Save button shows a loading state; both options disabled | Client taps Save | The write completes or fails |
| Error | Error banner "We couldn't load your message channel. Try again." with a retry action, in place of the choice control | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline. Reconnect to update your message channel." at the top; the choice control remains visible but both options and Save are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and the controls re-enable |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Channel eligibility (whether a WhatsApp choice actually results in WhatsApp being used for a given send) is governed by FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule). This screen performs no send-time validation itself -- it captures the client's stated preference only; FEAT-26.SPEC-004 evaluates eligibility fresh on every send.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-06.SPEC-005 (Consent & Email Preferences) | FEAT-06 |
| Successful Save | FEAT-06.SPEC-005 (Consent & Email Preferences) | FEAT-06 |

## Data Model

**Creates:** None.
**Reads:** Client -- `preferred_message_channel`; Messaging Consent -- channel, state, phone_number, for this Client and Pro (read-only, to show the currently effective choice; matched via FEAT-06.SPEC-008's client identity scoping).
**Updates:** Client -- `preferred_message_channel` (SMS | WhatsApp, default SMS). This is the only field this spec writes; it is hosted via FEAT-06.SPEC-005's Message channel element and is not part of Messaging Consent. This spec never writes Messaging Consent: Save also triggers the channel-scoped consent grant, which FEAT-06's existing consent-capture mechanism records (the same write path FEAT-06.SPEC-005 uses to re-grant texting consent), consistent with this feature's Non-Goals on consent-write ownership.
**Deletes:** None.

## Business Rules

- XBR-15 governs the consent discipline this screen's choice feeds into: no message is sent on a channel without active, channel-scoped consent for that client and Pro.
- The screen shows "WhatsApp" as selected only when `preferred_message_channel` is WhatsApp and WhatsApp consent is currently active for the client's current phone number; otherwise it shows "Text" (the stored preference is never altered by a revoke or number change -- only by the client's own Save here).
- Choosing WhatsApp here changes only the client's stated preference (`preferred_message_channel` on the Client record); whether a given send actually goes out over WhatsApp is decided fresh, per send, by FEAT-26.SPEC-004 -- this screen's choice never guarantees WhatsApp delivery, since eligibility also depends on the channel-scoped consent state and the number remaining reachable on WhatsApp.
- The consent line shown when WhatsApp is selected states plainly that the client's phone number and booking-related message content are used to send WhatsApp updates about their appointment, naming no vendor, consistent with FEAT-26.SPEC-002's Consent and Disclosure section.
- A client who changes their phone number has their existing channel preference and consent treated as not applicable to the new number until fresh consent is captured (per the Client entity's Contention note: "a Pro phone-number change invalidates access links and requires fresh texting consent"); this screen shows "Text" as the default in that interim, since FEAT-26.SPEC-004 routes to text/email until fresh consent exists.

## Edge Cases

- **Client selects WhatsApp on two devices at effectively the same time** -- Per the Messaging Consent entity's Contention resolution, the most recent explicit client action by timestamp wins; the device whose write lands second sees its own choice reflected only if its write was the later of the two on its next load of this screen. Resolution: reject-with-refresh is not needed here since both writes are the client's own explicit, idempotent choice -- last-write-wins by timestamp, consistent with the dependency map's Contention note for Messaging Consent.
- **A STOP reply for this client's phone number is processed by FEAT-14 at effectively the same moment the client taps Save here to choose WhatsApp** -- Per FEAT-14's precedence rule (XBR-15), the most recent explicit client action by timestamp wins; if the STOP reply's timestamp is later, the resulting state is no active consent on any channel, and the client sees "Text" shown as the active channel (the no-text/no-WhatsApp default) on their next view of this screen, despite the WhatsApp tap having succeeded as a write.
- **Client selects WhatsApp while offline, then regains connectivity** -- The Save button is disabled while offline (per the Offline/Degraded state), so no queued write exists; the client must tap Save again once connectivity returns.
- **Client taps Save with the same channel already active (no change made)** -- The save proceeds and shows the same "Message channel updated." confirmation; no error is raised for an unchanged value.
- **Client double-taps Save** -- The second tap is ignored while the first write is in flight.
- **Client's phone number changes (via FEAT-13) between this screen's load and Save** -- Per the Client entity's Contention note, the existing consent record is invalidated for the new number; the Save attempt for WhatsApp is not silently accepted -- FEAT-06's underlying consent-capture mechanism reports the number mismatch, and this screen shows the inline error "Your phone number has changed -- update your consent before choosing a message channel," pointing the client back to FEAT-06.SPEC-005.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|--------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Navigation (inbound/outbound); hosts this screen | The "Message channel" element (SMS / WhatsApp) in FEAT-06.SPEC-005 opens this screen and hosts the `preferred_message_channel` setting; this screen's back arrow and successful Save return there. FEAT-06.SPEC-005 also records the channel-scoped consent grant through its consent-capture mechanism. |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes the Messaging Consent record read and written to the matched Client with this Pro |
| FEAT-14 (Messaging Consent Management) | References (outbound) | Owns consent-state precedence and the no-message fallback rule this screen's choice is subject to |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (outbound) | Evaluates, fresh on every send, whether the preference captured here actually results in a WhatsApp send |
| FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | References (outbound) | Its Consent and Disclosure section states the same sharing wording this screen's consent line summarizes |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| whatsapp_channel_selected | previous channel (text / whatsapp / none) | Client's Save write to WhatsApp succeeds | N/A -- no metric in success-metrics.md names WhatsApp Reminders as its Connected Feature; retained per product-features.md's own Signals field for this feature (`whatsapp_channel_selected`) so channel-preference adoption is observable once the feature ships |
| whatsapp_channel_switched_to_text | -- | Client's Save write back to text succeeds | N/A -- same reason: no Stage 2 metric is connected to this feature; retained as the inverse signal to whatsapp_channel_selected |

## Acceptance Criteria

**FEAT-26.SPEC-001-AC-01:** Given Riley has never set a WhatsApp preference, when she opens this screen, then "Text" is shown selected and no consent line is shown.

**FEAT-26.SPEC-001-AC-02:** Given Riley taps "WhatsApp", when the tap registers, then the WhatsApp option shows selected and the consent line naming her phone number and message content appears below it.

**FEAT-26.SPEC-001-AC-03:** Given Riley has selected WhatsApp and taps Save, when the write succeeds, then she sees the toast "Message channel updated." and her choice is reflected as WhatsApp on her next view of this screen.

**FEAT-26.SPEC-001-AC-04:** Given Riley's channel preference is currently WhatsApp, when she selects "Text" and taps Save, then the write succeeds and her preference reverts to text.

**FEAT-26.SPEC-001-AC-05:** Given Talia (the Pro) looks for any way to view or set a client's message channel, when she looks anywhere in her own screens, then no such control exists -- channel choice is Riley's own setting only.

**FEAT-26.SPEC-001-AC-06:** Given the initial load of Riley's channel preference fails, when the failure occurs, then an error banner "We couldn't load your message channel. Try again." appears with a retry action.

**FEAT-26.SPEC-001-AC-07:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then a banner explains she is offline and both channel options and Save become disabled.

**FEAT-26.SPEC-001-AC-08:** Given Riley taps Save twice rapidly, when the second tap registers, then it is ignored while the first write is still in progress.

**FEAT-26.SPEC-001-AC-09:** Given Riley selects WhatsApp on two devices at effectively the same time, when both writes are processed, then the most recent explicit action by timestamp wins, and the earlier device's view shows the later value on its next load.

**FEAT-26.SPEC-001-AC-10:** Given a STOP reply for Riley's phone number is processed by FEAT-14 with a later timestamp than her WhatsApp selection here, when precedence is resolved, then her next view of this screen shows no active channel (the no-message default), despite her WhatsApp tap having succeeded as a write.

**FEAT-26.SPEC-001-AC-11:** Given Riley's phone number has changed since this screen last loaded, when she taps Save to choose WhatsApp, then the write is rejected with "Your phone number has changed -- update your consent before choosing a message channel," pointing her to FEAT-06.SPEC-005.

**FEAT-26.SPEC-001-AC-12:** Given Riley is viewing this screen, when she taps the back arrow, then the screen closes and she returns to FEAT-06.SPEC-005 (Consent & Email Preferences) with no pending selection saved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 4 (pending selection, saving, error, offline) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |

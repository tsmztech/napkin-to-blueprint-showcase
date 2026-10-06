# FEAT-26 — WhatsApp Reminders

This chapter covers WhatsApp Reminders (FEAT-26), a Nice-to-Have-tier feature. It carries 4 specifications carrying 49 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-26.SPEC-001 | WhatsApp Channel Preference | screen | 12 |
| FEAT-26.SPEC-002 | WhatsApp Send & Delivery-Status Capability | integration | 12 |
| FEAT-26.SPEC-003 | WhatsApp Delivery Fallback | automation | 12 |
| FEAT-26.SPEC-004 | WhatsApp Channel Eligibility & Consent Rule | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: WhatsApp Reminders

## Summary

**Feature:** WhatsApp Reminders
**ID:** FEAT-26
**Description:** Confirmations and reminders can optionally be sent over WhatsApp instead of, or alongside, SMS, for clients and pros who prefer it.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** User-Facing
**Rationale:** BRIEF.md's Ecosystem & Integrations states this explicitly: "WhatsApp is a nice-to-have later, not v1." Phased to Later exactly as the brief specifies, and it slots into the existing Automated Booking Messaging (FEAT-08) mechanism as an additional channel rather than a new capability. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Opt for WhatsApp as the delivery channel for confirmations and reminders
- Fall back to SMS or email automatically if WhatsApp delivery is unavailable

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-26.SPEC-001 | WhatsApp Channel Preference | Screen | The Client | Client-facing surface, reached from their manage link, where they opt in to WhatsApp as their delivery channel for confirmations and reminders |
| FEAT-26.SPEC-002 | WhatsApp Send & Delivery-Status Capability | Integration | The Client, The Pro | External capability that sends WhatsApp messages and reports back delivery status (Queued/Sent/Delivered/Failed) for FEAT-08's confirmation, reminder, and change-notice content when the client's channel is WhatsApp |
| FEAT-26.SPEC-003 | WhatsApp Delivery Fallback | Automation | The Client, The Pro, Platform Operator (Support) | On a failed or unavailable WhatsApp send, automatically falls back to SMS or email per the client's existing consent state, and flags the delivery gap the same way FEAT-08 does for a failed text |
| FEAT-26.SPEC-004 | WhatsApp Channel Eligibility & Consent Rule | Logic/Rule | The Client, The Pro | Decides, for each send, whether WhatsApp is used -- checking the client's channel preference against their channel-aware Messaging Consent state -- before handing the send to SPEC-002 or deferring to FEAT-08.SPEC-011's text/email decision |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-26.SPEC-001, FEAT-26.SPEC-004 | SPEC-001 is the surface where the client makes the choice; SPEC-004 is the rule that honors it (or not) on every subsequent send | Phase 2 (Explicit) |
| Fall back to SMS or email automatically if WhatsApp delivery is unavailable | FEAT-26.SPEC-003, FEAT-26.SPEC-004 | SPEC-004 determines up front whether WhatsApp is eligible for a send at all; SPEC-003 handles the case where it was eligible but the send itself failed or the number turned out to be unreachable on WhatsApp | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-26.SPEC-002 | WhatsApp Send & Delivery-Status Capability | Phase 4 (External Dependencies lens) | WhatsApp is a distinct external communications capability from the transactional text/email capabilities FEAT-08.SPEC-012/013 already own -- a separate provider surface with its own opt-in and delivery-status contract, not a mode of the existing two. The context package's External Touchpoints note explicitly asks the Analyst to decide ownership here: this Brief specifies WhatsApp send/delivery-status as this feature's own Integration spec rather than folding it into FEAT-08.SPEC-012/013, because those two specs are scoped by name to text and email; the fallback path this feature uses, however, still routes through FEAT-08.SPEC-012/013 unchanged (see FEAT-26.SPEC-003) |

**Communications field examined, 0 Notification specs (intentional):** The feature's `**Communications:**` field states "This feature is itself an additional communications channel for the messages FEAT-08 already sends" -- it names no message content of its own. Every message this feature could carry over WhatsApp (confirmation, reminder, change notice, Pro alert) is already a Notification spec owned by FEAT-08 (SPEC-001, SPEC-002, SPEC-004, SPEC-005, SPEC-006). This feature changes those specs' delivery channel, not their content, audience, or existence, so it introduces no duplicate Notification specs of its own.

**Journey Step Coverage (Phase 7, Check 3):** No journey in user-journeys.md names WhatsApp Reminders in its Connected Features, and no journey step in the context package requires coverage from this feature -- confirmed against the context package's own statement that this check has zero required rows for this Brief. The two journey steps shown for context (Client's First Booking Step 5; Riley Manages an Existing Booking Step 1) are owned by FEAT-08 and are already covered there; this feature would carry the same content over an additional channel once the client opts in, without altering journey ownership.

## Entity-Lifecycle Coverage Matrix

**Entity: Message** *(WhatsApp-channel scope only -- every other channel's lifecycle is FEAT-08's)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-26.SPEC-002 | Creates one Message record (type, channel=WhatsApp, recipient, content_summary, send time, initial Queued status) whenever SPEC-004 hands it a send | The message's type and content are assembled by the relevant FEAT-08 Notification spec (SPEC-001/002/004/005/006); this spec only performs the WhatsApp-channel dispatch and record creation |
| Read (single) | N/A -- owned by FEAT-16 | Delivery status display is the Booking & Payment Activity Record's responsibility, same as every other channel (Data Notes: "Displayed: delivery channel/status") | Not a gap -- inherits FEAT-08's disposition |
| Read (list) | N/A -- owned by FEAT-12, FEAT-16 | Same reasoning as Read (single) | Not a gap |
| Update | FEAT-26.SPEC-002, FEAT-26.SPEC-003 | delivery_status only; SPEC-002's inbound delivery-status event drives Sent/Delivered/Failed, and a Failed status is what triggers SPEC-003's fallback | -- |
| Delete/Archive | N/A -- immutable once sent | No soft or hard delete applies to a WhatsApp-channel Message any more than any other channel: it is part of the same append-only Message lifecycle FEAT-08 already establishes as a non-goal ("immutable once sent," retained for FEAT-16's activity record) -- no new decision is needed for this channel | No restore/cascade/retention question arises because nothing is ever deleted |
| State Transition | FEAT-26.SPEC-002, FEAT-26.SPEC-003 | Queued -> Sent -> Delivered \| Failed; a Failed WhatsApp send re-enters via SPEC-003's fallback path, which creates a new Message record on the SMS or email channel (through FEAT-08.SPEC-012/013) rather than mutating the failed one | Consistent with FEAT-08's no-mutation-after-failure pattern |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Messaging Consent | FEAT-26.SPEC-004 | Reads the client's channel-aware consent state before every send to decide whether WhatsApp is currently eligible, per this feature's Validation & Limits field ("consent is channel-aware, not a blanket 'texting is fine' assumption") |
| Booking | FEAT-26.SPEC-002 | Reads the same appointment/message content FEAT-08's Notification specs already assembled, to dispatch it over WhatsApp instead of text |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client opts in to WhatsApp as their delivery channel | Capture the channel preference; hand the actual channel-scoped consent grant to FEAT-06/FEAT-14's existing consent-capture mechanism (Messaging Consent's Update column names only FEAT-14 and FEAT-06 -- this feature never writes to that entity) | Cross-feature (delegated write) | FEAT-26.SPEC-001 |
| FEAT-08 is about to compose and send a confirmation, reminder, or change notice to a client whose preference is WhatsApp | Check channel eligibility (preference + channel-scoped consent) | Standalone Logic/Rule | FEAT-26.SPEC-004 |
| SPEC-004 finds WhatsApp eligible | Dispatch the message over WhatsApp and create the Message record | Standalone Integration | FEAT-26.SPEC-002 |
| SPEC-004 finds WhatsApp ineligible (no preference, no consent for that channel, or a changed/unconfirmed number) | Defer to FEAT-08.SPEC-011's existing text/email channel decision | Cross-feature | FEAT-26.SPEC-004 -> FEAT-08.SPEC-011 |
| A WhatsApp send fails, or the delivery-status event reports Failed | Fall back to SMS or email per the client's existing consent state, and flag the delivery gap the way FEAT-08.SPEC-009 already does | Standalone Automation | FEAT-26.SPEC-003 |
| The fallback in SPEC-003 completes | Dispatch through FEAT-08.SPEC-012 (text) or FEAT-08.SPEC-013 (email), unchanged | Cross-feature | FEAT-26.SPEC-003 -> FEAT-08.SPEC-012/013 |

## Shared Context

**Shared Entities:**
- Message -- created and channel-scoped by SPEC-002; updated (delivery_status only) by SPEC-002 and SPEC-003; read externally by FEAT-12 and FEAT-16, same as every other channel. Fields relevant to this feature: channel (WhatsApp value), content_summary, recipient, send time, delivery_status.
- Messaging Consent (read-only) -- read by SPEC-004 for channel, state, and phone_number before every WhatsApp-eligible send; never created or updated by this feature.

**Shared UI Patterns:**
- WhatsApp Channel Preference (SPEC-001) reuses FEAT-06's manage-link surface pattern -- the same entry point, phone-width layout, and plain-language copy style as every other screen a client reaches through their booking-specific link -- so the WhatsApp option appears as an added choice inside an existing pattern rather than a new visual language.

**Shared Validation:**
- SPEC-004 is the sole rule this feature owns; every WhatsApp-eligible dispatch attempt defers to it rather than re-deriving eligibility inline, exactly as FEAT-08.SPEC-011 is the single deferred-to rule for the text/email decision.

## Internal Dependency Map

```
SPEC-001 (WhatsApp Channel Preference) -> [client opts in] -> FEAT-06 / FEAT-14 (channel-scoped consent capture)
FEAT-08 (confirmation / reminder / change-notice composition) -> [client's channel preference is WhatsApp] -> SPEC-004 (WhatsApp Channel Eligibility & Consent Rule)
SPEC-004 -> [eligible] -> SPEC-002 (WhatsApp Send & Delivery-Status Capability)
SPEC-004 -> [not eligible] -> FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule)
SPEC-002 -> [send fails or delivery status reports Failed] -> SPEC-003 (WhatsApp Delivery Fallback)
SPEC-003 -> [fallback dispatch] -> FEAT-08.SPEC-012 (Text) / FEAT-08.SPEC-013 (Email)
```

**Default Entry:** N/A -- this feature has no navigable menu entry point; SPEC-001 is reached only by a client tapping into their booking-specific manage link (FEAT-06), the same disposition FEAT-08 has for its own client-facing screen.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-26.SPEC-001 | Outbound | FEAT-06 (Client Booking Identity) | Hosts the manage-link surface SPEC-001 is reached through, and owns writing the client's re-granted consent state | Client opens their manage link |
| FEAT-26.SPEC-001 / SPEC-004 | Outbound | FEAT-14 (Messaging Consent Management) | Channel-scoped consent discipline applies before WhatsApp is used, and any STOP-style revoke is honored on the very next send | Client opts in, or later revokes |
| FEAT-26.SPEC-002 / SPEC-004 | Inbound | FEAT-08 (Automated Booking Messaging) | This feature dispatches FEAT-08's existing confirmation, reminder, and change-notice content over an additional channel; it does not compose or own that content | Every send to a WhatsApp-preferring client |
| FEAT-26.SPEC-003 | Outbound | FEAT-08 (Automated Booking Messaging, SPEC-012/013) | Fallback lands on FEAT-08's existing text and email Integration specs, unchanged | WhatsApp send fails or is unavailable |
| FEAT-26.SPEC-002 / SPEC-003 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Every WhatsApp send, delivery event, and fallback is written to the append-only activity record, same as every other channel | Every message sent, delivered, failed, or retried |
| FEAT-26.SPEC-003 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A WhatsApp delivery gap is flagged on the Pro's dashboard, same as a failed text | WhatsApp send fails and the fallback is used |

## Non-Functional Notes

**Data volumes / growth:** N/A at MVP -- this is a Later-phase feature with zero volume until built; once live, WhatsApp-channel Message records grow with the same booking-volume pattern FEAT-08 already documents (roughly 20-40 bookings a week per pro), just distributed across an additional channel rather than adding new volume.

**Responsiveness:** When live, a WhatsApp-channel send inherits FEAT-08's existing responsiveness expectations unchanged -- confirmations within about a minute of payment, reminders at their computed time inside the 8am-9pm daytime window (ASMP-29) -- this feature adds no distinct timing expectation of its own.

**Data sensitivity / privacy:** WhatsApp-channel messages carry the same personal data (recipient contact and appointment details) as every other channel and are visible only to the Pro for their own bookings and to Support for delivery-status troubleshooting only (ASMP-23), consistent with FEAT-08's classification of the Message entity.

**Compliance flags:** US SMS-consent discipline extends to WhatsApp per this feature's own Validation & Limits field -- "consent is channel-aware, not a blanket 'texting is fine' assumption" -- so a client's texting consent never silently authorizes WhatsApp use; a separate, explicit channel-scoped grant is required before this feature's rule (SPEC-004) treats WhatsApp as eligible (ASMP-24).

## Non-Goals

- **WhatsApp availability before the Later phase** -- Excluded per scope-boundaries.md's Deferral Note and BRIEF.md's Ecosystem & Integrations: "WhatsApp is a nice-to-have later, not v1." This entire feature is out of scope until the product reaches its Later phase.
- **Marketing or promotional WhatsApp messages** -- Excluded per scope-boundaries.md (SC-15): clients consent to booking-related messages only, and this feature carries nothing beyond the confirmations, reminders, and change notices FEAT-08 already sends -- never a broader message category.
- **New Notification specs for WhatsApp-specific message content** -- Adjacency analysis: this feature's Communications field names no message content of its own; every message it could carry is already specified by FEAT-08's Notification specs (SPEC-001, SPEC-002, SPEC-004, SPEC-005, SPEC-006). Duplicating that content as separate WhatsApp-flavored Notification specs would fork a single source of truth for message wording across channels, which the product definition gives no reason to do.
- **Consent-write ownership by this feature** -- Adjacency analysis, grounded in the dependency map's Messaging Consent entity: its Update column names only FEAT-14 (revoke) and FEAT-06 (client re-grant) as writers. This feature reads Messaging Consent (Connected Entities: "Messaging Consent (read)") but never writes to it -- a client's WhatsApp opt-in is captured through FEAT-06/FEAT-14's existing consent mechanism, not through a parallel write path this feature would otherwise need to own and reconcile.



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



# Integration Spec: WhatsApp Send & Delivery-Status Capability

## Overview

**Name:** WhatsApp Send & Delivery-Status Capability
**ID:** FEAT-26.SPEC-002
**Type:** Integration
**Purpose:** Sends FEAT-08's confirmation, reminder and change-notice content over WhatsApp for clients whose channel is eligible, and reports back each message's delivery status.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders

## Scope and Non-Goals

**In Scope:**
- Sending a WhatsApp message carrying content already composed by FEAT-08's Notification specs (booking confirmation, appointment reminder, booking change & refund notice), once FEAT-26.SPEC-004 has found the send eligible
- Receiving and reporting back delivery status (Queued, Sent, Delivered, Failed) for every WhatsApp message sent
- Reporting when a send is rejected because the recipient's number is not reachable on WhatsApp
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client data is shared with this capability

**Non-Goals:**
- Choosing the WhatsApp vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific vendor.
- Deciding whether a given send is eligible for WhatsApp at all -- owned by FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule); this spec sends whatever it is given once that decision has already been made.
- Composing the confirmation, reminder or change-notice content -- owned by FEAT-08.SPEC-001, FEAT-08.SPEC-002 and FEAT-08.SPEC-004; this spec dispatches that already-composed content over an additional channel.
- Retrying a failed or unavailable WhatsApp send, or falling back to text or email -- owned by FEAT-26.SPEC-003 (WhatsApp Delivery Fallback), which consumes this spec's Failed status and send-rejected event as its own triggers.
- Standard text and email messaging -- owned by FEAT-08.SPEC-012 and FEAT-08.SPEC-013; this spec covers WhatsApp only, and the fallback path FEAT-26.SPEC-003 uses routes back through those two specs unchanged.

## Capability Category

**Category:** Transactional WhatsApp messaging
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies), extended to WhatsApp per BRIEF.md's Ecosystem & Integrations: "WhatsApp is a nice-to-have later, not v1"
**External Touchpoint:** "Transactional WhatsApp messaging -- optional client channel for confirmations, reminders and change notices, from Later" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-26, FEAT-08, FEAT-14)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|-------------------------|------------------------------|
| Riley receives her booking confirmation over WhatsApp instead of text, when she has chosen WhatsApp and it is eligible | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-001 (Booking Confirmation Message) |
| Riley receives her pre-appointment reminder over WhatsApp | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-002 (Appointment Reminder Message) |
| Riley receives a cancellation, reschedule or refund notice over WhatsApp | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-004 (Booking Change & Refund Notice) |
| Talia sees the WhatsApp channel and delivery status on any message sent to a WhatsApp-preferring client, same as any other channel | -- (inherits FEAT-08's existing delivery-status visibility) | FEAT-12 (Pro Daily Schedule Dashboard), FEAT-16 (Booking & Payment Activity Record) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|-----------------|-----------|---------|
| Recipient phone number | Client -- phone | Every WhatsApp send | The capability must know where to deliver the message |
| Message body text | Message -- the composed content for that send (already resolved by the sending Notification spec: FEAT-08.SPEC-001, 002 or 004) | Every WhatsApp send | The capability needs the exact content to transmit |
| Sender identity (the Pro's account, in vendor-neutral terms) | Pro Account -- an account-level sending identity | Every WhatsApp send | Lets the recipient attribute the message consistently to the sending Pro's account |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|----------------|------------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent WhatsApp message | Message -- delivery_status |
| Send-rejected notice (the recipient's number is not reachable on WhatsApp) | The capability determines, at send time, that the number cannot receive a WhatsApp message | Message -- delivery_status set to Failed, with the rejection reason available to FEAT-26.SPEC-003 as its trigger data |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|---------------|----------------|------------------|
| Delivery status: Sent | The capability confirms the WhatsApp message left the sending system | Message.delivery_status set to Sent | None -- an intermediate status, not shown to either party | -- |
| Delivery status: Delivered | The capability confirms the message reached the recipient's WhatsApp | Message.delivery_status set to Delivered | None directly -- delivery success is the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the message could not be delivered | Message.delivery_status set to Failed | Triggers FEAT-26.SPEC-003's fallback to text or email; no direct client feedback (Riley never receives a "delivery failed" message about her own confirmation) | FEAT-26.SPEC-003 |
| Send rejected -- number not WhatsApp-reachable | The capability determines at send time that the recipient's number has no WhatsApp account or cannot receive WhatsApp messages | Message.delivery_status set to Failed; the rejection is distinguished from a delivery failure only in the reason recorded, not in the resulting status | Triggers FEAT-26.SPEC-003's fallback to text or email, identically to a Failed delivery status; no direct client feedback | FEAT-26.SPEC-003 |

## Degradation Behavior

No screen in this feature sends a WhatsApp message synchronously in front of a user -- every send is background/asynchronous to the triggering event (a booking confirmation, a scheduled reminder, or a change notice), the same disposition as FEAT-08.SPEC-012's text sends. The rows below use FEAT-08's Notification specs as the "affected screen" column, since those specs' content is what this capability dispatches; the client-facing experience of a delayed or failed WhatsApp send is entirely governed by FEAT-26.SPEC-003's fallback, not by a loading state on any screen.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|----------------------------|-------------------|--------------------|------------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it, since the confirmation is sent in the background after payment completes | The send attempt is recorded as Failed once a defined timeout is reached; FEAT-26.SPEC-003's fallback to text or email takes over -- no booking-flow screen is blocked | Recorded as Failed (send-rejected, per Inbound Events) and handed to FEAT-26.SPEC-003 |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling as above -- no user-facing screen is affected while the send is slow | Same Failed-then-fallback handling as above | Same Failed-then-fallback handling as above |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling as above | Same Failed-then-fallback handling as above | Same Failed-then-fallback handling as above |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | N/A -- this screen only captures the client's channel preference; it never itself waits on this capability | N/A -- a capability outage never blocks Riley from choosing WhatsApp as her preference; eligibility and delivery are evaluated later, at send time | N/A -- this screen sends no WhatsApp message itself, so a send cannot be rejected here |

## Consent and Disclosure

- **WhatsApp opt-in disclosure at FEAT-26.SPEC-001** -- Before a client's channel preference is saved as WhatsApp, the consent line on FEAT-26.SPEC-001 states plainly that her phone number and booking-related message content are used to send her WhatsApp updates about her appointment, naming no vendor; the write proceeds only once she taps Save with that line visible.
- **What is shared with the capability** -- The recipient's phone number and the already-composed message content for that single send; no vendor is named to the client, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes about the client, payment or card details, and any content beyond the single message being sent never reach this capability. Card data is never held or transmitted by the product at all (SC-11), and this capability has no channel through which it could receive it.
- **Consent remains channel-aware and revocable** -- Every WhatsApp send this capability makes is governed by FEAT-26.SPEC-004's fresh-per-send eligibility check against channel-scoped Messaging Consent; a client who reverts to text or whose consent lapses stops receiving WhatsApp sends on the very next message, per XBR-15.

## Edge Cases

- **A delivery-status event arrives for a Message whose Booking has since been cancelled and archived into history** -- The event is recorded against the Booking's retained history record (bookings are never deleted, per feature-dependency-map.md), and no user feedback fires beyond what FEAT-26.SPEC-003 already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and FEAT-26.SPEC-003's fallback does not re-trigger for an already-resolved Message.
- **Events arrive out of order (a Delivered status arrives before its preceding Sent status)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time; an out-of-order Sent arriving after Delivered does not regress the status.
- **The capability goes down mid-send, with no confirmation either way** -- If no Sent or Failed status is ever received within a defined timeout, the send is treated as Failed for the purpose of triggering FEAT-26.SPEC-003's fallback, so a message is never left in an indefinite unknown state.
- **A send-rejected notice arrives for a number that was WhatsApp-reachable at an earlier send** -- The rejection is honored for this send regardless of past reachability (a client's WhatsApp account may have been deleted or the number reassigned since); FEAT-26.SPEC-003's fallback still applies, and no assumption from a prior successful send carries forward.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation over WhatsApp when FEAT-26.SPEC-004 finds the channel eligible |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder over WhatsApp when eligible |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice over WhatsApp when eligible |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | Triggered by (inbound) | Hands this spec the send once eligibility is confirmed |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | References (inbound) | The Consent and Disclosure wording here matches the consent line shown at opt-in |
| FEAT-26.SPEC-003 (WhatsApp Delivery Fallback) | Triggers (outbound) | A Failed delivery status or a send-rejected event fires this automation |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every WhatsApp send and delivery event is written to the append-only activity record, same as every other channel |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Delivery status for a WhatsApp-channel message is visible to Talia the same way as any other channel |

## Analytics and Success Signals

- **whatsapp_send_attempted** (sending_spec: spec ID) -- N/A -- no metric in success-metrics.md names WhatsApp Reminders as its Connected Feature; retained as the operational baseline behind delivery-status observability, mirroring FEAT-08.SPEC-012's identical baseline event for text.
- **whatsapp_delivery_status_received** (status: sent / delivered / failed) -- N/A -- no Stage 2 metric is connected to this feature; retained because an unobserved WhatsApp delivery gap would otherwise undermine "Reminder Response Rate" (connected to FEAT-08) for clients who opted into WhatsApp, without either metric being able to detect why.
- **whatsapp_send_rejected** (reason: number_not_reachable) -- N/A -- no Stage 2 metric measures WhatsApp reachability specifically; retained so the real-world size of the fallback path (FEAT-26.SPEC-003) is observable rather than assumed.

## Acceptance Criteria

**FEAT-26.SPEC-002-AC-01:** Given Riley has an eligible WhatsApp preference and a confirmation is ready to send, when FEAT-26.SPEC-004 hands the send to this capability, then it sends the message and reports back a delivery status.

**FEAT-26.SPEC-002-AC-02:** Given a WhatsApp message sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered and no further action is taken.

**FEAT-26.SPEC-002-AC-03:** Given a WhatsApp message sent through this capability cannot be delivered, when the Failed status arrives, then FEAT-26.SPEC-003's fallback automation is triggered.

**FEAT-26.SPEC-002-AC-04:** Given Riley's number has no WhatsApp account, when a send to her is attempted, then the capability reports a send-rejected event, the Message is recorded Failed, and FEAT-26.SPEC-003's fallback automation is triggered identically to a delivery failure.

**FEAT-26.SPEC-002-AC-05:** Given the capability is temporarily slow to respond, when a reminder is queued for sending, then no client-facing screen shows a waiting state, since the send is asynchronous to the reminder schedule.

**FEAT-26.SPEC-002-AC-06:** Given the capability is down when a confirmation attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed and handed to FEAT-26.SPEC-003.

**FEAT-26.SPEC-002-AC-07:** Given the same Delivered event for one Message is delivered twice by the capability, when the second event arrives, then nothing changes and no duplicate action fires.

**FEAT-26.SPEC-002-AC-08:** Given a Delivered event arrives before its preceding Sent event for the same Message, when both are processed, then the Message reflects Delivered and the late-arriving Sent event does not regress it.

**FEAT-26.SPEC-002-AC-09:** Given Riley is shown the WhatsApp consent line on FEAT-26.SPEC-001 before her first opt-in, when she reads it, then it states plainly that her phone number and booking message content are used to send her WhatsApp updates, naming no vendor.

**FEAT-26.SPEC-002-AC-10:** Given a delivery-status event arrives for a Message tied to a booking that has since been cancelled and archived, when the event is processed, then it is recorded against the retained history record with no additional user-facing feedback beyond FEAT-26.SPEC-003's governance.

**FEAT-26.SPEC-002-AC-11:** Given Riley's card details are never held by the product, when this capability sends any WhatsApp message, then no payment or card data is ever included in the message content or the data exchanged with the capability.

**FEAT-26.SPEC-002-AC-12:** Given a client's WhatsApp channel is currently ineligible (per FEAT-26.SPEC-004), when a send is about to go out, then this capability is never invoked for that send; the message routes to FEAT-08.SPEC-011's text/email decision instead.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 4 (screens/specs; N/A cells for FEAT-26.SPEC-001 excluded from the count where genuinely inapplicable) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: WhatsApp Delivery Fallback

## Overview

**Name:** WhatsApp Delivery Fallback
**ID:** FEAT-26.SPEC-003
**Type:** Automation
**Purpose:** When a WhatsApp send fails or the recipient's number is unreachable on WhatsApp, automatically falls back to text or email per the client's existing texting consent, and flags the delivery gap the way FEAT-08 already does for a failed text.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders

## Scope and Non-Goals

**In Scope:**
- Falling back to text or email when a WhatsApp send fails or is reported unreachable
- Flagging the resulting delivery gap to the Pro, consistent with FEAT-08.SPEC-009's existing pattern for a failed text

**Non-Goals:**
- Retrying the WhatsApp send itself before falling back -- the Brief's Alternate flow describes an immediate fallback to text or email on WhatsApp failure or unavailability, not a WhatsApp-channel retry; this differs deliberately from FEAT-08.SPEC-009's own single retry-before-fallback pattern for text, since WhatsApp reachability failures (a number with no WhatsApp account) are not the kind of transient failure a retry would resolve.
- Deciding whether text or email is the fallback channel -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule), which this automation defers to exactly as every other client-directed send in the product does; this spec only triggers that decision and the send that follows it.
- Sending the fallback text or email itself, or handling a failure of that fallback send -- owned by FEAT-08.SPEC-012 (Transactional Text Messaging Capability) and FEAT-08.SPEC-013 (Transactional Email Capability); once the fallback becomes a standard text or email send, any further failure of it is governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), not by this spec.
- Composing the alert content shown to the Pro -- owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec only triggers it with the WhatsApp-specific condition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-------------|-------------|------------------|
| A WhatsApp send is reported Failed | FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Fires whenever the WhatsApp capability reports a Failed delivery status for a Message this feature sent | Message (type, recipient, content_summary), Booking or Pro Account reference |
| A WhatsApp send is reported rejected (number not WhatsApp-reachable) | FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Fires whenever the capability reports, at send time, that the recipient's number cannot receive WhatsApp messages | Message (type, recipient, content_summary), Booking or Pro Account reference |

## Processing Logic

1. Receive the Failed or send-rejected event for a WhatsApp Message from FEAT-26.SPEC-002.
2. Defer to FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) to determine the fallback channel: text if the client's texting consent is currently Granted or Re-granted and their phone number matches, otherwise email.
3. Create a new Message record on the resulting fallback channel with the same content (per the Message entity's lifecycle note: a fallback creates a second, immutable Message record rather than mutating the failed WhatsApp one), and send it through FEAT-08.SPEC-012 (text) or FEAT-08.SPEC-013 (email).
4. Trigger FEAT-08.SPEC-006 (Pro Attention Alert) to flag the delivery gap, noting the WhatsApp send that failed or was rejected and which fallback channel was used, referencing the affected Booking or Pro notification.
5. If the fallback send subsequently fails, FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) takes over as the standard retry-then-fallback path for that new text or email Message -- this automation's own responsibility ends once the fallback send has been handed to FEAT-08.SPEC-012 or FEAT-08.SPEC-013.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|----------------|------------------|-------------------|
| Fallback dispatched on text | The client's texting consent is currently active and matches their phone number | A new Message record created (channel: text, delivery_status: Queued/Sent); the original WhatsApp Message's delivery_status remains Failed as its own immutable record | Client receives the message by text; Pro sees a delivery-gap flag (WhatsApp unavailable, fell back to text) | FEAT-08.SPEC-006, FEAT-08.SPEC-011, FEAT-08.SPEC-012 |
| Fallback dispatched on email | The client's texting consent is not currently active, or their phone number does not match | A new Message record created (channel: email, delivery_status: Queued/Sent); the original WhatsApp Message's delivery_status remains Failed | Client receives the message by email; Pro sees a delivery-gap flag (WhatsApp unavailable, fell back to email) | FEAT-08.SPEC-006, FEAT-08.SPEC-011, FEAT-08.SPEC-013 |
| Fallback send itself later fails | The text or email fallback Message subsequently reports Failed | Handled by FEAT-08.SPEC-009's own retry-then-fallback path, not this automation | Talia sees FEAT-08.SPEC-009's escalated attention alert, layered on top of this automation's own flag | FEAT-08.SPEC-009 |
| Automation failure | This automation's own orchestration cannot run (e.g., the fallback dispatch itself is unavailable) | No fallback attempted | The original Failed/rejected WhatsApp status stands; this is indistinguishable from a permanently failed delivery from the Pro's perspective until detected and results in the same escalated flag once resolved | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Message -- type, channel, recipient, content_summary, delivery_status (the failed or rejected WhatsApp record). Messaging Consent -- channel, state, phone_number, read indirectly through FEAT-08.SPEC-011's channel decision.
**Creates:** None directly -- the new fallback Message record is created by FEAT-08.SPEC-012 or FEAT-08.SPEC-013 as part of dispatching the fallback send this automation triggers.
**Updates:** None -- the original WhatsApp Message's delivery_status is left as Failed, set by FEAT-26.SPEC-002; this automation never mutates it, consistent with the Message entity's immutable-per-attempt lifecycle.
**Deletes:** None -- Messages are never deleted (Message entity lifecycle: immutable once sent).

## Business Rules

- XBR-15 governs the fallback channel decision this automation triggers: no text is sent without active texting consent for that client and Pro; otherwise email is used.
- XBR-17's delivery-gap discipline extends to WhatsApp: a failed or unavailable WhatsApp send is never silently dropped -- it always results in a fallback dispatch and a flag on the Pro's dashboard, recorded in the Booking's activity timeline (FEAT-16).
- This automation applies uniformly to every Notification spec whose content can be carried over WhatsApp (FEAT-08.SPEC-001, 002, 004) -- it is not specific to any one message type.
- A fallback creates a new Message record rather than mutating the failed WhatsApp one, preserving each channel attempt as its own immutable record, consistent with the append-only nature FEAT-16 relies on for dispute evidence.
- This automation's own responsibility is exactly one hop: WhatsApp failure to a single fallback dispatch on text or email. It never retries WhatsApp, and it never itself handles a second-level failure of the fallback -- that is FEAT-08.SPEC-009's territory once the fallback becomes a standard text or email send.

## Edge Cases

- **The WhatsApp send fails on a message whose content has since become stale (e.g., the booking was cancelled between the original WhatsApp attempt and this automation's fallback)** -- The fallback still sends the message as originally composed; a subsequent, distinct change notice (FEAT-08.SPEC-004) informs the client of the cancellation separately, since this automation's job is delivering the message it was given, not re-validating its content against the booking's latest state.
- **The WhatsApp capability and the text capability are both down at the same time** -- The WhatsApp Failed event still fires this automation, which attempts the text fallback; that attempt also fails and is handed to FEAT-08.SPEC-009's own retry-then-fallback (to email), so the client is never left without an attempted delivery on some channel, and the Pro's alert is not lost even during a multi-capability outage.
- **The same WhatsApp Message somehow reports Failed twice (a duplicate delivery-status event)** -- The second Failed report for a Message already in a fallback-complete state is a no-op; this automation does not dispatch a second fallback for the same original WhatsApp send.
- **Concurrent trigger firing (two different WhatsApp Messages for the same client fail at the same time)** -- Each Message's fallback runs independently; a WhatsApp confirmation failing and a WhatsApp reminder failing for the same client at the same moment each produce their own fallback dispatch and Pro alert.
- **Trigger fires while a previous run is in flight for the same Message** -- A duplicate Failed or rejected event for a Message whose fallback is already in progress is ignored; only the original triggering event drives the fallback for that Message.
- **A client's channel preference is WhatsApp but their texting consent was revoked before this automation runs** -- Step 2 correctly routes the fallback to email, since FEAT-08.SPEC-011's decision is re-evaluated fresh at fallback time, not assumed from the client's WhatsApp preference.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Triggered by (inbound) | A Failed delivery status or a send-rejected event fires this automation |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (outbound) | Decides text vs. email for the fallback dispatch |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Sends the fallback when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Sends the fallback when email is the chosen channel |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Affects (outbound) | Governs any further failure of the fallback text or email Message this automation creates |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Every WhatsApp fallback used or unresolved failure flags the Pro |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004 | Affects (outbound) | Any WhatsApp-channel Message these specs' content generates is subject to this automation's fallback handling |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The WhatsApp delivery gap appears on the dashboard's attention list |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every WhatsApp failure and fallback event is recorded in the append-only activity record |

## Analytics and Success Signals

- **whatsapp_delivery_failed_fallback_used** (fallback_channel: text / email) -- N/A -- no metric in success-metrics.md names WhatsApp Reminders as its Connected Feature; retained per product-features.md's own Signals field for this feature (`whatsapp_delivery_failed_fallback_used`) so the size and shape of the fallback path is observable once the feature ships, mirroring FEAT-08.SPEC-009's identical reasoning for its own `message_delivery_fallback_used` event.
- **whatsapp_fallback_dispatch_failed** () -- N/A -- no Stage 2 metric measures total WhatsApp-path delivery failure; retained as the operational signal behind XBR-17's "never silently dropped" guarantee extended to this feature's channel.

## Acceptance Criteria

**FEAT-26.SPEC-003-AC-01:** Given a WhatsApp confirmation to Riley is reported Failed, when this automation processes the event, then it defers to FEAT-08.SPEC-011 to decide the fallback channel before dispatching anything.

**FEAT-26.SPEC-003-AC-02:** Given Riley's texting consent is currently active and matches her phone number, when the fallback dispatch runs, then a new Message record is created on the text channel and sent through FEAT-08.SPEC-012.

**FEAT-26.SPEC-003-AC-03:** Given Riley's texting consent is not currently active, when the fallback dispatch runs, then a new Message record is created on the email channel and sent through FEAT-08.SPEC-013.

**FEAT-26.SPEC-003-AC-04:** Given a WhatsApp send to Riley is reported rejected because her number has no WhatsApp account, when this automation processes the event, then it falls back identically to a Failed delivery status.

**FEAT-26.SPEC-003-AC-05:** Given the fallback dispatch (text or email) succeeds, when delivery completes, then Talia still receives a delivery-gap alert (FEAT-08.SPEC-006) noting WhatsApp was unavailable and which channel the fallback used, even though Riley did receive the message.

**FEAT-26.SPEC-003-AC-06:** Given the WhatsApp send fails and the resulting fallback send also later fails, when the fallback failure is confirmed, then FEAT-08.SPEC-009's own retry-then-fallback path governs that failure, not this automation.

**FEAT-26.SPEC-003-AC-07:** Given the same WhatsApp Message reports a Failed status twice, when the second report arrives, then it is treated as a no-op and no second fallback is dispatched.

**FEAT-26.SPEC-003-AC-08:** Given the booking a failed WhatsApp message concerns is cancelled between the original failure and this automation's fallback, when the fallback sends, then it still delivers the originally composed content, and the cancellation is communicated separately via FEAT-08.SPEC-004.

**FEAT-26.SPEC-003-AC-09:** Given both the WhatsApp and text capabilities are unavailable at the same time, when the WhatsApp failure fires this automation, then the text fallback attempt also fails and is handed to FEAT-08.SPEC-009's own retry-then-fallback to email, so the client is not left without an attempted delivery.

**FEAT-26.SPEC-003-AC-10:** Given two different WhatsApp Messages for the same client fail at effectively the same time, when both are processed, then each falls back independently with its own Pro alert.

**FEAT-26.SPEC-003-AC-11:** Given a fallback text Message is created after a failed WhatsApp send, when the original WhatsApp Message record is inspected later, then it still shows delivery_status Failed as its own immutable record, distinct from the new text Message.

**FEAT-26.SPEC-003-AC-12:** Given Support views the activity record for a booking whose message fell back from WhatsApp, when Support inspects the record, then both the failed WhatsApp attempt and the successful fallback appear as separate entries, per FEAT-16's append-only record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (WhatsApp failed, WhatsApp send-rejected) | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: WhatsApp Channel Eligibility & Consent Rule

## Overview

**Name:** WhatsApp Channel Eligibility & Consent Rule
**ID:** FEAT-26.SPEC-004
**Type:** Logic/Rule
**Purpose:** Decides, for every outbound confirmation, reminder or change notice, whether WhatsApp is used -- checking the client's channel preference against their channel-aware Messaging Consent state -- before handing the send to FEAT-26.SPEC-002 or deferring to FEAT-08.SPEC-011's text/email decision.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The channel-eligibility decision (WhatsApp vs. defer to text/email) applied before every client-directed send this feature could carry
- Re-checking eligibility at send time so a same-session preference change or consent revoke is honored immediately
- The fresh-consent requirement after a client's phone number changes, extended from FEAT-08.SPEC-011's identical rule to the WhatsApp channel

**Non-Goals:**
- Capturing or writing the client's channel preference or channel-scoped consent -- the preference (`preferred_message_channel` on the Client record, SMS | WhatsApp, default SMS) is written by FEAT-26.SPEC-001 (WhatsApp Channel Preference), which hands the channel-scoped consent grant to FEAT-06/FEAT-14's existing consent-capture mechanism; this spec only reads the resulting state and never writes either.
- Deciding text vs. email once WhatsApp has been found ineligible -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule), which this spec defers to as its own next step, not a rule this spec re-derives.
- Sending the WhatsApp message once eligibility is confirmed -- owned by FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability); this spec only decides whether to hand the send to it.
- Handling a WhatsApp send that fails or is rejected after this spec found it eligible -- owned by FEAT-26.SPEC-003 (WhatsApp Delivery Fallback); this spec's decision is made once, before the send is attempted, and does not re-evaluate after a failure.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp from Later) |
| state | enum | Granted / Revoked / Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-08.SPEC-001 | Booking Confirmation Message | Channel decision evaluated immediately before send, ahead of FEAT-08.SPEC-011's own text/email decision |
| FEAT-08.SPEC-002 | Appointment Reminder Message | Channel decision evaluated immediately before send, ahead of FEAT-08.SPEC-011's own text/email decision |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | Channel decision evaluated immediately before send, ahead of FEAT-08.SPEC-011's own text/email decision |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Receives the send whenever this spec finds WhatsApp ineligible; this spec always runs before it (SG-13) |
| FEAT-26.SPEC-002 | WhatsApp Send & Delivery-Status Capability | Never invoked for a send this spec has not first found eligible |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-----------------------------|----------------|------------------|-----------|
| channel | Must be one of the product's defined channels (text; WhatsApp) | Always | On read, before the eligibility decision | N/A -- this field is written by FEAT-06/FEAT-14 via FEAT-26.SPEC-001's consent hand-off, not entered directly in this spec's flow; the separate Client.preferred_message_channel (SMS | WhatsApp) is written by FEAT-26.SPEC-001 and read here | No |
| state | Must be one of Granted / Revoked / Re-granted | Always | On read, before the eligibility decision | N/A -- written by FEAT-05/FEAT-06/FEAT-14 | No |
| timestamp | No validation beyond data type -- system-set, not user-entered in this spec's flow | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- captured verbatim by FEAT-06/FEAT-26.SPEC-001 as evidence | Always | -- | -- | -- |
| phone_number | Must match the Client's current phone number for the WhatsApp channel to be treated as eligible | Always | On every send, before selecting WhatsApp as the channel | N/A -- a mismatch is a silent routing decision (defer to FEAT-08.SPEC-011), not a user-facing validation error, since no user is filling out a form at this point | Yes (blocks WhatsApp; does not block the send itself, which proceeds through FEAT-08.SPEC-011) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|------------------|-------|-----------------|
| WhatsApp eligibility gate | channel, state, phone_number | WhatsApp is used only when the Client's `preferred_message_channel` is WhatsApp AND the Messaging Consent channel is WhatsApp AND state is Granted or Re-granted AND phone_number matches the Client's current phone_number; otherwise defer to FEAT-08.SPEC-011's text/email decision | N/A -- this is a routing decision with no user-facing error; the client simply receives the message on the resulting channel |
| Stale-state resolution | state, timestamp | When two consent-state changes could apply (e.g., a STOP reply and an in-app channel change arriving close together, per the Messaging Consent entity's Contention note), the most recent explicit client action by timestamp wins | N/A -- resolved automatically; if the state is genuinely uncertain, WhatsApp is treated as ineligible and the send defers to FEAT-08.SPEC-011 |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Read Messaging Consent state (to decide a send's WhatsApp eligibility) | The Pro (Talia) | Read-only, own clients' consent only, for the Pro's own visibility of delivery channel (per FEAT-12) | -- |
| Read Messaging Consent state (to decide a send's WhatsApp eligibility) | The Client (Riley) | This spec itself does not surface eligibility state to the Client directly; the Client's own view/change of their channel preference is FEAT-26.SPEC-001's screen, not this rule | -- |
| Read Messaging Consent state (to decide a send's WhatsApp eligibility) | Platform Operator (Support) | View-only, for troubleshooting a delivery issue -- Support never changes consent or channel preference | -- |
| Change channel preference or consent state | The Client (Riley) | Own-only -- only the Client whose consent it is may change it (Access Matrix: Messaging & Consent = Own-only for the Client); performed via FEAT-26.SPEC-001, FEAT-06 or FEAT-14, never through this spec | Control not exposed anywhere in this spec's flow -- consent or channel changes never happen as a side effect of a message send |
| Change channel preference or consent state | The Pro (Talia) | Never -- the Pro can see but never override a client's texting consent, and the same boundary applies to a client's channel choice (Access Matrix note) | The delivery-channel field is read-only wherever the Pro views it (FEAT-12); no control to change it is ever shown to the Pro |
| Change channel preference or consent state | Platform Operator (Support) | Never | No control exists for Support to change consent or channel preference under any circumstance |
| Override this spec's eligibility decision for a single send | The Pro (Talia) | Never -- the Pro cannot force a WhatsApp send against an ineligible client, or force text/email against an eligible WhatsApp-preferring one, even for her own client | No override control exists anywhere in the product; the channel decision is fully automatic |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|----------------------|----------------|------------------------|
| WhatsApp eligibility for a given send | Derived: eligible if the Client's `preferred_message_channel` is WhatsApp AND Messaging Consent.channel is WhatsApp AND state is Granted/Re-granted AND phone_number matches the Client's current phone_number; ineligible (defer to FEAT-08.SPEC-011) otherwise | On every client-directed send this feature could carry, evaluated fresh each time (never cached from a prior send) | No -- this is a system-computed routing decision with no user-facing override |

## Business Rules

- XBR-15 governs this spec's eligibility decision the same way it governs FEAT-08.SPEC-011's text/email decision: no message is sent on a channel without active, channel-scoped consent for that client and Pro; a revoke or channel switch is honored on the very next message; a changed phone number requires fresh consent.
- The eligibility decision is re-evaluated at send time, not cached from the moment the client set their WhatsApp preference (FEAT-26.SPEC-001) or from a prior message -- this is the mechanism by which a preference change or a revoke is honored on the very next message, since there is no message in flight that can still route to WhatsApp after ineligibility takes effect.
- A client who changes their phone number (per feature-dependency-map.md's Client Contention note: "a Pro phone-number change invalidates access links and requires fresh texting consent") has their existing Messaging Consent, on any channel, treated as not applicable to the new number until fresh consent is captured for it; every send in the interim defers to FEAT-08.SPEC-011, which itself routes to email under the same phone-mismatch condition.
- This spec runs strictly before FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule); FEAT-08.SPEC-011 notes this ordering (SG-13) and remains the sole text/email decision. Also: the channel preference is read from the Client record's `preferred_message_channel`, not from Messaging Consent.
- This spec runs strictly before FEAT-08.SPEC-011 in the channel-decision order for every send this feature could carry: WhatsApp eligibility is checked first, and only an ineligible result falls through to FEAT-08.SPEC-011's binary text/email decision -- the two rules are never evaluated independently of each other for the same send.
- Every Notification spec whose content can be carried over WhatsApp (FEAT-08.SPEC-001, 002, 004) defers to this spec's decision rather than implementing its own WhatsApp eligibility logic, per this feature's Shared Validation section ("SPEC-004 is the sole rule this feature owns").

## Edge Cases

- **A STOP reply and an in-app WhatsApp opt-in for the same client arrive within the same second** -- Per the Messaging Consent entity's Contention resolution, the most recent explicit client action by timestamp wins; if the timestamps are genuinely indistinguishable, WhatsApp is treated as ineligible and the send defers to FEAT-08.SPEC-011, since an uncertain consent state must never risk an unwanted send on any channel.
- **The client's phone number changes mid-session while a message is queued to send** -- The eligibility decision, evaluated at send time (not queue time), correctly defers to FEAT-08.SPEC-011 once the number-mismatch condition is detected, even if the message was queued while the old number was still valid.
- **A client has WhatsApp consent Granted but no phone number on file (a data inconsistency that should not occur given FEAT-26.SPEC-001's capture flow)** -- The phone_number match condition cannot be satisfied, so WhatsApp is ineligible and the send defers to FEAT-08.SPEC-011, which is never blocked outright since email is always available as the client provided one at booking.
- **A client revokes consent, and a message that was already in the middle of a WhatsApp-send attempt when the revoke landed** -- The already-initiated send completes on the channel it started on (this spec governs the decision at the moment of initiating a send, not a mid-flight cancellation of an attempt already underway); the very next message after the revoke is what is guaranteed to honor the new state.
- **A client switches their channel preference from WhatsApp back to text (FEAT-26.SPEC-001) between two scheduled reminders** -- The next reminder evaluates this spec fresh and finds WhatsApp ineligible (channel no longer WhatsApp), deferring immediately to FEAT-08.SPEC-011; no message in flight is affected because the decision is never cached.
- **Consent state is Re-granted after a prior Revoked state, on the WhatsApp channel** -- Re-granted is treated identically to Granted for this spec's eligibility decision -- WhatsApp becomes eligible again immediately, consistent with the entity's three-state model treating Re-granted as an active-consent state, not a distinct tier.

## Acceptance Criteria

**FEAT-26.SPEC-004-AC-01:** Given Riley has chosen WhatsApp as her channel, has active Messaging Consent (Granted) on the WhatsApp channel, and her phone number on file matches her consent record, when any client-directed message in FEAT-08 is about to send, then WhatsApp is selected and the send is handed to FEAT-26.SPEC-002.

**FEAT-26.SPEC-004-AC-02:** Given Riley has never set a WhatsApp preference, when any client-directed message is about to send, then this spec finds WhatsApp ineligible and defers to FEAT-08.SPEC-011's text/email decision.

**FEAT-26.SPEC-004-AC-03:** Given Riley has chosen WhatsApp but her consent on that channel is Revoked, when a message is about to send, then WhatsApp is ineligible and the send defers to FEAT-08.SPEC-011.

**FEAT-26.SPEC-004-AC-04:** Given Riley switches her preference from WhatsApp to text between her confirmation (sent by WhatsApp) and her reminder, when the reminder is about to send, then it defers to FEAT-08.SPEC-011, honoring the switch on the very next message.

**FEAT-26.SPEC-004-AC-05:** Given Riley's phone number changes and fresh consent has not yet been captured for the new number, when a message is about to send, then WhatsApp is ineligible and the send defers to FEAT-08.SPEC-011, never routing to the old or unconsented number.

**FEAT-26.SPEC-004-AC-06:** Given Riley's WhatsApp consent state transitions from Revoked to Re-granted, when the next message is about to send, then WhatsApp is selected as eligible, since Re-granted is treated as active consent.

**FEAT-26.SPEC-004-AC-07:** Given a STOP reply and an in-app WhatsApp opt-in for the same client arrive within the same second with indistinguishable timestamps, when the eligibility decision is evaluated, then WhatsApp is treated as ineligible and the send defers to FEAT-08.SPEC-011.

**FEAT-26.SPEC-004-AC-08:** Given Talia (the Pro) views a client's delivery channel on her dashboard, when she looks for a way to override this spec's decision, then no such control exists anywhere in the product.

**FEAT-26.SPEC-004-AC-09:** Given Support is troubleshooting a delivery issue, when Support views the Messaging Consent record, then Support sees the channel and state read-only and has no control to change them.

**FEAT-26.SPEC-004-AC-10:** Given Riley has WhatsApp consent Granted but no phone number on file due to a data inconsistency, when a message is about to send, then this spec finds WhatsApp ineligible and the send defers to FEAT-08.SPEC-011 rather than being blocked outright.

**FEAT-26.SPEC-004-AC-11:** Given every Notification spec whose content can be carried over WhatsApp (FEAT-08.SPEC-001, 002, 004) needs a channel decision, when each composes its send, then each defers to this spec first rather than implementing separate WhatsApp eligibility logic.

**FEAT-26.SPEC-004-AC-12:** Given a WhatsApp send to Riley has already begun processing at the moment her consent revoke is recorded, when that specific send completes, then it completes on the channel it started on; the very next message after the revoke is the one guaranteed to honor the new ineligible state.

**FEAT-26.SPEC-004-AC-13:** Given Riley (the Client) wants to change her own channel preference or consent, when she looks for how to do so, then the control exists only in FEAT-26.SPEC-001, FEAT-06 or FEAT-14, never as a side effect of viewing or receiving a message under this spec.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


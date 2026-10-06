# FEAT-14 — Messaging Consent Management

This chapter covers Messaging Consent Management (FEAT-14), a Important-tier feature. It carries 9 specifications carrying 86 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-14.SPEC-001 | Consent & Preferences | screen | 9 |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | screen | 8 |
| FEAT-14.SPEC-003 | Consent Capture at Booking | automation | 10 |
| FEAT-14.SPEC-004 | Opt-Out / STOP Processing | automation | 11 |
| FEAT-14.SPEC-005 | Consent Re-Grant Action | automation | 8 |
| FEAT-14.SPEC-006 | Concurrent Consent Update Resolution | logic-rule | 11 |
| FEAT-14.SPEC-007 | Textability Determination Rule | logic-rule | 10 |
| FEAT-14.SPEC-008 | Phone Number Change Consent Invalidation Rule | logic-rule | 9 |
| FEAT-14.SPEC-009 | Opt-Out Confirmation Message | notification | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Messaging Consent Management

## Summary

**Feature:** Messaging Consent Management
**ID:** FEAT-14
**Description:** The system that captures a client's explicit opt-in to text messaging at booking, lets a client withdraw consent at any time, and ensures every reminder and confirmation respects the current consent state.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Constraints state plainly: "clients must explicitly agree to receive texts when they book, and reminders must respect that consent," citing US texting rules as the reason. Ranked Important rather than Core because it is a compliance-and-preference layer underneath the Core messaging feature (FEAT-08) rather than a capability a client seeks out on its own; it must still ship in MVP because it is a regulatory precondition for FEAT-08 to operate lawfully.

**Key Capabilities:**
- Capture explicit opt-in at the moment of booking (never pre-checked)
- Let a client withdraw consent at any time via a link included in messages
- Fall back to email automatically for any client without active texting consent
- Let a client opt back in to texts later from their own booking view (FEAT-06) or on their next booking

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-14.SPEC-001 | Consent & Preferences | Screen | The Client | The consent section hosted inside FEAT-06.SPEC-005 (the single client-facing Preferences screen): current texting consent status, with an action to opt back in to texting |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | Screen | The Client | The page a client lands on after tapping an opt-out link in a message, confirming texting has been turned off |
| FEAT-14.SPEC-003 | Consent Capture at Booking | Automation | The Client, The Pro | Records the client's opt-in/opt-out choice at booking (or a later booking) as a Messaging Consent record with state, timestamp, channel, and the exact consent wording shown |
| FEAT-14.SPEC-004 | Opt-Out / STOP Processing | Automation | The Client, The Pro | Revokes texting consent immediately, with no grace period, on a tapped opt-out link or an inbound "STOP" reply |
| FEAT-14.SPEC-005 | Consent Re-Grant Action | Automation | The Client | Records a client's opt-back-in to texting from the Consent & Preferences section (hosted in FEAT-06.SPEC-005) |
| FEAT-14.SPEC-006 | Concurrent Consent Update Resolution | Logic/Rule | The Client, The Pro | Resolves a STOP reply and an in-app re-grant arriving close together by most-recent-action timestamp, defaulting to the no-text state when uncertain |
| FEAT-14.SPEC-007 | Textability Determination Rule | Logic/Rule | The Client, The Pro, Platform Operator (Support) | Computes whether a client is currently textable, consumed by FEAT-08's channel selection and by the Pro's planning view |
| FEAT-14.SPEC-008 | Phone Number Change Consent Invalidation Rule | Logic/Rule | The Client, The Pro | Invalidates existing texting consent whenever a client's phone number changes, requiring fresh consent before texting the new number |
| FEAT-14.SPEC-009 | Opt-Out Confirmation Message | Notification | The Client, Platform Operator (Support) | Sends a confirmation message after a STOP reply is processed, acknowledging texting has been turned off and that future messages will arrive by email |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Capture explicit opt-in at the moment of booking (never pre-checked) | FEAT-14.SPEC-003 | Records the opt-in choice, timestamp, channel, and exact consent wording shown the moment a booking is submitted; the never-pre-checked default itself lives in FEAT-05's booking screen | Phase 2 (Explicit) |
| Let a client withdraw consent at any time via a link included in messages | FEAT-14.SPEC-002, FEAT-14.SPEC-004, FEAT-14.SPEC-009 | SPEC-004 revokes consent the instant a link is tapped or a STOP reply arrives; SPEC-002 is the landing confirmation for the link path; SPEC-009 is the confirmation message for the text-reply path | Phase 2 (Explicit) |
| Fall back to email automatically for any client without active texting consent | FEAT-14.SPEC-007 | Determines current textability that FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) reads before every client-directed send to choose text or email | Phase 2 (Explicit) |
| Let a client opt back in to texts later from their own booking view (FEAT-06) or on their next booking | FEAT-14.SPEC-001, FEAT-14.SPEC-005, FEAT-14.SPEC-003 | SPEC-001 is the consent section hosted inside FEAT-06.SPEC-005 (the Preferences screen, reached from FEAT-06.SPEC-003/004); SPEC-005 records the re-grant made there; SPEC-003 records a re-grant made at a subsequent booking | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-14.SPEC-006 | Concurrent Consent Update Resolution | Phase 5 (Rule-Constraint Discovery) + Phase 6 (Negative/Failure) | The dependency map's Contention line for Messaging Consent describes a race condition (a STOP reply and an in-app re-grant arriving close together) with an explicit conflict-resolution rule and a named fail-safe Error state (defaults to no-text) -- conditional logic shared across SPEC-004 and SPEC-005 that crosses the standalone-Logic/Rule threshold |
| FEAT-14.SPEC-008 | Phone Number Change Consent Invalidation Rule | Phase 5 (Rule-Constraint Discovery) | XBR-15, which this feature owns ("a changed phone number requires fresh consent"), and the Client entity's Contention line ("a Pro phone-number change invalidates access links and requires fresh texting consent") describe a conditional rule with no home in any capability-mapped spec |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | Phase 6 (Negative/Failure) applied to SPEC-004's tap outcome | A link tap needs somewhere to land and a defined Error/Permission-Denied state (an already-used, expired, or foreign booking's link) -- a genuine screen, not a bare toast, mirroring the pattern other messaging-link taps in this product require |

## Entity-Lifecycle Coverage Matrix

**Entity: Messaging Consent**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-14.SPEC-003 | Creates the record at a client's first booking with a given Pro: state, timestamp, channel, exact consent wording shown, and phone_number | Triggered by FEAT-05's booking submission, per the dependency map's "Created by FEAT-05"; this feature owns the write logic and the record's shape |
| Read (single) | FEAT-14.SPEC-001 | The client's own consent status is displayed in the Consent & Preferences section hosted in FEAT-06.SPEC-005 | The Pro's read of the same state (for planning, View-only) is external, owned by FEAT-12's dashboard; Support's read is likewise external, owned by its own troubleshooting view -- not a gap, confirmed against the dependency map's "Read by FEAT-06, FEAT-08, FEAT-12, FEAT-20, FEAT-26, FEAT-30" |
| Read (list) | N/A | No list of Messaging Consent records exists anywhere in the product -- exactly one record per client-Pro relationship, surfaced only as a single current status, never as a browsable list | -- |
| Update | FEAT-14.SPEC-003, FEAT-14.SPEC-004, FEAT-14.SPEC-005 | SPEC-003 updates the record on a subsequent booking if consent state changes; SPEC-004 updates it to Revoked; SPEC-005 updates it to Re-granted | State ordering when updates race is resolved by SPEC-006 |
| Delete/Archive | N/A -- owned by FEAT-13 | Deleted only as part of a full client deletion (FEAT-13, per XBR-19); this feature never deletes a Messaging Consent record on its own. Hard delete of the record's identifying details, no restore path, no cascade beyond the Client deletion that triggers it; evidence required by US SMS-consent law is retained in de-identified form per SC-22 -- an explicit, sourced design decision, not an omission | -- |
| State Transition | FEAT-14.SPEC-004, FEAT-14.SPEC-005, FEAT-14.SPEC-006 | Granted -> Revoked -> Re-granted; SPEC-006 governs which transition wins when two updates arrive close together, defaulting to the no-text state when the outcome is uncertain | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-14.SPEC-003, FEAT-14.SPEC-001, FEAT-14.SPEC-008 | Reads which Client-Pro relationship a consent record belongs to, and the client's current phone_number to detect a change that requires fresh consent |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client submits a booking with an opt-in choice (FEAT-05) | Create (or update, on a later booking) the Messaging Consent record with state, timestamp, channel, and exact wording shown | Standalone Automation | FEAT-14.SPEC-003 |
| Client taps an opt-out link in a message (FEAT-08) | Revoke consent immediately and show a landing confirmation | Standalone Automation + Screen | FEAT-14.SPEC-004 / FEAT-14.SPEC-002 |
| Client replies "STOP" to a text (inbound event on FEAT-08's text capability) | Revoke consent immediately and send back a confirmation message | Standalone Automation + Notification | FEAT-14.SPEC-004 / FEAT-14.SPEC-009 |
| Client taps "opt back in" in the Consent & Preferences section (hosted in FEAT-06.SPEC-005) | Record the re-grant | Standalone Automation | FEAT-14.SPEC-005 |
| A STOP reply and an in-app re-grant arrive close together | Resolve to the most recent explicit client action by timestamp; default to the no-text state if the outcome is uncertain | Standalone Logic/Rule | FEAT-14.SPEC-006 |
| Any feature is about to send a client-directed message | Determine whether the client is currently textable so the sender can choose text or email | Standalone Logic/Rule | FEAT-14.SPEC-007 |
| A client's phone number changes (FEAT-13) | Invalidate existing texting consent for that client-Pro relationship, requiring a fresh opt-in before texting the new number | Standalone Logic/Rule | FEAT-14.SPEC-008 |
| A client's record is deleted (FEAT-13) | Remove the consent record's identifying details; retain de-identified evidence where law requires | Cross-feature -- owned by FEAT-13 (XBR-19) | FEAT-13 responsibility |

## Shared Context

**Shared Entities:**
- Messaging Consent -- created and updated by SPEC-003, SPEC-004, SPEC-005; state-transition ordering governed by SPEC-006; read (single, client-facing) by SPEC-001; read (rule output) by SPEC-007. Fields: channel, state (Granted | Revoked | Re-granted), timestamp, exact consent wording shown, phone_number.
- Client (read-only) -- SPEC-003, SPEC-001, and SPEC-008 all read the Client record to identify the owning client-Pro relationship and the current phone_number.

**Shared UI Patterns:**
- Plain-language consent status wording -- SPEC-001's client-facing status label and SPEC-002's landing confirmation both describe the same two states (texting on / texting off, falling back to email) in identical wording, so a client is never shown two different descriptions of the same underlying state.

**Shared Validation:**
- FEAT-14.SPEC-006 owns the conflict-resolution and fail-safe-default rule that SPEC-004 and SPEC-005 both defer to whenever their writes could race.
- FEAT-14.SPEC-007 is the single source of truth for "is this client currently textable" that every other spec and feature (FEAT-08, FEAT-12) reads rather than re-deriving the state itself.

## Internal Dependency Map

```
FEAT-05 (Public Booking Page & Booking Flow) -> [client submits booking with opt-in choice] -> SPEC-003 (Consent Capture at Booking)
SPEC-003 (Consent Capture at Booking) -> [client re-books with a changed choice] -> SPEC-003 (updates the existing record)
FEAT-08 (Automated Booking Messaging) -> [client taps an opt-out link in a message] -> SPEC-002 (Opt-Out Link Landing) -> SPEC-004 (Opt-Out / STOP Processing)
FEAT-08 (Automated Booking Messaging, inbound text event) -> [client replies STOP] -> SPEC-004 (Opt-Out / STOP Processing) -> SPEC-009 (Opt-Out Confirmation Message)
FEAT-06.SPEC-003 / FEAT-06.SPEC-004 (my bookings) -> [client opens Preferences] -> FEAT-06.SPEC-005 (Consent & Email Preferences, host screen) -> [renders consent section] -> SPEC-001 (Consent & Preferences)
SPEC-001 (Consent & Preferences) -> [client taps opt back in] -> SPEC-005 (Consent Re-Grant Action)
SPEC-004 (Opt-Out / STOP Processing) / SPEC-005 (Consent Re-Grant Action) -> [near-simultaneous updates] -> SPEC-006 (Concurrent Consent Update Resolution)
SPEC-003 / SPEC-004 / SPEC-005 / SPEC-006 -> [consent state changes] -> SPEC-007 (Textability Determination Rule) -> FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule)
FEAT-13 (Client Record Management) -> [Pro edits a client's phone number] -> SPEC-008 (Phone Number Change Consent Invalidation Rule) -> SPEC-003 (fresh consent required before next text)
```

**Default Entry:** N/A -- this feature has no dashboard or menu entry point of its own. SPEC-001 is not a standalone screen and is never navigated to directly: its only entry point is FEAT-06.SPEC-005, the Preferences screen that hosts it (itself reached from FEAT-06.SPEC-003 and FEAT-06.SPEC-004), and SPEC-002 only by tapping an opt-out link inside a message.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-14.SPEC-003 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Reads the client's opt-in choice made on the booking screen to create the consent record | Client submits a booking |
| FEAT-14.SPEC-001 | Inbound | FEAT-06 (Client Booking Identity) | Hosted as the consent section inside FEAT-06.SPEC-005 (Consent & Email Preferences), its only entry point; FEAT-06.SPEC-003/004 navigate to FEAT-06.SPEC-005, not to this spec | Client opens the Preferences screen |
| FEAT-14.SPEC-002 / FEAT-14.SPEC-004 | Inbound | FEAT-08 (Automated Booking Messaging) | The opt-out link tapped, and the STOP-reply inbound event, both arrive through messages FEAT-08 sends and the text capability FEAT-08.SPEC-012 owns | Client taps a link or replies STOP to any product message |
| FEAT-14.SPEC-009 | Outbound | FEAT-08 (Automated Booking Messaging) | Sends the STOP-reply confirmation through the text/email capability FEAT-08.SPEC-012 / FEAT-08.SPEC-013 own | STOP reply processed |
| FEAT-14.SPEC-007 | Outbound | FEAT-08 (Automated Booking Messaging) | FEAT-08.SPEC-011's channel-selection rule reads this feature's textability determination before every client-directed send | Every client-directed send |
| FEAT-14.SPEC-007 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | The Pro's planning view reads current textability (View-only; cannot override) | Pro views their schedule |
| FEAT-14.SPEC-008 | Inbound | FEAT-13 (Client Record Management) | A Pro's edit to a client's phone number triggers consent invalidation | Pro edits a client's phone number |
| FEAT-14.SPEC-003 / FEAT-14.SPEC-004 / FEAT-14.SPEC-005 | Outbound | FEAT-13 (Client Record Management) | A client deletion removes the consent record's identifying details and retains de-identified evidence, per XBR-19 | Client deletion is requested |

## Non-Functional Notes

**Data volumes / growth:** Exactly one Messaging Consent record per client-Pro relationship, holding only current state plus its history of transitions -- growth tracks the client base, not message volume, and stays small even over a pro's multi-year history (SC-22).

**Responsiveness:** A revoke (link tap or STOP reply) is honored on the very next message sent, with no grace period; a re-grant takes effect immediately for the client's next message as well (Validation & Limits field; XBR-15).

**Data sensitivity / privacy:** Every Messaging Consent record is compliance evidence tied to a phone number, kept as proof of consent under US SMS-consent rules (ASMP-24); it is personal data visible only to the Client and their Pro, with Platform Operator (Support) limited to View-only access and never able to override a client's revoked consent (Access field). SPEC-001 and SPEC-002 must remain fully usable at phone width, with a screen reader, and without relying on color alone (ASMP-28).

**Compliance flags:** US SMS-consent rules apply to this feature directly: explicit, never-pre-checked opt-in captured at booking; a revoke honored immediately with no grace period; a changed phone number invalidates existing consent (ASMP-24, XBR-15). This feature owns the consent state that FEAT-08 must uphold on every send.

## Non-Goals

- **Marketing or promotional text campaigns** -- Excluded per scope-boundaries.md (SC-15): clients consent to booking-related texts only, and Chairtime sends nothing beyond confirmations, reminders, change notices, and access links; this feature governs consent for that fixed set of message types, not a broader marketing-preference system.
- **WhatsApp as a consent channel** -- Deferred to a later phase per scope-boundaries.md's Relevant Deferral Notes: "WhatsApp is a nice-to-have later, not v1" (BRIEF.md, Ecosystem & Integrations); the channel field's WhatsApp value is reserved for that later phase and carries no behavior today.
- **A consent record shared or visible across pros** -- Excluded per scope-boundaries.md (SC-04) and SC-03: a client's identity, and therefore their consent, is scoped to one client-Pro relationship; booking with a second Pro creates an entirely separate, unconnected Messaging Consent record.
- **Purge of consent evidence after client deletion** -- Intentional lifecycle decision surfaced by the CRUD matrix: the exact consent wording, timestamp, and channel are retained in de-identified form after a client deletion because they are the compliance evidence US SMS-consent rules require (ASMP-24, SC-22, XBR-19) -- there is no purge path to design.
- **Sending the product's own confirmations and reminders** -- This feature governs the consent state that gates every send but does not compose or dispatch confirmations, reminders, or change notices itself; that remains Automated Booking Messaging's (FEAT-08) responsibility, per this feature's Communications field ("N/A -- this feature governs communications rather than sending its own") and the Interactions field ("Depended on by FEAT-08").



# Screen Spec: Consent & Preferences

## Overview

**Name:** Consent & Preferences
**ID:** FEAT-14.SPEC-001
**Type:** Screen
**Purpose:** The consent section of the client's Preferences screen: the client's current texting consent status for this Pro, with an action to opt back in to texting when it is currently off. This section is hosted inside FEAT-06.SPEC-005 (Consent & Email Preferences), the single client-facing Preferences screen; it is not a separately reachable screen.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

**Hosting note:** FEAT-06.SPEC-005 (Consent & Email Preferences) is the single client-facing Preferences screen; FEAT-06.SPEC-003 and FEAT-06.SPEC-004 navigate to it. This spec defines the consent section that FEAT-06.SPEC-005 hosts and keeps the consent-control behavior (status display and re-grant) governed by FEAT-14's rules (XBR-15). FEAT-06.SPEC-005 owns the surrounding screen, its header, and the email field.

## Scope and Non-Goals

**In Scope:**
- Displaying the client's current texting consent status for this Pro (on/off), derived from FEAT-14.SPEC-007's textability determination
- A plain-language explanation that messages arrive by email when texting is off
- An action to re-grant (opt back in to) texting when it is currently off

**Non-Goals:**
- Hosting the screen itself (header, back navigation, page-level loading and error states, entry points) or editing the client's own email address -- owned by FEAT-06.SPEC-005 (Consent & Email Preferences), which holds the Client entity's email field write path; this feature never writes Client contact fields (Feature Dependency Map: Client is a read-only referenced entity for FEAT-14).
- Revoking texting consent from this screen -- excluded per this feature's own Key Capabilities: the product's only revoke paths are the opt-out link (FEAT-14.SPEC-002) and a STOP reply (FEAT-14.SPEC-004); offering a second, in-app revoke control here would create a third precedence case for FEAT-14.SPEC-006 to resolve with no product benefit.
- Showing consent history or past state transitions -- excluded per scope-boundaries.md SC-22: only the current status is client-facing; the timestamp and exact wording shown are retained solely as compliance evidence, never surfaced as a browsable log.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Client opens the Preferences screen (reached from "Preferences" on FEAT-06.SPEC-003 and FEAT-06.SPEC-004); this consent section renders inside it | The Pro relationship already scoped by FEAT-06.SPEC-005's viewing session; current consent status loads fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own texting consent status for this one Pro | Re-grant texting consent (only when currently off) | -- |
| The Pro (Talia) | No | No | The Pro sees textability for planning through FEAT-12's dashboard only (View, never override, per the Access Matrix's Messaging & Consent row); this client-facing screen has no Pro entry point |
| Platform Operator (Support) | No | No | Support's View-only access to Messaging Consent (Access Matrix) is delivered through its own troubleshooting view, never this client-facing screen (scope-boundaries SC-05) |
| Unauthenticated | No | No | Reachable only as the consent section of FEAT-06.SPEC-005, inside an already-authenticated viewing session; a direct, unauthenticated attempt to reach the host screen is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; on return after the underlying access link has become Used or Expired, the client is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Section heading "Texting" inside FEAT-06.SPEC-005's Preferences screen; the screen title and back arrow belong to the host screen (FEAT-06.SPEC-005).

**Body:** A single status section:
- A status line: "Texting: on" or "Texting: off", reflecting FEAT-14.SPEC-007's current textability determination
- When off, a plain-language line directly below: "Your confirmations and reminders go to your email instead."
- When off, a "Turn texting back on" button below that line
- When on, no button is shown -- the status line is the entire content

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Status section stacks vertically, full width.
- **Medium size class and above:** Same vertical order, content column capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (host screen, FEAT-06.SPEC-005) | Tap | Handled by FEAT-06.SPEC-005, which returns to FEAT-06.SPEC-003 or FEAT-06.SPEC-004 | Host screen closes | Standard navigation transition |
| "Turn texting back on" button (shown only when status is off) | Tap | Triggers FEAT-14.SPEC-005 (Consent Re-Grant Action) | Button shows a brief loading state | Success: status line updates to "Texting: on", button disappears, confirmation "Texting turned back on". Failure: inline message "Couldn't update your preference. Try again." with the button re-enabled |
| "Turn texting back on" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Within this section: status line -> "Turn texting back on" button (when shown); the host screen's back arrow precedes the section.
- **Announcements:** The confirmation "Texting turned back on" and the failure message are announced to assistive technology as they appear; the status line's change from "off" to "on" is announced alongside the confirmation.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the status line will appear | Screen first opens | Data finishes loading |
| Populated (Texting On) | "Texting: on" shown, no button | Data loads with active consent (Granted or Re-granted) | Client navigates away, or the client's consent is revoked elsewhere and this screen is reloaded |
| Populated (Texting Off) | "Texting: off" with the email-fallback line and "Turn texting back on" button | Data loads with Revoked consent | Client's re-grant action succeeds |
| Saving | The button shows a loading state | Client taps "Turn texting back on" | The write completes or fails |
| Error | Error banner "We couldn't load your preferences. Try again." in place of the status section, with a retry action | The initial status load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline. Reconnect to update your texting preference." at the top; the status line remains visible but the "Turn texting back on" button is disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and the button re-enables |

## Validation Rules

**Option B -- Inline:**

This screen accepts no free-text input. The single action (re-grant) has no field-level validation; its only precondition is that the current status is off, which the screen itself enforces by hiding the button when status is on.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (host screen's control) | FEAT-06.SPEC-005 returns the client to the screen they arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | FEAT-06 |

## Data Model

**Creates:** None.
**Reads:** Messaging Consent -- channel, state, timestamp for this Client and Pro, resolved through FEAT-14.SPEC-007's textability determination.
**Updates:** None directly -- the re-grant write is performed by the triggered automation, FEAT-14.SPEC-005.
**Deletes:** None.

## Business Rules

- The status shown here is exactly the value FEAT-14.SPEC-007 computes -- this screen never derives its own consent logic or caches a stale value across sessions.
- The re-grant action is the only consent change this screen can cause; it is executed entirely by FEAT-14.SPEC-005, and any concurrent-write precedence (e.g., a STOP reply arriving close together) is governed by FEAT-14.SPEC-006.
- XBR-15 governs the underlying behavior this status reflects: no text is sent without active texting consent; a revoke is honored on the very next message; a re-grant takes effect for the client's next message.

## Edge Cases

- **Status becomes stale between load and any action on this screen** -- This screen is a load-time snapshot, not live-updating. If the client's consent changes elsewhere (e.g., a STOP reply arrives) while this screen is open showing "on", the screen does not update in place; the next time the client opens this screen, it reflects the reconciled state per FEAT-14.SPEC-006. No action is available while showing "on" that could conflict with an external change.
- **Client taps "Turn texting back on" at effectively the same moment a STOP reply arrives for the same phone number** -- Per FEAT-14.SPEC-006, the most recent explicit client action by timestamp wins; if the STOP reply's timestamp is later, the write still succeeds as a re-grant attempt, but the resulting state is Revoked, and the client sees "Texting: off" on their next view of this screen rather than the "on" state the tap requested.
- **Client double-taps "Turn texting back on"** -- The second tap is ignored while the first write is in flight (button in loading state).
- **Network failure during the re-grant write** -- Inline message "Couldn't update your preference. Try again." appears; the status line remains unchanged (off) and the button re-enables for another attempt.
- **Client navigates away while the re-grant write is in flight** -- The write completes in the background regardless of navigation; the status is correct on the client's next visit to this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Hosted by (inbound) | The single client-facing Preferences screen renders this consent section; FEAT-06.SPEC-003 and FEAT-06.SPEC-004 reach this section only through it |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Supplies the current status this screen displays |
| FEAT-14.SPEC-005 (Consent Re-Grant Action) | Triggers (outbound) | The "Turn texting back on" button fires this automation |
| FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) | References (inbound) | Governs precedence if a re-grant races a revoke |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| consent_preferences_viewed | current_status (on / off) | Screen finishes loading | N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references texting-consent behavior; retained as an operational signal so this screen's usage is observable |
| consent_regrant_attempted | outcome (succeeded / failed / superseded_by_revoke) | Client taps "Turn texting back on" and the write resolves | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-14.SPEC-001-AC-01:** Given Riley's texting consent for Talia is currently Revoked, when she opens this screen, then it shows "Texting: off", the email-fallback line, and a "Turn texting back on" button.

**FEAT-14.SPEC-001-AC-02:** Given Riley's texting consent is currently Granted, when she opens this screen, then it shows "Texting: on" with no button.

**FEAT-14.SPEC-001-AC-03:** Given Riley taps "Turn texting back on", when the write succeeds, then the status line updates to "Texting: on", the button disappears, and she sees "Texting turned back on".

**FEAT-14.SPEC-001-AC-04:** Given Riley taps "Turn texting back on" and the write fails, when the failure is reported, then she sees "Couldn't update your preference. Try again." and the button remains available.

**FEAT-14.SPEC-001-AC-05:** Given Riley taps "Turn texting back on" at the same time a STOP reply for her phone number is processed with a later timestamp, when precedence resolves per FEAT-14.SPEC-006, then her status shows "Texting: off" on her next view of this screen.

**FEAT-14.SPEC-001-AC-06:** Given the initial load of Riley's status fails, when the failure occurs, then an error banner "We couldn't load your preferences. Try again." appears with a retry action.

**FEAT-14.SPEC-001-AC-07:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then a banner explains she is offline and the "Turn texting back on" button becomes disabled.

**FEAT-14.SPEC-001-AC-08:** Given Riley taps "Turn texting back on" twice rapidly, when the second tap registers, then it is ignored while the first write is still in progress.

**FEAT-14.SPEC-001-AC-09:** Given Riley looks for a way to edit her email address inside this consent section, when she reviews the layout, then no such control exists in the section -- the email field lives elsewhere on the host screen, FEAT-06.SPEC-005.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 4 (loading, error, saving, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Opt-Out Link Landing

## Overview

**Name:** Opt-Out Link Landing
**ID:** FEAT-14.SPEC-002
**Type:** Screen
**Purpose:** The page a client lands on after tapping the opt-out link included in a text message, confirming that texting has been turned off.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- The landing page shown after a valid opt-out link tap
- Triggering FEAT-14.SPEC-004 (Opt-Out / STOP Processing) to revoke consent
- The error state for an already-used, expired, or foreign opt-out link

**Non-Goals:**
- Processing the STOP text-reply path -- owned entirely by FEAT-14.SPEC-004; this screen exists only for the link-tap path, since a text reply has no screen to land on.
- Offering a re-grant action on this page -- excluded per this feature's Key Capabilities: re-granting happens only from FEAT-14.SPEC-001 (Consent & Preferences) or at a later booking (FEAT-14.SPEC-003), never as an immediate undo on the confirmation page itself, so a client cannot accidentally reverse a deliberate opt-out in the same tap sequence that requested it.
- Showing the client's other preferences (email address, booking details) -- excluded per scope-boundaries.md SC-15: this page confirms one thing, the opt-out, and does not become a general preferences hub; a client who wants more goes to FEAT-14.SPEC-001 via FEAT-06's "my bookings".

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| External (a message sent through FEAT-08.SPEC-012, transactional text messaging capability) | Client taps the opt-out link embedded in any text message | The link's own reference to the client-Pro relationship the consent applies to |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | None -- this page is a confirmation, not an interactive form | -- |
| The Pro (Talia) | No | No | The Pro has no entry point to this page; it exists only for the client who received and tapped the specific link |
| Platform Operator (Support) | No | No | Support's View-only access to consent state is delivered through its own troubleshooting view, never this page (scope-boundaries SC-05) |
| Unauthenticated | Yes (via the link itself) | No | The link is the entire authentication for this page -- no sign-in is required or offered; a client who reaches this page without a valid link sees the Invalid Link state described below, never a sign-in prompt |
| Expired session | N/A -- this page has no session concept | N/A | This page is a single-use landing reached directly by a link; it has no ongoing session to expire |

single-role product — no restricted elements (this screen serves only the Client role; the two "No/No" rows above describe roles with no entry point, not a restriction on an entry point they have)

## Layout and Content

**Header:** None -- this page has no navigation chrome; it is a standalone confirmation destination.

**Body:** A single centered block:
- A confirmation heading: "Texting turned off"
- A plain-language body line: "You won't receive any more texts from {pro_display_name}. Your confirmations and reminders go to your email instead."
- No action buttons -- the page is informational only

**Footer:** None.

### Responsive Behavior

- **Compact and above:** Uniform scaling, no structural change -- the confirmation block is short enough to need no restructuring at any size.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Page load (implicit -- no interactive element) | Client's tap on the opt-out link resolves to this page | Triggers FEAT-14.SPEC-004 (Opt-Out / STOP Processing) for the link's referenced client-Pro relationship | Page shows a brief loading state, then the confirmation | Confirmation heading and body appear once the automation completes |

### Accessibility Notes

- **Focus order:** The confirmation heading receives focus on page load so assistive technology announces it immediately, since there is no other interactive content to navigate to.
- **Announcements:** The confirmation heading and body are announced together on load; the Invalid Link state's message is announced the same way when that state renders instead.
- **Keyboard alternatives:** N/A -- this page has no interactive controls.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief loading indicator in place of the confirmation block | Page first opens while FEAT-14.SPEC-004 processes the revoke | Processing completes |
| Confirmed | "Texting turned off" heading with the plain-language body line | FEAT-14.SPEC-004 successfully revokes consent | Terminal -- the client closes the page; there is nothing further to do |
| Invalid Link | "This link has already been used or is no longer valid." with no further action offered | The link is already used, expired, or does not resolve to a genuine client-Pro relationship | Terminal -- the client closes the page |
| Offline/Degraded | "You're offline. Reconnect and tap the link again to confirm." in place of the confirmation block | The page cannot reach the product to process the revoke because the client's device is offline | Connectivity is restored and the client reopens the link -- there is no queued action on this page, since a compliance-sensitive opt-out must never be silently deferred |

## Validation Rules

Not applicable -- this page accepts no user input. The only validity check is on the link itself, governed by FEAT-14.SPEC-004's processing (whether the link is Issued and unexpired, or already used/expired/foreign).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| None | This page has no outbound navigation -- it is a terminal confirmation destination, not a hub | -- |

## Data Model

**Creates:** None directly -- the underlying Messaging Consent update is performed by FEAT-14.SPEC-004.
**Reads:** Messaging Consent -- enough of the link's reference to resolve which client-Pro relationship the opt-out applies to, and Pro Account -- display_name, to render the confirmation's plain-language body.
**Updates:** None directly (delegated to FEAT-14.SPEC-004).
**Deletes:** None.

## Business Rules

- This page never asks the client to confirm the opt-out a second time -- tapping the link is itself the explicit action; no-grace-period revocation (XBR-15) means the effect is immediate and the page reflects an already-completed action, not a pending one.
- The Invalid Link state never reveals which of "already used," "expired," or "foreign booking" applies -- a single generic message is shown regardless of cause, consistent with the product's pattern for other link-based screens (e.g., FEAT-06.SPEC-002's identical discipline for access links) so that no link's validity state is probed by trial.
- FEAT-14.SPEC-004 is the sole owner of the actual consent write this page triggers; this screen never writes Messaging Consent state itself.

## Edge Cases

- **The same opt-out link is tapped twice (e.g., opened on two devices)** -- The first tap's processing by FEAT-14.SPEC-004 revokes consent and completes; the second tap finds the link already resolved to a revoked state and shows the Invalid Link message, since the link's single-use discipline mirrors FEAT-06.SPEC-007's Access Link pattern -- there is no harm in showing "already used" here, because the underlying consent state is already the safe, revoked outcome either way.
- **The client's texting consent was already Revoked before this tap (e.g., a prior STOP reply)** -- The page still shows the Confirmed state ("Texting turned off"), since the client's intent and the actual state agree; FEAT-14.SPEC-004 treats this as a no-op write with the same user-visible outcome.
- **The link resolves to a client-Pro relationship that no longer exists (the Client record was deleted, per FEAT-13/XBR-19)** -- The Invalid Link state is shown; a deleted client's consent record no longer exists to update, so the page does not attempt to process the tap as a live revoke.
- **Client is offline when tapping the link** -- The Offline/Degraded state is shown with no queued write; per XBR-15's no-grace-period discipline, an opt-out must be processed at the moment the client acts, never silently deferred to a later reconnect, so the client is asked to tap the link again once back online rather than assuming success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Navigation (inbound) | The opt-out link embedded in every product text lands here |
| FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Triggers (outbound) | Page load triggers the revoke for the link's referenced relationship |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (outbound) | Enforces this rule's textability determination indirectly: the landing outcome reflects whether FEAT-14.SPEC-004's revoke changed the state that FEAT-14.SPEC-007 evaluates |
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | This screen's generic Invalid Link message mirrors that spec's single-use, no-cause-disclosed pattern for link-based screens |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| opt_out_link_landing_shown | outcome (confirmed / invalid_link / offline) | Page finishes loading | N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature; retained as an operational signal so opt-out volume via the link path is observable |

## Acceptance Criteria

**FEAT-14.SPEC-002-AC-01:** Given Riley taps a valid, unused opt-out link in a text from Talia, when the page loads, then FEAT-14.SPEC-004 processes the revoke and the page shows "Texting turned off" with the email-fallback explanation naming Talia.

**FEAT-14.SPEC-002-AC-02:** Given Riley taps an opt-out link that was already used, when the page loads, then it shows "This link has already been used or is no longer valid." with no further action offered.

**FEAT-14.SPEC-002-AC-03:** Given Riley taps an opt-out link whose client-Pro relationship no longer exists, when the page loads, then it shows the same Invalid Link message as an already-used link, never revealing the specific cause.

**FEAT-14.SPEC-002-AC-04:** Given Riley's texting consent was already Revoked before this tap, when the page processes the tap, then it still shows "Texting turned off" as a no-op confirmation.

**FEAT-14.SPEC-002-AC-05:** Given Riley is offline when she taps the opt-out link, when the page attempts to load, then it shows "You're offline. Reconnect and tap the link again to confirm." with no write attempted.

**FEAT-14.SPEC-002-AC-06:** Given Riley taps the same valid opt-out link twice from two devices, when the second tap is processed after the first has already revoked consent, then the second device shows the Invalid Link message.

**FEAT-14.SPEC-002-AC-07:** Given Riley reaches this page, when she looks for a way to undo the opt-out immediately, then no such control exists on this page.

**FEAT-14.SPEC-002-AC-08:** Given Riley reaches this page, when she looks for navigation elsewhere on the product, then none is offered -- the page is a terminal confirmation destination.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 1 | 1 |
| States | 4 (loading, confirmed, invalid link, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Consent Capture at Booking

## Overview

**Name:** Consent Capture at Booking
**ID:** FEAT-14.SPEC-003
**Type:** Automation
**Purpose:** Records the client's opt-in or opt-out choice made at booking as a Messaging Consent record, with state, timestamp, channel, and the exact consent wording shown, creating the record on a client's first booking with a Pro and updating it on a later booking if the choice changes.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- Creating the Messaging Consent record on a client's first booking with a given Pro
- Updating the existing Messaging Consent record when a returning client makes a different choice on a later booking
- Capturing the exact consent wording shown at the moment of the choice, as compliance evidence

**Non-Goals:**
- The opt-in checkbox and its never-pre-checked default -- owned by FEAT-05.SPEC-003 (Client Details & Consent); this automation begins once that screen's booking submission carries the client's choice, and never renders the checkbox itself.
- Processing a revoke via opt-out link or STOP reply -- owned by FEAT-14.SPEC-004; this automation only ever writes Granted (a fresh opt-in) or the initial Revoked state (an opt-out at booking), never a mid-relationship revoke.
- Resolving a race between this automation's write and a concurrent STOP reply or re-grant -- owned by FEAT-14.SPEC-006; this automation always fires from a single, sequential booking submission and has no concurrent-trigger case of its own beyond the general two-run-in-flight edge case below.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client submits a booking with an opt-in/opt-out choice (first booking with this Pro) | FEAT-05.SPEC-003 (Client Details & Consent), where the choice is captured, and FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), where the booking is submitted | Fires on a successful booking submission where no Messaging Consent record yet exists for this Client-Pro relationship | The client's opt-in/opt-out choice, the exact consent wording shown on FEAT-05.SPEC-003, the client's phone number, the current timestamp |
| Client submits a later booking with a different opt-in/opt-out choice than their existing record | FEAT-05.SPEC-003 (Client Details & Consent), where the new choice is captured, and FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), where the booking is submitted | Fires on a successful booking submission where a Messaging Consent record already exists for this Client-Pro relationship and the newly submitted choice differs from its current state | The client's new choice, the exact consent wording shown at this booking, the client's phone number, the current timestamp, the existing record's prior state |

## Processing Logic

1. Receive the booking submission's opt-in/opt-out choice, the exact consent wording text shown on FEAT-05.SPEC-003 at that moment, the client's phone number, and the timestamp of submission.
2. Check whether a Messaging Consent record already exists for this Client-Pro relationship.
3. If no record exists: create one with channel set to text, state set to Granted (if the client opted in) or Revoked (if the client opted out), timestamp set to the submission time, consent_wording set to the exact text shown, and phone_number set to the client's phone number at booking.
4. If a record already exists: compare the newly submitted choice to the record's current state.
   - If the choice matches the current state, leave the record unchanged (no-op; the booking's own acknowledgment is still logged by FEAT-16, but this automation makes no write).
   - If the choice differs, update the record: set state to Granted or Revoked per the new choice, timestamp to the current submission time, consent_wording to the wording shown at this booking, and phone_number to the client's current phone number.
5. Confirm the write completed before allowing FEAT-05's booking confirmation step to proceed to FEAT-08's confirmation-message channel selection.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First-booking creation | No prior Messaging Consent record exists for this relationship | Messaging Consent record created with the submitted choice | None distinct from the booking confirmation itself -- consent capture is silent and folded into the booking flow | FEAT-05.SPEC-005 (Booking Confirmation), FEAT-08.SPEC-011 (reads the resulting state for the confirmation message's channel) |
| Later-booking update (choice changed) | A record exists and the new choice differs from its current state | Messaging Consent record's state, timestamp, consent_wording, and phone_number updated | None distinct from the booking confirmation itself | FEAT-05.SPEC-005, FEAT-08.SPEC-011, FEAT-14.SPEC-001 (the client's next view of their status reflects the change) |
| Later-booking no-op (choice unchanged) | A record exists and the new choice matches its current state | None | None distinct from the booking confirmation itself | -- |
| Write failure | The consent write cannot complete (a processing error, not a validation failure -- the choice itself is always a simple binary value) | No Messaging Consent record is created or updated | The booking submission itself is not blocked or rolled back by a consent-write failure alone; the booking proceeds, and the client's channel defaults to the safer no-text state (email) until the write can be confirmed, per this feature's own Error-state discipline | FEAT-08.SPEC-011 (defaults to email when consent state is unconfirmed) |

## Data Model

**Creates:** Messaging Consent -- channel, state, timestamp, consent_wording, phone_number, on a client's first booking with a Pro.
**Reads:** Client -- to identify the owning Client-Pro relationship and current phone_number (read-only; this automation never writes the Client entity).
**Updates:** Messaging Consent -- state, timestamp, consent_wording, phone_number, on a later booking with a changed choice.
**Deletes:** None.

## Business Rules

- XBR-15: no text is sent without active texting consent; this automation is the sole creation path for that consent record, and its write must complete before any message tied to the same booking selects a channel (FEAT-08.SPEC-011).
- Consent capture must never be pre-checked at the source screen (FEAT-05.SPEC-003); this automation records exactly the choice the client made, never a default assumption.
- The consent_wording captured is the literal text shown to the client at that specific booking, not a generic or later-edited version of the disclosure -- consistent with ASMP-24's evidentiary requirement.
- A client booking with a second Pro creates an entirely separate, unconnected Messaging Consent record (scope-boundaries SC-04); this automation never reuses or looks up a consent record from a different Pro relationship.
- A phone number change invalidates the existing record before this automation would next run for that relationship (FEAT-14.SPEC-008); if this automation fires for a booking made under a newly changed number with no fresh consent yet on file, it treats the booking as if capturing consent for the first time under that number.

## Edge Cases

- **Client submits a booking with a phone number that differs from any existing Messaging Consent record's phone_number for this same Pro relationship (a very recent number change)** -- Per FEAT-14.SPEC-008, the prior record was already invalidated by the number change; this automation treats the submission as fresh consent capture for the new number and updates the record's phone_number, state, timestamp, and consent_wording accordingly.
- **The exact consent wording shown includes dynamic content (e.g., the Pro's display name)** -- The wording is captured exactly as rendered at that moment, including any such substitution, since the evidentiary requirement is what the client actually saw, not a template.
- **A booking is submitted and then immediately cancelled before payment completes** -- If FEAT-05's checkout re-validation (FEAT-05.SPEC-006) rejects the booking before this automation's trigger condition (a successful booking submission) is met, this automation never fires; a rejected or abandoned checkout never produces or updates a Messaging Consent record.
- **Concurrent trigger firing (two devices submit a booking for the same Client-Pro relationship at effectively the same time -- not realistically possible for the same client, but a data-consistency case worth stating)** -- The Client entity's own contention rule (phone-number match resolves to a single record) applies first; whichever booking submission commits first at the Client/Booking level is the one this automation processes, and the second submission's consent write, if it differs, is handled as a later-booking update against the just-created record, following the standard update path in Processing Logic step 4.
- **Trigger fires while a previous run for the same relationship is still in flight** -- A second submission for the same Client-Pro relationship cannot be in flight at the same time in practice, since FEAT-05's checkout flow serializes one booking submission at a time per client session; if it were to occur, the second run's read of "does a record exist" would wait for the first run's write to commit, and then proceed as a later-booking update rather than a duplicate creation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Triggered by (inbound) | Source of the opt-in/opt-out choice and the exact consent wording shown |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Triggered by (inbound) | Booking submission is the moment this automation fires |
| FEAT-05.SPEC-005 (Booking Confirmation) | Affects (outbound) | The confirmation step proceeds only after this automation's write completes |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | Affects (outbound) | Reads the resulting Messaging Consent state to choose the confirmation message's channel |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (inbound) | Governs the state this automation finds when a phone number changed since the client's last booking |
| FEAT-14.SPEC-001 (Consent & Preferences) | Affects (outbound) | The client's next view of their status reflects any change this automation makes |

## Analytics and Success Signals

- **consent_captured** (outcome: created / updated / no_op; choice: opt_in / opt_out) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references consent capture; retained as an operational signal so opt-in/opt-out volume at booking is observable.
- **consent_write_failed** (booking reference) -- N/A -- same reason as above; retained so a silent consent-capture failure is never invisible, consistent with this feature's own Error-state discipline defaulting to no-text on doubt.

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given Riley books with Talia for the first time and checks the opt-in box, when her booking submission succeeds, then a Messaging Consent record is created with state Granted, the exact wording shown, her phone number, and the submission timestamp.

**FEAT-14.SPEC-003-AC-02:** Given Riley books with Talia for the first time and leaves the opt-in box unchecked, when her booking submission succeeds, then a Messaging Consent record is created with state Revoked and the same wording/timestamp/phone_number capture.

**FEAT-14.SPEC-003-AC-03:** Given Riley already has an active (Granted) Messaging Consent record with Talia, when she books again and leaves her choice unchanged, then no write occurs to the existing record.

**FEAT-14.SPEC-003-AC-04:** Given Riley's existing Messaging Consent record with Talia is Revoked, when she books again and checks the opt-in box this time, then the record is updated to Granted with a fresh timestamp and wording capture.

**FEAT-14.SPEC-003-AC-05:** Given Riley books with a second Pro, Jordan, for the first time, when her booking submission succeeds, then a wholly separate Messaging Consent record is created for the Riley-Jordan relationship, unconnected to her Riley-Talia record.

**FEAT-14.SPEC-003-AC-06:** Given Riley's booking submission is rejected by checkout re-validation before payment completes, when the rejection occurs, then no Messaging Consent record is created or updated.

**FEAT-14.SPEC-003-AC-07:** Given the consent write fails due to a processing error after a successful booking, when the failure occurs, then the booking itself still completes and the client's channel defaults to email until the write is confirmed.

**FEAT-14.SPEC-003-AC-08:** Given Riley's phone number changed since her last booking with Talia and her prior consent was invalidated per FEAT-14.SPEC-008, when she submits a new booking with a fresh opt-in choice, then this automation treats it as first-time capture under the new number.

**FEAT-14.SPEC-003-AC-09:** Given the consent wording shown to Riley at this specific booking includes Talia's display name, when the record is created or updated, then the consent_wording field stores that exact rendered text.

**FEAT-14.SPEC-003-AC-10:** Given this automation's write for Riley's booking has not yet committed, when FEAT-08.SPEC-011 evaluates the channel for the resulting confirmation message, then it waits for the write to complete before selecting a channel, per the sequencing in Processing Logic step 5.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first booking, later booking with changed choice) | 2 |
| Outcome Paths | 4 (creation, update, no-op, write failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Opt-Out / STOP Processing

## Overview

**Name:** Opt-Out / STOP Processing
**ID:** FEAT-14.SPEC-004
**Type:** Automation
**Purpose:** Revokes a client's texting consent immediately, with no grace period, whether triggered by a tapped opt-out link or an inbound "STOP" text reply.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- Revoking Messaging Consent to the Revoked state from either trigger path
- Determining which client-Pro relationship the revoke applies to, for each trigger path
- Firing the opt-out confirmation notification after a successful STOP-reply revoke

**Non-Goals:**
- Rendering the opt-out link landing page itself -- owned by FEAT-14.SPEC-002; this automation is what that screen triggers, not the screen.
- Composing or sending the confirmation message's content -- owned by FEAT-14.SPEC-009 (Opt-Out Confirmation Message); this automation only fires that notification as an outcome, per Business Rules below.
- Re-granting consent -- owned by FEAT-14.SPEC-005; this automation only ever moves consent toward Revoked, never the reverse.
- Deciding precedence when this automation's revoke races a concurrent re-grant -- owned by FEAT-14.SPEC-006; this automation performs its own write unconditionally and defers to that spec's rule for the resulting final state when a race is detected.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Opt-out link tapped | FEAT-14.SPEC-002 (Opt-Out Link Landing) | Fires when the landing page loads for a link that is Issued and unexpired, and resolves to a genuine, still-existing client-Pro relationship | The relationship's Messaging Consent record reference, the current timestamp |
| Inbound "STOP" reply received | FEAT-08.SPEC-012 (Transactional Text Messaging Capability), Inbound Events | Fires when the capability reports an inbound reply containing the keyword "STOP" (or a recognized equivalent) from a phone number matching an existing Client record's phone | The replying phone number, the current timestamp, the message thread's Pro reference (which Pro's number the reply was sent to) |

## Processing Logic

1. Resolve the triggering event to a specific Messaging Consent record: for a link tap, use the link's own reference; for a STOP reply, match the replying phone number and the Pro's number it replied to against an existing Client record and that Client's Messaging Consent record for this Pro.
2. If no matching record can be resolved (an unrecognized phone number, or a link referencing a relationship that no longer exists), stop processing -- there is nothing to revoke.
3. Set the resolved Messaging Consent record's state to Revoked and its timestamp to the current processing time. Consent_wording and channel are left unchanged from the record's prior state -- the revoke does not alter the historical evidence of when and how consent was originally given.
4. If the resolved record's state was already Revoked, treat the write as a no-op (the state does not change further, and the timestamp is not disturbed by a redundant revoke) -- see Edge Cases.
5. If the trigger was a STOP reply (not a link tap), fire FEAT-14.SPEC-009 (Opt-Out Confirmation Message) for the resolved Client-Pro relationship. A link-tap revoke does not fire this notification, since the landing page (FEAT-14.SPEC-002) itself is the client-facing confirmation.
6. Confirm the write completed so that any message already queued to send for this relationship re-evaluates its channel through FEAT-08.SPEC-011 before dispatch.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Revoke via link tap | Link tap resolves to an active or already-revoked relationship | Messaging Consent state set to Revoked, timestamp updated (unless already Revoked) | The landing page (FEAT-14.SPEC-002) shows the Confirmed state | FEAT-14.SPEC-002, FEAT-08.SPEC-011 |
| Revoke via STOP reply | STOP reply resolves to an existing Client-Pro relationship | Messaging Consent state set to Revoked, timestamp updated (unless already Revoked) | No in-app screen feedback (the client is not in the product); FEAT-14.SPEC-009 sends a confirmation message | FEAT-14.SPEC-009, FEAT-08.SPEC-011 |
| No matching relationship | The link references a relationship that no longer exists, or the STOP reply's phone number matches no Client record for the Pro it replied to | No data change | Link tap: FEAT-14.SPEC-002 shows the Invalid Link state. STOP reply: no confirmation is sent, since there is no relationship to confirm against | FEAT-14.SPEC-002 |
| Already revoked (no-op) | The resolved record's state was already Revoked | No further state change | Link tap: FEAT-14.SPEC-002 still shows Confirmed (the client's intent and the state agree). STOP reply: FEAT-14.SPEC-009 still fires, since a redundant STOP still deserves the same reassurance | FEAT-14.SPEC-002, FEAT-14.SPEC-009 |
| Processing failure | The write itself cannot complete (a processing error, not a resolution failure) | No data change | Link tap: FEAT-14.SPEC-002 falls back to its Offline/Degraded-style messaging is not applicable here since this is a server-side failure, not connectivity -- the page instead shows the Invalid Link message rather than falsely confirming a revoke that did not happen, since a compliance-critical action must never claim success it cannot guarantee. STOP reply: no confirmation is sent, and the record is retried per Edge Cases | FEAT-14.SPEC-002 |

## Data Model

**Reads:** Client -- to match a replying phone number to a Client record (STOP path only). Messaging Consent -- the existing record for the resolved relationship.
**Creates:** None -- this automation only updates an existing record; the record itself is always created by FEAT-14.SPEC-003 at first booking.
**Updates:** Messaging Consent -- state set to Revoked, timestamp updated.
**Deletes:** None.

## Business Rules

- XBR-15: a revoke is honored on the very next message, with no grace period -- this automation's write must complete, and any in-flight channel decision (FEAT-08.SPEC-011) must observe the new state, before the next message for this relationship sends.
- A STOP reply always fires the confirmation notification (FEAT-14.SPEC-009), including when the state was already Revoked -- a client who sends STOP deserves acknowledgment every time, since they cannot see the product's internal state.
- A link-tap revoke never fires the separate confirmation notification -- the landing page itself is the confirmation; sending a text confirmation for an action just completed by tapping a link in a text would be redundant.
- This automation never re-derives or overwrites consent_wording or channel on a revoke -- those fields remain the historical evidence of the original opt-in, per ASMP-24; only state and timestamp change.
- When this automation's revoke and a concurrent action (e.g., an in-app re-grant from FEAT-14.SPEC-005) could both apply to the same record at effectively the same time, FEAT-14.SPEC-006 determines the final state; this automation always performs its own write and does not itself compare timestamps against a competing write.

## Edge Cases

- **STOP reply arrives for a phone number that matches a Client record for one Pro but the reply was sent to a different Pro's number** -- The relationship is resolved using both the replying number and the specific Pro's number the reply was sent to (per the Client entity's identity being scoped to one Pro); if no Client record exists for that specific Pro-phone pairing, this is treated as "no matching relationship."
- **The resolved Messaging Consent record was already Revoked when a STOP reply arrives** -- Per Processing Logic step 4, the write is a no-op for the state itself, but the confirmation notification still fires (Business Rules above), so the client always gets acknowledgment.
- **A STOP reply and an in-app re-grant (FEAT-14.SPEC-005) arrive within moments of each other for the same relationship** -- Concurrent trigger firing: both automations perform their own writes independently; FEAT-14.SPEC-006 governs which state (Revoked from this automation, or Re-granted from FEAT-14.SPEC-005) is the one that persists, based on the most recent explicit action's timestamp, defaulting to the no-text state if the outcome is uncertain.
- **This automation's write is still in flight when a second STOP reply arrives from the same number moments later (e.g., the client texts STOP twice)** -- The second trigger's processing waits for the first write to commit, then proceeds as the "already revoked" no-op outcome; the confirmation notification still fires for the second STOP, consistent with the "always acknowledge" rule above.
- **A processing failure occurs while writing the revoke for a STOP reply** -- The write is retried using this feature's own conservative default: until the write is confirmed, FEAT-08.SPEC-011 treats the relationship's consent as uncertain and routes any message to email rather than risk a text going out after an unconfirmed revoke; the retry follows the same delivery-retry discipline as the product's other message-processing paths (platform parameter: `message-delivery-retry-count`).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-002 (Opt-Out Link Landing) | Triggered by (inbound) | The landing page's load triggers this automation for the link-tap path |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggered by (inbound) | The STOP-reply inbound event triggers this automation |
| FEAT-14.SPEC-009 (Opt-Out Confirmation Message) | Affects (outbound) | Fired after a successful STOP-reply revoke |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | Affects (outbound) | Reads the resulting Revoked state for the very next message's channel decision |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (outbound) | Enforces this rule: once this automation's revoke is written, FEAT-14.SPEC-007 evaluates Textable = false on its next read, which is how the revoke is honored on the very next message |
| FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) | References (inbound) | Governs the final state when this automation's write races FEAT-14.SPEC-005's re-grant write |

## Analytics and Success Signals

- **consent_revoked** (trigger: link_tap / stop_reply; was_already_revoked: yes / no) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references opt-out volume; retained as an operational signal so revoke activity by trigger path is observable.
- **consent_revoke_unresolved** (trigger: link_tap / stop_reply; reason: no_matching_relationship / processing_failure) -- N/A -- same reason as above; retained so an unresolved compliance-critical revoke attempt is never silently invisible.

## Acceptance Criteria

**FEAT-14.SPEC-004-AC-01:** Given Riley taps a valid opt-out link, when FEAT-14.SPEC-002 triggers this automation, then her Messaging Consent state is set to Revoked with the current timestamp.

**FEAT-14.SPEC-004-AC-02:** Given Riley replies "STOP" to a text from Talia and her phone number matches her Client record with Talia, when the inbound event is processed, then her Messaging Consent state is set to Revoked and FEAT-14.SPEC-009 fires the confirmation message.

**FEAT-14.SPEC-004-AC-03:** Given Riley's Messaging Consent with Talia is already Revoked, when she replies "STOP" again, then no further state change occurs but FEAT-14.SPEC-009 still fires the confirmation.

**FEAT-14.SPEC-004-AC-04:** Given Riley taps an opt-out link and her consent is already Revoked, when the automation processes it, then the landing page still shows the Confirmed state.

**FEAT-14.SPEC-004-AC-05:** Given a STOP reply arrives from a phone number that matches no Client record for the Pro it was sent to, when the automation attempts to resolve it, then no write occurs and no confirmation is sent.

**FEAT-14.SPEC-004-AC-06:** Given Riley's revoke via a STOP reply completes, when the very next message for her booking with Talia is about to send, then FEAT-08.SPEC-011 selects email as the channel.

**FEAT-14.SPEC-004-AC-07:** Given a link-tap revoke completes successfully, when the outcome is evaluated, then no separate confirmation notification is fired, since the landing page itself is the confirmation.

**FEAT-14.SPEC-004-AC-08:** Given a STOP reply and an in-app re-grant for the same relationship arrive within the same second, when both automations perform their writes, then FEAT-14.SPEC-006 determines the final persisted state.

**FEAT-14.SPEC-004-AC-09:** Given Riley texts "STOP" twice in quick succession, when the second reply is processed while the first's write is still in flight, then the second is treated as an already-revoked no-op and still triggers its own confirmation message.

**FEAT-14.SPEC-004-AC-10:** Given a processing failure occurs while writing a STOP-triggered revoke, when the failure is detected, then FEAT-08.SPEC-011 routes any pending message for that relationship to email until the write is confirmed.

**FEAT-14.SPEC-004-AC-11:** Given this automation never alters consent_wording or channel on a revoke, when a record's state changes to Revoked, then its consent_wording and channel fields remain exactly as originally captured by FEAT-14.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (link tap, STOP reply) | 2 |
| Outcome Paths | 5 (link revoke, STOP revoke, no match, already-revoked no-op, processing failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Consent Re-Grant Action

## Overview

**Name:** Consent Re-Grant Action
**ID:** FEAT-14.SPEC-005
**Type:** Automation
**Purpose:** Records a client's opt-back-in to texting when they tap "Turn texting back on" on the Consent & Preferences screen.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- Writing the Re-granted state to an existing Messaging Consent record when the client explicitly opts back in from FEAT-14.SPEC-001
- Resolving to the correct client-Pro relationship for the write

**Non-Goals:**
- Rendering the "Turn texting back on" button or its screen states -- owned by FEAT-14.SPEC-001; this automation is what that screen triggers.
- Re-granting as part of a later booking's choice -- owned by FEAT-14.SPEC-003, which handles the booking-time path to the same Re-granted-equivalent outcome (recorded as Granted there, since it is a fresh booking-time choice rather than an in-app toggle); this automation is exclusively the FEAT-14.SPEC-001 screen's action.
- Deciding precedence when this automation's write races a concurrent revoke -- owned by FEAT-14.SPEC-006; this automation performs its write unconditionally and defers to that spec's rule for the final persisted state.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps "Turn texting back on" | FEAT-14.SPEC-001 (Consent & Preferences) | Fires when the button is tapped, which is shown only when the client's current Messaging Consent state for this Pro is Revoked | The client-Pro relationship reference (from the authenticated viewing session), the current timestamp |

## Processing Logic

1. Receive the trigger from FEAT-14.SPEC-001, carrying the client-Pro relationship reference already scoped by that screen's viewing session.
2. Read the existing Messaging Consent record for this relationship.
3. Set the record's state to Re-granted and its timestamp to the current processing time. Consent_wording, channel, and phone_number are left unchanged -- a re-grant does not re-capture new consent wording, since the original booking-time or opt-back-in disclosure already covers ongoing messages once texting is active again.
4. Confirm the write completed so that FEAT-14.SPEC-001 can update its displayed status and any subsequent message for this relationship re-evaluates its channel through FEAT-08.SPEC-011.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Re-grant succeeds | The write completes normally | Messaging Consent state set to Re-granted, timestamp updated | FEAT-14.SPEC-001 shows "Texting turned back on" and updates the status line to "Texting: on" | FEAT-14.SPEC-001, FEAT-08.SPEC-011 |
| Re-grant superseded by a concurrent revoke | A STOP reply or opt-out link tap resolves with a later timestamp for the same relationship at effectively the same moment | Messaging Consent's final state is Revoked, per FEAT-14.SPEC-006's precedence rule, even though this automation's own write attempt succeeded in isolation | FEAT-14.SPEC-001 shows "Texting: off" on the client's next view, despite the "Texting turned back on" confirmation having appeared at the moment of the tap | FEAT-14.SPEC-006, FEAT-14.SPEC-001 |
| Write failure | The write itself cannot complete (a processing error) | No data change | FEAT-14.SPEC-001 shows "Couldn't update your preference. Try again." and the button remains available | FEAT-14.SPEC-001 |

## Data Model

**Reads:** Messaging Consent -- the existing record for the resolved relationship.
**Creates:** None -- this automation only updates an existing record, which always already exists by the time a client can reach FEAT-14.SPEC-001 (created no later than their first booking, per FEAT-14.SPEC-003).
**Updates:** Messaging Consent -- state set to Re-granted, timestamp updated.
**Deletes:** None.

## Business Rules

- A re-grant takes effect immediately for the client's next message, mirroring XBR-15's same immediacy for a revoke -- there is no grace period or delay in either direction.
- Re-granted is treated identically to Granted by every downstream consumer of consent state (FEAT-08.SPEC-011, FEAT-14.SPEC-007) -- it is not a lesser or probationary tier of consent.
- This automation never re-captures consent_wording -- the wording field remains the historical record of the client's original opt-in disclosure, consistent with FEAT-14.SPEC-004's identical treatment of that field on a revoke.
- When this automation's write and a concurrent revoke could both apply to the same record, FEAT-14.SPEC-006 determines the final persisted state; this automation always performs its own write and does not itself compare timestamps against a competing write.

## Edge Cases

- **The client's Messaging Consent record is already Granted or Re-granted when this automation fires (a stale screen state -- the button should not have been shown)** -- The write proceeds harmlessly: state is set to Re-granted (unchanged in practical effect from Granted) and the timestamp updates; no error is raised, since re-affirming active consent is not a meaningful failure.
- **A STOP reply for the same relationship arrives within the same second as this automation's trigger** -- Concurrent trigger firing: both automations write independently; FEAT-14.SPEC-006 resolves which timestamp is later and, if genuinely indistinguishable, defaults to the no-text (Revoked) state.
- **The client double-taps "Turn texting back on" before the first write completes** -- FEAT-14.SPEC-001's screen-level debounce (button disabled while loading) prevents a second trigger from firing while the first is in flight; if a second trigger were to reach this automation regardless, it would find the record already Re-granted and proceed as a harmless re-affirming write, identical to the stale-state case above.
- **The client's phone number changed and their consent was invalidated (FEAT-14.SPEC-008) before they attempt this re-grant** -- FEAT-14.SPEC-001 reflects the invalidated (Revoked) state and still offers the button, since from the client's perspective texting is off; this automation's write still succeeds, setting state to Re-granted, but the phone_number field is left unchanged from the invalidated record -- the client's next booking (FEAT-14.SPEC-003) is what updates phone_number to the current number, since this screen-triggered automation has no phone-number input of its own to write a new value from.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Consent & Preferences) | Triggered by (inbound) | The "Turn texting back on" button fires this automation |
| FEAT-14.SPEC-001 (Consent & Preferences) | Affects (outbound) | The screen's status line and confirmation reflect this automation's outcome |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | Affects (outbound) | Reads the resulting Re-granted state for the client's next message |
| FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) | References (inbound) | Governs the final state when this automation's write races a concurrent revoke |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (inbound) | Explains why the button can be shown even after a number change invalidated consent |

## Analytics and Success Signals

- **consent_regranted** (was_superseded_by_revoke: yes / no) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references re-grant volume; retained as an operational signal so opt-back-in activity is observable.

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given Riley's Messaging Consent with Talia is Revoked, when she taps "Turn texting back on" on FEAT-14.SPEC-001, then her record's state is set to Re-granted with the current timestamp.

**FEAT-14.SPEC-005-AC-02:** Given Riley's re-grant write succeeds, when FEAT-14.SPEC-001 receives the outcome, then it shows "Texting turned back on" and updates its status line to "Texting: on".

**FEAT-14.SPEC-005-AC-03:** Given Riley's re-grant write completes, when the very next message for her booking with Talia is about to send, then FEAT-08.SPEC-011 selects text as the channel.

**FEAT-14.SPEC-005-AC-04:** Given a STOP reply for Riley's relationship with Talia arrives within the same second as her re-grant tap with a later timestamp, when both writes complete, then FEAT-14.SPEC-006 resolves the final state to Revoked.

**FEAT-14.SPEC-005-AC-05:** Given Riley's record is already Granted when this automation fires due to a stale screen state, when the write proceeds, then no error is raised and the state is set to Re-granted without disrupting the client experience.

**FEAT-14.SPEC-005-AC-06:** Given the re-grant write fails due to a processing error, when the failure is reported, then FEAT-14.SPEC-001 shows "Couldn't update your preference. Try again." and the button remains available.

**FEAT-14.SPEC-005-AC-07:** Given Riley's phone number was changed and her consent invalidated per FEAT-14.SPEC-008, when she taps "Turn texting back on", then the write succeeds setting state to Re-granted, but phone_number is left unchanged pending her next booking.

**FEAT-14.SPEC-005-AC-08:** Given this automation never re-captures consent wording, when a record's state changes to Re-granted, then its consent_wording field remains exactly as originally captured by FEAT-14.SPEC-003.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, superseded, write failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Concurrent Consent Update Resolution

## Overview

**Name:** Concurrent Consent Update Resolution
**ID:** FEAT-14.SPEC-006
**Type:** Logic/Rule
**Purpose:** Resolves a STOP reply and an in-app re-grant (or any two consent-changing writes) arriving close together for the same client-Pro relationship, by most-recent-explicit-action timestamp, defaulting to the no-text state when the outcome is uncertain.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The precedence rule between two consent-state-changing writes for the same relationship arriving close together
- The fail-safe default (no-text) applied when precedence cannot be determined
- Which writes are "explicit client actions" subject to this rule, and which are not

**Non-Goals:**
- Performing the writes themselves -- owned by FEAT-14.SPEC-004 (revoke) and FEAT-14.SPEC-005 (re-grant); this spec governs which of their writes is the one that persists, not how each write is made.
- The booking-time creation or update path -- owned by FEAT-14.SPEC-003; a booking submission is always a single, sequential write with no concurrent counterpart in the same instant, so it never enters this spec's conflict scenario.
- Deriving the client-visible textability status from the resolved state -- owned by FEAT-14.SPEC-007, which reads whatever final state this spec's resolution produces.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp reserved for a later phase) |
| state | enum | Granted \| Revoked \| Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-004 | Opt-Out / STOP Processing | Defers to this spec for the final persisted state whenever its revoke write could race a concurrent re-grant |
| FEAT-14.SPEC-005 | Consent Re-Grant Action | Defers to this spec for the final persisted state whenever its re-grant write could race a concurrent revoke |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Reads the state this spec's resolution produces before every client-directed send |
| FEAT-14.SPEC-007 | Textability Determination Rule | Reads the state this spec's resolution produces to compute textability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | No validation beyond data type -- unaffected by conflict resolution | Always | -- | -- | -- |
| state | Must resolve to exactly one of Granted / Revoked / Re-granted after any concurrent-write scenario -- never left ambiguous or partially applied | Always | On every detected concurrent write | N/A -- this is a system-resolved value with no user-facing entry; resolution is automatic | Yes (the record is never left with two pending, unresolved writes) |
| timestamp | Must reflect the winning write's own timestamp, not the moment resolution itself runs | Always | On every detected concurrent write | N/A -- system-set | Yes |
| consent_wording | No validation beyond data type -- never altered by conflict resolution, per FEAT-14.SPEC-004 and FEAT-14.SPEC-005's shared rule that this field is untouched by any state-only write | Always | -- | -- | -- |
| phone_number | No validation beyond data type -- never altered by conflict resolution | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Most-recent-explicit-action precedence | state, timestamp | When two writes (one from FEAT-14.SPEC-004, one from FEAT-14.SPEC-005) target the same Messaging Consent record within a window where both are "in flight" at once, the write with the later timestamp is the one whose state and timestamp persist; the earlier write's state does not persist, even though its own write attempt completed | N/A -- resolved automatically; no user-facing error, since both actions were genuinely taken and one legitimately supersedes the other |
| Fail-safe default on genuine uncertainty | state, timestamp | If the two competing timestamps are indistinguishable (recorded at the same instant, or the order in which they were received cannot be established), the record resolves to Revoked (the no-text state), regardless of which write technically committed last in storage | N/A -- resolved automatically; this is the feature's named Error-state discipline (product-features.md: "if in doubt, the system defaults to the safer (no-text) state") |
| STOP reply always acknowledged regardless of resolution outcome | state | Even when a STOP reply's write is the one that does NOT persist (because a later re-grant superseded it), FEAT-14.SPEC-004's confirmation-notification rule is unaffected by this spec -- the confirmation was already correct at the moment it was sent, describing the state as it stood then | N/A -- notification behavior is owned by FEAT-14.SPEC-004/009, not restated here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger a competing write (revoke or re-grant) that this rule may need to resolve | The Client (Riley) | Own-only -- only the Client whose consent it is (Access Matrix: Messaging & Consent = Own-only for the Client) | -- |
| Trigger a competing write (revoke or re-grant) that this rule may need to resolve | The Pro (Talia) | Never -- the Pro has no path to change a client's consent state (Access Matrix: "the Pro can see but never override") | No control exists anywhere for the Pro to write consent state, so the Pro can never be a party to a conflict this rule resolves |
| Trigger a competing write that this rule may need to resolve | Platform Operator (Support) | Never | Support's View-only access (Access Matrix) never includes a write path, so Support can never be a party to a conflict this rule resolves |
| Read the resolved final state | The Client (Riley) | Own-only, via FEAT-14.SPEC-001 | -- |
| Read the resolved final state | The Pro (Talia) | View-only, for planning, via FEAT-12's dashboard reading FEAT-14.SPEC-007's output | -- |
| Read the resolved final state | Platform Operator (Support) | View-only, for troubleshooting | -- |
| Override or manually force a resolution outcome | Any role | Never -- resolution is fully automatic for every role, including the Pro and Support | No control exists for any role to override the resolved state; the rule applies uniformly regardless of who is asking |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Final persisted state after a detected write conflict | The state and timestamp of whichever of the two competing writes carries the later timestamp; Revoked if the two timestamps are indistinguishable | Whenever FEAT-14.SPEC-004 and FEAT-14.SPEC-005 both target the same record within the same resolution window | No -- fully automatic, no override for any role |

## Business Rules

- XBR-15 and this feature's own Error-state discipline both require the same posture: when a client's current consent is uncertain, the safe default is no-text, never a guess that could send a text without valid consent.
- A conflict this spec resolves is detected only between FEAT-14.SPEC-004's and FEAT-14.SPEC-005's writes -- the two automations that can change an existing record's state after creation. FEAT-14.SPEC-003's booking-time writes are excluded per the Non-Goals above, since they are never concurrent with another write by construction (one client, one sequential booking submission).
- The resolution is silent to the client and the Pro -- neither is shown a "your action was overridden" message; the Client sees only the resulting state on their next view of FEAT-14.SPEC-001, and the Pro sees only the resulting state via FEAT-14.SPEC-007's output on FEAT-12.
- This rule never changes consent_wording or phone_number -- only state and timestamp are subject to resolution, consistent with FEAT-14.SPEC-004 and FEAT-14.SPEC-005 both leaving those fields untouched on their own writes.

## Edge Cases

- **A STOP reply's write and an in-app re-grant's write are both recorded with the exact same timestamp value (down to the finest precision the product records)** -- The two timestamps are treated as indistinguishable; the fail-safe default applies and the record resolves to Revoked, per the Cross-Field Rules' second entry.
- **The re-grant's write technically commits to storage a fraction of a second before the STOP reply's write, but the STOP reply's own timestamp (when the client sent it) is earlier than the re-grant's timestamp (when the client tapped the button)** -- Precedence follows the timestamp of the client's explicit action, not the order in which the writes happened to commit in storage; the re-grant's later action-timestamp wins, even though its write committed second.
- **Only one of the two writes actually occurs (e.g., a STOP reply arrives with no concurrent re-grant at all)** -- There is no conflict to resolve; the single write's state and timestamp simply persist as written by FEAT-14.SPEC-004, and this spec's rule is never invoked.
- **A third write (e.g., a later booking's choice via FEAT-14.SPEC-003) arrives after this spec has already resolved a two-way conflict** -- The prior resolution is now history; FEAT-14.SPEC-003's own write (Business Rules, Non-Goals above) simply updates the record in the normal sequential way, since by the time a new booking is submitted there is no longer a live conflict in progress.
- **The Pro views a client's textability on FEAT-12 at the exact moment a conflict is being resolved** -- The Pro's view reflects whatever state is currently persisted at the moment of the read; if the read happens to land between the two writes, the Pro may briefly see the earlier of the two states, but the very next read after resolution completes shows the final resolved state -- the Pro's read is never itself blocked or delayed waiting for resolution.

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given a STOP reply for Riley's relationship with Talia is written with an earlier timestamp than a concurrent in-app re-grant, when both writes are evaluated, then the record resolves to Re-granted with the re-grant's timestamp.

**FEAT-14.SPEC-006-AC-02:** Given an in-app re-grant is written with an earlier timestamp than a concurrent STOP reply for the same relationship, when both writes are evaluated, then the record resolves to Revoked with the STOP reply's timestamp.

**FEAT-14.SPEC-006-AC-03:** Given the two competing writes' timestamps are indistinguishable, when resolution runs, then the record resolves to Revoked regardless of which write committed to storage last.

**FEAT-14.SPEC-006-AC-04:** Given only a STOP reply arrives with no concurrent re-grant, when it is processed, then no conflict resolution is invoked and the STOP reply's write persists as-is.

**FEAT-14.SPEC-006-AC-05:** Given a re-grant's write commits to storage before a STOP reply's write, but the STOP reply's own action-timestamp is later, when resolution runs, then the STOP reply's later action-timestamp wins and the record resolves to Revoked.

**FEAT-14.SPEC-006-AC-06:** Given Riley (the Client) is the only role that can trigger either competing write, when Talia (the Pro) looks for any way to influence the outcome, then no such control exists for her.

**FEAT-14.SPEC-006-AC-07:** Given Support views a resolved consent record, when they look for an override control, then none exists -- Support sees the resolved state read-only.

**FEAT-14.SPEC-006-AC-08:** Given a conflict resolves to Revoked, when Riley next views FEAT-14.SPEC-001, then she sees "Texting: off" with no explanation that her re-grant tap was overridden.

**FEAT-14.SPEC-006-AC-09:** Given a conflict is resolved, when the record's consent_wording and phone_number are inspected, then both remain exactly as they were before the conflicting writes -- only state and timestamp changed.

**FEAT-14.SPEC-006-AC-10:** Given a resolution favors the re-grant, when FEAT-08.SPEC-011 next evaluates the channel for a message to Riley, then it selects text, consistent with the resolved Re-granted state.

**FEAT-14.SPEC-006-AC-11:** Given Talia's FEAT-12 dashboard reads Riley's textability at the exact moment a conflict is mid-resolution, when her next read occurs immediately after, then it reflects the final resolved state, with the earlier read never blocked or delayed by the resolution itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Textability Determination Rule

## Overview

**Name:** Textability Determination Rule
**ID:** FEAT-14.SPEC-007
**Type:** Logic/Rule
**Purpose:** Computes, as a single authoritative answer, whether a given client is currently textable for a given Pro relationship -- the value every other feature reads instead of re-deriving consent logic itself.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The textable / not-textable determination for a single Client-Pro relationship, evaluated fresh on every read
- The conditions under which the determination is true (active consent, matching phone number) or false (everything else)
- Making this determination the single source of truth every consuming spec reads rather than re-implements

**Non-Goals:**
- Deciding what a consumer does with the result (choosing text vs. email for a send, or displaying a status label) -- owned by each consuming spec: FEAT-08.SPEC-011 for channel selection, FEAT-12 for the Pro's planning display, FEAT-14.SPEC-001 for the client's own status view.
- Writing or changing the Messaging Consent record -- owned by FEAT-14.SPEC-003, FEAT-14.SPEC-004, and FEAT-14.SPEC-005; this spec only reads the record, never modifies it.
- Resolving a race between two writes to the record -- owned by FEAT-14.SPEC-006; this spec always reads whatever state that resolution (or the absence of any conflict) has already settled on.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp reserved for a later phase) |
| state | enum | Granted \| Revoked \| Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

**Referenced (read-only):** Client -- phone, to compare against the consent record's phone_number.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-001 | Consent & Preferences | Reads this determination to render the client's own status line |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | Reads this determination indirectly through FEAT-14.SPEC-004's outcome to know whether the revoke was a no-op |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Reads this determination before every client-directed send in FEAT-08 to choose text or email |
| FEAT-12 | Pro Daily Schedule Dashboard (Pro's planning view) | Reads this determination, View-only, so the Pro can see whether a client can currently be texted |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | Must equal "text" for this determination to consider consent applicable at all | Always | On every determination | N/A -- read-only evaluation, no user-facing error; a non-text channel simply evaluates as not-textable-by-text | No |
| state | Must be Granted or Re-granted for the determination to resolve true; Revoked resolves false | Always | On every determination | N/A -- read-only evaluation | No |
| timestamp | No validation beyond data type -- not itself part of the true/false determination, only relevant to FEAT-14.SPEC-006's prior resolution of which state is current | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- irrelevant to the determination itself | Always | -- | -- | -- |
| phone_number | Must match the Client's current phone field for the determination to resolve true; a mismatch resolves false regardless of state | Always | On every determination | N/A -- read-only evaluation | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Textability gate | state, phone_number, channel | Textable = true only when state is Granted or Re-granted, AND phone_number matches the Client's current phone, AND channel is text; otherwise Textable = false | N/A -- a computed boolean, never a user-facing error |
| No-record default | (absence of a Messaging Consent record) | If no Messaging Consent record exists yet for the relationship (a state that should not occur once FEAT-14.SPEC-003 has run at first booking, but is defined for completeness), Textable = false | N/A -- defaults to the safe no-text answer |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read the textability determination for a relationship | The Pro (Talia) | Read-only, own clients only, for planning (Access Matrix: Messaging & Consent = View for the Pro) | -- |
| Read the textability determination for a relationship | The Client (Riley) | Own-only, via FEAT-14.SPEC-001's status display | -- |
| Read the textability determination for a relationship | Platform Operator (Support) | View-only, for troubleshooting | -- |
| Read the textability determination for a relationship | Any other feature's automation or notification spec (e.g., FEAT-08.SPEC-011) | Always -- this determination is the shared read surface every client-directed send consults | -- |
| Override the computed determination for a single send or view | The Pro (Talia) | Never -- the Pro cannot force a "textable" answer against a client's revoked consent, even for her own client | No override control exists anywhere in the product; the determination is fully automatic and non-negotiable |
| Override the computed determination for a single send or view | Platform Operator (Support) | Never | No override control exists for Support under any circumstance |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Textable (boolean) | Derived: true when Messaging Consent.state is Granted or Re-granted AND Messaging Consent.phone_number matches the Client's current phone AND channel is text; false in every other case, including when no record exists | Evaluated fresh on every read -- never cached from a prior evaluation or from booking time | No -- fully automatic, no override for any role |

## Business Rules

- XBR-15 governs this determination entirely: no text is sent without active consent for that client and Pro; otherwise email is used; a revoke is honored on the very next message; a changed phone number requires fresh consent -- every one of these is a direct restatement of the Textability gate above.
- This spec is the single source of truth every other spec and feature (FEAT-08, FEAT-12) reads rather than re-deriving the state itself, per the Brief's Shared Validation section; a consuming spec that computed its own version of this logic independently would risk drifting from this rule over time.
- The determination is evaluated fresh at the moment of each read, never cached -- this is the mechanism by which a revoke is honored on the very next message: there is no stale "textable" answer left over from before a revoke.
- FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) lists this rule as an enforcer, and this rule in turn enforces FEAT-14.SPEC-008: the phone-number-match condition here is how an invalidated consent immediately evaluates as not textable, with no separate write required.
- Re-granted is treated identically to Granted -- textability is a two-state answer (yes/no), not a three-tier reflection of the underlying three-state consent model.

## Edge Cases

- **A client has Granted consent but no phone number on file (a data inconsistency that should not occur given FEAT-05's capture flow, which requires a phone number to create any Client record)** -- The phone_number match condition cannot be satisfied against an empty Client.phone, so Textable resolves false; the client is never left in an undefined state, and the consuming spec (FEAT-08.SPEC-011) routes to email.
- **A client's phone number changes mid-session while a determination is being read for a message already queued to send** -- The determination re-evaluates at the moment of the actual read (send time), not at queue time, so a number change is reflected correctly even for an in-flight queued message.
- **FEAT-14.SPEC-006 is mid-resolution of a conflicting pair of writes at the exact moment this rule is evaluated** -- The determination reads whatever state is currently persisted at that instant; if the read lands between the two writes, it may reflect the earlier of the two states, but the very next read after resolution completes reflects the final resolved state -- this spec does not itself wait for or participate in that resolution.
- **The consent record's channel is a value other than text (a future WhatsApp-consented record, per the entity's reserved field)** -- Textable (for the text channel this feature governs) resolves false, since the Textability gate requires channel to be text; a WhatsApp-specific determination is a distinct concern the entity's channel field reserves for FEAT-26's later phase, not something this spec computes.
- **No Messaging Consent record exists at all for the relationship being queried (an integration error elsewhere, since FEAT-14.SPEC-003 should always create one at first booking)** -- Per the No-record default cross-field rule, Textable resolves false; a missing record is treated identically to a Revoked one rather than causing an error or an assumed-true default.

## Acceptance Criteria

**FEAT-14.SPEC-007-AC-01:** Given Riley's Messaging Consent with Talia is Granted and her phone number matches the record, when this rule is evaluated, then it returns Textable = true.

**FEAT-14.SPEC-007-AC-02:** Given Riley's Messaging Consent with Talia is Revoked, when this rule is evaluated, then it returns Textable = false.

**FEAT-14.SPEC-007-AC-03:** Given Riley's Messaging Consent state is Re-granted, when this rule is evaluated, then it returns Textable = true, identical to a Granted state.

**FEAT-14.SPEC-007-AC-04:** Given Riley's phone number no longer matches her consent record's phone_number (a recent change), when this rule is evaluated, then it returns Textable = false regardless of the state field.

**FEAT-14.SPEC-007-AC-05:** Given no Messaging Consent record exists at all for a queried relationship, when this rule is evaluated, then it returns Textable = false.

**FEAT-14.SPEC-007-AC-06:** Given Talia (the Pro) views her dashboard, when FEAT-12 reads this rule's output for one of her clients, then she sees the current textability with no control to override it.

**FEAT-14.SPEC-007-AC-07:** Given FEAT-08.SPEC-011 is about to select a channel for a message to Riley, when it reads this rule, then it receives a freshly evaluated result, never a cached value from an earlier point in the session.

**FEAT-14.SPEC-007-AC-08:** Given Riley's consent record's channel field holds a non-text value, when this rule is evaluated for the text channel, then it returns Textable = false for texting purposes.

**FEAT-14.SPEC-007-AC-09:** Given a read of this rule happens to land between two writes that FEAT-14.SPEC-006 is resolving, when the read completes, then it reflects whichever state is currently persisted at that instant, without waiting for resolution to finish.

**FEAT-14.SPEC-007-AC-10:** Given Support views a client's textability for troubleshooting, when they look for a way to change the underlying determination, then no such control exists -- the view is read-only.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Phone Number Change Consent Invalidation Rule

## Overview

**Name:** Phone Number Change Consent Invalidation Rule
**ID:** FEAT-14.SPEC-008
**Type:** Logic/Rule
**Purpose:** Invalidates a client's existing texting consent whenever their phone number changes, so fresh consent is required before the new number is ever texted.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- Detecting a Client phone-number change and invalidating the affected Messaging Consent record
- Defining what "invalidated" means for the record's fields and for downstream textability
- The relationship between this rule and the fresh-consent capture that follows

**Non-Goals:**
- Changing the Client's phone number itself -- owned by FEAT-13.SPEC-002 (Client Contact Edit); this spec only reacts to a change that has already been saved there.
- Invalidating Access Links on a phone-number change -- owned by FEAT-06 (Client Booking Identity), per the dependency map's Client Contention note; this spec covers Messaging Consent only, a distinct entity with a distinct rule.
- Capturing the fresh consent required after invalidation -- owned by FEAT-14.SPEC-003, which this rule's invalidated record simply makes eligible for a first-time-style capture again.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp reserved for a later phase) |
| state | enum | Granted \| Revoked \| Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

**Referenced (read-only):** Client -- phone, the field whose change triggers this rule.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-002 | Client Contact Edit | Saving a changed phone number is the event this rule reacts to |
| FEAT-14.SPEC-003 | Consent Capture at Booking | Reads the invalidated state as its starting point when the client's next booking captures fresh consent |
| FEAT-14.SPEC-007 | Textability Determination Rule | Reflects the invalidation immediately, since its phone-number-match condition already fails once the Client's phone no longer matches the consent record |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | No validation beyond data type -- unaffected by invalidation | Always | -- | -- | -- |
| state | Not directly changed by this rule -- invalidation is expressed through the phone_number mismatch (see Cross-Field Rules), not by forcing state to Revoked | Always | -- | -- | -- |
| timestamp | Not updated by this rule -- the original grant/revoke timestamp remains historically accurate; invalidation is a separate, derived condition layered on top | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- untouched by a phone-number change; it remains evidence of what was shown for the old number | Always | -- | -- | -- |
| phone_number | Left unchanged on the Messaging Consent record itself when the Client's phone changes -- the record's phone_number continues to reflect the number the original consent applied to, which is now stale relative to Client.phone | On every Client phone-number change | On Client phone-number save (FEAT-13.SPEC-002) | N/A -- no user-facing error; the mismatch this creates is exactly the mechanism that expresses invalidation | No (this is the intended effect, not a rejected value) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Invalidation-by-mismatch | Messaging Consent.phone_number, Client.phone | The moment Client.phone changes and no longer equals the Messaging Consent record's phone_number, the record is treated as not applicable to the client's current number -- FEAT-14.SPEC-007's textability determination resolves false for it, exactly as it would for a Revoked record, without this rule needing to write a new state value | N/A -- the mismatch condition itself is the invalidation; no separate write is required |
| Fresh-consent eligibility | Messaging Consent.phone_number, Client.phone | Once invalidated by mismatch, the relationship becomes eligible for FEAT-14.SPEC-003 to capture fresh consent at the client's next booking, which updates phone_number to the current number alongside the new state | N/A -- this is the designed recovery path, not an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change a client's phone number (the event this rule reacts to) | The Pro (Talia) | Full, on her own clients only (Access Matrix: Client Records = Full for the Pro), via FEAT-13.SPEC-002 | -- |
| Change a client's phone number | The Client (Riley) | Never -- phone is the client's identity key and is not directly editable by the client themselves (product-features.md, Client entity); only the Pro corrects it | Phone number field is not offered as editable anywhere in the client-facing product |
| Trigger or waive this invalidation rule directly | Any role | Never -- invalidation is automatic and unconditional whenever a phone-number change is saved; no role can opt a client out of needing fresh consent after a number change | No control exists for any role to bypass the fresh-consent requirement after a phone-number change, since it is a US SMS-consent requirement (ASMP-24), not a product preference |
| Read whether a relationship's consent is currently invalidated by a phone-number mismatch | The Pro (Talia) | View-only, via FEAT-14.SPEC-007's textability output on FEAT-12 | -- |
| Read whether a relationship's consent is currently invalidated by a phone-number mismatch | The Client (Riley) | Own-only, via FEAT-14.SPEC-001, where an invalidated relationship simply shows "Texting: off" like any other revoked state | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective textability of an invalidated relationship | Derived to false via FEAT-14.SPEC-007's phone_number-match condition, without any direct write to this rule's governed record | Continuously, from the moment Client.phone changes until fresh consent is captured | No |

## Business Rules

- XBR-15: a changed phone number requires fresh consent -- this rule is that requirement's entire mechanism, expressed as a mismatch condition rather than a forced state change, so the historical record of the original consent (its timestamp and wording) is preserved unaltered as evidence of what was once agreed for the old number.
- Invalidation is silent to both the Pro and the Client at the moment it happens -- no notification fires from this rule itself; the Client simply sees "Texting: off" on their next view of FEAT-14.SPEC-001, and the Pro sees the client as not currently textable on FEAT-12, both through the ordinary textability read path.
- This rule never deletes or overwrites the prior consent evidence -- consent_wording, the original timestamp, and the old phone_number all remain on the record exactly as captured, satisfying the retention expectation in scope-boundaries.md SC-22 alongside the fresh-consent requirement.
- The client re-gains texting only through the ordinary capture paths that already exist -- a subsequent booking (FEAT-14.SPEC-003) or, once the Pro's phone-number edit has propagated, an in-app re-grant (FEAT-14.SPEC-005) -- this rule creates no new consent-capture surface of its own.

## Edge Cases

- **The Pro corrects a typo in the phone number that does not actually represent a different real-world number (e.g., fixing a transposed digit for the same client)** -- The rule cannot distinguish a typo fix from a genuine number change; it applies invalidation uniformly to any saved change in the Client.phone value, per XBR-15's plain requirement, even though the practical risk is the same in both cases -- fresh consent is required either way.
- **The client re-grants texting via FEAT-14.SPEC-005 for a relationship whose consent is currently invalidated by a phone-number mismatch** -- The write succeeds and sets state to Re-granted, but phone_number remains at its old, stale value (per FEAT-14.SPEC-005's own scope), so the mismatch persists and FEAT-14.SPEC-007 still resolves Textable = false until a booking (FEAT-14.SPEC-003) updates phone_number to the current number.
- **The Pro changes the phone number and then changes it back to the original number within the same session** -- Each save is evaluated independently; after the second save, Client.phone once again equals the Messaging Consent record's phone_number, so the mismatch condition no longer holds and textability reflects the record's state field again as if no interruption occurred -- there is no "invalidation history" that persists once the numbers realign.
- **A phone-number change happens while a message is already queued to send to the old number** -- FEAT-14.SPEC-007's fresh-at-read-time evaluation (not cached from queue time) means the send, when it actually goes out, correctly finds the mismatch and routes to email, consistent with FEAT-08.SPEC-011's send-time channel decision.
- **The client has never had an existing Messaging Consent record at all when their phone number changes (a client who has never texted-opted-in at any booking)** -- There is nothing to invalidate; this rule has no effect, and the client's status continues to reflect whatever their prior state already was (Revoked from their original booking-time choice).

## Acceptance Criteria

**FEAT-14.SPEC-008-AC-01:** Given Talia corrects Riley's phone number on FEAT-13.SPEC-002, when the change saves, then Riley's existing Messaging Consent record's phone_number no longer matches her Client.phone.

**FEAT-14.SPEC-008-AC-02:** Given Riley's phone number was just changed, when FEAT-14.SPEC-007 evaluates her textability, then it returns Textable = false, even if her consent state field still reads Granted.

**FEAT-14.SPEC-008-AC-03:** Given Riley's phone number changed and her consent is now invalidated, when she books again with Talia and opts in, then FEAT-14.SPEC-003 captures fresh consent, updating phone_number to the current number.

**FEAT-14.SPEC-008-AC-04:** Given Riley's phone number changed, when the invalidation occurs, then her original consent_wording and timestamp remain unchanged on the record.

**FEAT-14.SPEC-008-AC-05:** Given Riley (the Client) looks for a way to change her own phone number in the product, when she reviews her available settings, then no such control exists -- only Talia can change it via FEAT-13.SPEC-002.

**FEAT-14.SPEC-008-AC-06:** Given Talia fixes a typo in Riley's phone number that represents the same real number, when the change saves, then the rule still invalidates Riley's consent, since the rule cannot distinguish a typo fix from a genuine change.

**FEAT-14.SPEC-008-AC-07:** Given Riley's consent is invalidated by a phone-number mismatch, when she taps "Turn texting back on" on FEAT-14.SPEC-001, then the write sets state to Re-granted but the mismatch persists until her next booking updates phone_number.

**FEAT-14.SPEC-008-AC-08:** Given Talia changes Riley's phone number and then reverts it to the original value in the same session, when the second save completes, then the mismatch no longer exists and Riley's textability reflects her state field as before.

**FEAT-14.SPEC-008-AC-09:** Given Riley has never had a Messaging Consent record at all, when her phone number changes, then this rule has no effect, since there is no record to invalidate.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Opt-Out Confirmation Message

## Overview

**Name:** Opt-Out Confirmation Message
**ID:** FEAT-14.SPEC-009
**Type:** Notification
**Purpose:** Acknowledges to a client, immediately after a STOP reply is processed, that texting has been turned off and that their confirmations and reminders will now arrive by email instead.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- The confirmation sent after FEAT-14.SPEC-004 processes a STOP text reply
- Both channels this confirmation uses and the rationale for each

**Non-Goals:**
- A confirmation for the link-tap opt-out path -- excluded per FEAT-14.SPEC-004's own Business Rules: FEAT-14.SPEC-002's landing page is itself the confirmation for that path, so this notification exists only for the STOP-reply path, which has no screen to land on.
- Any message beyond this single acknowledgment -- this feature sends nothing else of its own (product-features.md, Communications: "N/A -- this feature governs communications rather than sending its own, aside from an opt-out confirmation acknowledgment"); ongoing confirmations and reminders remain FEAT-08's responsibility once routed to email.
- A re-grant confirmation -- excluded per this feature's Key Capabilities: only the opt-out path names a confirmation message; a re-grant's feedback is delivered in-app by FEAT-14.SPEC-001, not as a separate message.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | Always, immediately after the STOP reply is processed | This confirmation is the direct reply completing the exchange the client themselves just initiated by texting STOP -- it is the terminal message in that same reply thread, not a new proactively-initiated text subject to the ongoing consent gate FEAT-08.SPEC-011 applies to future messages; the client is reachable at the exact number and moment they just texted from |
| Email | Always, sent alongside the text reply, when the client has an email address on file for this Pro | Every future message for this relationship will arrive by email (per XBR-15, texting consent is now Revoked); sending this specific confirmation by email too means the confirmation itself models the channel the client will actually experience going forward, and reaches the client even if the text reply is not seen for any reason |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| STOP reply processed | FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Fires only for the STOP-reply trigger path of that automation, after its revoke write completes (including the already-revoked no-op case) | The replying phone number, the Pro's display_name, the Client's email (if on file), the processing timestamp |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the client whose STOP reply this confirms, per the Access Matrix's Messaging & Consent = Own-only for the Client. No other role receives this confirmation; the Pro is not a party to it, consistent with the Pro's View-only relationship to a client's consent.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this confirmation has no independent opt-out or channel preference of its own | -- | -- | N/A -- suppressing the acknowledgment of a client's own STOP action would leave them uncertain whether their opt-out took effect, so this confirmation is never itself subject to a preference toggle |

**Quiet Hours:** N/A -- this confirmation is a direct, immediate reply to the client's own just-sent message, not a scheduled or ambient notification; it is exempt from any quiet-hours window in the same way a reply to an inbound message would be, since holding it would leave the client uncertain their STOP was received.

## Content Definition

**Text:**
- **Body:** You're unsubscribed from texts from {pro_display_name}. Your confirmations and reminders go to your email instead.
- **CTA:** None -- this is a terminal acknowledgment with no action to take.

**Email:**
- **Subject:** Texting turned off for {pro_display_name}
- **Body:**
  Hi,

  You replied STOP, so texting is now turned off for your bookings with {pro_display_name}.

  Your confirmations and reminders go to your email instead. If you'd like to turn texting back on later, you can do so from your booking preferences.
- **CTA:** None -- this is a terminal acknowledgment with no action to take.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia's Studio | Never empty -- display_name is a required field on every live Pro Account (product-features.md, Pro Account fields) |

## Delivery Rules

**Batching:** None -- exactly one confirmation is sent per STOP-reply processing event; there is no scenario in which multiple pending confirmations for the same client-Pro relationship could accumulate, since FEAT-14.SPEC-004's already-revoked no-op case still fires exactly one confirmation per reply received, not a suppressed or merged one.
**Deduplication:** One confirmation per processed STOP reply. A second STOP reply from the same client, even moments later, produces its own separate confirmation (per FEAT-14.SPEC-004's Business Rules: a redundant STOP still deserves acknowledgment); this is intentional non-deduplication, not a gap.
**Retry on failure:** Text delivery failure is retried per the product's standard message-delivery discipline (platform parameter: `message-delivery-retry-count`); after the final failure, the email send (already dispatched independently, not as a fallback of the text) remains the delivery of record, since both channels are sent in parallel, not in sequence. Email delivery failure has no further fallback beyond its own standard retry, since Revoked consent removes text as an alternate channel for this client going forward.
**Expiry:** This confirmation never expires unsent -- because it is a direct, immediate reply to an event that already fully completed (the revoke write), there is no future point at which sending it becomes moot; if both channels are ultimately undeliverable, the client's opt-out itself is still fully in effect regardless, since FEAT-14.SPEC-004's revoke write does not depend on this notification succeeding.

## Edge Cases

- **The client has no email address on file for this Pro** -- Only the text confirmation is sent; the email leg is simply skipped, since there is no address to send to. This is consistent with product-features.md's rule that email is required only when texts are declined at booking -- a client opting out later by STOP may not have provided one, and the text confirmation alone still fulfills the acknowledgment.
- **A second STOP reply arrives moments after the first, while the first confirmation is still being delivered** -- Per Deduplication above, a second, separate confirmation is sent for the second reply; both are legitimate acknowledgments of two real STOP actions, even if redundant from the client's perspective.
- **The underlying Messaging Consent record is deleted between the STOP reply's processing and this notification's send (a client deletion request, FEAT-13/XBR-19, arriving in the same narrow window)** -- The confirmation still sends using the phone number and email captured at trigger time, since the acknowledgment concerns the STOP action itself, which already happened; the now-deleted record does not retroactively cancel a confirmation for an action that was real when it occurred.
- **Text delivery fails but email succeeds (or vice versa)** -- Since both channels are dispatched independently rather than one as a fallback of the other, a failure on one channel has no effect on the other; the client receives the acknowledgment on whichever channel succeeds, and both failing simultaneously leaves the opt-out itself still fully in effect regardless (Delivery Rules, Expiry).
- **Quiet hours (if any exist for other notifications in the product) would otherwise apply to the moment of this send** -- Per the Quiet Hours field above, this confirmation is exempt, since it is the direct reply to the client's own just-sent message, not an ambient or scheduled notification a quiet-hours window is meant to protect against.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Triggered by (inbound) | The STOP-reply path fires this notification after its revoke write completes |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Sends the text leg of this confirmation |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Sends the email leg of this confirmation |
| FEAT-14.SPEC-001 (Consent & Preferences) | References | The client's next view of this screen reflects the same Revoked status this confirmation describes |

## Analytics and Success Signals

- **opt_out_confirmation_sent** (channel: text / email; delivery_status) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references opt-out acknowledgment; retained as an operational signal so this confirmation's delivery reliability is observable.
- **opt_out_confirmation_delivery_failed** (channel) -- N/A -- same reason as above; retained so a silent delivery failure on this compliance-adjacent acknowledgment is never invisible.

## Acceptance Criteria

**FEAT-14.SPEC-009-AC-01:** Given Riley replies "STOP" to a text from Talia and her phone matches her Client record, when FEAT-14.SPEC-004 completes the revoke, then Riley receives a text reading "You're unsubscribed from texts from Talia's Studio. Your confirmations and reminders go to your email instead." and an email with the subject "Texting turned off for Talia's Studio", given she has an email on file.

**FEAT-14.SPEC-009-AC-02:** Given Riley has no email address on file for Talia, when the confirmation fires, then only the text leg is sent and no email is attempted.

**FEAT-14.SPEC-009-AC-03:** Given Riley's consent was already Revoked before this STOP reply (a redundant STOP), when FEAT-14.SPEC-004 processes it as a no-op, then this notification still fires and Riley receives the same confirmation.

**FEAT-14.SPEC-009-AC-04:** Given Riley texts "STOP" twice within moments, when both replies are processed, then two separate confirmations are sent, one per reply.

**FEAT-14.SPEC-009-AC-05:** Given the text leg of this confirmation fails to deliver, when the failure is reported, then the email leg, already sent independently, is unaffected and remains the delivery Riley receives.

**FEAT-14.SPEC-009-AC-06:** Given both the text and email legs of this confirmation fail to deliver, when the failures are reported, then Riley's underlying opt-out remains fully in effect regardless, since FEAT-14.SPEC-004's revoke write does not depend on this notification's success.

**FEAT-14.SPEC-009-AC-07:** Given Riley opts out via the link-tap path instead of a STOP reply, when FEAT-14.SPEC-004 processes it, then this notification does not fire, since FEAT-14.SPEC-002's landing page is itself the confirmation for that path.

**FEAT-14.SPEC-009-AC-08:** Given the product defines no preference to suppress this confirmation, when Riley looks for a way to turn it off, then no such control exists anywhere in the product.

**FEAT-14.SPEC-009-AC-09:** Given the confirmation is sent during hours that would otherwise be within any quiet-hours window defined elsewhere in the product, when the STOP reply is processed, then the confirmation sends immediately regardless, per its quiet-hours exemption.

**FEAT-14.SPEC-009-AC-10:** Given Riley's Client record is deleted in the narrow window between her STOP reply's processing and this notification's send, when the notification fires, then it still sends using the phone number and email captured at trigger time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference exists, confirmed absent) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |


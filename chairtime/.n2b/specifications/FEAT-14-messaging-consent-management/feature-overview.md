---
document_type: feature-overview
feature_number: FEAT-14
feature_name: Messaging Consent Management
feature_slug: messaging-consent-management
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 9
screen_count: 2
automation_count: 3
logic_rule_count: 3
integration_count: 0
notification_count: 1
---

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

---
document_type: feature-overview
feature_number: FEAT-26
feature_name: WhatsApp Reminders
feature_slug: whatsapp-reminders
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 1
automation_count: 1
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

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

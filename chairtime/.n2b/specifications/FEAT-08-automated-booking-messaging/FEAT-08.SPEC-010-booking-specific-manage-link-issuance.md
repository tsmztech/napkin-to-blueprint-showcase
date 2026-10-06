---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-010
spec_name: Booking-Specific Manage Link Issuance
spec_slug: booking-specific-manage-link-issuance
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Booking-Specific Manage Link Issuance

## Overview

**Name:** Booking-Specific Manage Link Issuance
**ID:** FEAT-08.SPEC-010
**Type:** Automation
**Purpose:** Mints the booking-specific manage link carried in every confirmation and reminder, scoped to exactly one booking, and reissues a fresh one whenever the Pro reschedules that booking.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Creating a booking-specific Access Link at confirmation send and at reminder send
- Reissuing a fresh link after a Pro-initiated reschedule
- Enforcing the link's single-booking scope and its automatic expiry once the appointment passes

**Non-Goals:**
- Resolving or redeeming a tapped link (checking Issued/Used/Expired state, opening the actual booking view) -- owned by FEAT-06 (Client Booking Identity); this spec only mints the link, per the Entity-Lifecycle Coverage Matrix's explicit division of ownership.
- The on-demand "my bookings" link a returning client requests -- owned by FEAT-06; that link has a different scope (all of a client's bookings with one Pro) and a different expiry (30 minutes, single-use), distinct from this spec's booking-specific, until-appointment-passes link.
- Issuing a fresh link after a client-initiated reschedule -- product-features.md and the Brief's Entity-Lifecycle Coverage Matrix name only the Pro-initiated reschedule (via FEAT-30) as triggering reissuance; a client-initiated reschedule (FEAT-10) is itself reached through an already-valid link and does not need a new one issued mid-flow.
- Deciding which message a link is embedded in -- owned by FEAT-08.SPEC-001, FEAT-08.SPEC-002, and FEAT-08.SPEC-004, each of which requests a link from this automation when composing their content.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A confirmation is about to be sent | FEAT-08.SPEC-001 (Booking Confirmation Message) | Fires immediately before the confirmation's content is composed | Booking reference |
| A reminder is about to be sent | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires immediately before the reminder's content is composed | Booking reference |
| A Pro-initiated reschedule occurs | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) | Fires whenever the Pro reschedules a client's booking to a new time | Booking reference (with updated start_time) |

## Processing Logic

1. Receive a request to mint (or reissue) a manage link for a specific Booking.
2. Check whether an active, unexpired Access Link already exists for this Booking (for example, a link already issued at confirmation time, still valid when the reminder later needs one).
3. If an active link already exists and the request is not a reschedule-triggered reissuance, reuse the existing link rather than minting a duplicate -- the confirmation and reminder for the same still-unrescheduled booking share one link.
4. If no active link exists, or the request is a reschedule-triggered reissuance, create a new Access Link scoped to exactly this one Booking, with its expiry set to the Booking's (possibly newly rescheduled) appointment start_time.
5. On a reschedule-triggered reissuance, mark any prior link for this Booking as superseded, so it no longer resolves (per FEAT-08.SPEC-008's Edge Cases, a tap on a superseded link routes to the Expired/Invalid Link state).
6. Return the resulting link to the requesting spec for embedding in its message content.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link created (first issuance) | No active link exists for this Booking | Access Link created (scope: this Booking; expiry: appointment start_time; state: Issued) | None directly -- the link appears embedded in the requesting message | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Existing link reused | An active, unexpired link for this Booking already exists and no reschedule occurred | None -- the existing Access Link record is returned unchanged | None directly | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Link reissued (Pro reschedule) | FEAT-30 reports a Pro-initiated reschedule for this Booking | The prior Access Link's state is superseded; a new Access Link is created scoped to the same Booking with the new appointment's expiry | The client's next message (the change notice, FEAT-08.SPEC-004) carries the fresh link; the old link stops working | FEAT-08.SPEC-004, FEAT-08.SPEC-008 |
| Automation failure | Link issuance itself fails (e.g., the underlying link-generation step is unavailable) | No Access Link is created or reissued | The requesting message's send is held, per FEAT-08.SPEC-001/002's Edge Cases, and treated as a send failure under FEAT-08.SPEC-009's retry path once a link becomes available | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-009 |

## Data Model

**Reads:** Booking -- start_time, state.
**Creates:** Access Link -- scope (one Booking), expiry (until the appointment passes), state (Issued).
**Updates:** Access Link -- state set to superseded on a reschedule-triggered reissuance (the prior link for this Booking).
**Deletes:** None -- a superseded or expired link is never deleted; it simply stops resolving, consistent with the Access Link entity's "expires automatically" lifecycle (no delete/archive action exists).

## Business Rules

- XBR-18 governs this spec: booking-specific links stop working once the appointment passes, and a Pro reschedule issues a fresh manage link.
- Exactly one active manage link exists per Booking at any time (excluding the brief moment of transition during a reschedule-triggered reissuance) -- the confirmation and reminder for an unrescheduled booking intentionally share the same link rather than each minting its own, so a client using an earlier message's link after receiving a later one still reaches the same, current booking.
- A booking-specific link's scope is exactly one Booking -- it never grants access to any other booking, even another booking by the same Client with the same Pro, per XBR-18's "access links open only that client's bookings with that Pro" read together with this spec's explicit single-booking scope.
- This link is distinct in kind from FEAT-06's on-demand "my bookings" link: no single-use restriction applies here (Step 3's reuse behavior depends on this), since a client legitimately needs to tap the same link multiple times (once to acknowledge a reminder, again later to check details) without it burning out.

## Edge Cases

- **A reminder is about to be sent for a booking whose confirmation link is still active** -- The reminder reuses the existing link (Step 3); no second link is minted for the same still-current booking.
- **The Pro reschedules a booking twice in quick succession** -- Each reschedule triggers its own reissuance; only the link from the most recent reschedule remains active, and every earlier link (including the original) is superseded.
- **A client taps an old, superseded link after a Pro reschedule** -- Per FEAT-08.SPEC-008's Edge Cases, the tap resolves to the Expired/Invalid Link state, directing the client to their latest message.
- **Concurrent trigger firing (the confirmation and reminder both request a link for the same booking at effectively the same time -- possible only in the rare case of a very-soon appointment)** -- The check-then-create sequence (Steps 2--4) ensures only one Access Link is ultimately created for the booking; the second request's check finds the first request's just-created link and reuses it rather than creating a duplicate.
- **Trigger fires while a previous run is in flight for the same Booking (a reschedule reissuance overlaps with an in-flight confirmation link request)** -- The reissuance is applied after the in-flight request completes, so the confirmation always uses a link that is not immediately stale; if the reschedule's new start_time is already in effect by the time the confirmation's link is embedded, the confirmation uses the freshly reissued link rather than a moment-old superseded one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Requests a link to embed |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Requests a link to embed (two reply-action variants of the same link) |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated reschedule ("Reschedule committed" outcome) triggers reissuance of a fresh manage link (XBR-18) |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Affects (outbound) | Carries the freshly reissued link after a reschedule |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | References (outbound) | Enforces this spec's scope/expiry rules at the moment of a reply tap |
| FEAT-06 (Client Booking Identity) | References (outbound) | Owns resolving and redeeming the link this spec mints |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | Reads the link's Booking scope to identify the booking when a client acts through it |

## Analytics and Success Signals

- **manage_link_issued** (trigger: confirmation / reminder / reschedule_reissuance) -- N/A -- no Stage 2 metric measures link issuance volume directly; it is an enabling mechanism behind "Self-Service Access Success", which measures the client's use of the link, not its minting.
- **manage_link_superseded** (reason: pro_reschedule) -- N/A -- no Stage 2 metric tracks link supersession; retained so the reschedule-driven reissuance path (XBR-18) is observable end to end.

## Acceptance Criteria

**FEAT-08.SPEC-010-AC-01:** Given Riley's booking has no active manage link, when her confirmation is about to send, then a new Access Link is created scoped to that one booking, expiring when the appointment passes.

**FEAT-08.SPEC-010-AC-02:** Given Riley's confirmation link is still active when her reminder is about to send, when the reminder is composed, then it reuses the existing link rather than minting a new one.

**FEAT-08.SPEC-010-AC-03:** Given the Pro reschedules Riley's booking, when the reschedule completes, then the prior link is superseded and a new Access Link is created scoped to the same booking with the new appointment's expiry.

**FEAT-08.SPEC-010-AC-04:** Given Riley taps her original manage link after the Pro rescheduled her booking, when the link resolves, then she reaches FEAT-08.SPEC-003's (or FEAT-06's) Expired/Invalid state rather than the current booking.

**FEAT-08.SPEC-010-AC-05:** Given Riley's booking-specific link is scoped only to her one booking, when the link is inspected by any spec, then it never resolves to any other booking, even another one of Riley's own bookings with the same Pro.

**FEAT-08.SPEC-010-AC-06:** Given Riley's appointment has passed, when anyone taps her booking-specific link, then it no longer resolves, per its automatic expiry.

**FEAT-08.SPEC-010-AC-07:** Given the Pro reschedules the same booking twice in quick succession, when both reschedules complete, then only the link from the second (most recent) reschedule remains active.

**FEAT-08.SPEC-010-AC-08:** Given link issuance itself fails at confirmation time, when the confirmation attempts to compose, then the send is held and treated as a delivery failure once a link becomes available, per FEAT-08.SPEC-009.

**FEAT-08.SPEC-010-AC-09:** Given both a confirmation and a reminder request a link for the same booking at effectively the same time, when both requests are processed, then only one Access Link is created and both messages reference it.

**FEAT-08.SPEC-010-AC-10:** Given Riley taps her manage link twice (once to acknowledge a reminder, once later to check details), when each tap resolves, then both succeed against the same still-active link, since this link is not single-use.

**FEAT-08.SPEC-010-AC-11:** Given a Pro-initiated reschedule reissuance overlaps with an in-flight confirmation link request for the same booking, when both complete, then the confirmation ultimately embeds the freshly reissued link, not a moment-old superseded one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (confirmation, reminder, Pro reschedule) | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

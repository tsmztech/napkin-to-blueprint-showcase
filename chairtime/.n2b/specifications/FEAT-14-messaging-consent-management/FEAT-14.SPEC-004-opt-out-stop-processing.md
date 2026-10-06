---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-004
spec_name: Opt-Out / STOP Processing
spec_slug: opt-out-stop-processing
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

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

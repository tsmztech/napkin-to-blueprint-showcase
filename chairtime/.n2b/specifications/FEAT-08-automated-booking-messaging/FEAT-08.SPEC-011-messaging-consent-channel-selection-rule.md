---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-08.SPEC-011
spec_name: Messaging Consent & Channel Selection Rule
spec_slug: messaging-consent-channel-selection-rule
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Messaging Consent & Channel Selection Rule

## Overview

**Name:** Messaging Consent & Channel Selection Rule
**ID:** FEAT-08.SPEC-011
**Type:** Logic/Rule
**Purpose:** Decides text vs. email for every outbound client-directed message this feature sends, based on the client's active Messaging Consent, honoring a revoke on the very next message and requiring fresh consent after a phone number change.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The channel-selection decision (text vs. email) applied before every client-directed send in this feature
- Re-checking consent at send time so a same-session revoke is honored immediately
- The fresh-consent requirement after a client's phone number changes

**Non-Goals:**
- Capturing consent for the first time -- owned by FEAT-05 (Public Booking Page & Booking Flow), which writes the initial Messaging Consent record at booking.
- Processing a revoke itself (a STOP reply or an opt-out link tap) -- owned by FEAT-14 (Messaging Consent Management), which updates the Messaging Consent record; this spec only reads the resulting state.
- Channel selection for Pro-directed notifications (FEAT-08.SPEC-005, FEAT-08.SPEC-006) -- those are governed by the Pro's own notification_preferences (FEAT-27), a distinct preference from client texting consent, since the Pro is never subject to SMS-consent rules for her own account's alerts.
- WhatsApp as a channel option -- deferred to a later phase per scope-boundaries.md's Relevant Deferral Notes; this spec's channel decision is binary (text or email). WhatsApp Reminders (FEAT-26) extends it by running its own eligibility rule, FEAT-26.SPEC-004, before this spec; this spec is reached only when FEAT-26.SPEC-004 finds the send not WhatsApp-eligible.

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
|---------|-----------|-------------------|
| FEAT-08.SPEC-001 | Booking Confirmation Message | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-002 | Appointment Reminder Message | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | Consults this spec's outcome indirectly: a text failure's fallback to email is a delivery-failure path, distinct from this spec's consent-driven channel choice, but both ultimately route through the same email capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | Must be one of the product's defined channels (text; WhatsApp from Later) | Always | On read, before channel decision | N/A -- this field is written by FEAT-05/FEAT-14, not entered directly in this spec's flow | No |
| state | Must be one of Granted / Revoked / Re-granted | Always | On read, before channel decision | N/A -- written by FEAT-05/FEAT-06/FEAT-14 | No |
| timestamp | No validation beyond data type -- system-set, not user-entered in this spec's flow | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- captured verbatim by FEAT-05/FEAT-06 as evidence | Always | -- | -- | -- |
| phone_number | Must match the Client's current phone number for consent to be treated as active for texting | Always | On every send, before selecting text as the channel | N/A -- a mismatch is a silent routing decision (email is used), not a user-facing validation error, since no user is filling out a form at this point | Yes (blocks text; does not block the send itself, which proceeds by email) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Consent-to-channel gate | state, phone_number | Text is used only when state is Granted or Re-granted AND phone_number matches the Client's current phone_number; otherwise email is used | N/A -- this is a routing decision with no user-facing error; the client simply receives the message on the resulting channel |
| Stale-state resolution | state, timestamp | When two consent-state changes could apply (e.g., a STOP reply and an in-app re-grant arriving close together, per the Messaging Consent entity's Contention note), the most recent explicit client action by timestamp wins | N/A -- resolved automatically; if the state is genuinely uncertain, the no-text state applies (FEAT-14's Error state) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read Messaging Consent state (to decide a send's channel) | The Pro (Talia) | Read-only, own clients' consent only, for the Pro's own visibility of textability (per FEAT-12) | -- |
| Read Messaging Consent state (to decide a send's channel) | The Client (Riley) | This spec itself does not surface consent state to the Client directly; the Client's own view/change of their consent is FEAT-06/FEAT-14's screen, not this rule | -- |
| Read Messaging Consent state (to decide a send's channel) | Platform Operator (Support) | View-only, for troubleshooting a delivery issue -- Support never changes consent | -- |
| Change Messaging Consent state (grant, revoke, re-grant) | The Client (Riley) | Own-only -- only the Client whose consent it is may change it (Access Matrix: Messaging & Consent = Own-only for the Client); performed via FEAT-05, FEAT-06, or FEAT-14, never through this spec | Control not exposed anywhere in this spec's flow -- consent changes never happen as a side effect of a message send |
| Change Messaging Consent state (grant, revoke, re-grant) | The Pro (Talia) | Never -- the Pro can see but never override a client's consent (Access Matrix note: "the Pro can see but never override a client's texting consent") | The consent-state field is read-only wherever the Pro views it (FEAT-12); no control to change it is ever shown to the Pro |
| Change Messaging Consent state (grant, revoke, re-grant) | Platform Operator (Support) | Never | No control exists for Support to change consent under any circumstance |
| Override this spec's channel decision for a single send | The Pro (Talia) | Never -- the Pro cannot force a text send against a client's revoked consent, even for her own client | No override control exists anywhere in the product; the channel decision is fully automatic |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Selected channel for a given send | Derived: text if Messaging Consent.state is Granted/Re-granted and phone_number matches the Client's current phone_number; email otherwise | On every client-directed send, evaluated fresh each time (never cached from a prior send) | No -- this is a system-computed routing decision with no user-facing override |

## Business Rules

- XBR-15 governs this spec entirely: no text is sent without active Messaging Consent for that client and Pro; otherwise email is used; a revoke is honored on the very next message; a changed phone number requires fresh consent.
- The channel decision is re-evaluated at send time, not cached from booking time or from a prior message -- this is the mechanism by which "a revoke is honored on the very next message" is satisfied: there is no message in flight that can still go out by text after a revoke, because the decision is made fresh immediately before each send.
- A client who changes their phone number (recorded via FEAT-13, per feature-dependency-map.md's Client Contention note: "a Pro phone-number change invalidates access links and requires fresh texting consent") has their existing Messaging Consent treated as not applicable to the new number until fresh consent is captured for it; every send in the interim routes to email.
- FEAT-14.SPEC-007 (Textability Determination Rule) is the authority for whether a client is textable (XBR-15); this spec's consent-to-channel gate applies that determination at send time and never redefines it.
- FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) is the authority for resolving near-simultaneous consent changes; this spec's stale-state resolution follows it and defaults to no-text when the state is uncertain.
- FEAT-14.SPEC-008 (Phone-Number-Change Consent Invalidation Rule) is the authority for treating consent as not applicable after a phone number change; this spec's phone_number match check enforces it on every send.
- FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) runs before this spec for every client-directed send in FEAT-08.SPEC-001, 002, and 004; a send it finds WhatsApp-eligible never reaches this spec, and any other send proceeds here unchanged.
- Every other Notification spec in this feature (FEAT-08.SPEC-001, 002, 004) defers to this spec's decision rather than each implementing its own channel logic, per the Brief's Shared Validation section.

## Edge Cases

- **A STOP reply and an in-app re-grant arrive within the same second** -- Per the Messaging Consent entity's Contention resolution, the most recent explicit client action by timestamp wins; if the timestamps are genuinely indistinguishable, the no-text (email) state applies, since an uncertain consent state must never risk an unwanted text.
- **The client's phone number changes mid-session while a message is queued to send** -- The channel decision, evaluated at send time (not queue time), correctly routes to email once the number-mismatch condition is detected, even if the message was queued while the old number was still valid.
- **A client has Messaging Consent Granted but no phone number on file (a data inconsistency that should not occur given FEAT-05's capture flow)** -- The phone_number match condition cannot be satisfied, so email is used; the send is never blocked outright, since email is always available as the client provided one at booking (product-features.md: "provide an email address when declining texts").
- **A client revokes consent, and a message that was already in the middle of a text-send attempt when the revoke landed** -- The already-initiated send completes on the channel it started on (this spec governs the decision at the moment of initiating a send, not a mid-flight cancellation of an attempt already underway); the very next message after the revoke is what is guaranteed to honor the new state.
- **Consent state is Re-granted after a prior Revoked state** -- Re-granted is treated identically to Granted for this spec's channel decision -- text becomes available again immediately, consistent with the entity's three-state model treating Re-granted as an active-consent state, not a distinct tier.

## Acceptance Criteria

**FEAT-08.SPEC-011-AC-01:** Given Riley has active Messaging Consent (Granted) and her phone number on file matches her consent record, when any client-directed message in this feature is about to send, then text is selected as the channel.

**FEAT-08.SPEC-011-AC-02:** Given Riley has Revoked her Messaging Consent, when any client-directed message is about to send, then email is selected as the channel.

**FEAT-08.SPEC-011-AC-03:** Given Riley revokes her consent between her confirmation (sent by text) and her reminder, when the reminder is about to send, then it is sent by email, honoring the revoke on the very next message.

**FEAT-08.SPEC-011-AC-04:** Given Riley's phone number changes and fresh consent has not yet been captured for the new number, when a message is about to send, then it routes to email, never to the old or unconsented number.

**FEAT-08.SPEC-011-AC-05:** Given Riley's consent state transitions from Revoked to Re-granted, when the next message is about to send, then text is selected as the channel, since Re-granted is treated as active consent.

**FEAT-08.SPEC-011-AC-06:** Given a STOP reply and an in-app re-grant for the same client arrive within the same second with indistinguishable timestamps, when the channel decision is evaluated, then email is used, since an uncertain state defaults to no-text.

**FEAT-08.SPEC-011-AC-07:** Given Talia (the Pro) views a client's textability on her dashboard, when she looks for a way to override a revoked consent, then no such control exists anywhere in the product.

**FEAT-08.SPEC-011-AC-08:** Given Support is troubleshooting a delivery issue, when Support views the Messaging Consent record, then Support sees the state read-only and has no control to change it.

**FEAT-08.SPEC-011-AC-09:** Given Riley has Granted consent but no phone number on file due to a data inconsistency, when a message is about to send, then it routes to email rather than being blocked outright.

**FEAT-08.SPEC-011-AC-10:** Given every Notification spec in this feature (FEAT-08.SPEC-001, 002, 004) needs to choose a channel, when each composes its send, then each defers to this spec's decision rather than implementing separate channel logic.

**FEAT-08.SPEC-011-AC-11:** Given a text send to Riley has already begun processing at the moment her revoke is recorded, when that specific send completes, then it completes on the channel it started on; the very next message after the revoke is the one guaranteed to honor the new state.

**FEAT-08.SPEC-011-AC-12:** Given Riley (the Client) wants to change her own consent, when she looks for how to do so, then the control exists only in FEAT-06/FEAT-14, never as a side effect of viewing or receiving a message under this spec.

**FEAT-08.SPEC-011-AC-13:** Given a Pro-directed notification (FEAT-08.SPEC-005 or FEAT-08.SPEC-006) needs a channel decision, when it evaluates channels, then it uses the Pro's own notification_preferences (FEAT-27), not this spec's Messaging Consent rule.

**FEAT-08.SPEC-011-AC-14:** Given Riley's consent state is Granted but FEAT-14.SPEC-007 determines she is not textable (for example, her phone number changed per FEAT-14.SPEC-008 and fresh consent is not captured), when a message is about to send, then this spec selects email, applying FEAT-14.SPEC-006/007/008 rather than its own separate determination.

**FEAT-08.SPEC-011-AC-15:** Given Riley is WhatsApp-eligible under FEAT-26.SPEC-004, when a confirmation, reminder, or change notice is about to send, then FEAT-26.SPEC-004 decides first and this spec's text/email decision is not evaluated; given she is not WhatsApp-eligible, then this spec decides text or email as usual.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 8 | 8 |
| Edge Cases | 5 | 5 |

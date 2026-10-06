---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-26.SPEC-004
spec_name: WhatsApp Channel Eligibility & Consent Rule
spec_slug: whatsapp-channel-eligibility-consent-rule
parent_feature: FEAT-26
parent_feature_name: WhatsApp Reminders
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 13
---

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

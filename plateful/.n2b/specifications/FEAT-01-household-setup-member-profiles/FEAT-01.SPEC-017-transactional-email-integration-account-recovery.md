---
document_type: spec
spec_type: integration
spec_id: FEAT-01.SPEC-017
spec_name: Transactional Email Integration (Account & Recovery)
spec_slug: transactional-email-integration-account-recovery
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Integration Spec: Transactional Email Integration (Account & Recovery)

## Overview

**Name:** Transactional Email Integration (Account & Recovery)
**ID:** FEAT-01.SPEC-017
**Type:** Integration
**Purpose:** Sends the account-creation confirmation email and the sign-in recovery email through the transactional email capability.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Sending the account-confirmation email when a new account is created
- Sending the reset-link email when a sign-in reset is requested
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what account data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (safety-report emails, plan-ready email fallback, billing confirmations, export/deletion emails) -- each is owned by the feature whose Communications require it (FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the account-creation and recovery emails named in this feature's own Communications
- The screen mechanics of requesting account creation or a reset -- owned by FEAT-01.SPEC-001 and FEAT-01.SPEC-002; this spec defines only the capability behavior those screens trigger

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Requires transactional email for account sign-up and sign-in recovery" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-01.SPEC-017, covers the account & recovery portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A newly created account receives a confirmation that their account exists | Create an account and sign in | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| An adult who cannot sign in receives a reset link by email | Create an account and sign in (recovery) | FEAT-01.SPEC-002 (Password Recovery) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Account email address | Member Profile -- sign_in (email component) | Account is created | The capability needs a destination address to deliver the confirmation |
| Reset token and its expiry | Derived -- a single-use, time-limited token tied to the account | A sign-in reset is requested | The capability delivers a link the account holder uses to complete the reset |

No other Member Profile or Household field ever leaves the product through this integration. Dietary Rule data, household facts, and any other account holder's data are never included in these emails.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household or Member Profile field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Confirmation email delivered | The capability confirms the account-confirmation email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-01.SPEC-001 already proceeded to setup regardless of delivery, per this feature's non-blocking design | FEAT-01.SPEC-001 |
| Reset email delivered | The capability confirms the reset-link email reached the recipient's inbox | None | None -- FEAT-01.SPEC-002 already shows the neutral confirmation message regardless of delivery status | FEAT-01.SPEC-002 |
| Send failed | The capability reports it could not deliver either email (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; the account-creation or reset-request screen flow is not blocked by this outcome | FEAT-01.SPEC-001, FEAT-01.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | No user-visible effect -- account creation completes and the organiser proceeds to FEAT-01.SPEC-003 regardless of how long the confirmation email takes to send | No user-visible effect -- account creation is not blocked by this capability being unavailable; the confirmation email is queued to send once the capability recovers | No user-visible effect on this screen -- a rejected send (e.g., an invalid-looking address) does not prevent account creation or setup from proceeding, since the email is a courtesy confirmation, not a required verification gate |
| FEAT-01.SPEC-002 (Password Recovery) | No user-visible effect on the request step -- the neutral confirmation message appears regardless of send speed, so timing differences never disclose whether an account exists | No user-visible effect on the request step -- the same neutral confirmation appears; the reset email is queued to send once the capability recovers. If the capability remains down long enough that no reset link ever arrives, the adult sees no error (consistent with never disclosing account existence) and can request again later | N/A -- a rejected send at the request step produces the same neutral confirmation as any other outcome, by design, so no rejection state is ever distinguishable to the user here |

## Consent and Disclosure

- **Account confirmation email disclosure** -- The account-creation confirmation on FEAT-01.SPEC-001 implies, as standard product behavior, that a confirmation email will be sent to the address just provided; no separate consent prompt interrupts account creation, since sending an account holder a message at their own just-provided address is expected behavior for creating an account, not a data-sharing decision requiring a choice.
- **Reset email disclosure** -- The confirmation message on FEAT-01.SPEC-002, "If an account exists for {entered email}, a reset link is on its way. Check your inbox.", is itself the disclosure that an email will be sent to that address if it matches an account.
- **What is never shared** -- Household facts, Dietary Rule data (including any child's allergy information), and any other member's data are never included in the account-confirmation or reset emails; only the account holder's own email address and a system-generated reset token leave the product through this integration.

## Edge Cases

- **Reset email event arrives for an account that was deleted between the request and the send** -- The event is discarded silently; no email is sent for a deleted account, and no user feedback fires, since the requester received the neutral confirmation regardless of outcome.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry is triggered beyond the single retry policy already in effect.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a reset email** -- If the send was not confirmed initiated, the reset request is treated as not yet sent and is queued for retry once the capability recovers; the requester's neutral confirmation message is unaffected either way.
- **An adult requests a reset twice in quick succession before the first email's send confirms** -- Both are queued; per FEAT-01.SPEC-002's business rules, the newer link becomes the valid one, so only the most recent reset token matters even if both emails eventually deliver.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Triggered by (inbound) | Successful account creation triggers the confirmation email |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-01.SPEC-002 (Password Recovery) | Triggered by (inbound) | A reset request triggers the reset-link email |
| FEAT-01.SPEC-002 (Password Recovery) | Affects (outbound) | Degradation behavior and the neutral confirmation message surface here |

## Analytics and Success Signals

- **account_confirmation_email_sent** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 metric measures confirmation-email delivery directly; retained as an operational signal for email-capability health.
- **reset_email_sent** (delivery outcome: delivered / failed) -- supports success-metrics.md: "First-Session Onboarding Completion" (a returning organiser blocked from signing in depends on this email arriving to resume setup or return to their household).

## Acceptance Criteria

**FEAT-01.SPEC-017-AC-01:** Given Maya creates a new account on FEAT-01.SPEC-001, when the account is created, then this integration sends an account-confirmation email to her registered address.

**FEAT-01.SPEC-017-AC-02:** Given Sam requests a sign-in reset on FEAT-01.SPEC-002 for his registered email, when the request is submitted, then this integration sends a reset-link email to that address.

**FEAT-01.SPEC-017-AC-03:** Given the transactional email capability is slow, when Maya creates an account, then account creation and her navigation to FEAT-01.SPEC-003 are unaffected by the delay.

**FEAT-01.SPEC-017-AC-04:** Given the transactional email capability is down, when Sam requests a reset, then he still sees the neutral confirmation message and the email is queued to send once the capability recovers.

**FEAT-01.SPEC-017-AC-05:** Given the transactional email capability rejects the send for an account-confirmation email, when this occurs, then Maya's account creation and setup flow proceed unaffected.

**FEAT-01.SPEC-017-AC-06:** Given a reset-email send event arrives for an account that was deleted since the request, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-01.SPEC-017-AC-07:** Given a "send failed" event for a reset email is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-01.SPEC-017-AC-08:** Given a "delivered" event arrives after an earlier "failed" event for the same reset email attempt, when it is processed, then the delivery-status flag corrects to delivered, since the later event by timestamp governs.

**FEAT-01.SPEC-017-AC-09:** Given an adult requests a reset twice in quick succession, when both emails are eventually sent, then only the most recently generated reset token is valid, per FEAT-01.SPEC-002's business rules.

**FEAT-01.SPEC-017-AC-10:** Given Maya has never seen a data-sharing disclosure interrupt account creation, when she reviews the confirmation message on FEAT-01.SPEC-002, then she recognizes it as the disclosure that a reset email will be sent if her entered address matches an account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |

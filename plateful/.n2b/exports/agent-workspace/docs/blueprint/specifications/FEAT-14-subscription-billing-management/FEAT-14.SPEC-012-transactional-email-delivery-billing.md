---
document_type: spec
spec_type: integration
spec_id: FEAT-14.SPEC-012
spec_name: Transactional Email Delivery (Billing)
spec_slug: transactional-email-delivery-billing
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Integration Spec: Transactional Email Delivery (Billing)

## Overview

**Name:** Transactional Email Delivery (Billing)
**ID:** FEAT-14.SPEC-012
**Type:** Integration
**Purpose:** Product boundary to the transactional email capability used to deliver billing confirmations and the grace-period notice.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Sending the billing confirmation email (FEAT-14.SPEC-010's email variants)
- Sending the payment-failure grace-period notice email (FEAT-14.SPEC-011)
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what billing data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (account/recovery, safety reports, plan-ready fallback, export/deletion/support-acknowledgement emails) -- each is owned by the feature whose Communications require it (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the billing-related emails named in this feature's own Communications
- Deciding the exact wording of each email's subject and body -- owned by FEAT-14.SPEC-010 and FEAT-14.SPEC-011, whose Content Definition sections this spec delivers verbatim
- The in-app channel for either notification -- owned by FEAT-14.SPEC-010 and FEAT-14.SPEC-011 directly; this spec covers only the email channel

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for ... billing and grace-period notices ..." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-14.SPEC-012, covers the billing-confirmation and grace-period-notice portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya receives an email confirming an upgrade, period switch, downgrade, cancellation, or grace-period resolution | Upgrade to paid, Manage billing | FEAT-14.SPEC-010 (Billing Confirmation Notification) |
| Maya receives an email notice when a renewal payment fails, with a path to update payment details | Manage billing | FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Organiser's email address | Member Profile -- sign_in (email component, organiser only) | Any FEAT-14.SPEC-010 or FEAT-14.SPEC-011 email is triggered | The capability needs a destination address to deliver the email |
| Organiser's first name | Member Profile -- display_name (organiser) | Same as above | Personalizes the greeting, per the exact content templates in FEAT-14.SPEC-010 and FEAT-14.SPEC-011 |
| Billing content (tier, billing_period, grace-period end date) | Subscription -- tier, billing_period; derived grace-period end date | Same as above | The email's exact content, per FEAT-14.SPEC-010 and FEAT-14.SPEC-011's Content Definition sections |

No payment method details, billing_history amounts beyond what the confirmation content itself states, Dietary Rule data, or any other household member's data ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household, Member Profile, or Subscription field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Billing confirmation email delivered | The capability confirms the email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-14.SPEC-010's in-app notification already delivered regardless of email delivery status | FEAT-14.SPEC-010 |
| Grace-period notice email delivered | The capability confirms the email reached the recipient's inbox | None | None -- FEAT-14.SPEC-011's in-app notice already delivered regardless of email delivery status | FEAT-14.SPEC-011 |
| Send failed | The capability reports it could not deliver either email (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; the triggering notification's in-app channel is unaffected | FEAT-14.SPEC-010, FEAT-14.SPEC-011 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | No user-visible effect -- the in-app confirmation appears regardless of email send speed | No user-visible effect -- the in-app confirmation is unaffected; the email is queued to send once the capability recovers, up to its own retry limit | No user-visible effect on Maya's session -- a rejected send does not block or alter the in-app confirmation, since email is a durable-record channel, not a required verification gate |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | No user-visible effect -- the in-app notice and billing-state banner appear regardless of email send speed | No user-visible effect on the in-app channel; the email is queued to send once the capability recovers, within the notice's own pending-then-superseded window (FEAT-14.SPEC-011's Delivery Rules) | No user-visible effect on Maya's session -- the in-app notice and billing-state banner remain the primary, always-delivered record of the grace period |

## Consent and Disclosure

- **Billing-email disclosure** -- Sending a billing confirmation or grace-period notice email to the organiser's own registered address is standard product behavior for a consequential account change; no separate consent prompt interrupts an upgrade, period switch, downgrade, cancellation, or grace-period event, since the organiser already provided that email address for exactly this kind of account communication (per FEAT-01.SPEC-017's account-level disclosure).
- **What is never shared** -- Payment method details, the full billing_history list, Dietary Rule data (including any child's allergy information), and every other household member's data never leave the product through this integration; only the organiser's own email, first name, and the specific billing content named in Data Exchanged are included.

## Edge Cases

- **A billing-email send event arrives for a household that was deleted between the trigger and the send** -- The event is discarded silently; no email is sent for a deleted household, and no user feedback fires, since the in-app channel for that household no longer exists either.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry beyond the standard policy is triggered.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a grace-period notice** -- If the send was not confirmed initiated, it is treated as not yet sent and is queued for retry once the capability recovers, within FEAT-14.SPEC-011's own pending-then-superseded window; the in-app notice is unaffected either way.
- **A grace-period notice email is still queued when the grace period resolves** -- Per FEAT-14.SPEC-011's Edge Cases, the queued send is cancelled rather than delivered late, since the resolution confirmation (FEAT-14.SPEC-010) supersedes it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggered by (inbound) | Every confirmation variant's email channel is sent through this integration |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Triggered by (inbound) | The grace-period notice's email channel is sent through this integration |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |

## Analytics and Success Signals

- **billing_email_sent** (notification: confirmation / grace_notice; delivery outcome: delivered / failed) -- supports success-metrics.md: "Paying Household Retention" (a household unreachable by email during a grace period is at greater risk of an unresolved lapse this metric tracks)
- **billing_email_degradation_shown** (condition: slow / down / rejected; notification: confirmation / grace_notice) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-14.SPEC-012-AC-01:** Given Maya's upgrade completes, when FEAT-14.SPEC-010 fires, then this integration sends the upgrade-confirmation email to her registered address.

**FEAT-14.SPEC-012-AC-02:** Given a renewal payment failure opens a grace period, when FEAT-14.SPEC-011 fires, then this integration sends the grace-period notice email to Maya's registered address.

**FEAT-14.SPEC-012-AC-03:** Given the transactional email capability is slow, when a billing confirmation is triggered, then Maya's in-app confirmation is unaffected by the delay.

**FEAT-14.SPEC-012-AC-04:** Given the transactional email capability is down, when the grace-period notice is triggered, then Maya still sees the in-app notice and billing-state banner, and the email is queued to send once the capability recovers.

**FEAT-14.SPEC-012-AC-05:** Given the transactional email capability rejects a billing-confirmation send, when this occurs, then Maya's in-app confirmation and her session are unaffected.

**FEAT-14.SPEC-012-AC-06:** Given a billing email send event arrives for a household deleted since the trigger, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-14.SPEC-012-AC-07:** Given a "send failed" event for a billing email is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-14.SPEC-012-AC-08:** Given a "delivered" event arrives after an earlier "failed" event for the same billing-email attempt, when it is processed, then the delivery-status flag corrects to delivered.

**FEAT-14.SPEC-012-AC-09:** Given the capability goes down mid-send for a grace-period notice that was not confirmed initiated, when this occurs, then the send is queued for retry and the in-app notice is unaffected.

**FEAT-14.SPEC-012-AC-10:** Given a grace-period notice email is still queued when the grace period resolves, when the resolution occurs, then the queued send is cancelled rather than delivered late.

**FEAT-14.SPEC-012-AC-11:** Given Maya has never seen a data-sharing prompt interrupt a billing action, when she receives a billing email, then she recognizes it as standard account communication to the address she already provided, per FEAT-01.SPEC-017's account-level disclosure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 notifications x 3 conditions) | 6 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |

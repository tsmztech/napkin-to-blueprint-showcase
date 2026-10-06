---
document_type: spec
spec_type: integration
spec_id: FEAT-18.SPEC-012
spec_name: Transactional Email (Account & Data)
spec_slug: transactional-email-account-data
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Transactional Email (Account & Data)

## Overview

**Name:** Transactional Email (Account & Data)
**ID:** FEAT-18.SPEC-012
**Type:** Integration
**Purpose:** Product boundary to the transactional email capability used to deliver export-ready, deletion-completed, and support-acknowledgement email.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Sending the export-ready email (FEAT-18.SPEC-013's email variant)
- Sending the household-deletion-completed email (FEAT-18.SPEC-014's email variant)
- Sending the support-request-acknowledgement email (FEAT-18.SPEC-015's email variant)
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what account, household-summary, and support-message data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (account/recovery, safety reports, plan-ready fallback, billing confirmations) -- each is owned by the feature whose Communications require it (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the account-and-data emails named in this feature's own Communications
- Deciding the exact wording of each email's subject and body -- owned by FEAT-18.SPEC-013, FEAT-18.SPEC-014, and FEAT-18.SPEC-015, whose Content Definition sections this spec delivers verbatim
- The in-app channel for any of the three notifications -- owned directly by FEAT-18.SPEC-013, FEAT-18.SPEC-014, and FEAT-18.SPEC-015; this spec covers only the email channel
- Attaching or delivering the export file itself through email -- the export is downloaded in-app from FEAT-18.SPEC-001; this spec's export-ready email links back to that screen rather than carrying the file as an attachment

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for ... data-export and deletion confirmations, ... and support acknowledgements ..." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-18.SPEC-012, covers the export-ready, deletion-completed, and support-acknowledgement portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya receives an email once her requested export is ready to download | Export household data | FEAT-18.SPEC-013 (Export Ready Notification) |
| Maya receives an email once her household's deletion has fully completed | Delete the household | FEAT-18.SPEC-014 (Household Deletion Completed Notification) |
| The member who contacted support receives an email acknowledging their message | Contact support | FEAT-18.SPEC-015 (Support Request Acknowledgement) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|-------------------|-----------|------------|
| Maya's email address | Member Profile -- sign_in (email component, organiser only) | FEAT-18.SPEC-013 or FEAT-18.SPEC-014 is triggered | The capability needs a destination address to deliver the email |
| Maya's first name / display name | Member Profile -- display_name (organiser) | Same as above | Personalizes the greeting, per the exact content templates in FEAT-18.SPEC-013 and FEAT-18.SPEC-014 |
| Export readiness content (ready date) | Derived -- the completed export's ready date, no export content itself | FEAT-18.SPEC-013 is triggered | The email's exact content, per FEAT-18.SPEC-013's Content Definition |
| Household deletion completion content (completion date) | Derived -- the completed deletion's completion date, no deleted household content itself | FEAT-18.SPEC-014 is triggered | The email's exact content, per FEAT-18.SPEC-014's Content Definition |
| Reporting member's email address and first name | Member Profile -- sign_in (email component), display_name (the member who submitted the support description) | FEAT-18.SPEC-015 is triggered | The capability needs a destination address and a personalization value for the acknowledgement |

No export file contents, no Dietary Rule data (including any child's allergy information), no other household member's data, no payment details, and no support-description text beyond what FEAT-18.SPEC-015's own content template states ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|------------------|-------------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household, Member Profile, or Support Request field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|----------------|-------------------|--------------------|
| Export-ready email delivered | The capability confirms the email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-18.SPEC-001's in-app Ready state already reflects readiness regardless of email delivery status | FEAT-18.SPEC-013 |
| Deletion-completed email delivered | The capability confirms the email reached the recipient's inbox | None | None -- deletion is already complete regardless of email delivery status | FEAT-18.SPEC-014 |
| Support-acknowledgement email delivered | The capability confirms the email reached the recipient's inbox | None | None -- the reporting member's screen-level confirmation (FEAT-18.SPEC-005) already showed a same-screen success message | FEAT-18.SPEC-015 |
| Send failed | The capability reports it could not deliver any of the three emails (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; each triggering notification's in-app channel is unaffected | FEAT-18.SPEC-013, FEAT-18.SPEC-014, FEAT-18.SPEC-015 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-------------------|--------------------|------------------------|
| FEAT-18.SPEC-001 (Export Household Data) | No user-visible effect -- the in-app Ready state appears regardless of email send speed | No user-visible effect -- the in-app Ready state is unaffected; the export-ready email is queued to send once the capability recovers, up to its own retry limit | No user-visible effect on Maya's session -- a rejected send does not block or alter the in-app Ready state, since email is a durable-record channel here, not a required confirmation gate |
| FEAT-18.SPEC-003 (Delete Household) | No user-visible effect -- the deletion-in-progress and (later) completion state is driven by FEAT-18.SPEC-008's own processing, not by email | No user-visible effect on the in-app channel; the completion email is queued to send once the capability recovers | No user-visible effect on Maya's session -- the in-app completion state remains the primary, always-delivered record |
| FEAT-18.SPEC-005 (Contact Support) | No user-visible effect -- the same-screen "we've got your message" confirmation appears regardless of email send speed | No user-visible effect on the in-app confirmation; the acknowledgement email is queued to send once the capability recovers | No user-visible effect on the reporting member's session -- the same-screen confirmation already told them the message was received |

## Consent and Disclosure

- **Export-ready and deletion-completed email disclosure** -- Sending Maya a confirmation email at her own registered address once an export is ready or a deletion completes is standard product behavior for an action she just took; no separate consent prompt interrupts the request or confirmation flow, since she already provided that email address for exactly this kind of account communication (per FEAT-01.SPEC-017's account-level disclosure).
- **Support-acknowledgement disclosure** -- FEAT-18.SPEC-005's same-screen confirmation, "Thanks -- we've got your message and will follow up by email.", is itself the disclosure that an email will follow to the reporting member's registered address.
- **What is never shared** -- Export file contents, Dietary Rule data (including any child's allergy information), any other household member's data, and payment details never leave the product through this integration; only the recipient's own email, first name, and the specific content named in Data Exchanged are included.

## Edge Cases

- **An export-ready or deletion-completed email send event arrives for a household deleted between the trigger and the send** -- For the export-ready email, the event is discarded silently if the household no longer exists (a household deletion in the interim would have cancelled the in-flight export per FEAT-18.SPEC-006's Edge Cases); the deletion-completed email is unaffected by this scenario, since it fires only once deletion has already fully completed.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry beyond the standard policy is triggered.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a support acknowledgement** -- If the send was not confirmed initiated, it is treated as not yet sent and is queued for retry once the capability recovers; the reporting member's in-app confirmation is unaffected either way.
- **Maya requests a second export while the first export's ready email is still queued (capability was down)** -- Each export's ready email is tracked as its own independent send attempt; the second export's own readiness triggers its own email once it, too, is ready, without interfering with the first's queued retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-013 (Export Ready Notification) | Triggered by (inbound) | The export-ready email channel is sent through this integration |
| FEAT-18.SPEC-013 (Export Ready Notification) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Triggered by (inbound) | The deletion-completed email channel is sent through this integration |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Affects (outbound) | Degradation behavior surfaces here |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Triggered by (inbound) | The acknowledgement email channel is sent through this integration |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Affects (outbound) | Degradation behavior surfaces here |
| FEAT-01.SPEC-017 (Transactional Email Integration -- Account Recovery) | References (inbound) | Shares the account-level email disclosure this spec relies on |

## Analytics and Success Signals

- **account_data_email_sent** (notification: export_ready / deletion_completed / support_ack; delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures this feature's email delivery directly; retained per product-features.md's Signals field (data_export_completed, household_deleted, support_contacted) as the operational record of these confirmations actually reaching households
- **account_data_email_degradation_shown** (condition: slow / down / rejected; notification: export_ready / deletion_completed / support_ack) -- N/A -- no Stage 2 success metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-18.SPEC-012-AC-01:** Given Maya's export becomes ready, when FEAT-18.SPEC-013 fires, then this integration sends the export-ready email to her registered address.

**FEAT-18.SPEC-012-AC-02:** Given Maya's household deletion fully completes, when FEAT-18.SPEC-014 fires, then this integration sends the deletion-completed email to her registered address.

**FEAT-18.SPEC-012-AC-03:** Given Sam submits a support description, when FEAT-18.SPEC-015 fires, then this integration sends the acknowledgement email to Sam's registered address.

**FEAT-18.SPEC-012-AC-04:** Given the transactional email capability is slow, when an export-ready email is triggered, then Maya's in-app Ready state is unaffected by the delay.

**FEAT-18.SPEC-012-AC-05:** Given the transactional email capability is down, when the deletion-completed email is triggered, then Maya's in-app completion state is unaffected and the email is queued to send once the capability recovers.

**FEAT-18.SPEC-012-AC-06:** Given the transactional email capability rejects a support-acknowledgement send, when this occurs, then the reporting member's same-screen confirmation on FEAT-18.SPEC-005 remains unaffected.

**FEAT-18.SPEC-012-AC-07:** Given an export-ready email send event arrives for a household deleted since the trigger, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-18.SPEC-012-AC-08:** Given a "send failed" event for any of this spec's three emails is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-18.SPEC-012-AC-09:** Given a "delivered" event arrives after an earlier "failed" event for the same email attempt, when it is processed, then the delivery-status flag corrects to delivered.

**FEAT-18.SPEC-012-AC-10:** Given the capability goes down mid-send for a support acknowledgement that was not confirmed initiated, when this occurs, then the send is queued for retry and the reporting member's in-app confirmation is unaffected.

**FEAT-18.SPEC-012-AC-11:** Given Maya requests a second export while the first export's ready email is still queued, when the second export becomes ready, then its own email is sent independently without disrupting the first's queued retry.

**FEAT-18.SPEC-012-AC-12:** Given Maya has never seen a data-sharing prompt interrupt an export or deletion action, when she receives an export-ready or deletion-completed email, then she recognizes it as standard account communication to the address she already provided, per FEAT-01.SPEC-017's account-level disclosure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |

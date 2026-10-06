---
document_type: spec
spec_type: integration
spec_id: FEAT-14.SPEC-001
spec_name: Transactional Email Delivery
spec_slug: transactional-email-delivery
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Integration Spec: Transactional Email Delivery

## Overview

**Name:** Transactional Email Delivery
**ID:** FEAT-14.SPEC-001
**Type:** Integration
**Purpose:** Sends every composed email through the product's transactional email delivery capability and reports delivery, bounce, and failure status back so the product can track and act on outcomes.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- Sending a fully composed email (handed off by FEAT-14.SPEC-002) through the transactional email delivery capability
- Receiving delivery, bounce, and failure status back from the capability and reporting it for FEAT-14.SPEC-003 to consume
- Product-level behavior when the capability is slow, unavailable, or rejects a send
- Disclosure of what recipient and content data is shared with the capability

**Non-Goals:**
- Deciding which notification type an event produces, or who is entitled to receive it -- owned by FEAT-14.SPEC-004 (Notification Type & Recipient Entitlement Rules); this spec sends whatever fully composed, entitlement-checked email FEAT-14.SPEC-002 hands it.
- Composing the email's subject, body, sender name, or branding -- owned by FEAT-14.SPEC-002 (dispatch) and FEAT-14.SPEC-005 (presentation rules); this spec transmits already-composed content, it does not write any of it.
- Retrying a failed send or deciding when retries are exhausted -- owned by FEAT-14.SPEC-003 (Delivery Status Tracking & Retry), which consumes this spec's inbound events and issues re-send requests back through this same integration.
- Choosing the transactional email vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md's Ecosystem & Integrations section records no user mandate for a specific email vendor.

## Capability Category

**Category:** Transactional email delivery
**Dependency Source:** ASMP-29 -- "Transactional email delivery capability" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email delivery with delivery and bounce status (ASMP-29)" row in feature-dependency-map.md, ## External Touchpoints (Integration Specs: FEAT-14.SPEC-001)
**Vendor Mandate:** None -- BRIEF.md's Ecosystem & Integrations section states only that "all notifications to clients and freelancers go by email," naming no specific vendor; vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Every composed, entitlement-checked email is actually transmitted to its recipient | Event-triggered email dispatch | FEAT-14.SPEC-002 (Notification Composition & Dispatch) |
| The product knows, per notification, whether an email was delivered, bounced, or failed | Delivery tracking | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) |
| A failed send is retried automatically through this same capability before being given up on | Delivery tracking | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient identity | Client Contact -- name, email (or Freelancer Account -- name, sign-in email, for freelancer-addressed notifications) | Every send attempt | The capability must know who to address the email to |
| Sender identity | Freelancer Account -- business_name; Branding Profile -- logo, brand_colour (formatted per FEAT-14.SPEC-005's rules) | Every send attempt | The capability displays the sender name and branding the recipient sees |
| Email content | The composed subject and body handed off by FEAT-14.SPEC-002 (values drawn from the triggering feature's own Notification spec's Content Definition) | Every send attempt | This is the message itself; the capability cannot deliver an email without it |
| Notification reference | Notification -- notification_type, an internal reference used to correlate outcomes | Every send attempt | Ties the capability's delivery/bounce report back to the correct Notification record |

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery confirmation | The capability confirms the message reached the recipient's mail system | Notification -- delivery_status (Delivered) |
| Bounce report with a reason category (when available) | The recipient's address rejects the message (permanent failure) | Notification -- delivery_status (Bounced) |
| Transient failure report | The message could not be sent on this attempt (temporary condition) | Notification -- delivery_status (Failed), consumed by FEAT-14.SPEC-003 for retry decisioning |

No deliverable files, card or bank details (ASMP-24), or content beyond the specific composed message ever leave the product through this capability; the referral mark (XBR-32) and branding (XBR-31) are the only additions beyond the triggering feature's own content, both applied at composition time by FEAT-14.SPEC-005.

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivered | The capability confirms the message reached the recipient's mail system | Notification.delivery_status set to Delivered | None -- delivery is silent to the recipient and to Nadia unless she is specifically reviewing a project's history | FEAT-14.SPEC-003 |
| Bounced | The recipient's address permanently rejects the message (e.g., the address does not exist) | Notification.delivery_status set to Bounced | None immediately; if this is the final outcome after retries, FEAT-14.SPEC-006 warns Nadia | FEAT-14.SPEC-003, FEAT-14.SPEC-006 (after retries exhausted) |
| Failed (transient) | The message could not be transmitted on this attempt for a temporary reason (e.g., the recipient's mail system is briefly unreachable) | Notification.delivery_status set to Failed | None immediately; FEAT-14.SPEC-003 schedules a retry | FEAT-14.SPEC-003 |

Retry counting, exhaustion decisions, and the hand-off to the delivery-failure warning involve multi-step, branching logic, so they are routed to FEAT-14.SPEC-003, an Automation spec whose Trigger Definition names this spec as its external-event source.

## Degradation Behavior

This feature owns no screens (Type: Platform; feature-overview.md, States: "no primary browsing view of its own"), and dispatch happens as a background process independent of any triggering screen's completion (feature-overview.md, States: "sending happens automatically in the background, independent of either party's connectivity"). No screen in the product sends a synchronous request to this capability or waits on its response -- every triggering screen (e.g., FEAT-02.SPEC-005, Proposal Send) has already completed its own action before this integration is ever invoked.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| N/A -- no screen sends a request to this capability; every triggering screen's own action completes before dispatch begins | N/A -- a slow send only delays when Notification.delivery_status transitions from Queued to Sent/Delivered; the triggering screen already completed and shows no blocked or degraded state | N/A -- a capability-down period leaves affected Notifications Queued; FEAT-14.SPEC-003 retries once the capability recovers, within the standing retry window (platform parameter: `transactional-email-retry-window`); no screen shows an error | N/A -- a rejected send (e.g., a malformed address) is handled identically to the Bounced inbound event above, entirely by FEAT-14.SPEC-003, without blocking any screen |

## Consent and Disclosure

- **Account-level disclosure of the delivery mechanism** -- Nadia's account setup (FEAT-20, Onboarding / First-Run Setup) and her account settings (FEAT-21, Settings & Account Management) state that every client-facing and freelancer-facing notification is sent through a transactional email delivery capability, which receives the recipient's name and email address and the composed message content for each notification. This is not a per-notification consent choice: email is the product's sole notification channel (BRIEF.md, Ecosystem & Integrations), and transactional notifications core to the record cannot be declined (XBR-30); Nadia's only control is over optional notification types, exercised in FEAT-21 before a notification is ever composed (FEAT-14.SPEC-004).
- **Recipient-facing disclosure** -- client contacts (Owen, Priya) are not separately asked to consent to receiving product emails; receiving email is the entire mechanism by which they are invited into and kept informed about the portal (FEAT-05, FEAT-18), and every email they receive names the freelancer's business as the sender (FEAT-14.SPEC-005), so its origin is never disguised.
- **What is never shared** -- deliverable files themselves, card or bank details (ASMP-24), and any content beyond the specific composed message for that one notification (a Reviewer never receives invoice content, per the Access Matrix and FEAT-14.SPEC-004's entitlement rules; a client never receives another client's information, per ASMP-23).

## Edge Cases

- **A delivery or bounce event arrives for a Notification that no longer exists (e.g., the account was deleted, FEAT-24)** -- The event is discarded silently; FEAT-24's account deletion removes Notification records as part of account deletion, and no orphaned status update is applied or surfaced.
- **The same delivery event is reported twice** -- The second report changes nothing: a Notification already Delivered stays Delivered, and FEAT-14.SPEC-003 does not re-fire any dependent behavior (FEAT-14.SPEC-006 does not warn twice for the same notification).
- **Events arrive out of order (a later Delivered report arrives before an earlier transient Failed report for the same send attempt)** -- The Notification reflects the most recent event by the event's own occurrence time, not arrival time; a Delivered outcome that occurred after a transient failure and successful retry overrides the stale Failed report.
- **The capability goes down mid-send, with no confirmation either way** -- The Notification remains Queued (never advanced to Sent or Delivered without confirmation), so FEAT-14.SPEC-003 treats it as unconfirmed and retries once the capability recovers, rather than assuming either success or failure.
- **A bounce report gives no specific reason** -- FEAT-14.SPEC-003 and FEAT-14.SPEC-006 treat a reason-less bounce identically to a categorized one; Nadia's delivery warning states that the email could not be delivered without inventing a reason the capability did not provide.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-002 (Notification Composition & Dispatch) | Triggered by (inbound) | Hands this spec a fully composed, entitlement-checked email to send |
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Triggers (outbound) | Every inbound event (Delivered / Bounced / Failed) is consumed by this automation, which also issues retry send requests back through this integration |
| FEAT-14.SPEC-005 (Recognizable & Branded Email Presentation Rules) | References (inbound) | Sender name, subject, and formatting rules this spec's outbound content must already satisfy before it is sent |
| FEAT-02.SPEC-011, FEAT-03.SPEC-006, FEAT-03.SPEC-007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004 (other features' own Notification specs) | References (inbound) | Each names this spec as the underlying delivery, retry, and bounce/failure-reporting capability their own email is sent through, per the Brief's Shared Validation section |

## Analytics and Success Signals

- **notification_email_sent** (notification_type, attempt_number) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_email_delivered** (notification_type) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_email_bounced** (notification_type, reason_category) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_email_failed_transient** (notification_type) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-14.SPEC-001-AC-01:** Given FEAT-14.SPEC-002 hands this integration a fully composed proposal-sent email addressed to Owen, when the send is transmitted, then the capability reports back a delivery outcome and Notification.delivery_status updates accordingly.

**FEAT-14.SPEC-001-AC-02:** Given a notification has been sent, when the capability reports the Delivered event, then Notification.delivery_status is set to Delivered and no further action is taken.

**FEAT-14.SPEC-001-AC-03:** Given a send is rejected because Owen's email address does not exist, when the Bounced event arrives, then Notification.delivery_status is set to Bounced and FEAT-14.SPEC-003 begins its retry-then-warn handling.

**FEAT-14.SPEC-001-AC-04:** Given a send attempt fails for a temporary reason, when the Failed event arrives, then Notification.delivery_status is set to Failed and FEAT-14.SPEC-003 schedules a retry.

**FEAT-14.SPEC-001-AC-05:** Given the capability is slow to confirm a send, when the triggering screen (e.g., FEAT-02.SPEC-005) has already completed its own action, then no screen shows a blocked or degraded state.

**FEAT-14.SPEC-001-AC-06:** Given the capability is down when a notification is queued, when the outage lasts through the standing retry window, then the notification is retried automatically once the capability recovers, per FEAT-14.SPEC-003.

**FEAT-14.SPEC-001-AC-07:** Given a send is rejected for a malformed address, when the rejection is reported, then it is handled identically to a Bounced event and no screen shows an error.

**FEAT-14.SPEC-001-AC-08:** Given Nadia opens her account settings (FEAT-21), when she reviews the delivery-mechanism disclosure, then it names the recipient identity and message content shared with the capability, consistent with this spec's Data Exchanged section.

**FEAT-14.SPEC-001-AC-09:** Given a recipient receives any product email, when they read it, then the sender name identifies the freelancer's business, never a disguised or unrelated origin.

**FEAT-14.SPEC-001-AC-10:** Given a Notification's underlying account has been deleted (FEAT-24), when a delivery event arrives afterward for that notification, then it is discarded silently and no status update is applied.

**FEAT-14.SPEC-001-AC-11:** Given a Notification is already Delivered, when the same Delivered event is reported a second time, then nothing changes and FEAT-14.SPEC-006 is not triggered a second time.

**FEAT-14.SPEC-001-AC-12:** Given a Delivered event for a later attempt arrives before a stale Failed event for the same notification, when both are processed, then the Notification reflects the Delivered outcome by its true occurrence time, not arrival order.

**FEAT-14.SPEC-001-AC-13:** Given the capability goes down mid-send with no confirmation either way, when the outage is detected, then the Notification remains Queued rather than being marked Sent or Delivered without confirmation.

**FEAT-14.SPEC-001-AC-14:** Given a bounce report carries no specific reason, when FEAT-14.SPEC-006 later warns Nadia, then the warning states delivery failed without inventing a reason the capability did not provide.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (all N/A, justified: no screen sends a request to this capability) | 3 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |

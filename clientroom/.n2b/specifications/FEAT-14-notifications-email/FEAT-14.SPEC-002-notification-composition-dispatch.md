---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-002
spec_name: Notification Composition & Dispatch
spec_slug: notification-composition-dispatch
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Automation Spec: Notification Composition & Dispatch

## Overview

**Name:** Notification Composition & Dispatch
**ID:** FEAT-14.SPEC-002
**Type:** Automation
**Purpose:** The shared dispatch engine every other feature's triggering event calls -- it resolves the entitled recipient(s) and notification type for one triggering event, creates the Notification record, and hands the composed email to the delivery capability.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- Receiving a triggering event from any other feature and creating the corresponding Notification record(s)
- Checking recipient entitlement and preferences at the moment of dispatch (delegated to FEAT-14.SPEC-004)
- Composing sender name, subject formatting, and branding per FEAT-14.SPEC-005's presentation rules, on top of the content the triggering feature's own Notification spec defines
- Handing the composed, entitlement-checked email to FEAT-14.SPEC-001 for transmission

**Non-Goals:**
- Deciding *when* a triggering event fires -- owned by each triggering feature's own screen or automation (e.g., FEAT-02.SPEC-005 decides when a proposal is sent); this spec begins only once that event fires.
- The one-type-per-event mapping, entitlement, and preference rules themselves -- owned by FEAT-14.SPEC-004; this spec calls that spec's rules rather than re-deriving them.
- Sender-name, subject, and branding formatting rules themselves -- owned by FEAT-14.SPEC-005; this spec applies those rules rather than defining them.
- Actually transmitting the email and reporting delivery/bounce/failure status -- owned by FEAT-14.SPEC-001 and FEAT-14.SPEC-003; this spec hands off to them and stops.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Proposal sent, edited-and-resent, or resent | FEAT-02.SPEC-011 (Proposal Sent/Resent Email) | Fires on original send, void-and-resend, or unchanged resend | Project, client, Primary Contact(s), proposal reference, variant (original/edited/resent) |
| Proposal accepted, or change request sent | FEAT-03.SPEC-006 (Acceptance Confirmation Notification), FEAT-03.SPEC-007 (Change Request Notification) | Fires on acceptance recorded or a change-request note sent | Project, accepting/requesting contact, proposal reference |
| Sign-in link requested (initial or fresh) | FEAT-05.SPEC-008 (Magic-Link Sign-In Email) | Fires whenever a contact requests a sign-in link | Contact identity, single-use link reference |
| Deliverable ready to review | FEAT-06.SPEC-006 (Deliverable Ready Notification) | Fires only once an upload has fully completed (XBR-12) | Project, milestone, deliverable reference, entitled contacts |
| Deliverable comment posted, or freelancer reply posted | FEAT-07.SPEC-003 (Client Comment Alert to Freelancer), FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Fires on a new comment or reply | Comment author, comment text reference, pinned deliverable/milestone |
| Milestone approved | FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification) | Fires when an approval is recorded | Project, milestone, approving contact |
| Invoice generated and issued (automatic or ad hoc) | FEAT-09.SPEC-010 (Invoice Issued/Copy Confirmation Notification) | Fires whenever an invoice is sent | Project, invoice reference, amount, currency, triggering event |
| Payment succeeds | FEAT-10.SPEC-007 (Payment Confirmation Notification) | Fires on a confirmed successful payment | Invoice reference, payment amount, payer identity |
| Day-3 / day-10 automatic reminder, or manual reminder | FEAT-11.SPEC-004 (Overdue Reminder Email) | Fires when the reminder schedule or a manual reminder action fires | Invoice reference, reminder type, days overdue |
| Contact invited (a Primary or Reviewer added or invited) | FEAT-18.SPEC-010 (New Contact Invitation Email), FEAT-18.SPEC-011 (Primary-Invited Colleague Alert) | Fires when Nadia or a Primary contact invites a new contact; SPEC-011 additionally alerts Nadia when a Primary contact did the inviting | New contact identity, inviting party, client |
| Sign-in email change confirmed | FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | Fires when the new sign-in email is committed | Prior sign-in email, new sign-in email |
| Plan changed, or renewal charge fails | FEAT-23.SPEC-008 (Plan & Billing Notifications) | Fires on a plan tier/status change or the first failed renewal charge | Plan reference, prior and new tier/status, failure reason |
| Data export ready, or account deletion confirmed | FEAT-24.SPEC-008 (Export-Ready Notification), FEAT-24.SPEC-009 (Account Deletion Final Warning Notification) | Fires when the archive becomes Ready, or when Nadia gives explicit deletion confirmation | Freelancer Account identity, archive link window or confirmation timestamp |
| Refund or project cancellation recorded | FEAT-25.SPEC-007 (Refund & Cancellation Notification) | Fires when a refund or cancellation commit succeeds | Invoice or project reference, refund type and amount |
| Proposal signature recorded | FEAT-26.SPEC-004 (Signed-Copy Confirmation Notification) | Fires when the signature write succeeds | Proposal reference, signer name, accepted_at |
| Custom domain verified | FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Fires once when verification succeeds | Freelancer Account identity, domain name |
| Payment connection becomes Connected or Needs attention | FEAT-32.SPEC-006 (Connection Status Notifications) | Fires when the connection status sync applies either outcome | Freelancer Account identity, attention reason |
| Freelancer sign-up completed (welcome email) | FEAT-20 (Onboarding / First-Run Setup) | Fires once account creation completes | Freelancer Account identity |
| Support request sent, or support session opened/closed | FEAT-31.SPEC-006 (Support Request Confirmation), FEAT-31.SPEC-007 (Support Session Notice) | Fires on a support request, or on session open/close (XBR-29) | Support Access Session reference, operator identity, freelancer account |
| Payment processor reports a chargeback or reversal | FEAT-25.SPEC-008 (Chargeback Notification), relaying FEAT-32.SPEC-002 | Fires when a dispute event is relayed (XBR-21) | Invoice reference, dispute type |

## Processing Logic

1. Receive the triggering event's data from the source spec (event type, affected project/entity reference, and any event-specific fields it provides).
2. Determine the notification_type for this triggering event, per the one-type-per-event mapping owned by FEAT-14.SPEC-004.
3. Resolve the entitled recipient(s) for this notification_type against the Access Matrix, and, for an optional type, against the freelancer's current notification preferences -- both evaluated now, at dispatch time, not at some earlier moment (FEAT-14.SPEC-004). If zero recipients are entitled, proceed to the No Entitled Recipients outcome without creating a Notification record.
4. For each entitled recipient, create one Notification record with notification_type, recipient, and delivery_status set to Queued.
5. Compose the sender name and apply branding and the referral mark to the notification, per FEAT-14.SPEC-005's presentation rules, on top of the subject/body content the triggering feature's own Notification spec defines.
6. Hand the composed, entitlement-checked email for each created Notification to FEAT-14.SPEC-001 for transmission.
7. If the automation itself cannot complete resolution or composition for a processing reason (not an entitlement denial), still create the intended Notification record with delivery_status set to Failed immediately, so it enters FEAT-14.SPEC-003's standard retry-then-warn handling on its next cycle, rather than leaving no record of the intended event at all.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Notification dispatched | One or more recipients are entitled to the triggering event's notification type | One Notification record per entitled recipient created (Queued), handed to FEAT-14.SPEC-001 | None directly; delivery or failure surfaces later via FEAT-14.SPEC-003 / FEAT-14.SPEC-006 | FEAT-14.SPEC-001, FEAT-14.SPEC-003 |
| No entitled recipients | FEAT-14.SPEC-004's entitlement check returns zero recipients (a role not entitled to this event, or an optional type the freelancer has switched off) | No Notification record is created | None -- silent by design; a switched-off optional notification not firing is the intended behavior, not a failure | FEAT-14.SPEC-004 |
| Partial dispatch | Some recipients of the triggering event are entitled, others are not (e.g., Owen is entitled, Priya as Reviewer is not, for the same proposal-sent event) | A Notification record is created only for each entitled recipient | None -- entitled recipients receive their email normally; non-entitled contacts see nothing, which is the intended per-recipient filtering, not an error | FEAT-14.SPEC-004 |
| Dispatch resolution failure | The automation itself cannot complete resolution or composition for a processing reason | A Notification record is still created for the intended recipient with delivery_status set to Failed immediately (bypassing Queued); the triggering feature's own action (e.g., the proposal save) is never blocked or reverted -- non-blocking | None immediately; the Failed record enters FEAT-14.SPEC-003's normal retry-then-warn handling like any other failed send | FEAT-14.SPEC-003, FEAT-14.SPEC-006 (after retries exhausted) |

## Data Model

**Reads:** Client Contact -- name, email, role, status (recipient resolution); Freelancer Account -- name, sign-in email, notification_preferences (preference checks and freelancer-addressed notifications); Branding Profile -- logo, brand_colour (composition); Support Access Session -- for the support-notice trigger; the triggering feature's own event data (project, entity references).
**Creates:** Notification -- notification_type, recipient, delivery_status (Queued or, on a resolution failure, Failed).
**Updates:** None directly -- sent_at and subsequent delivery_status transitions are owned by FEAT-14.SPEC-003 once dispatch is handed off.
**Deletes:** None.

## Business Rules

- Each triggering event maps to exactly one notification_type, avoiding duplicate or missing emails (FEAT-14.SPEC-004 owns this mapping).
- Recipients are limited to contacts entitled to the event per the Access Matrix (XBR-08).
- An optional notification type checks the freelancer's current notification preferences at dispatch time; transactional types core to the record always send regardless of preference (XBR-30).
- The freelancer's branding and the referral mark are applied to every client-facing email without overriding the branding (XBR-31, XBR-32) -- both applied by FEAT-14.SPEC-005 during this automation's composition step.
- A deliverable-ready notification's trigger already guarantees the upload is fully complete before firing (XBR-12); this automation does not re-check upload completeness, since that guarantee is owned by the trigger source (FEAT-06.SPEC-006).
- Support-session announcements are transactional and always fire regardless of preference (XBR-29).
- This automation runs synchronously with respect to entitlement and composition, but its dispatch to FEAT-14.SPEC-001 is non-blocking relative to the triggering screen's own completion (feature-overview.md, States: "sending happens automatically in the background, independent of either party's connectivity").

## Edge Cases

- **Two triggering events for the same underlying entity fire in quick succession (e.g., a proposal is void-and-resent twice within moments)** -- Each triggering-event occurrence produces its own independent Notification record; no coalescing happens at this layer. Batching within a single notification type, if any, is a decision made in that triggering feature's own Notification spec, not here.
- **Concurrent trigger firing for the same recipient from two different triggering features at the same moment (e.g., a comment alert and a milestone-approval confirmation arrive together)** -- Each creates its own independent Notification record; both proceed to FEAT-14.SPEC-001 independently and are tracked by FEAT-14.SPEC-003 without interfering with each other.
- **Trigger fires while a previous dispatch run for the same exact triggering-event occurrence is still in flight (a resolution retry)** -- The automation checks whether a Notification record already exists for that exact triggering-event occurrence before creating a new one, so a resolution retry never produces a duplicate Notification.
- **The triggering event's project or client has since been archived** -- Archiving does not revoke recipient entitlement; the dispatch still proceeds normally, since archiving removes a record from active view without withdrawing a client's or contact's Access Matrix entitlement.
- **Nadia turns off an optional notification type in the instant between the triggering event firing and this automation evaluating entitlement** -- The preference is evaluated at the moment this automation runs its entitlement check, not at the earlier instant the event fired; the freshly-off preference wins and no Notification record is created for that occurrence.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-004 (Notification Type & Recipient Entitlement Rules) | Triggered by (inbound, at call time within this automation) | Supplies the notification-type mapping and recipient entitlement/preference decision this automation applies |
| FEAT-14.SPEC-005 (Recognizable & Branded Email Presentation Rules) | Triggered by (inbound, at call time within this automation) | Supplies the sender-name, branding, and referral-mark rules this automation applies during composition |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Affects (outbound) | Receives the composed, entitlement-checked email for transmission |
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Affects (outbound) | Tracks the delivery outcome of every Notification this automation creates |
| FEAT-02.SPEC-011, FEAT-03.SPEC-006, FEAT-03.SPEC-007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004 | Triggered by (inbound) | Each is a triggering feature's own Notification spec that calls this dispatch engine for its event |
| FEAT-18.SPEC-010, FEAT-18.SPEC-011, FEAT-20.SPEC-006, FEAT-21.SPEC-011, FEAT-23.SPEC-008, FEAT-24.SPEC-008, FEAT-24.SPEC-009, FEAT-25.SPEC-007, FEAT-25.SPEC-008, FEAT-26.SPEC-004, FEAT-27.SPEC-004, FEAT-31.SPEC-006, FEAT-31.SPEC-007, FEAT-32.SPEC-006 | Triggered by (inbound) | Each is a triggering feature's own Notification spec (contact-invite, welcome, account-critical confirmation, billing, export/deletion, refund/cancellation, signature, domain, dispute, support, or payment-connection event) that calls this dispatch engine |

## Analytics and Success Signals

- **notification_dispatch_created** (notification_type, recipient_role) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_dispatch_skipped_no_entitlement** (notification_type, reason: role_not_entitled / preference_off) -- N/A -- no Stage 2 metric measures deliberate non-sends; retained so the entitlement/preference filter's effect is observable and distinct from a delivery failure.
- **notification_dispatch_resolution_failed** (notification_type) -- supports success-metrics.md: "Notification Delivery Reliability" (a resolution failure becomes a Failed-status Notification and is counted the same as any other delivery failure)

## Acceptance Criteria

**FEAT-14.SPEC-002-AC-01:** Given Nadia sends a proposal to Owen, when FEAT-02.SPEC-011 fires this automation, then a Notification record is created for Owen with notification_type "proposal sent" and delivery_status Queued, and the composed email is handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-02:** Given Owen accepts a proposal, when FEAT-03.SPEC-006 fires this automation, then a Notification record is created for the entitled recipients and handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-03:** Given Priya requests a fresh sign-in link, when FEAT-05.SPEC-008 fires this automation, then a Notification record is created for Priya's own contact record and handed to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-04:** Given a deliverable's upload fully completes, when FEAT-06.SPEC-006 fires this automation, then Notification records are created for every entitled contact on that project.

**FEAT-14.SPEC-002-AC-05:** Given Priya leaves a comment on a deliverable, when FEAT-07.SPEC-003 fires this automation, then a Notification record is created for Nadia.

**FEAT-14.SPEC-002-AC-06:** Given Owen approves a milestone, when FEAT-08.SPEC-007 fires this automation, then Notification records are created for Owen and Nadia per that spec's audience.

**FEAT-14.SPEC-002-AC-07:** Given an invoice is generated and issued, when FEAT-09.SPEC-010 fires this automation, then a Notification record is created for Owen.

**FEAT-14.SPEC-002-AC-08:** Given Owen's payment succeeds, when FEAT-10.SPEC-007 fires this automation, then Notification records are created for Owen and Nadia.

**FEAT-14.SPEC-002-AC-09:** Given an invoice becomes overdue and the day-3 reminder fires, when FEAT-11.SPEC-004 fires this automation, then a Notification record is created for Owen.

**FEAT-14.SPEC-002-AC-10:** Given Owen invites Priya as a Reviewer contact, when FEAT-18's contact-invitation event fires this automation, then a Notification record is created for Priya.

**FEAT-14.SPEC-002-AC-11:** Given Nadia completes sign-up, when FEAT-20's welcome-email event fires this automation, then a Notification record is created for Nadia.

**FEAT-14.SPEC-002-AC-12:** Given Dana opens a support session on Nadia's account, when FEAT-31's session-opened event fires this automation, then a Notification record is created for Nadia, unaffected by any of Nadia's optional preferences (XBR-29).

**FEAT-14.SPEC-002-AC-13:** Given the payment processor reports a chargeback on a paid invoice, when FEAT-25/FEAT-32's dispute event fires this automation, then a Notification record is created for Nadia (XBR-21).

**FEAT-14.SPEC-002-AC-14:** Given Nadia has switched off an optional notification type, when a triggering event of that type fires, then FEAT-14.SPEC-004's entitlement check returns zero recipients and no Notification record is created.

**FEAT-14.SPEC-002-AC-15:** Given a proposal-sent event fires and the client has one Primary contact (Owen) and one Reviewer contact (Priya), when this automation resolves recipients, then a Notification record is created only for Owen, not Priya.

**FEAT-14.SPEC-002-AC-16:** Given this automation cannot complete resolution or composition for a triggering event due to a processing error, when the failure occurs, then a Notification record is still created for the intended recipient with delivery_status set to Failed, and the triggering feature's own action is not blocked or reverted.

**FEAT-14.SPEC-002-AC-17:** Given Nadia turns off an optional notification type at the exact moment a triggering event of that type fires, when this automation evaluates entitlement, then the preference state at evaluation time wins and no Notification record is created.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 13 | 13 |
| Outcome Paths | 4 (dispatched, no entitled recipients, partial dispatch, resolution failure) | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 5 | 5 |

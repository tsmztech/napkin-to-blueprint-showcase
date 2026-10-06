# FEAT-14 — Notifications (Email)

This chapter covers Notifications (Email), a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 6 specifications carrying 98 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-14.SPEC-001 | Transactional Email Delivery | integration | 14 |
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | automation | 17 |
| FEAT-14.SPEC-003 | Delivery Status Tracking & Retry | automation | 12 |
| FEAT-14.SPEC-004 | Notification Type & Recipient Entitlement Rules | logic-rule | 21 |
| FEAT-14.SPEC-005 | Recognizable & Branded Email Presentation Rules | logic-rule | 19 |
| FEAT-14.SPEC-006 | Delivery Failure Warning to Freelancer | notification | 15 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Notifications (Email)

## Summary

**Feature:** Notifications (Email)
**ID:** FEAT-14
**Description:** Every event that needs to reach a person — a new proposal, a deliverable ready for review, an approval request, an invoice, a reminder, a payment confirmation — is delivered by email, since clients will not install an app.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Ecosystem & Integrations: "all notifications to clients and freelancers go by email. Clients will not install an app." Every other feature's Communications field depends on this delivery mechanism existing. MVP phase: nothing client-facing can reach the client without it. [RESEARCH-INFORMED: client-facing emails landing in spam and messages failing to send are recurring complaints across HoneyBook, Bonsai, Moxie, and SuiteDash (G2, Capterra, Trustpilot, HIGH); because email is the only channel to clients, deliverability is part of this feature, not an afterthought] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Event-triggered email dispatch — every notable event sends the correct email to the correct recipient
- Delivery tracking — failed or bounced sends are surfaced rather than silently lost
- Recognizable, trustworthy emails — every email names the freelancer and the project in its sender name and subject, carries her branding (FEAT-19), and stays plain and consistent so it is not mistaken for spam [RESEARCH-INFORMED: client emails landing in spam are reported for HoneyBook and SuiteDash, and reliability of sending is a category-wide complaint across four products (G2, Capterra, Trustpilot reviews)]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-14.SPEC-001 | Transactional Email Delivery | Integration | All | Sends every composed email through the product's transactional email delivery capability and reports back delivery, bounce, and failure status |
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | Automation | All | The shared dispatch engine every other feature's triggering event calls: creates the Notification record for one triggering event and hands it to the delivery capability |
| FEAT-14.SPEC-003 | Delivery Status Tracking & Retry | Automation | All | Records each notification's delivery outcome, retries a failed send automatically a limited number of times, then hands off to the delivery-failure warning once retries are exhausted |
| FEAT-14.SPEC-004 | Notification Type & Recipient Entitlement Rules | Logic/Rule | All | Governs the one-type-per-event mapping, Access-Matrix-limited recipients, and which notifications a preference can switch off versus which always send |
| FEAT-14.SPEC-005 | Recognizable & Branded Email Presentation Rules | Logic/Rule | All | Governs sender name, subject line, branding, and plain, consistent formatting so every email is identifiable as coming from the freelancer's practice and reads as trustworthy, not spam |
| FEAT-14.SPEC-006 | Delivery Failure Warning to Freelancer | Notification | Nadia (Freelancer), Dana (Support Operator) | Warns Nadia on the affected project when a notification's delivery fails after retries are exhausted; viewable read-only by Dana inside a support session |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Event-triggered email dispatch | FEAT-14.SPEC-002, FEAT-14.SPEC-001 | The dispatch automation creates the correct Notification record for each triggering event from another feature; the Integration spec performs the actual send | Phase 2 (Explicit) |
| Delivery tracking | FEAT-14.SPEC-003, FEAT-14.SPEC-001 | The Integration spec reports delivery and bounce status back inbound; the tracking automation records it against the notification, retries, and surfaces failures rather than losing them silently | Phase 2 (Explicit) |
| Recognizable, trustworthy emails | FEAT-14.SPEC-005 | Dedicated Logic/Rule spec fixes sender-name, subject, branding, and plain-formatting requirements every dispatched email must satisfy | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-14.SPEC-001 | Transactional Email Delivery | Phase 4 (External Dependencies lens) | ASMP-29 names a transactional-email delivery capability, with delivery and bounce status reported back, that this feature relies on; the dependency map's External Touchpoints row expects this capability to be specified by an Integration-type spec owned by this feature -- any trigger-response crossing the product boundary belongs in a standalone Integration spec |
| FEAT-14.SPEC-004 | Notification Type & Recipient Entitlement Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits and Access fields carry conditional, cross-cutting rules (one notification type per triggering event, recipients limited to Access-Matrix-entitled contacts, optional-vs-transactional per XBR-30) that every other feature's Notification spec must follow -- shared rules crossing the standalone-spec threshold |
| FEAT-14.SPEC-006 | Delivery Failure Warning to Freelancer | Phase 4 (Notification surfacing lens) | The Communications field names this feature's own message -- "its own message is limited to delivery-failure warnings to Nadia" -- with a channel, an audience, content, and a delivery-timing rule (after retries exhaust), which requires a standalone Notification spec rather than an inline side-effect |

## Entity-Lifecycle Coverage Matrix

**Entity: Notification**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-14.SPEC-002 | The dispatch engine creates one Notification record per triggering event, set to `Queued`, when another feature's event fires and passes its entitlement check (FEAT-14.SPEC-004) | Other features' own Notification specs (e.g. FEAT-02.SPEC-011) decide *what* content and *when*; this feature's dispatch engine is what actually creates and sends the record |
| Read (single) | FEAT-14.SPEC-003 | The delivery-tracking automation reads a notification's current status to decide whether to retry or surface a warning | Also read by FEAT-31 (delivery warnings, read-only, outside this feature's scope) |
| Read (list) | N/A | No in-feature listing view exists; a freelancer-facing notification history/list is the explicit scope of FEAT-29 (In-App Notification Center), deferred to Later per scope-boundaries.md | This feature's own States field states plainly: "no primary browsing view of its own" |
| Update | FEAT-14.SPEC-003 | Delivery-tracking automation writes `delivery_status` as it changes, fed by inbound status events from FEAT-14.SPEC-001 | -- |
| Delete/Archive | N/A | Dependency map states Notification is "Deleted by FEAT-24" only -- account-deletion is the sole removal path; no in-feature delete or archive exists. This is an explicit non-goal, not an omission: notifications are operational delivery records, not something a person removes individually, and their content already respects role scope so retaining them carries no additional exposure | -- |
| State Transition | FEAT-14.SPEC-003 | Drives `delivery_status` through Queued -> Sent -> Delivered, or Queued -> Sent -> Failed/Bounced -> (retry) -> Delivered or Failed | Failed/Bounced after retries exhausted triggers FEAT-14.SPEC-006 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client Contact | FEAT-14.SPEC-002, FEAT-14.SPEC-004 | Resolves the entitled recipient's name and email address for a given triggering event and role |
| Freelancer Account | FEAT-14.SPEC-002, FEAT-14.SPEC-004 | Reads Nadia's notification preferences to decide whether an optional notification type sends, and her identity for sender-name composition |
| Branding Profile | FEAT-14.SPEC-005 | Reads logo and brand colour to apply to every client-facing email (XBR-31) |
| Project | FEAT-14.SPEC-003, FEAT-14.SPEC-006 | The affected project a delivery warning is attached to and surfaced on |
| Support Access Session | FEAT-14.SPEC-002 | Reads a newly opened or closed session to trigger the session-started/ended notice to Nadia (XBR-29) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A triggering event fires in another feature (e.g. proposal sent, deliverable ready, invoice issued, reminder due) | Resolve the entitled recipient(s) and notification type, then create a Notification record | Standalone Automation | FEAT-14.SPEC-002 |
| A notification is created | Check the recipient's entitlement against the Access Matrix and, for an optional type, the freelancer's notification preferences | Standalone Logic/Rule | FEAT-14.SPEC-004 |
| A notification is created | Compose the sender name, subject line, and body formatting, applying the freelancer's branding | Standalone Logic/Rule | FEAT-14.SPEC-005 |
| A notification is queued | Send it through the transactional email delivery capability | Standalone Integration | FEAT-14.SPEC-001 |
| The delivery capability reports a send outcome (delivered, bounced, failed) | Update the notification's delivery status | Standalone Automation | FEAT-14.SPEC-003 |
| A send attempt fails or bounces | Retry automatically, a limited number of times | Standalone Automation | FEAT-14.SPEC-003 |
| All retries on a notification are exhausted | Warn Nadia on the affected project rather than letting the message vanish silently | Standalone Notification | FEAT-14.SPEC-006 |
| A delivery warning is shown to Nadia | Nadia navigates from the warning to the client contact list to correct a mistyped address | Cross-feature | FEAT-18 responsibility (navigation target) |
| Nadia turns off an optional notification type in Settings | The dispatch engine stops creating notifications of that type for her account; transactional types are unaffected (XBR-30) | Cross-feature | FEAT-21 responsibility (owns notification preferences); enforced here by FEAT-14.SPEC-004 |
| A support session opens or a support request is received | Dispatch the session-started notice or the request-confirmation email to Nadia | Cross-feature | FEAT-31 responsibility (owns the triggering event); dispatched and delivered by FEAT-14.SPEC-002/SPEC-001 |
| A payment processor reports a chargeback or reversal on a paid invoice | Dispatch a notification to Nadia | Cross-feature | FEAT-25/FEAT-32 responsibility (own the triggering event, XBR-21); dispatched and delivered by FEAT-14.SPEC-002/SPEC-001 |
| An email is composed for any client-facing event | The referral mark is included per XBR-32 without overriding branding | Cross-feature | FEAT-33 responsibility (owns the referral mark content); applied at composition time by FEAT-14.SPEC-005 |

## Shared Context

**Shared Entities:**
- Notification -- created by SPEC-002, updated (delivery_status) and state-transitioned by SPEC-003 fed by SPEC-001's inbound status, read by SPEC-003 for retry decisioning and by SPEC-006 for warning content. Fields: notification_type, recipient, sent_at, delivery_status (Queued, Sent, Delivered, Failed, Bounced).
- Client Contact, Freelancer Account, Branding Profile, Project, Support Access Session -- read-only across this feature's specs for recipient resolution, preference checks, branding, warning placement, and support-notice triggering respectively (see Referenced Entities above).

**Shared UI Patterns:**
- N/A -- this feature owns no screens of its own. Its States field states plainly this is "a background dispatch feature with no primary browsing view of its own" (Feature Type: Platform). The email itself is the only surface, and every client-facing email's layout follows the presentation rules in SPEC-005; the delivery-warning display is a small element on the affected project's own screen, owned by FEAT-01, not by this feature.

**Shared Validation:**
- SPEC-004 and SPEC-005 define every recipient-entitlement, notification-typing, and presentation rule this feature enforces. Every other feature's own Notification spec that composes a message for its event (FEAT-02.SPEC-011; FEAT-03.SPEC-006, FEAT-03.SPEC-007; FEAT-05.SPEC-008; FEAT-06.SPEC-006; FEAT-07.SPEC-003, FEAT-07.SPEC-004; FEAT-08.SPEC-007; FEAT-09.SPEC-010; FEAT-10.SPEC-007; FEAT-11.SPEC-004) relies on SPEC-001 for delivery and on SPEC-004/SPEC-005 for entitlement and presentation, rather than each re-deriving them.

**Signals:** `notification_sent` and `notification_bounced` are emitted by SPEC-001/SPEC-003 as delivery outcomes are recorded; `notification_delivery_failed` is emitted by SPEC-003 when retries are exhausted, immediately preceding SPEC-006's dispatch.

## Internal Dependency Map

```
{another feature's triggering event} -> [event fires] -> SPEC-002 (Notification Composition & Dispatch)
SPEC-002 (Notification Composition & Dispatch) -> [checks entitlement and preferences using] -> SPEC-004 (Notification Type & Recipient Entitlement Rules)
SPEC-002 (Notification Composition & Dispatch) -> [composes sender, subject, branding using] -> SPEC-005 (Recognizable & Branded Email Presentation Rules)
SPEC-002 (Notification Composition & Dispatch) -> [queues and sends via] -> SPEC-001 (Transactional Email Delivery)
SPEC-001 (Transactional Email Delivery) -> [reports delivery/bounce status] -> SPEC-003 (Delivery Status Tracking & Retry)
SPEC-003 (Delivery Status Tracking & Retry) -> [send failed, retry] -> SPEC-001 (Transactional Email Delivery)
SPEC-003 (Delivery Status Tracking & Retry) -> [retries exhausted] -> SPEC-006 (Delivery Failure Warning to Freelancer)
SPEC-006 (Delivery Failure Warning to Freelancer) -> [Nadia corrects the contact's address] -> FEAT-18 (client contact list)
```

**Default Entry:** N/A -- this feature has no screen and no navigable entry point of its own (Feature Type: Platform; States field: "no primary browsing view of its own"). SPEC-002 is the functional starting point, invoked only by another feature's triggering event, never navigated to directly.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-14.SPEC-002 | Inbound | FEAT-02 (Proposal Creation & Sending) | FEAT-02's own Notification spec (FEAT-02.SPEC-011) calls this feature's dispatch engine to actually send the proposal email | Nadia sends a proposal |
| FEAT-14.SPEC-002 | Inbound | FEAT-03 (Proposal Acceptance) | FEAT-03's Notification specs (FEAT-03.SPEC-006, FEAT-03.SPEC-007) call this feature's dispatch engine | Owen accepts or requests changes on a proposal |
| FEAT-14.SPEC-002 | Inbound | FEAT-05 (Client Portal Access, Magic-Link Login) | FEAT-05's Notification spec (FEAT-05.SPEC-008) calls this feature's dispatch engine for the magic-link sign-in email | A contact requests a sign-in link, or requests a fresh one after expiry |
| FEAT-14.SPEC-002 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | FEAT-06's Notification spec (FEAT-06.SPEC-006) calls this feature's dispatch engine, only once an upload is fully complete (XBR-12) | A deliverable becomes ready to review |
| FEAT-14.SPEC-002 | Inbound | FEAT-07 (Deliverable Review & Feedback) | FEAT-07's Notification specs (FEAT-07.SPEC-003, FEAT-07.SPEC-004) call this feature's dispatch engine | A comment is left on a deliverable |
| FEAT-14.SPEC-002 | Inbound | FEAT-08 (Milestone Approval) | FEAT-08's Notification spec (FEAT-08.SPEC-007) calls this feature's dispatch engine | Owen approves a milestone or requests changes |
| FEAT-14.SPEC-002 | Inbound | FEAT-09 (Invoice Generation & Sending) | FEAT-09's Notification spec (FEAT-09.SPEC-010) calls this feature's dispatch engine | An invoice is generated and issued, manually or auto-issued on approval |
| FEAT-14.SPEC-002 | Inbound | FEAT-10 (Invoice Payment Processing) | FEAT-10's Notification spec (FEAT-10.SPEC-007) calls this feature's dispatch engine | A payment succeeds |
| FEAT-14.SPEC-002 | Inbound | FEAT-11 (Automated Payment Reminders) | FEAT-11's Notification spec (FEAT-11.SPEC-004) calls this feature's dispatch engine | A day-3 or day-10 automatic reminder, or a one-click manual reminder, fires |
| FEAT-14.SPEC-002 | Inbound | FEAT-18 (Client Contact Management) | Contact invitation and role events call this feature's dispatch engine; recipient resolution reads the Client Contact records FEAT-18 owns | Nadia or a Primary contact invites a Reviewer contact |
| FEAT-14.SPEC-006 | Outbound | FEAT-18 (Client Contact Management) | A bounced invitation's delivery warning navigates Nadia to the contact list to correct the address | Nadia opens the delivery warning on the affected project |
| FEAT-14.SPEC-005 | Inbound | FEAT-19 (Branding) | Reads the freelancer's Branding Profile to apply logo and colour to every dispatched email (XBR-31) | Any client-facing email is composed |
| FEAT-14.SPEC-002 | Inbound | FEAT-20 (Freelancer Onboarding & Sign-Up) | The welcome-email event calls this feature's dispatch engine | Nadia completes sign-up |
| FEAT-14.SPEC-004 | Outbound | FEAT-21 (Notification Preferences & Settings) | FEAT-21's settings screen lists the optional notification types this feature defines, for Nadia to toggle; SPEC-004 enforces that transactional types cannot be switched off (XBR-30) | Nadia opens her notification preferences |
| FEAT-14.SPEC-002 | Inbound | FEAT-24 (Account Export & Deletion) | Notification records for the account are deleted as part of account deletion; this feature performs no in-feature delete of its own | Nadia's account is deleted |
| FEAT-14.SPEC-002 | Inbound | FEAT-25 (Invoice/Project Cancellation & Disputes) | A chargeback or reversal event calls this feature's dispatch engine to notify Nadia (XBR-21) | The payment processor reports a dispute on a paid invoice |
| FEAT-14.SPEC-002 | Outbound | FEAT-29 (In-App Notification Center, Later) | Notification records this feature creates are the data source a future in-app feed would read; no in-feature list view exists yet | Deferred to Later per scope-boundaries.md |
| FEAT-14.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | A support request or support-session open/close event calls this feature's dispatch engine to notify Nadia (XBR-29) | Nadia sends a support request, or Dana opens/closes a session |
| FEAT-14.SPEC-006 | Outbound | FEAT-31 (Operator Support Access) | Dana views delivery warnings read-only inside her logged support session | Dana opens a support session on an account with an open delivery warning |
| FEAT-14.SPEC-002 | Inbound | FEAT-32 (Payment Processor Integration) | A processor-reported dispute event calls this feature's dispatch engine (XBR-21) | The payment processor reports a chargeback or reversal |
| FEAT-14.SPEC-005 | Inbound | FEAT-33 (Portal Referral Attribution) | Every client-facing email includes the referral mark FEAT-33 owns, applied without overriding branding (XBR-32) | Any client-facing email is composed |

## Non-Functional Notes

**Data volumes / growth:** A Notification record is created for nearly every event across every other feature this one depends on (proposals, deliverables, approvals, invoices, reminders, sign-in links, support notices), so volume scales directly with total product activity across a few thousand freelancers, each with 3–15 active clients (scope-boundaries.md, SC-21); dispatch and delivery-status tracking must stay reliable at that combined event volume, not just at one feature's own volume.

**Responsiveness:** Delivery failures must be surfaced to the freelancer within minutes rather than silently lost (ASMP-26); this feature's Data Notes field states delivery warnings are "displayed... to Nadia when relevant," so SPEC-003's retry-then-warn handoff to SPEC-006 is timing-sensitive, not a background best-effort job.

**Data sensitivity / privacy:** Notification records hold personal data of the recipient (name and email address) and message content that can reference a client's project, classified GDPR-class personal data (ASMP-24); content strictly respects role scope so a Reviewer contact never receives invoice content and a client never receives another client's information (ASMP-23), and Dana's read of a delivery warning is limited to the warning itself, never the underlying message content she isn't otherwise entitled to.

**Compliance flags:** GDPR-class handling applies to every notification's recipient and content data (ASMP-24); no card or payment data is ever carried in a notification, since that handling belongs entirely to the payment-processing capability used by FEAT-10 and FEAT-32.

## Non-Goals

- **An in-app notification feed or history view** -- Deferred per scope-boundaries.md: "In-App Notification Center" (FEAT-29) is explicitly Target Phase Later, "because email already satisfies BRIEF.md's stated notification channel"; this feature's own States field confirms it has "no primary browsing view of its own" in MVP.
- **Any notification channel other than email (SMS, push, in-app)** -- Excluded per BRIEF.md, Ecosystem & Integrations: "all notifications to clients and freelancers go by email. Clients will not install an app." Email is the only notification channel in MVP (scope-boundaries.md, deferral note on FEAT-29).
- **A configurable notification or workflow builder** -- Excluded per scope-boundaries.md SC-11: this product "ships fixed, sensible behavior" for its triggers (e.g., auto-invoice on approval, day-3/day-10 reminders) rather than a freelancer-configurable rules or automation builder; notification triggers and content are fixed by each owning feature, not customizable.
- **A general-purpose messaging or chat inbox between freelancer and client** -- Excluded per scope-boundaries.md SC-15: feedback stays pinned to deliverables and milestones (FEAT-07) and proposal change-requests (FEAT-03) rather than becoming another open channel; this feature only ever delivers discrete, event-triggered transactional emails.
- **In-feature deletion or archival of individual notifications** -- Intentional lifecycle decision surfaced by the CRUD matrix: the dependency map states Notification records are "Deleted by FEAT-24" only, as part of full account deletion; they are operational delivery records, not something a person curates or removes individually, so no in-feature delete/archive path exists and none is needed.
- **Scoped notification-management permissions for a freelancer-side team** -- Excluded per scope-boundaries.md SC-01: the product is solo-freelancer only for v1, so notification preferences and delivery-warning visibility have exactly the roles modeled here (Nadia manages preferences; Dana views warnings read-only); no bookkeeper- or contractor-style scoped access exists.



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



# Automation Spec: Delivery Status Tracking & Retry

## Overview

**Name:** Delivery Status Tracking & Retry
**ID:** FEAT-14.SPEC-003
**Type:** Automation
**Purpose:** Records each notification's delivery outcome, retries a failed send automatically a limited number of times, then hands off to the delivery-failure warning once retries are exhausted or the failure is permanent.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- Recording delivery, bounce, and failure outcomes against the Notification record
- Retrying a transient send failure automatically, within a bounded count and window
- Distinguishing a permanent failure (bounce) from a transient one for retry purposes
- Handing off to the delivery-failure warning once retries are exhausted or a bounce is final

**Non-Goals:**
- Creating the Notification record or resolving recipient entitlement -- owned by FEAT-14.SPEC-002 and FEAT-14.SPEC-004; this spec only updates records that already exist.
- The actual transmission of an email, and the raw delivered/bounced/failed reporting itself -- owned by FEAT-14.SPEC-001; this spec consumes that spec's inbound events.
- The content, channel, and delivery rules of the delivery-failure warning itself -- owned by FEAT-14.SPEC-006; this spec only triggers it once retries are exhausted.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Notification delivery outcome reported | FEAT-14.SPEC-001 (Transactional Email Delivery) | Fires whenever the delivery capability reports Delivered, Bounced, or Failed for a Queued or previously-Failed Notification | Notification reference, outcome, bounce/failure reason category if provided |
| Retry interval elapsed for a Failed notification | System (schedule-based, internal to this automation) | Fires when a Failed notification's next scheduled retry attempt is due, within platform parameter: `transactional-email-retry-window` of the first attempt | Notification reference, retry attempt count so far |

## Processing Logic

1. Receive the delivery outcome reported by FEAT-14.SPEC-001 for a Notification.
2. If the outcome is Delivered: set Notification.delivery_status to Delivered and stop -- no further action.
3. If the outcome is Bounced (a permanent failure -- the address itself is invalid): set Notification.delivery_status to Bounced. A bounce is never retried, since retrying an identical, invalid address would produce the same outcome again and only delay the warning Nadia needs; proceed directly to Step 6.
4. If the outcome is Failed (a transient failure): set Notification.delivery_status to Failed. If the Notification's retry count is below platform parameter: `transactional-email-retry-count`, schedule the next retry attempt (re-invoking FEAT-14.SPEC-001) within platform parameter: `transactional-email-retry-window` of the first attempt, and increment the retry count.
5. When a scheduled retry's outcome is reported, return to Step 2 (Delivered) or repeat Step 3/4 as applicable, using the current retry count.
6. Once retries are exhausted (the retry count has reached platform parameter: `transactional-email-retry-count` with no Delivered outcome) or a Bounce occurred, finalize Notification.delivery_status as Bounced or Failed and hand off to FEAT-14.SPEC-006 so Nadia is warned on the affected project.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Delivered | The capability confirms delivery, on the first attempt or any retry | Notification.delivery_status set to Delivered | None | -- |
| Bounced -- immediate warning | The capability reports a permanent bounce | Notification.delivery_status set to Bounced | Delivery warning appears on the affected project for Nadia | FEAT-14.SPEC-006 |
| Failed -- retry scheduled | A transient failure occurs and the retry count is below platform parameter: `transactional-email-retry-count` | Notification.delivery_status set to Failed; retry count incremented; a retry is scheduled | None yet -- retrying is not surfaced to Nadia unless and until it is exhausted | -- |
| Failed -- retries exhausted | A transient failure recurs until the retry count reaches platform parameter: `transactional-email-retry-count` with no Delivered outcome | Notification.delivery_status finalized as Failed | Delivery warning appears on the affected project for Nadia | FEAT-14.SPEC-006 |
| Automation failure (tracking itself cannot process an outcome report, e.g., the Notification reference cannot be resolved) | A processing error occurs within this automation, distinct from the delivery outcome itself | No Notification status change is applied; the outcome report is not lost -- it is re-processed on the automation's next run | None immediately; non-blocking to any triggering screen, since this automation never blocks a user action | -- |

## Data Model

**Reads:** Notification -- delivery_status, notification_type, recipient, sent_at, retry count; Project -- the affected project a delivery warning is attached to, read to hand off to FEAT-14.SPEC-006.
**Creates:** None -- this automation never creates a Notification; that is owned by FEAT-14.SPEC-002.
**Updates:** Notification -- delivery_status (Queued -> Sent -> Delivered, or Queued -> Sent -> Failed/Bounced -> (retry) -> Delivered or Failed).
**Deletes:** None.

## Business Rules

- Delivery failures are surfaced to the freelancer within minutes rather than silently lost (ASMP-26) -- retries proceed promptly within platform parameter: `transactional-email-retry-window`, not delayed for days.
- A Bounce (permanent) is never retried, since a retry against an invalid address would reproduce the identical outcome and only delay the warning Nadia needs.
- A transient Failed outcome retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` -- the same retry contract every triggering feature's own Notification spec already cites when describing this capability's behavior (e.g., FEAT-02.SPEC-011, FEAT-05.SPEC-008, FEAT-09.SPEC-010).
- Delivery status transitions move toward a terminal state (Delivered, or Failed/Bounced-after-retries) only; a Notification never regresses from Delivered back to an earlier state.
- At most one delivery-failure warning is generated per Notification, regardless of how many duplicate outcome reports arrive afterward -- deduplicated here before FEAT-14.SPEC-006 is ever invoked.

## Edge Cases

- **Concurrent trigger firing -- two outcome reports for the same Notification arrive at effectively the same time (e.g., a retry's Failed report and a stale earlier report)** -- The automation applies the most recent true outcome by the event's own occurrence time (per FEAT-14.SPEC-001), so a later Delivered is never overwritten by an earlier, stale Failed report.
- **Trigger fires while a previous run is in flight -- a retry attempt's outcome report arrives while this automation is still applying the prior attempt's outcome for the same Notification** -- Processing for a single Notification is serialized: the second report waits for the first to finish applying its status change before being applied, so delivery_status transitions never interleave inconsistently for one Notification. Runs for different Notifications proceed independently and do not queue behind each other.
- **A Notification reaches its final retry count boundary at the same moment that final retry succeeds** -- The successful Delivered outcome wins over the "retries exhausted" determination; a late success is always honored, since the entire purpose of retrying is to catch it.
- **The affected project is archived between a Failed outcome and the retries-exhausted moment** -- The delivery warning still surfaces per FEAT-14.SPEC-006, since archiving removes a project from active view without erasing its record, and a bouncing contact address may still need correcting.
- **The underlying account is deleted (FEAT-24) while a Notification is mid-retry** -- Retry processing for that Notification stops; the Notification record itself is removed as part of account deletion, consistent with FEAT-14.SPEC-001's edge-case handling for deleted accounts.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggered by (inbound) | Every delivery outcome this spec processes originates from this integration's inbound events, and retry attempts are re-issued back through it |
| FEAT-14.SPEC-002 (Notification Composition & Dispatch) | References (inbound) | Reads the Notification records this automation created |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | Fires once retries are exhausted or a bounce is final |

## Analytics and Success Signals

- **notification_retry_attempted** (notification_type, attempt_number) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_delivery_succeeded_after_retry** (notification_type, attempt_number) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_delivery_failed_final** (notification_type, reason: bounced / retries_exhausted) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given a Notification is Queued for Owen, when FEAT-14.SPEC-001 reports Delivered on the first attempt, then Notification.delivery_status is set to Delivered and no retry occurs.

**FEAT-14.SPEC-003-AC-02:** Given a Notification's send is reported Bounced, when this automation processes the outcome, then Notification.delivery_status is set to Bounced immediately and no retry is scheduled.

**FEAT-14.SPEC-003-AC-03:** Given a Notification's send is reported Failed for a transient reason, when this automation processes the outcome and the retry count is below platform parameter: `transactional-email-retry-count`, then Notification.delivery_status is set to Failed and a retry is scheduled.

**FEAT-14.SPEC-003-AC-04:** Given a Notification has been retried and reaches platform parameter: `transactional-email-retry-count` attempts with no Delivered outcome, when the final retry's outcome is processed, then Notification.delivery_status is finalized as Failed and FEAT-14.SPEC-006 is triggered.

**FEAT-14.SPEC-003-AC-05:** Given a bounced Notification, when this automation finalizes it, then FEAT-14.SPEC-006 is triggered immediately without waiting for any retry count.

**FEAT-14.SPEC-003-AC-06:** Given a Notification succeeds on its second retry attempt, when the Delivered outcome is processed, then Notification.delivery_status is set to Delivered and no delivery warning is ever generated.

**FEAT-14.SPEC-003-AC-07:** Given two outcome reports for the same Notification arrive at effectively the same time, when this automation processes them, then the outcome that occurred more recently (by its own event time) is the one reflected, regardless of arrival order.

**FEAT-14.SPEC-003-AC-08:** Given a retry outcome report arrives while this automation is still applying the prior report's status change for the same Notification, when both are processed, then they are applied in sequence and the Notification never shows an inconsistent intermediate state.

**FEAT-14.SPEC-003-AC-09:** Given a Notification's final retry succeeds at the exact moment its retry count reaches the limit, when both conditions are evaluated together, then the Delivered outcome takes precedence over marking the notification Failed.

**FEAT-14.SPEC-003-AC-10:** Given the affected project is archived while a Notification is retrying, when retries are later exhausted, then the delivery warning still appears per FEAT-14.SPEC-006.

**FEAT-14.SPEC-003-AC-11:** Given the freelancer's account is deleted while a Notification is mid-retry, when the deletion completes, then retry processing for that Notification stops and the record is removed with the account.

**FEAT-14.SPEC-003-AC-12:** Given this automation cannot resolve a Notification reference due to a processing error, when the outcome report cannot be applied, then no status change is made, no user is blocked, and the report is re-processed on the automation's next run.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (outcome reported, retry interval elapsed) | 2 |
| Outcome Paths | 5 (delivered, bounced-immediate, failed-retry-scheduled, failed-exhausted, automation failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Notification Type & Recipient Entitlement Rules

## Overview

**Name:** Notification Type & Recipient Entitlement Rules
**ID:** FEAT-14.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs the one-type-per-event mapping, Access-Matrix-limited recipients, and which notifications a preference can switch off versus which always send.
**Parent Feature:** FEAT-14 -- Notifications (Email)
**Governed Entity:** Notification

## Scope and Non-Goals

**In Scope:**
- The complete registry of notification types, one per triggering event across the product, with its entitled recipient role(s)
- Recipient entitlement checking against the Access Matrix for every notification type
- The transactional-versus-optional classification for every notification type, and the preference-override rule (XBR-30)
- Field-level rules for the Notification entity and authorization rules for every action on it

**Non-Goals:**
- Composing the email's actual subject, body, sender name, or branding -- owned by FEAT-14.SPEC-002 (dispatch) and FEAT-14.SPEC-005 (presentation rules); this spec decides *who* receives *which type*, not what the email says or looks like.
- The mechanics of creating the Notification record or handing it to the delivery capability -- owned by FEAT-14.SPEC-002; this spec supplies the rules that automation calls.
- The screen where Nadia actually toggles an optional preference -- owned by FEAT-21 (Settings & Account Management) as FEAT-21.SPEC-002 (Notification Preferences); this spec defines which types are eligible to be toggled and enforces the outcome, not the settings screen itself.
- Delivery, bounce, and retry status tracking -- owned by FEAT-14.SPEC-001 and FEAT-14.SPEC-003; this spec governs only which Notification gets created and for whom, not what happens to it afterward.

## Governed Entity

**Entity:** Notification
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_type | enum | Exactly one triggering event per type (required) |
| recipient | reference | A Client Contact or the Freelancer Account entitled to the triggering event per the Access Matrix (required) |
| sent_at | date | Timestamp of the dispatch attempt (required, system-derived) |
| delivery_status | enum | Queued, Sent, Delivered, Failed, Bounced (required, system-derived) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | On every triggering event, before a Notification record is created -- resolves notification_type and entitled recipient(s), and checks preference state for any type this spec classifies Optional |
| FEAT-14.SPEC-003 | Delivery Status Tracking & Retry | Reads delivery_status transitions this spec's field rules constrain to a one-directional terminal state machine |
| FEAT-21.SPEC-002 | Notification Preferences (FEAT-21) | Lists only the notification types this spec classifies Optional for Nadia to toggle; a type this spec classifies Transactional is never offered as a toggle |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_type | Must be one of the fixed registry of types defined in Business Rules below -- exactly one type per triggering event, never a free-text or ad hoc value | Always | On create (FEAT-14.SPEC-002) | Internal invariant, never user-facing: "notification_type must map to a registered triggering event" -- a type outside the registry is a defect in the triggering feature's own spec, not a runtime condition a user encounters | Yes |
| recipient | Must be a Client Contact or the Freelancer Account that is entitled to this notification_type per the Authorization Rules and registry below | Always | On create (FEAT-14.SPEC-002) | Internal invariant, never user-facing: "recipient must be entitled to notification_type" -- an unentitled recipient is never resolved in the first place, so no Notification record for them is ever attempted (see FEAT-14.SPEC-002's No Entitled Recipients outcome) | Yes |
| sent_at | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| delivery_status | Must be one of Queued, Sent, Delivered, Failed, Bounced, and may only transition toward a terminal state (never regress from Delivered to an earlier state), per FEAT-14.SPEC-003's state machine | Always | On update (FEAT-14.SPEC-003) | Internal invariant, never user-facing: "delivery_status transitions must be forward-only" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Recipient-entitlement-by-type | notification_type, recipient | The recipient's role (Nadia, Owen, Priya) must appear in the entitled-role set for that notification_type in the registry (Business Rules); a role not in that set is never resolved as a recipient for that type | Internal invariant, never user-facing -- a non-entitled contact simply never has a Notification record created for that type (FEAT-14.SPEC-002's No Entitled Recipients / Partial Dispatch outcomes); no error is ever shown to any human |
| Preference-gates-optional-types-only | notification_type, recipient (Freelancer Account -- notification_preferences) | A preference toggle may govern a notification_type only if that type is classified Optional in the registry; a Transactional type ignores the preference state entirely and always resolves its entitled recipients (XBR-30) | Internal invariant -- FEAT-21's preferences screen is never offered a toggle for a Transactional type in the first place, so this condition cannot be violated through the product's own UI |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Notification record | System, via FEAT-14.SPEC-002, following this spec's entitlement check | Always, when at least one recipient is entitled | -- |
| Receive (be resolved as the recipient of) a Notification | Nadia (Freelancer) | For every notification_type whose registry entry entitles the Freelancer Account (all types addressed to her practice or account) | Not applicable -- Nadia is entitled to every notification type that names her; no denial path exists for her own account's notifications |
| Receive (be resolved as the recipient of) a Notification | Owen (Client Primary Contact) | Only for notification_types whose registry entry entitles the Primary Contact role, and only for his own client company (Own-only) | He is simply never resolved as a recipient for a type that does not entitle Primary contacts (e.g., an invoice-content type addressed only within the freelancer's own account is not applicable here, but any type restricted to Nadia); no error is shown anywhere, since the concept of "denial" does not surface to a contact who was never a candidate recipient |
| Receive (be resolved as the recipient of) a Notification | Priya (Client Reviewer Contact) | Only for notification_types whose registry entry entitles the Reviewer Contact role, and only for her own client company (Own-only); she is never entitled to proposal-content or invoicing-content types, per the Access Matrix | She is silently never resolved as a recipient for a type outside her entitlement (e.g., proposal_sent, invoice_issued, payment_confirmation, overdue_reminder); no error or "hidden" state is shown to her, since she is not a candidate recipient for those types in the first place |
| Receive (be resolved as the recipient of) a Notification | Dana (Support Operator) | Never -- Dana is never a recipient of any Notification; her Notifications & Help access is "View (delivery warnings only)" | Dana is never resolved as a recipient by this spec's rules under any notification_type |
| View delivery status / a delivery warning | Nadia (Freelancer) | Full -- sees delivery warnings for any Notification tied to her own account's projects | -- |
| View delivery status / a delivery warning | Dana (Support Operator) | View only, delivery warnings only (FEAT-14.SPEC-006), inside a logged support session (FEAT-31); never the underlying subject/body content of the notification itself | A direct attempt to view a Notification's full content is never offered to Dana; her support session surfaces only the delivery-warning summary defined by FEAT-14.SPEC-006 |
| View delivery status / a delivery warning | Owen, Priya (Client Contacts) | Never -- client contacts have no view into Notification records or delivery status as data; they only ever receive the email itself | No delivery-status view exists anywhere in the client portal; the concept of a Notification record is invisible to client contacts entirely |
| Update delivery_status | System, via FEAT-14.SPEC-003, following the defined state machine | Always, forward-only | -- |
| Toggle a notification_type's preference on/off | Nadia (Freelancer) | Only for a notification_type classified Optional in the registry below | An attempt to toggle a Transactional type is never offered as a control in FEAT-21's preferences screen; it does not exist as an actionable element |
| Delete a Notification record | No one, in-feature | Never -- Notification records are removed only by FEAT-24 (account deletion) | No delete action exists anywhere in the product for an individual Notification record |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_type | Derived from the registry mapping (Business Rules) based on which triggering feature/spec fired | On create | No -- never manually chosen by any user |
| recipient | Derived from entitlement resolution against the Access Matrix and, for an Optional type, the current preference state | On create | No -- never manually chosen by any user |
| delivery_status | Defaults to Queued | On create | No |
| sent_at | Set to the time FEAT-14.SPEC-001 confirms a transmission attempt was made | On the first dispatch attempt | No |

## Business Rules

**Notification Type Registry** -- the complete, closed set of notification types this product defines in MVP, one per triggering event (Validation & Limits, feature-overview.md), with its entitled recipient role(s) and its transactional-versus-optional classification:

| Notification Type | Triggering Feature / Spec | Entitled Recipient Role(s) | Classification |
|---|---|---|---|
| proposal_sent | FEAT-02 (FEAT-02.SPEC-011) | Owen (Primary) | Transactional |
| proposal_accepted_confirmation | FEAT-03 (FEAT-03.SPEC-006) | Owen (Primary), Nadia | Transactional |
| proposal_change_requested | FEAT-03 (FEAT-03.SPEC-007) | Nadia | Transactional |
| magic_link_sign_in | FEAT-05 (FEAT-05.SPEC-008) | Owen (Primary) or Priya (Reviewer) -- whichever contact requested | Transactional |
| deliverable_ready | FEAT-06 (FEAT-06.SPEC-006) | Owen (Primary), Priya (Reviewer) | Transactional |
| client_comment_alert | FEAT-07 (FEAT-07.SPEC-003) | Nadia | Transactional |
| freelancer_reply_alert | FEAT-07 (FEAT-07.SPEC-004) | Owen (Primary), Priya (Reviewer) | Transactional |
| milestone_approved_confirmation | FEAT-08 (FEAT-08.SPEC-007) | Owen (Primary), Nadia | Transactional |
| invoice_issued | FEAT-09 (FEAT-09.SPEC-010) | Owen (Primary), Nadia | Transactional |
| payment_confirmation | FEAT-10 (FEAT-10.SPEC-007) | Owen (Primary), Nadia | Transactional |
| overdue_reminder | FEAT-11 (FEAT-11.SPEC-004) | Owen (Primary) | Transactional |
| contact_invitation | FEAT-18 (FEAT-18.SPEC-010) | The newly invited contact (Owen or Priya) | Transactional |
| welcome_email | FEAT-20 (FEAT-20.SPEC-006) | Nadia | Transactional |
| support_session_notice | FEAT-31 (FEAT-31.SPEC-007) | Nadia | Transactional (XBR-29 mandates it is "always announced to the freelancer by email") |
| chargeback_notice | FEAT-25 (FEAT-25.SPEC-008), from the reversal notice FEAT-32.SPEC-002 relays | Nadia | Transactional (XBR-21) |
| delivery_failure_warning | FEAT-14.SPEC-006 (this feature) | Nadia (Dana views it read-only, never as a recipient) | Transactional (cannot itself be turned off, or a lost email's warning could itself be silently lost) |
| primary_invited_colleague_alert | FEAT-18 (FEAT-18.SPEC-011), fired by FEAT-18.SPEC-004 | Nadia (never a client contact) | Transactional (access-relevant change to who can see her work; parallel to XBR-29) |
| sign_in_email_change_confirmation | FEAT-21 (FEAT-21.SPEC-011), fired by FEAT-21.SPEC-005 | Nadia -- one copy to her prior sign-in email and one to her new sign-in email, both her own account | Transactional (XBR-30; security-sensitive credential change) |
| plan_change_confirmation | FEAT-23 (FEAT-23.SPEC-008), fired by FEAT-23.SPEC-004 and FEAT-23.SPEC-006 | Nadia | Transactional (XBR-30) -- one type covering the upgrade, downgrade, cancellation-confirmed, paid-plan-ended and lapsed variants, since each is a different outcome of the same plan-change event |
| subscription_charge_failed | FEAT-23 (FEAT-23.SPEC-008), fired by FEAT-23.SPEC-003 via FEAT-23.SPEC-004 | Nadia | Transactional (XBR-30; account-standing alert) |
| data_export_ready | FEAT-24 (FEAT-24.SPEC-008), fired by FEAT-24.SPEC-003 | Nadia | Transactional (XBR-30; confirms an action she requested) |
| account_deletion_final_warning | FEAT-24 (FEAT-24.SPEC-009), fired by FEAT-24.SPEC-002 | Nadia | Transactional (XBR-30; record of an irreversible action) |
| refund_recorded | FEAT-25 (FEAT-25.SPEC-007), fired by FEAT-25.SPEC-003 | Owen (Primary) only -- never Priya (no billing visibility), never Nadia (the actor) | Transactional (XBR-30) |
| project_cancelled | FEAT-25 (FEAT-25.SPEC-007), fired by FEAT-25.SPEC-004 | Owen (Primary) only -- never Priya, never Nadia | Transactional (XBR-30) |
| signed_copy_confirmation | FEAT-26 (FEAT-26.SPEC-004), fired by FEAT-26.SPEC-002 | Owen (Primary, the signing contact), Nadia | Transactional (XBR-30; evidentiary record, sent in addition to proposal_accepted_confirmation) |
| custom_domain_verified_confirmation | FEAT-27 (FEAT-27.SPEC-004), fired by FEAT-27.SPEC-002 | Nadia | Transactional (XBR-30) |
| support_request_confirmation | FEAT-31 (FEAT-31.SPEC-006), fired by FEAT-31.SPEC-001 | Nadia | Transactional (XBR-30; receipt of her own request) |
| payment_connection_connected | FEAT-32 (FEAT-32.SPEC-006), fired by FEAT-32.SPEC-003 | Nadia | Transactional (XBR-30) |
| payment_connection_needs_attention | FEAT-32 (FEAT-32.SPEC-006), fired by FEAT-32.SPEC-003 | Nadia | Transactional (XBR-30; affects whether she can get paid) |

**Zero notification types are currently classified Optional.** Every type registered above is either an evidentiary or action-required record for a client (proposal, deliverable, approval, invoice, payment, reminder), a security- or access-critical message (magic-link sign-in, contact invitation), a transparency mandate the product itself commits to (support-session notice, XBR-29), a financial-integrity notice (chargeback, XBR-21), or an account-critical confirmation to Nadia (sign-in email change, plan and billing changes, payment-connection status, custom-domain verification, data export, account deletion, support-request receipt, invited-colleague alert, refund and cancellation records, signed copy) -- none is a discretionary, marketing-style alert. This is consistent with every sibling Notification spec in this run (FEAT-02.SPEC-011, FEAT-03.SPEC-006/007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003/004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004, FEAT-18.SPEC-010/011, FEAT-20.SPEC-006, FEAT-21.SPEC-011, FEAT-23.SPEC-008, FEAT-24.SPEC-008/009, FEAT-25.SPEC-007/008, FEAT-26.SPEC-004, FEAT-27.SPEC-004, FEAT-31.SPEC-006/007, FEAT-32.SPEC-006), each of which independently states it is transactional and cannot be disabled, citing XBR-30. The preference mechanism itself is still fully specified and enforced here (Cross-Field Rules, Authorization Rules) exactly as XBR-30 requires, so any future notification type this product adds can be classified Optional by an update to this registry without any change to the dispatch or entitlement machinery.

- Registry coverage is complete: every Notification spec in the product (26 across FEAT-02 through FEAT-32 and FEAT-14.SPEC-006) has at least one row above (29 rows in all). A spec that reports two distinct triggering events (FEAT-23.SPEC-008: plan change and failed charge; FEAT-25.SPEC-007: refund and cancellation; FEAT-32.SPEC-006: connected and needs attention) has one row per event; a spec whose variants are outcomes of one event (FEAT-23.SPEC-008's plan-change variants, FEAT-11.SPEC-004's day-3, day-10 and manual reminders) has a single row.
- Each notification_type maps to exactly one triggering event, avoiding duplicate or missing emails for the same underlying occurrence (feature-overview.md, Validation & Limits).
- Recipients are limited to contacts entitled to the event per the Access Matrix (XBR-08) -- a Reviewer contact never receives invoice or proposal-content emails; a client never receives another client's information (ASMP-23).
- A Transactional type's recipient resolution never consults notification_preferences; an Optional type's recipient resolution consults the Freelancer Account's current notification_preferences at the moment FEAT-14.SPEC-002 runs its entitlement check, not at some earlier moment (XBR-30).
- Dana (Support Operator) is never entitled as a recipient of any notification_type; her only relationship to a Notification is viewing its delivery-warning summary, read-only, inside a logged support session (FEAT-31), per the Access Matrix's "View (delivery warnings only)."
- This registry is the single source of truth FEAT-21's notification-preferences screen (FEAT-21.SPEC-002) lists from; that screen only ever offers a toggle for a type classified Optional here.

## Edge Cases

- **A notification_type is entitled to both a Primary and a Reviewer contact, but the client currently has no Reviewer contact on record** -- The registry entry's entitlement is unaffected; a Notification record is simply created only for the contact(s) that currently exist and are entitled (e.g., freelancer_reply_alert creates a record for Owen alone if no Priya-equivalent contact exists for that client).
- **A client has two Primary contacts** -- Every current Primary contact receives their own Notification record and their own copy of the email for a type entitling the Primary role, per FEAT-02.SPEC-011's own edge case; this spec's entitlement rule applies per-contact, not per-role-slot.
- **A contact's role changes (Reviewer promoted to Primary, or vice versa) between the triggering event firing and this spec's entitlement check running** -- The role in effect at the moment FEAT-14.SPEC-002 runs the entitlement check governs, consistent with the preference-evaluated-at-dispatch-time rule; a role change made a moment before the check is honored, one made a moment after is not (XBR-08, FEAT-18 owns role changes).
- **A notification_type this registry classifies Transactional is mistakenly offered as a toggle by a future change to FEAT-21** -- This spec's Cross-Field Rule (Preference-gates-optional-types-only) is the authoritative boundary: even if a toggle were surfaced, the entitlement check here ignores preference state entirely for a Transactional type, so the type continues to send regardless.
- **A future notification type is added to the product with no registry entry yet** -- Per Field Validation Rules, notification_type must map to a registered triggering event; an unregistered type is a defect in the triggering feature's own spec to be caught before that feature's specs pass review, not a runtime condition this spec's entitlement engine is expected to handle gracefully.

## Acceptance Criteria

**FEAT-14.SPEC-004-AC-01:** Given a triggering event fires with a notification_type that matches an entry in the registry, when FEAT-14.SPEC-002 checks this spec's rules, then the type is accepted and entitlement resolution proceeds.

**FEAT-14.SPEC-004-AC-02:** Given a Notification record is being created, when its recipient is resolved, then the recipient's role must appear in that notification_type's entitled-role set, or no record is created for that would-be recipient.

**FEAT-14.SPEC-004-AC-03:** Given a Notification's delivery_status is Delivered, when any subsequent event is processed, then delivery_status never regresses to an earlier state (Queued, Sent, Failed, or Bounced).

**FEAT-14.SPEC-004-AC-04:** Given a proposal_sent event fires for a client with Owen as Primary and Priya as Reviewer, when recipients are resolved, then a Notification record is created for Owen and none for Priya, since proposal_sent entitles only the Primary role.

**FEAT-14.SPEC-004-AC-05:** Given a deliverable_ready event fires, when recipients are resolved, then Notification records are created for both Owen and Priya, since deliverable_ready entitles both roles.

**FEAT-14.SPEC-004-AC-06:** Given a proposal_change_requested event fires, when recipients are resolved, then a Notification record is created only for Nadia; Owen and Priya are never resolved as recipients for this type.

**FEAT-14.SPEC-004-AC-07:** Given Nadia (Freelancer) is the subject of a milestone_approved_confirmation event, when recipients are resolved, then she is always entitled and a Notification record is created for her alongside Owen's.

**FEAT-14.SPEC-004-AC-08:** Given Dana (Support Operator) is inside a logged support session, when she looks for a way to view a Notification's full content, then no such view exists -- she can see only the delivery-warning summary defined by FEAT-14.SPEC-006.

**FEAT-14.SPEC-004-AC-09:** Given Owen looks for a way to see his own or another Notification's delivery status as data, when he looks in the portal, then no such view exists for client contacts.

**FEAT-14.SPEC-004-AC-10:** Given every notification_type in the registry is currently classified Transactional, when Nadia opens her notification preferences (FEAT-21), then no toggle is offered for any of them.

**FEAT-14.SPEC-004-AC-11:** Given a hypothetical future notification_type were classified Optional, when Nadia turns it off in her preferences, then FEAT-14.SPEC-002's entitlement check for that type returns zero recipients for her account going forward.

**FEAT-14.SPEC-004-AC-12:** Given a Transactional notification_type, when FEAT-14.SPEC-002 checks entitlement, then the Freelancer Account's notification_preferences are never consulted, and every entitled recipient always resolves.

**FEAT-14.SPEC-004-AC-13:** Given a support_session_notice event fires (Dana opens a session), when recipients are resolved, then Nadia is always entitled, per XBR-29's mandate that support sessions are always announced by email.

**FEAT-14.SPEC-004-AC-14:** Given a chargeback_notice event fires, when recipients are resolved, then Nadia is always entitled, per XBR-21.

**FEAT-14.SPEC-004-AC-15:** Given a new contact is invited via FEAT-18, when the contact_invitation event fires, then the newly invited contact (Owen or Priya, whichever role they were assigned) is the sole entitled recipient.

**FEAT-14.SPEC-004-AC-16:** Given a client has two Primary contacts, when a notification_type entitling the Primary role fires, then each Primary contact receives their own independent Notification record.

**FEAT-14.SPEC-004-AC-17:** Given a contact's role changes from Reviewer to Primary a moment before FEAT-14.SPEC-002 runs its entitlement check, when the check runs, then the updated (Primary) role governs entitlement for that occurrence.

**FEAT-14.SPEC-004-AC-18:** Given no Reviewer contact currently exists for a client, when a notification_type entitling the Reviewer role fires, then a Notification record is created only for the currently existing entitled contact(s), with no error for the missing role.

**FEAT-14.SPEC-004-AC-19:** Given a notification_type outside the registry is somehow supplied to this spec's rules, when FEAT-14.SPEC-002 checks it, then the type is rejected as an internal defect and no Notification record is created under an unregistered type.

**FEAT-14.SPEC-004-AC-20:** Given anyone attempts to delete an individual Notification record within the product, when they look for such an action, then none exists -- Notification records are removed only by FEAT-24's account deletion.

**FEAT-14.SPEC-004-AC-21:** Given any of the 26 Notification specs in the product fires its triggering event (for example FEAT-24.SPEC-008's export-ready event, FEAT-26.SPEC-004's signature-recorded event, or FEAT-32.SPEC-006's needs-attention event), when FEAT-14.SPEC-002 looks up its notification_type, then a registry row exists naming that spec, and recipients resolve per that row (data_export_ready to Nadia only; signed_copy_confirmation to Owen and Nadia; payment_connection_needs_attention to Nadia only; refund_recorded to Owen only).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 11 | 11 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 (plus the 29-row registry covering all 26 Notification specs) | 6 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Recognizable & Branded Email Presentation Rules

## Overview

**Name:** Recognizable & Branded Email Presentation Rules
**ID:** FEAT-14.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs sender name, subject line, branding, and plain, consistent formatting so every email is identifiable as coming from the freelancer's practice and reads as trustworthy, not spam.
**Parent Feature:** FEAT-14 -- Notifications (Email)
**Governed Entity:** Branding Profile (as applied to email presentation; also draws on Freelancer Account's business_name for sender-name composition)

## Scope and Non-Goals

**In Scope:**
- The sender-name composition rule applied to every notification
- The subject-line context requirement applied to every notification
- Where and how the freelancer's Branding Profile (logo, brand colour) is applied to a notification's visual presentation, and the neutral-default fallback
- Where and how the referral mark is included on a client-facing email, without overriding branding (XBR-32)
- The plain, consistent formatting baseline every notification's content must follow

**Non-Goals:**
- Capturing or validating the Branding Profile's own fields (logo size/format limits, brand-colour legibility adjustment) -- owned by FEAT-19 (Freelancer Branding); this spec consumes an already-valid Branding Profile and defines only how it is applied to email presentation.
- The exact subject and body wording of any single notification type -- owned by each triggering feature's own Notification spec (e.g., FEAT-02.SPEC-011), which must satisfy this spec's rules but authors its own content.
- Deciding who receives which notification type -- owned by FEAT-14.SPEC-004; this spec governs how an already-entitled email looks and reads, not who it goes to.
- The referral mark's own content and destination -- owned by FEAT-33 (Portal Referral Attribution); this spec governs only that the mark is included and does not override branding.

## Governed Entity

**Entity:** Branding Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| logo | image (optional) | Freelancer's logo, within size/format limits set by FEAT-19; a neutral default applies when unset |
| brand_colour | colour (optional) | Freelancer's primary brand colour, legibility-adjusted by FEAT-19; a neutral default applies when unset |

**Referenced (not governed) for composition:** Freelancer Account -- business_name, name (sender-name fallback); Project -- project_name, Invoice -- invoice_number, Milestone -- name, Client -- client_name (subject-line context identifiers, whichever is relevant to the notification_type).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | Applies these rules during its composition step, on every notification, before handing the email to FEAT-14.SPEC-001 |
| FEAT-02.SPEC-011, FEAT-03.SPEC-006, FEAT-03.SPEC-007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004 | Each feature's own Notification spec | Each authors its own subject/body content within the boundaries this spec sets (e.g., the sender-name and branding rules), rather than re-deriving them independently |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| logo | No new validation here -- FEAT-19 already enforces size and format limits before a Branding Profile is saved; this spec only defines the fallback when the field is empty (neutral default) | When absent | At composition time (FEAT-14.SPEC-002) | -- | -- |
| brand_colour | No new validation here -- FEAT-19 already performs legibility adjustment before a Branding Profile is saved; this spec only defines the fallback when the field is empty (neutral default) | When absent | At composition time | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Sender-name composition | Freelancer Account -- business_name, name | The sender display name is business_name; if business_name is not yet set, it falls back to the freelancer's account name, so no email is ever sent with a blank sender identity | Internal invariant -- never user-facing; a Freelancer Account without at least a name cannot exist (FEAT-20 requires it at sign-up) |
| Subject-line context requirement | notification_type, and the record identifier relevant to that type (Project.project_name, Invoice.invoice_number, Milestone.name, or Client.client_name) | Every subject line names at least one specific record identifier matched to the recipient's own context: a client-facing subject (to Owen or Priya) references the project, invoice, or milestone; a freelancer-facing copy (to Nadia) references the client company name, since she manages several clients at once | Internal invariant, enforced by each triggering feature's own Notification spec review, not by a runtime check |
| Branding applies to client-facing recipients only | Branding Profile -- logo, brand_colour; recipient role | Every notification addressed to a client contact (Owen or Priya) applies the freelancer's Branding Profile to its visual presentation, falling back to a neutral default when unset (XBR-31); a notification addressed to Nadia herself does not receive this branding treatment, since branding exists to build trust with an external audience, not to decorate her own inbox | -- |
| Referral mark applies to client-facing recipients only | recipient role | Every notification addressed to a client contact includes the referral mark, without overriding branding (XBR-32); a notification addressed to Nadia herself never carries the referral mark, since it exists to reach potential new freelancers through clients and peers, not her own account | -- |
| Greeting consistency | recipient -- name | Every notification body opens with "Hi {first_name}," using the recipient's own name -- never a generic, unaddressed, or promotional-style opening | Internal invariant -- a missing recipient name is impossible, since name is required at Client Contact creation (FEAT-18) and at Freelancer Account creation (FEAT-20) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read the Branding Profile for email composition (this spec's own action) | System, via FEAT-14.SPEC-002 | Always, for every notification addressed to a client contact | -- |
| Edit the Branding Profile (logo, brand_colour) | Nadia (Freelancer) | Always, her own account -- owned and enforced by FEAT-19, referenced here only for completeness | -- |
| Edit the Branding Profile | Owen, Priya, Dana | Never | Not offered anywhere in their access; Dana's View-only account access (Access Matrix) includes no branding controls |
| Preview how a branded email will render | Nadia (Freelancer) | Full, via each triggering feature's own preview (e.g., FEAT-02.SPEC-002 Proposal Preview) applying this spec's rules -- not a screen this spec itself owns | -- |
| Preview how a branded email will render | Owen, Priya, Dana | Never, as a dedicated preview action -- they see the rendered result only by receiving the actual email (Owen, Priya) or viewing the delivery-warning summary (Dana, FEAT-14.SPEC-006) | No preview action exists for these roles; this is not a denial of an otherwise-available action, since no such standalone preview surface exists for them in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Sender display name | business_name; falls back to Freelancer Account.name if business_name is unset | On every notification send | No -- not a per-email choice; Nadia changes her business_name in FEAT-21 (Settings), which then applies to every future send |
| Header branding (logo, brand_colour) | Branding Profile.logo / brand_colour if set; otherwise a clean neutral default (owned by FEAT-19) | On every notification addressed to a client contact | No -- not a per-email choice; Nadia changes her Branding Profile in FEAT-19, which then applies to every future client-facing send |
| Referral mark presence | Always included on every client-facing notification | On every notification addressed to a client contact | No -- per XBR-32, the referral mark appears on every client-facing email on every plan in MVP; there is no opt-out |
| Subject-line context identifier | Derived from the notification_type's registry entry (FEAT-14.SPEC-004) and the triggering event's own data (project, invoice, milestone, or client reference) | On every notification send | No -- system-derived, not user-chosen |

## Business Rules

- Every notification's sender name identifies the freelancer's business, never a generic or disguised origin, giving the recipient's inbox a recognizable, consistent source across every email they receive from that freelancer (feature-overview.md, Key Capabilities: "every email names the freelancer and the project in its sender name and subject").
- Every subject line names the specific record it concerns, so a recipient managing several concurrent client relationships or freelancer engagements (a client contact who works with more than one freelancer, or Nadia managing several clients) can tell emails apart at a glance without opening each one.
- The freelancer's branding (logo, brand colour) and the referral mark apply to every client-facing email consistently, falling back to a clean neutral default when branding is unset, and the referral mark never overrides the branding (XBR-31, XBR-32).
- Formatting stays plain and consistent across every notification type -- a simple header, a personal greeting, clear body text, one primary call-to-action button, and a plain sign-off -- deliberately avoiding promotional styling, since client-facing emails landing in spam is a documented, recurring failure mode this feature exists to avoid [RESEARCH-INFORMED: spam-folder delivery of client emails is a recurring complaint for HoneyBook and SuiteDash, and message reliability is a category-wide quality bar (G2, Capterra, Trustpilot, HIGH)].
- Branding and the referral mark are deliberately withheld from notifications addressed to Nadia herself: they exist to build external trust and drive growth through clients and peers (XBR-32), not to decorate her own working inbox, so her own copies stay plain and functional.

## Edge Cases

- **The freelancer has not set a Branding Profile at all** -- Every client-facing email falls back cleanly to the neutral default logo and colour, consistent with FEAT-02.SPEC-011's own stated fallback behavior; no email is ever sent with a missing or broken branding element.
- **The freelancer's business_name is not yet set (only her personal account name exists)** -- Sender name and any subject slot that would otherwise use business_name use her personal account name instead, so no email is ever sent with a blank sender or subject slot.
- **A notification is addressed to Nadia herself (e.g., proposal_accepted_confirmation's copy to Nadia)** -- Branding and the referral mark are not applied to her copy; only the sender-name, subject-context, and greeting rules apply, since those support recognizability for her own record-keeping, not external trust-building.
- **The freelancer's brand colour is adjusted for legibility by FEAT-19 after this spec last read it** -- This spec always reads the current, already-legibility-adjusted value at composition time; it never caches or re-derives contrast itself, so a legibility fix Nadia makes in FEAT-19 takes effect on the very next notification sent.
- **A single triggering event addresses more than one role in the same occurrence (e.g., invoice_issued to both Owen and Nadia)** -- Each recipient's copy is composed independently against this spec's rules for their own audience type: Owen's copy is client-facing (branded, with the referral mark); Nadia's copy is not (unbranded, no referral mark), even though both originate from the same triggering event.

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given the freelancer has set both a logo and a brand colour, when a client-facing notification is composed, then the email's presentation applies that logo and colour.

**FEAT-14.SPEC-005-AC-02:** Given the freelancer has not set a Branding Profile, when a client-facing notification is composed, then the email falls back to the neutral default logo and colour.

**FEAT-14.SPEC-005-AC-03:** Given the freelancer's brand colour was recently adjusted for legibility by FEAT-19, when the next notification is composed, then it uses the current, already-adjusted colour value.

**FEAT-14.SPEC-005-AC-04:** Given the freelancer has set a business_name, when any notification is composed, then the sender display name is that business_name.

**FEAT-14.SPEC-005-AC-05:** Given the freelancer has not yet set a business_name, when any notification is composed, then the sender display name falls back to her personal account name.

**FEAT-14.SPEC-005-AC-06:** Given a notification is addressed to Owen about a specific project, when its subject line is composed, then it names the project (or the specific invoice/milestone identifier relevant to the event).

**FEAT-14.SPEC-005-AC-07:** Given a notification is addressed to Nadia about a specific client's action, when its subject line is composed, then it names the client company, since Nadia manages several clients.

**FEAT-14.SPEC-005-AC-08:** Given any notification body is composed, when it opens, then it greets the recipient by name ("Hi {first_name},"), never a generic or unaddressed opening.

**FEAT-14.SPEC-005-AC-09:** Given a notification is addressed to Owen or Priya, when it is composed, then it includes the referral mark without overriding the freelancer's branding.

**FEAT-14.SPEC-005-AC-10:** Given a notification is addressed to Nadia herself, when it is composed, then it does not carry the referral mark and does not apply client-facing branding.

**FEAT-14.SPEC-005-AC-11:** Given any notification type, when its content is composed, then it follows the plain, consistent formatting baseline (simple header, personal greeting, clear body, one primary CTA, plain sign-off) rather than promotional styling.

**FEAT-14.SPEC-005-AC-12:** Given Nadia edits her Branding Profile in FEAT-19, when she saves the change, then it is not offered as a per-email override anywhere -- it applies uniformly to every future client-facing send.

**FEAT-14.SPEC-005-AC-13:** Given Nadia looks for a way to turn off the referral mark on client-facing emails, when she checks her settings, then no such control exists, per XBR-32.

**FEAT-14.SPEC-005-AC-14:** Given Dana is inside a logged support session, when she looks for a way to preview a client-facing email's rendered branding, then no such preview action exists for her role.

**FEAT-14.SPEC-005-AC-15:** Given Owen or Priya attempt to edit the freelancer's Branding Profile, when they look for such a control, then none exists anywhere in their portal access.

**FEAT-14.SPEC-005-AC-16:** Given an invoice_issued event addresses both Owen and Nadia in the same occurrence, when each copy is composed, then Owen's copy is branded with the referral mark and Nadia's copy is not.

**FEAT-14.SPEC-005-AC-17:** Given a client has no Reviewer contact on record, when a notification entitling the Reviewer role would otherwise be composed for one, then no presentation rule is violated -- the rule set applies per actual recipient, not per role slot.

**FEAT-14.SPEC-005-AC-18:** Given the freelancer's logo file was accepted by FEAT-19's own size and format validation, when this spec composes an email, then it applies that logo without re-validating its size or format.

**FEAT-14.SPEC-005-AC-19:** Given a Freelancer Account exists (required at sign-up, FEAT-20), when any notification is composed, then a sender name and a recipient greeting name are always available -- neither is ever blank.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Notification Spec: Delivery Failure Warning to Freelancer

## Overview

**Name:** Delivery Failure Warning to Freelancer
**ID:** FEAT-14.SPEC-006
**Type:** Notification
**Purpose:** Warns Nadia on the affected project when a notification's delivery fails after retries are exhausted (or bounces permanently), so a lost email is surfaced rather than silently vanishing; viewable read-only by Dana inside a support session.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- The warning shown to Nadia when a Notification's delivery fails permanently (bounce) or its retries are exhausted (FEAT-14.SPEC-003)
- The read-only variant Dana sees inside a logged support session (FEAT-31)
- Batching, deduplication, and persistence (expiry) behavior for this warning

**Non-Goals:**
- Deciding when a notification has permanently failed or exhausted its retries -- owned by FEAT-14.SPEC-003; this spec begins where that spec's failure outcome hands off.
- Correcting the underlying cause (e.g., a mistyped contact email address) -- owned by FEAT-18 (Client Contact Management & Roles); this spec's only involvement is deep-linking Nadia there.
- Automatically re-sending the originally failed notification once the underlying issue is fixed -- outside this feature's dispatch/tracking scope; a fresh send happens only if the originating feature offers its own resend action (e.g., FEAT-02.SPEC-007, Proposal Resend), which this spec does not itself trigger.
- A full in-app notification feed or history of past warnings -- excluded per scope-boundaries.md and the Brief's own States field ("no primary browsing view of its own"); In-App Notification Center (FEAT-29) is explicitly deferred to Later.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app (a persistent element on the affected project's own screen, owned by FEAT-01) | Always, whenever a Notification tied to that project has a Bounced or retries-exhausted Failed outcome | This is a persistent state indicator on a screen Nadia already owns and visits, not a discrete push-style interruption or a notification inbox -- it is categorically distinct from the in-app notification *feed* concept the Brief's Non-Goals excludes (FEAT-29, deferred to Later); "surfaced... within minutes" (ASMP-26) means the warning becomes available promptly, not that it is actively pushed to her outside the product she already uses to manage the affected project. |

Email is deliberately not used for this warning: the failure being reported is itself about an email that could not be delivered, and Nadia's own inbox is not where the product routes account-operational state (feature-overview.md, States: "no primary browsing view of its own"); the project screen she already checks for status is the surviving signal, consistent with FEAT-11.SPEC-004 and other sibling specs treating the delivery warning as project-level state, not a separate message.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Notification permanently bounced | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Fires immediately when a send is reported Bounced -- no retry is attempted for a permanent failure | Notification reference, notification_type, recipient, affected project reference |
| Notification retries exhausted | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Fires when a transient Failed outcome recurs until the retry count reaches platform parameter: `transactional-email-retry-count` with no Delivered outcome | Notification reference, notification_type, recipient, affected project reference |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- Full access, per the Access Matrix's Notifications & Help row; she is the sole active recipient of this warning, since it exists to let her act on a communication failure on her own account. Dana (Support Operator) has "View (delivery warnings only)" per the Access Matrix -- she sees the same warning read-only, and only inside a logged support session (FEAT-31), never the underlying message's subject or body content. Owen and Priya never see this warning; it concerns the freelancer's operational awareness of her own delivery problem, not client-facing content.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional warning | -- | Always on | -- |

This warning cannot be turned off: per XBR-30, "delivery failures are surfaced to the freelancer as warnings on the affected project," and a preference to silence the mechanism that reports a lost communication would defeat the purpose it exists for.

**Quiet Hours:** N/A -- this warning is a persistent state indicator rendered when Nadia opens the affected project's screen, not a discrete, time-stamped interruption; there is no moment of delivery for quiet hours to govern, and the product defines no quiet-hours window for this feature.

## Content Definition

**In-app (single failure):**
- **Title:** Delivery issue on this project
- **Body:** The {notification_type_label} email to {recipient_name} could not be delivered{failure_reason_clause}.
- **CTA:** Review contact details -- deep-links to FEAT-18 (Client Contact Management & Roles), the client contact list for the affected client, so Nadia can correct a mistyped or outdated address

**In-app (batched -- 2+ open failures on the same project, see Delivery Rules):**
- **Title:** {count} delivery issues on this project
- **Body:** A list with one line per failure: {notification_type_label} to {recipient_name}{failure_reason_clause}
- **CTA:** Review contact details -- deep-links to FEAT-18, the client contact list for the affected client

**In-app (Dana's read-only support-session variant):**
- **Title:** Delivery issue on this project (support view)
- **Body:** The {notification_type_label} email to {recipient_name} could not be delivered{failure_reason_clause}. This summary is read-only; the message's own content is not shown.
- **CTA:** None -- Dana's session is read-only (Access Matrix: Support Access "Full (opens read-only sessions, always logged)," Notifications & Help "View (delivery warnings only)"); no action control is offered to her

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {notification_type_label} | Notification -- notification_type, rendered in plain language (per the registry in FEAT-14.SPEC-004) | "invoice" (for invoice_issued) | Never empty -- notification_type is required on every Notification record (FEAT-14.SPEC-004) |
| {recipient_name} | Client Contact -- name (or Freelancer Account -- name, for a freelancer-addressed notification that itself failed) | Owen Marsh | Never empty -- name is required at Client Contact creation (FEAT-18) and Freelancer Account creation (FEAT-20) |
| {failure_reason_clause} | Derived from the Notification's bounce/failure reason category (FEAT-14.SPEC-001), when the capability provided one | " (the address could not be found)" | Renders as an empty string, so the sentence reads "...could not be delivered." with no invented reason, per FEAT-14.SPEC-001's edge case for a reason-less bounce |
| {count} | Derived -- number of open (unresolved) delivery-failure warnings currently on this project | 3 | Never empty -- the batched variant only renders with 2 or more open failures |

## Delivery Rules

**Batching:** All open delivery-failure warnings for the same project are shown as one persistent indicator, using the batched variant when 2 or more failures are currently open (unresolved) on that project; a new failure on a project with no existing open warning renders the single-failure variant.
**Deduplication:** At most one open warning entry exists per failed Notification; FEAT-14.SPEC-003 never re-fires this notification a second time for the same Notification once its warning entry exists, even if a duplicate outcome report arrives.
**Retry on failure:** N/A -- this is a persistent in-app state, not a transmitted message; there is no send of this warning itself to fail or retry. It either renders correctly when Nadia (or Dana, in a support session) opens the affected project's screen, or the screen's own general error handling applies (owned by FEAT-01), unrelated to this spec's delivery rules.
**Expiry:** This warning does not expire on a timer -- a lost communication remains a real, unresolved problem until addressed. It persists until the underlying Notification's condition changes (e.g., the originating feature's own resend succeeds and produces a new, separately-tracked Notification that later reports Delivered) or Nadia's corrective action otherwise resolves it; there is no automatic dismissal.

## Edge Cases

- **The underlying record is voided or superseded before Nadia acts on the warning (e.g., a proposal is edited and re-sent, per FEAT-02.SPEC-011)** -- The original warning remains as historical context (the original send genuinely did fail), but is not treated as still-actionable once the superseding event's own notification completes successfully; both her project screen and this spec's content reflect the current, superseding notification's outcome once known.
- **Multiple delivery failures accumulate on the same project across different notification types (e.g., a bounced invitation and a bounced invoice email)** -- They batch into the single project-level indicator (Delivery Rules above); each failure remains individually visible within the expanded view, and Nadia can address each independently.
- **Nadia corrects the contact's email address via FEAT-18, but the originally failed notification is never automatically retried** -- The warning remains open until she takes the originating feature's own action to resend (where one exists, e.g., FEAT-02.SPEC-007); this spec does not itself re-trigger the business event, since doing so is outside its dispatch and tracking scope.
- **The affected project is archived while a warning is open** -- The warning still renders on the archived project's own view when Nadia or Dana opens it, consistent with FEAT-14.SPEC-003's own handling: archiving removes a project from active view without erasing its record.
- **The freelancer's account is deleted (FEAT-24) while a warning is open** -- The warning is removed along with the Notification record it describes, as part of account deletion; no orphaned warning persists.
- **Dana opens a support session on a project with an open warning** -- She sees the read-only support-session variant with no action control; she cannot dismiss, resolve, or act on the warning in any way, consistent with her View-only, read-only support access (Access Matrix, FEAT-31).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Triggered by (inbound) | A Bounced outcome or exhausted retries fires this notification |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Supplies the bounce/failure reason category, when available, rendered in {failure_reason_clause} |
| FEAT-14.SPEC-004 (Notification Type & Recipient Entitlement Rules) | References (inbound) | Supplies the notification_type registry this spec's {notification_type_label} placeholder renders in plain language |
| FEAT-01 (Client & Project Management) | References (inbound) | Owns the affected project's own screen, on which this warning is a small persistent element |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | The CTA deep-links here so Nadia can correct a mistyped or outdated contact email address |
| FEAT-31 (Operator Support Access) | References (inbound) | Governs Dana's read-only, logged access to this warning's support-session variant |

## Analytics and Success Signals

- **delivery_warning_created** (notification_type, reason: bounced / retries_exhausted) -- supports success-metrics.md: "Notification Delivery Reliability"
- **delivery_warning_viewed** (viewer_role: freelancer / support_operator) -- supports success-metrics.md: "Notification Delivery Reliability"
- **delivery_warning_cta_tapped** (destination: FEAT-18 client contact list) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given an invoice_issued notification to Owen bounces permanently, when FEAT-14.SPEC-003 reports the Bounced outcome, then this warning appears on the affected project for Nadia immediately, without waiting for any retry.

**FEAT-14.SPEC-006-AC-02:** Given a proposal_sent notification to Owen fails transiently and exhausts its retries, when the final retry's outcome is processed, then this warning appears on the affected project for Nadia.

**FEAT-14.SPEC-006-AC-03:** Given Nadia opens the affected project's screen, when one open delivery failure exists, then she sees the single-failure variant naming the notification type, the recipient, and, if available, the reason.

**FEAT-14.SPEC-006-AC-04:** Given a project has two open delivery failures of different notification types, when Nadia opens the project's screen, then she sees the batched variant listing both.

**FEAT-14.SPEC-006-AC-05:** Given Nadia taps "Review contact details" on this warning, then she is taken to FEAT-18's client contact list for the affected client.

**FEAT-14.SPEC-006-AC-06:** Given Nadia has no preference control for this warning, when a delivery failure occurs, then the warning always appears -- there is no way to switch it off, per XBR-30.

**FEAT-14.SPEC-006-AC-07:** Given a bounce report carries no specific reason from the delivery capability, when this warning renders, then it states delivery failed without inventing a reason.

**FEAT-14.SPEC-006-AC-08:** Given the same Notification's exhausted-retries outcome is reported twice due to a duplicate event, when this notification is triggered, then only one open warning entry exists for that Notification, not two.

**FEAT-14.SPEC-006-AC-09:** Given Nadia corrects the bouncing contact's email address via FEAT-18, when she returns to the affected project, then the original warning remains open until she separately resends the failed communication through its originating feature's own resend action.

**FEAT-14.SPEC-006-AC-10:** Given the affected project has since been archived, when Nadia opens the archived project, then the open delivery warning still appears on its screen.

**FEAT-14.SPEC-006-AC-11:** Given Nadia's account is deleted, when the deletion completes, then any open delivery warnings tied to her account are removed along with the Notification records they describe.

**FEAT-14.SPEC-006-AC-12:** Given Dana opens a support session on an account with an open delivery warning, when she views the affected project, then she sees the read-only support-session variant with no action control, and never the underlying message's subject or body content.

**FEAT-14.SPEC-006-AC-13:** Given Owen or Priya are viewing their own portal, when a delivery failure occurs on their project, then they never see this warning in any form -- it exists only for Nadia (and Dana, read-only).

**FEAT-14.SPEC-006-AC-14:** Given a proposal is edited and re-sent after its original send bounced (FEAT-02.SPEC-011), when the edited version's own notification succeeds, then the original warning remains as historical context while the current state reflects the successful re-send.

**FEAT-14.SPEC-006-AC-15:** Given this warning is an in-app persistent element rather than a transmitted message, when the affected project's screen loads normally, then the warning renders correctly with no separate delivery/retry mechanics of its own.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (bounced, retries exhausted) | 2 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry) | 4 |
| Edge Cases | 6 | 6 |

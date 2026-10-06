---
document_type: feature-overview
feature_number: FEAT-14
feature_name: Notifications (Email)
feature_slug: notifications-email
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 0
automation_count: 2
logic_rule_count: 2
integration_count: 1
notification_count: 1
---

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

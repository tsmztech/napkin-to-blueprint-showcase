---
document_type: feature-overview
feature_number: FEAT-06
feature_name: Client Booking Identity
feature_slug: client-booking-identity
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 4
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 1
---

# Feature Breakdown Brief: Client Booking Identity

## Summary

**Feature:** Client Booking Identity
**ID:** FEAT-06
**Description:** The lightweight, password-free way a client proves it's them when they come back to view, reschedule, or cancel a booking -- a phone number plus a one-tap link sent to that phone, rather than any signup wall or password.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Open Questions names this exactly ("A phone number plus a magic link, or something else?") and its Target Users & Roles states clients "must not face a signup wall or need a password-style account." Phone-plus-link is the lightest workable mechanism that still lets a client manage a specific booking without exposing any other client's or pro's data. MVP: without it, a client has no way to self-serve a reschedule or cancellation, which the brief requires (FEAT-10).

**Key Capabilities:**
- Request a one-tap access link sent by text to the phone number used at booking
- View only this pro's bookings tied to that phone number, past and upcoming
- Access expires and must be re-requested after a short period for security
- Open a specific booking directly from the manage link inside its confirmation or reminder, without requesting a new link
- Update their own texting consent and email address for this Pro

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-06.SPEC-001 | Access Link Request | Screen | Client | Client enters their phone number and requests a one-tap access link to view their bookings with this Pro |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Automation | Client | System validates a tapped access link (on-demand or booking-specific), marks it Used or Expired, and routes the client to the right screen |
| FEAT-06.SPEC-003 | My Bookings List | Screen | Client | Client views their own past and upcoming bookings with this one Pro after redeeming an on-demand link |
| FEAT-06.SPEC-004 | Booking Detail via Manage Link | Screen | Client | Client views one booking, opened either from the My Bookings list or directly via a booking-specific manage link |
| FEAT-06.SPEC-005 | Consent & Email Preferences | Screen | Client | Client updates their own texting consent (including re-granting it) and email address for this Pro |
| FEAT-06.SPEC-006 | Access Link Delivery | Notification | Client | Sends the requested one-tap access link by text (with active consent) or email, with retry on delivery failure |
| FEAT-06.SPEC-007 | Access Link Lifecycle & Scope Rules | Logic/Rule | Client | Governs link expiry (30-minute single-use vs. booking-specific until the appointment passes), single-use enforcement, and per-phone-number request rate limiting |
| FEAT-06.SPEC-008 | Client Identity & Privacy Isolation Rule | Logic/Rule | Client | Governs phone-to-Client matching scoped to one Pro and the hard boundary that no client can ever see another phone number's or Pro's bookings |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Request a one-tap access link sent by text to the phone used at booking | FEAT-06.SPEC-001 | Primary purpose of the request screen; issuance obeys SPEC-007's expiry/rate-limit rules and SPEC-008's phone matching | Phase 2 (Explicit) |
| View only this pro's bookings tied to that phone number, past and upcoming | FEAT-06.SPEC-003 | Primary purpose of the My Bookings List screen, scoped by SPEC-008's isolation rule | Phase 2 (Explicit) |
| Access expires and must be re-requested after a short period for security | FEAT-06.SPEC-002, FEAT-06.SPEC-007 | Redemption checks the link's expiry/single-use state defined by the lifecycle rule and prompts re-request when it fails | Phase 2 (Explicit) |
| Open a specific booking directly from the manage link, without requesting a new link | FEAT-06.SPEC-002, FEAT-06.SPEC-004 | Validation routes a valid booking-specific link straight into the Booking Detail screen with no request step | Phase 2 (Explicit) |
| Update their own texting consent and email address for this Pro | FEAT-06.SPEC-005 | Primary purpose of the Consent & Email Preferences screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-06.SPEC-006 | Access Link Delivery | Phase 4 (Notification surfacing) | The Communications field names a message sent by text or email with real delivery rules (channel choice, retry on failure) -- this is not a bare success toast, so it needed its own Notification spec |
| FEAT-06.SPEC-008 | Client Identity & Privacy Isolation Rule | Phase 3 (Entity-Lifecycle) + Phase 5 (Rule Discovery) | The Access field states a hard privacy boundary ("a client can never view another phone number's bookings") that governs Create/Read on Access Link and Read on Client/Booking across three separate specs -- a rule shared across multiple specs crosses the standalone threshold |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Phase 4 (Trigger-Response) | Tapping any link (on-demand or booking-specific) triggers cross-entity processing -- token lookup, expiry/single-use check, state write, routing decision -- exceeding a simple inline data write |

## Entity-Lifecycle Coverage Matrix

**Entity: Access Link**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-06.SPEC-001 | On-demand link generated on request, scoped to all of this client's bookings with this Pro | Booking-specific links are created by FEAT-08 (confirmations/reminders) and by FEAT-30 (fresh link after a Pro reschedule) -- cross-feature |
| Read (single) | FEAT-06.SPEC-002 | Validation looks up the tapped link's token, scope, and current state | -- |
| Read (list) | N/A | No screen lists a client's issued links; a link is only ever looked up by its own token | -- |
| Update | FEAT-06.SPEC-002 | State written to Used (first successful redemption) or left to expire | -- |
| Delete/Archive | N/A | Links are never explicitly deleted -- they transition to Expired on timeout (soft state only, no hard delete); no purge policy is stated in Stage 2, so this is recorded as an explicit non-goal below rather than a silent gap | -- |
| State Transition | FEAT-06.SPEC-002 | Issued -> Used (first tap wins) or Issued -> Expired (30 minutes elapse for on-demand links, or the appointment passes for booking-specific links); governed by FEAT-06.SPEC-007 | A second device opening an already-used single-use link is shown the "request a new link" prompt (SPEC-002) |

**Entity: Client (partial -- this feature reads and updates one field; it does not create or delete Client records)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Client records are created by FEAT-05 (first booking) and FEAT-30 (Pro books a client in) -- out of this feature's scope | -- |
| Read (single) | FEAT-06.SPEC-001, FEAT-06.SPEC-002 | Phone number entered or carried in a link is matched to exactly one Client record for this Pro, governed by FEAT-06.SPEC-008 | -- |
| Read (list) | N/A | This feature never browses multiple Client records -- it matches to exactly one record per phone number per Pro; browsing belongs to Client List Search & Filter (FEAT-12) | -- |
| Update | FEAT-06.SPEC-005 | Client updates their own email address for this Pro; the Message channel element hosted on SPEC-005 also sets the client's preferred_message_channel (SMS | WhatsApp, default SMS), written by FEAT-26.SPEC-001 and kept separate from Messaging Consent | Contact details and private notes remain Pro-editable only (FEAT-13); this feature never touches those fields |
| Delete/Archive | N/A | Client deletion is owned by FEAT-13 (hard delete of contact details on request, de-identified financial history retained); this feature has no delete capability | -- |
| State Transition | N/A | The Client entity carries no lifecycle states this feature participates in | -- |

**Entity: Messaging Consent (partial -- this feature re-grants consent; revocation and consent-state ownership belong to FEAT-14)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Captured at booking by FEAT-05 -- out of this feature's scope | -- |
| Read (single) | FEAT-06.SPEC-005 | Displays the client's current consent state before offering to re-grant it | -- |
| Read (list) | N/A | One consent record per channel per client is read directly; no listing view exists here | -- |
| Update | FEAT-06.SPEC-005 (hosting the FEAT-14.SPEC-001 consent section) | Client re-grants consent for this Pro from the consent section that FEAT-14.SPEC-001 defines and FEAT-06.SPEC-005 hosts; the most-recent-explicit-action rule (owned by FEAT-14) decides precedence against a concurrent STOP reply | Revocation itself (STOP / opt-out link) is FEAT-14's spec, not this feature's -- cross-feature |
| Delete/Archive | N/A | Deleted only as part of client deletion, owned by FEAT-13 | -- |
| State Transition | FEAT-06.SPEC-005 | Revoked -> Re-granted, performed by the client from this feature's Consent & Email Preferences screen | Granted -> Revoked is FEAT-14's transition |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-06.SPEC-002, FEAT-06.SPEC-003, FEAT-06.SPEC-004 | SPEC-002 reads the referenced booking of a booking-specific link (to validate scope and detect that the appointment has passed) and to route to it; SPEC-003 lists and SPEC-004 displays only the bookings belonging to the matched Client with this Pro |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client submits a phone number on the request screen | System checks the per-phone-number hourly request rate limit | Standalone Logic/Rule | FEAT-06.SPEC-007 |
| Client's request passes the rate-limit check | System matches the phone number to a Client record scoped to this Pro | Standalone Logic/Rule | FEAT-06.SPEC-008 |
| Phone number matches a Client record | System generates a single-use, 30-minute on-demand Access Link | Inline in triggering screen | FEAT-06.SPEC-001 |
| An Access Link is generated | Link is delivered by text (if active texting consent) or email otherwise | Standalone Notification | FEAT-06.SPEC-006 |
| Phone number has no matching Client record for this Pro | Show a plain "no bookings found" message with no hint the number exists elsewhere | Inline in triggering screen | FEAT-06.SPEC-001 |
| Message delivery (text or email) fails | Offer an immediate retry | Standalone Notification (failure handling) | FEAT-06.SPEC-006 |
| Client taps any access link (on-demand or booking-specific) | Validate the token's scope, expiry, and used/unused state; write the resulting state | Standalone Automation | FEAT-06.SPEC-002 |
| Validated link is on-demand and unused | Route the client to their bookings list | Standalone Automation | FEAT-06.SPEC-002 |
| Validated link is booking-specific and unused | Route the client straight into that one booking, with no code or password | Standalone Automation | FEAT-06.SPEC-002 |
| Link is expired, already used, or invalid | Show a single-tap "request a new link" prompt, no separate reset flow | Inline in triggering screen | FEAT-06.SPEC-001 |
| A single-use link is opened on a second device after first use | First use wins; the second device sees the "request a new link" prompt | Standalone Automation | FEAT-06.SPEC-002 |
| Client taps a booking-specific link after the appointment has passed | Link no longer works; client sees the same "request a new link" pattern | Standalone Automation | FEAT-06.SPEC-002 |
| Client re-grants texting consent or updates their email | Write the updated value; consent precedence and next-message effect are governed by FEAT-14's rule (XBR-15) | Inline in triggering screen (cross-feature rule reference) | FEAT-06.SPEC-005 |
| Pro changes a client's phone number (FEAT-13) | This client's existing access links are invalidated and fresh texting consent is required before the next message | Cross-feature -- logged in touchpoints | FEAT-13 responsibility (XBR-18) |
| Pro reschedules a booking (FEAT-30) | A fresh booking-specific manage link is issued, replacing the prior one | Cross-feature -- logged in touchpoints | FEAT-30 / FEAT-08 responsibility |

## Shared Context

**Shared Entities:**
- Access Link -- created by SPEC-001 (on-demand) and, cross-feature, by FEAT-08 and FEAT-30 (booking-specific); read and state-updated by SPEC-002; lifecycle governed by SPEC-007. Fields: scope (all bookings with one Pro, or one booking), expiry, state (Issued | Used | Expired).
- Client -- matched by phone in SPEC-001, SPEC-002, and SPEC-008; email field updated by SPEC-005. Field in scope here: phone (identity key, read-only to this feature), email (this feature's one writable field).
- Messaging Consent -- read and re-granted by SPEC-005; consent state and revocation owned by FEAT-14. Fields: channel, state, timestamp, phone_number.
- Booking -- read-only by SPEC-002 (booking-specific link validation and routing), SPEC-003 (list) and SPEC-004 (detail), always scoped to the Client matched by SPEC-008.

**Shared UI Patterns:**
- "No hint" empty/error pattern -- SPEC-001 (no bookings found for a phone number) and SPEC-002 (invalid, expired, or already-used link) both show a plain, non-alarming message that never reveals whether a phone number or link exists elsewhere. Spec Writers for both should keep this wording and tone consistent, per the Access field's hard privacy boundary.
- Single-tap re-request -- SPEC-001 and SPEC-002 both surface the identical "request a new link" action with no separate "reset" flow to learn, per Primary Flows & Alternates.

**Shared Validation:**
- FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) is referenced, not duplicated, by SPEC-001 (rate limit and expiry at issuance) and SPEC-002 (expiry, single-use, and scope check at redemption).
- FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) is referenced, not duplicated, by SPEC-001, SPEC-002, and SPEC-005 for phone-to-Client matching and the cross-client/cross-Pro isolation boundary.

## Internal Dependency Map

```
SPEC-001 (Access Link Request) -> [client submits phone number] -> SPEC-008 (Client Identity & Privacy Isolation Rule)
SPEC-001 (Access Link Request) -> [rate-limit check] -> SPEC-007 (Access Link Lifecycle & Scope Rules)
SPEC-001 (Access Link Request) -> [link generated] -> SPEC-006 (Access Link Delivery)
SPEC-006 (Access Link Delivery) -> [client taps the delivered link] -> SPEC-002 (Access Link Validation & Redemption)
SPEC-002 (Access Link Validation & Redemption) -> [checks expiry/single-use/scope against] -> SPEC-007 (Access Link Lifecycle & Scope Rules)
SPEC-002 (Access Link Validation & Redemption) -> [on-demand link, valid] -> SPEC-003 (My Bookings List)
SPEC-002 (Access Link Validation & Redemption) -> [booking-specific link, valid] -> SPEC-004 (Booking Detail via Manage Link)
SPEC-002 (Access Link Validation & Redemption) -> [invalid, expired, or already used] -> SPEC-001 (Access Link Request) [re-request prompt]
SPEC-003 (My Bookings List) -> [client selects a booking] -> SPEC-004 (Booking Detail via Manage Link)
SPEC-003 (My Bookings List) -> [client opens preferences] -> SPEC-005 (Consent & Email Preferences)
SPEC-004 (Booking Detail via Manage Link) -> [client opens preferences] -> SPEC-005 (Consent & Email Preferences)
SPEC-005 (Consent & Email Preferences) -> [hosts consent section] -> FEAT-14.SPEC-001; [Message channel element] -> FEAT-26.SPEC-001
SPEC-005 (Consent & Email Preferences) -> [validates the matched Client record via] -> SPEC-008 (Client Identity & Privacy Isolation Rule)
```

**Default Entry:** SPEC-001 (Access Link Request) -- the screen shown when a client navigates to manage their booking without already holding a valid link.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-06.SPEC-001 | Related (no navigation) | FEAT-05 (Public Booking Page & Booking Flow) | FEAT-05.SPEC-003 performs the returning-client lookup in place and does not navigate to FEAT-06.SPEC-001; that in-place lookup applies the same identity-match rule as FEAT-06.SPEC-008. SPEC-001 has no entry point from FEAT-05.SPEC-003 | Client enters a known phone number on the booking page |
| FEAT-06.SPEC-002 | Inbound | FEAT-08 (Automated Booking Messaging) | Client taps the manage link carried inside a confirmation or reminder message | Client taps manage link |
| FEAT-06.SPEC-002 | Inbound | FEAT-30 (Pro Booking Management) | A fresh booking-specific manage link, issued after a Pro reschedules, replaces the prior one | Pro reschedules a booking |
| FEAT-06.SPEC-004 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | Client moves from viewing a booking to cancelling or rescheduling it | Client chooses cancel or reschedule |
| FEAT-06.SPEC-005 | Outbound | FEAT-14 (Messaging Consent Management) | SPEC-005 is the single client-facing Preferences screen and hosts FEAT-14.SPEC-001 as its consent section; consent state and revocation rules remain FEAT-14's (XBR-15) | Client opens preferences (from SPEC-003 or SPEC-004) |
| FEAT-06.SPEC-005 | Outbound | FEAT-26 (WhatsApp Reminders) | SPEC-005 carries a Message channel element (SMS / WhatsApp) linking to FEAT-26.SPEC-001, which writes the client's preferred_message_channel | Client taps the Message channel element |
| FEAT-06.SPEC-003 | Outbound | FEAT-21 (Recurring Appointments) | The My Bookings List links to the recurring series section defined by FEAT-21.SPEC-002 | Client opens their recurring series |
| FEAT-06.SPEC-003 | Outbound | FEAT-20 (Waitlist for Cancelled Slots -- v1) | Client leaves a waitlist entry from their bookings list | Client taps leave |
| FEAT-06.SPEC-004 | Outbound | FEAT-22 (In-App Balance Payment -- v1) | Client pays the remaining balance from a booking | Client taps pay balance |
| FEAT-06.SPEC-006 | Outbound | FEAT-08 (Automated Booking Messaging) | Access-link delivery rides the transactional text-messaging and transactional-email capabilities (ASMP-32) that FEAT-08's Integration spec owns for the product; this feature supplies the message content and delivery rule, not the capability contract | Access link generated |
| FEAT-06.SPEC-008 | Inbound | FEAT-13 (Client Record Management) | A Pro-initiated phone number change invalidates this client's existing access links and requires fresh texting consent (XBR-18) | Pro edits a client's phone number |

## Non-Functional Notes

**Data volumes / growth:** Access Link volume tracks booking volume rather than accumulating independently -- a few hundred Pros in year one, each with roughly 100-500 clients and 20-40 bookings a week (scope-boundaries SC-19), and each on-demand link's 30-minute or single-appointment life keeps active link volume bounded regardless of total booking history.

**Responsiveness:** This is an online-only identity check by design (correctness over convenience, per the feature's States field); link generation and delivery must feel immediate, with an in-place loading indicator rather than a blank screen while the link is generated and sent, and a plain message whenever connectivity is required and missing (ASMP-27).

**Data sensitivity / privacy:** The Access Link is a bearer credential -- short-lived and scoped to exactly one client with one Pro (ASMP-30) -- and must never be guessable or reusable beyond its stated scope. The client's phone, email, and booking list surfaced through this feature are personal data visible only to that Pro and to the client themselves (ASMP-23); a client can never see another phone number's bookings under any circumstance, a hard privacy boundary carried directly from BRIEF.md's Constraints.

**Compliance flags:** ASMP-24 (US SMS-consent rules) governs the texting channel used to deliver access links and the consent re-grant this feature performs; access-link and email delivery both rely on the transactional text-messaging and transactional-email category-level capabilities named in ASMP-32, whose Integration contract is specified by FEAT-08 rather than duplicated here.

## Non-Goals

- **Password or account-based client identity** -- Excluded per BRIEF.md's Target Users & Roles: clients "must not face a signup wall or need a password-style account." Phone-plus-link is the sole client identity mechanism; no password, PIN, or account-creation flow is in scope.
- **Cross-Pro client identity or a shared client profile** -- Excluded per scope-boundaries SC-04 (AUDIT-EXCLUDED 4): a phone number's Client record is scoped to exactly one Pro; booking with a second Pro always creates a separate, unconnected record rather than a merged or shared identity.
- **Support bypass of a client's access link** -- Excluded per the Access Matrix and scope-boundaries SC-05: Platform Operator (Support) has no access through this mechanism; support never uses or bypasses a client's access link, even for troubleshooting a help request.
- **Automatic purge of expired or used Access Links** -- Stage 2 states only that links "expire automatically," with no purge window specified. The Feature Analyst records this as an explicit non-goal rather than a silent gap: links are retained as an audit trail of issuance and use with no automatic deletion, consistent with SC-22's product-wide principle that operational history is retained for the life of the account; a definite retention/purge policy is left open for a later Stage 4 decision.
- **Promotional or marketing use of the access-link channel** -- Excluded per scope-boundaries SC-15: clients consent to booking-related texts only, and this feature's messages (and every other Chairtime message) are limited to confirmations, reminders, change notices, and access links -- never marketing content.

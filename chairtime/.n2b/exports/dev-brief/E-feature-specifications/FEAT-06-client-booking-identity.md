# FEAT-06 — Client Booking Identity

This chapter covers Client Booking Identity (FEAT-06), a Core-tier feature. It carries 8 specifications carrying 97 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-06.SPEC-001 | Access Link Request | screen | 12 |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | automation | 11 |
| FEAT-06.SPEC-003 | My Bookings List | screen | 12 |
| FEAT-06.SPEC-004 | Booking Detail via Manage Link | screen | 11 |
| FEAT-06.SPEC-005 | Consent & Email Preferences | screen | 16 |
| FEAT-06.SPEC-006 | Access Link Delivery | notification | 10 |
| FEAT-06.SPEC-007 | Access Link Lifecycle & Scope Rules | logic-rule | 13 |
| FEAT-06.SPEC-008 | Client Identity & Privacy Isolation Rule | logic-rule | 12 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Access Link Request

## Overview

**Name:** Access Link Request
**ID:** FEAT-06.SPEC-001
**Type:** Screen
**Purpose:** Client enters the phone number they used at booking and requests a one-tap access link sent to that phone, so they can view or manage their bookings with this one Pro without a password.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Phone number entry and submission to request an on-demand access link
- The "no bookings found" outcome for a phone number with no matching Client record for this Pro
- The single-tap "request a new link" re-entry point used after an invalid, expired, or already-used link (FEAT-06.SPEC-002)
- Surfacing a link-delivery failure with an immediate retry

**Non-Goals:**
- Validating and redeeming a tapped link -- owned by FEAT-06.SPEC-002 (Access Link Validation & Redemption); this screen only issues the request
- Viewing the client's bookings -- owned by FEAT-06.SPEC-003 (My Bookings List), reached only after a valid link is redeemed
- Any password, PIN, or account-creation flow -- excluded per BRIEF.md's Target Users & Roles: clients "must not face a signup wall or need a password-style account"
- Pro sign-in -- the Pro has their own dashboard login (FEAT-29), entirely out of this feature's scope per product-features.md's Access field

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry | Client navigates directly to manage their booking without already holding a valid link (e.g., typed the manage-my-booking address, or opened it from memory) | None -- form starts empty |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | The tapped link was invalid, expired, or already used | A single-tap "request a new link" prompt is shown in place of the empty form's first state |
| FEAT-20.SPEC-001 (Join Waitlist) | Client taps the access-link link on the waitlist Confirmed state | None -- form starts empty |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" when a fresh access link could not be issued at send time | None -- form starts empty so the client can request a new link |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Enter a phone number and request a link for their own number | -- |
| The Pro (Talia) | No | No | This mechanism is not the Pro's sign-in path (product-features.md, Access); a Pro who lands here sees the same screen as any client would, since the screen has no way to distinguish a Pro visiting by phone from a client -- the Pro's own sign-in is reached only through FEAT-29, never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); support has no route into this screen at all |
| Unauthenticated | Yes | Yes | Not applicable -- this screen is itself the unauthenticated entry point; no separate unauthenticated state exists here |
| Expired session | N/A | N/A | N/A -- this screen precedes any session; there is nothing here that can expire until after a link is requested and redeemed (see FEAT-06.SPEC-003 and FEAT-06.SPEC-004 for their own expired-session handling) |

## Layout and Content

**Header:** Screen title "Manage your booking" with the Pro's display name shown below it (e.g., "with Talia").

**Body:** A single-column form with:
- Phone number input (required, telephone-formatted), with helper text "Enter the phone number you used when booking"
- "Send me a link" action button, directly below the input
- A result region below the button that shows nothing until the form is submitted, then shows exactly one of: a confirmation message, the "no bookings found" message, a delivery-failure message with a retry action, or (when arriving from FEAT-06.SPEC-002) the "request a new link" prompt pre-populating this same result region above an otherwise empty phone field

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width, result region directly below the button.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Phone number input | Type | Captures the entered digits | Field shows entered text | Standard input focus state |
| Phone number input | Blur (empty or malformed) | Triggers field-level validation (this screen's own basic format check) | Error state on field | "Enter a valid phone number" below the field |
| "Send me a link" button | Tap | 1. Validates phone format. 2. If valid, checks the request rate limit governed by FEAT-06.SPEC-007. 3. If under the limit, matches the phone number to a Client record for this Pro, governed by FEAT-06.SPEC-008. 4. If matched, generates a single-use, 30-minute on-demand Access Link inline on this screen. 5. Triggers FEAT-06.SPEC-006 (Access Link Delivery) to send the link. | Button shows a brief loading state while the link is generated and sent | Success: result region shows "Check your phone for a link to view your bookings." No match: result region shows the plain "no bookings found" message. Rate-limited: result region shows the rate-limit message. Delivery failure: result region shows the failure message with a retry action. |
| "Send me a link" button (while loading) | Tap | No action -- ignored while a request is already in flight | None | Button remains in its loading state |
| "Try again" (delivery-failure retry) | Tap | Re-attempts delivery of the same still-valid Access Link via FEAT-06.SPEC-006, or, if it has since expired, generates a fresh one following the same steps as the original request | Button shows a brief loading state | Success: same confirmation message as above. Repeated failure: failure message remains with the retry action still available. |
| "Request a new link" (shown after an invalid/expired/used link) | Tap | Clears the prior prompt and returns the form to its normal empty state, ready for a fresh phone number entry | Result region clears, phone field is focused and empty | Standard empty-form appearance |

### Accessibility Notes

- **Focus order:** Phone number input -> "Send me a link" button -> result region (when populated).
- **Validation and result announcements:** The field's error state and the result region's message (confirmation, no-bookings, rate-limit, or delivery-failure) are announced to assistive technology as they appear, and the result message is programmatically associated with the form.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Phone field empty, button enabled, no result region content | Screen first opens with no incoming re-request context | Client begins typing or the screen is entered via FEAT-06.SPEC-002 with a re-request prompt |
| Filling | Phone field contains client input | Client types in the field | Client taps "Send me a link" or navigates away |
| Sending | Button shows loading state, form disabled | Client taps "Send me a link" or "Try again" | Request completes with any outcome below |
| Link Sent | Result region shows "Check your phone for a link to view your bookings." | Phone matched a Client record and delivery was attempted successfully | Client leaves the screen (typically to check their phone) |
| No Bookings Found | Result region shows the plain "no bookings found" message; phone field remains editable for another attempt | Phone number does not match any Client record for this Pro | Client edits the phone number and resubmits |
| Rate Limited | Result region shows a plain message that no more links can be requested for this number right now and to try again shortly | The per-phone-number request rate limit (FEAT-06.SPEC-007) is exceeded | The rate-limit window (FEAT-06.SPEC-007) elapses and the client resubmits |
| Delivery Failed | Result region shows "We couldn't send your link. Try again." with a "Try again" action | Text or email delivery of the generated link fails (FEAT-06.SPEC-006) | Client taps "Try again" |
| Re-Request Prompt | Result region shows "That link no longer works. Request a new one." with a single-tap "Request a new link" action; phone field is empty below it | Client arrives from FEAT-06.SPEC-002 after an invalid, expired, or already-used link | Client taps "Request a new link", returning to the Empty state |
| Offline/Degraded | Banner "You're offline. Connect to the internet to request a link." at the top; the phone field remains visible but the "Send me a link" button is disabled while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the button re-enables and the banner clears |

## Validation Rules

**Option B -- Inline (simple validation, not shared beyond this screen):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Phone number | Required, must be a valid reachable phone number format | On blur and on submit | "Enter a valid phone number" |

Rate-limit and phone-to-Client matching rules are not defined here -- see FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) for the rate limit and FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) for matching.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Client taps the delivered link (outside this screen, on their phone) | FEAT-06.SPEC-002 (Access Link Validation & Redemption) | -- |

## Data Model

**Creates:** Access Link -- scope set to "all of this client's bookings with this Pro," expiry set to 30 minutes from creation, state set to Issued. Created only when the phone number matches a Client record (FEAT-06.SPEC-008) and the rate limit permits it (FEAT-06.SPEC-007).
**Reads:** Client -- phone field, matched against the entered number for this Pro only (FEAT-06.SPEC-008).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Rate limiting on link requests per phone number is governed by FEAT-06.SPEC-007 -- this screen enforces it but does not redefine it.
- Phone-to-Client matching and the cross-client/cross-Pro privacy isolation boundary are governed by FEAT-06.SPEC-008 -- a phone number with no match never reveals whether it exists for another Pro or another client.
- Every generated Access Link is delivered by FEAT-06.SPEC-006, which decides the channel (text with active consent, otherwise email).
- XBR-18: Access links open only that one client's bookings with that one Pro; on-demand links are single-use for 30 minutes.

## Edge Cases

- **Client resubmits before a previously issued, still-valid link expires** -- A new Access Link is generated and delivered as normal; the prior link remains valid independently until it is used or its own 30-minute expiry is reached (governed by FEAT-06.SPEC-007). The client is not warned about the earlier link.
- **Client double-taps "Send me a link"** -- The second tap is ignored while the first request is in flight (button in loading state).
- **Client enters a phone number that matches a Client record for a different Pro entirely** -- Treated identically to no match at all: the plain "no bookings found" message, with no hint that the number exists for another Pro (FEAT-06.SPEC-008).
- **Client navigates away while the request is in flight** -- The request completes in the background; if the client returns to this screen, it reloads in the Empty state rather than showing a stale result.
- **No Access Link screen ever updates an existing Access Link, Client, or Booking record** -- this is a creation-only screen (it reads Client to match, and creates a new Access Link), so no concurrent-edit conflict entry applies here; the Access Link entity's own single-use contention is handled at redemption time by FEAT-06.SPEC-002.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | Rate-limit check at request time and the 30-minute expiry set on the created link |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (outbound) | Phone-to-Client matching and the privacy isolation boundary |
| FEAT-06.SPEC-006 (Access Link Delivery) | Triggers (outbound) | A successful match triggers delivery of the generated link |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | Routes back here with a re-request prompt after an invalid, expired, or already-used link |
| FEAT-05.SPEC-003 (Client Details & Consent) | Related (no navigation) | FEAT-05.SPEC-003 performs the returning-client lookup in place (name pre-fill) and does not navigate here; that in-place lookup applies the same identity-match rule as FEAT-06.SPEC-008 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| access_link_requested | outcome (matched / no_match / rate_limited), entry source (default / re_request_prompt / from_waitlist) | "Send me a link" completes processing | supports success-metrics.md: "Self-Service Access Success" |
| access_link_delivery_failed | retry_attempted (yes / no) | Text or email delivery of the generated link fails | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-001-AC-01:** Given Riley is on the Access Link Request screen, when she enters the phone number she used at booking and taps "Send me a link", then the number is matched to her Client record, an Access Link is generated, delivery is triggered, and the result region shows "Check your phone for a link to view your bookings."

**FEAT-06.SPEC-001-AC-02:** Given Riley is on the Access Link Request screen, when she taps "Send me a link" with the phone field empty, then the field shows the error "Enter a valid phone number" and no request is sent.

**FEAT-06.SPEC-001-AC-03:** Given Riley enters a phone number with no matching Client record for this Pro, when she taps "Send me a link", then the result region shows a plain "no bookings found" message with no hint about whether the number exists elsewhere.

**FEAT-06.SPEC-001-AC-04:** Given Riley has already requested the maximum number of links for her phone number within the current rate-limit window (FEAT-06.SPEC-007), when she taps "Send me a link" again, then the result region shows the rate-limited message and no new link is generated.

**FEAT-06.SPEC-001-AC-05:** Given Riley's requested link fails to deliver, when the failure is detected, then the result region shows "We couldn't send your link. Try again." with a "Try again" action.

**FEAT-06.SPEC-001-AC-06:** Given Riley taps "Try again" after a delivery failure and the original link is still valid, when the retry is processed, then the same Access Link is re-delivered rather than a new one being generated.

**FEAT-06.SPEC-001-AC-07:** Given Riley arrives at this screen from FEAT-06.SPEC-002 after tapping an expired link, when the screen loads, then it shows the "That link no longer works. Request a new one." prompt with a single-tap "Request a new link" action.

**FEAT-06.SPEC-001-AC-08:** Given Riley is viewing the re-request prompt, when she taps "Request a new link", then the screen returns to its empty state with the phone field cleared and focused.

**FEAT-06.SPEC-001-AC-09:** Given Riley is on this screen with no connectivity, when the screen detects she is offline, then the banner "You're offline. Connect to the internet to request a link." appears and the "Send me a link" button is disabled.

**FEAT-06.SPEC-001-AC-10:** Given Riley loses connectivity while the offline banner is shown, when connectivity is restored, then the banner clears and the "Send me a link" button re-enables.

**FEAT-06.SPEC-001-AC-11:** Given Riley taps "Send me a link" and the request is still processing, when she taps the button again immediately, then the second tap is ignored and the button remains in its loading state.

**FEAT-06.SPEC-001-AC-12:** Given Riley arrives at this screen from the waitlist Confirmed state (FEAT-20.SPEC-001) or a waitlist expiry notification (FEAT-20.SPEC-009), when the screen loads, then the form starts empty and she enters her phone number and taps "Send me a link".

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (no bookings found, rate limited, delivery failed, re-request prompt, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Access Link Validation & Redemption

## Overview

**Name:** Access Link Validation & Redemption
**ID:** FEAT-06.SPEC-002
**Type:** Automation
**Purpose:** System validates a tapped access link -- on-demand or booking-specific -- checks its scope, expiry, and used/unused state, writes the resulting state, and routes the client to the right screen.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Looking up the token carried by any tapped access link (on-demand or booking-specific)
- Checking the link's expiry, single-use state, and scope against FEAT-06.SPEC-007's lifecycle rules
- Marking the link Used on first successful redemption
- Routing the client to FEAT-06.SPEC-003 (on-demand, valid) or FEAT-06.SPEC-004 (booking-specific, valid), or back to FEAT-06.SPEC-001 with a re-request prompt (invalid, expired, or already used)

**Non-Goals:**
- Generating the access link -- on-demand links are created by FEAT-06.SPEC-001, booking-specific links by FEAT-08 and FEAT-30; this spec only consumes an already-issued link
- Displaying the client's bookings once routed -- owned by FEAT-06.SPEC-003 and FEAT-06.SPEC-004
- Enforcing the rate limit on how often a link can be requested -- owned by FEAT-06.SPEC-007; this spec enforces expiry and single-use state, not request frequency
- Any password or code-entry fallback if the link fails -- excluded per BRIEF.md's Target Users & Roles: the only recovery path is requesting a new link (FEAT-06.SPEC-001), never a separate reset flow to learn

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps a delivered on-demand access link | FEAT-06.SPEC-006 (Access Link Delivery) | Always, whenever a delivered link is tapped, regardless of its current validity | The tapped link's token |
| Client taps a booking-specific manage link from a confirmation or reminder | FEAT-08.SPEC-012 (text delivery) or FEAT-08.SPEC-013 (email delivery) | Always, whenever a booking-specific link embedded in a confirmation or reminder message is tapped | The tapped link's token |
| Client taps a fresh booking-specific manage link issued after a Pro reschedule | FEAT-30 (Pro Booking Management) | Fires when the client taps the replacement link issued after the Pro reschedules the booking | The tapped link's token |

## Processing Logic

1. Extract the token from the tapped link.
2. Look up the Access Link record by that token.
3. If no Access Link record matches the token, treat it as invalid (Outcome: Invalid Token).
4. If a record is found, read its scope (all bookings with this Pro, or one booking), expiry, and current state (Issued | Used | Expired).
5. If the state is already Used, treat it as invalid (Outcome: Already Used).
6. If the state is Issued, check expiry per FEAT-06.SPEC-007: an on-demand link is expired once 30 minutes have elapsed since creation; a booking-specific link is expired once the referenced Booking's appointment start time has passed.
7. If expired by either measure, mark the link's state Expired and treat the redemption as invalid (Outcome: Expired).
8. If the link is Issued and not expired, mark its state Used and record the redemption timestamp -- this is the point at which "first tap wins" is decided.
9. Confirm the link's scope still resolves to exactly one Client record with this Pro (and, for a booking-specific link, exactly one Booking belonging to that Client), per FEAT-06.SPEC-008.
10. Route the client based on scope: an on-demand link routes to FEAT-06.SPEC-003 (My Bookings List); a booking-specific link routes directly to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for that one booking, with no code or password.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Valid, on-demand | Link found, Issued, not expired, scope is "all bookings with this Pro" | Access Link state set to Used, redemption timestamp recorded | Client is routed straight into their bookings list | FEAT-06.SPEC-003 |
| Valid, booking-specific | Link found, Issued, not expired, scope is one Booking | Access Link state set to Used, redemption timestamp recorded | Client is routed straight into that one booking's detail, with no code or password | FEAT-06.SPEC-004 |
| Invalid Token | No Access Link record matches the tapped token | None | Client is routed to FEAT-06.SPEC-001 showing the "request a new link" prompt; the message never distinguishes this from any other invalid outcome | FEAT-06.SPEC-001 |
| Already Used | Link found but its state is already Used (a second device opening a single-use link after first use) | None -- the Used state and its original redemption timestamp are left unchanged | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt | FEAT-06.SPEC-001 |
| Expired | Link found, Issued, but its expiry condition (30 minutes for on-demand, appointment passed for booking-specific) has been reached | Access Link state set to Expired | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt | FEAT-06.SPEC-001 |
| Scope resolution failure | Link is otherwise valid but its scoped Client or Booking record can no longer be resolved (e.g., the referenced booking record no longer exists in a state this feature can show) | None | Client is routed to FEAT-06.SPEC-001 showing the identical "request a new link" prompt -- never a technical error, per the "no hint" pattern | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Access Link -- token, scope, expiry, state. Client -- matched via the Access Link's scope, for this Pro only, governed by FEAT-06.SPEC-008. Booking -- for a booking-specific link, the referenced Booking's start time (to evaluate expiry) and its owning Client (to confirm scope).

**Booking reader:** This spec is a Booking reader, as the feature-overview.md Referenced Entities table records (FEAT-06.SPEC-002, FEAT-06.SPEC-003, FEAT-06.SPEC-004): it reads the referenced Booking's start time (expiry evaluation at Steps 6-7) and owning Client (scope confirmation at Step 9).
**Creates:** None -- this automation never creates a new Access Link.
**Updates:** Access Link -- state set to Used (with redemption timestamp) on a valid redemption, or to Expired when an Issued link is found past its expiry condition at redemption time.
**Deletes:** None -- links are never deleted, only transitioned to Used or Expired (scope-boundaries.md: expired and used links are retained as an audit trail, with no automatic purge).

## Business Rules

- XBR-18: Access links open only that one client's bookings with that one Pro; on-demand links are single-use for 30 minutes; booking-specific links stop working once the appointment passes; a Pro reschedule issues a fresh manage link.
- Expiry and single-use enforcement follow FEAT-06.SPEC-007 exactly -- this automation applies those rules, it does not redefine them.
- Phone-to-Client matching and the privacy isolation boundary follow FEAT-06.SPEC-008 -- a link can never resolve to a Client or Booking outside its own stated scope.
- "First tap wins": for a single-use on-demand link, the tap that reaches Step 8 first is the one that succeeds; every subsequent tap of the same token is Already Used, regardless of which device made it.
- The re-request message is identical across Invalid Token, Already Used, Expired, and Scope Resolution Failure -- the client is never told which of these occurred, per the "no hint" empty/error pattern shared with FEAT-06.SPEC-001.

## Edge Cases

- **A single-use link is opened on a second device after first use** -- The first device's tap reaches Step 8 first and succeeds; the second device's tap finds the link already Used and is routed to the identical re-request prompt. First use wins (dependency map, Access Link Contention).
- **Client taps a booking-specific link after the appointment has passed** -- The expiry check at Step 6 finds the appointment start time already past; the link is marked Expired and the client sees the same re-request pattern as any other invalid link.
- **Pro reschedules a booking while the client is mid-tap on the old manage link** -- If the old link's tap reaches Step 8 before the reschedule replaces it, the redemption is honored for the booking's prior time (the client is not left in an inconsistent state); if the reschedule commits first, the old link is superseded by the fresh one FEAT-30 issues, and the old link's tap resolves as Expired via its now-stale scope (Scope Resolution Failure), so the client is not shown outdated appointment details.
- **Concurrent trigger firing (the same token tapped from two devices at effectively the same time)** -- Only one tap can reach Step 8 first; that tap alone transitions the link to Used. The other tap, whichever arrives second, always evaluates the state as already Used (Already Used outcome) -- there is no scenario where both taps succeed.
- **Trigger fires while a previous run for the same token is still in flight** -- A second tap for the same token that arrives before the first run has finished writing the Used state is held until the first run completes, then evaluates the now-updated state and resolves to Already Used. Runs for different tokens proceed independently and never block one another.
- **Client's phone number was changed by the Pro (FEAT-13) between link issuance and tap** -- Per XBR-18, a Pro-initiated phone number change invalidates the client's existing access links; the tapped link resolves as Scope Resolution Failure and the client sees the standard re-request prompt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-006 (Access Link Delivery) | Triggered by (inbound) | On-demand links delivered by this spec are what the client taps to fire this automation |
| FEAT-08.SPEC-012 (Text send and delivery) / FEAT-08.SPEC-013 (Email send and delivery) | Triggered by (inbound, external-event source) | These are the integration sources of the booking-specific manage links embedded in confirmations and reminders; a tap on such a link fires this automation |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A fresh booking-specific manage link issued after a Pro reschedule fires this automation when tapped |
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (outbound) | Expiry and single-use rules applied at Steps 6-8 |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (outbound) | Scope resolution and the privacy isolation boundary applied at Step 9 |
| FEAT-06.SPEC-001 (Access Link Request) | Affects (outbound) | Every invalid outcome routes here with the re-request prompt |
| FEAT-06.SPEC-003 (My Bookings List) | Affects (outbound) | A valid on-demand redemption routes here |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Affects (outbound) | A valid booking-specific redemption routes here |

**Note:** This spec is a Booking reader (see Data Model above); it never writes a Booking.

## Analytics and Success Signals

- **access_link_used** (scope: on_demand / booking_specific) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_expired_unused** (scope: on_demand / booking_specific) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_redemption_failed** (reason: invalid_token / already_used / scope_resolution_failure) -- N/A -- no Stage 2 metric distinguishes these sub-reasons from a plain expiry; retained so a silent-seeming failure mode is still observable internally rather than invisible.

## Acceptance Criteria

**FEAT-06.SPEC-002-AC-01:** Given Riley taps a valid, unused, unexpired on-demand access link within 30 minutes of it being issued, when the automation validates it, then the link is marked Used and Riley is routed to her My Bookings List (FEAT-06.SPEC-003).

**FEAT-06.SPEC-002-AC-02:** Given Riley taps a valid, unused, unexpired booking-specific manage link from a reminder text, when the automation validates it, then the link is marked Used and Riley is routed straight into that booking's detail (FEAT-06.SPEC-004), with no code or password required.

**FEAT-06.SPEC-002-AC-03:** Given Riley taps a link whose token matches no Access Link record, when the automation looks it up, then Riley is routed to FEAT-06.SPEC-001 showing the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-04:** Given Riley's on-demand link was already used from another device, when Riley taps the same link, then the automation finds it already Used and routes Riley to FEAT-06.SPEC-001 showing the identical "request a new link" prompt.

**FEAT-06.SPEC-002-AC-05:** Given more than 30 minutes have passed since Riley's on-demand link was issued, when Riley taps it, then the automation marks it Expired and routes her to FEAT-06.SPEC-001 showing the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-06:** Given Riley's booking-specific manage link points to an appointment whose start time has already passed, when Riley taps it, then the automation marks it Expired and routes her to the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-07:** Given Riley's link is otherwise valid but Talia changed Riley's phone number since the link was issued (FEAT-13, XBR-18), when Riley taps the link, then the automation resolves it as Scope Resolution Failure and routes her to the "request a new link" prompt.

**FEAT-06.SPEC-002-AC-08:** Given Riley's single-use on-demand link is tapped from two devices at effectively the same time, when the automation processes both taps, then exactly one succeeds (marks the link Used and routes to FEAT-06.SPEC-003) and the other resolves as already used, routing to FEAT-06.SPEC-001.

**FEAT-06.SPEC-002-AC-09:** Given a second tap of the same token arrives while the first tap's redemption is still being processed, when the automation handles the second tap, then it waits for the first run to complete and then evaluates the link as already Used.

**FEAT-06.SPEC-002-AC-10:** Given Talia reschedules Riley's booking after issuing a manage link but before Riley taps the old one, when Riley taps the old link, then it resolves as Scope Resolution Failure and Riley is routed to the "request a new link" prompt rather than seeing outdated appointment details.

**FEAT-06.SPEC-002-AC-11:** Given Riley taps a booking-specific link issued fresh by FEAT-30 after a Pro reschedule, when the automation validates it, then it is treated exactly like any other valid booking-specific link and routes Riley to FEAT-06.SPEC-004 for the rescheduled appointment.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: My Bookings List

## Overview

**Name:** My Bookings List
**ID:** FEAT-06.SPEC-003
**Type:** Screen
**Purpose:** Client views their own past and upcoming bookings with this one Pro after redeeming an on-demand access link.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Listing the matched Client's upcoming and past bookings with this Pro
- Navigating from a booking row into its detail (FEAT-06.SPEC-004)
- Surfacing an active waitlist entry with a "Leave" action (outbound to FEAT-20)
- A link out to update texting consent and email preferences (FEAT-06.SPEC-005)
- A recurring-series section, shown only when the client holds a series with this Pro, linking to FEAT-21.SPEC-002 (My Recurring Series)

**Non-Goals:**
- Viewing or acting on a single booking in detail -- owned by FEAT-06.SPEC-004
- Cancelling or rescheduling a booking -- owned by FEAT-10 (Client-Initiated Cancel/Reschedule), reached from FEAT-06.SPEC-004
- Browsing any other client's bookings, or this client's bookings with any other Pro -- excluded per scope-boundaries SC-03 and SC-04: cross-pro and cross-client visibility do not exist in this product
- Joining a waitlist -- owned by FEAT-20 (Waitlist for Cancelled Slots); this screen only surfaces an existing entry's "Leave" action
- Viewing, setting up, or cancelling a recurring series -- owned by FEAT-21 (FEAT-21.SPEC-001 sets one up, FEAT-21.SPEC-002 manages it); this screen only links to the series section

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | A valid, unexpired, unused on-demand access link is redeemed | The matched Client's identity for this Pro (established by FEAT-06.SPEC-008) |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps back from a booking's detail | None -- list re-displays as last loaded |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Client taps back after updating preferences | None -- list re-displays as last loaded |
| FEAT-21.SPEC-002 (My Recurring Series) | Client taps back from the recurring series screen | None -- list re-displays as last loaded |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own bookings with this one Pro only, past and upcoming | Select a booking to view detail; leave a waitlist entry; navigate to preferences | -- |
| The Pro (Talia) | No | No | This mechanism is not the Pro's own dashboard; the Pro's equivalent view is FEAT-12 (Pro Daily Schedule Dashboard), reached through Pro sign-in (FEAT-29), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); this screen has no support entry point |
| Unauthenticated | No | No | This screen is reachable only immediately after FEAT-06.SPEC-002 redeems a valid link; a direct, unauthenticated attempt to open it is redirected to FEAT-06.SPEC-001 to request a link |
| Expired session | No | No | The redeemed link's viewing session lasts only for the current page (until the tab is closed or reloaded); reloading or returning after the underlying link has already transitioned to Used is treated as an unauthenticated attempt and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "My Bookings" with the Pro's display name shown below it, and a "Preferences" link (top-right) to FEAT-06.SPEC-005.

**Body:**
- **Upcoming** section: one row per upcoming booking, each showing service name, date and time, and paid/balance-due status. Tapping a row navigates to FEAT-06.SPEC-004.
- **Waitlist** section (shown only when the client has an active waitlist entry): one row per active entry showing the requested service and date range, with a "Leave" action next to it.
- **Recurring appointments** section (shown only when the client holds at least one Recurring Series with this Pro; no extra UI at all otherwise): one row "Recurring appointments" showing the number of active series, tapping it navigates to FEAT-21.SPEC-002.
- **Past** section: one row per past booking, each showing service name, date, and outcome (completed, no-show, cancelled). Tapping a row navigates to FEAT-06.SPEC-004.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order Upcoming, Waitlist, Recurring appointments, Past, each full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Preferences" link | Tap | Navigate to FEAT-06.SPEC-005 (Consent & Email Preferences) | Screen transitions | Standard navigation transition |
| Upcoming booking row | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for that booking | Screen transitions | Standard navigation transition |
| Past booking row | Tap | Navigate to FEAT-06.SPEC-004 for that booking | Screen transitions | Standard navigation transition |
| Recurring appointments row | Tap | Navigate to FEAT-21.SPEC-002 (My Recurring Series) | Screen transitions | Standard navigation transition |
| Waitlist "Leave" action | Tap | Confirms intent, then navigates to FEAT-20.SPEC-002 (My Waitlists) within FEAT-20 (Waitlist for Cancelled Slots) to remove the entry | Confirmation prompt appears before the outbound navigation | Confirmation dialog "Leave the waitlist for {service}?" with "Leave" and "Cancel" |

### Accessibility Notes

- **Focus order:** "Preferences" link -> Upcoming rows in date order -> Waitlist row(s) and their "Leave" actions -> Recurring appointments row -> Past rows in date order (most recent first).
- **Dynamic announcements:** When a waitlist entry is removed after confirmation, its row's removal from the list is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the list will appear | Screen first loads after redemption | Data finishes loading |
| Populated | Upcoming, Waitlist (if any), and Past sections shown with their rows | Data loads successfully with at least one booking | Client navigates away |
| No Upcoming | Upcoming section shows "No upcoming bookings" in place of rows; Past section (if any) still shows | Client has no upcoming bookings with this Pro | A new upcoming booking appears on a future visit to this screen |
| No Past | Past section shows "No past bookings yet" in place of rows; Upcoming section (if any) still shows | Client has no past bookings with this Pro | A booking completes and appears here on a future visit |
| Error | Error banner "We couldn't load your bookings. Try again." with a retry action, in place of the list | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | The already-loaded list remains visible read-only; a banner "You're offline -- reconnect to view details or leave a waitlist." appears; row taps and the "Leave" action are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and actions re-enable |

## Validation Rules

Not applicable -- this screen has no user input fields; validation of the access that brought the client here is governed by FEAT-06.SPEC-002 (Access Link Validation & Redemption) and FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Preferences" link tap | FEAT-06.SPEC-005 (Consent & Email Preferences) | -- |
| Booking row tap (upcoming or past) | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | -- |
| Recurring appointments row tap | FEAT-21.SPEC-002 (My Recurring Series) | FEAT-21 (Recurring Appointments) |
| Waitlist "Leave" confirmed | FEAT-20.SPEC-002 (My Waitlists), waitlist removal | FEAT-20 (Waitlist for Cancelled Slots) |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start time, price/deposit status, and state (for outcome labels), scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Waitlist Entry -- service, date range, and state (Requested/Notified), scoped to the same Client. Recurring Series -- only a count of the matched Client's active series with this Pro, to decide whether the Recurring appointments section is shown.
**Updates:** None directly -- the "Leave" action's actual removal is performed by FEAT-20.
**Deletes:** None directly.

## Business Rules

- Every booking and waitlist entry shown is scoped to exactly the Client record matched by FEAT-06.SPEC-008 for this one Pro -- no cross-client or cross-Pro data can ever appear here.
- Access to this screen exists only as the outcome of a successful redemption by FEAT-06.SPEC-002; there is no independent sign-in to reach it.
- XBR-18: this screen's entire contents are scoped to the client identified by the redeemed access link.

## Edge Cases

- **Client leaves this screen open and a booking's status changes on the Pro's side in the meantime (e.g., Talia marks it completed)** -- The already-loaded row keeps showing its state as of load time; the current state is fetched fresh when the client taps into FEAT-06.SPEC-004, so no stale action is ever taken against an out-of-date booking. This screen performs no writes, so no concurrent-edit conflict applies to it directly.
- **Client has both zero upcoming and zero past bookings** -- Both sections show their respective empty messages; this is a rare state (a matched Client record implies at least one historical booking) but is handled without an error.
- **Client taps a Past booking that was later deleted from view due to a client-record deletion elsewhere (FEAT-13)** -- Not applicable in practice: FEAT-13's client deletion is refused while an upcoming booking exists, and this screen's session ends when the access link that produced it expires; a client viewing this list mid-session before their own record is deleted continues to see the data as loaded.
- **Client double-taps a booking row** -- The second tap is ignored while the navigation to FEAT-06.SPEC-004 is already in progress.
- **Client reloads the page after the link has already been marked Used** -- Treated as an unauthenticated attempt (per Access and Visibility): the client is redirected to FEAT-06.SPEC-001 with the "request a new link" prompt rather than seeing a broken or empty list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | A valid on-demand redemption routes here |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes every row shown to the matched Client with this Pro |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | Selecting a booking opens its detail |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Navigation (outbound) | The "Preferences" link opens the client's own settings |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound/inbound) | The Recurring appointments row opens the client's series; its back arrow returns here |
| FEAT-20.SPEC-002 (My Waitlists) -- within FEAT-20 (Waitlist for Cancelled Slots) | Navigation (outbound) | The "Leave" action removes a waitlist entry there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| bookings_list_viewed | upcoming_count, past_count | Screen finishes loading with data | supports success-metrics.md: "Self-Service Access Success" |
| bookings_list_load_failed | -- | The initial data load fails | supports success-metrics.md: "Self-Service Access Success" |
| waitlist_leave_confirmed | -- | Client confirms leaving a waitlist entry from this screen | N/A -- no Stage 2 metric measures waitlist activity from this feature; retained so the action's use here is observable rather than invisible. |

## Acceptance Criteria

**FEAT-06.SPEC-003-AC-01:** Given Riley has just redeemed a valid on-demand access link, when the My Bookings List loads, then she sees her upcoming and past bookings with Talia only, and no bookings from any other Pro or client.

**FEAT-06.SPEC-003-AC-02:** Given Riley is on the My Bookings List with one upcoming booking, when she taps that booking's row, then she is taken to its detail on FEAT-06.SPEC-004.

**FEAT-06.SPEC-003-AC-03:** Given Riley has no upcoming bookings with Talia, when the list loads, then the Upcoming section shows "No upcoming bookings" instead of any rows.

**FEAT-06.SPEC-003-AC-04:** Given Riley has an active waitlist entry for a service, when the list loads, then a Waitlist section appears showing that entry with a "Leave" action.

**FEAT-06.SPEC-003-AC-05:** Given Riley taps "Leave" on her waitlist entry, when she confirms in the dialog, then she is taken to FEAT-20 to complete the removal.

**FEAT-06.SPEC-003-AC-06:** Given Riley taps "Preferences" from the My Bookings List, when the tap registers, then she is taken to FEAT-06.SPEC-005 (Consent & Email Preferences).

**FEAT-06.SPEC-003-AC-07:** Given the initial load of Riley's bookings fails, when the failure occurs, then an error banner "We couldn't load your bookings. Try again." appears with a retry action.

**FEAT-06.SPEC-003-AC-08:** Given Riley loses connectivity while viewing her already-loaded list, when connectivity drops, then the list remains visible read-only, a banner explains she is offline, and row taps and the "Leave" action are disabled.

**FEAT-06.SPEC-003-AC-09:** Given Riley's connectivity is restored after being offline on this screen, when connectivity returns, then the offline banner clears and row taps and the "Leave" action re-enable.

**FEAT-06.SPEC-003-AC-10:** Given Riley's on-demand access link has already transitioned to Used, when she reloads this screen directly, then she is redirected to FEAT-06.SPEC-001 with the "request a new link" prompt rather than seeing the list again.

**FEAT-06.SPEC-003-AC-11:** Given Riley holds an active recurring series with Talia, when the My Bookings List loads, then a "Recurring appointments" row appears, and tapping it takes her to FEAT-21.SPEC-002 (My Recurring Series).

**FEAT-06.SPEC-003-AC-12:** Given Riley holds no recurring series with Talia, when the My Bookings List loads, then no Recurring appointments section or row appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loading, no upcoming, no past, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Booking Detail via Manage Link

## Overview

**Name:** Booking Detail via Manage Link
**ID:** FEAT-06.SPEC-004
**Type:** Screen
**Purpose:** Client views one booking's full detail, opened either from the My Bookings list or directly via a booking-specific manage link, and starts a cancel, reschedule, balance payment, or preferences action from it.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Displaying one booking's full detail for the matched Client
- Entry points into cancelling/rescheduling (FEAT-10), paying the balance (FEAT-22, v1), and preferences (FEAT-06.SPEC-005)
- The concurrent-edit conflict behavior when the Pro changes the booking while this screen is open

**Non-Goals:**
- Listing multiple bookings -- owned by FEAT-06.SPEC-003 (My Bookings List)
- Performing the actual cancel or reschedule -- owned by FEAT-10; this screen only starts that flow
- Performing the actual balance payment -- owned by FEAT-22 (v1); this screen only starts that flow
- Any Pro-side view of this booking -- the Pro's equivalent is FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), entirely separate screens reached only through Pro sign-in

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Client taps an upcoming or past booking row | The selected Booking's reference |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | A valid, unexpired, unused booking-specific manage link is redeemed | The one Booking the link is scoped to |
| FEAT-10.SPEC-001 (Cancel Booking) | Client completes a cancellation, taps "Keep my booking", or taps back | The same Booking reference, showing its current state |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client confirms an outside-window reschedule successfully | The same Booking reference with its new start_time |
| FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) | Client taps "Manage my booking" in the cancellation/reschedule notice | The affected Booking reference (the original booking for a cancellation, the new booking for a late reschedule) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full detail of a booking that belongs to their own matched Client record with this Pro | Start cancel/reschedule, start balance payment (v1), open preferences | -- |
| The Pro (Talia) | No | No | This is not the Pro's own booking view; the Pro's equivalent is reached through FEAT-12/FEAT-30 via Pro sign-in, never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-06.SPEC-003 (already-authenticated navigation) or a valid booking-specific link redeemed by FEAT-06.SPEC-002; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; reloading after the underlying access link has transitioned to Used is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title showing the service name, with a back arrow (returns to FEAT-06.SPEC-003 when arrived from there, or shows no back arrow when arrived directly via a booking-specific link, since there is no list to return to in that session).

**Body:**
- Appointment summary: date, time, duration, and the studio address (shown per this booking's confirmation-only disclosure)
- Payment summary: price agreed, deposit amount and paid status, balance due
- Cancellation policy summary: the plain-language wording and window that was acknowledged at booking, and the resulting outcome if cancelled now (refund vs. kept), consistent with FEAT-09's policy engine
- Status line: current booking state (Confirmed, Awaiting Outcome, Completed, No-Show, Cancelled, Rescheduled)
- Action row: "Cancel or Reschedule" button (only when the booking's state allows it, per FEAT-10/FEAT-09's rules); "Pay Balance" button (v1, only when a balance is due and unpaid); a "Preferences" link to FEAT-06.SPEC-005

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; the action row's buttons stack full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; action row buttons sit side by side instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (when present) | Tap | Navigate to FEAT-06.SPEC-003 | Screen closes | Standard navigation transition |
| "Cancel or Reschedule" button | Tap | Navigate to FEAT-10 (Client-Initiated Cancel/Reschedule) for this booking | Screen transitions | Standard navigation transition |
| "Pay Balance" button (v1) | Tap | Navigate to FEAT-22 (In-App Balance Payment) for this booking | Screen transitions | Standard navigation transition |
| "Preferences" link | Tap | Navigate to FEAT-06.SPEC-005 | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow (when present) -> appointment summary -> payment summary -> cancellation policy summary -> status line -> "Cancel or Reschedule" -> "Pay Balance" (when shown) -> "Preferences".
- **Dynamic announcements:** A concurrent-edit conflict message (see Edge Cases) is announced to assistive technology as soon as it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the detail will appear | Screen first opens | Data finishes loading |
| Populated | Full booking detail shown as described in Layout and Content | Data loads successfully | Client navigates away |
| Error | Error banner "We couldn't load this booking. Try again." with a retry action | The initial data load fails | Client taps Retry and the load succeeds |
| Booking No Longer Available | Plain message "This booking is no longer available." replaces the detail, with no further detail shown | The booking's underlying record cannot be resolved for this client (e.g., a scope mismatch) | Client navigates back to FEAT-06.SPEC-003 (when reachable) or requests a new link |
| Offline/Degraded | The already-loaded detail remains visible read-only; a banner "You're offline -- reconnect to take action on this booking." appears; the action row's buttons are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and buttons re-enable |

## Validation Rules

Not applicable -- this screen has no user input fields. Access to it is governed by FEAT-06.SPEC-002 and FEAT-06.SPEC-008; the actions it exposes and their own rules belong to FEAT-10, FEAT-22, and FEAT-09.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (when present) | FEAT-06.SPEC-003 (My Bookings List) | -- |
| "Cancel or Reschedule" tap | Cancel/reschedule flow, FEAT-10.SPEC-001 (Cancel Booking) | FEAT-10 (Client-Initiated Cancel/Reschedule) |
| "Pay Balance" tap (v1) | Balance payment flow, FEAT-22.SPEC-001 (Balance Payment) | FEAT-22 (In-App Balance Payment) |
| "Preferences" tap | FEAT-06.SPEC-005 (Consent & Email Preferences) | -- |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, price_agreed, deposit_amount, policy_version, state, balance_due, cancellation/reschedule timestamps, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Cancellation Policy -- the version referenced by the booking, for the plain-language wording and outcome preview.
**Updates:** None directly -- cancel, reschedule, and balance payment are all performed by the destination specs this screen navigates to.
**Deletes:** None.

## Business Rules

- Every field shown is scoped to the one Booking resolved by the redeemed link or by selection from FEAT-06.SPEC-003 -- never another client's or another Pro's booking.
- The "Cancel or Reschedule" button is shown only when the booking's current state permits it (per FEAT-09's cancellation policy engine and XBR-12's outcome windows); a Completed or No-Show booking never shows this button.
- The "Pay Balance" button (v1) is shown only when balance_due is greater than zero and unpaid, per XBR-23.
- The studio address is shown here because this is a booked client's own confirmation-equivalent view, consistent with the Pro Account's confirmation-only disclosure rule.
- XBR-18: this screen's contents are scoped to the client identified by the redeemed access link (or by the prior selection from FEAT-06.SPEC-003, itself scoped the same way).

## Edge Cases

- **Talia cancels or reschedules this booking on her side while Riley has this screen open** -- The Booking entity's dependency-map Contention note calls for reject-with-refresh under high contention. If Riley taps "Cancel or Reschedule" after Talia's change has committed, her action is rejected and she sees a dialog: "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action that reloads the current state; her original tap is not carried through to FEAT-10 against stale data.
- **Booking is marked Completed or No-Show by Talia while Riley is viewing** -- The already-loaded screen does not silently rewrite itself mid-view; the updated status and the disappearance of the "Cancel or Reschedule" button take effect the next time the screen is loaded (fresh navigation or reload), consistent with this being a snapshot view, not a live-updating one.
- **Booking-specific link is redeemed for a booking that has since been fully cancelled and its record scope no longer matches** -- The screen shows the "Booking No Longer Available" state rather than stale or partial detail.
- **Client taps "Cancel or Reschedule" twice rapidly** -- The second tap is ignored while the navigation to FEAT-10 is already in progress.
- **Client loses connectivity mid-view, then taps "Pay Balance"** -- The button is disabled while offline (per the Offline/Degraded state), so the tap has no effect until connectivity returns.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | Selecting a booking there opens this screen |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | A valid booking-specific redemption routes directly here |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes the displayed booking to the matched Client with this Pro |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Navigation (outbound) | The "Preferences" link opens the client's own settings |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Navigation (outbound) | "Cancel or Reschedule" starts that flow for this booking |
| FEAT-22 (In-App Balance Payment) | Navigation (outbound) | "Pay Balance" starts that flow for this booking (v1) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| booking_detail_viewed | entry source (list / manage_link), booking_state | Screen finishes loading with data | supports success-metrics.md: "Self-Service Access Success" |
| booking_detail_conflict_shown | -- | Riley's action is rejected because Talia's change committed first | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Riley taps a valid booking-specific manage link, when FEAT-06.SPEC-002 redeems it, then she lands directly on this booking's detail with no code or password step.

**FEAT-06.SPEC-004-AC-02:** Given Riley selects a booking from FEAT-06.SPEC-003, when the detail screen loads, then it shows the appointment summary, payment summary, cancellation policy summary, and status line for that one booking.

**FEAT-06.SPEC-004-AC-03:** Given Riley's booking is in a Completed state, when the detail screen loads, then no "Cancel or Reschedule" button is shown.

**FEAT-06.SPEC-004-AC-04:** Given Riley's booking has a balance due and unpaid (v1), when the detail screen loads, then a "Pay Balance" button is shown; given the balance is already paid in full, then the button is not shown.

**FEAT-06.SPEC-004-AC-05:** Given Riley taps "Cancel or Reschedule" on a cancellable booking, when the tap registers, then she is taken to FEAT-10 for that booking.

**FEAT-06.SPEC-004-AC-06:** Given Talia cancels Riley's booking on her side while Riley is viewing this screen, when Riley then taps "Cancel or Reschedule", then her action is rejected with the dialog "This booking's details changed. Refresh to see the latest before continuing." and no stale request reaches FEAT-10.

**FEAT-06.SPEC-004-AC-07:** Given Riley taps "Refresh" in the conflict dialog, when the reload completes, then the screen shows the booking's current, up-to-date state.

**FEAT-06.SPEC-004-AC-08:** Given the initial load of this booking's detail fails, when the failure occurs, then an error banner "We couldn't load this booking. Try again." appears with a retry action.

**FEAT-06.SPEC-004-AC-09:** Given the booking this screen would show can no longer be resolved for Riley's Client record, when the screen attempts to load it, then the "Booking No Longer Available" message is shown instead of any booking detail.

**FEAT-06.SPEC-004-AC-10:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the loaded detail remains visible read-only, an offline banner appears, and the action buttons are disabled.

**FEAT-06.SPEC-004-AC-11:** Given Riley taps "Preferences" from this screen, when the tap registers, then she is taken to FEAT-06.SPEC-005 (Consent & Email Preferences).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (loading, error, booking no longer available, offline) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Consent & Email Preferences

## Overview

**Name:** Consent & Email Preferences
**ID:** FEAT-06.SPEC-005
**Type:** Screen
**Purpose:** Client updates their own texting consent (including re-granting it after opting out) and their email address for this Pro.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Displaying the client's current texting consent state for this Pro
- Re-granting texting consent from a Revoked state
- Editing the client's own email address on file for this Pro
- Being the single client-facing Preferences screen: it hosts the consent section defined by FEAT-14.SPEC-001 (the consent display and re-grant control below are that section)
- A "Message channel" element (SMS or WhatsApp) linking to FEAT-26.SPEC-001, which writes the client's preferred_message_channel

**Non-Goals:**
- Revoking texting consent (STOP reply or opt-out link) -- owned by FEAT-14 (Messaging Consent Management); this screen only offers re-granting, never revocation
- Editing the client's name, phone number, or private notes -- phone is the identity key (read-only to this feature) and name/notes remain Pro-editable only via FEAT-13
- Setting texting preferences for the Pro's own account -- out of scope; this screen is the Client's own settings only, unrelated to FEAT-27 (Pro Profile & Booking Page Settings)
- Writing the channel choice itself -- FEAT-26.SPEC-001 owns the channel selection and writes Client.preferred_message_channel; this screen only shows the current value and links out

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Client taps "Preferences" | None -- current consent and email load fresh |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps "Preferences" | None -- current consent and email load fresh |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | Client taps the back arrow or completes a successful Save there | None -- current consent, channel, and email load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own texting consent state and email for this one Pro | Re-grant texting consent (when Revoked); edit their own email | -- |
| The Pro (Talia) | No | No | The Pro can see but never override a client's texting consent (Access Matrix, Messaging & Consent); that read-only view belongs to FEAT-13's client record screen, not this client-facing screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); this screen has no support entry point |
| Unauthenticated | No | No | Reachable only from FEAT-06.SPEC-003 or FEAT-06.SPEC-004 within an already-authenticated viewing session; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; returning after the underlying access link has transitioned to Used is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "Preferences" with a back arrow returning to the screen the client arrived from.

**Body:**
- Texting consent section (this is the FEAT-14.SPEC-001 consent section, hosted here; consent-control behavior is governed by FEAT-14 under XBR-15): current state shown as plain text ("Texting: on" or "Texting: off"); when off, a "Turn texting back on" button is shown
- Message channel section: one row "Message channel" showing the current value of Client.preferred_message_channel ("SMS" or "WhatsApp"; "SMS" when never set), with a chevron that navigates to FEAT-26.SPEC-001
- Email section: a single email input pre-filled with the current email on file (or empty if none), with a "Save" action

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically, full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the screen the client arrived from; if the email input's current text differs from the last-loaded/last-saved email value, a confirmation dialog appears first instead (see Edge Cases) | Screen closes (no unsaved edit), or confirmation dialog opens (unsaved edit present) | Standard navigation transition, or the dialog's copy when shown |
| "Turn texting back on" button (shown only when consent is Revoked) | Tap | Writes the client's texting consent to Re-granted for this Pro; precedence against any concurrent revocation follows FEAT-14's rule (XBR-15) | Button shows a brief loading state, then the consent line updates to "Texting: on" | Confirmation message "Texting turned back on" |
| "Message channel" row | Tap | Navigate to FEAT-26.SPEC-001 (WhatsApp Channel Preference); if the email input differs from the last-saved value, the unsaved-changes dialog appears first, as for the back arrow | Screen transitions (or dialog opens) | Standard navigation transition |
| Email input | Type | Captures the entered text | Field shows entered text | Standard input focus state |
| Email input | Blur (non-empty, malformed) | Triggers field-level format validation | Error state on field | "Enter a valid email address" below the field |
| "Save" button (email) | Tap | Validates the email format, then updates the Client's email field for this Pro | Button shows a brief loading state during save | Success: toast "Preferences saved." Failure: inline error message. |
| Unsaved-changes dialog -- "Discard" | Tap | Discards the unsaved email edit and navigates back to the screen the client arrived from | Dialog closes, screen closes, the typed email value is not persisted | Standard navigation transition |
| Unsaved-changes dialog -- "Keep Editing" | Tap | Dismisses the dialog; the client remains on this screen with the unsaved email text still in the field | Dialog closes, screen remains open | Focus returns to the email input |

### Accessibility Notes

- **Focus order:** Back arrow -> texting consent line and its "Turn texting back on" button (when shown) -> "Message channel" row -> email input -> "Save" button.
- **Validation and confirmation announcements:** The email field's error state and both confirmation messages ("Texting turned back on", "Preferences saved.") are announced to assistive technology as they appear.
- **Unsaved-changes dialog:** The dialog traps focus between "Discard" and "Keep Editing" until dismissed; its text is announced to assistive technology when it opens.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the consent state and email field will appear | Screen first opens | Data finishes loading |
| Populated (Consent On) | "Texting: on" shown, no re-grant button; email field pre-filled | Data loads with active consent | Client navigates away, or consent is revoked elsewhere (FEAT-14) and this screen is reloaded |
| Populated (Consent Off) | "Texting: off" shown with the "Turn texting back on" button; email field pre-filled | Data loads with revoked consent | Client taps "Turn texting back on" and it succeeds |
| Saving | The acted-on control (re-grant button or Save) shows a loading state | Client taps "Turn texting back on" or "Save" | The write completes or fails |
| Error | Error banner "We couldn't load your preferences. Try again." with a retry action, in place of the form | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline. Reconnect to update your preferences." at the top; the form remains visible but the re-grant button and Save are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and the controls re-enable |
| Unsaved-Changes Confirmation | Modal dialog "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options, form inert behind it | Client taps the back arrow while the email input's text differs from the last-loaded/last-saved value | Client taps "Discard" (navigates away, edit not saved) or "Keep Editing" (dialog closes, form remains with the edit intact) |

## Validation Rules

**Option B -- Inline (simple validation, not shared beyond this screen):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Email | Valid email format (when non-empty) | On blur and on Save | "Enter a valid email address" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (no unsaved edit) | The screen the client arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | -- |
| Back arrow tap (unsaved edit, "Discard" confirmed) | The screen the client arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | -- |
| "Message channel" row tap (no unsaved edit, or "Discard" confirmed) | FEAT-26.SPEC-001 (WhatsApp Channel Preference) | FEAT-26 (WhatsApp Reminders) |

## Data Model

**Creates:** None.
**Reads:** Messaging Consent -- channel, state, timestamp for this Client and Pro, matched via FEAT-06.SPEC-008. Client -- email field and preferred_message_channel (SMS | WhatsApp, default SMS), for this Pro.
**Updates:** Messaging Consent -- state transitioned from Revoked to Re-granted (FEAT-06.SPEC-005 is this feature's only write path for consent; precedence rules are owned by FEAT-14). Client -- email field only; no other Client field is writable from this screen. Client.preferred_message_channel is displayed here but written only by FEAT-26.SPEC-001, and it is not part of Messaging Consent (consent stays governed by FEAT-14).
**Deletes:** None.

## Business Rules

- Only the client's own email and their own texting consent state are ever shown or editable here -- name, phone, private notes, and booking notes stay entirely out of this screen's reach.
- This screen is the single client-facing Preferences screen; FEAT-14.SPEC-001 is the consent section hosted inside it, so there is no second Preferences screen to reach from FEAT-06.SPEC-003 or FEAT-06.SPEC-004.
- The Message channel value is stored as Client.preferred_message_channel (SMS | WhatsApp, default SMS), written by FEAT-26.SPEC-001 and kept separate from Messaging Consent.
- XBR-15: no text is sent without active texting consent; a revoke is honored on the very next message; this screen offers only re-granting, never revocation.
- The re-grant written here follows FEAT-14's most-recent-explicit-action rule against any concurrent STOP reply -- the timestamp of whichever explicit action is later wins.
- Client email and consent updates here trace to FEAT-06.SPEC-008 for phone-to-Client matching -- the write always targets the one Client record already scoped by the viewing session.

## Edge Cases

- **Client taps "Turn texting back on" at effectively the same moment a STOP reply arrives for the same phone number (FEAT-14)** -- Per FEAT-14's rule (XBR-15), the most recent explicit action by timestamp wins; if the STOP reply's timestamp is later, the consent state ends as Revoked despite the client's tap succeeding as a write, and the client sees "Texting: off" on their next view of this screen rather than the "on" state their tap requested.
- **Client edits their email while offline, then regains connectivity** -- The Save button is disabled while offline (per the Offline/Degraded state), so no queued write exists; the client must tap Save again once connectivity returns.
- **Client saves an email identical to the one already on file** -- The save proceeds and shows the same "Preferences saved." confirmation; no error is raised for an unchanged value.
- **Client clears the email field entirely and taps Save while texting consent is Off** -- Per product-features.md's Client entity rule, email is required when texts are declined; the Save is rejected with "An email is required while texting is off." and the field remains in edit state.
- **Client double-taps "Turn texting back on"** -- The second tap is ignored while the first write is in flight.
- **Talia edits this same client's contact details in FEAT-13 (e.g., correcting a typo elsewhere on the record) while Riley is saving her own email here** -- Per the dependency map's Contention note for the Client entity, the two writes touch different purposes but the same record; resolution is last-write-wins between the two saves. Riley's email save completes normally and is not rejected by Talia's concurrent edit; if Talia's edit happens to also touch the email field at the same moment, whichever save commits last is the value that persists, and Riley sees her own value reflected only if her save was the later of the two on her next visit to this screen.
- **Client edits the email field and taps the back arrow without tapping Save** -- The edit is never discarded silently: a confirmation dialog appears -- "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options. Tapping "Discard" navigates back to the screen the client arrived from and the typed email value is not persisted (the field reverts to the last-saved value on any later visit). Tapping "Keep Editing" dismisses the dialog and leaves the client on this screen with the typed value still in the field, unsaved. This mirrors FEAT-13.SPEC-002's unsaved-changes pattern for the Pro-side contact edit screen. The texting-consent re-grant control is unaffected by this edge case -- it writes immediately on tap and holds no draft state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | The "Preferences" link there opens this screen |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound) | The "Preferences" link there opens this screen |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes the consent and email shown and written to the matched Client with this Pro |
| FEAT-14.SPEC-001 (Consent & Preferences) | Hosts (outbound) | The consent section on this screen; FEAT-14.SPEC-001 lists this screen in its Entry Points |
| FEAT-14 (Messaging Consent Management) | References (outbound) | Owns consent state precedence, revocation, and the no-text fallback rule this screen's re-grant write is subject to |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | Navigation (outbound/inbound) | The Message channel row opens it; its back arrow and successful Save return here. It writes Client.preferred_message_channel |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| texting_consent_regranted | -- | Client's re-grant write succeeds | supports success-metrics.md: "Self-Service Access Success" |
| client_email_updated | had_previous_value (yes / no) | Client's email save succeeds | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-005-AC-01:** Given Riley's texting consent for Talia is currently Revoked, when she opens this screen, then it shows "Texting: off" with a "Turn texting back on" button.

**FEAT-06.SPEC-005-AC-02:** Given Riley taps "Turn texting back on", when the write succeeds, then the consent line updates to "Texting: on" and she sees "Texting turned back on".

**FEAT-06.SPEC-005-AC-03:** Given Riley's texting consent is already active, when she opens this screen, then it shows "Texting: on" with no re-grant button shown.

**FEAT-06.SPEC-005-AC-04:** Given Riley enters an invalid email format and blurs the field, when validation runs, then the field shows "Enter a valid email address" and Save is not enabled.

**FEAT-06.SPEC-005-AC-05:** Given Riley enters a valid new email and taps Save, when the save completes, then she sees the toast "Preferences saved." and the field reflects the new email on a later visit.

**FEAT-06.SPEC-005-AC-06:** Given Riley's texting consent is Off and she clears the email field entirely, when she taps Save, then the save is rejected with "An email is required while texting is off." and the field stays editable.

**FEAT-06.SPEC-005-AC-07:** Given Riley taps "Turn texting back on" at the same time a STOP reply for her phone number is processed by FEAT-14 with a later timestamp, when precedence is resolved, then her consent state ends as Revoked and shows "Texting: off" on her next view.

**FEAT-06.SPEC-005-AC-08:** Given the initial load of Riley's preferences fails, when the failure occurs, then an error banner "We couldn't load your preferences. Try again." appears with a retry action.

**FEAT-06.SPEC-005-AC-09:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then a banner explains she is offline and both the re-grant button and Save become disabled.

**FEAT-06.SPEC-005-AC-10:** Given Riley taps "Turn texting back on" twice rapidly, when the second tap registers, then it is ignored while the first write is still in progress.

**FEAT-06.SPEC-005-AC-11:** Given Riley has typed a new email value into the field but has not tapped Save, when she taps the back arrow, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-06.SPEC-005-AC-12:** Given the unsaved-changes dialog is shown after Riley's back-arrow tap, when she taps "Discard", then she is returned to the screen she arrived from and her typed email edit is not saved.

**FEAT-06.SPEC-005-AC-13:** Given the unsaved-changes dialog is shown after Riley's back-arrow tap, when she taps "Keep Editing", then the dialog closes, she remains on this screen, and her typed email value is still in the field, unsaved.

**FEAT-06.SPEC-005-AC-14:** Given Riley has never chosen a channel, when she opens this screen, then the Message channel row shows "SMS".

**FEAT-06.SPEC-005-AC-15:** Given Riley taps the Message channel row with no unsaved email edit, when the tap registers, then she is taken to FEAT-26.SPEC-001, and on returning here the row shows the value FEAT-26.SPEC-001 saved.

**FEAT-06.SPEC-005-AC-16:** Given Riley has an unsaved email edit and taps the Message channel row, when the tap registers, then the "You have unsaved changes. Discard?" dialog appears before any navigation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (consent off, saving, error, offline, unsaved-changes confirmation) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Notification Spec: Access Link Delivery

## Overview

**Name:** Access Link Delivery
**ID:** FEAT-06.SPEC-006
**Type:** Notification
**Purpose:** Sends the requested one-tap on-demand access link by text (with active consent) or email, so the client can view their bookings, with an immediate retry on delivery failure.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Delivering the on-demand access link generated by FEAT-06.SPEC-001, by text or email depending on the client's texting consent
- Delivery-failure and retry behavior for that delivery

**Non-Goals:**
- Delivering booking-specific manage links embedded in confirmations and reminders -- those are delivered as part of FEAT-08's confirmation and reminder content (FEAT-08.SPEC-012 text, FEAT-08.SPEC-013 email); this spec supplies only the on-demand link's own content and delivery rule, per the Cross-Feature Touchpoints entry for this spec
- Choosing the underlying text or email sending capability itself -- that category-level capability contract belongs to FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email); this spec rides those capabilities rather than owning them
- Any marketing or promotional content -- excluded per scope-boundaries SC-15: every Chairtime message, including this one, is limited to its stated transactional purpose

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active texting consent for this Pro (FEAT-14) | The client just asked for a link on their phone; text puts the one-tap link exactly where they are already looking, and matches the product's core "one-tap" promise |
| Email | The client does not have active texting consent for this Pro | XBR-15 (no text is sent without active consent) requires a fallback; email is the client's alternative contact method already on file |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| On-demand access link generated | FEAT-06.SPEC-001 (Access Link Request) | Fires whenever FEAT-06.SPEC-001 successfully matches a phone number to a Client record and generates a new on-demand Access Link | The generated link's address, the matched Client's name and texting-consent state, the Pro's display name |
| Delivery retry requested | FEAT-06.SPEC-001 (Access Link Request) | Fires when the client taps "Try again" after a delivery failure | The same data as the original trigger, plus whether the original link is still valid |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the sole recipient, matched to the Client record the request resolved to (FEAT-06.SPEC-008). No other role receives this notification: the Pro does not use this mechanism and Platform Operator (Support) never sees or bypasses client identity (scope-boundaries SC-05).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (channel selector for this message) | Active / Revoked | As last set by the client | FEAT-06.SPEC-005 (Consent & Email Preferences); revocation itself is set via FEAT-14 |

**Quiet Hours:** N/A -- this message is delivered immediately in direct response to the client's own just-in-time request; the client is, by definition, actively engaged at that moment, so no quiet-hours window applies (unlike FEAT-08's scheduled reminders under XBR-16).

## Content Definition

**Text:**
- **Body:** Tap to view your bookings with {pro_display_name}: {access_link_url}. This link works once and expires in 30 minutes.
- **CTA:** The link itself is the call to action -- deep-links to FEAT-06.SPEC-002 (Access Link Validation & Redemption), which then routes to FEAT-06.SPEC-003.

**Email:**
- **Subject:** Your link to view your bookings with {pro_display_name}
- **Body:**
  Hi {client_name},

  Tap the link below to view your bookings with {pro_display_name}.

  {access_link_url}

  This link works once and expires in 30 minutes. If you didn't request this, you can ignore this email.
- **CTA (button):** View my bookings -- deep-links to FEAT-06.SPEC-002 (Access Link Validation & Redemption), which then routes to FEAT-06.SPEC-003.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- display_name is a required field on the Pro Account |
| {access_link_url} | Access Link -- the generated link address for this record | (a one-time link address) | Never empty -- this notification is triggered only once the link exists |
| {client_name} | Client -- name | Riley | Never empty -- name is a required field on the Client record |

## Delivery Rules

**Batching:** N/A -- each request produces exactly one link and one delivery attempt; there is never more than one pending instance to batch, since a fresh request while a prior link is still valid delivers as its own separate message (FEAT-06.SPEC-001, Edge Cases).
**Deduplication:** At most one delivery attempt per generated Access Link on the original trigger. A "Try again" retry re-sends the same still-valid link rather than generating a new one; if the original link has since expired, the retry is not sent and the client is shown the "request a new link" prompt on FEAT-06.SPEC-001 instead.
**Retry on failure:** Delivery failure is surfaced to the client immediately on FEAT-06.SPEC-001 with a manual "Try again" action (no fixed automatic retry count); each tap of "Try again" is one further delivery attempt, following the same channel-choice rule as the original.
**Expiry:** The message itself does not expire independently of its Access Link: once the link reaches its own 30-minute expiry (FEAT-06.SPEC-007), a delivery that arrives late (or a retry attempted after expiry) is not sent as a live link -- the client is shown the "request a new link" prompt instead, since a message pointing at an already-expired link would be worse than none.

## Edge Cases

- **Client's texting consent is revoked between the request and the moment of sending** -- The channel decision is made at send time, not at trigger time: if consent is no longer active when the message is about to send, it is sent by email instead of text.
- **The generated Access Link expires before the message is delivered (e.g., a delayed send)** -- Per the Expiry rule, an already-expired link is not sent as a live link; the client instead sees the "request a new link" prompt the next time they check FEAT-06.SPEC-001, rather than receiving a message that would fail the moment they tap it.
- **Client has no email on file and texting consent is not active** -- Per the Client entity's field rule, email is required whenever texting is declined, so this combination cannot occur; if it is ever encountered, delivery fails outright and the client sees the standard delivery-failure message on FEAT-06.SPEC-001 with no channel to fall back to.
- **Client taps "Try again" after the underlying link already expired** -- The retry check (Deduplication and Expiry rules) finds the link no longer valid and does not resend it; the client is routed to the "request a new link" prompt instead of receiving a dead link a second time.
- **The Pro account this client's link belongs to is closed or paused between request and delivery** -- Per XBR-14, a paused account keeps existing bookings and client self-service unchanged, so this delivery still proceeds normally; a closed account's client self-service is governed by FEAT-29's closure rules, outside this spec's scope.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Access Link Request) | Triggered by (inbound) | A successful link generation, or a client-initiated retry, fires this notification |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (outbound) | The delivered link's CTA deep-links here |
| FEAT-06.SPEC-007 (Access Link Lifecycle & Scope Rules) | References (inbound) | The 30-minute expiry this spec's Expiry rule depends on |
| FEAT-14 (Messaging Consent Management) | References (inbound) | Governs the texting-consent state this spec's channel choice depends on |
| FEAT-08.SPEC-012 (Text send and delivery) / FEAT-08.SPEC-013 (Email send and delivery) | References (outbound) | The underlying text and email sending capability this message rides, per the Cross-Feature Touchpoints entry for this spec |

## Analytics and Success Signals

- **access_link_delivered** (channel: text / email) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_delivery_retried** (channel: text / email; link_still_valid: yes / no) -- supports success-metrics.md: "Self-Service Access Success"
- **access_link_delivery_channel_fallback** (from: text; to: email) -- N/A -- no Stage 2 metric measures channel fallback specifically; retained so the fallback path stays observable rather than invisible.

## Acceptance Criteria

**FEAT-06.SPEC-006-AC-01:** Given Riley has active texting consent for Talia and requests a link, when FEAT-06.SPEC-001 generates the Access Link, then this spec delivers it by text with the exact body wording defined above.

**FEAT-06.SPEC-006-AC-02:** Given Riley does not have active texting consent for Talia, when her Access Link is generated, then this spec delivers it by email to the address on file, with the exact subject and body wording defined above.

**FEAT-06.SPEC-006-AC-03:** Given Riley's texting consent is revoked between her request and the moment of sending, when the message is about to send, then it is sent by email instead of text.

**FEAT-06.SPEC-006-AC-04:** Given the initial delivery attempt fails, when the failure is detected, then FEAT-06.SPEC-001 shows the delivery-failure message with a "Try again" action.

**FEAT-06.SPEC-006-AC-05:** Given Riley taps "Try again" while her original Access Link is still valid, when the retry is processed, then the same link is re-sent rather than a new one being generated.

**FEAT-06.SPEC-006-AC-06:** Given Riley taps "Try again" after her original Access Link has expired, when the retry is processed, then no message is sent and Riley sees the "request a new link" prompt instead.

**FEAT-06.SPEC-006-AC-07:** Given a generated Access Link is not delivered until after its own 30-minute expiry has passed, when the send is attempted, then it is not sent as a live link and Riley sees the "request a new link" prompt on her next visit to FEAT-06.SPEC-001.

**FEAT-06.SPEC-006-AC-08:** Given Riley's Pro account is paused for a subscription lapse (XBR-14) at the moment of her request, when she requests an access link, then delivery proceeds normally and is unaffected by the pause.

**FEAT-06.SPEC-006-AC-09:** Given Riley's delivered text message contains her one-tap link, when she taps it, then she is taken into FEAT-06.SPEC-002 for validation, with no separate reset or code-entry step.

**FEAT-06.SPEC-006-AC-10:** Given this notification is never batched, when Riley makes two separate requests in succession before the first link expires, then she receives two separate messages, each pointing at its own distinct link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 (original request, retry) | 2 |
| Preference States | 2 (active consent, revoked consent) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Access Link Lifecycle & Scope Rules

## Overview

**Name:** Access Link Lifecycle & Scope Rules
**ID:** FEAT-06.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs the Access Link entity's expiry (30-minute single-use vs. booking-specific until the appointment passes), single-use enforcement, scope, and the per-phone-number request rate limit.
**Parent Feature:** FEAT-06 -- Client Booking Identity
**Governed Entity:** Access Link

## Scope and Non-Goals

**In Scope:**
- Expiry rules for on-demand and booking-specific Access Links
- Single-use enforcement ("first tap wins")
- Scope definition (all bookings with one Pro, vs. one booking)
- The per-phone-number request rate limit enforced at issuance
- Authorization over who may create and redeem an Access Link

**Non-Goals:**
- Phone-to-Client matching and the cross-client/cross-Pro privacy boundary -- owned by FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule); this spec governs the link's own lifecycle, not identity matching
- The actual redemption processing steps (lookup, state write, routing) -- owned by FEAT-06.SPEC-002 (Access Link Validation & Redemption), which applies these rules
- The content and channel of the delivered message -- owned by FEAT-06.SPEC-006 (Access Link Delivery)
- Automatic purge or deletion of expired or used links -- explicitly recorded as a non-goal in feature-overview.md's Non-Goals: Stage 2 states only that links "expire automatically," with no purge window specified; links are retained as an audit trail with no automatic deletion, consistent with scope-boundaries SC-22

## Governed Entity

**Entity:** Access Link
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope | enum | All of this client's bookings with one Pro (on-demand), or one specific Booking (booking-specific) |
| expiry | derived | The timestamp after which the link can no longer be redeemed |
| state | enum | Issued \| Used \| Expired |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Access Link Request | Rate-limit check before creating a new on-demand link; sets scope and initial expiry at creation |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Expiry check, single-use check, and scope resolution at the moment of redemption; writes the resulting Used or Expired state |

## Field Validation Rules

{Access Link has no client-facing input form -- its fields are entirely system-derived at creation and system-managed thereafter. Every field is addressed below.}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope | No validation beyond data type -- system-set at creation to exactly one of the two defined values, never user-entered | Always | -- | -- | -- |
| expiry | Must be computed exactly per the Defaults and Derivations below; never a value the client can set or extend | Always | On creation (set) and on redemption (checked) | -- (this is a system rule, not a user-facing validation) | Yes (redemption is blocked once expiry is reached) |
| state | Must follow the transition path Issued -> Used or Issued -> Expired only; no other transition is valid, and no role can set it directly | Always | On redemption | -- (this is a system rule, not a user-facing validation) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Expiry depends on scope | scope, expiry | An on-demand link's expiry is 30 minutes from creation; a booking-specific link's expiry is the referenced Booking's appointment start time | N/A -- enforced at creation, not user-facing |
| State transition is single-use | state, scope | A link already in state Used can never be redeemed again regardless of scope; the second attempt always resolves as already used (FEAT-06.SPEC-002) | N/A -- surfaced to the client as the shared "request a new link" prompt (FEAT-06.SPEC-001), not a specific error text |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request (create) an on-demand Access Link | The Client (Riley) | Their phone number matches a Client record for this Pro (FEAT-06.SPEC-008), and the per-phone-number request rate limit has not been exceeded | Rate limit exceeded: FEAT-06.SPEC-001 shows a plain message that no more links can be requested for this number right now; no new link is created |
| Request (create) an on-demand Access Link | The Pro (Talia) | Never -- the Pro does not use this mechanism | This action is not shown or reachable from the Pro's own dashboard; the Pro has no path to create a client-facing Access Link |
| Request (create) an on-demand Access Link | Platform Operator (Support) | Never | Support access never uses or bypasses client identity (scope-boundaries SC-05); no route to this action exists for Support |
| Redeem (tap) an Access Link | The Client (Riley) | The link is Issued, unexpired, and scoped to a Client/Booking this same client identity resolves to | Denied: FEAT-06.SPEC-002 routes to the shared "request a new link" prompt on FEAT-06.SPEC-001, never revealing the specific reason |
| Redeem (tap) an Access Link | The Pro (Talia) | Never -- the Pro reaches bookings only through Pro sign-in (FEAT-29), never through a client's Access Link | A Pro who somehow taps a client's link is treated identically to any other client viewer; the link resolves against whatever Client/Booking it is scoped to, never against the Pro's own account |
| Redeem (tap) an Access Link | Platform Operator (Support) | Never -- support never uses or bypasses a client's access link, even for troubleshooting (scope-boundaries SC-05) | Support has no operational path that redeems a client's link; if a link were ever tapped from a support context it would be treated the same as any other viewer, never as a support bypass |
| Directly set an Access Link's state (Used or Expired) | No role -- state transitions are automation-driven only (FEAT-06.SPEC-002) | Always denied to every role | There is no control, for any role, that writes this field directly; all transitions happen inside FEAT-06.SPEC-002's processing |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Issued | On creation | No |
| expiry (on-demand scope) | Creation timestamp + 30 minutes | On creation | No |
| expiry (booking-specific scope) | The referenced Booking's appointment start_time | On creation (by FEAT-08 or FEAT-30, which create booking-specific links) | No |
| scope | "All bookings with this Pro" for links created by FEAT-06.SPEC-001; "one Booking" for links created by FEAT-08 or FEAT-30 | On creation | No |

## Business Rules

- XBR-18: Client identity is a phone number with one Pro only; access links open only that client's bookings with that Pro; on-demand links are single-use for 30 minutes; booking-specific links stop working once the appointment passes; a Pro reschedule issues a fresh manage link.
- Per-phone-number request rate limit: at most 5 on-demand Access Link requests per phone number within any rolling 60-minute window, enforced by FEAT-06.SPEC-001 before a new link is created. This limit exists to prevent message flooding (product-features.md, Validation & Limits) and applies regardless of whether prior requests succeeded, failed to match, or were themselves rate-limited.
- The rate limit counts requests, not delivered messages -- a request that finds no matching Client record (FEAT-06.SPEC-008) still counts toward the limit, so the limit cannot be used to probe which phone numbers exist.
- A link's expiry, once set at creation, is never extended or renewed by a later action (including a delivery retry, per FEAT-06.SPEC-006) -- only a brand-new request creates a brand-new expiry.
- Expired and Used links are retained indefinitely as an audit trail of issuance and use; scope-boundaries SC-22 governs history retention, and no purge policy exists for this entity.

## Edge Cases

- **Rate limit counted exactly at the boundary (the 5th request within the window)** -- The 5th request within the rolling 60-minute window succeeds normally; the 6th request within that same window is rejected with the rate-limited message.
- **Rate limit window rolls forward mid-use** -- The window is rolling, not fixed to the clock hour: a request made 61 minutes after an earlier one no longer counts that earlier request against the limit, even if requests in between still do.
- **On-demand link redeemed at exactly 30 minutes and 0 seconds after creation** -- Treated as expired; the 30-minute window is inclusive of everything strictly before the boundary, not at or after it.
- **Booking-specific link redeemed at the exact appointment start_time** -- Treated as expired; "until the appointment passes" means strictly before start_time, not at or after it.
- **A booking-specific link's Booking is rescheduled to a new time before the link is tapped** -- Per XBR-18, the reschedule issues a fresh manage link with a new expiry tied to the new start_time; the old link's own expiry is not recalculated -- it is instead superseded, and FEAT-06.SPEC-002 resolves a tap on the old link as a scope resolution failure rather than checking it against the new time.
- **Client requests a new on-demand link while a previously issued one is still Issued and unexpired** -- Both links independently follow this spec's rules; the earlier link remains redeemable until it is used or reaches its own 30-minute expiry, and creating the new one does not expire or invalidate the earlier one.

## Acceptance Criteria

**FEAT-06.SPEC-007-AC-01:** Given Riley requests her first on-demand access link this hour, when FEAT-06.SPEC-001 checks the rate limit, then the request is allowed and a new Access Link is created with state Issued.

**FEAT-06.SPEC-007-AC-02:** Given Riley has already made 5 access-link requests for her phone number within the current rolling 60-minute window, when she requests a 6th, then it is rejected with the rate-limited message and no new link is created.

**FEAT-06.SPEC-007-AC-03:** Given Riley's 6th request was rejected at minute 10 of the window, when 61 minutes have passed since her 1st request, then a new request from her no longer counts that 1st request against the limit.

**FEAT-06.SPEC-007-AC-04:** Given an on-demand Access Link was created at 2:00pm, when Riley taps it at 2:29pm, then it is treated as unexpired and redemption proceeds.

**FEAT-06.SPEC-007-AC-05:** Given an on-demand Access Link was created at 2:00pm, when Riley taps it at exactly 2:30pm or later, then it is treated as expired.

**FEAT-06.SPEC-007-AC-06:** Given a booking-specific Access Link scoped to an appointment starting at 4:00pm, when Riley taps it at 3:59pm, then it is treated as unexpired.

**FEAT-06.SPEC-007-AC-07:** Given a booking-specific Access Link scoped to an appointment starting at 4:00pm, when Riley taps it at exactly 4:00pm or later, then it is treated as expired.

**FEAT-06.SPEC-007-AC-08:** Given Talia reschedules the booking a booking-specific link points to, when FEAT-30 issues a fresh manage link for the new time, then the old link's redemption is resolved as a scope resolution failure rather than being checked against the new appointment time.

**FEAT-06.SPEC-007-AC-09:** Given a prior on-demand link Riley received is still Issued and unexpired, when she requests a new one, then the new link is created independently and the prior link remains separately redeemable until it is used or expires on its own.

**FEAT-06.SPEC-007-AC-10:** Given the Pro (Talia) has no path in her own dashboard to request or redeem a client's Access Link, when she looks for one, then no such control exists -- this mechanism is not the Pro's sign-in path.

**FEAT-06.SPEC-007-AC-11:** Given Platform Operator (Support) is assisting a Pro with a help request, when Support attempts to use or bypass a client's Access Link, then no operational path exists for doing so.

**FEAT-06.SPEC-007-AC-12:** Given no role or control can set an Access Link's state directly, when a link's state changes, then it always changes only as a result of FEAT-06.SPEC-002's redemption processing.

**FEAT-06.SPEC-007-AC-13:** Given Riley's request finds no matching Client record for this Pro, when the rate limit is evaluated for her next request within the same window, then the earlier, unmatched request still counts toward the limit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Client Identity & Privacy Isolation Rule

## Overview

**Name:** Client Identity & Privacy Isolation Rule
**ID:** FEAT-06.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs phone-to-Client matching scoped to one Pro, and the hard boundary that no client can ever see another phone number's or Pro's bookings.
**Parent Feature:** FEAT-06 -- Client Booking Identity
**Governed Entity:** Client (partial -- this feature reads phone and email, and updates email; it never creates or deletes Client records)

## Scope and Non-Goals

**In Scope:**
- The exact matching logic that resolves a phone number to a Client record, scoped to one Pro
- The privacy isolation boundary that prevents any cross-client or cross-Pro visibility
- Authorization over which role may perform phone-to-Client matching and what happens when matching fails or succeeds
- The "no hint" behavior when a phone number does not match

**Non-Goals:**
- Creating or deleting Client records -- owned by FEAT-05 (first booking) and FEAT-30 (Pro books a client in) for creation, and FEAT-13 for deletion; this spec governs only the read-time matching this feature performs
- Editing the client's name, phone number, or private notes -- phone is a read-only identity key to this feature, and name/notes remain Pro-editable only via FEAT-13
- Access Link expiry, single-use, and rate-limit rules -- owned by FEAT-06.SPEC-007; this spec governs identity matching, not link lifecycle
- Any cross-pro client profile or shared identity across Pros -- excluded per scope-boundaries SC-04: a phone number's Client record is scoped to exactly one Pro; booking with a second Pro always creates a separate, unconnected record

## Governed Entity

**Entity:** Client (partial)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Required, 1-100 characters; read-only to this feature |
| phone | text | Required, valid reachable format; identity key within one Pro; read-only to this feature -- this is the field this spec's matching logic operates on |
| email | text | Required when texts are declined, otherwise optional; the one field this feature writes (via FEAT-06.SPEC-005) |
| private_note | text | Pro-only, up to 1,000 characters; never read or written by this feature |
| booking_notes | text | The client's optional note per booking; never read or written by this feature |
| booking_history | derived | Derived list of this client's Bookings with this Pro only; read (not written) by FEAT-06.SPEC-003 and FEAT-06.SPEC-004, scoped by this spec's matching rule |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Access Link Request | Phone-to-Client matching at the moment a link is requested |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Scope resolution against the matched Client/Booking at the moment a link is redeemed |
| FEAT-06.SPEC-005 | Consent & Email Preferences | Matching used to identify which Client record's email and consent are being read and written |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| phone | Must exactly match an existing Client record's phone field, scoped to the one Pro in context | Always, whenever this feature performs a match | On link request (FEAT-06.SPEC-001) and on redemption (FEAT-06.SPEC-002) | No error is ever shown for "not found" -- see Business Rules for the no-hint pattern | Yes (a non-match blocks any further access) |
| name, private_note, booking_notes | No validation beyond data type -- these fields are never read or written by this feature | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Match is Pro-scoped | phone, Pro Account (context) | The same phone number may match different Client records under different Pros; a match is valid only within the one Pro the request or link is already scoped to | N/A -- a match under a different Pro is treated identically to no match at all |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Match a phone number to a Client record | The Client (Riley) | The entered or link-carried phone number exactly matches a Client record for the one Pro in context | No match: FEAT-06.SPEC-001 shows the plain "no bookings found" message with no hint about whether the number exists elsewhere; FEAT-06.SPEC-002 routes to the shared "request a new link" prompt |
| Match a phone number to a Client record | The Pro (Talia) | Never through this feature -- the Pro's own client lookup is FEAT-13 (Client Record Management), not this mechanism | Not applicable here -- this action is not exposed to the Pro through this feature at all |
| Match a phone number to a Client record | Platform Operator (Support) | Never | Support access never uses or bypasses client identity (scope-boundaries SC-05); no route to this action exists for Support |
| View another client's bookings or another Pro's data through this feature | No role, ever | Never -- this is a hard boundary, not a conditional one | There is no control, message, or path that exposes this; a client can never view another phone number's bookings even if they guess or mistype one (product-features.md, Validation & Limits) |
| Read the matched Client's email and consent state | The Client (Riley) | Only for the Client record their own phone number matched | Not applicable to any other role -- see the "view another client's bookings" row above |
| Update the matched Client's email | The Client (Riley) | Only for the Client record their own phone number matched (via FEAT-06.SPEC-005) | Not applicable to any other role through this feature |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Matched Client identity (session-scoped) | Derived once, at the moment a phone number is matched (FEAT-06.SPEC-001) or an Access Link is redeemed (FEAT-06.SPEC-002); held for the remainder of that viewing session | On match / on redemption | No -- the client cannot switch which Client identity a session resolves to mid-session; a different phone number requires a fresh request |

## Business Rules

- SC-03: cross-pro or cross-client visibility does not exist in this product; there is no shared or comparative view across accounts of any kind.
- SC-04: a phone number's Client record is scoped to exactly one Pro; a person booking with a second Pro has two unconnected records, never a merged or shared identity.
- The "no hint" pattern: a phone number with no matching Client record for this Pro produces the identical experience as a phone number that matches a different Pro entirely, or a mistyped number -- the client is never told which case applies (shared with FEAT-06.SPEC-002's identical redemption-failure pattern).
- XBR-18: access links open only that client's bookings with that Pro -- the matching performed here is what makes that boundary enforceable at every read this feature performs.
- Client deletion (owned by FEAT-13, XBR-19) removes the matched record entirely; a later booking under the same phone number creates a new, unconnected Client record rather than resurrecting the old one, so a match performed after such a deletion and rebooking always resolves to the new record only.

## Edge Cases

- **Client enters a phone number with different formatting than what is on file (e.g., extra spaces or a leading country code)** -- The match is performed on the normalized, reachable-format value already stored on the Client record; formatting differences that resolve to the same underlying number still match. A number that resolves to a genuinely different value does not match.
- **Two Client records under different Pros share the same phone number** -- Each match is scoped strictly to the one Pro already in context (from the booking page or link the client is using); the other Pro's record is never considered, checked, or referenced in any way, and its existence is never revealed.
- **Client's phone number was changed by the Pro (FEAT-13) after a link was issued but before it is redeemed** -- Per XBR-18, the phone change invalidates the client's existing access links; a redemption attempt against the old scope no longer resolves to a valid match and is treated as a scope resolution failure (FEAT-06.SPEC-002).
- **Client's Client record was deleted (FEAT-13, XBR-19) after a link was issued but before it is redeemed** -- The match can no longer resolve; redemption fails as a scope resolution failure, with the same no-hint experience as any other failed match.
- **A malicious actor tries many phone numbers in sequence to discover which ones exist for a Pro** -- Every non-match returns the identical "no bookings found" message with no timing, wording, or behavioral difference from a match that then finds zero bookings would show if such a state were possible; the per-phone-number rate limit (FEAT-06.SPEC-007) also bounds how many attempts a single number can drive in a given window.
- **Client rebooks with the same Pro under the same phone number after their prior Client record was deleted** -- FEAT-05 or FEAT-30 creates a new Client record (per FEAT-13's XBR-19); this spec's matching then resolves the phone number to that new record only, with no continuity to the deleted one's history.

## Acceptance Criteria

**FEAT-06.SPEC-008-AC-01:** Given Riley enters the phone number on file for her Client record with Talia, when FEAT-06.SPEC-001 matches it, then it resolves to her Client record for Talia and access proceeds.

**FEAT-06.SPEC-008-AC-02:** Given Riley enters a phone number with no matching Client record for Talia, when the match is attempted, then FEAT-06.SPEC-001 shows the plain "no bookings found" message with no hint about whether the number exists elsewhere.

**FEAT-06.SPEC-008-AC-03:** Given a phone number matches a Client record under a different Pro but not under Talia, when Riley enters it on Talia's request screen, then it is treated identically to no match at all.

**FEAT-06.SPEC-008-AC-04:** Given Riley's Client record with Talia was deleted and she later rebooks with the same phone number, when she next requests an access link, then it matches only her new Client record, with no visibility into the deleted record's history.

**FEAT-06.SPEC-008-AC-05:** Given Talia changed Riley's phone number in her client record (FEAT-13) after issuing Riley a link, when Riley taps that old link, then it fails to resolve and is treated as a scope resolution failure.

**FEAT-06.SPEC-008-AC-06:** Given Talia looks for a way to match a phone number to a client record through this feature's own screens, when she looks, then no such control exists -- her own client lookup is FEAT-13, not this mechanism.

**FEAT-06.SPEC-008-AC-07:** Given Platform Operator (Support) is assisting with a help request, when Support attempts to match a client's phone number through this feature, then no operational path exists for doing so.

**FEAT-06.SPEC-008-AC-08:** Given Riley's phone number has matched her Client record with Talia, when FEAT-06.SPEC-003 loads her bookings, then only bookings belonging to that matched Client record with Talia are shown -- never another client's or another Pro's bookings.

**FEAT-06.SPEC-008-AC-09:** Given a malicious actor submits many different phone numbers in sequence, when each is checked, then every non-match returns the identical message with no distinguishing detail, and the per-phone-number rate limit (FEAT-06.SPEC-007) bounds the attempts a single number can drive.

**FEAT-06.SPEC-008-AC-10:** Given Riley's phone number is entered with extra spaces compared to what is stored, when the match normalizes both values to the same reachable format, then it still resolves to her Client record.

**FEAT-06.SPEC-008-AC-11:** Given Riley's matched Client identity is established for a viewing session (FEAT-06.SPEC-001 or FEAT-06.SPEC-002), when she views FEAT-06.SPEC-005, then only her own email and consent state for Talia are shown and editable.

**FEAT-06.SPEC-008-AC-12:** Given two different phone numbers each have a Client record with Talia, when one client's phone number is entered, when a match is performed, then it resolves only to that one client's own record, never exposing the other's.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |


---
document_type: feature-overview
feature_number: FEAT-08
feature_name: Automated Booking Messaging
feature_slug: automated-booking-messaging
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 13
screen_count: 1
automation_count: 4
logic_rule_count: 1
integration_count: 2
notification_count: 5
---

# Feature Breakdown Brief: Automated Booking Messaging

## Summary

**Feature:** Automated Booking Messaging
**ID:** FEAT-08
**Description:** The client receives an immediate confirmation the moment a booking is paid, and an automatic reminder before the appointment with a one-tap "I'll be there / I need to reschedule" response — replacing the Pro's habit of texting reminders by hand.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Vision states this exactly: "a confirmation text lands immediately... a reminder arrives with a one-tap 'I'll be there / I need to reschedule.'" This directly replaces the founder's stated evening admin burden and is central to the "pro never chases" success criterion. Automated client reminders are a baseline expectation across the profiled market, not a differentiator, so correctness and reliability of delivery matter more than novelty here.

**Key Capabilities:**
- Send an immediate confirmation message on successful booking
- Send an automatic reminder a set time before the appointment (two days, per BRIEF.md's example)
- Offer a one-tap "I'll be there" or "I need to reschedule" response from the reminder itself
- Include in every confirmation the service, date and time (with timezone), deposit paid, balance due in person, the studio location, the cancellation cut-off, a manage link, and an "add to my calendar" option
- Tell the client when their booking is cancelled, rescheduled or refunded by either party, including what happened to the deposit
- Notify the Pro of new bookings, client cancellations and reschedules, and anything needing attention (message delivery failure, calendar reconnection, refund failure, card-issuer dispute), in-app and — per the Pro's preferences in FEAT-27 — by text or email

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-08.SPEC-001 | Booking Confirmation Message | Notification | The Client, Platform Operator (Support) | Sends the immediate post-payment confirmation with service, date/time and timezone, deposit paid, balance due, studio location, cancellation cut-off, manage link and add-to-calendar option |
| FEAT-08.SPEC-002 | Appointment Reminder Message | Notification | The Client, Platform Operator (Support) | Sends the pre-appointment reminder carrying the one-tap "I'll be there" / "I need to reschedule" options |
| FEAT-08.SPEC-003 | Reminder Reply Acknowledgment | Screen | The Client | The page a client lands on after tapping "I'll be there," confirming the acknowledgment was recorded |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | Notification | The Client, Platform Operator (Support) | Tells the client when their booking is cancelled, rescheduled or refunded by either party, including deposit outcome and refund-in-progress status |
| FEAT-08.SPEC-005 | Pro Booking Activity Notification | Notification | The Pro, Platform Operator (Support) | Notifies the Pro of new bookings and client-initiated cancellations/reschedules, per the Pro's notification preferences |
| FEAT-08.SPEC-006 | Pro Attention Alert | Notification | The Pro, Platform Operator (Support) | Notifies the Pro of anything needing attention — message delivery failure, calendar reconnection, refund failure, card-issuer dispute |
| FEAT-08.SPEC-007 | Reminder Scheduling & Timing Window Enforcement | Automation | The Pro, The Client | Computes the reminder send time (two days before appointment by default), enforces the 8am–9pm daytime-hours window, and suppresses a separate reminder for a booking made after its reminder point |
| FEAT-08.SPEC-008 | Reminder Reply Routing | Automation | The Client, The Pro | Handles the one-tap reply: records "I'll be there" as an acknowledgment or routes "I need to reschedule" into Client-Initiated Cancel/Reschedule (FEAT-10) |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | Automation | The Client, The Pro, Platform Operator (Support) | On a failed text, retries once, then falls back to email, and flags the delivery gap on the Pro's dashboard — never silently dropped |
| FEAT-08.SPEC-010 | Booking-Specific Manage Link Issuance | Automation | The Client | Creates the booking-specific manage link carried in every confirmation and reminder, and reissues a fresh one after a Pro-initiated reschedule |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Logic/Rule | The Client, The Pro | Decides text-vs-email for every outbound client message based on active Messaging Consent, honoring a revoke on the very next message |
| FEAT-08.SPEC-012 | Transactional Text Messaging Capability | Integration | The Client, The Pro | External capability that sends text messages and reports back delivery status (Queued/Sent/Delivered/Failed) for every text this feature and other features send |
| FEAT-08.SPEC-013 | Transactional Email Capability | Integration | The Client, The Pro | External capability that sends email messages (fallback channel and consent-declined clients) and reports back delivery status |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Send an immediate confirmation message on successful booking | FEAT-08.SPEC-001, FEAT-08.SPEC-007, FEAT-08.SPEC-012, FEAT-08.SPEC-013 | SPEC-001 is the message itself; SPEC-007 enforces the "within about a minute" timing; SPEC-012/013 perform the actual send | Phase 2 (Explicit) |
| Send an automatic reminder a set time before the appointment | FEAT-08.SPEC-002, FEAT-08.SPEC-007 | SPEC-002 is the reminder message; SPEC-007 computes when it fires and enforces the daytime-hours window and the late-booking exception | Phase 2 (Explicit) |
| Offer a one-tap "I'll be there" or "I need to reschedule" response | FEAT-08.SPEC-002, FEAT-08.SPEC-003, FEAT-08.SPEC-008 | SPEC-002 embeds the two tap options; SPEC-008 processes whichever is tapped; SPEC-003 is the landing page for the acknowledgment path | Phase 2 (Explicit) |
| Include full confirmation content (service, date/time+timezone, deposit paid, balance due, studio location, cancellation cut-off, manage link, add-to-calendar) | FEAT-08.SPEC-001, FEAT-08.SPEC-010 | SPEC-001 assembles and sends the content; SPEC-010 supplies the booking-specific manage link it embeds | Phase 2 (Explicit) |
| Tell the client when their booking is cancelled, rescheduled or refunded by either party | FEAT-08.SPEC-004 | Single Notification spec covers all three change types and both actors (client- and Pro-initiated), including deposit/refund outcome | Phase 2 (Explicit) |
| Notify the Pro of new bookings, client cancellations/reschedules, and anything needing attention | FEAT-08.SPEC-005, FEAT-08.SPEC-006 | SPEC-005 covers routine booking-activity notifications; SPEC-006 covers the four named attention-worthy conditions, both respecting FEAT-27's notification preferences | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-08.SPEC-007 | Reminder Scheduling & Timing Window Enforcement | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field states three distinct conditional rules (default 2-day timing, 8am–9pm window with nearest-allowed-time fallback, no separate reminder for a late booking) that apply across every reminder send — a shared, conditional rule set crossing the standalone-Logic/Rule and Automation threshold; modeled as an Automation because it drives a time-based trigger, not a static validation |
| FEAT-08.SPEC-008 | Reminder Reply Routing | Phase 4 (Trigger-Response) | The Primary Flows field states the "I'll be there" tap "simply acknowledges" while "I need to reschedule" routes into FEAT-10 — two different cross-feature/cross-entity consequences of the same tap, which is processing logic, not a bare inline interaction |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | Phase 4 (Trigger-Response) + Phase 6 (Negative/Failure) | The Alternate flow ("a text fails to deliver; the system retries once and... falls back to email and flags the delivery gap") and XBR-17 (owned by this feature) both describe multi-step failure handling with cross-feature effects (flags FEAT-12's dashboard, recorded in FEAT-16's timeline) |
| FEAT-08.SPEC-010 | Booking-Specific Manage Link Issuance | Phase 3 (Entity-Lifecycle Analysis) | The Access Link entity's Create lifecycle names FEAT-08 as a creator ("booking-specific manage links in confirmations and reminders; fresh link after a FEAT-30 reschedule") with no existing spec covering that creation — a CRUD-matrix gap |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Phase 5 (Rule-Constraint Discovery) | XBR-15 (no text without consent; email otherwise; revoke honored on the very next message; changed phone number needs fresh consent) is a conditional rule shared across every Notification spec in this feature — crosses the shared-across-specs threshold for a standalone Logic/Rule |
| FEAT-08.SPEC-003 | Reminder Reply Acknowledgment | Phase 6 (Negative/Failure) applied to SPEC-008's tap outcome | A one-tap acknowledgment still needs somewhere to land and a defined Error/Permission-Denied state (expired booking-specific link) — this is a genuine screen, not a bare toast, because the link can be expired, already used from another device, or belong to a passed appointment |
| FEAT-08.SPEC-012, FEAT-08.SPEC-013 | Transactional Text Messaging Capability / Transactional Email Capability | Phase 4 (External Dependencies lens) | ASMP-32 names both category-level capabilities as dependencies of this feature, and the External Touchpoints table lists FEAT-08 first among the features that rely on each — this feature owns the Integration specs that FEAT-06, FEAT-14, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29 and FEAT-30 send their own messages through |

## Entity-Lifecycle Coverage Matrix

**Entity: Message**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006 | Each outbound notification creates one Message record (type, channel, recipient, content summary, send time, initial Queued status) at the moment it is dispatched | FEAT-26 (Later) and access-link/deposit-request messages on behalf of FEAT-06/FEAT-30 also create Message records, but on those features' triggers, not this one's |
| Read (single) | N/A -- owned by FEAT-16 | Message display (delivery status per send) is the Booking & Payment Activity Record's responsibility, per this feature's Data Notes ("Displayed: delivery status to the Pro (via FEAT-16)") | Not a gap: this feature writes, FEAT-16 and FEAT-12 read |
| Read (list) | N/A -- owned by FEAT-12, FEAT-16 | Same reasoning as Read (single); FEAT-12 also reads delivery-failure flags for the dashboard's attention list | Not a gap |
| Update | FEAT-08.SPEC-009, FEAT-08.SPEC-012, FEAT-08.SPEC-013 | delivery_status only, per the Message entity's own lifecycle note; the retry/fallback Automation and the two Integration specs' inbound delivery events drive the transition | -- |
| Delete/Archive | N/A -- immutable once sent | No soft or hard delete applies. Once a Message is sent it is never removed or edited; this is an explicit, sourced design decision (Message entity lifecycle: "Deleted: N/A — immutable once sent"), not an omission -- messages form part of the append-only activity record FEAT-16 relies on for dispute evidence | No restore/cascade/retention question arises because nothing is ever deleted |
| State Transition | FEAT-08.SPEC-009, FEAT-08.SPEC-012, FEAT-08.SPEC-013 | Queued -> Sent -> Delivered \| Failed; a Failed text re-enters via SPEC-009's retry-then-fallback path, which creates a second Message record on the fallback channel rather than mutating the failed one | Keeps each channel attempt as its own immutable record, consistent with the no-delete rule above |

**Entity: Access Link** *(booking-specific scope only -- the on-demand "my bookings" link is FEAT-06's)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-08.SPEC-010 | Issued at confirmation send and at reminder send, scoped to that one booking; a fresh link is issued after a Pro-initiated reschedule (FEAT-30), per XBR-18 | -- |
| Read (single) | N/A -- owned by FEAT-06 | Resolving/redeeming a tapped link (checking Issued/Used/Expired and scoping access) is Client Booking Identity's responsibility; this feature only mints the link | Not a gap -- confirmed against the dependency map's "Read by FEAT-10" and FEAT-06's ownership of client access |
| Read (list) | N/A -- no list exists | A booking-specific Access Link is a single bearer link with no list surface anywhere in the product | -- |
| Update | N/A -- owned by FEAT-06 | State changes (Issued -> Used -> Expired) are written by FEAT-06 when a link is redeemed, not by this feature | Not a gap |
| Delete/Archive | N/A -- expires automatically | No delete/archive action exists: a booking-specific link simply stops working once the appointment passes (a time-based expiry baked into its own expiry field), which is an explicit design decision from the Access Link entity's Lifecycle note, not an omission | No restore path or retention question -- an expired link is never revived; a fresh one is issued instead |
| State Transition | N/A -- owned by FEAT-06 | Issued -> Used -> Expired transitions are FEAT-06's write, triggered when the Client taps the link this feature created | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-08.SPEC-001, SPEC-002, SPEC-004, SPEC-005, SPEC-006, SPEC-007, SPEC-008, SPEC-010 | Reads appointment time, service, pricing, deposit/balance figures, state, and policy details to compose every message and to compute reminder timing |
| Messaging Consent | FEAT-08.SPEC-011 | Read before every client-directed send to decide text vs. email, per XBR-15 |
| Pro Account | FEAT-08.SPEC-001, SPEC-005, SPEC-006, SPEC-007, SPEC-011 | Reads studio_address (confirmations only), timezone (reminder timing, timestamp display), and notification_preferences (Pro notification channel routing) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A booking's deposit payment completes (FEAT-07) | Compose and send the confirmation within about a minute | Standalone Notification | FEAT-08.SPEC-001 |
| A confirmed booking reaches its reminder point (two days before by default) | Compose and send the reminder with one-tap options, inside the 8am–9pm window | Standalone Notification, timing owned by a paired Automation | FEAT-08.SPEC-002 / FEAT-08.SPEC-007 |
| A booking is made after its own reminder point would have fired | Suppress the separate reminder; the confirmation already sent serves instead | Standalone Automation | FEAT-08.SPEC-007 |
| Client taps "I'll be there" in a reminder | Record the acknowledgment on the Booking (attendance_reply) and show a landing confirmation | Standalone Automation + Screen | FEAT-08.SPEC-008 / FEAT-08.SPEC-003 |
| Client taps "I need to reschedule" in a reminder | Route into Client-Initiated Cancel/Reschedule (FEAT-10) via the booking-specific manage link | Standalone Automation (belongs to FEAT-10 from that point on) | FEAT-08.SPEC-008 |
| Booking is cancelled, rescheduled, or refunded by either the Client (FEAT-10) or the Pro (FEAT-30) | Tell the client what happened and the deposit outcome (including "refund in progress") | Standalone Notification | FEAT-08.SPEC-004 |
| New booking is made, or a client cancels/reschedules their own booking | Notify the Pro, per notification_preferences (in-app, text, and/or email) | Standalone Notification | FEAT-08.SPEC-005 |
| A text delivery fails, a calendar connection needs reconnecting (FEAT-04), a refund fails to complete (FEAT-09/FEAT-30), or a card-issuer dispute is opened (FEAT-16) | Notify the Pro that something needs attention, per notification_preferences | Standalone Notification | FEAT-08.SPEC-006 |
| A text send fails | Retry once; on continued failure, fall back to email and flag the delivery gap on the Pro's dashboard | Standalone Automation | FEAT-08.SPEC-009 |
| A confirmation or reminder is about to be sent | Mint a booking-specific manage link scoped to that one booking | Standalone Automation | FEAT-08.SPEC-010 |
| A Pro-initiated reschedule occurs (FEAT-30) | Issue a fresh manage link for the changed booking | Standalone Automation | FEAT-08.SPEC-010 |
| Any client-directed message is about to send | Check active Messaging Consent; choose text if granted, email otherwise; re-check on every send so a same-session revoke is honored immediately | Standalone Logic/Rule | FEAT-08.SPEC-011 |
| A client's phone number changes | Require fresh consent before texting the new number again | Standalone Logic/Rule (delegates the consent-state change itself to FEAT-14) | FEAT-08.SPEC-011 |
| Any spec in this feature (or FEAT-06, FEAT-14, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30) needs to send a text | Perform the external text send and report back delivery status | Standalone Integration | FEAT-08.SPEC-012 |
| Any spec needs to send a fallback or consent-declined email | Perform the external email send and report back delivery status | Standalone Integration | FEAT-08.SPEC-013 |
| A person forwards a confirmation or reminder to someone else | The forwarded manage link opens only that single booking and nothing beyond it, and stops working once the appointment has passed | Standalone Logic/Rule (inline authorization check within the manage-link flow, enforced by FEAT-06's link resolution using the scope FEAT-08.SPEC-010 set) | FEAT-08.SPEC-010 / FEAT-08.SPEC-011 |

## Shared Context

**Shared Entities:**
- Message -- created by SPEC-001, SPEC-002, SPEC-004, SPEC-005, SPEC-006; updated (delivery_status only) by SPEC-009, SPEC-012, SPEC-013; read externally by FEAT-12 and FEAT-16. Fields: type, channel, content_summary, recipient, send time, delivery_status.
- Access Link (booking-specific scope) -- created by SPEC-010; read/updated externally by FEAT-06. Fields: scope (one booking), expiry (until appointment passes), state (Issued/Used/Expired).
- Booking (read-only) -- every Notification and the reminder-timing Automation read appointment_time, service, price_agreed, deposit_amount, balance_due, state, and attendance_reply.
- Messaging Consent (read-only) -- SPEC-011 reads channel, state, phone_number before every client-directed send.
- Pro Account (read-only) -- SPEC-001/005/006/007/011 read studio_address, timezone, and notification_preferences.

**Shared UI Patterns:**
- One-tap reply vocabulary ("I'll be there" / "I need to reschedule") -- used identically in SPEC-002's reminder content and honored identically whether the client received it by text or email, per the Validation & Limits field ("the one-tap replies are link taps, so a reply works the same by text or email").
- Plain-language delivery-gap framing -- SPEC-004's "refund in progress" wording and SPEC-006's Pro-facing attention-alert wording both describe an in-flight, not-yet-complete condition the same way, so neither party is left wondering whether something silently failed (XBR-10, XBR-17).

**Shared Validation:**
- FEAT-08.SPEC-011 owns the text-vs-email channel decision (XBR-15) that every other Notification spec in this feature defers to before sending.
- FEAT-08.SPEC-007 owns the reminder-timing rules (default two days, 8am–9pm window, late-booking suppression) that SPEC-002 defers to for when it fires.

## Internal Dependency Map

```
FEAT-07 (Deposit Payment at Booking) -> [payment completes] -> SPEC-001 (Booking Confirmation Message)
SPEC-001 (Booking Confirmation Message) -> [needs a link] -> SPEC-010 (Booking-Specific Manage Link Issuance)
SPEC-001 (Booking Confirmation Message) -> [needs a channel] -> SPEC-011 (Messaging Consent & Channel Selection Rule) -> SPEC-012 (Text) / SPEC-013 (Email)
SPEC-007 (Reminder Scheduling & Timing Window Enforcement) -> [reminder point reached] -> SPEC-002 (Appointment Reminder Message)
SPEC-002 (Appointment Reminder Message) -> [needs a link] -> SPEC-010 (Booking-Specific Manage Link Issuance)
SPEC-002 (Appointment Reminder Message) -> [needs a channel] -> SPEC-011 (Messaging Consent & Channel Selection Rule) -> SPEC-012 / SPEC-013
SPEC-002 (Appointment Reminder Message) -> [client taps a reply] -> SPEC-008 (Reminder Reply Routing)
SPEC-008 (Reminder Reply Routing) -> ["I'll be there"] -> SPEC-003 (Reminder Reply Acknowledgment)
SPEC-008 (Reminder Reply Routing) -> ["I need to reschedule"] -> FEAT-10 (Client-Initiated Cancel/Reschedule)
FEAT-10 / FEAT-30 -> [cancel, reschedule, or refund event] -> SPEC-004 (Booking Change & Refund Notice)
FEAT-05 / FEAT-10 -> [new booking or client cancel/reschedule] -> SPEC-005 (Pro Booking Activity Notification)
FEAT-04 / FEAT-09 / FEAT-30 / FEAT-16 -> [delivery failure, reconnection, refund failure, dispute] -> SPEC-006 (Pro Attention Alert)
SPEC-012 (Text) / SPEC-013 (Email) -> [send fails] -> SPEC-009 (Message Delivery Retry & Fallback) -> SPEC-013 (Email) [on continued failure] and SPEC-006 (Pro Attention Alert)
FEAT-30 -> [Pro reschedules] -> SPEC-010 (Booking-Specific Manage Link Issuance) [fresh link]
```

**Default Entry:** N/A -- this feature has no navigable screen of its own except SPEC-003, which a client only ever reaches by tapping a reminder link; there is no dashboard or menu entry point into "Automated Booking Messaging" for either role.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-08.SPEC-001 | Inbound | FEAT-07 (Deposit Payment at Booking) | Reads the completed payment to trigger the confirmation | Deposit payment completes |
| FEAT-08.SPEC-001 / SPEC-002 / SPEC-007 | Inbound | FEAT-27 (Pro Profile & Booking Page Settings) | Reads studio location and timezone used in message content and reminder timing | Every send |
| FEAT-08.SPEC-003 | Outbound | FEAT-06 (Client Booking Identity) | The manage link a client taps resolves through FEAT-06's link handling | Client taps the manage link in a confirmation or reminder |
| FEAT-08.SPEC-008 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | "I need to reschedule" hands off directly into the reschedule flow for that booking | Client taps "I need to reschedule" |
| FEAT-08.SPEC-008 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | "I'll be there" status feeds the dashboard's per-booking display | Client taps "I'll be there" |
| FEAT-08.SPEC-004 | Inbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | Reads a client-initiated cancel/reschedule to notify the client of the resulting change | Client cancels or reschedules |
| FEAT-08.SPEC-004 / SPEC-005 / SPEC-006 | Inbound | FEAT-30 (Pro Booking Management) | Reads Pro-initiated cancels, reschedules, and refunds to notify the client, and reads Pro-side refund failures to alert the Pro | Pro acts on a booking |
| FEAT-08.SPEC-004 / SPEC-006 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine, via automatic refund triggering) | Reads automatic refund outcomes and refund-in-progress states to notify the client and alert the Pro on failure | Automatic refund succeeds, is in progress, or fails |
| FEAT-08.SPEC-006 | Inbound | FEAT-04 (Two-Way Calendar Sync) | Reads a lapsed connection to alert the Pro | Calendar connection needs reconnecting |
| FEAT-08.SPEC-006 | Inbound | FEAT-16 (Booking & Payment Activity Record) | Reads a card-issuer dispute notice to alert the Pro | A dispute is opened |
| FEAT-08.SPEC-009 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A delivery gap is flagged on the Pro's dashboard | A text fails and the email fallback is used |
| FEAT-08.SPEC-001 / SPEC-002 / SPEC-004 / SPEC-005 / SPEC-006 / SPEC-009 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Every send and every delivery-failure/fallback event is written to the append-only activity record | Every message sent or retried |
| FEAT-08.SPEC-011 | Inbound | FEAT-14 (Messaging Consent Management) | Reads active Messaging Consent state before every client-directed send | Every client-directed send |
| FEAT-08.SPEC-010 | Outbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | The booking-specific manage link it issues is what FEAT-10 reads to identify the booking | Client taps the link |
| FEAT-08.SPEC-012 / SPEC-013 | Outbound | FEAT-06, FEAT-14, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | These features send their own messages (access links, opt-out confirmations, pause notices, waitlist notices, recurring-series notices, deposit requests, sign-in codes and alerts, Pro-management notices) through the text/email capability this feature owns | Each feature's own trigger |

## Non-Functional Notes

**Data volumes / growth:** One Message record per send, growing with booking volume (SC-19: roughly 20–40 bookings a week per pro, each producing at least a confirmation and a reminder, plus any change notices); since Messages are immutable and never deleted, volume grows steadily over a pro's multi-year history (SC-22, ASMP-22), but each record is small (a content summary, not a full transcript).

**Responsiveness:** Confirmations arrive within about a minute of payment; reminders fire at their computed time and only within roughly 8am–9pm in the Pro's timezone, moving to the nearest allowed time otherwise (ASMP-29, XBR-16). A failed text is retried once and falls back to email without the client or Pro ever being left to wonder whether it went out (XBR-17, ASMP-26 — correctness over uptime numbers).

**Data sensitivity / privacy:** Every message carries personal data (recipient contact and appointment details) and is visible only to the Pro for their own bookings and to Support for delivery-status troubleshooting only — never the client's raw text replies, only the resulting status (Access field; ASMP-23). No text is ever sent without active Messaging Consent for that client and Pro (XBR-15, ASMP-24). The Pro's studio_address, which may be a home address, is disclosed only inside a booked client's own confirmation and reminder, never more broadly (Pro Account entity's Data Sensitivity note). Every message and its landing screen (SPEC-003) must remain fully usable at phone width, with a screen reader, and without relying on color alone (ASMP-28).

**Compliance flags:** US SMS-consent rules apply to every client text: explicit opt-in captured at booking (owned by FEAT-14), honored immediately on opt-out, and sends restricted to reasonable daytime hours (ASMP-24, ASMP-29). This is the feature that must uphold both rules on every send, since it owns message timing (XBR-16) and reads consent before every send (SPEC-011).

## Non-Goals

- **Marketing or promotional text campaigns** -- Excluded per scope-boundaries.md (SC-15): clients consent to booking-related texts only, and this feature sends nothing beyond confirmations, reminders, change notices, and access links.
- **WhatsApp as a messaging channel** -- Deferred to a later phase per scope-boundaries.md's Relevant Deferral Notes: "WhatsApp is a nice-to-have later, not v1" (BRIEF.md, Ecosystem & Integrations); WhatsApp Reminders (a separate feature) will extend the channel set this feature's Integration specs support once built.
- **Instagram-based delivery of confirmations, reminders, or alerts** -- Excluded per scope-boundaries.md (SC-06): Chairtime has no Instagram integration beyond serving as the destination for the bio link, so every message in this feature travels by text or email only.
- **Chairtime deciding a card-issuer dispute** -- Excluded per scope-boundaries.md (SC-17): FEAT-08.SPEC-006 alerts the Pro that a dispute exists and FEAT-16 supplies the evidence timeline, but this feature never rules on the dispute itself; that is owned by the Pro and the payment processor's dispute process.
- **Waitlist notification for a freed slot** -- Deferred to v1 per scope-boundaries.md's Relevant Deferral Notes and the Feature Dependency Map (XBR-28): at MVP a freed slot is simply open on the public page with no notification sent; from v1, Waitlist for Cancelled Slots (FEAT-20) sends that notification on its own behalf, not through this feature's Notification specs.
- **Retention or purge policy for sent messages** -- Intentional lifecycle decision surfaced by the CRUD matrix: a Message is immutable and kept for the life of the account (SC-22) as part of the append-only activity record FEAT-16 relies on for dispute evidence; there is no delete/archive path to design.

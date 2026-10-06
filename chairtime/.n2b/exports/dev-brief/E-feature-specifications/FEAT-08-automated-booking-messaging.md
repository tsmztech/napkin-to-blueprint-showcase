# FEAT-08 — Automated Booking Messaging

This chapter covers Automated Booking Messaging (FEAT-08), a Core-tier feature. It carries 13 specifications carrying 167 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-08.SPEC-001 | Booking Confirmation Message | notification | 15 |
| FEAT-08.SPEC-002 | Appointment Reminder Message | notification | 12 |
| FEAT-08.SPEC-003 | Reminder Reply Acknowledgment | screen | 11 |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | notification | 13 |
| FEAT-08.SPEC-005 | Pro Booking Activity Notification | notification | 12 |
| FEAT-08.SPEC-006 | Pro Attention Alert | notification | 16 |
| FEAT-08.SPEC-007 | Reminder Scheduling & Timing Window Enforcement | automation | 12 |
| FEAT-08.SPEC-008 | Reminder Reply Routing | automation | 11 |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | automation | 12 |
| FEAT-08.SPEC-010 | Booking-Specific Manage Link Issuance | automation | 11 |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | logic-rule | 15 |
| FEAT-08.SPEC-012 | Transactional Text Messaging Capability | integration | 14 |
| FEAT-08.SPEC-013 | Transactional Email Capability | integration | 13 |

The feature breakdown brief follows, then every specification in full.


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



# Notification Spec: Booking Confirmation Message

## Overview

**Name:** Booking Confirmation Message
**ID:** FEAT-08.SPEC-001
**Type:** Notification
**Purpose:** Tells the client, the moment their deposit payment succeeds, that their appointment is confirmed -- carrying every detail they need (what, when, where, what was paid, what's still owed, when they can no longer cancel free, and how to manage or add the booking to their own calendar) without needing to ask the Pro anything.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The immediate post-payment confirmation message, on both of the product's client channels (text and email)
- Every content element BRIEF.md's Vision and this feature's Key Capabilities name: service, date/time with timezone, deposit paid, balance due in person, studio location, cancellation cut-off, manage link, add-to-calendar option
- Generating the add-to-calendar export itself: a calendar-event (.ics) link built directly from this Booking's own fields (service, start_time, studio_address) at the moment the confirmation is composed. This is a distinct action from the manage link (opens the booking in-product to manage it) -- add-to-calendar puts the appointment on the client's own calendar app and needs no in-product access grant, since it discloses nothing beyond what this message already shows the client
- Delivery timing (within about a minute of payment) and its channel selection

**Non-Goals:**
- Deciding text vs. email for this send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule), which this spec defers to before every send.
- Minting the manage link itself -- owned by FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance); this spec only embeds the link it produces.
- Retrying a failed send or falling back to email after a failed text -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), which governs delivery failure for every Message this feature creates, including this one.
- Confirming a cancellation, reschedule, or refund -- excluded per product-features.md's Communications field, which treats the change/refund notice as a distinct communication; covered by FEAT-08.SPEC-004 (Booking Change & Refund Notice).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | BRIEF.md's Vision names text as the primary confirmation channel ("a confirmation text lands immediately"); Riley is mid-session on her phone right after paying and expects the confirmation where she already is |
| Email | The client has not granted texting consent, or provided only an email at booking (BRIEF.md's stated fallback: "Email confirmations are acceptable as a fallback") | Every client who declines texting still supplies an email at booking (product-features.md, Client entity), so the confirmation is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Deposit payment completes and the Booking flips to Confirmed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Always, on every successful first-time deposit capture for a booking | Booking (service, start_time, price_agreed, deposit_amount, balance_due, policy_version), Client (name, phone, email), Pro Account (studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only for the Client) -- the sole recipient of this message, since it discloses that client's own appointment and payment details. Platform Operator (Support) never receives this message; per the Access Matrix, Support has View-only access to delivery status (not message content) for troubleshooting.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking (Granted or Revoked, per the client's choice) | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This confirmation itself carries no on/off toggle: it is the transactional record of a payment the client just made, not a discretionary reminder. A client cannot opt out of being told their own booking is confirmed -- only the channel it arrives on varies, per FEAT-08.SPEC-011.

**Quiet Hours:** N/A -- the confirmation is a direct, expected response to an action the client just took (paying); it is not an unprompted interruption, so the daytime-hours window that governs FEAT-08.SPEC-002's reminders (XBR-16) does not apply here. The confirmation sends at whatever time the payment completes, day or night.

## Content Definition

**Text:**
- **Body:** {pro_display_name} confirmed: {service_name} on {appointment_date} at {appointment_time} ({timezone}). Deposit paid: {deposit_amount}. Balance due at your visit: {balance_due}. Location: {studio_address}. Free cancel/reschedule until {cancellation_cutoff}. Manage: {manage_link} Add to calendar: {calendar_export_link}
- **CTA (manage):** {manage_link} -- deep-links directly to this one booking through FEAT-06's (Client Booking Identity) link resolution
- **CTA (add to calendar):** {calendar_export_link} -- opens/downloads a calendar-event (.ics) file for this appointment on the client's own device; a separate action from the manage link, not a manage-link sub-option

**Email:**
- **Subject:** Confirmed: {service_name} with {pro_display_name} on {appointment_date}
- **Body:**
  Hi {client_first_name},

  Your appointment is confirmed:

  Service: {service_name}
  Date & time: {appointment_date} at {appointment_time} ({timezone})
  Location: {studio_address}
  Deposit paid: {deposit_amount}
  Balance due at your visit: {balance_due}
  Free to cancel or reschedule until: {cancellation_cutoff}

  Need to make a change? Use the link below. Want it on your calendar? Add it with one tap.
- **CTA (button, manage):** Manage my booking -- deep-links directly to this booking (FEAT-06's Client Booking Identity link resolution)
- **CTA (button, calendar):** Add to calendar -- opens/downloads the {calendar_export_link} calendar-event (.ics) file for this appointment

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- display_name is required (Pro Account entity, feature-dependency-map.md) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {appointment_date} / {appointment_time} | Booking -- start_time, rendered in the Pro's timezone | Oct 4, 2026 / 2:30 PM | Never empty -- start_time is fixed at booking |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {deposit_amount} | Booking -- deposit_amount | $40.00 | Never empty -- fixed at booking (XBR-05) |
| {balance_due} | Booking -- balance_due (derived: price_agreed − deposit_amount) | $85.00 | Renders as "$0.00" when the deposit equals the full price -- never blank |
| {studio_address} | Pro Account -- studio_address | 123 Main St, Suite 4, Austin, TX | Never empty -- required before go-live (XBR-26) |
| {cancellation_cutoff} | Derived -- Booking.start_time minus Cancellation Policy.window_hours | Oct 2, 2026, 2:30 PM | Never empty -- every Booking carries a policy_version with a window_hours value (XBR-08) |
| {manage_link} | Access Link -- created by FEAT-08.SPEC-010, scoped to this Booking | chairtime.app/m/8f2a1c | If link issuance fails, the confirmation is held and retried per FEAT-08.SPEC-009's failure handling -- a confirmation is never sent without its manage link |
| {calendar_export_link} | Derived -- a calendar-event (.ics) file generated at send time directly from Booking (service, start_time) and Pro Account (studio_address, timezone) fields; not an Access Link and not scoped/expiring the way {manage_link} is, since it discloses nothing the confirmation itself does not already show | chairtime.app/cal/8f2a1c.ics | Never empty -- generated deterministically from fields that are always present on a confirmed Booking (same fields the confirmation body itself requires) |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one confirmation is sent per Booking, at the single moment its deposit is captured. There is nothing to batch: a client receives at most one Booking Confirmation Message per booking.
**Deduplication:** At most one confirmation per Booking. FEAT-07.SPEC-002's deposit capture is itself guaranteed to fire at most once per booking (per its own idempotency rules, FEAT-07.SPEC-004), so this notification's trigger cannot re-fire for the same booking; a redundant trigger attempt is a no-op.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17) -- never silently dropped.
**Expiry:** None -- a booking confirmation never becomes not-worth-sending. Even a late-arriving confirmation (after a retry/fallback cycle) still carries currently accurate information, since Booking fields are fixed at booking time (XBR-04, XBR-05).

## Edge Cases

- **Deposit captured but manage-link issuance (FEAT-08.SPEC-010) has not yet completed** -- The confirmation send waits for the link; it is never sent with a placeholder or missing link. If link issuance itself fails, this is treated as a send failure under FEAT-08.SPEC-009's retry/fallback path.
- **Client has both texting consent and an email on file** -- Per FEAT-08.SPEC-011, text is used; no duplicate confirmation is also sent by email in this case.
- **Client's phone number changed since booking but before the confirmation sends** -- FEAT-08.SPEC-011 requires fresh consent for a changed number (XBR-15); until fresh consent exists, the confirmation routes to email, never to the old or unconsented number.
- **Pro's studio_address is edited between booking and confirmation send** -- The confirmation shows the studio_address value at send time (current Pro Account field), since the confirmation is generated at send, not pre-composed at booking; this is consistent with the Pro Account entity carrying one current address rather than a per-booking snapshot.
- **Booking is cancelled in the brief window between payment capture and confirmation send** -- The confirmation still sends (it reports what was true at the moment of successful payment); the client also promptly receives the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the cancellation, so the client is never left believing a cancelled booking is still active.
- **Client taps {calendar_export_link} after the confirmation has already been superseded by a cancellation or reschedule (FEAT-08.SPEC-004)** -- The calendar file still adds successfully, since it is a self-contained record of what the confirmation stated at send time, not a live-refreshing link; the client's device calendar entry may then be stale, which is why the manage link (not the calendar file) is the channel of record for any subsequent change.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Successful deposit capture and Booking confirmation fires this notification |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies the {manage_link} placeholder embedded in every confirmation |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is the chosen channel or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | References (outbound) | Covers the client-facing message if the booking is subsequently changed |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | The manage link's tap opens this booking through FEAT-06's link resolution |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of this notification is written to the append-only activity record |

## Analytics and Success Signals

- **confirmation_sent** (channel: text / email; delay_seconds since payment capture) -- N/A -- no Stage 2 metric measures confirmation delivery speed directly, though the target time is stated in Non-Functional Notes; retained so send-latency compliance with the "within about a minute" commitment is observable.
- **confirmation_delivery_confirmed** (channel) -- N/A -- no dedicated Stage 2 metric; delivery reliability of this message feeds the trust that underlies "Deposit Capture Rate" (success-metrics.md) but is not itself that metric's numerator or denominator.
- **confirmation_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"
- **confirmation_calendar_link_tapped** (channel: text / email) -- N/A -- no Stage 2 metric measures calendar-export usage directly; retained so the "add to my calendar" capability's actual usage is observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-001-AC-01:** Given Riley just paid her deposit for a Full Set Lashes appointment and has active texting consent, when the payment capture completes (FEAT-07.SPEC-002), then Riley receives a text confirming the service, date/time with timezone, deposit paid, balance due, studio location, cancellation cut-off, and a manage link, within about a minute.

**FEAT-08.SPEC-001-AC-02:** Given Riley declined texting at booking and provided an email, when her deposit payment completes, then she receives the confirmation by email instead, with the same content elements.

**FEAT-08.SPEC-001-AC-03:** Given Riley receives her text confirmation, when she taps the manage link, then it opens her booking through FEAT-06's link resolution, scoped to this one appointment.

**FEAT-08.SPEC-001-AC-04:** Given a service priced at $125 with a $40 deposit, when the confirmation is composed, then it shows "Deposit paid: $40.00" and "Balance due at your visit: $85.00".

**FEAT-08.SPEC-001-AC-05:** Given the Pro's cancellation policy window is 24 hours and the appointment is at 2:30 PM on Oct 4, when the confirmation is composed, then it states the cancellation cut-off as Oct 3, 2:30 PM.

**FEAT-08.SPEC-001-AC-06:** Given Riley pays her deposit but manage-link issuance has not yet completed, when the confirmation send is attempted, then the send waits for the link and never dispatches a confirmation missing its manage link.

**FEAT-08.SPEC-001-AC-07:** Given Riley's booking is cancelled 30 seconds after her deposit captures, when the confirmation and the cancellation both process, then Riley receives both the confirmation (reflecting the moment of successful payment) and the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the cancellation.

**FEAT-08.SPEC-001-AC-08:** Given Riley's deposit capture (FEAT-07.SPEC-002) is guaranteed to fire at most once for her booking, when the confirmation trigger is evaluated, then at most one confirmation is ever sent for that booking.

**FEAT-08.SPEC-001-AC-09:** Given a text confirmation to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the confirmation, by email, and the delivery gap is flagged on the Pro's dashboard.

**FEAT-08.SPEC-001-AC-10:** Given Riley's phone number changed since her last booking and fresh texting consent has not yet been captured, when a new booking's confirmation is composed, then it is sent by email, never to the unconsented number.

**FEAT-08.SPEC-001-AC-11:** Given the Pro edits her studio_address between Riley's booking and the confirmation send, when the confirmation is composed, then it shows the studio_address value current at send time.

**FEAT-08.SPEC-001-AC-12:** Given Riley has both texting consent and an email on file, when her confirmation sends, then it arrives once, by text, and no duplicate email confirmation is also sent.

**FEAT-08.SPEC-001-AC-13:** Given Platform Operator (Support) is troubleshooting a delivery issue for Riley's booking, when Support views the Message record, then Support sees delivery status only, never the confirmation's content as a raw text.

**FEAT-08.SPEC-001-AC-14:** Given Riley taps "Manage my booking" from her email confirmation, when the tap registers, then the confirmation_cta_tapped event fires with destination "manage_link".

**FEAT-08.SPEC-001-AC-15:** Given Riley receives her confirmation by text or email, when she taps "Add to calendar" ({calendar_export_link}), then a calendar-event file is produced for her Full Set Lashes appointment carrying the service, date/time in her Pro's timezone, and studio location, independently of whether she also taps the manage link, and the confirmation_calendar_link_tapped event fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Notification Spec: Appointment Reminder Message

## Overview

**Name:** Appointment Reminder Message
**ID:** FEAT-08.SPEC-002
**Type:** Notification
**Purpose:** Reminds the client before their appointment and offers a one-tap "I'll be there" or "I need to reschedule" response, replacing the Pro's habit of texting reminders by hand.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The pre-appointment reminder message content, on both client channels (text and email)
- The two embedded one-tap reply options and their exact wording
- Delivery timing and window constraints as computed by the paired Automation spec

**Non-Goals:**
- Computing when the reminder fires (the default two-day lead time, the 8am--9pm window, and the late-booking suppression rule) -- owned by FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement), which this spec's Trigger section defers to entirely.
- Processing which reply was tapped -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing); this spec only defines what is sent and its two reply options, not what happens after a tap.
- The landing page shown after "I'll be there" is tapped -- owned by FEAT-08.SPEC-003 (Reminder Reply Acknowledgment).
- Choosing text vs. email for this send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | BRIEF.md's Vision names the reminder as a text with tap-reply options; a link tap works identically from a text |
| Email | The client has not granted texting consent | The Validation & Limits field states explicitly: "the one-tap replies are link taps, so a reply works the same by text or email" -- no client is left without a working reminder because they declined texting |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A confirmed Booking reaches its computed reminder send time | FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Fires once per Booking, at the time FEAT-08.SPEC-007 computes, provided FEAT-08.SPEC-007 has not suppressed the reminder for a late booking | Booking (service, start_time, deposit_amount, balance_due, policy_version), Client (name, phone, email), Pro Account (studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) never receives this message; Support has View-only access to delivery status only.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether the reminder sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

The reminder carries no separate on/off toggle beyond texting consent itself -- product-features.md defines no reminder-mute preference; the reminder is a core commitment of the product's promise to the client (know when their appointment is and be able to respond), not a discretionary alert the client can silence while keeping the booking.

**Quiet Hours:** The reminder's own send-timing window (roughly 8am--9pm in the Pro's timezone, XBR-16) is computed and enforced entirely by FEAT-08.SPEC-007 before this spec's trigger ever fires -- by the time this notification is triggered, the window has already been satisfied. This spec therefore applies no separate quiet-hours logic of its own; see FEAT-08.SPEC-007 for the window computation and its nearest-allowed-time fallback.

## Content Definition

**Text:**
- **Body:** Reminder: {service_name} with {pro_display_name} on {appointment_date} at {appointment_time} ({timezone}). Balance due at your visit: {balance_due}. Reply or tap: {ill_be_there_link} I'll be there, or {reschedule_link} I need to reschedule.
- **CTA (I'll be there):** {ill_be_there_link} -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), which records the acknowledgment and forwards to FEAT-08.SPEC-003 (Reminder Reply Acknowledgment)
- **CTA (I need to reschedule):** {reschedule_link} -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), which routes into FEAT-10 (Client-Initiated Cancel/Reschedule) via the booking-specific manage link

**Email:**
- **Subject:** Reminder: your appointment with {pro_display_name} on {appointment_date}
- **Body:**
  Hi {client_first_name},

  Just a reminder about your upcoming appointment:

  Service: {service_name}
  Date & time: {appointment_date} at {appointment_time} ({timezone})
  Balance due at your visit: {balance_due}

  Let us know you're coming, or reschedule if something's come up:
- **CTA (button, I'll be there):** I'll be there -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing)
- **CTA (button, I need to reschedule):** I need to reschedule -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), routing to FEAT-10

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time, rendered in the Pro's timezone | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty (required per account, XBR-25) |
| {balance_due} | Booking -- balance_due (derived) | $85.00 | Renders "$0.00" when nothing further is owed |
| {ill_be_there_link} / {reschedule_link} | Access Link -- the booking-specific manage link (FEAT-08.SPEC-010), each carrying a distinct reply-action parameter | chairtime.app/m/8f2a1c?r=yes / ?r=resched | If link issuance fails, the reminder is held and retried per FEAT-08.SPEC-009 -- never sent without working reply links |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one reminder is sent per Booking, at its single computed reminder time (FEAT-08.SPEC-007). A client with two separate upcoming bookings with the same Pro receives two separate reminders, each tied to its own appointment; the product defines no household- or client-level batching of reminders across bookings.
**Deduplication:** At most one reminder per Booking. FEAT-08.SPEC-007 computes exactly one reminder send time per booking and marks it produced once fired; a scheduler re-evaluation never re-sends a reminder already dispatched for that booking.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the delivery gap flagged on the Pro's dashboard (XBR-17).
**Expiry:** A reminder that has not been delivered by the time its Booking's appointment start_time passes is no longer sent -- reminding about an appointment that has already happened or passed its usefulness window serves no purpose. In that case, the underlying delivery failure is still flagged to the Pro via FEAT-08.SPEC-006 (Pro Attention Alert) so the gap is never silent.

## Edge Cases

- **Client taps "I'll be there" or "I need to reschedule" after the appointment has already passed** -- FEAT-08.SPEC-010's link scoping (booking-specific links stop working once the appointment passes, XBR-18) means the tap lands on FEAT-08.SPEC-003's expired-link state rather than processing a stale reply.
- **Both reply links are tapped (client changes their mind)** -- FEAT-08.SPEC-008 processes only the first tap it receives; a second tap on the other link is treated as a new action against the booking's then-current state (for example, if "I need to reschedule" already routed into FEAT-10, a later "I'll be there" tap on the same reminder no longer applies once the booking has moved into the reschedule flow).
- **The reminder's send time is reached but FEAT-08.SPEC-007 suppressed it (late booking)** -- No reminder is triggered at all for that booking; this is a design decision (the confirmation already sent serves instead), not a delivery failure, so no gap is flagged.
- **Reminder scheduled but the booking is cancelled or rescheduled before the reminder time arrives** -- The reminder is cancelled and never sent; the client instead already has the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the change. Reminding about a booking that no longer exists in its original form would confuse rather than help.
- **Client's texting consent is revoked between the confirmation and the reminder** -- The reminder honors the consent state current at reminder send time, per FEAT-08.SPEC-011's re-check-on-every-send rule; it is sent by email even though the confirmation went by text.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Triggered by (inbound) | Computes and fires this notification's send time |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | Triggers (outbound) | Both CTA links deep-link into this automation |
| FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) | References (outbound) | Reached via FEAT-08.SPEC-008 after an "I'll be there" tap |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | Reached via FEAT-08.SPEC-008 after an "I need to reschedule" tap |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies both reply link placeholders |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual send |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Fires if the reminder expires undelivered |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of reminder is written to the append-only activity record |

## Analytics and Success Signals

- **reminder_sent** (channel: text / email) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_confirmed** (reply: ill_be_there) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_reschedule_requested** (reply: reschedule) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_expired_undelivered** (reason: delivery_failure) -- N/A -- no Stage 2 metric measures undelivered reminders directly; retained so a silently-missed reminder is never invisible, consistent with XBR-17.

## Acceptance Criteria

**FEAT-08.SPEC-002-AC-01:** Given Riley's booking reaches its computed reminder time and she has active texting consent, when FEAT-08.SPEC-007 fires the trigger, then Riley receives a text with the service, date/time, balance due, and both "I'll be there" and "I need to reschedule" tap options.

**FEAT-08.SPEC-002-AC-02:** Given Riley declined texting, when her reminder fires, then she receives the same content by email with both reply options as buttons.

**FEAT-08.SPEC-002-AC-03:** Given Riley taps "I'll be there" in her text reminder, when the tap registers, then FEAT-08.SPEC-008 records the acknowledgment and Riley lands on FEAT-08.SPEC-003.

**FEAT-08.SPEC-002-AC-04:** Given Riley taps "I need to reschedule" in her email reminder, when the tap registers, then FEAT-08.SPEC-008 routes her into FEAT-10 (Client-Initiated Cancel/Reschedule) for that booking.

**FEAT-08.SPEC-002-AC-05:** Given Riley's booking was cancelled before her reminder's computed send time arrived, when the send time passes, then no reminder is sent, since the reminder was cancelled at cancellation.

**FEAT-08.SPEC-002-AC-06:** Given Riley's booking was made after its own reminder point would have fired (FEAT-08.SPEC-007's suppression rule), when the scheduler evaluates it, then no reminder is triggered for that booking.

**FEAT-08.SPEC-002-AC-07:** Given a text reminder to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the reminder by email before her appointment.

**FEAT-08.SPEC-002-AC-08:** Given Riley's reminder cannot be delivered on any channel before her appointment's start_time passes, when the expiry cutoff is reached, then the reminder is no longer sent and FEAT-08.SPEC-006 flags the delivery gap to the Pro.

**FEAT-08.SPEC-002-AC-09:** Given Riley taps "I need to reschedule" after her appointment has already passed, when she follows the link, then she reaches FEAT-08.SPEC-003's expired-link state, not an active reschedule flow, because the booking-specific link has expired.

**FEAT-08.SPEC-002-AC-10:** Given Riley taps "I'll be there" and then taps "I need to reschedule" moments later on the same reminder, when the second tap is processed, then it is evaluated against the booking's then-current state rather than silently overwriting the first acknowledgment.

**FEAT-08.SPEC-002-AC-11:** Given Riley has two separate upcoming bookings with the same Pro, when each reaches its own computed reminder time, then she receives two separate reminders, each naming only its own appointment.

**FEAT-08.SPEC-002-AC-12:** Given Riley revokes texting consent between her confirmation and her reminder, when the reminder's send time arrives, then it is sent by email, honoring the consent state current at send time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Reminder Reply Acknowledgment

## Overview

**Name:** Reminder Reply Acknowledgment
**ID:** FEAT-08.SPEC-003
**Type:** Screen
**Purpose:** Confirms to the client, after they tap "I'll be there" in a reminder, that their attendance was recorded -- and handles the cases where the tapped link no longer works.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The landing page a client reaches after FEAT-08.SPEC-008 records an "I'll be there" acknowledgment
- The Error/Permission-Denied states for a booking-specific link that is expired, already used, or belongs to a passed appointment

**Non-Goals:**
- Processing the tap itself (recording the acknowledgment on the Booking) -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing); this screen only renders after that processing completes.
- The "I need to reschedule" path -- that tap routes directly into FEAT-10 (Client-Initiated Cancel/Reschedule) and never reaches this screen, per the Brief's Internal Dependency Map.
- Any booking management action (viewing full booking details, cancelling, rescheduling) -- this screen is a one-line confirmation, not a booking dashboard; those actions live in FEAT-06 (Client Booking Identity) and FEAT-10, reached only if the client explicitly navigates onward.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) via FEAT-08.SPEC-008 (Reminder Reply Routing) | Client taps "I'll be there" in a text or email reminder | Booking reference (from the tapped booking-specific manage link); the acknowledgment has already been recorded on the Booking by the time this screen renders |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to the one Booking the tapped link names | Navigate onward to manage the booking (via a link to FEAT-06) | -- |
| The Pro (Talia) | Not applicable -- this screen is never navigated to by the Pro; the Pro instead sees the "I'll be there" status reflected on the dashboard (FEAT-12) | No | If a Pro somehow opens the link, it is treated as any other visitor without that specific client's session context: the same booking-specific link content is shown (the link carries no signed-in-role distinction), since the link's scope, not a login, is what gates access |
| Platform Operator (Support) | Not applicable -- Support has no client-link access; Support views delivery status via FEAT-16, never by using or bypassing a client's access link (XBR-24) | No | Support is never issued or expected to open a booking-specific manage link; there is no dedicated denial experience because this path does not exist for Support |
| Unauthenticated | Yes -- this screen requires no sign-in; the booking-specific link itself is the credential (XBR-18) | Yes, for the single acknowledgment already recorded before arrival | N/A -- there is no signed-in-only version of this screen; a person with no valid link sees the Expired/Invalid state below instead of the acknowledgment |
| Expired session | N/A -- this screen has no session concept; each visit is scoped entirely to the tapped link | N/A | A visit with an expired or already-used link shows the Expired/Invalid Link state (see States), which reads "This link is no longer active. Request a new link from your confirmation or reminder message, or contact {pro_display_name} directly." |

## Layout and Content

**Header:** No back arrow (this screen is a landing destination, not a step in a flow the client navigated forward through) and no page chrome beyond the Pro's display name, so the client immediately recognizes whose appointment this concerns.

**Body:** A single centered content block:
- A confirmation icon/mark (non-interactive, decorative)
- Heading: "You're all set, {client_first_name}"
- A one-line summary restating the appointment: "{service_name} with {pro_display_name} on {appointment_date} at {appointment_time}"
- A secondary line: "We've let {pro_display_name} know you're coming."
- A single link/button: "Manage my booking" (navigates to FEAT-06's booking view via the same booking-specific link's scope)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width content block as described, vertically centered on the visible viewport.
- **Medium size class and above:** Same single-column content, capped at a consistent platform-wide narrow-content width and horizontally centered; no structural change beyond width capping, since this screen has no dense content that benefits from a wider layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Manage my booking" link | Tap | Navigate to FEAT-06 (Client Booking Identity), scoped to the same Booking | Screen transitions to the booking view | Standard navigation transition |
| Confirmation icon and summary text | -- | Display-only, non-interactive | None | None |

### Accessibility Notes

- **Focus order:** Heading is announced first on screen load (as the page's primary landmark), followed by the summary line, then the "Manage my booking" link.
- **Load announcement:** On successful load, the heading "You're all set, {client_first_name}" is announced to assistive technology as the page's content, since there is no separate loading transition for the client to perceive (the acknowledgment was already recorded before this screen renders).
- **Keyboard alternatives:** The single interactive element ("Manage my booking") is a standard link, fully reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Acknowledged (default) | The confirmation content described in Layout and Content | The tapped link is valid, unexpired, and its acknowledgment was successfully recorded by FEAT-08.SPEC-008 | Client navigates onward via "Manage my booking" or closes the page |
| Expired/Invalid Link | Heading "This link is no longer active." Body: "Request a new link from your confirmation or reminder message, or contact {pro_display_name} directly." No further action available on this screen. | The tapped link has expired (the appointment has passed, XBR-18) or was already used from a prior visit that already recorded the acknowledgment | Client leaves the page; there is no in-page recovery path, since a booking-specific link is never reissued from this screen |
| Loading | A brief, minimal loading indicator while the link's validity and the Booking reference are resolved | Immediately on tapping the reminder link, before resolution completes | Resolution completes, transitioning to Acknowledged or Expired/Invalid Link |
| Error | Message: "Something went wrong loading your confirmation. Try the link again from your reminder message." with no retry button on this screen (the client re-opens the original message's link) | The Booking reference cannot be resolved for a reason other than expiry (a transient failure) | Client re-opens the link from their original reminder message |
| Offline/Degraded | N/A -- this screen requires connectivity to resolve the link and display the current acknowledgment; without connectivity the link simply fails to load, which the client's device reports as a standard page-load failure, not a screen-level offline state this spec defines | Connectivity lost while attempting to open the link | Connectivity restored and the link is reopened |

## Validation Rules

Not applicable -- this screen accepts no user input; the acknowledgment was already recorded by FEAT-08.SPEC-008 before this screen renders.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Manage my booking" tap | FEAT-06 (Client Booking Identity), booking view | FEAT-06 |
| Expired/Invalid Link -- no in-page action | -- (client leaves or re-opens the original message) | -- |

## Data Model

**Creates:** None -- the acknowledgment itself (Booking.attendance_reply) is written by FEAT-08.SPEC-008 before this screen loads; this screen only reads the result.
**Reads:** Booking -- service, start_time, attendance_reply (to confirm it reflects "I'll be there"); Client -- name (for the greeting); Pro Account -- display_name.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The acknowledgment this screen confirms is written exactly once per booking by FEAT-08.SPEC-008; this screen never re-records or overwrites it, even on a repeat visit to the same valid link.
- Access to this screen is governed entirely by the booking-specific Access Link's own scope and expiry rules (XBR-18), owned by FEAT-06/FEAT-08.SPEC-010 -- no separate sign-in or session state exists for this screen.
- A single-use "already used" state is deliberately not shown as an error: revisiting a still-unexpired link after its acknowledgment was recorded simply re-renders the same Acknowledged content, since re-confirming a true fact is harmless (unlike a single-use on-demand access link, this booking-specific link is reusable until the appointment passes, per the Access Link entity's lifecycle in feature-dependency-map.md).

## Edge Cases

- **Client opens the link twice from the same device** -- The second visit re-renders the same Acknowledged state; no error and no re-processing, since the underlying Booking.attendance_reply is unchanged.
- **Client opens the link on a second device after already acknowledging on the first** -- Same as above: this booking-specific link is not single-use, so both devices show the Acknowledged state consistently.
- **Client forwards the reminder message to someone else, who taps the link** -- The recipient sees the same Acknowledged (or Expired) content scoped to that one booking; they cannot navigate to any other booking, client, or Pro data through this screen (XBR-18).
- **Appointment passes between the reminder being sent and the client tapping the link** -- The link has expired per XBR-18; the client sees the Expired/Invalid Link state rather than a stale acknowledgment.
- **The Pro reschedules the booking after the reminder was sent but before the client taps "I'll be there"** -- FEAT-08.SPEC-010 issues a fresh manage link on a Pro-initiated reschedule; the old reminder's link no longer resolves to the current appointment and shows the Expired/Invalid Link state, directing the client to their latest message instead.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Navigation (inbound) | The "I'll be there" tap in the reminder leads here |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | References (inbound) | Records the acknowledgment this screen confirms |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Owns the link's scope, validity, and expiry that gate this screen |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | "Manage my booking" opens the full booking view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reminder_acknowledgment_viewed | link_state (valid / expired) | Screen loads | supports success-metrics.md: "Reminder Response Rate" |
| reminder_acknowledgment_manage_tapped | -- | Client taps "Manage my booking" | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-08.SPEC-003-AC-01:** Given Riley taps "I'll be there" in her reminder text, when FEAT-08.SPEC-008 records the acknowledgment, then she lands on this screen showing "You're all set, Riley" with her appointment summary.

**FEAT-08.SPEC-003-AC-02:** Given Riley is on the Acknowledged state, when she taps "Manage my booking", then she navigates to FEAT-06's booking view for the same appointment.

**FEAT-08.SPEC-003-AC-03:** Given Riley's appointment has already passed, when she taps the "I'll be there" link from an old reminder, then she sees "This link is no longer active." with no acknowledgment content shown.

**FEAT-08.SPEC-003-AC-04:** Given Riley opens the same valid link twice from two different devices, when each visit resolves, then both show the identical Acknowledged content, with no error on the second visit.

**FEAT-08.SPEC-003-AC-05:** Given Riley forwards her reminder to a friend and the friend taps the link, when the link resolves, then the friend sees only Riley's one booking's acknowledgment content and cannot reach any other booking or client data.

**FEAT-08.SPEC-003-AC-06:** Given the Pro reschedules Riley's booking after sending the reminder, when Riley later taps the original "I'll be there" link, then she sees the Expired/Invalid Link state, since a fresh link was issued for the rescheduled booking.

**FEAT-08.SPEC-003-AC-07:** Given the link resolves but the Booking reference cannot be loaded due to a transient failure, when the page attempts to render, then Riley sees "Something went wrong loading your confirmation. Try the link again from your reminder message."

**FEAT-08.SPEC-003-AC-08:** Given Riley's device loses connectivity while opening the link, when the page attempts to load, then the link fails to load as a standard page-load failure, and no partial or stale acknowledgment content is shown.

**FEAT-08.SPEC-003-AC-09:** Given Riley is on the Acknowledged screen, when it renders, then no interactive elements beyond "Manage my booking" are present, and the heading and summary are read-only content.

**FEAT-08.SPEC-003-AC-10:** Given the tapped link is scoped to a Booking with attendance_reply already set to "I'll be there" from a prior visit, when this screen loads again, then it re-renders the same Acknowledged content without re-processing the acknowledgment.

**FEAT-08.SPEC-003-AC-11:** Given a screen reader user reaches this page, when it finishes loading, then the heading "You're all set, {client_first_name}" is announced as the primary landmark, followed by the appointment summary and the "Manage my booking" link in that order.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 4 (acknowledged, expired/invalid, loading, error) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Booking Change & Refund Notice

## Overview

**Name:** Booking Change & Refund Notice
**ID:** FEAT-08.SPEC-004
**Type:** Notification
**Purpose:** Tells the client, promptly, when their booking is cancelled, rescheduled, or refunded by either themselves or the Pro -- including exactly what happened to their deposit -- so no client is ever left wondering whether a change went through or whether their money is safe.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The client-facing notice for every booking-change type: client cancellation, client reschedule, Pro cancellation, Pro reschedule, and a refund completing or entering an in-progress state
- The deposit outcome for each change type (refunded, kept, carried over, or in progress)
- Variant content per change type, sharing one delivery-rules definition

**Non-Goals:**
- Deciding the deposit outcome itself (refund vs. keep, full vs. carried-over) -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec only reports the outcome FEAT-09 or FEAT-30 determines.
- The Pro-facing notification of the same events -- covered separately by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is client-facing only.
- Alerting the Pro that a refund failed -- that is a Pro-attention condition owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec's "refund in progress" client wording is the client-side counterpart to that same underlying failure.
- The original booking confirmation content -- owned by FEAT-08.SPEC-001; this spec covers only subsequent changes to an already-confirmed booking.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | A cancellation, reschedule, or refund is time-sensitive and financially material; the client should learn of it wherever they already receive booking messages |
| Email | The client has not granted texting consent | Ensures the notice always reaches the client, per BRIEF.md's stated email fallback |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client cancels or reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successfully saved cancellation or reschedule | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome), Cancellation Policy (window_hours) |
| Pro cancels or reschedules a client's booking | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) | Always, on a successfully saved Pro-initiated cancellation or reschedule | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome) |
| Pro issues a goodwill refund | FEAT-30.SPEC-009 (Goodwill Refund Commit) / FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) | On a successfully processed goodwill refund | Deposit Transaction (Refunded outcome) |
| Automatic refund succeeds, enters progress, or fails | FEAT-09.SPEC-005 (Automatic Deposit Refund) (Cancellation & No-Show Policy Engine) | On the corresponding Deposit Transaction state transition | Deposit Transaction (status: Refunded / Refund in Progress) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this notice sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |

This notice carries no separate opt-out: a change to the client's own paid booking is transactional information the client cannot decline to receive, consistent with the treatment of FEAT-08.SPEC-001.

**Quiet Hours:** N/A -- like the confirmation, this notice is the direct, expected report of a change that just happened to the client's own booking, not an unprompted interruption; it sends immediately regardless of time of day. XBR-16's daytime window applies only to the discretionary pre-appointment reminder (FEAT-08.SPEC-002), not to this transactional notice.

## Content Definition

**Text (client-initiated cancellation, outside window -- full refund):**
- **Body:** Your {appointment_date} appointment with {pro_display_name} has been cancelled. Your {deposit_amount} deposit is being refunded to your card. Manage: {manage_link}

**Text (client-initiated cancellation, inside window -- deposit kept):**
- **Body:** Your {appointment_date} appointment with {pro_display_name} has been cancelled. Per the cancellation policy you agreed to, your {deposit_amount} deposit is kept. Manage: {manage_link}

**Text (client-initiated reschedule, outside window -- deposit carried over):**
- **Body:** Your appointment with {pro_display_name} has been moved to {new_appointment_date} at {new_appointment_time} ({timezone}). Your {deposit_amount} deposit carries over -- nothing further to pay now. Manage: {manage_link}

**Text (Pro-initiated cancellation -- always full refund):**
- **Body:** {pro_display_name} has cancelled your {appointment_date} appointment. Your {deposit_amount} deposit is being refunded to your card in full. Manage: {manage_link}

**Text (Pro-initiated reschedule):**
- **Body:** {pro_display_name} has moved your appointment to {new_appointment_date} at {new_appointment_time} ({timezone}). Manage: {manage_link}

**Text (refund in progress):**
- **Body:** Your {deposit_amount} refund for your {appointment_date} appointment with {pro_display_name} is in progress. You don't need to do anything -- it will complete automatically. Manage: {manage_link}

**Email (mirrors each text variant above):**
- **Subject:** Update on your appointment with {pro_display_name}
- **Body:** Hi {client_first_name}, followed by the plain-language equivalent of the matching text variant above, with the same facts (what changed, the new time if applicable, and the exact deposit outcome).
- **CTA (button):** Manage my booking -- deep-links to the booking-specific manage link

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {appointment_date} | Booking -- start_time (the original time, for a cancellation notice) | Oct 4, 2026 | Never empty -- fixed at booking |
| {new_appointment_date} / {new_appointment_time} | Booking -- start_time (the updated time, for a reschedule notice) | Oct 11, 2026 / 2:30 PM | Never empty when the variant is a reschedule notice -- this placeholder is not used in cancellation variants |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty (required per account) |
| {deposit_amount} | Deposit Transaction -- amount | $40.00 | Never empty -- fixed once at booking |
| {manage_link} | Access Link -- the booking-specific manage link (FEAT-08.SPEC-010) | chairtime.app/m/8f2a1c | If the underlying booking is now fully closed out (completed history), the link still resolves to a read-only view of that booking's final state |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- each change event (a cancellation, a reschedule, or a refund-state transition) produces its own single notice at the moment it occurs. A booking that is rescheduled and later cancelled produces two separate notices, one per event, since each is a distinct fact the client needs at the time it happens.
**Deduplication:** At most one notice per triggering event. A refund transitioning from "in progress" to "completed" is itself a second, distinct event and produces its own follow-up notice (see the refund-in-progress content variant and its natural successor, the original cancellation/refund confirmation content once the refund actually completes) -- these are not duplicates of the same event.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard.
**Expiry:** None -- a change or refund notice never becomes not-worth-sending; it reports a fact about the client's own money and appointment that remains true and relevant no matter when it is finally delivered.

## Edge Cases

- **Client cancels their own booking, then the Pro also attempts an action on it before the notice sends** -- Per the Booking entity's Contention resolution (reject-with-refresh, feature-dependency-map.md), only the first committed transition applies; this notice reports the transition that actually committed, never a stale or since-superseded one.
- **A refund fails outright rather than merely being slow** -- The client still sees the "refund in progress" wording, never a failure message; the underlying failure is retried automatically and flagged only to the Pro (FEAT-08.SPEC-006), per XBR-10's "never dropped, never exposed to the client as a failure" framing.
- **A client reschedule lands inside the cancellation window (treated as late cancellation plus new deposit, per XBR-09)** -- This notice reports both facts plainly: the original deposit is kept under the policy, and the new booking's own confirmation (FEAT-08.SPEC-001) covers the new deposit separately; the two are never merged into one ambiguous message.
- **The Pro cancels a booking that the client had already tried to cancel moments earlier** -- Whichever transition committed first is the one this notice reports (Booking entity Contention rule); the client is not sent two conflicting notices for the same terminal state.
- **Client's texting consent is revoked between booking and this notice** -- The notice honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) / FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule fires this notice; FEAT-10.SPEC-006 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit) / FEAT-30.SPEC-009 (Goodwill Refund Commit) / FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) / FEAT-30.SPEC-012 (Pro Booking Management notifications) | Triggered by (inbound) | A Pro-initiated cancellation, reschedule, or goodwill refund fires this notice; FEAT-30.SPEC-012 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | An automatic refund succeeding, entering progress, or failing (client-visible as "in progress") fires this notice |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Covers the Pro-facing counterpart when a refund fails |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | References (outbound) | Covers the Pro-facing counterpart for cancellations and reschedules |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies the {manage_link} placeholder |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual send |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of notice is written to the append-only activity record |

## Analytics and Success Signals

- **change_notice_sent** (change_type: client_cancel / client_reschedule / pro_cancel / pro_reschedule / refund_in_progress; channel) -- supports success-metrics.md: "Automatic Refund Correctness"
- **change_notice_refund_outcome_shown** (outcome: refunded / kept / carried_over / in_progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **change_notice_sent** (change_type: pro_cancel / pro_reschedule) -- supports success-metrics.md: "Pro Change Correctness"
- **change_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given Riley cancels her booking outside the Pro's cancellation window, when FEAT-10 completes the cancellation, then Riley receives a notice stating her deposit is being refunded to her card.

**FEAT-08.SPEC-004-AC-02:** Given Riley cancels her booking inside the cancellation window, when FEAT-10 completes the cancellation, then Riley receives a notice stating her deposit is kept per the policy she agreed to.

**FEAT-08.SPEC-004-AC-03:** Given Riley reschedules her booking outside the window, when FEAT-10 completes the reschedule, then Riley receives a notice with the new date/time and confirmation that her deposit carries over with nothing further to pay.

**FEAT-08.SPEC-004-AC-04:** Given the Pro cancels Riley's booking, when FEAT-30 completes the cancellation, then Riley receives a notice stating her deposit is being refunded in full, regardless of timing.

**FEAT-08.SPEC-004-AC-05:** Given the Pro reschedules Riley's booking to a new time, when FEAT-30 completes the reschedule, then Riley receives a notice with the new date and time.

**FEAT-08.SPEC-004-AC-06:** Given an automatic refund for Riley's cancelled booking cannot complete immediately, when FEAT-09 sets the Deposit Transaction to Refund in Progress, then Riley receives a notice stating the refund is in progress and she does not need to do anything.

**FEAT-08.SPEC-004-AC-07:** Given a refund that was "in progress" for Riley later completes, when FEAT-09 updates the Deposit Transaction to Refunded, then Riley receives a follow-up notice confirming the refund completed.

**FEAT-08.SPEC-004-AC-08:** Given Riley reschedules inside the cancellation window, when FEAT-10 processes the late reschedule (XBR-09), then Riley receives a notice stating her original deposit is kept, distinct from the separate confirmation for her new booking's own deposit.

**FEAT-08.SPEC-004-AC-09:** Given Riley cancels her own booking and, moments later, the Pro also attempts to cancel it, when only the first transition commits (Booking entity Contention rule), then Riley receives exactly one notice, reflecting the committed transition.

**FEAT-08.SPEC-004-AC-10:** Given a refund for Riley's booking fails outright on the Pro's payout side, when the failure occurs, then Riley still sees only the "refund in progress" wording, never a failure message, while the Pro is separately alerted via FEAT-08.SPEC-006.

**FEAT-08.SPEC-004-AC-11:** Given Riley has revoked texting consent since booking, when a change notice for her booking is triggered, then it is sent by email, honoring her current consent state.

**FEAT-08.SPEC-004-AC-12:** Given a text change notice to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email.

**FEAT-08.SPEC-004-AC-13:** Given Riley taps "Manage my booking" from a change notice, when the tap registers, then the change_notice_cta_tapped event fires and she is taken to her booking through the manage link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 4 (client cancel/reschedule, Pro cancel/reschedule/goodwill refund, automatic refund outcome) | 4 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Pro Booking Activity Notification

## Overview

**Name:** Pro Booking Activity Notification
**ID:** FEAT-08.SPEC-005
**Type:** Notification
**Purpose:** Tells the Pro, on her own channels, that a new booking arrived or a client cancelled or rescheduled their own appointment -- so Talia learns about routine schedule changes without having to keep the dashboard open.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The Pro-facing notice for a new booking and for a client-initiated cancellation or reschedule
- In-app, text, and email delivery per the Pro's own notification_preferences (FEAT-27)

**Non-Goals:**
- Anything needing the Pro's attention (delivery failures, calendar reconnection, refund failure, disputes) -- owned by FEAT-08.SPEC-006 (Pro Attention Alert), which is a distinct, higher-urgency notification class from routine activity.
- The client-facing counterpart of the same events -- owned by FEAT-08.SPEC-001 (new booking) and FEAT-08.SPEC-004 (cancellation/reschedule).
- Setting or changing notification_preferences -- owned by FEAT-27 (Pro Profile & Booking Page Settings); this spec only reads and honors that setting.
- Pro-initiated cancellations or reschedules -- the Pro does not need to be told about her own actions; this spec covers only client-initiated activity, per the Brief's Side-Effect Inventory.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, regardless of the Pro's text/email preference | The Pro's dashboard (FEAT-12) is her primary daily touchpoint (BRIEF.md's Vision: "you glance at your phone between clients"); in-app presence is the baseline, never opt-out-able |
| Text | The Pro's notification_preferences include text for this notification type | Talia is between clients on her phone most of the day; a text reaches her without opening the app |
| Email | The Pro's notification_preferences include email for this notification type | Some Pros prefer a running email record of activity they can search later |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New booking is made | FEAT-05 (Public Booking Page & Booking Flow) via FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Always, on a booking's successful confirmation | Booking (service, start_time, deposit_amount), Client (name) |
| Client cancels their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successful client cancellation | Booking (original time, cancellation timestamp), Client (name), Deposit Transaction (outcome) |
| Client reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successful client reschedule | Booking (original time, new time), Client (name) |

## Audience and Preferences

**Recipients:** The Pro (Talia) tied to the affected Booking's account (Access Matrix: Booking & Payment = Full for the Pro). Platform Operator (Support) has View-only access to delivery status only; the Client is not a recipient of this notification (it is not their content to receive -- they get FEAT-08.SPEC-001 / FEAT-08.SPEC-004 instead).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Booking activity notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text (BRIEF.md's Vision centers texting as the Pro's own habit-replacement channel) | FEAT-27 (Pro Profile & Booking Page Settings) |

In-app is never an option to disable: the Pro Account entity's notification_preferences field governs text/email routing only, consistent with the dashboard being the system of record for her day.

**Quiet Hours:** N/A -- product-features.md and assumptions-constraints.md define no quiet-hours window for Pro-facing notifications; the Pro's own working day is not modeled as having off-hours the product enforces on her behalf, unlike the client-consent-driven daytime window that governs client texting (XBR-16, ASMP-29), which exists to satisfy US SMS-consent rules for the Client -- a rule that does not apply to the Pro's own opted-in business communications about her own account.

## Content Definition

**In-app:**
- **Title:** New booking: {client_name}
- **Body:** {service_name} on {appointment_date} at {appointment_time}. Deposit paid: {deposit_amount}.
- **CTA:** View booking -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard), the specific booking row

**In-app (client cancellation):**
- **Title:** {client_name} cancelled
- **Body:** {service_name} on {appointment_date} at {appointment_time}. Deposit: {deposit_outcome_summary}.
- **CTA:** View schedule -- deep-links to FEAT-12

**In-app (client reschedule):**
- **Title:** {client_name} rescheduled
- **Body:** {service_name} moved from {original_appointment_date} to {new_appointment_date} at {new_appointment_time}.
- **CTA:** View schedule -- deep-links to FEAT-12

**Text (new booking):**
- **Body:** New booking: {client_name}, {service_name} on {appointment_date} at {appointment_time}. Deposit paid: {deposit_amount}.

**Text (cancellation):**
- **Body:** {client_name} cancelled their {appointment_date} appointment. Deposit: {deposit_outcome_summary}.

**Text (reschedule):**
- **Body:** {client_name} moved their appointment from {original_appointment_date} to {new_appointment_date} at {new_appointment_time}.

**Email (mirrors each text variant with a subject line "Booking update: {client_name}" and a "View on dashboard" button deep-linking to FEAT-12).**

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {client_name} | Client -- name | Riley Chen | Never empty -- required at client creation |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {original_appointment_date} / {new_appointment_date} / {new_appointment_time} | Booking -- start_time before/after a reschedule | Oct 4, 2026 / Oct 11, 2026 / 3:00 PM | Never empty when the variant is a reschedule notice |
| {deposit_amount} | Booking -- deposit_amount | $40.00 | Never empty -- fixed at booking |
| {deposit_outcome_summary} | Derived -- Deposit Transaction.status rendered in plain language ("refunded" / "kept per policy" / "carried over") | refunded | Renders "pending" if the outcome has not yet been determined at notification time -- never left blank |

## Delivery Rules

**Batching:** Notifications for the same Pro across different bookings are not batched -- each booking event is its own notice, since a Pro benefits from knowing immediately which specific booking changed rather than waiting for a digest. Multiple client-initiated changes to the same booking in quick succession (e.g., a reschedule immediately followed by a cancellation) each produce their own notice, in the order they occur.
**Deduplication:** At most one notification per triggering event per channel. A booking confirmation event and a cancellation event on the same booking are distinct events and both produce their own notice; the same single event is never re-delivered on retry beyond the retry rule below.
**Retry on failure:** Governed by FEAT-08.SPEC-009 for the text channel specifically: a failed text is retried once, then falls back to email, with the gap also visible in-app on the dashboard's attention list (since in-app is never itself the failing channel). Email delivery failure for this notification type is not separately retried beyond the underlying email capability's own delivery attempt (FEAT-08.SPEC-013); the in-app copy remains the surviving record regardless.
**Expiry:** None -- a booking-activity notice remains relevant however late it arrives, since it reports a fact about the Pro's own schedule that stays true; the in-app copy also persists indefinitely as part of the Pro's dashboard/activity history (FEAT-16), so there is no "too late to matter" cutoff.

## Edge Cases

- **The Pro has all notification channels other than in-app turned off** -- She still sees the in-app notice on her dashboard; nothing about her schedule ever depends solely on a channel she has muted.
- **A client cancels and reschedules the same booking within seconds of each other** -- Both events produce their own notice, delivered in the order the underlying Booking transitions committed, never merged into one ambiguous message.
- **The Pro is mid-way through viewing her dashboard when a new booking arrives** -- The in-app notice appears without requiring a manual refresh, consistent with FEAT-12's live-updating nature; text/email notices, if enabled, arrive independently on their own channel.
- **Two client actions on two different bookings occur at effectively the same time** -- Each produces its own independent notice; the notifications are not merged across bookings even though they land close together in time.
- **The Pro's notification_preferences are changed between the triggering event and delivery** -- Per this spec's channel-decision rule (evaluated at delivery time, consistent with the notification methodology's default), a preference change that lands before the message actually dispatches is honored; the in-app copy is unaffected either way since it is never optional.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | A new confirmed booking fires this notification |
| FEAT-10.SPEC-004 (Booking Update Commit) / FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule fires this notification; FEAT-10.SPEC-006 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-012 (Pro Booking Management notifications) | Triggered by (inbound) | FEAT-30.SPEC-012 is the trigger-and-audience contract for Pro-side booking activity; this spec owns the message content the Pro receives |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences that govern text/email channel selection |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Every CTA deep-links to the affected booking on the dashboard |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the text/email sends when those channels are enabled |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback for the text channel |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send is written to the append-only activity record |

## Analytics and Success Signals

- **pro_notification_sent** (type: new_booking / client_cancel / client_reschedule; channels: in_app / text / email) -- N/A -- no Stage 2 metric directly measures Pro notification delivery; this event supports the operational goal named in the feature's rationale (the Pro learns of changes without opening the dashboard) rather than a named success-metrics.md target.
- **pro_notification_cta_tapped** (destination: dashboard_booking_row) -- supports success-metrics.md: "Daily Dashboard Glance Speed"

## Acceptance Criteria

**FEAT-08.SPEC-005-AC-01:** Given Talia has notification_preferences set to "In-app + text" (the default), when a new client books, then she receives both an in-app notice and a text naming the client, service, and time.

**FEAT-08.SPEC-005-AC-02:** Given Talia has set notification_preferences to "In-app only", when Riley cancels her booking, then Talia sees the in-app notice and receives no text or email.

**FEAT-08.SPEC-005-AC-03:** Given Talia has notification_preferences set to "In-app + text + email", when Riley reschedules her booking, then Talia receives the reschedule notice on all three channels with the original and new appointment times.

**FEAT-08.SPEC-005-AC-04:** Given Riley cancels her booking inside the cancellation window, when Talia's notification is composed, then it shows the deposit outcome as "kept per policy".

**FEAT-08.SPEC-005-AC-05:** Given Riley cancels her booking outside the window, when Talia's notification is composed, then it shows the deposit outcome as "refunded".

**FEAT-08.SPEC-005-AC-06:** Given Riley reschedules and then cancels the same booking within seconds, when both events process, then Talia receives two separate notices, in the order the transitions committed.

**FEAT-08.SPEC-005-AC-07:** Given a text notification to Talia fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Talia still receives the notice by email, and the in-app copy remains visible regardless.

**FEAT-08.SPEC-005-AC-08:** Given Talia is actively viewing her dashboard, when a new booking arrives, then the in-app notice appears without her needing to manually refresh.

**FEAT-08.SPEC-005-AC-09:** Given two different clients each cancel a different booking at effectively the same moment, when both events process, then Talia receives two independent notices, one per booking.

**FEAT-08.SPEC-005-AC-10:** Given Talia taps the "View booking" CTA on a new-booking notice, when the tap registers, then she is taken to that booking's row on FEAT-12's dashboard.

**FEAT-08.SPEC-005-AC-11:** Given Talia has never changed her notification_preferences, when her account is created, then her default is "In-app + text".

**FEAT-08.SPEC-005-AC-12:** Given Support is troubleshooting a delivery issue for one of Talia's notifications, when Support views the Message record, then Support sees delivery status only, never the notification's rendered content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 3 (new booking, client cancel, client reschedule) | 3 |
| Preference States | 4 (in-app only, +text, +email, +text+email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Pro Attention Alert

## Overview

**Name:** Pro Attention Alert
**ID:** FEAT-08.SPEC-006
**Type:** Notification
**Purpose:** Tells the Pro immediately when something needs her attention -- a message delivery failure, a calendar connection needing reconnection, a refund that failed to complete, a card-issuer dispute, a reminder that failed to schedule, or a deposit request that expired unpaid -- so nothing about her business is ever a silent failure she discovers late.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The six named attention conditions: message delivery failure, calendar reconnection needed, refund failure, card-issuer dispute opened, reminder-scheduling failure, deposit request expired unpaid
- The Pro-facing expiry notice content for an unpaid deposit request: this spec is the content owner, FEAT-03.SPEC-007 is the trigger, and FEAT-30.SPEC-013 references this spec rather than defining duplicate content
- In-app, text, and email delivery per the Pro's notification_preferences

**Non-Goals:**
- Routine booking activity (new bookings, client cancellations/reschedules) -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is reserved for conditions requiring action, not routine updates.
- Diagnosing or resolving the underlying condition (why the calendar needs reconnecting, why a refund failed, why a reminder failed to schedule) -- owned by the feature or spec that owns each condition (FEAT-04, FEAT-09/FEAT-30, FEAT-16, FEAT-08.SPEC-007, FEAT-03.SPEC-007); this spec only alerts, per its Trigger section.
- Deciding a card-issuer dispute -- excluded per scope-boundaries.md (SC-17): Chairtime never rules on the dispute; this spec only tells the Pro one exists and where to find the evidence.
- The client-facing side of a refund failure -- covered separately by FEAT-08.SPEC-004, which shows the client only the reassuring "in progress" wording, never the failure itself.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, regardless of the Pro's text/email preference | These are the dashboard's "attention list" items (product-features.md, FEAT-12); the dashboard is the canonical place a Pro checks for anything needing action |
| Text | The Pro's notification_preferences include text for this notification type | An attention-worthy condition is time-sensitive; a text reaches Talia immediately even when she is away from the app |
| Email | The Pro's notification_preferences include email for this notification type | Gives the Pro a durable record of attention items she can search or forward if needed (e.g., for a dispute) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A text or email message fails delivery on all channels attempted | FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Fires after the retry-then-fallback sequence itself cannot deliver the message on any channel | Booking reference, recipient (client), message type |
| Calendar connection needs reconnecting | FEAT-04 (Two-Way Calendar Sync) | Fires when the connection status changes to Needs Reconnection | Calendar connection status, last_successful_sync |
| An automatic or goodwill refund fails to complete | FEAT-09 (Cancellation & No-Show Policy Engine) / FEAT-30 (Pro Booking Management) | Fires when a refund attempt does not succeed and enters a retry state | Deposit Transaction (status, outcome_reason), Booking reference |
| A card-issuer dispute is opened | FEAT-16 (Booking & Payment Activity Record) | Fires when a dispute notice is received for a Deposit Transaction | Booking reference, Deposit Transaction (Disputed status) |
| A Pro-created deposit request's hold expires with the deposit never paid | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Fires when FEAT-03.SPEC-007 transitions the Booking to Expired (unpaid) and the hold's computed expiry has passed with the deposit never paid; FEAT-03.SPEC-007 is the sole writer of that transition and is the trigger only | Booking reference, Booking (service, start_time), Client (name) |
| A confirmed Booking's reminder-scheduling computation cannot run | FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Fires when that automation cannot compute or store a reminder_send_time for a confirmed Booking (e.g., timezone data unavailable at confirmation time) | Booking reference, Booking (start_time, service) |

## Audience and Preferences

**Recipients:** The Pro (Talia) whose account the condition affects (Access Matrix: Booking & Payment, Payouts, Activity Record & Insights = Full/View for the Pro). Platform Operator (Support) has View-only access to delivery status and to the dispute evidence itself (via FEAT-16), consistent with the Access Matrix, but is never a recipient of this notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Attention alert channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | FEAT-27 (Pro Profile & Booking Page Settings) |

In-app is never disable-able, matching FEAT-08.SPEC-005's treatment -- an attention item always appears on the dashboard's attention list regardless of the Pro's text/email choice, since the dashboard is the guaranteed floor for anything needing her action.

**Quiet Hours:** N/A -- product-features.md defines no quiet-hours window for Pro-facing alerts, and an attention-worthy condition (a failed refund, a dispute, a delivery gap) is exactly the kind of time-sensitive information a quiet-hours delay would work against; it is delivered as soon as it is known, on every enabled channel.

## Content Definition

**In-app (message delivery failure):**
- **Title:** Message didn't get through
- **Body:** {client_name}'s {message_type} couldn't be delivered by text, so we sent it by email instead.
- **CTA:** View details -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard), attention list

**In-app (calendar reconnection needed):**
- **Title:** Reconnect your calendar
- **Body:** Your calendar connection needs to be reconnected so new bookings keep syncing.
- **CTA:** Reconnect -- deep-links to FEAT-04

**In-app (refund failure):**
- **Title:** A refund needs your attention
- **Body:** The {deposit_amount} refund for {client_name}'s {appointment_date} booking couldn't complete automatically. We're retrying it.
- **CTA:** View details -- deep-links to FEAT-12, attention list

**In-app (dispute opened):**
- **Title:** A charge is being disputed
- **Body:** {client_name}'s card issuer has opened a dispute on their {appointment_date} deposit. Review the booking timeline to respond.
- **CTA:** View timeline -- deep-links to FEAT-16

**In-app (reminder-scheduling failure):**
- **Title:** A reminder couldn't be scheduled
- **Body:** We couldn't schedule the automatic reminder for {client_name}'s {appointment_date} appointment. You may want to reach out directly before the visit.
- **CTA:** View details -- deep-links to FEAT-12, attention list

**In-app (deposit request expired unpaid):**
- **Title:** Deposit request expired
- **Body:** {client_name}'s deposit request for {appointment_date} expired unpaid. The slot has been released.
- **CTA:** View schedule -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard)

**Text (each condition, one line, matching the in-app body without the CTA link text spelled out as a button):**
- **Body (message delivery failure):** {client_name}'s message couldn't be delivered by text -- sent by email instead. View: {dashboard_link}
- **Body (calendar reconnection):** Your calendar connection needs reconnecting so bookings keep syncing. Reconnect: {calendar_link}
- **Body (refund failure):** A {deposit_amount} refund for {client_name} couldn't complete automatically -- we're retrying it. View: {dashboard_link}
- **Body (dispute opened):** {client_name}'s card issuer opened a dispute on their deposit. View: {timeline_link}
- **Body (reminder-scheduling failure):** We couldn't schedule the reminder for {client_name}'s {appointment_date} appointment. View: {dashboard_link}
- **Body (deposit request expired):** {client_name}'s deposit request for {appointment_date} expired unpaid. The slot has been released. View: {dashboard_link}

**Email (mirrors each text variant with subject "Needs your attention: {condition_summary}" and a button matching the in-app CTA).**

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {client_name} | Client -- name | Riley Chen | Never empty (required field); N/A for the calendar-reconnection variant, which names no client |
| {message_type} | Derived -- the failed Message's type (confirmation / reminder / change notice) | reminder | Never empty -- every failed Message has a type |
| {deposit_amount} | Deposit Transaction -- amount | $40.00 | Never empty -- fixed once at booking |
| {appointment_date} | Booking -- start_time | Oct 4, 2026 | Never empty -- fixed at booking |
| {dashboard_link} / {calendar_link} / {timeline_link} | Derived -- in-product navigation targets to FEAT-12, FEAT-04, and FEAT-16 respectively | (in-product link) | Never empty -- these are fixed navigation destinations, not per-record generated links |
| {condition_summary} | Derived -- a short label per condition ("message delivery", "calendar reconnection", "refund", "dispute", "reminder scheduling", "deposit request expired") | refund | Never empty -- one of exactly six fixed values |

## Delivery Rules

**Batching:** Not batched by default -- each attention condition is its own alert, since each names a distinct action the Pro may need to take. If the same underlying calendar-reconnection condition would otherwise re-alert repeatedly while unresolved, only one active alert exists per condition instance at a time (see Deduplication) rather than a recurring stream.
**Deduplication:** At most one active alert per open condition. A calendar connection that remains in Needs Reconnection state does not re-alert on every subsequent sync attempt -- the existing unresolved in-app item stands until the Pro reconnects (FEAT-04) or the condition otherwise clears. A refund retry that fails again produces no new alert beyond the first, since the same in-app item already reflects "we're retrying it" until it resolves.
**Retry on failure:** This notification's own text/email delivery is retried once and falls back to email, per FEAT-08.SPEC-009, the same as every other message this feature sends -- with one exception: because the in-app channel is always active and never itself the failing channel, an attention alert about a *different* condition (e.g., a refund failure) is never left with no surviving delivery even if its own text/email attempt also fails.
**Expiry:** None -- an attention item never expires undelivered in the sense of being dropped; it remains on the dashboard's attention list until the Pro resolves or acknowledges the underlying condition (reconnects the calendar, the refund retry succeeds, the dispute is addressed).

## Edge Cases

- **The same client has two separate reminders fail delivery on the same day** -- Each failure is a distinct Message and produces its own alert, since each concerns a different booking's delivery gap.
- **A refund failure alert is still open when the retry later succeeds** -- The open attention item is cleared from the dashboard's attention list once the Deposit Transaction reaches Refunded; no further alert fires for that same refund attempt.
- **The Pro has all channels other than in-app disabled** -- She still sees every attention item on her dashboard; nothing needing her action ever depends solely on a muted channel.
- **A dispute is opened on a booking whose client has since been deleted (FEAT-13)** -- The alert still fires, using the de-identified financial record retained for exactly this purpose (XBR-19); the alert states the booking and dispute details without a client name, since the identifying contact details no longer exist.
- **Two different attention conditions occur for the same Pro at effectively the same time (e.g., a delivery failure and a refund failure)** -- Each produces its own independent alert; the two are never merged, since they name different required actions.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | A final delivery failure after retry-then-fallback fires this alert |
| FEAT-04 (Two-Way Calendar Sync) | Triggered by (inbound) | A lapsed connection fires this alert |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | An automatic refund failure fires this alert |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A goodwill refund failure fires this alert |
| FEAT-16 (Booking & Payment Activity Record) | Triggered by (inbound) | A card-issuer dispute notice fires this alert; also where the dispute CTA leads |
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Triggered by (inbound) | A failure to compute or store a reminder_send_time for a confirmed Booking fires this alert |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggered by (inbound) | An unpaid deposit-request hold expiring fires the expiry alert; this spec owns the content |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | References (outbound) | References this spec for the Pro's expiry notice content instead of defining its own |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every alert send is written to the append-only activity record |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Every non-dispute CTA deep-links to the dashboard's attention list |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the text/email sends when enabled |

## Analytics and Success Signals

- **pro_attention_alert_sent** (condition: message_delivery_failure / calendar_reconnection / refund_failure / dispute_opened; channels) -- supports success-metrics.md: "Automatic Refund Correctness"
- **pro_attention_alert_sent** (condition: calendar_reconnection) -- supports success-metrics.md: "Calendar Sync Reliability"
- **pro_attention_alert_resolved** (condition, time_to_resolve) -- N/A -- no Stage 2 metric measures time-to-resolution for attention items; retained to make the product's "never silently dropped" commitment (XBR-17, XBR-10) observable end to end.
- **pro_attention_alert_sent** (condition: reminder_scheduling_failure) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that never got scheduled is visible here rather than only showing up later as a gap in that metric's numerator.
- **pro_attention_alert_sent** (condition: deposit_request_expired; channels) -- supports success-metrics.md: "Pro Change Correctness"

## Acceptance Criteria

**FEAT-08.SPEC-006-AC-01:** Given Riley's reminder text fails delivery and the email fallback is also exhausted per FEAT-08.SPEC-009, when the final failure occurs, then Talia receives an in-app alert "Message didn't get through" naming Riley and the message type.

**FEAT-08.SPEC-006-AC-02:** Given Talia's calendar connection lapses into Needs Reconnection, when the status changes, then she receives an alert on every enabled channel with a "Reconnect" CTA to FEAT-04.

**FEAT-08.SPEC-006-AC-03:** Given an automatic refund for one of Talia's clients fails to complete, when FEAT-09 flags it, then Talia receives an alert stating the refund couldn't complete automatically and that it is being retried.

**FEAT-08.SPEC-006-AC-04:** Given a card-issuer dispute is opened on one of Talia's bookings, when FEAT-16 records the dispute, then Talia receives an alert with a "View timeline" CTA to FEAT-16.

**FEAT-08.SPEC-006-AC-05:** Given Talia's calendar connection remains in Needs Reconnection for several days, when subsequent sync attempts continue to fail, then no additional alert fires beyond the original unresolved item on her dashboard.

**FEAT-08.SPEC-006-AC-06:** Given a previously failed refund retry later succeeds, when the Deposit Transaction reaches Refunded, then the open attention item clears from Talia's dashboard and no further alert fires for that refund.

**FEAT-08.SPEC-006-AC-07:** Given Talia has set notification_preferences to "In-app only", when a dispute is opened, then she sees the in-app alert and receives no text or email.

**FEAT-08.SPEC-006-AC-08:** Given a dispute is opened on a booking whose client record has since been deleted, when the alert fires, then it names the booking and dispute details without a client name, using the retained de-identified financial record.

**FEAT-08.SPEC-006-AC-09:** Given Talia experiences both a delivery failure and a refund failure at effectively the same time, when both conditions are detected, then she receives two separate, independent alerts.

**FEAT-08.SPEC-006-AC-10:** Given this alert's own text send fails, when FEAT-08.SPEC-009's retry-then-fallback runs, then Talia still receives the alert by email, and the in-app copy is unaffected regardless.

**FEAT-08.SPEC-006-AC-11:** Given Riley has two separate reminders fail delivery on the same day for two different bookings, when both failures occur, then Talia receives two distinct alerts, one per booking.

**FEAT-08.SPEC-006-AC-12:** Given Talia taps "View details" on a refund-failure alert, when the tap registers, then she is taken to the attention list on her dashboard (FEAT-12).

**FEAT-08.SPEC-006-AC-13:** Given Platform Operator (Support) is assisting Talia with a dispute, when Support views the booking's activity record, then Support sees the same dispute evidence Talia sees, per Support's View access under XBR-24, and this alert itself is never sent to Support.

**FEAT-08.SPEC-006-AC-14:** Given FEAT-08.SPEC-007's reminder-scheduling computation fails to run for one of Talia's confirmed bookings, when the failure is detected, then Talia receives an attention alert naming the client and appointment date, with a "View details" CTA to her dashboard's attention list.

**FEAT-08.SPEC-006-AC-15:** Given a Pro-created deposit request's hold expires unpaid and FEAT-03.SPEC-007 marks the Booking Expired (unpaid), when the trigger reaches this spec, then Talia receives the in-app alert "Deposit request expired" naming the client and appointment date, plus text and email per her notification_preferences, with a "View schedule" CTA to FEAT-12.

**FEAT-08.SPEC-006-AC-16:** Given Talia has set notification_preferences to "In-app only" and a deposit request expires unpaid, when the alert fires, then she sees the in-app alert and receives no text or email, and Riley receives no message about the expiry from this spec.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 6 (delivery failure, calendar reconnection, refund failure, dispute, reminder-scheduling failure, deposit request expired) | 6 |
| Preference States | 4 (in-app only, +text, +email, +text+email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Reminder Scheduling & Timing Window Enforcement

## Overview

**Name:** Reminder Scheduling & Timing Window Enforcement
**ID:** FEAT-08.SPEC-007
**Type:** Automation
**Purpose:** Computes when each confirmed booking's reminder should fire, keeps every reminder inside the daytime send window (roughly 8am--9pm per XBR-16; platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`) in the Pro's timezone, and suppresses a separate reminder when a booking is made after its own reminder point has already passed.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Computing the reminder send time for every confirmed Booking (default lead time before the appointment)
- Enforcing the daytime-hours send window and its nearest-allowed-time fallback
- Suppressing a reminder for a booking made after its own reminder point would already have passed
- Cancelling a scheduled reminder when the underlying booking is cancelled or rescheduled before the reminder fires

**Non-Goals:**
- The reminder's content and channel -- owned by FEAT-08.SPEC-002 (Appointment Reminder Message); this spec only decides when (or whether) that notification fires.
- Processing a client's reply once the reminder is sent -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing).
- Retrying a failed reminder send -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback); this spec's job ends once it fires the reminder trigger at the correct time.
- Letting the Pro configure the lead time or window per account -- product-features.md defines no such setting for this feature; the lead time and window are single values for every Pro (XBR-16; platform parameter: `reminder-lead-time-days`, platform parameter: `reminder-window-start-hour`, platform parameter: `reminder-window-end-hour`), not Pro-configurable preferences, consistent with scope-boundaries.md's absence of any reminder-timing customization capability.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Booking is confirmed (deposit captured) | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires once per Booking, immediately on confirmation, to compute (or suppress) that booking's reminder time | Booking (start_time), Pro Account (timezone) |
| A confirmed Booking's reminder time is reached | Schedule-based (system clock, evaluated against each Booking's computed reminder_send_time) | Fires when the current time in the Pro's timezone reaches the computed and window-adjusted send time | Booking (service, start_time, deposit_amount, balance_due) |
| A Booking with a scheduled, not-yet-fired reminder is cancelled or rescheduled | FEAT-10 (Client-Initiated Cancel/Reschedule) / FEAT-30 (Pro Booking Management) | Fires whenever a Booking's state changes away from Confirmed, or its start_time changes, before its reminder has fired | Booking (new state or new start_time) |

## Processing Logic

1. On Booking confirmation, read the Booking's start_time and the Pro Account's timezone.
2. Compute the candidate reminder time as start_time minus platform parameter: `reminder-lead-time-days` (BRIEF.md's stated example: two days before the appointment).
3. Compare the candidate reminder time to the current time. If the candidate reminder time has already passed (the booking was made too close to its own appointment for a two-day-ahead reminder to make sense), suppress the reminder entirely for this Booking -- no reminder is ever scheduled, and the confirmation already sent (FEAT-08.SPEC-001) serves as the client's only pre-appointment message.
4. If the candidate reminder time has not yet passed, check whether it falls inside the daytime send window (platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone -- BRIEF.md's stated example: roughly 8am to 9pm).
5. If the candidate time falls outside the window, move it forward to the window's start time on the same day if the candidate was before the window opened, or to the window's start time on the next day if the candidate was after the window closed.
6. Store the resulting reminder_send_time on the Booking.
7. At the stored reminder_send_time, fire the trigger that initiates FEAT-08.SPEC-002 (Appointment Reminder Message), carrying the Booking reference.
8. If the Booking is cancelled, rescheduled, or otherwise leaves the Confirmed state before its stored reminder_send_time is reached, cancel the scheduled reminder -- recompute a fresh reminder_send_time from the new start_time if the booking was rescheduled and remains Confirmed; clear the scheduled reminder entirely if the booking was cancelled.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Reminder scheduled at the default time | Candidate reminder time is within the daytime window | Booking.reminder_send_time set | None directly -- the client sees only the reminder itself when it later fires | FEAT-08.SPEC-002 |
| Reminder scheduled with a window-adjusted time | Candidate reminder time falls outside 8am--9pm | Booking.reminder_send_time set to the nearest allowed time | None directly | FEAT-08.SPEC-002 |
| Reminder suppressed (late booking) | The candidate reminder time has already passed at confirmation time | Booking.reminder_send_time left unset; a suppressed flag recorded for traceability | None -- no reminder is ever shown as pending; the confirmation already sent is the client's only pre-appointment message | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Reminder fires | The system clock reaches Booking.reminder_send_time and the Booking is still Confirmed | None on the Booking itself -- triggers FEAT-08.SPEC-002 | Client receives the reminder message | FEAT-08.SPEC-002 |
| Reminder cancelled (booking cancelled) | Booking leaves Confirmed state before reminder_send_time | Booking.reminder_send_time cleared | None -- no reminder fires for a cancelled booking | FEAT-08.SPEC-002 |
| Reminder rescheduled (booking rescheduled) | Booking's start_time changes while it remains Confirmed | Booking.reminder_send_time recomputed from the new start_time, re-running Steps 2--6 | None directly -- a fresh reminder is scheduled at the newly computed time | FEAT-08.SPEC-002 |
| Automation failure | The scheduling computation itself cannot run (e.g., timezone data unavailable at confirmation time) | No reminder_send_time is set | No client-facing feedback; this automation's own failure directly fires FEAT-08.SPEC-006 (Pro Attention Alert)'s reminder-scheduling-failure trigger, so the Pro sees the gap rather than discovering a missing reminder only when the appointment arrives | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Booking -- start_time, state; Pro Account -- timezone.
**Creates:** None.
**Updates:** Booking -- reminder_send_time (computed field owned by this automation).
**Deletes:** None.

## Business Rules

- XBR-16 governs this spec entirely: reminders go out only between roughly 8am and 9pm in the Pro's timezone (platform parameter: `reminder-window-start-hour` / platform parameter: `reminder-window-end-hour`); confirmations are unaffected by this window (they are handled by FEAT-08.SPEC-001, which is not a discretionary reminder); a booking made after its own reminder point gets no separate reminder.
- The default lead time is platform parameter: `reminder-lead-time-days`, matching BRIEF.md's stated example of two days before the appointment.
- The window-adjustment rule always moves a candidate time forward in time (to later the same day or to the next day), never backward, so a reminder is never sent earlier than intended to fit the window.
- Suppression (Step 3) is evaluated once, at confirmation time, against the current time -- not re-evaluated later, so a booking that was made in time for a reminder is never retroactively suppressed just because the window computation later needs adjustment.

## Edge Cases

- **A booking is made exactly at the boundary of its own reminder point** -- If the two-day-ahead time has not yet passed at the moment of confirmation (even by a small margin), the reminder is scheduled normally; only a candidate time already in the past at confirmation is suppressed.
- **The Pro's timezone changes between booking confirmation and the reminder firing** -- Per XBR-25, the Pro's timezone is a per-account setting; if it changes, the window check re-evaluates against the currently configured timezone for any not-yet-fired reminder, since Booking.reminder_send_time was computed once but the daytime-window boundary is a live, timezone-relative concept the system re-derives at fire time for still-pending reminders.
- **A rescheduled booking's new time also falls after its own new reminder point would have passed** -- The same suppression rule (Step 3) applies to the recomputed time: if the new appointment is too soon for a two-day-ahead reminder, no reminder is scheduled for the rescheduled booking either, and the change notice (FEAT-08.SPEC-004) is the client's only additional message.
- **Concurrent trigger firing (two bookings confirmed at effectively the same time)** -- Each Booking's reminder computation runs independently against its own start_time and the shared Pro Account timezone; neither computation affects the other, since reminder_send_time is a per-booking field.
- **Trigger fires while a previous run is in flight for the same Booking** -- A second confirmation event cannot occur for the same Booking (FEAT-07.SPEC-002 guarantees exactly one capture per booking), so no two scheduling computations ever run concurrently for one Booking; a cancellation/reschedule event for a Booking whose reminder computation is still in flight is queued to apply immediately after the in-flight computation completes, so the final stored reminder_send_time always reflects the Booking's latest state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Booking confirmation starts the reminder-time computation |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule re-triggers cancellation or recomputation |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated cancellation or reschedule re-triggers cancellation or recomputation |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Affects (outbound) | This automation's fired trigger is what starts that notification |
| FEAT-08.SPEC-001 (Booking Confirmation Message) | References (outbound) | Serves as the client's sole pre-appointment message when this automation suppresses a reminder |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | This automation's own failure to compute or store a reminder_send_time fires FEAT-08.SPEC-006's reminder-scheduling-failure alert |

## Analytics and Success Signals

- **reminder_scheduled** (lead_time_days, window_adjusted: yes / no) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_suppressed_late_booking** () -- N/A -- no Stage 2 metric measures suppression frequency directly; retained so the late-booking exception's actual frequency is observable rather than assumed.
- **reminder_schedule_cancelled** (reason: booking_cancelled / booking_rescheduled) -- N/A -- no Stage 2 metric tracks cancelled reminder schedules; this event supports operational visibility into the scheduling pipeline's correctness rather than a named success metric.

## Acceptance Criteria

**FEAT-08.SPEC-007-AC-01:** Given Riley books an appointment 10 days out, when the booking confirms, then the reminder is scheduled for platform parameter: `reminder-lead-time-days` before the appointment, provided that time falls within the daytime window.

**FEAT-08.SPEC-007-AC-02:** Given Riley's computed reminder time falls before platform parameter: `reminder-window-start-hour` in the Pro's timezone, when the scheduling computation runs, then the reminder is moved forward to platform parameter: `reminder-window-start-hour` the same day.

**FEAT-08.SPEC-007-AC-03:** Given Riley's computed reminder time falls after platform parameter: `reminder-window-end-hour` in the Pro's timezone, when the scheduling computation runs, then the reminder is moved forward to platform parameter: `reminder-window-start-hour` the next day.

**FEAT-08.SPEC-007-AC-04:** Given Riley books an appointment sooner than platform parameter: `reminder-lead-time-days` away, when the booking confirms, then no reminder is scheduled, and the confirmation already sent is her only pre-appointment message.

**FEAT-08.SPEC-007-AC-05:** Given Riley's booking has a scheduled reminder that has not yet fired, when Riley cancels the booking, then the scheduled reminder is cleared and never fires.

**FEAT-08.SPEC-007-AC-06:** Given Riley's booking has a scheduled reminder that has not yet fired, when the Pro reschedules the booking to a new time, then the reminder time is recomputed from the new start_time.

**FEAT-08.SPEC-007-AC-07:** Given a rescheduled booking's new appointment is now less than two days out, when the recomputation runs, then the reminder is suppressed for the rescheduled booking as well.

**FEAT-08.SPEC-007-AC-08:** Given a Booking's stored reminder_send_time is reached and the Booking is still Confirmed, when the system clock crosses that time, then FEAT-08.SPEC-002 is triggered for that Booking.

**FEAT-08.SPEC-007-AC-09:** Given two bookings confirm at effectively the same instant, when both scheduling computations run, then each Booking's reminder_send_time is computed independently and correctly.

**FEAT-08.SPEC-007-AC-10:** Given a Booking's reminder computation is in flight when a cancellation event for the same Booking arrives, when the in-flight computation completes, then the cancellation is applied immediately afterward and no reminder fires for the cancelled Booking.

**FEAT-08.SPEC-007-AC-11:** Given the Pro changes her account timezone while a Booking's reminder is still pending, when the window is next evaluated for that pending reminder, then it uses the currently configured timezone.

**FEAT-08.SPEC-007-AC-12:** Given the scheduling computation itself fails to run for a confirmed Booking, when no reminder_send_time is set, then no client-facing error appears, and this automation's failure fires FEAT-08.SPEC-006's reminder-scheduling-failure alert to the Pro.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (confirmation, scheduled fire, cancel/reschedule) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Reminder Reply Routing

## Overview

**Name:** Reminder Reply Routing
**ID:** FEAT-08.SPEC-008
**Type:** Automation
**Purpose:** Processes whichever one-tap reply a client makes on a reminder -- recording "I'll be there" as an acknowledgment, or handing "I need to reschedule" off into the reschedule flow.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Validating the tapped booking-specific link before acting on either reply
- Recording the "I'll be there" acknowledgment on the Booking
- Routing "I need to reschedule" into FEAT-10, scoped to the same booking

**Non-Goals:**
- The reminder message content and its two link URLs -- owned by FEAT-08.SPEC-002; this spec only processes a tap on those links.
- The landing screen shown after acknowledgment -- owned by FEAT-08.SPEC-003.
- The reschedule flow itself once handed off -- owned entirely by FEAT-10 from that point on, per the Brief's Internal Dependency Map ("[routes into] FEAT-10 ... from that point on").
- Minting or validating the manage link's scope and expiry rules in general -- owned by FEAT-08.SPEC-010 and FEAT-06; this spec consumes those rules but does not define them.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps "I'll be there" | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires whenever the "I'll be there" link is opened, regardless of channel (text or email) | Booking reference (from the link), current Booking state |
| Client taps "I need to reschedule" | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires whenever the "I need to reschedule" link is opened, regardless of channel | Booking reference (from the link), current Booking state |

## Processing Logic

1. Receive the tapped link's Booking reference and reply type ("I'll be there" or "I need to reschedule").
2. Validate the link: confirm the referenced Booking exists, its appointment has not yet passed, and the link has not been superseded by a fresher link (e.g., issued after a Pro-initiated reschedule, per FEAT-08.SPEC-010).
3. If the link is invalid or expired, take no action on the Booking and route the client to FEAT-08.SPEC-003's Expired/Invalid Link state.
4. If the link is valid and the reply is "I'll be there", set Booking.attendance_reply to "I'll be there" and record the reply timestamp.
5. If the link is valid and the reply is "I need to reschedule", do not modify Booking.attendance_reply; instead hand off directly into FEAT-10 (Client-Initiated Cancel/Reschedule), carrying the same Booking reference, so the client lands in the reschedule flow for that one booking.
6. For an "I'll be there" reply, forward the client to FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) once Step 4 completes.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Acknowledgment recorded | Valid link, "I'll be there" tapped | Booking.attendance_reply set to "I'll be there" | Client lands on FEAT-08.SPEC-003's Acknowledged state | FEAT-08.SPEC-003, FEAT-12 |
| Routed to reschedule | Valid link, "I need to reschedule" tapped | None on the Booking directly (FEAT-10 owns any subsequent change) | Client lands in FEAT-10's reschedule flow for this booking | FEAT-10 |
| Link invalid or expired | The Booking cannot be found, the appointment has passed, or a fresher link supersedes this one | None | Client lands on FEAT-08.SPEC-003's Expired/Invalid Link state | FEAT-08.SPEC-003 |
| Second reply on an already-acknowledged booking | Client taps "I need to reschedule" after already having tapped "I'll be there" (or the reverse) | The later tap's outcome is evaluated against the Booking's then-current state; if the Booking has already moved out of Confirmed (e.g., already in the reschedule flow), the second tap is routed into FEAT-10 rather than silently ignored | Client is routed into FEAT-10's reschedule flow, or shown FEAT-08.SPEC-003's Acknowledged/Expired state, depending on the Booking's then-current state | FEAT-08.SPEC-003, FEAT-10 |
| Automation failure | The routing step itself cannot complete (e.g., link resolution service unavailable) | None | Client sees FEAT-08.SPEC-003's generic Error state: "Something went wrong loading your confirmation. Try the link again from your reminder message." | FEAT-08.SPEC-003 |

## Data Model

**Reads:** Booking -- state, start_time, attendance_reply; Access Link -- scope, expiry, state (via FEAT-06/FEAT-08.SPEC-010's link resolution).
**Creates:** None.
**Updates:** Booking -- attendance_reply (set only on a valid "I'll be there" reply).
**Deletes:** None.

## Business Rules

- "I'll be there" simply acknowledges -- it never changes the Booking's state, price, or schedule; it is purely informational for the Pro's dashboard (FEAT-12).
- "I need to reschedule" performs no reschedule itself; it only opens the door into FEAT-10, where the client picks a new time subject to that feature's own validation and policy rules (XBR-09).
- Both reply links work identically whether the reminder was received by text or email, per product-features.md's Validation & Limits field ("the one-tap replies are link taps, so a reply works the same by text or email").
- A booking-specific link's scope, expiry, and superseded-by-a-fresher-link rules are owned by FEAT-08.SPEC-010/FEAT-06 (XBR-18); this automation enforces those rules at the moment of the tap but does not define them.

## Edge Cases

- **Client taps "I'll be there" twice from the same link** -- The second tap is a no-op that re-confirms the same attendance_reply value; Booking.attendance_reply is not re-timestamped, and the client sees the same Acknowledged screen.
- **Client taps "I need to reschedule" after already tapping "I'll be there" on the same booking** -- The tap is still honored: the client is routed into FEAT-10, since wanting to reschedule after all is a legitimate, later change of mind that the flat "I'll be there" flag does not block.
- **The appointment passes between the reminder being sent and either reply being tapped** -- Per XBR-18, the booking-specific link has expired; both reply types route to FEAT-08.SPEC-003's Expired/Invalid Link state rather than acting on a stale reply.
- **Concurrent trigger firing (the client taps both links in different browser tabs at nearly the same time)** -- Whichever tap's validation completes first determines the outcome; the Booking's Contention resolution (reject-with-refresh, feature-dependency-map.md) means the second tap is evaluated against the Booking's now-updated state, so it either succeeds consistently (if compatible) or is redirected to reflect the Booking's current state rather than silently overwriting the first outcome.
- **Trigger fires while a previous run is in flight for the same Booking** -- A second tap on the same link while the first tap's processing is still resolving is queued to evaluate against the Booking's post-processing state, preventing two conflicting writes to attendance_reply from racing each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Both reply links deep-link into this automation |
| FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) | Affects (outbound) | Receives the client after an acknowledgment or an invalid-link outcome |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Affects (outbound) | Receives the client after a reschedule-reply outcome |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Owns the link scope/expiry rules this automation enforces at tap time |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Displays the "I'll be there" status once recorded |

## Analytics and Success Signals

- **reminder_reply_confirmed** () -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_reschedule_requested** () -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_invalid_link** (reason: expired / superseded / not_found) -- N/A -- no Stage 2 metric measures invalid-link tap frequency; retained to keep the link-expiry design's real-world frequency observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-008-AC-01:** Given Riley taps "I'll be there" on a valid, unexpired link, when the automation processes the tap, then Booking.attendance_reply is set to "I'll be there" and she is forwarded to FEAT-08.SPEC-003.

**FEAT-08.SPEC-008-AC-02:** Given Riley taps "I need to reschedule" on a valid, unexpired link, when the automation processes the tap, then she is routed into FEAT-10's reschedule flow for that same booking, and Booking.attendance_reply is unchanged.

**FEAT-08.SPEC-008-AC-03:** Given Riley's appointment has already passed, when she taps either reply link, then she is routed to FEAT-08.SPEC-003's Expired/Invalid Link state and no Booking field changes.

**FEAT-08.SPEC-008-AC-04:** Given Riley taps "I'll be there" twice from the same link, when the second tap is processed, then it is a no-op and she sees the same Acknowledged screen without a new timestamp being recorded.

**FEAT-08.SPEC-008-AC-05:** Given Riley already tapped "I'll be there" and later taps "I need to reschedule" on the same reminder, when the second tap is processed, then she is routed into FEAT-10's reschedule flow, honoring her later choice.

**FEAT-08.SPEC-008-AC-06:** Given the Pro reschedules Riley's booking after the reminder was sent, when Riley later taps either link from the superseded reminder, then she is routed to FEAT-08.SPEC-003's Expired/Invalid Link state, since a fresher link now governs the booking.

**FEAT-08.SPEC-008-AC-07:** Given Riley received her reminder by email rather than text, when she taps either reply link, then the outcome is identical to a text-received reminder.

**FEAT-08.SPEC-008-AC-08:** Given Riley taps both reply links in two browser tabs at nearly the same time, when both taps are processed, then the second tap is evaluated against the Booking's state as updated by the first, rather than racing it.

**FEAT-08.SPEC-008-AC-09:** Given the link-resolution capability is temporarily unavailable when Riley taps a reply link, when the automation cannot complete, then she sees FEAT-08.SPEC-003's generic Error state.

**FEAT-08.SPEC-008-AC-10:** Given Riley's "I'll be there" acknowledgment is recorded, when the Pro next views her dashboard (FEAT-12), then the booking shows the "I'll be there" status.

**FEAT-08.SPEC-008-AC-11:** Given a second tap arrives on the same link while the first tap's processing is still in flight, when the first tap's processing completes, then the second tap evaluates against the Booking's post-processing state rather than writing a conflicting attendance_reply concurrently.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (I'll be there, I need to reschedule) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Message Delivery Retry & Fallback

## Overview

**Name:** Message Delivery Retry & Fallback
**ID:** FEAT-08.SPEC-009
**Type:** Automation
**Purpose:** When a text fails to deliver, retries it once and then falls back to email, flagging the delivery gap on the Pro's dashboard so no message this feature sends is ever silently dropped.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Retry and fallback handling for every Message this feature's Notification specs create (confirmation, reminder, change notice, Pro activity notification, Pro attention alert)
- Flagging an unresolved delivery gap to the Pro

**Non-Goals:**
- Deciding the initial channel (text vs. email) for a send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule); this spec only handles what happens after a chosen channel's send attempt fails.
- The text and email sends themselves -- owned by FEAT-08.SPEC-012 and FEAT-08.SPEC-013 (the Integration specs); this spec orchestrates retry/fallback around their reported outcomes, not the sends themselves.
- Retrying an email delivery failure with a further fallback channel -- product-features.md and the Brief's Alternate flow describe only a text-then-email fallback chain; there is no channel beyond email to fall back to, so an email failure is handled as a final failure per Outcome Definitions below, not a retry loop.
- Composing the delivery-gap alert's content -- owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec only triggers it.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A text send is reported Failed | FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Fires whenever the text capability reports a Failed delivery status for a Message this feature sent | Message (type, recipient, content_summary), Booking or Pro Account reference |
| An email send (as the fallback) is reported Failed | FEAT-08.SPEC-013 (Transactional Email Capability) | Fires whenever the email fallback itself is reported Failed, marking the end of the retry/fallback chain | Message (type, recipient, content_summary) |

## Processing Logic

1. Receive the Failed delivery-status event for a text Message.
2. Retry the same text send once, through FEAT-08.SPEC-012, up to platform parameter: `message-delivery-retry-count` times (BRIEF.md's stated behavior: retries once).
3. If the retried text succeeds (reported Sent or Delivered), the delivery is complete; no fallback or alert is needed.
4. If the retried text also fails, create a new Message record on the email channel with the same content (per the Message entity's lifecycle note: a fallback creates a second, immutable Message record rather than mutating the failed one) and send it through FEAT-08.SPEC-013.
5. If the email fallback succeeds, the delivery is complete on the fallback channel; flag the gap to the Pro (Step 6) regardless, since a text-to-email fallback is itself information the Pro should see, even though the client did receive the message.
6. Trigger FEAT-08.SPEC-006 (Pro Attention Alert) to flag the delivery gap, referencing the affected Booking or Pro notification and the fact that a fallback to email was used.
7. If the email fallback also fails, this is a final, unresolved delivery failure: trigger FEAT-08.SPEC-006 with the escalated condition (no channel succeeded), so the Pro knows the client may not have received the message at all.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Retry succeeds | The retried text send is reported Sent/Delivered | Message.delivery_status updated to Sent/Delivered | None -- delivery completed on the original channel, no gap to flag | FEAT-08.SPEC-012 |
| Fallback succeeds | The retried text fails; the email fallback succeeds | A new Message record created (channel: email, delivery_status: Sent/Delivered); the original text Message's delivery_status remains Failed as its own immutable record | Client receives the message by email; Pro sees a delivery-gap flag (fallback used) | FEAT-08.SPEC-006, FEAT-08.SPEC-013 |
| Both channels fail | The retried text fails and the email fallback also fails | Both Message records (text, email) show delivery_status Failed | No message reaches the client on this attempt; Pro sees an escalated delivery-gap flag | FEAT-08.SPEC-006 |
| No action needed | The original text send succeeds on first attempt | None (this automation is not triggered) | N/A | -- |
| Automation failure | The retry/fallback orchestration itself cannot run (e.g., the automation's own processing is unavailable) | No retry or fallback attempted | The original Failed status stands; this is itself indistinguishable from "both channels fail" from the Pro's perspective, and results in the same escalated flag once detected | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Message -- type, channel, recipient, content_summary, delivery_status.
**Creates:** Message -- a new record on the fallback (email) channel when the text retry fails, per the Message entity's immutable-per-attempt lifecycle.
**Updates:** Message -- delivery_status on the original (text) Message record, reflecting the retry's outcome.
**Deletes:** None -- Messages are never deleted (Message entity lifecycle: immutable once sent).

## Business Rules

- XBR-17 governs this spec entirely: a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard and recorded in the Booking timeline (FEAT-16) -- never silently dropped.
- This automation applies uniformly to every Notification spec in this feature (FEAT-08.SPEC-001, 002, 004, 005, 006) -- it is not specific to client-directed messages; a Pro notification's own text failing follows the identical retry-then-fallback path.
- The retry count is fixed at platform parameter: `message-delivery-retry-count` -- this is not a per-message or per-client configurable value.
- A fallback creates a new Message record rather than mutating the failed one, preserving each channel attempt as its own immutable record, consistent with the append-only nature FEAT-16 relies on for dispute evidence.

## Edge Cases

- **The retry succeeds on a message whose content has since become stale (e.g., the booking was cancelled between the original failed attempt and the retry)** -- The retry still sends the message as originally composed; a subsequent, distinct change notice (FEAT-08.SPEC-004) informs the client of the cancellation separately, since this automation's job is delivery of the message it was given, not re-validating its content against the booking's latest state.
- **Both the text and email capabilities are down at the same time** -- Both attempts fail; the escalated "both channels fail" outcome fires, and the Pro's alert is retried on its own channels per FEAT-08.SPEC-006's own delivery rules, so the escalation itself is not lost even during a capability-wide outage.
- **The same Message somehow reports Failed twice (a duplicate delivery-status event)** -- The second Failed report for a Message already in a fallback or retry-complete state is a no-op; this automation does not retry or fall back a second time for the same original send.
- **Concurrent trigger firing (two different Messages for the same client fail delivery at the same time)** -- Each Message's retry/fallback runs independently; a text confirmation failing and a text reminder failing for the same client at the same moment each produce their own retry, fallback, and Pro alert.
- **Trigger fires while a previous run is in flight for the same Message** -- A duplicate Failed event for a Message whose retry is already in progress is ignored; only the original triggering event drives the retry-then-fallback sequence for that Message.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggered by (inbound) | A Failed text delivery-status event fires this automation |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggered by (inbound); Triggers (outbound) | A Failed email (fallback) event also fires this automation; a successful retry-triggered fallback send uses this capability |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006 | Affects (outbound) | Any Message these specs create is subject to this automation's retry/fallback handling |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Every fallback used or unresolved failure flags the Pro |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The delivery gap appears on the dashboard's attention list |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every retry and fallback event is recorded in the append-only activity record |

## Analytics and Success Signals

- **message_delivery_retry_attempted** (original_channel: text) -- N/A -- no Stage 2 metric directly measures retry frequency; retained to make the reliability commitment (XBR-17) operationally observable.
- **message_delivery_fallback_used** (original_channel: text; fallback_channel: email) -- N/A -- no Stage 2 metric names messaging delivery reliability directly; retained because a silent delivery gap would otherwise undermine "Reminder Response Rate" and "Booking Completion Speed" without either metric being able to detect why.
- **message_delivery_failed_both_channels** () -- N/A -- no Stage 2 metric measures total delivery failure; retained as the operational signal behind XBR-17's "never silently dropped" guarantee.

## Acceptance Criteria

**FEAT-08.SPEC-009-AC-01:** Given a confirmation text to Riley fails delivery, when this automation retries it once, then the retry attempt is made through FEAT-08.SPEC-012 before any fallback occurs.

**FEAT-08.SPEC-009-AC-02:** Given the retried text succeeds, when the retry's delivery status reports Delivered, then no fallback is used and no Pro alert fires.

**FEAT-08.SPEC-009-AC-03:** Given the retried text also fails, when the fallback step runs, then a new Message record is created on the email channel and sent through FEAT-08.SPEC-013, and Riley receives the message by email.

**FEAT-08.SPEC-009-AC-04:** Given the email fallback succeeds after a failed text and retry, when delivery completes, then Talia still receives a delivery-gap alert (FEAT-08.SPEC-006) noting the fallback was used, even though Riley did receive the message.

**FEAT-08.SPEC-009-AC-05:** Given both the retried text and the email fallback fail, when both failures are confirmed, then Talia receives an escalated attention alert stating no channel succeeded.

**FEAT-08.SPEC-009-AC-06:** Given a Pro notification (not a client message) fails delivery by text, when this automation processes it, then the identical retry-then-fallback path applies as for a client-directed message.

**FEAT-08.SPEC-009-AC-07:** Given the same failed Message reports a Failed status twice, when the second report arrives, then it is treated as a no-op and no second retry or fallback is attempted.

**FEAT-08.SPEC-009-AC-08:** Given the booking a failed message concerns is cancelled between the original failure and the retry, when the retry sends, then it still delivers the originally composed content, and the cancellation is communicated separately via FEAT-08.SPEC-004.

**FEAT-08.SPEC-009-AC-09:** Given both the text and email capabilities are unavailable at the same time, when both attempts fail, then the Pro's escalated alert is still delivered through its own retry rules (FEAT-08.SPEC-006), not lost to the same outage.

**FEAT-08.SPEC-009-AC-10:** Given two different Messages for the same client fail delivery at effectively the same time, when both are processed, then each retries and falls back independently.

**FEAT-08.SPEC-009-AC-11:** Given a fallback email Message is created after a failed text, when the original text Message record is inspected later, then it still shows delivery_status Failed as its own immutable record, distinct from the new email Message.

**FEAT-08.SPEC-009-AC-12:** Given Support views the activity record for a booking whose message fell back to email, when Support inspects the record, then both the failed text attempt and the successful email fallback appear as separate entries, per FEAT-16's append-only record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (text failed, email fallback failed) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Booking-Specific Manage Link Issuance

## Overview

**Name:** Booking-Specific Manage Link Issuance
**ID:** FEAT-08.SPEC-010
**Type:** Automation
**Purpose:** Mints the booking-specific manage link carried in every confirmation and reminder, scoped to exactly one booking, and reissues a fresh one whenever the Pro reschedules that booking.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Creating a booking-specific Access Link at confirmation send and at reminder send
- Reissuing a fresh link after a Pro-initiated reschedule
- Enforcing the link's single-booking scope and its automatic expiry once the appointment passes

**Non-Goals:**
- Resolving or redeeming a tapped link (checking Issued/Used/Expired state, opening the actual booking view) -- owned by FEAT-06 (Client Booking Identity); this spec only mints the link, per the Entity-Lifecycle Coverage Matrix's explicit division of ownership.
- The on-demand "my bookings" link a returning client requests -- owned by FEAT-06; that link has a different scope (all of a client's bookings with one Pro) and a different expiry (30 minutes, single-use), distinct from this spec's booking-specific, until-appointment-passes link.
- Issuing a fresh link after a client-initiated reschedule -- product-features.md and the Brief's Entity-Lifecycle Coverage Matrix name only the Pro-initiated reschedule (via FEAT-30) as triggering reissuance; a client-initiated reschedule (FEAT-10) is itself reached through an already-valid link and does not need a new one issued mid-flow.
- Deciding which message a link is embedded in -- owned by FEAT-08.SPEC-001, FEAT-08.SPEC-002, and FEAT-08.SPEC-004, each of which requests a link from this automation when composing their content.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A confirmation is about to be sent | FEAT-08.SPEC-001 (Booking Confirmation Message) | Fires immediately before the confirmation's content is composed | Booking reference |
| A reminder is about to be sent | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires immediately before the reminder's content is composed | Booking reference |
| A Pro-initiated reschedule occurs | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) | Fires whenever the Pro reschedules a client's booking to a new time | Booking reference (with updated start_time) |

## Processing Logic

1. Receive a request to mint (or reissue) a manage link for a specific Booking.
2. Check whether an active, unexpired Access Link already exists for this Booking (for example, a link already issued at confirmation time, still valid when the reminder later needs one).
3. If an active link already exists and the request is not a reschedule-triggered reissuance, reuse the existing link rather than minting a duplicate -- the confirmation and reminder for the same still-unrescheduled booking share one link.
4. If no active link exists, or the request is a reschedule-triggered reissuance, create a new Access Link scoped to exactly this one Booking, with its expiry set to the Booking's (possibly newly rescheduled) appointment start_time.
5. On a reschedule-triggered reissuance, mark any prior link for this Booking as superseded, so it no longer resolves (per FEAT-08.SPEC-008's Edge Cases, a tap on a superseded link routes to the Expired/Invalid Link state).
6. Return the resulting link to the requesting spec for embedding in its message content.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link created (first issuance) | No active link exists for this Booking | Access Link created (scope: this Booking; expiry: appointment start_time; state: Issued) | None directly -- the link appears embedded in the requesting message | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Existing link reused | An active, unexpired link for this Booking already exists and no reschedule occurred | None -- the existing Access Link record is returned unchanged | None directly | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Link reissued (Pro reschedule) | FEAT-30 reports a Pro-initiated reschedule for this Booking | The prior Access Link's state is superseded; a new Access Link is created scoped to the same Booking with the new appointment's expiry | The client's next message (the change notice, FEAT-08.SPEC-004) carries the fresh link; the old link stops working | FEAT-08.SPEC-004, FEAT-08.SPEC-008 |
| Automation failure | Link issuance itself fails (e.g., the underlying link-generation step is unavailable) | No Access Link is created or reissued | The requesting message's send is held, per FEAT-08.SPEC-001/002's Edge Cases, and treated as a send failure under FEAT-08.SPEC-009's retry path once a link becomes available | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-009 |

## Data Model

**Reads:** Booking -- start_time, state.
**Creates:** Access Link -- scope (one Booking), expiry (until the appointment passes), state (Issued).
**Updates:** Access Link -- state set to superseded on a reschedule-triggered reissuance (the prior link for this Booking).
**Deletes:** None -- a superseded or expired link is never deleted; it simply stops resolving, consistent with the Access Link entity's "expires automatically" lifecycle (no delete/archive action exists).

## Business Rules

- XBR-18 governs this spec: booking-specific links stop working once the appointment passes, and a Pro reschedule issues a fresh manage link.
- Exactly one active manage link exists per Booking at any time (excluding the brief moment of transition during a reschedule-triggered reissuance) -- the confirmation and reminder for an unrescheduled booking intentionally share the same link rather than each minting its own, so a client using an earlier message's link after receiving a later one still reaches the same, current booking.
- A booking-specific link's scope is exactly one Booking -- it never grants access to any other booking, even another booking by the same Client with the same Pro, per XBR-18's "access links open only that client's bookings with that Pro" read together with this spec's explicit single-booking scope.
- This link is distinct in kind from FEAT-06's on-demand "my bookings" link: no single-use restriction applies here (Step 3's reuse behavior depends on this), since a client legitimately needs to tap the same link multiple times (once to acknowledge a reminder, again later to check details) without it burning out.

## Edge Cases

- **A reminder is about to be sent for a booking whose confirmation link is still active** -- The reminder reuses the existing link (Step 3); no second link is minted for the same still-current booking.
- **The Pro reschedules a booking twice in quick succession** -- Each reschedule triggers its own reissuance; only the link from the most recent reschedule remains active, and every earlier link (including the original) is superseded.
- **A client taps an old, superseded link after a Pro reschedule** -- Per FEAT-08.SPEC-008's Edge Cases, the tap resolves to the Expired/Invalid Link state, directing the client to their latest message.
- **Concurrent trigger firing (the confirmation and reminder both request a link for the same booking at effectively the same time -- possible only in the rare case of a very-soon appointment)** -- The check-then-create sequence (Steps 2--4) ensures only one Access Link is ultimately created for the booking; the second request's check finds the first request's just-created link and reuses it rather than creating a duplicate.
- **Trigger fires while a previous run is in flight for the same Booking (a reschedule reissuance overlaps with an in-flight confirmation link request)** -- The reissuance is applied after the in-flight request completes, so the confirmation always uses a link that is not immediately stale; if the reschedule's new start_time is already in effect by the time the confirmation's link is embedded, the confirmation uses the freshly reissued link rather than a moment-old superseded one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Requests a link to embed |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Requests a link to embed (two reply-action variants of the same link) |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated reschedule ("Reschedule committed" outcome) triggers reissuance of a fresh manage link (XBR-18) |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Affects (outbound) | Carries the freshly reissued link after a reschedule |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | References (outbound) | Enforces this spec's scope/expiry rules at the moment of a reply tap |
| FEAT-06 (Client Booking Identity) | References (outbound) | Owns resolving and redeeming the link this spec mints |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | Reads the link's Booking scope to identify the booking when a client acts through it |

## Analytics and Success Signals

- **manage_link_issued** (trigger: confirmation / reminder / reschedule_reissuance) -- N/A -- no Stage 2 metric measures link issuance volume directly; it is an enabling mechanism behind "Self-Service Access Success", which measures the client's use of the link, not its minting.
- **manage_link_superseded** (reason: pro_reschedule) -- N/A -- no Stage 2 metric tracks link supersession; retained so the reschedule-driven reissuance path (XBR-18) is observable end to end.

## Acceptance Criteria

**FEAT-08.SPEC-010-AC-01:** Given Riley's booking has no active manage link, when her confirmation is about to send, then a new Access Link is created scoped to that one booking, expiring when the appointment passes.

**FEAT-08.SPEC-010-AC-02:** Given Riley's confirmation link is still active when her reminder is about to send, when the reminder is composed, then it reuses the existing link rather than minting a new one.

**FEAT-08.SPEC-010-AC-03:** Given the Pro reschedules Riley's booking, when the reschedule completes, then the prior link is superseded and a new Access Link is created scoped to the same booking with the new appointment's expiry.

**FEAT-08.SPEC-010-AC-04:** Given Riley taps her original manage link after the Pro rescheduled her booking, when the link resolves, then she reaches FEAT-08.SPEC-003's (or FEAT-06's) Expired/Invalid state rather than the current booking.

**FEAT-08.SPEC-010-AC-05:** Given Riley's booking-specific link is scoped only to her one booking, when the link is inspected by any spec, then it never resolves to any other booking, even another one of Riley's own bookings with the same Pro.

**FEAT-08.SPEC-010-AC-06:** Given Riley's appointment has passed, when anyone taps her booking-specific link, then it no longer resolves, per its automatic expiry.

**FEAT-08.SPEC-010-AC-07:** Given the Pro reschedules the same booking twice in quick succession, when both reschedules complete, then only the link from the second (most recent) reschedule remains active.

**FEAT-08.SPEC-010-AC-08:** Given link issuance itself fails at confirmation time, when the confirmation attempts to compose, then the send is held and treated as a delivery failure once a link becomes available, per FEAT-08.SPEC-009.

**FEAT-08.SPEC-010-AC-09:** Given both a confirmation and a reminder request a link for the same booking at effectively the same time, when both requests are processed, then only one Access Link is created and both messages reference it.

**FEAT-08.SPEC-010-AC-10:** Given Riley taps her manage link twice (once to acknowledge a reminder, once later to check details), when each tap resolves, then both succeed against the same still-active link, since this link is not single-use.

**FEAT-08.SPEC-010-AC-11:** Given a Pro-initiated reschedule reissuance overlaps with an in-flight confirmation link request for the same booking, when both complete, then the confirmation ultimately embeds the freshly reissued link, not a moment-old superseded one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (confirmation, reminder, Pro reschedule) | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Messaging Consent & Channel Selection Rule

## Overview

**Name:** Messaging Consent & Channel Selection Rule
**ID:** FEAT-08.SPEC-011
**Type:** Logic/Rule
**Purpose:** Decides text vs. email for every outbound client-directed message this feature sends, based on the client's active Messaging Consent, honoring a revoke on the very next message and requiring fresh consent after a phone number change.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- The channel-selection decision (text vs. email) applied before every client-directed send in this feature
- Re-checking consent at send time so a same-session revoke is honored immediately
- The fresh-consent requirement after a client's phone number changes

**Non-Goals:**
- Capturing consent for the first time -- owned by FEAT-05 (Public Booking Page & Booking Flow), which writes the initial Messaging Consent record at booking.
- Processing a revoke itself (a STOP reply or an opt-out link tap) -- owned by FEAT-14 (Messaging Consent Management), which updates the Messaging Consent record; this spec only reads the resulting state.
- Channel selection for Pro-directed notifications (FEAT-08.SPEC-005, FEAT-08.SPEC-006) -- those are governed by the Pro's own notification_preferences (FEAT-27), a distinct preference from client texting consent, since the Pro is never subject to SMS-consent rules for her own account's alerts.
- WhatsApp as a channel option -- deferred to a later phase per scope-boundaries.md's Relevant Deferral Notes; this spec's channel decision is binary (text or email). WhatsApp Reminders (FEAT-26) extends it by running its own eligibility rule, FEAT-26.SPEC-004, before this spec; this spec is reached only when FEAT-26.SPEC-004 finds the send not WhatsApp-eligible.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp from Later) |
| state | enum | Granted / Revoked / Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-08.SPEC-001 | Booking Confirmation Message | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-002 | Appointment Reminder Message | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | Channel decision evaluated immediately before send |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | Consults this spec's outcome indirectly: a text failure's fallback to email is a delivery-failure path, distinct from this spec's consent-driven channel choice, but both ultimately route through the same email capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | Must be one of the product's defined channels (text; WhatsApp from Later) | Always | On read, before channel decision | N/A -- this field is written by FEAT-05/FEAT-14, not entered directly in this spec's flow | No |
| state | Must be one of Granted / Revoked / Re-granted | Always | On read, before channel decision | N/A -- written by FEAT-05/FEAT-06/FEAT-14 | No |
| timestamp | No validation beyond data type -- system-set, not user-entered in this spec's flow | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- captured verbatim by FEAT-05/FEAT-06 as evidence | Always | -- | -- | -- |
| phone_number | Must match the Client's current phone number for consent to be treated as active for texting | Always | On every send, before selecting text as the channel | N/A -- a mismatch is a silent routing decision (email is used), not a user-facing validation error, since no user is filling out a form at this point | Yes (blocks text; does not block the send itself, which proceeds by email) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Consent-to-channel gate | state, phone_number | Text is used only when state is Granted or Re-granted AND phone_number matches the Client's current phone_number; otherwise email is used | N/A -- this is a routing decision with no user-facing error; the client simply receives the message on the resulting channel |
| Stale-state resolution | state, timestamp | When two consent-state changes could apply (e.g., a STOP reply and an in-app re-grant arriving close together, per the Messaging Consent entity's Contention note), the most recent explicit client action by timestamp wins | N/A -- resolved automatically; if the state is genuinely uncertain, the no-text state applies (FEAT-14's Error state) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read Messaging Consent state (to decide a send's channel) | The Pro (Talia) | Read-only, own clients' consent only, for the Pro's own visibility of textability (per FEAT-12) | -- |
| Read Messaging Consent state (to decide a send's channel) | The Client (Riley) | This spec itself does not surface consent state to the Client directly; the Client's own view/change of their consent is FEAT-06/FEAT-14's screen, not this rule | -- |
| Read Messaging Consent state (to decide a send's channel) | Platform Operator (Support) | View-only, for troubleshooting a delivery issue -- Support never changes consent | -- |
| Change Messaging Consent state (grant, revoke, re-grant) | The Client (Riley) | Own-only -- only the Client whose consent it is may change it (Access Matrix: Messaging & Consent = Own-only for the Client); performed via FEAT-05, FEAT-06, or FEAT-14, never through this spec | Control not exposed anywhere in this spec's flow -- consent changes never happen as a side effect of a message send |
| Change Messaging Consent state (grant, revoke, re-grant) | The Pro (Talia) | Never -- the Pro can see but never override a client's consent (Access Matrix note: "the Pro can see but never override a client's texting consent") | The consent-state field is read-only wherever the Pro views it (FEAT-12); no control to change it is ever shown to the Pro |
| Change Messaging Consent state (grant, revoke, re-grant) | Platform Operator (Support) | Never | No control exists for Support to change consent under any circumstance |
| Override this spec's channel decision for a single send | The Pro (Talia) | Never -- the Pro cannot force a text send against a client's revoked consent, even for her own client | No override control exists anywhere in the product; the channel decision is fully automatic |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Selected channel for a given send | Derived: text if Messaging Consent.state is Granted/Re-granted and phone_number matches the Client's current phone_number; email otherwise | On every client-directed send, evaluated fresh each time (never cached from a prior send) | No -- this is a system-computed routing decision with no user-facing override |

## Business Rules

- XBR-15 governs this spec entirely: no text is sent without active Messaging Consent for that client and Pro; otherwise email is used; a revoke is honored on the very next message; a changed phone number requires fresh consent.
- The channel decision is re-evaluated at send time, not cached from booking time or from a prior message -- this is the mechanism by which "a revoke is honored on the very next message" is satisfied: there is no message in flight that can still go out by text after a revoke, because the decision is made fresh immediately before each send.
- A client who changes their phone number (recorded via FEAT-13, per feature-dependency-map.md's Client Contention note: "a Pro phone-number change invalidates access links and requires fresh texting consent") has their existing Messaging Consent treated as not applicable to the new number until fresh consent is captured for it; every send in the interim routes to email.
- FEAT-14.SPEC-007 (Textability Determination Rule) is the authority for whether a client is textable (XBR-15); this spec's consent-to-channel gate applies that determination at send time and never redefines it.
- FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) is the authority for resolving near-simultaneous consent changes; this spec's stale-state resolution follows it and defaults to no-text when the state is uncertain.
- FEAT-14.SPEC-008 (Phone-Number-Change Consent Invalidation Rule) is the authority for treating consent as not applicable after a phone number change; this spec's phone_number match check enforces it on every send.
- FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) runs before this spec for every client-directed send in FEAT-08.SPEC-001, 002, and 004; a send it finds WhatsApp-eligible never reaches this spec, and any other send proceeds here unchanged.
- Every other Notification spec in this feature (FEAT-08.SPEC-001, 002, 004) defers to this spec's decision rather than each implementing its own channel logic, per the Brief's Shared Validation section.

## Edge Cases

- **A STOP reply and an in-app re-grant arrive within the same second** -- Per the Messaging Consent entity's Contention resolution, the most recent explicit client action by timestamp wins; if the timestamps are genuinely indistinguishable, the no-text (email) state applies, since an uncertain consent state must never risk an unwanted text.
- **The client's phone number changes mid-session while a message is queued to send** -- The channel decision, evaluated at send time (not queue time), correctly routes to email once the number-mismatch condition is detected, even if the message was queued while the old number was still valid.
- **A client has Messaging Consent Granted but no phone number on file (a data inconsistency that should not occur given FEAT-05's capture flow)** -- The phone_number match condition cannot be satisfied, so email is used; the send is never blocked outright, since email is always available as the client provided one at booking (product-features.md: "provide an email address when declining texts").
- **A client revokes consent, and a message that was already in the middle of a text-send attempt when the revoke landed** -- The already-initiated send completes on the channel it started on (this spec governs the decision at the moment of initiating a send, not a mid-flight cancellation of an attempt already underway); the very next message after the revoke is what is guaranteed to honor the new state.
- **Consent state is Re-granted after a prior Revoked state** -- Re-granted is treated identically to Granted for this spec's channel decision -- text becomes available again immediately, consistent with the entity's three-state model treating Re-granted as an active-consent state, not a distinct tier.

## Acceptance Criteria

**FEAT-08.SPEC-011-AC-01:** Given Riley has active Messaging Consent (Granted) and her phone number on file matches her consent record, when any client-directed message in this feature is about to send, then text is selected as the channel.

**FEAT-08.SPEC-011-AC-02:** Given Riley has Revoked her Messaging Consent, when any client-directed message is about to send, then email is selected as the channel.

**FEAT-08.SPEC-011-AC-03:** Given Riley revokes her consent between her confirmation (sent by text) and her reminder, when the reminder is about to send, then it is sent by email, honoring the revoke on the very next message.

**FEAT-08.SPEC-011-AC-04:** Given Riley's phone number changes and fresh consent has not yet been captured for the new number, when a message is about to send, then it routes to email, never to the old or unconsented number.

**FEAT-08.SPEC-011-AC-05:** Given Riley's consent state transitions from Revoked to Re-granted, when the next message is about to send, then text is selected as the channel, since Re-granted is treated as active consent.

**FEAT-08.SPEC-011-AC-06:** Given a STOP reply and an in-app re-grant for the same client arrive within the same second with indistinguishable timestamps, when the channel decision is evaluated, then email is used, since an uncertain state defaults to no-text.

**FEAT-08.SPEC-011-AC-07:** Given Talia (the Pro) views a client's textability on her dashboard, when she looks for a way to override a revoked consent, then no such control exists anywhere in the product.

**FEAT-08.SPEC-011-AC-08:** Given Support is troubleshooting a delivery issue, when Support views the Messaging Consent record, then Support sees the state read-only and has no control to change it.

**FEAT-08.SPEC-011-AC-09:** Given Riley has Granted consent but no phone number on file due to a data inconsistency, when a message is about to send, then it routes to email rather than being blocked outright.

**FEAT-08.SPEC-011-AC-10:** Given every Notification spec in this feature (FEAT-08.SPEC-001, 002, 004) needs to choose a channel, when each composes its send, then each defers to this spec's decision rather than implementing separate channel logic.

**FEAT-08.SPEC-011-AC-11:** Given a text send to Riley has already begun processing at the moment her revoke is recorded, when that specific send completes, then it completes on the channel it started on; the very next message after the revoke is the one guaranteed to honor the new state.

**FEAT-08.SPEC-011-AC-12:** Given Riley (the Client) wants to change her own consent, when she looks for how to do so, then the control exists only in FEAT-06/FEAT-14, never as a side effect of viewing or receiving a message under this spec.

**FEAT-08.SPEC-011-AC-13:** Given a Pro-directed notification (FEAT-08.SPEC-005 or FEAT-08.SPEC-006) needs a channel decision, when it evaluates channels, then it uses the Pro's own notification_preferences (FEAT-27), not this spec's Messaging Consent rule.

**FEAT-08.SPEC-011-AC-14:** Given Riley's consent state is Granted but FEAT-14.SPEC-007 determines she is not textable (for example, her phone number changed per FEAT-14.SPEC-008 and fresh consent is not captured), when a message is about to send, then this spec selects email, applying FEAT-14.SPEC-006/007/008 rather than its own separate determination.

**FEAT-08.SPEC-011-AC-15:** Given Riley is WhatsApp-eligible under FEAT-26.SPEC-004, when a confirmation, reminder, or change notice is about to send, then FEAT-26.SPEC-004 decides first and this spec's text/email decision is not evaluated; given she is not WhatsApp-eligible, then this spec decides text or email as usual.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 8 | 8 |
| Edge Cases | 5 | 5 |



# Integration Spec: Transactional Text Messaging Capability

## Overview

**Name:** Transactional Text Messaging Capability
**ID:** FEAT-08.SPEC-012
**Type:** Integration
**Purpose:** Sends every text message the product needs to deliver -- confirmations, reminders, change notices, Pro notifications, access links, and every other feature's text-based messages -- through an external text-messaging capability, and reports back each message's delivery status.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Sending a text message on behalf of any spec in this feature, or in FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, or FEAT-30, per the Feature Dependency Map's External Touchpoints table
- Receiving and reporting back delivery status (Queued, Sent, Delivered, Failed) for every text sent
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client and Pro data is shared with this capability

**Non-Goals:**
- Choosing the text-messaging vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific vendor.
- Deciding whether a given message should be sent by text at all -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule); this spec sends whatever it is given once that decision has already been made.
- Retrying a failed text or falling back to email -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), which consumes this spec's Failed status as its own trigger.
- WhatsApp messaging -- a distinct capability owned by FEAT-26.SPEC-002, deferred to Later per scope-boundaries.md; this spec covers standard text messaging only.

## Capability Category

**Category:** Transactional text messaging
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional text messaging" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-20, FEAT-21, FEAT-26)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley receives an immediate booking confirmation by text | Send an immediate confirmation message on successful booking | FEAT-08.SPEC-001 |
| Riley receives a pre-appointment reminder with one-tap reply options by text | Send an automatic reminder a set time before the appointment | FEAT-08.SPEC-002 |
| Riley receives a cancellation, reschedule, or refund notice by text | Tell the client when their booking is cancelled, rescheduled or refunded | FEAT-08.SPEC-004 |
| Talia receives a new-booking or client-activity notice by text | Notify the Pro of new bookings, client cancellations and reschedules | FEAT-08.SPEC-005 |
| Talia receives an attention alert by text | Notify the Pro of anything needing attention | FEAT-08.SPEC-006 |
| Riley receives an access link or opt-out confirmation by text (on behalf of FEAT-06, FEAT-14) | Enabling capability for those features' own client-facing texts | FEAT-06, FEAT-14 |
| Talia receives billing, sign-in, waitlist, or recurring-series texts (on behalf of FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-29, FEAT-30) | Enabling capability for those features' own texts | FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-29, FEAT-30 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient phone number | Client -- phone (or Pro Account -- sign_in_mobile, for a Pro-directed text) | Every text send | The capability must know where to deliver the message |
| Message body text | Message -- the composed content for that send (already resolved from the sending spec's template) | Every text send | The capability needs the exact content to transmit |
| Sender identity (the Pro's account, in vendor-neutral terms) | Pro Account -- an account-level sending identity | Every text send | Lets the recipient's carrier and device attribute the message consistently to Chairtime/the sending Pro's account |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent text | Message -- delivery_status |
| Inbound reply content (a tapped link's URL parameters, or a STOP keyword) | The client replies to or taps a link in a received text | Routed to the specific automation that owns the reply (FEAT-08.SPEC-008 for a reminder reply; FEAT-14.SPEC-004 for a STOP reply) -- this spec never stores the raw reply text itself beyond what the routing needs, per ASMP-23's "the Pro ... does not receive the client's replies as raw texts" |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery status: Sent | The capability confirms the text left the sending system | Message.delivery_status set to Sent | None -- an intermediate status, not shown to either party | FEAT-08.SPEC-009 |
| Delivery status: Delivered | The capability confirms the text reached the recipient's device | Message.delivery_status set to Delivered | None directly -- delivery success is the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the text could not be delivered | Message.delivery_status set to Failed | Triggers FEAT-08.SPEC-009's retry-then-fallback; no direct client feedback (the client never receives a "delivery failed" message about their own confirmation) | FEAT-08.SPEC-009 |
| Inbound reply/link tap received | The client taps a link embedded in a received text, or replies with a keyword (e.g., STOP) | Routed to the owning automation (FEAT-08.SPEC-008 or FEAT-14.SPEC-004); no data lands directly in this spec | The reply's own outcome screen (owned by the receiving automation) | FEAT-08.SPEC-008, FEAT-14 |
| Booking-specific manage link tapped from a delivered text | The client taps the manage link embedded in a confirmation or reminder text | No data lands in this spec; the tap is handed to FEAT-06.SPEC-002, which validates the Access Link and resolves it | The client lands on the booking detail (FEAT-06.SPEC-004) or the "request a new link" prompt (FEAT-06.SPEC-001), as decided by FEAT-06.SPEC-002 | FEAT-06.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it, since the confirmation is sent in the background after payment completes | The send attempt is recorded as Failed once a timeout is reached; FEAT-08.SPEC-009's retry-then-fallback to email takes over -- no booking-flow screen is blocked, since confirmation sending is asynchronous to the payment flow | Same as Capability Down: recorded as Failed and handed to FEAT-08.SPEC-009 |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling as above -- no user-facing screen is affected while the send is slow | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling as above | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |
| FEAT-08.SPEC-005 / FEAT-08.SPEC-006 (Pro Notifications) | Same background handling as above; the in-app copy of the notification is unaffected regardless, since in-app never depends on this capability | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |

N/A -- no screen in this feature sends a text synchronously in front of the user (every send in this feature is background/asynchronous to the triggering user action), so no screen shows a loading or blocked state tied directly to this capability; all degradation surfaces instead through FEAT-08.SPEC-009's retry/fallback and FEAT-08.SPEC-006's Pro-facing alert.

## Consent and Disclosure

- **Texting consent captured at booking** -- Before any text is sent to a client, FEAT-05 (Public Booking Page & Booking Flow) captures explicit opt-in with the exact wording shown at booking, per ASMP-24's US SMS-consent requirement; this spec never initiates a send without FEAT-08.SPEC-011 confirming that consent is currently active.
- **What is shared with the capability** -- The disclosure a client sees at booking states plainly that their phone number and booking-related message content are used to send them text updates about their appointment; it names no vendor, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes about the client, payment/card details, and any content beyond the single message being sent never reach this capability. Card data is never held or transmitted by the product at all (SC-11), and this capability has no channel through which it could receive it.
- **Opt-out is always available** -- Every client-directed text this capability sends on behalf of any feature (this one or another) includes or is otherwise governed by an opt-out mechanism owned by FEAT-14 (Messaging Consent Management); this spec's disclosure obligation includes never sending a client text once FEAT-08.SPEC-011 reports consent as inactive.

## Edge Cases

- **A delivery-status event arrives for a Message whose Booking has since been deleted from active view (cancelled and archived into history)** -- The event is recorded against the Booking's retained history record (bookings are never deleted, per feature-dependency-map.md), and no user feedback fires beyond what FEAT-08.SPEC-009 already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and FEAT-08.SPEC-009's retry-then-fallback does not re-trigger for an already-resolved Message.
- **Events arrive out of order (a Delivered status arrives before its preceding Sent status)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time; an out-of-order Sent arriving after Delivered does not regress the status.
- **The capability goes down mid-send, with no confirmation either way** -- If no Sent or Failed status is ever received within a defined timeout, the send is treated as Failed for the purpose of triggering FEAT-08.SPEC-009's retry-then-fallback, so a message is never left in an indefinite unknown state.
- **A reply/link tap arrives for a Message whose Booking has already moved past the point that reply is meaningful (e.g., a reminder reply tap after the booking was already cancelled)** -- The routing automation (FEAT-08.SPEC-008) evaluates the tap against the Booking's current state and responds accordingly (typically the Expired/Invalid Link state), rather than this integration spec making that judgment itself.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation when text is the chosen channel |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder when text is the chosen channel |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice when text is the chosen channel |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggered by (inbound) | Sends the Pro notification when text is enabled |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggered by (inbound) | Sends the Pro alert when text is enabled |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Affects (outbound) | A Failed delivery status fires this automation |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | Affects (outbound) | Routes inbound reply-link taps |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Governs whether this capability is used for a given send |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Affects (outbound) | Receives every tap of a booking-specific manage link embedded in a text sent through this capability |
| FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | Triggered by (inbound) | Each sends its own texts through this shared capability, per the External Touchpoints table |

## Analytics and Success Signals

- **text_send_attempted** (sending_spec: spec ID) -- N/A -- no Stage 2 metric measures raw send-attempt volume; retained as the operational baseline behind delivery-status metrics.
- **text_delivery_status_received** (status: sent / delivered / failed) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that never delivers cannot be responded to, so delivery reliability directly gates this metric's numerator.
- **text_send_rejected** (reason category) -- N/A -- no Stage 2 metric measures rejection frequency; retained so the capability's real-world reliability is observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-012-AC-01:** Given Riley has active texting consent and a confirmation is ready to send, when FEAT-08.SPEC-011 selects text as the channel, then this capability sends the message and reports back a delivery status.

**FEAT-08.SPEC-012-AC-02:** Given a text sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered and no further action is taken.

**FEAT-08.SPEC-012-AC-03:** Given a text sent through this capability cannot be delivered, when the Failed status arrives, then FEAT-08.SPEC-009's retry-then-fallback automation is triggered.

**FEAT-08.SPEC-012-AC-04:** Given Riley taps the "I'll be there" link inside a text sent through this capability, when the tap is received, then it is routed to FEAT-08.SPEC-008 for processing, and no raw reply content is stored beyond what that routing needs.

**FEAT-08.SPEC-012-AC-05:** Given the capability is temporarily slow to respond, when a confirmation is queued for sending, then no client-facing screen shows a waiting state, since the send is asynchronous to the payment flow.

**FEAT-08.SPEC-012-AC-06:** Given the capability is down when a reminder attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed and handed to FEAT-08.SPEC-009.

**FEAT-08.SPEC-012-AC-07:** Given the same Delivered event for one Message is delivered twice by the capability, when the second event arrives, then nothing changes and no duplicate action fires.

**FEAT-08.SPEC-012-AC-08:** Given a Delivered event arrives before its preceding Sent event for the same Message, when both are processed, then the Message reflects Delivered and the late-arriving Sent event does not regress it.

**FEAT-08.SPEC-012-AC-09:** Given FEAT-14 needs to send an opt-out confirmation text, when it requests a send through this capability, then the send and its delivery-status reporting behave identically to a send requested by this feature's own specs.

**FEAT-08.SPEC-012-AC-10:** Given Riley is shown the texting-consent disclosure at booking, when she reads it, then it states plainly that her phone number and booking message content are used to send her text updates, naming no vendor.

**FEAT-08.SPEC-012-AC-11:** Given a delivery-status event arrives for a Message tied to a booking that has since been cancelled and archived, when the event is processed, then it is recorded against the retained history record with no additional user-facing feedback beyond FEAT-08.SPEC-009's governance.

**FEAT-08.SPEC-012-AC-12:** Given Riley's card details are never held by the product, when this capability sends any text, then no payment or card data is ever included in the message content or the data exchanged with the capability.

**FEAT-08.SPEC-012-AC-13:** Given a client's texting consent is inactive at send time, when FEAT-08.SPEC-011 evaluates the channel, then this capability is never invoked for that send; the message routes to FEAT-08.SPEC-013 (email) instead.

**FEAT-08.SPEC-012-AC-14:** Given Riley taps the manage link in a delivered confirmation text, when the tap arrives, then it is handed to FEAT-06.SPEC-002 for validation and this capability stores nothing from the tap beyond routing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 8 | 8 |
| Inbound Events | 5 | 5 |
| Degradation Paths | 4 (screens; slow/down/rejects handled uniformly per screen) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |



# Integration Spec: Transactional Email Capability

## Overview

**Name:** Transactional Email Capability
**ID:** FEAT-08.SPEC-013
**Type:** Integration
**Purpose:** Sends every email message the product needs to deliver -- as the fallback channel after a failed text and as the primary channel for clients who decline texting -- through an external transactional-email capability, and reports back each message's delivery status.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Sending an email on behalf of any spec in this feature, or in FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, or FEAT-30, per the External Touchpoints table
- Receiving and reporting back delivery status for every email sent
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client and Pro data is shared with this capability

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 decision; BRIEF.md records no mandate.
- Deciding whether a given message should be sent by email -- owned by FEAT-08.SPEC-011 (as the client's chosen channel) or FEAT-08.SPEC-009 (as the fallback after a failed text); this spec sends whatever it is given once that decision has already been made.
- Retrying a failed email with a further fallback -- product-features.md and the Brief describe only a text-then-email chain; there is no channel beyond email, so an email failure is a final failure handled per this spec's own Inbound Events, not a further automated fallback.
- Marketing or promotional email content -- excluded per scope-boundaries.md (SC-15), consistent with this feature's texting exclusion of the same; every email sent through this capability is transactional (booking-related) content only.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (fallback channel)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-18, FEAT-21, FEAT-26)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley (having declined texting) receives her booking confirmation by email | Send an immediate confirmation message on successful booking (fallback path) | FEAT-08.SPEC-001 |
| Riley receives her reminder by email when texting is declined | Send an automatic reminder a set time before the appointment (fallback path) | FEAT-08.SPEC-002 |
| Riley receives a change/refund notice by email when texting is declined | Tell the client when their booking is cancelled, rescheduled or refunded (fallback path) | FEAT-08.SPEC-004 |
| Riley receives any message by email after a failed text | Retry once, then fall back to email, so no client message is ever silently dropped | FEAT-08.SPEC-009 |
| Talia receives Pro notifications and alerts by email when enabled | Notify the Pro of new bookings, changes, and anything needing attention | FEAT-08.SPEC-005, FEAT-08.SPEC-006 |
| Riley/Talia receive access links, opt-out confirmations, billing notices, and other features' emails (on behalf of FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30) | Enabling capability for those features' own email sends | Those features' own specs |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient email address | Client -- email (or Pro Account -- sign_in_email, for a Pro-directed email) | Every email send | The capability must know where to deliver the message |
| Message subject and body content | Message -- the composed content for that send | Every email send | The capability needs the exact content to transmit |
| Sender identity (an account-level sending identity, vendor-neutral) | Pro Account -- an account-level sending identity | Every email send | Lets the recipient's mail client attribute the message consistently |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent email | Message -- delivery_status |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery status: Sent | The capability confirms the email left the sending system | Message.delivery_status set to Sent | None | -- |
| Delivery status: Delivered | The capability confirms acceptance by the recipient's mail server | Message.delivery_status set to Delivered | None -- the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the email could not be delivered (e.g., an invalid or bouncing address) | Message.delivery_status set to Failed | If this Failed status is itself the fallback attempt after a failed text (FEAT-08.SPEC-009), it escalates to FEAT-08.SPEC-006's "both channels fail" Pro alert; if it is a primary-channel email send (no prior text attempted), it also triggers FEAT-08.SPEC-006 so the Pro learns her client may not have received the message at all | FEAT-08.SPEC-009, FEAT-08.SPEC-006 |
| Booking-specific manage link tapped from a delivered email | The client taps the manage link embedded in a confirmation or reminder email | No data lands in this spec; the tap is handed to FEAT-06.SPEC-002, which validates the Access Link and resolves it | The client lands on the booking detail (FEAT-06.SPEC-004) or the "request a new link" prompt (FEAT-06.SPEC-001), as decided by FEAT-06.SPEC-002 | FEAT-06.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it | Recorded as Failed once a timeout is reached; escalated per Inbound Events above -- no booking-flow screen is blocked | Same as Capability Down |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling | Same Failed-and-escalate handling | Same Failed-and-escalate handling |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling | Same Failed-and-escalate handling | Same Failed-and-escalate handling |
| FEAT-08.SPEC-005 / FEAT-08.SPEC-006 (Pro Notifications) | Same background handling; the in-app copy is unaffected regardless | Same Failed-and-escalate handling | Same Failed-and-escalate handling |

N/A -- no screen in this feature sends an email synchronously in front of the user; every send is background/asynchronous to its triggering event, so degradation surfaces through Message.delivery_status and the escalation path above, not through a blocked or waiting screen.

## Consent and Disclosure

- **Email is disclosed as the fallback and no-texting-consent channel at booking** -- FEAT-05's booking flow states that a client who declines texting will receive booking updates by email instead, and that email is used automatically if a text ever fails to deliver; this is disclosed once, at booking, not re-disclosed on every individual fallback event.
- **What is shared with the capability** -- The client's email address and the content of the specific transactional message being sent; no vendor is named, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes, payment/card details, and any content beyond the single message being sent never reach this capability; card data is never held by the product at all (SC-11).
- **No marketing use** -- The disclosure at booking states that email is used only for the client's own booking-related messages, never for marketing or promotional content, consistent with scope-boundaries.md (SC-15).

## Edge Cases

- **A delivery-status event arrives for a Message tied to a since-cancelled and archived Booking** -- Recorded against the Booking's retained history record; no additional user-facing feedback beyond what the triggering Notification spec already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and no duplicate escalation fires.
- **Events arrive out of order (Delivered arrives before Sent)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time.
- **The capability goes down mid-send with no confirmation either way** -- If no Sent or Failed status is received within a defined timeout, the send is treated as Failed for escalation purposes, so a message is never left in an indefinite unknown state.
- **An email fallback is attempted for a client whose email address is missing or malformed** -- This should not occur given FEAT-05's requirement that an email be captured whenever texting is declined (product-features.md), but if it does, the send is recorded as Failed immediately (an invalid-address rejection) and escalates directly to FEAT-08.SPEC-006, since there is no further fallback channel beyond email.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation when email is the chosen or fallback channel |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder when email is the chosen or fallback channel |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice when email is the chosen or fallback channel |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggered by (inbound) | Sends the Pro notification when email is enabled |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggered by (inbound); Triggers (outbound) | Sends the Pro alert when email is enabled; also receives escalations when this capability itself fails |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | Performs the fallback send after a failed text |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Selects email as the primary channel for consent-declined clients |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Affects (outbound) | Receives every tap of a booking-specific manage link embedded in an email sent through this capability |
| FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | Triggered by (inbound) | Each sends its own emails through this shared capability |

## Analytics and Success Signals

- **email_send_attempted** (sending_spec: spec ID; reason: primary_channel / text_fallback) -- N/A -- no Stage 2 metric measures raw send-attempt volume; retained as the operational baseline behind delivery-status metrics.
- **email_delivery_status_received** (status: sent / delivered / failed) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that reaches no client on any channel cannot be responded to.
- **email_send_failed** (reason category) -- N/A -- no Stage 2 metric measures email failure frequency directly; retained so the "never silently dropped" guarantee (XBR-17) is observable at the final channel in the chain.

## Acceptance Criteria

**FEAT-08.SPEC-013-AC-01:** Given Riley declined texting at booking and provided an email, when her confirmation is ready to send, then this capability sends it and reports back a delivery status.

**FEAT-08.SPEC-013-AC-02:** Given a text to Riley failed and FEAT-08.SPEC-009's fallback runs, when the fallback email is sent, then this capability delivers it and reports the outcome back to the Message record created for that fallback.

**FEAT-08.SPEC-013-AC-03:** Given an email sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered.

**FEAT-08.SPEC-013-AC-04:** Given a fallback email sent through this capability also fails, when the Failed status arrives, then FEAT-08.SPEC-006's escalated "both channels fail" alert fires for the Pro.

**FEAT-08.SPEC-013-AC-05:** Given a primary-channel email (no text attempted, consent-declined client) fails to deliver, when the Failed status arrives, then FEAT-08.SPEC-006 alerts the Pro that the client may not have received the message.

**FEAT-08.SPEC-013-AC-06:** Given the capability is down when an email attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed for escalation purposes.

**FEAT-08.SPEC-013-AC-07:** Given the same Delivered event for one Message is delivered twice, when the second event arrives, then nothing changes and no duplicate escalation fires.

**FEAT-08.SPEC-013-AC-08:** Given a Delivered event arrives before its preceding Sent event, when both are processed, then the Message reflects Delivered and the late Sent event does not regress it.

**FEAT-08.SPEC-013-AC-09:** Given Riley is shown the email-fallback disclosure at booking, when she declines texting, then she is told plainly that booking updates will arrive by email instead and that email is used automatically if a text ever fails.

**FEAT-08.SPEC-013-AC-10:** Given a client's email address is missing or malformed at fallback time, when the send is attempted, then it is recorded as Failed immediately and escalates directly to FEAT-08.SPEC-006, since no further fallback channel exists.

**FEAT-08.SPEC-013-AC-11:** Given FEAT-18 needs to send a billing notice by email, when it requests a send through this capability, then the send and its delivery-status reporting behave identically to a send requested by this feature's own specs.

**FEAT-08.SPEC-013-AC-12:** Given this capability sends any email, when its content is composed, then no marketing or promotional content is ever included -- only the specific transactional message the triggering spec composed.

**FEAT-08.SPEC-013-AC-13:** Given Riley taps the manage link in a delivered confirmation or reminder email, when the tap arrives, then it is handed to FEAT-06.SPEC-002 for validation and this capability stores nothing from the tap beyond routing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 4 (screens; slow/down/rejects handled uniformly per screen) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |


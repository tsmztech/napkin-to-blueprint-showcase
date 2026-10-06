---
document_type: product-features
produced_by: product-visionary
variant: draft
status: draft
created: 2026-09-26
coherence_check: passed
---

# Product Features

## Summary

This product defines 26 features: 12 Core, 7 Important, 7 Nice-to-Have. By phase: 17 MVP, 6 v1, 3 Later. By type: 18 User-Facing, 6 Platform, 2 Lifecycle. The product manages 14 domain entities. Core features close the end-to-end value loop the founder named — a client books and pays a deposit in under a minute, and a no-show is handled automatically under the pro's own policy; Important features cover the supporting record-keeping, setup, consent, and billing machinery a real business needs; Nice-to-Have features answer the brief's open questions (waitlist, recurring appointments, in-app balance payment, tipping) and later-market polish (search, insights, WhatsApp).

## Domain Entity Inventory

### Entity: Pro Account
- **Description:** The solo professional's account — identity, timezone, currency, and the subscription that keeps the account active.
- **Lifecycle:** Created (signup) -> Active -> (optionally) Paused/Cancelled (subscription lapse)
- **Created by:** Pro Onboarding & Setup Wizard (FEAT-15)
- **Managed by:** Pro Subscription Billing & Account Management (FEAT-18)
- **Referenced by:** nearly every feature; directly by Service & Pricing Management (FEAT-01), Availability & Working Hours Setup (FEAT-02), Public Booking Page & Booking Flow (FEAT-05), Platform Support Read-Only Access (FEAT-19)

### Entity: Service
- **Description:** A bookable offering the pro sells — name, price, duration, and its deposit rule.
- **Lifecycle:** Created -> Active -> (optionally) Archived (hidden from new bookings, retained on past bookings)
- **Created by:** Service & Pricing Management (FEAT-01)
- **Managed by:** Service & Pricing Management (FEAT-01)
- **Referenced by:** Public Booking Page & Booking Flow (FEAT-05), Real-Time Slot Availability Engine (FEAT-03), Deposit Payment at Booking (FEAT-07), Booking & Revenue Insights (FEAT-25)

### Entity: Availability Rule
- **Description:** The pro's recurring working hours and per-service buffer time.
- **Lifecycle:** Created -> Active -> Edited (versioned by effective date so past bookings are unaffected)
- **Created by:** Availability & Working Hours Setup (FEAT-02)
- **Managed by:** Availability & Working Hours Setup (FEAT-02)
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03)

### Entity: Time Block
- **Description:** A manually blocked span of time (personal time off, a held slot) that removes availability without a booking.
- **Lifecycle:** Created -> Active -> Expired/Deleted
- **Created by:** Manual Time Blocking (FEAT-17)
- **Managed by:** Manual Time Blocking (FEAT-17)
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03)

### Entity: Calendar Connection
- **Description:** The link between a Pro Account and their personal Google or Apple calendar, used to read busy times and write confirmed bookings.
- **Lifecycle:** Connected -> Active (syncing) -> Disconnected (reconnection required)
- **Created by:** Two-Way Calendar Sync (FEAT-04)
- **Managed by:** Two-Way Calendar Sync (FEAT-04)
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03)

### Entity: Client
- **Description:** A record of a person who has booked (or attempted to book) with a specific pro — contact details, notes, and booking history with that pro only.
- **Lifecycle:** Created (first booking) -> Active -> Deleted (on request)
- **Created by:** Public Booking Page & Booking Flow (FEAT-05)
- **Managed by:** Client Record Management (FEAT-13)
- **Referenced by:** Client Booking Identity (FEAT-06), Pro Daily Schedule Dashboard (FEAT-12), Client List Search & Filter (FEAT-24)

### Entity: Booking
- **Description:** A confirmed (or attempted) appointment: service, time, client, deposit status, and current lifecycle state.
- **Lifecycle:** Pending Payment -> Confirmed -> (Completed | Cancelled | No-Show | Rescheduled)
- **Created by:** Public Booking Page & Booking Flow (FEAT-05)
- **Managed by:** Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Daily Schedule Dashboard (FEAT-12)
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03), Automated Booking Messaging (FEAT-08), Cancellation & No-Show Policy Engine (FEAT-09), Booking & Payment Activity Record (FEAT-16), Booking & Revenue Insights (FEAT-25), Recurring/Standing Appointments (FEAT-21), Waitlist for Cancelled Slots (FEAT-20)

### Entity: Deposit Transaction
- **Description:** The record of a client's card deposit for one booking — amount, status (held/captured/refunded/forfeited), and its outcome.
- **Lifecycle:** Authorized -> Captured -> (Refunded | Forfeited)
- **Created by:** Deposit Payment at Booking (FEAT-07)
- **Managed by:** Cancellation & No-Show Policy Engine (FEAT-09), No-Show Marking & Deposit Forfeiture (FEAT-11)
- **Referenced by:** Booking & Payment Activity Record (FEAT-16), Booking & Revenue Insights (FEAT-25), In-App Balance Payment (FEAT-22)

### Entity: Cancellation Policy
- **Description:** The pro's own rule set: the cancellation/reschedule window and what happens to the deposit inside vs. outside it.
- **Lifecycle:** Created (during onboarding) -> Active -> Edited (versioned; a booking is always governed by the policy in force when it was made)
- **Created by:** Pro Onboarding & Setup Wizard (FEAT-15)
- **Managed by:** Cancellation & No-Show Policy Engine (FEAT-09)
- **Referenced by:** Public Booking Page & Booking Flow (FEAT-05), Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11)

### Entity: Messaging Consent
- **Description:** A client's explicit opt-in (or opt-out) to receive text messages from a specific pro, and its timestamp/scope.
- **Lifecycle:** Granted -> Active -> Revoked
- **Created by:** Public Booking Page & Booking Flow (FEAT-05) (captured at booking)
- **Managed by:** Messaging Consent Management (FEAT-14)
- **Referenced by:** Automated Booking Messaging (FEAT-08)

### Entity: Message
- **Description:** A record of a single notification sent (confirmation, reminder, cancellation notice) — channel, content summary, delivery status.
- **Lifecycle:** Queued -> Sent -> (Delivered | Failed)
- **Created by:** Automated Booking Messaging (FEAT-08)
- **Managed by:** N/A -- messages are immutable once sent
- **Referenced by:** Booking & Payment Activity Record (FEAT-16)

### Entity: Subscription
- **Description:** The pro's own paid plan with Chairtime — status, billing cycle, and payment method on file.
- **Lifecycle:** Trial/Signup -> Active -> (Payment Failed) -> (Cancelled)
- **Created by:** Pro Onboarding & Setup Wizard (FEAT-15)
- **Managed by:** Pro Subscription Billing & Account Management (FEAT-18)
- **Referenced by:** Platform Support Read-Only Access (FEAT-19)

### Entity: Waitlist Entry
- **Description:** A client's standing request to be notified if a specific service/day opens up from a cancellation.
- **Lifecycle:** Requested -> Notified -> (Converted to Booking | Expired)
- **Created by:** Waitlist for Cancelled Slots (FEAT-20)
- **Managed by:** Waitlist for Cancelled Slots (FEAT-20)
- **Referenced by:** Client-Initiated Cancel/Reschedule (FEAT-10) (a cancellation can trigger a waitlist notification)

### Entity: Recurring Series
- **Description:** A standing pattern ("every 3 weeks") that generates individual Bookings on a schedule.
- **Lifecycle:** Created -> Active -> (Paused | Ended)
- **Created by:** Recurring/Standing Appointments (FEAT-21)
- **Managed by:** Recurring/Standing Appointments (FEAT-21)
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03) (reserves future slots)

## Core Features

### Service & Pricing Management

**ID:** FEAT-01

**Description:** The Pro defines the services they offer — name, price, duration, and the deposit rule for that service (fixed amount or percentage) — and controls which services are currently bookable.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** Directly required by BRIEF.md's Vision: a client "picks a service" with "prices and how long each takes" stated in plain words before booking. Without this, the booking page has nothing to show. MVP: the product cannot function without at least one bookable service.

**Connected Entities:** Service (create, read, update, archive)

**Key Capabilities:**
- Add a service — name, price, duration, and its deposit rule (fixed amount or percentage of price)
- Edit a service — update price, duration, or deposit rule for future bookings without altering past ones
- Archive a service — hide it from new bookings while keeping history intact
- Reorder services as they appear on the booking page

**Primary Flows & Alternates:**
- Happy path: Pro adds a service with name, price, duration, and deposit rule; it immediately appears on the public booking page.
- Alternate: Pro edits a live service's price; existing confirmed bookings keep the price and deposit that were agreed at booking time.
- Alternate: Pro archives a service with upcoming bookings; those bookings are honored and shown as-is, but the service disappears from new booking choices.

**States:** Empty: a new Pro Account with zero services sees a guided prompt to add their first service rather than a blank list. Loading: N/A — service lists are small (a handful to a few dozen) and load instantly. Error: a failed save keeps the entered values on screen with a clear retry action. Offline-degraded: N/A — this is a setup screen used on a stable connection between clients, not a mobile in-the-moment flow.

**Validation & Limits:** Service name required (1–80 characters); price must be a positive amount; duration required and must be a positive number of minutes; deposit rule must be either a fixed amount less than the service price or a percentage between 1–100%.

**Access:** The Pro has Full access per the Access Matrix in user-persona.md. Clients never see a setup view — they see only the resulting public list of bookable services. Platform Operator (Support) has View-only access for troubleshooting.

**Communications:** N/A — this is a setup action with no notification of its own.

**Data Notes:** Captured: name, price, duration, deposit rule. Displayed: the pro's own service list and, publicly, the client-facing service list on the booking page. Derived: none. Source: pro input only.

**Interactions:** Feeds Public Booking Page & Booking Flow (FEAT-05), Real-Time Slot Availability Engine (FEAT-03) (service duration drives slot length), and Deposit Payment at Booking (FEAT-07) (deposit rule).

**Signals:** service_added, service_edited, service_archived.

### Availability & Working Hours Setup

**ID:** FEAT-02

**Description:** The Pro sets their recurring working hours and the buffer time they need between clients, forming the base schedule the availability engine works from.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Pro sets "working hours, buffer time between clients" as a core setup action. MVP: real-time availability has nothing to compute from without it.

**Connected Entities:** Availability Rule (create, update)

**Key Capabilities:**
- Set weekly working hours (per day of week, with multiple windows per day allowed)
- Set default buffer time applied between consecutive bookings
- Override buffer time per service where a service genuinely needs more or less

**Primary Flows & Alternates:**
- Happy path: Pro sets hours for each working day and a default buffer; the change is reflected in bookable slots going forward immediately.
- Alternate: Pro closes a normally-working day for a one-off reason using Manual Time Blocking (FEAT-17) rather than editing the recurring rule.
- Alternate: Pro changes hours mid-week; already-confirmed bookings outside the new hours are never silently cancelled — they remain honored and flagged for the Pro's attention.

**States:** Empty: a brand-new account has no hours set and cannot be booked until at least one working window exists; the setup wizard (FEAT-15) makes this the first required step. Loading: N/A — instant, small dataset. Error: a save failure preserves entered values with a retry option. Offline-degraded: N/A — setup screen, not an in-the-moment mobile flow.

**Validation & Limits:** Each working window requires a start time before its end time; buffer time must be zero or a positive number of minutes; overlapping windows on the same day are rejected with a clear message.

**Access:** The Pro has Full access. Clients never see this screen, only its effect (available times). Platform Operator (Support) has View-only access.

**Communications:** N/A — a setup action, not a notification trigger.

**Data Notes:** Captured: weekly hours and buffer settings. Displayed: on the Pro's own setup screen. Derived: none directly, but this data is the primary input the availability engine derives free slots from. Source: pro input only.

**Interactions:** Feeds Real-Time Slot Availability Engine (FEAT-03).

**Signals:** availability_hours_updated, buffer_time_updated.

### Real-Time Slot Availability Engine

**ID:** FEAT-03

**Description:** The system that computes, at the moment a client is looking, exactly which time slots are genuinely free — combining the Pro's working hours, buffer time, existing Chairtime bookings, manual time blocks, and busy times from the Pro's connected personal calendar — so a client can never select a time that is not truly open.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Vision and Success Criteria are explicit: "a genuinely free time," and "nobody has ever had a double booking." This is the mechanism that makes that promise true; every other feature that touches time depends on it.

**Connected Entities:** Booking (read), Availability Rule (read), Time Block (read), Calendar Connection (read), Recurring Series (read)

**Key Capabilities:**
- Compute the live set of open slots for a given service and date range
- Reserve a slot the instant a client begins paying, preventing a second client from grabbing it mid-checkout
- Release a held-but-unpaid slot automatically after a short timeout if payment is not completed

**Primary Flows & Alternates:**
- Happy path: client opens the booking page, picks a service, and sees only slots that account for hours, buffer, existing bookings, manual blocks, and external calendar busy time.
- Alternate: two clients open the same slot simultaneously; the first to complete payment wins it, and the second sees it disappear from the list before they can pay, with a clear "just booked" message rather than a payment error.
- Alternate: the Pro's external calendar sync is temporarily unavailable; the engine falls back to Chairtime-only data and visibly flags reduced confidence to the Pro (never to the client, who must never be shown a slot that turns out to be unavailable).

**States:** Empty: if a service has no open slots in the visible window, the client sees a plain "fully booked, check back or view other services" message, never a blank grid. Loading: a lightweight in-place loading indicator while slots compute; never a blank screen with no feedback. Error: if computation fails, the client sees a retry prompt rather than a stale or incorrect slot list — an incorrect slot is treated as worse than no slot list at all. Offline-degraded: N/A — availability must always reflect live, connected data; a stale offline slot list is exactly the double-booking risk the product exists to prevent.

**Validation & Limits:** A slot is only offered if the full service duration plus buffer fits entirely within an open working window with no conflicting booking, block, or external calendar event; slot holds during checkout expire after a short, fixed timeout (a few minutes) if payment is not completed.

**Access:** Clients see only the resulting open slots for one pro's public page — never another pro's schedule. The Pro sees their own full schedule. Platform Operator (Support) has View-only access for troubleshooting a specific pro's reported conflict.

**Communications:** N/A — this is a computation engine; it triggers no messages of its own (Automated Booking Messaging, FEAT-08, sends the resulting confirmations).

**Data Notes:** Displayed: computed available slots. Derived: entirely — every slot shown is computed live from Availability Rule, Time Block, Booking, Calendar Connection, and Recurring Series data; nothing here is directly entered by a user.

**Interactions:** Depends on Availability & Working Hours Setup (FEAT-02), Two-Way Calendar Sync (FEAT-04), Manual Time Blocking (FEAT-17), Recurring/Standing Appointments (FEAT-21); feeds Public Booking Page & Booking Flow (FEAT-05).

**Signals:** slot_list_computed, slot_held, slot_hold_expired, slot_conflict_prevented.

### Two-Way Calendar Sync

**ID:** FEAT-04

**Description:** The Pro connects their personal Google or Apple calendar. Busy time there blocks Chairtime availability, and confirmed Chairtime bookings appear on that personal calendar automatically.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Ecosystem & Integrations states this is two-way and "both matter" for Google and Apple. A pro who lives partly off-platform (personal appointments, a second job) cannot trust the availability engine without it, directly serving the "never silently double-book" success criterion.

**Connected Entities:** Calendar Connection (create, update, delete)

**Key Capabilities:**
- Connect a Google or Apple calendar
- See connection health (connected / needs reconnection)
- Disconnect a calendar at any time

**Primary Flows & Alternates:**
- Happy path: Pro connects their calendar during onboarding; from that point, external busy time blocks Chairtime slots and new Chairtime bookings appear on the personal calendar within moments of confirmation.
- Alternate: the connection lapses (revoked access, expired token); the Pro sees a clear "reconnect your calendar" prompt on their dashboard, and the availability engine visibly narrows its confidence rather than silently trusting stale data.
- Alternate: Pro disconnects intentionally; existing Chairtime bookings remain intact, but external busy time no longer factors into future availability until reconnected.

**States:** Empty: no calendar connected shows a plain explanation of what connecting does and why, not a technical error. Loading: a brief "syncing" indicator appears right after connecting. Error: a sync failure surfaces as a dashboard banner ("reconnect needed"), never a silent gap. Offline-degraded: the most recently synced busy times remain in effect until connectivity is restored.

**Validation & Limits:** Only one calendar of each supported kind may be connected per Pro Account at a time; sync latency target is near-immediate (busy time and new bookings should reflect within a couple of minutes each way).

**Access:** The Pro has Full access to connect/disconnect their own calendar. Clients have no visibility into this at all. Platform Operator (Support) has View-only access to connection health for troubleshooting.

**Communications:** A dashboard alert (not a text/email) when a connection needs reconnecting.

**Data Notes:** Captured: connection status and the minimum busy/free time data needed to block slots (not full event details). Displayed: connection status to the Pro. Derived: none. Source: the Pro's own calendar account, and Chairtime's own confirmed bookings written out to it.

**Interactions:** Feeds Real-Time Slot Availability Engine (FEAT-03); reads from Booking (FEAT-05, FEAT-10, FEAT-11) to write bookings out to the external calendar.

**Signals:** calendar_connected, calendar_disconnected, calendar_sync_failed, calendar_reconnected.

### Public Booking Page & Booking Flow

**ID:** FEAT-05

**Description:** The single, mobile-first page a client reaches from the Pro's Instagram bio link — showing the Pro's name, services with prices and durations, and the deposit rule in plain words — where a client picks a service, a genuinely free time, enters their name and phone, opts into texts, and pays the deposit, all in one continuous flow.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the literal product described in BRIEF.md's Vision: "a client opens the pro's link... picks a service and a genuinely free time... pays a card deposit." It is the entire reason the product exists and the founder's stated one-minute benchmark.

**Connected Entities:** Booking (create), Client (create), Service (read), Cancellation Policy (read), Messaging Consent (create — captured here)

**Key Capabilities:**
- View a Pro's services, prices, durations, and deposit rule in plain language
- Pick a service and a genuinely free time slot
- Enter name and phone, and opt in to text messages
- Complete deposit payment and receive an immediate on-screen confirmation

**Primary Flows & Alternates:**
- Happy path: client taps the bio link -> sees services -> picks one -> picks a free time -> enters name and phone -> opts in to texts -> pays the deposit by card -> sees a confirmation on screen, in under a minute end to end.
- Alternate: returning client — a client who has booked with this Pro before is recognized by phone number and can skip re-entering their name (BRIEF.md's identity mechanism, FEAT-06).
- Alternate: payment fails or is declined — the client sees a clear, specific reason and can retry with the same or a different card without losing their selected slot (within the slot hold timeout).

**States:** Empty: N/A — the booking page always shows the Pro's current service list; if a Pro has zero active services, the page shows a plain "temporarily not accepting bookings" message rather than a broken page. Loading: a lightweight indicator while slots load; the page never appears interactive before real availability has loaded. Error: a failed step (slot no longer available, payment failure) keeps all previously entered information intact so the client never has to start over. Offline-degraded: booking requires a live connection to guarantee correctness (per the Real-Time Slot Availability Engine's own offline stance); a client who loses connection mid-flow sees a plain "check your connection and try again" message with nothing charged.

**Validation & Limits:** Name required (1–100 characters); phone number required and must be a valid, reachable format; the client must actively check the texting opt-in — it is never pre-checked; a slot hold expires after a short fixed window if payment is not completed.

**Access:** Open to any Client, with no login required to reach the flow itself (BRIEF.md: "must not face a signup wall"). The Pro has Full/View access to see resulting bookings on their own dashboard. Platform Operator (Support) has View-only access.

**Communications:** Triggers the immediate booking confirmation handled by Automated Booking Messaging (FEAT-08).

**Data Notes:** Captured: chosen service and time, client name and phone, texting opt-in, deposit payment outcome. Displayed: service list, prices, durations, deposit rule, available slots. Derived: none directly; the resulting Booking record is the source others read from.

**Interactions:** Depends on Service & Pricing Management (FEAT-01), Real-Time Slot Availability Engine (FEAT-03), Client Booking Identity (FEAT-06), Deposit Payment at Booking (FEAT-07), Cancellation & No-Show Policy Engine (FEAT-09); feeds Automated Booking Messaging (FEAT-08) and Pro Daily Schedule Dashboard (FEAT-12).

**Signals:** booking_page_viewed, service_selected, slot_selected, booking_completed, booking_abandoned (with last completed step).

### Client Booking Identity

**ID:** FEAT-06

**Description:** The lightweight, password-free way a client proves it's them when they come back to view, reschedule, or cancel a booking — a phone number plus a one-tap link sent to that phone, rather than any signup wall or password.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Open Questions names this exactly ("A phone number plus a magic link, or something else?") and its Target Users & Roles states clients "must not face a signup wall or need a password-style account." As the Visionary's product judgment on this open question: phone-plus-link is the lightest workable mechanism that still lets a client manage a specific booking without exposing any other client's or pro's data. MVP: without it, a client has no way to self-serve a reschedule or cancellation, which the brief requires (FEAT-10).

**Connected Entities:** Client (read, update — matches by phone), Booking (read — scoped to that client and pro)

**Key Capabilities:**
- Request a one-tap access link sent by text to the phone number used at booking
- View only this pro's bookings tied to that phone number, past and upcoming
- Access expires and must be re-requested after a short period for security

**Primary Flows & Alternates:**
- Happy path: client taps "manage my booking" from a reminder text or the booking page, requests a link, taps it, and sees only their own bookings with this one pro.
- Alternate: client requests a link from a phone number with no bookings for this pro; they see a plain "no bookings found" message rather than an error, with no hint about whether the number exists elsewhere.
- Alternate: the access link expires before use; requesting a new one is a single tap, with no separate "reset" flow to learn.

**States:** Empty: a phone number with no bookings shows a plain, non-alarming message. Loading: brief indicator while the link is generated and sent. Error: failed link delivery offers an immediate retry. Offline-degraded: N/A — this is an online-only identity check by design (correctness over convenience).

**Validation & Limits:** Access links are single-use and time-limited (short expiry, on the order of minutes to a couple of hours); a client can never view another phone number's bookings even if they guess or mistype one.

**Access:** Open to any Client using their own phone number; a client can never see another client's bookings under any circumstance — this is a hard privacy boundary from BRIEF.md's Constraints. The Pro does not use this mechanism (the Pro has their own dashboard login, out of this feature's scope). Platform Operator (Support): None — support access does not use or bypass client identity.

**Communications:** Sends the one-tap access link by text (or email fallback per BRIEF.md's messaging constraint) each time one is requested.

**Data Notes:** Captured: nothing new — reuses the phone number captured at booking (FEAT-05). Displayed: the client's own booking list. Derived: none. Source: matched against existing Client records for this pro only.

**Interactions:** Depended on by Client-Initiated Cancel/Reschedule (FEAT-10); reads Client (created by FEAT-05) and Booking.

**Signals:** access_link_requested, access_link_used, access_link_expired_unused.

### Deposit Payment at Booking

**ID:** FEAT-07

**Description:** The client pays a card deposit — a fixed amount or a percentage of the service price, per the Pro's own rule — at the moment of booking, with the balance left due in person at the appointment.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision states the client "pays a card deposit," and the Business Context is explicit that the platform never stores or handles card data itself and takes no per-booking cut. This is the mechanism that solves the founder's core problem: deposits that used to be asked for by hand and often never arrived.

**Connected Entities:** Deposit Transaction (create), Booking (update — marks as paid/confirmed), Service (read — for the deposit rule)

**Key Capabilities:**
- Pay the exact deposit amount required by the selected service's rule, by card
- See a clear on-screen and confirmed record that the deposit succeeded
- Have a failed or declined payment explained clearly, with the slot held briefly to retry

**Primary Flows & Alternates:**
- Happy path: client enters card details at the payment step; the deposit is authorized and captured; the booking flips from pending to confirmed instantly.
- Alternate: the card is declined; the client sees the decline reason from the processor in plain language and can retry with another card without losing the held slot (within its short hold window).
- Alternate: payment succeeds but the confirmation step fails to load; the booking is still correctly confirmed server-side and the client is shown the confirmation on next page load or via the confirmation text, never double-charged and never left unsure whether they are booked.

**States:** Empty: N/A — payment is always tied to an in-progress booking, never a standalone screen. Loading: a clear "processing payment, do not close this page" state during authorization. Error: a specific, actionable message for each decline reason available from the processor. Offline-degraded: payment requires connectivity by nature; a connection drop mid-payment is treated as a failure with a safe retry, never an ambiguous charge.

**Validation & Limits:** Deposit amount is computed exactly from the service's rule (fixed amount, or percentage rounded to the nearest currency unit) and cannot be altered by the client; one deposit charge per booking.

**Access:** The Client pays for their own booking only. The Pro sees the resulting deposit status on their dashboard (Full/View) but never the card number itself — the processor owns all card data, per BRIEF.md's Constraints. Platform Operator (Support) has View-only access to transaction status, never to card data.

**Communications:** Feeds the confirmation message in Automated Booking Messaging (FEAT-08); a payment failure shows in-flow only and sends no separate message.

**Data Notes:** Captured: deposit amount and payment outcome. Displayed: paid/unpaid status on bookings, on both the client's confirmation and the Pro's dashboard. Derived: none — amount is computed once from Service at the moment of booking and then fixed. Source: the payment-processing capability the product depends on (BRIEF.md, Business Context) for the actual card handling; Chairtime holds only the outcome and amount.

**Interactions:** Depends on Service & Pricing Management (FEAT-01) and Public Booking Page & Booking Flow (FEAT-05); feeds Cancellation & No-Show Policy Engine (FEAT-09), No-Show Marking & Deposit Forfeiture (FEAT-11), and Booking & Payment Activity Record (FEAT-16).

**Signals:** deposit_payment_attempted, deposit_payment_succeeded, deposit_payment_failed (with decline reason category).

### Automated Booking Messaging

**ID:** FEAT-08

**Description:** The client receives an immediate confirmation the moment a booking is paid, and an automatic reminder before the appointment with a one-tap "I'll be there / I need to reschedule" response — replacing the Pro's habit of texting reminders by hand.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision states this exactly: "a confirmation text lands immediately... a reminder arrives with a one-tap 'I'll be there / I need to reschedule.'" This directly replaces the founder's stated evening admin burden and is central to the "pro never chases" success criterion.

**Connected Entities:** Message (create), Booking (read), Messaging Consent (read)

**Key Capabilities:**
- Send an immediate confirmation message on successful booking
- Send an automatic reminder a set time before the appointment (two days, per BRIEF.md's example)
- Offer a one-tap "I'll be there" or "I need to reschedule" response from the reminder itself

**Primary Flows & Alternates:**
- Happy path: booking completes -> confirmation text sent within moments; two days before the appointment -> reminder sent with the one-tap options; tapping "I'll be there" simply acknowledges, tapping "I need to reschedule" routes into Client-Initiated Cancel/Reschedule (FEAT-10).
- Alternate: the client never opted in to texting; the confirmation and reminder are sent by email instead, per BRIEF.md's fallback ("Email confirmations are acceptable as a fallback").
- Alternate: a text fails to deliver; the system retries once and, on continued failure, falls back to email and flags the delivery gap on the Pro's dashboard so the Pro is never blindsided by a client who "never got a reminder."

**States:** Empty: N/A — messages only ever exist tied to a booking. Loading: N/A — sending happens asynchronously in the background with no user-facing wait. Error: a delivery failure is retried and then falls back to email; it is never silently dropped. Offline-degraded: N/A — this is a server-side sending capability, not a client-facing interactive screen.

**Validation & Limits:** No message is sent to a phone number without active Messaging Consent (FEAT-14); reminder timing defaults to two days before the appointment, matching BRIEF.md's stated example.

**Access:** Only the Client tied to that Booking receives its messages; the Pro sees message history for their own bookings (via FEAT-16) but does not receive the client's replies as raw texts — only the resulting reschedule/confirmation status. Platform Operator (Support) has View-only access to delivery status for troubleshooting.

**Communications:** This feature *is* the communications: booking confirmation, pre-appointment reminder, and the reminder's one-tap reply handling.

**Data Notes:** Captured: message content summary and delivery status per send. Displayed: delivery status to the Pro (via FEAT-16). Derived: reminder timing is derived from the Booking's appointment time. Source: Booking and Messaging Consent records.

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), and Messaging Consent Management (FEAT-14); feeds Client-Initiated Cancel/Reschedule (FEAT-10) (via the reschedule tap) and Booking & Payment Activity Record (FEAT-16).

**Signals:** confirmation_sent, reminder_sent, reminder_reply_confirmed, reminder_reply_reschedule_requested, message_delivery_failed.

### Cancellation & No-Show Policy Engine

**ID:** FEAT-09

**Description:** The Pro defines their own cancellation window and what happens to the deposit inside vs. outside it; the system enforces that policy automatically and consistently on every cancellation and no-show, exactly as the client agreed to it at booking.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Business Context is exact: "a cancellation inside the pro's window forfeits the deposit... a cancellation outside the window refunds the deposit automatically," "under the pro's own cancellation policy that the client agreed to when booking." This is the mechanism behind the founder's headline promise: "the pro never chases a no-show again."

**Connected Entities:** Cancellation Policy (create, update), Deposit Transaction (update — refund or forfeit), Booking (read)

**Key Capabilities:**
- Set the cancellation/reschedule window (e.g., 24 hours before appointment)
- Automatically refund the deposit for a cancellation made outside the window
- Automatically flag a cancellation inside the window (or a no-show) for deposit forfeiture, applied via FEAT-11

**Primary Flows & Alternates:**
- Happy path: client cancels well outside the window -> deposit refunds automatically, no Pro action needed.
- Alternate: client cancels inside the window -> the client is shown, before confirming, exactly what will happen to their deposit under this pro's policy, so there is no surprise or dispute later.
- Alternate: Pro changes their policy going forward; every already-confirmed booking is still governed by the policy version shown to the client at the time they booked, never retroactively changed underneath them.

**States:** Empty: N/A — a default, sensible cancellation window is proposed during onboarding (FEAT-15) so no Pro Account exists without an active policy. Loading: N/A — policy evaluation is instantaneous at the moment of cancellation. Error: if automatic refund processing fails, the outcome is flagged clearly on the Pro's dashboard rather than silently failing. Offline-degraded: N/A — enforcement happens server-side regardless of either party's connectivity at the time.

**Validation & Limits:** The cancellation window must be a positive number of hours before the appointment; the policy in force is always the one shown to the client at the moment they booked (versioned, never edited retroactively for existing bookings).

**Access:** The Pro has Full access to set the policy. The Client sees the current policy in plain words during booking and at cancellation time (Own-only, read). Platform Operator (Support) has View-only access.

**Communications:** N/A — the policy's application is communicated in-flow (at booking and at cancellation), not as a separate standalone message; the outcome (refund confirmed / deposit kept) is included in the relevant confirmation.

**Data Notes:** Displayed: the plain-language policy on the booking page and at cancellation. Derived: the refund-vs-forfeit outcome for every cancellation is derived from comparing the cancellation timestamp to the booking's appointment time and the policy's window. Source: pro-set policy plus system clock.

**Interactions:** Feeds Public Booking Page & Booking Flow (FEAT-05) (policy display), Client-Initiated Cancel/Reschedule (FEAT-10), and No-Show Marking & Deposit Forfeiture (FEAT-11).

**Signals:** cancellation_policy_updated, cancellation_within_window_flagged, deposit_refund_triggered.

### Client-Initiated Cancel/Reschedule

**ID:** FEAT-10

**Description:** A client can cancel or reschedule their own booking, within the Pro's stated policy, without a phone call or a DM — from the reminder's one-tap option or the client's own booking-management link.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Client "reschedules or cancels within the policy window" as a defined capability, and the Vision's reminder flow ("I need to reschedule") depends on it existing.

**Connected Entities:** Booking (update — cancel or reschedule), Deposit Transaction (read — for policy outcome preview), Cancellation Policy (read)

**Key Capabilities:**
- Cancel an upcoming booking and see the deposit outcome before confirming
- Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy
- See the applicable cancellation window countdown before acting

**Primary Flows & Alternates:**
- Happy path: client accesses their booking (via FEAT-06), taps reschedule, picks a new free slot; the booking updates and both parties are notified.
- Alternate: client cancels inside the policy window; they see the deposit-forfeiture outcome plainly before confirming, so the action is never a surprise.
- Alternate: the desired new time is not available; the client sees the same real-time slot list as a fresh booking, never a stale or misleading option.

**States:** Empty: N/A — this flow only exists against an existing booking. Loading: brief indicator while re-checking live availability for a reschedule. Error: if the update fails to save, the original booking remains untouched and intact rather than left in an ambiguous state. Offline-degraded: requires connectivity, consistent with the availability engine's correctness-first stance.

**Validation & Limits:** A reschedule must land on a slot that passes the same validation as a new booking (FEAT-03); a booking already marked completed or no-show cannot be cancelled or rescheduled.

**Access:** Own-only for the Client (their own booking only, verified via FEAT-06). The Pro sees the resulting change on their dashboard (Full/View) and can also cancel/reschedule on the client's behalf as part of their own booking management. Platform Operator (Support) has View-only access.

**Communications:** Triggers a cancellation or reschedule confirmation to the client and a change notice to the Pro's dashboard (via FEAT-08's messaging mechanism).

**Data Notes:** Captured: the new time (if rescheduling) or cancellation timestamp. Displayed: updated booking status to both parties. Derived: deposit outcome, via Cancellation & No-Show Policy Engine (FEAT-09).

**Interactions:** Depends on Client Booking Identity (FEAT-06), Real-Time Slot Availability Engine (FEAT-03), Cancellation & No-Show Policy Engine (FEAT-09); feeds Automated Booking Messaging (FEAT-08), Waitlist for Cancelled Slots (FEAT-20), and Booking & Payment Activity Record (FEAT-16).

**Signals:** booking_cancelled_by_client, booking_rescheduled_by_client.

### No-Show Marking & Deposit Forfeiture

**ID:** FEAT-11

**Description:** The Pro marks a booking as a no-show when a client fails to appear, and the deposit is forfeited to the Pro automatically under the agreed policy — with no manual chasing, invoicing, or renegotiation required.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the founder's headline promise verbatim: "the pro never chases a no-show again," and BRIEF.md's Success Criteria states "Pros say 'I haven't had an unpaid no-show since I switched.'" The mechanism must be a single, low-effort action for the Pro.

**Connected Entities:** Booking (update — mark no-show), Deposit Transaction (update — forfeit)

**Key Capabilities:**
- Mark a past-due booking as a no-show in one tap from the daily schedule
- See the deposit automatically reflected as kept, with no separate invoicing step
- Reverse a mistaken no-show mark (e.g., the client did show up) within a short grace period

**Primary Flows & Alternates:**
- Happy path: appointment time passes with the client absent; the Pro taps "no-show" from the dashboard; the deposit is marked forfeited automatically, and the record is retained for any future dispute.
- Alternate: Pro mistakenly marks a no-show; they can undo it within a short grace period, restoring the booking to completed and the deposit to its prior state.
- Alternate: a client disputes the no-show later; the Pro (or Platform Operator Support, if asked to help) can pull up the exact booking, its agreed policy version, and its timeline from Booking & Payment Activity Record (FEAT-16) as evidence.

**States:** Empty: N/A — this action only appears against a specific past-due booking. Loading: N/A — instantaneous local action. Error: a failed forfeiture write is retried and flagged, never silently dropped, since money is at stake. Offline-degraded: marking requires connectivity so the forfeiture is recorded reliably and immediately — this is exactly the "never lose a deposit" correctness bar from BRIEF.md.

**Validation & Limits:** A booking can only be marked no-show after its appointment time has passed; the undo grace period is short and fixed (on the order of a day) to prevent indefinite ambiguity in the client's own records.

**Access:** The Pro has Full access to mark and unmark no-shows on their own bookings only. The Client sees the outcome (deposit kept) reflected in their own booking history but cannot mark or dispute it in-app beyond contacting the Pro directly. Platform Operator (Support) has View-only access, useful for dispute troubleshooting.

**Communications:** N/A — the deposit outcome is visible in the client's own booking history rather than triggering a separate confrontational notification; the Pro's action is deliberately low-friction and silent toward the client.

**Data Notes:** Captured: the no-show marking action and timestamp. Displayed: updated booking status and deposit outcome to the Pro; deposit status to the client. Derived: the forfeiture amount, from Cancellation & No-Show Policy Engine (FEAT-09).

**Interactions:** Depends on Cancellation & No-Show Policy Engine (FEAT-09) and Deposit Payment at Booking (FEAT-07); feeds Booking & Payment Activity Record (FEAT-16) and Booking & Revenue Insights (FEAT-25).

**Signals:** booking_marked_no_show, no_show_mark_undone, deposit_forfeited.

### Pro Daily Schedule Dashboard

**ID:** FEAT-12

**Description:** The Pro's primary, phone-first view: today's (and upcoming) bookings, each with a paid badge, a client note, and how much balance is still due in person — the screen the Pro glances at between clients.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision describes this exactly: "As the pro, you glance at your phone between clients: today's list, each booking with a paid badge, a client note, and how much is still due in person." This is the Pro's single most frequent touchpoint with the product.

**Connected Entities:** Booking (read, update — quick actions), Client (read), Deposit Transaction (read)

**Key Capabilities:**
- View today's bookings at a glance, in time order, with paid/unpaid and balance-due status
- View upcoming bookings beyond today
- Take quick actions directly from the list: mark no-show, view client note, jump to reschedule/cancel

**Primary Flows & Alternates:**
- Happy path: Pro opens the dashboard between clients and sees the day's remaining bookings, each with status, in seconds.
- Alternate: an empty day (no bookings) shows a plain, encouraging state rather than looking broken, with a shortcut to share the booking link.
- Alternate: a booking's calendar-sync status is uncertain (FEAT-04 flagged an issue); the dashboard visibly marks that booking's reliability rather than presenting it with false confidence.

**States:** Empty: a day with zero bookings shows a friendly "nothing booked yet today" state, never a bare blank screen. Loading: bookings render with a lightweight in-place indicator on slow connections. Error: a failed load shows the last successfully loaded data with a retry action, never an unexplained blank dashboard. Offline-degraded: the most recently loaded schedule remains viewable read-only; actions (like marking no-show) require reconnecting.

**Validation & Limits:** N/A — this is a read-and-quick-action view; no new data is created here beyond the quick actions themselves, which are validated by their own features (FEAT-11, FEAT-10).

**Access:** The Pro has Full access to their own schedule only. Clients have no access to this view. Platform Operator (Support) has View-only access for troubleshooting a specific reported issue.

**Communications:** N/A — this is a viewing surface; it does not itself send messages.

**Data Notes:** Displayed: booking time, service, client name/note, paid status, balance due. Derived: balance due (service price minus deposit paid). Source: reads Booking, Client, and Deposit Transaction records created elsewhere.

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), Client Record Management (FEAT-13); surfaces quick actions into No-Show Marking & Deposit Forfeiture (FEAT-11) and Client-Initiated Cancel/Reschedule (FEAT-10, pro-initiated side).

**Signals:** dashboard_viewed, quick_action_taken (with action type).

## Important Features

### Client Record Management

**ID:** FEAT-13

**Description:** The Pro maintains a simple record for each client — contact details, private notes, and booking history with that Pro — and can permanently delete a client's record on request.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Pro "sees every... client" and "can delete a client's record on request," and the Constraints section makes deletion-on-request an explicit regulatory-adjacent obligation. Ranked Important rather than Core because a client record is created automatically by the act of booking (FEAT-05) — this feature is about managing that record afterward, not about the core booking loop itself. MVP phase: the delete-on-request obligation is a launch-blocking privacy commitment, not something safe to defer.

**Connected Entities:** Client (read, update, delete)

**Key Capabilities:**
- View a client's contact details and full booking history with this Pro
- Add or edit a private note about a client (preferences, allergies noted informally, etc.)
- Permanently delete a client's record on their request

**Primary Flows & Alternates:**
- Happy path: Pro opens a client from the dashboard or booking list, reviews history, and adds a quick note after an appointment.
- Alternate: a client requests deletion; the Pro deletes the record, which removes contact details and notes while past financial records needed for dispute/audit purposes (FEAT-16) are retained in de-identified form per data-retention obligations.
- Alternate: a deleted client books again later; a new client record is created — the system does not silently resurrect the old one.

**States:** Empty: a client with no notes yet shows a plain empty note field, not an error. Loading: N/A — instant for the small per-pro client volumes described in BRIEF.md (100–500 clients). Error: a failed save preserves entered text with a retry option. Offline-degraded: the most recently loaded client list remains viewable read-only.

**Validation & Limits:** Notes are free text with a generous but bounded length (e.g., up to 1,000 characters); deletion is a deliberate, confirmed action (not reversible, consistent with "delete on request" meaning delete).

**Access:** The Pro has Full access to their own clients only. A Client can view their own contact details implicitly through their own booking history (FEAT-06) but does not see the Pro's private notes about them. Platform Operator (Support) has View-only access, explicitly excluding the Pro's private notes field, to respect the client's privacy even in support contexts.

**Communications:** N/A — record management itself sends no notifications; a deletion is silent to the client beyond honoring their own request.

**Data Notes:** Captured: private notes. Displayed: contact details, notes, booking history. Derived: none. Source: contact details from Public Booking Page & Booking Flow (FEAT-05); notes are direct pro input.

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05) (creates the Client); feeds Pro Daily Schedule Dashboard (FEAT-12) and Client List Search & Filter (FEAT-24).

**Signals:** client_note_added, client_record_viewed, client_record_deleted.

### Messaging Consent Management

**ID:** FEAT-14

**Description:** The system that captures a client's explicit opt-in to text messaging at booking, lets a client withdraw consent at any time, and ensures every reminder and confirmation respects the current consent state.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Constraints state plainly: "clients must explicitly agree to receive texts when they book, and reminders must respect that consent," citing US texting rules as the reason. Ranked Important rather than Core because it is a compliance-and-preference layer underneath the Core messaging feature (FEAT-08) rather than a capability a client seeks out on its own; it must still ship in MVP because it is a regulatory precondition for FEAT-08 to operate lawfully.

**Connected Entities:** Messaging Consent (create, update)

**Key Capabilities:**
- Capture explicit opt-in at the moment of booking (never pre-checked)
- Let a client withdraw consent at any time via a link included in messages
- Fall back to email automatically for any client without active texting consent

**Primary Flows & Alternates:**
- Happy path: client checks the opt-in box at booking; all future confirmations and reminders for that pro go by text.
- Alternate: client replies "STOP" or uses an opt-out link; texting consent is revoked immediately and future messages fall back to email.
- Alternate: a client never opted in at all; every message for their bookings goes by email from the start, with no degraded experience implied.

**States:** Empty: N/A — consent is always tied to a specific client-pro relationship, captured at first booking. Loading: N/A — instantaneous local state. Error: a failed consent-state update is treated conservatively — if in doubt, the system defaults to the safer (no-text) state rather than risk texting without valid consent. Offline-degraded: N/A — this is a background compliance state, not an interactive screen.

**Validation & Limits:** Consent must be an explicit, unchecked-by-default action; a revoke request is honored on the very next message sent, with no grace period.

**Access:** A Client manages only their own consent (Own-only). The Pro sees whether a given client can currently be texted (View, for planning purposes) but cannot override a client's revoked consent. Platform Operator (Support) has View-only access.

**Communications:** N/A — this feature governs communications rather than sending its own, aside from an opt-out confirmation acknowledgment.

**Data Notes:** Captured: opt-in/opt-out state and timestamp. Displayed: current consent status to the Pro (for planning) and to the client (in their own preferences). Derived: none. Source: direct client action at booking or via an opt-out link.

**Interactions:** Depended on by Automated Booking Messaging (FEAT-08); fed by Public Booking Page & Booking Flow (FEAT-05).

**Signals:** consent_granted, consent_revoked, message_routed_to_fallback_email.

### Pro Onboarding & Setup Wizard

**ID:** FEAT-15

**Description:** The guided, one-time setup flow that takes a brand-new Pro from signup to a live, shareable booking link — services, hours, deposit rule, cancellation policy, and calendar connection, in a sensible order with sensible defaults.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** Every product has a first run, and this one has an unusually high stakes first run: BRIEF.md's Constraints name a three-month runway to the first paying pro, so setup friction directly threatens the founder's timeline. Ranked Important rather than Core because, once complete, the wizard itself is never used again — the Core features it configures are what deliver ongoing value. MVP phase: a Pro cannot reach any Core feature without it.

**Connected Entities:** Pro Account (create), Service (create), Availability Rule (create), Cancellation Policy (create), Subscription (create), Calendar Connection (create — offered, not required, to complete)

**Key Capabilities:**
- Guided, ordered setup: account -> at least one service -> working hours -> deposit rule -> cancellation policy -> (optional) calendar connection -> subscription payment
- Sensible defaults offered at each step (e.g., a common cancellation window) that the Pro can accept or change
- A shareable booking link generated the moment setup is minimally complete

**Primary Flows & Alternates:**
- Happy path: Pro signs up, moves through each step in order, accepting or adjusting defaults, and receives their shareable link at the end.
- Alternate: Pro abandons setup partway through; on return, the wizard resumes exactly where they left off with earlier answers preserved, never forcing a restart.
- Alternate: Pro skips connecting a calendar during setup; they can complete the rest of onboarding and connect it later from settings without being blocked.

**States:** Empty: N/A — the wizard itself is the empty-state handler for a new account. Loading: N/A — each step is a simple form with instant local response. Error: a failed step preserves entered values with a retry option, consistent with every setup screen in this product. Offline-degraded: N/A — setup is a deliberate, connected session, not an in-the-moment mobile flow.

**Validation & Limits:** The booking link is not generated until the minimum required steps (account, one service, working hours, deposit rule, cancellation policy, active subscription) are complete; calendar connection is the one optional step.

**Access:** The Pro has Full access to their own onboarding. No other role touches this feature. Platform Operator (Support) has View-only access to see how far a specific pro has progressed, useful for support.

**Communications:** A welcome confirmation once the booking link goes live.

**Data Notes:** Captured: every setup field listed under Connected Entities. Displayed: setup progress and the resulting live link. Derived: none. Source: direct pro input at each step.

**Interactions:** Feeds Service & Pricing Management (FEAT-01), Availability & Working Hours Setup (FEAT-02), Cancellation & No-Show Policy Engine (FEAT-09), Two-Way Calendar Sync (FEAT-04), Pro Subscription Billing & Account Management (FEAT-18).

**Signals:** onboarding_started, onboarding_step_completed (with step name), onboarding_completed, onboarding_link_shared.

### Booking & Payment Activity Record

**ID:** FEAT-16

**Description:** An always-on, append-only record of every booking's key events — created, paid, confirmed, messaged, cancelled/rescheduled, marked no-show, refunded/forfeited — so the Pro (or, when asked to help, Platform Operator Support) has a trustworthy timeline to point to if a client ever disputes a charge.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Problem Statement names this precisely as a current failure: "no record when a client disputes a no-show charge." Ranked Important rather than Core because it is a record-keeping layer that supports the Core booking/deposit/no-show features rather than something a user directly seeks out day to day; it must still ship at MVP because the dispute scenario it prevents is a launch-day risk, not a later refinement.

**Connected Entities:** Booking (read), Deposit Transaction (read), Message (read)

**Key Capabilities:**
- View a chronological timeline of everything that happened to a specific booking
- See exactly which cancellation policy version applied and when it was shown to the client
- Reference this record when responding to a client's dispute

**Primary Flows & Alternates:**
- Happy path: a client disputes a no-show charge; the Pro opens the booking's timeline and sees the exact policy shown at booking, the appointment time, and the no-show mark's timestamp.
- Alternate: Platform Operator (Support) is asked to help with a dispute; they can view the same timeline read-only without being able to alter it.
- Alternate: a booking has an unusual gap (e.g., a message failed to send); the timeline shows that gap plainly rather than presenting a falsely clean record.

**States:** Empty: N/A — a timeline only exists for bookings that have happened; a brand-new booking simply starts its timeline at "created." Loading: N/A — small per-booking dataset, loads instantly. Error: N/A — this is a read-only, append-only log; there is no user-facing write path to fail. Offline-degraded: the most recently loaded timeline remains viewable read-only.

**Validation & Limits:** Entries are append-only and immutable once written — this is the property that makes the record trustworthy as dispute evidence; retained for as long as the associated Booking record exists.

**Access:** The Pro has Full (view) access to their own bookings' timelines. Platform Operator (Support) has View-only access. Clients do not see this internal timeline directly — they see the outcomes (their own confirmation, deposit status) through their own booking view, not this operational record.

**Communications:** N/A — this feature is a passive record, not a message sender.

**Data Notes:** Displayed: a chronological event list per booking. Derived: entirely — every entry is written automatically as other features act on the booking; nothing is directly entered here.

**Interactions:** Reads from Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), Automated Booking Messaging (FEAT-08), Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11).

**Signals:** activity_record_viewed.

### Manual Time Blocking

**ID:** FEAT-17

**Description:** The Pro can block off a span of time — a doctor's appointment, a vacation day, a personal commitment — removing it from bookable availability without needing to edit their recurring working hours.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** A direct, near-universal need once recurring hours exist: a Pro's actual availability always has one-off exceptions. Without it, the Pro would be forced to edit recurring hours for a single day, which is error-prone and easy to forget to revert. Important rather than Core: the product still functions on recurring hours alone at a pinch, but reliability (a hallmark of this brief) suffers without it. MVP phase: this is a day-one operational need, not a later refinement.

**Connected Entities:** Time Block (create, update, delete)

**Key Capabilities:**
- Block a span of time on a specific date (or a recurring pattern, e.g., "every Sunday")
- Remove a block to restore availability
- See blocked time reflected immediately in the slot engine

**Primary Flows & Alternates:**
- Happy path: Pro blocks tomorrow afternoon; that window disappears from bookable slots immediately.
- Alternate: a block is added over an already-booked slot; the existing booking is never silently affected — the Pro is warned and must explicitly decide (contact the client, or leave the booking as an exception to the block).
- Alternate: Pro removes a block early; the previously blocked time becomes bookable again right away.

**States:** Empty: a Pro with no blocks sees a plain "no time blocked" state. Loading: N/A — instant, small dataset. Error: a failed save is retried with entered values preserved. Offline-degraded: N/A — a setup-style action requiring connectivity for correctness.

**Validation & Limits:** A block's end time must be after its start time; a block cannot silently delete an existing conflicting booking.

**Access:** The Pro has Full access to their own blocks. Clients never see blocks directly — only their absence from available slots. Platform Operator (Support) has View-only access.

**Communications:** N/A — blocking time is a private scheduling action with no message trigger.

**Data Notes:** Captured: block start/end and an optional label (private to the Pro). Displayed: on the Pro's own schedule view. Derived: none.

**Interactions:** Feeds Real-Time Slot Availability Engine (FEAT-03).

**Signals:** time_block_added, time_block_removed, time_block_conflict_flagged.

### Pro Subscription Billing & Account Management

**ID:** FEAT-18

**Description:** The Pro's own flat monthly subscription to Chairtime — one price tier, card-based, cancel anytime — including seeing their current plan status and updating their payment method.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** BRIEF.md's Business Context is explicit: "revenue comes from a flat monthly subscription paid by each pro... cancel anytime, with one price tier in v1... no per-booking cut." Ranked Important rather than Core because it is the business's monetization mechanism rather than part of the client-facing booking loop the founder's headline promise describes; it must still ship at MVP because the product has no revenue model without it.

**Connected Entities:** Subscription (create, update, cancel)

**Key Capabilities:**
- Subscribe during onboarding with a card
- View current plan status and next billing date
- Update the payment method on file
- Cancel the subscription at any time, effective at the end of the current billing period

**Primary Flows & Alternates:**
- Happy path: Pro enters payment details once during onboarding; the subscription renews automatically each month with no further action.
- Alternate: a renewal payment fails; the Pro is notified and given a grace period to update their payment method before the account is paused (booking page taken offline to new bookings, but existing bookings and data preserved).
- Alternate: Pro cancels; the subscription remains active through the already-paid period and then lapses, with the account paused (not deleted) afterward.

**States:** Empty: N/A — an account cannot exist past onboarding without an active subscription. Loading: N/A — plan status is a small, instant read. Error: a failed payment update is retried with a clear reason and retry action. Offline-degraded: the last known plan status remains viewable read-only.

**Validation & Limits:** One price tier only in v1 — no plan selection is offered; cancellation takes effect at the end of the already-paid period, never an immediate mid-period cutoff that would feel like losing paid time.

**Access:** The Pro has Full access to their own subscription. Platform Operator (Support) has View-only access to plan status, useful for billing support questions. Clients have no visibility into this at all.

**Communications:** Payment-failure notice with a grace-period deadline; renewal receipt; cancellation confirmation.

**Data Notes:** Captured: subscription status, billing cycle, payment method reference (the payment-processing capability owns the actual card data, per BRIEF.md's Constraints). Displayed: current plan status and next billing date. Derived: none.

**Interactions:** Created during Pro Onboarding & Setup Wizard (FEAT-15); referenced by Platform Support Read-Only Access (FEAT-19).

**Signals:** subscription_started, subscription_payment_failed, subscription_payment_recovered, subscription_cancelled.

### Platform Support Read-Only Access

**ID:** FEAT-19

**Description:** The founder, in a support capacity, can open a read-only view into a specific Pro's account — services, schedule, bookings, and billing status — to help troubleshoot a reported problem, with no ability to edit anything and no client-facing access of any kind.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Target Users & Roles names this directly: "the founder needs only a read-only support view of a pro's account to help them... It is minimal admin access." Ranked Important rather than Core because it serves the business's operational need rather than either product role's own value; still needed at MVP because support requests will arrive from day one with real, paying pros.

**Connected Entities:** Pro Account (read), Service (read), Booking (read), Client (read — excluding private notes), Subscription (read)

**Key Capabilities:**
- Look up a specific Pro's account by request
- View their services, schedule, bookings, and billing status read-only
- View booking timelines (FEAT-16) to help resolve a dispute

**Primary Flows & Alternates:**
- Happy path: a Pro reports a confusing issue; the founder opens the read-only view, diagnoses it (e.g., a lapsed calendar connection), and guides the Pro to fix it themselves.
- Alternate: the founder attempts an action outside read-only scope (there is none available in this view by design) — the interface simply offers no edit controls at all, removing the possibility rather than blocking it after the fact.
- Alternate: the founder is asked about a client dispute; they view the relevant booking's activity record (FEAT-16) but not the Pro's private client notes, respecting the client-record privacy boundary even in support.

**States:** Empty: N/A — this view only exists once a specific Pro Account is looked up. Loading: N/A — small per-account dataset. Error: N/A — a read-only view with no write path to fail. Offline-degraded: N/A — an operational tool used in a connected context.

**Validation & Limits:** No write actions exist in this view at all — the constraint is structural, not a permission check that could be bypassed.

**Access:** Platform Operator (Support) has View access product-wide, scoped to one account at a time. Neither the Pro nor the Client has any awareness that this access exists as a routine matter — it is used only when a Pro has asked for help.

**Communications:** N/A — this is an internal tool with no client- or pro-facing messages of its own.

**Data Notes:** Displayed: read-only mirror of the Pro's own data, minus the Pro's private client notes. Derived: none. Source: reads existing records only; creates nothing.

**Interactions:** Reads Pro Onboarding & Setup Wizard (FEAT-15) output, Pro Daily Schedule Dashboard (FEAT-12) data, Booking & Payment Activity Record (FEAT-16), Pro Subscription Billing & Account Management (FEAT-18).

**Signals:** support_view_opened (with reason/ticket reference).

## Nice-to-Have Features

### Waitlist for Cancelled Slots

**ID:** FEAT-20

**Description:** A client can ask to be notified if a specific service and day opens up from someone else's cancellation, instead of repeatedly checking the booking page.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions names this exactly: "should a pro be able to offer a waitlist for slots that open up from cancellations?" As Visionary judgment: valuable but not required for the core one-minute-booking promise, and it depends on Client-Initiated Cancel/Reschedule (FEAT-10) already existing to generate openings. Phased to v1, once the core cancellation flow is proven.

**Connected Entities:** Waitlist Entry (create, update), Service (read)

**Key Capabilities:**
- Join a waitlist for a specific service/day when no slot is currently free
- Get notified the moment a matching slot opens from a cancellation
- Book directly from the notification before anyone else can grab the slot

**Primary Flows & Alternates:**
- Happy path: client finds no free slot, joins the waitlist for that day; a cancellation opens a matching slot; the client is notified and books within a short priority window.
- Alternate: two clients are on the waitlist for the same opening; the first to act on the notification gets it, and the other remains on the waitlist for the next opportunity.
- Alternate: the waitlist entry expires unclaimed after a set period with no matching opening; the client is informed rather than left wondering indefinitely.

**States:** Empty: no waitlist entries shows a plain "you're not on any waitlists" state. Loading: N/A — small dataset. Error: a failed join is retried. Offline-degraded: N/A — requires connectivity to reliably capture a fast-moving opening.

**Validation & Limits:** A waitlist entry is tied to one service and one day (or a small date range); a client may hold a small, bounded number of active waitlist entries at once.

**Access:** Own-only for the Client. The Pro has View access to see how many people are waitlisted for a given day (useful context, not an action). Platform Operator (Support) has View-only access.

**Communications:** A notification the moment a matching slot opens, with a short window to claim it before it returns to general availability.

**Data Notes:** Captured: requested service and date range. Displayed: waitlist position/status to the client; aggregate waitlist demand to the Pro. Derived: none.

**Interactions:** Depends on Client-Initiated Cancel/Reschedule (FEAT-10) (the source of openings) and Real-Time Slot Availability Engine (FEAT-03).

**Signals:** waitlist_joined, waitlist_notified, waitlist_converted_to_booking, waitlist_expired.

### Recurring/Standing Appointments

**ID:** FEAT-21

**Description:** A client can set up a standing appointment pattern (e.g., "every 3 weeks") with the same Pro, generating individual bookings automatically instead of booking fresh each time.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions asks directly whether this is "v1 or later." As Visionary judgment: valuable for retention-style services (lash fills, haircuts) but not required for the founder's three-month first-paying-pro timeline, and it adds real complexity to the availability engine. Phased to v1, once the single-booking core loop is proven reliable.

**Connected Entities:** Recurring Series (create, update, cancel), Booking (create — generated per occurrence)

**Key Capabilities:**
- Set up a recurring pattern from an existing booking ("repeat this every N weeks")
- See and manage the upcoming generated occurrences as a group
- Cancel the whole series, or just one upcoming occurrence, independently

**Primary Flows & Alternates:**
- Happy path: after booking, the client opts into a recurring pattern; future occurrences are generated and each still requires its own deposit per the Pro's rule.
- Alternate: a future occurrence's usual time is no longer available (the Pro changed hours); the client is notified in advance and asked to pick a new time for that occurrence only, without breaking the rest of the series.
- Alternate: client cancels just one occurrence; the series continues generating future ones normally.

**States:** Empty: a client with no recurring series sees no extra UI at all — this is fully optional. Loading: N/A — series management is a small, occasional action. Error: a failed occurrence generation is retried and, if it keeps failing, surfaces to the Pro as a flagged gap rather than a silently missed appointment. Offline-degraded: N/A — requires connectivity for correctness, consistent with the rest of scheduling.

**Validation & Limits:** A series must have a bounded interval (a reasonable minimum/maximum recurrence period); each generated occurrence is still subject to the same slot validation as any booking (FEAT-03).

**Access:** Own-only for the Client (their own series). The Pro has Full/View access to see and manage series tied to their own schedule. Platform Operator (Support) has View-only access.

**Communications:** A confirmation when a new occurrence is generated, and an advance notice if an occurrence needs a new time.

**Data Notes:** Captured: recurrence interval and originating service/time. Displayed: the series and its upcoming occurrences. Derived: each occurrence's specific Booking record is derived from the series pattern.

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05) and Real-Time Slot Availability Engine (FEAT-03) (which reserves future occurrence slots).

**Signals:** recurring_series_created, recurring_occurrence_generated, recurring_series_cancelled.

### In-App Balance Payment

**ID:** FEAT-22

**Description:** A client can optionally pay the remaining balance (beyond the deposit) through the app before or at the appointment, instead of paying the Pro directly in person.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions asks directly whether the balance "should be payable in the app, or stay in person." As Visionary judgment: the brief's default flow ("the balance is due at the appointment") works fine without this, so it is not required for MVP, but it is a natural, low-risk enhancement once deposit payment (FEAT-07) is proven. Phased to v1.

**Connected Entities:** Deposit Transaction (read), Booking (read, update — balance payment status)

**Key Capabilities:**
- Pay the remaining balance in-app at any point before or at the appointment
- See a running record of deposit paid vs. balance remaining

**Primary Flows & Alternates:**
- Happy path: client opens their booking and pays the balance in-app; the Pro's dashboard reflects "fully paid" instead of "balance due."
- Alternate: client chooses to pay in person instead — this remains the default, unaffected experience; in-app balance payment is purely additive, never required.
- Alternate: an in-app balance payment fails; the booking remains marked "balance due" exactly as if the client had never attempted it, with no partial or ambiguous state.

**States:** Empty: N/A — only appears against an existing confirmed booking with a balance due. Loading: standard payment-processing indicator. Error: a specific decline message, matching the deposit payment feature's pattern. Offline-degraded: requires connectivity, consistent with all payment actions.

**Validation & Limits:** The balance amount is fixed by the service price minus the deposit already paid and cannot be altered by the client.

**Access:** Own-only for the Client (pays their own balance). The Pro has View access to the resulting paid/unpaid status. Platform Operator (Support) has View-only access, never to card data.

**Communications:** A payment confirmation on successful balance payment.

**Data Notes:** Captured: balance payment outcome. Displayed: updated balance-due status to both parties. Derived: balance amount, from Service price minus Deposit Transaction amount.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-07); updates Pro Daily Schedule Dashboard (FEAT-12).

**Signals:** balance_payment_attempted, balance_payment_succeeded, balance_payment_failed.

### Tipping at Checkout

**ID:** FEAT-23

**Description:** A client can optionally add a tip when paying in-app (at deposit or, once available, at balance payment), which passes through to the Pro.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions asks directly "where does tipping fit, if anywhere?" As Visionary judgment: tipping has no bearing on the core no-show/deposit problem the product exists to solve, and depends on in-app balance payment (FEAT-22) to be meaningful (tipping on a deposit alone is an unusual pattern). Phased to Later.

**Connected Entities:** Deposit Transaction (read), Booking (read)

**Key Capabilities:**
- Add an optional tip amount at in-app payment time
- See tips reflected in the Pro's own payment records

**Primary Flows & Alternates:**
- Happy path: client is offered an optional tip at balance payment; they choose an amount (or none) and complete payment.
- Alternate: client skips tipping entirely — this must never feel like a required step or block the underlying payment.
- Alternate: client pays their balance in person instead, bypassing in-app tipping entirely; this remains a fully normal path.

**States:** Empty: N/A — only appears within an existing payment flow. Loading: N/A — part of the standard payment flow's own states. Error: N/A — tipping failure is treated as part of the underlying payment's own error handling, never a separate failure mode. Offline-degraded: N/A — inherits the payment flow's connectivity requirement.

**Validation & Limits:** Tip amount, when given, must be a non-negative value; never pre-selected to a default that could feel presumptive.

**Access:** Own-only for the Client (chooses their own tip). The Pro has View access to tips received. Platform Operator (Support) has View-only access, never to card data.

**Communications:** N/A — included in the existing payment confirmation, not a separate message.

**Data Notes:** Captured: optional tip amount. Displayed: tips received, on the Pro's payment records. Derived: none.

**Interactions:** Depends on In-App Balance Payment (FEAT-22).

**Signals:** tip_offered_shown, tip_added, tip_skipped.

### Client List Search & Filter

**ID:** FEAT-24

**Description:** As a Pro's client base grows, they can search and filter their client list by name, phone, or recent activity instead of scrolling a long list.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Scale & Non-Functional Expectations states a Pro may have "100–500 clients," a volume where an unfiltered list becomes genuinely unwieldy. Not required at launch (a new Pro starts with very few clients), so phased to v1 rather than MVP.

**Connected Entities:** Client (read — search/filter only)

**Key Capabilities:**
- Search clients by name or phone number
- Filter by recency (e.g., booked in the last 30 days) or upcoming-booking status

**Primary Flows & Alternates:**
- Happy path: Pro types a partial name into search and the client list narrows instantly.
- Alternate: a search returns no matches; a plain "no clients match" state is shown rather than an empty, unexplained list.

**States:** Empty: N/A — inherits the underlying client list's own empty state (FEAT-13). Loading: instant for the stated client volumes. Error: a failed search falls back to the full, unfiltered list rather than an error screen. Offline-degraded: search operates against the most recently loaded client list.

**Validation & Limits:** N/A — a read-only convenience feature with no input validation beyond a search box.

**Access:** The Pro has Full access to search their own clients. Platform Operator (Support) has View-only access, useful when helping locate a specific client's record during support.

**Communications:** N/A — a browsing convenience with no message trigger.

**Data Notes:** Displayed: filtered/searched subset of existing Client records. Derived: none — this is a view over existing data.

**Interactions:** Depends on Client Record Management (FEAT-13).

**Signals:** client_search_performed, client_filter_applied.

### Booking & Revenue Insights

**ID:** FEAT-25

**Description:** A simple, functional summary for the Pro of their own booking volume, deposits collected, and no-shows recovered over time — proof of the value the product is delivering, not a full analytics suite.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Success Criteria centers on the Pro noticing outcomes ("I haven't had an unpaid no-show since I switched"); a simple summary makes that outcome visible rather than only felt anecdotally, which supports the brief's referral-driven go-to-market ("most new pros arrive because another pro told them about it"). Not required for the core loop, so phased to v1.

**Connected Entities:** Booking (read), Deposit Transaction (read), Service (read)

**Key Capabilities:**
- See total bookings and deposits collected over a selected period
- See how much would have been lost to no-shows without automatic forfeiture (a "saved" figure)
- See which services are booked most often

**Primary Flows & Alternates:**
- Happy path: Pro opens the insights view and sees a simple period summary — bookings, deposits collected, no-shows recovered.
- Alternate: a Pro with very little history sees a plain "not enough data yet" state rather than a chart with a single data point that reads as broken.

**States:** Empty: a new account shows an encouraging "check back after a few weeks of bookings" message. Loading: a brief indicator while aggregating. Error: shows the last successfully computed summary with a retry option. Offline-degraded: the most recently viewed summary remains available read-only.

**Validation & Limits:** N/A — a read-only reporting view with no input validation; summary periods are bounded to sensible ranges (e.g., week/month/year) matching the Pro's own history depth.

**Access:** The Pro has Full (view) access to their own figures only — never compared against or visible to any other pro. Platform Operator (Support) has View-only access. Clients have no access.

**Communications:** N/A — a self-initiated view with no notification trigger.

**Data Notes:** Displayed: aggregated figures. Derived: entirely — every figure here is computed from existing Booking, Deposit Transaction, and Service records; nothing is captured directly.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-07) and No-Show Marking & Deposit Forfeiture (FEAT-11) for the "saved" figure.

**Signals:** insights_viewed, insights_period_changed.

### WhatsApp Reminders

**ID:** FEAT-26

**Description:** Confirmations and reminders can optionally be sent over WhatsApp instead of, or alongside, SMS, for clients and pros who prefer it.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** BRIEF.md's Ecosystem & Integrations states this explicitly: "WhatsApp is a nice-to-have later, not v1." Phased to Later exactly as the brief specifies, and it slots into the existing Automated Booking Messaging (FEAT-08) mechanism as an additional channel rather than a new capability.

**Connected Entities:** Message (create — additional channel), Messaging Consent (read)

**Key Capabilities:**
- Opt for WhatsApp as the delivery channel for confirmations and reminders
- Fall back to SMS or email automatically if WhatsApp delivery is unavailable

**Primary Flows & Alternates:**
- Happy path: client indicates a WhatsApp preference; confirmations and reminders deliver there instead of SMS.
- Alternate: WhatsApp delivery fails or is unavailable for that number; the system falls back to SMS or email per the client's existing consent state, exactly as FEAT-08 already does for SMS failures.

**States:** Empty: N/A — inherits Automated Booking Messaging's own states as an additional channel. Loading: N/A. Error: falls back per the alternate flow above, never silently dropped. Offline-degraded: N/A — server-side sending capability.

**Validation & Limits:** Requires the same explicit consent discipline as texting (FEAT-14) before use — consent is channel-aware, not a blanket "texting is fine" assumption.

**Access:** Clients opt in for their own messages (Own-only). The Pro sees delivery channel/status like any other message (View, via FEAT-16). Platform Operator (Support) has View-only access.

**Communications:** This feature is itself an additional communications channel for the messages FEAT-08 already sends.

**Data Notes:** Captured: channel preference. Displayed: delivery channel/status. Derived: none.

**Interactions:** Extends Automated Booking Messaging (FEAT-08); depends on Messaging Consent Management (FEAT-14).

**Signals:** whatsapp_channel_selected, whatsapp_delivery_failed_fallback_used.

## Feature Interaction Summary

| Feature | Depends On |
|---------|------------|
| FEAT-01 Service & Pricing Management | None |
| FEAT-02 Availability & Working Hours Setup | None |
| FEAT-03 Real-Time Slot Availability Engine | FEAT-02, FEAT-04, FEAT-17, FEAT-21 |
| FEAT-04 Two-Way Calendar Sync | None |
| FEAT-05 Public Booking Page & Booking Flow | FEAT-01, FEAT-03, FEAT-06, FEAT-07, FEAT-09 |
| FEAT-06 Client Booking Identity | FEAT-05 (reads Client created there) |
| FEAT-07 Deposit Payment at Booking | FEAT-01, FEAT-05 |
| FEAT-08 Automated Booking Messaging | FEAT-05, FEAT-07, FEAT-14 |
| FEAT-09 Cancellation & No-Show Policy Engine | None |
| FEAT-10 Client-Initiated Cancel/Reschedule | FEAT-03, FEAT-06, FEAT-09 |
| FEAT-11 No-Show Marking & Deposit Forfeiture | FEAT-07, FEAT-09 |
| FEAT-12 Pro Daily Schedule Dashboard | FEAT-05, FEAT-07, FEAT-13 |
| FEAT-13 Client Record Management | FEAT-05 |
| FEAT-14 Messaging Consent Management | FEAT-05 |
| FEAT-15 Pro Onboarding & Setup Wizard | None |
| FEAT-16 Booking & Payment Activity Record | FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11 |
| FEAT-17 Manual Time Blocking | None |
| FEAT-18 Pro Subscription Billing & Account Management | FEAT-15 |
| FEAT-19 Platform Support Read-Only Access | FEAT-15, FEAT-12, FEAT-16, FEAT-18 |
| FEAT-20 Waitlist for Cancelled Slots | FEAT-03, FEAT-10 |
| FEAT-21 Recurring/Standing Appointments | FEAT-03, FEAT-05 |
| FEAT-22 In-App Balance Payment | FEAT-07 |
| FEAT-23 Tipping at Checkout | FEAT-22 |
| FEAT-24 Client List Search & Filter | FEAT-13 |
| FEAT-25 Booking & Revenue Insights | FEAT-07, FEAT-11 |
| FEAT-26 WhatsApp Reminders | FEAT-08, FEAT-14 |

# Part D — Feature Catalog

This part is the full feature catalog followed by the cross-feature view: how features depend on one another, how users move between them, and the business rules that span them. The Domain Entity Inventory below also anchors Part F, the data-model view.


# Product Features

## Summary

This product defines 30 features: 14 Core, 9 Important, 7 Nice-to-Have. By phase: 23 MVP, 5 v1, 2 Later. By type: 20 User-Facing, 7 Platform, 3 Lifecycle. The product manages 18 domain entities. [MODIFIED: phase and type tallies recounted from the feature entries (the draft summary's tallies did not match its own entries); feature count raised from 26 to 30 and entity count from 14 to 18 after the completeness audit added Pro Profile & Booking Page Settings (FEAT-27), Payout Account Connection & Payout Visibility (FEAT-28), Pro Sign-In & Account Lifecycle (FEAT-29) and Pro Booking Management (FEAT-30), plus the Payout Account, Activity Event, Access Link and Balance Payment entities] Core features close the end-to-end value loop the founder named — a client books and pays a deposit in under a minute, the deposit reaches the pro, and a no-show or cancellation is handled automatically under the pro's own policy; Important features cover the supporting record-keeping, setup, consent, and billing machinery a real business needs; Nice-to-Have features answer the brief's open questions (waitlist, recurring appointments, in-app balance payment, tipping) and later-market polish (search, insights, WhatsApp).

## Domain Entity Inventory

### Entity: Pro Account
- **Description:** The solo professional's account — sign-in identity, public profile (display name, photo, short intro, studio location), booking link, timezone, currency, notification preferences, and the subscription that keeps the account active.
- **Lifecycle:** Created (signup) -> Active -> (optionally) Paused (subscription lapse or pro-chosen booking pause) -> (optionally) Closed (pro deletes the account)
- **Created by:** Pro Onboarding & Setup Wizard (FEAT-15), with sign-in credentials established by Pro Sign-In & Account Lifecycle (FEAT-29)
- **Managed by:** Pro Profile & Booking Page Settings (FEAT-27), Pro Sign-In & Account Lifecycle (FEAT-29), Pro Subscription Billing & Account Management (FEAT-18) [AUDIT-ADDED: 3 -- the entity had no feature for editing its profile fields or closing it; FEAT-27 and FEAT-29 now own those operations]
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
- **Description:** A record of a person who has booked (or attempted to book) with a specific pro — name, phone, optional email, the pro's private notes, and booking history with that pro only.
- **Lifecycle:** Created (first booking) -> Active -> Deleted (on request)
- **Created by:** Public Booking Page & Booking Flow (FEAT-05); Pro Booking Management (FEAT-30) when the Pro books a client in on their behalf
- **Managed by:** Client Record Management (FEAT-13) [AUDIT-ADDED: 3 -- a second creation path (pro-entered booking) and the optional email captured for the email fallback now map to this entity]
- **Referenced by:** Client Booking Identity (FEAT-06), Pro Daily Schedule Dashboard (FEAT-12), Client List Search & Filter (FEAT-24)

### Entity: Booking
- **Description:** A confirmed (or attempted) appointment: service, time, client, deposit status, and current lifecycle state.
- **Lifecycle:** Pending Payment -> Confirmed -> Awaiting Outcome (appointment time passed) -> (Completed | No-Show); or Confirmed -> (Cancelled by Client | Cancelled by Pro | Rescheduled)
- **Created by:** Public Booking Page & Booking Flow (FEAT-05), Pro Booking Management (FEAT-30), Recurring/Standing Appointments (FEAT-21)
- **Managed by:** Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Daily Schedule Dashboard (FEAT-12), Pro Booking Management (FEAT-30) [AUDIT-ADDED: 3 -- the Completed state had no owning transition and pro-side cancel/reschedule had no owning feature; FEAT-12 now records completion and FEAT-30 owns pro-initiated changes]
- **Referenced by:** Real-Time Slot Availability Engine (FEAT-03), Automated Booking Messaging (FEAT-08), Cancellation & No-Show Policy Engine (FEAT-09), Booking & Payment Activity Record (FEAT-16), Booking & Revenue Insights (FEAT-25), Recurring/Standing Appointments (FEAT-21), Waitlist for Cancelled Slots (FEAT-20)

### Entity: Deposit Transaction
- **Description:** The record of a client's card deposit for one booking — amount, status (captured/applied/refunded/forfeited/disputed), and its outcome. Card details themselves are never held; only the amount and outcome.
- **Lifecycle:** Authorized -> Captured -> (Applied to service on completion | Refunded | Forfeited); any captured deposit may additionally enter Disputed if the client raises a dispute with their card issuer
- **Created by:** Deposit Payment at Booking (FEAT-07)
- **Managed by:** Cancellation & No-Show Policy Engine (FEAT-09), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Booking Management (FEAT-30)
- **Referenced by:** Booking & Payment Activity Record (FEAT-16), Booking & Revenue Insights (FEAT-25), In-App Balance Payment (FEAT-22), Payout Account Connection & Payout Visibility (FEAT-28) [AUDIT-ADDED: 1 -- value-flow walk: the deposit's end states now cover completion (applied toward the service price), pro-issued refunds, and card-issuer disputes]

### Entity: Cancellation Policy
- **Description:** The pro's own rule set: the cancellation/reschedule window and what happens to the deposit inside vs. outside it.
- **Lifecycle:** Created (during onboarding) -> Active -> Edited (versioned; a booking is always governed by the policy in force when it was made)
- **Created by:** Pro Onboarding & Setup Wizard (FEAT-15)
- **Managed by:** Cancellation & No-Show Policy Engine (FEAT-09)
- **Referenced by:** Public Booking Page & Booking Flow (FEAT-05), Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11)

### Entity: Messaging Consent
- **Description:** A client's explicit opt-in (or opt-out) to receive text messages from a specific pro, per channel, with its timestamp and the exact consent wording shown.
- **Lifecycle:** Granted -> Active -> Revoked -> (optionally) Re-granted
- **Created by:** Public Booking Page & Booking Flow (FEAT-05) (captured at booking)
- **Managed by:** Messaging Consent Management (FEAT-14)
- **Referenced by:** Automated Booking Messaging (FEAT-08)

### Entity: Message
- **Description:** A record of a single notification sent (confirmation, reminder, cancellation notice) — channel, content summary, delivery status.
- **Lifecycle:** Queued -> Sent -> (Delivered | Failed)
- **Created by:** Automated Booking Messaging (FEAT-08)
- **Managed by:** N/A -- messages are immutable once sent
- **Referenced by:** Booking & Payment Activity Record (FEAT-16), Pro Daily Schedule Dashboard (FEAT-12) (delivery-failure flags)

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

### Entity: Payout Account
- **Description:** The Pro's connected account with the payment-processing capability, into which client deposits (and, from v1, balances and tips) are paid out — status, payout schedule, and recent payouts. Bank and identity details are held by the payment processor, never by the product.
- **Lifecycle:** Not Connected -> Verification Pending -> Active -> (Action Required | Disconnected)
- **Created by:** Payout Account Connection & Payout Visibility (FEAT-28)
- **Managed by:** Payout Account Connection & Payout Visibility (FEAT-28)
- **Referenced by:** Pro Onboarding & Setup Wizard (FEAT-15), Deposit Payment at Booking (FEAT-07), Pro Booking Management (FEAT-30), Platform Support Read-Only Access (FEAT-19) [AUDIT-ADDED: 1 -- value-flow walk found no entity or feature for where a deposit lands after the client pays]

### Entity: Activity Event
- **Description:** One immutable entry in a booking's or an account's timeline — what happened, when, and who or what caused it (client, Pro, the product automatically, or a support view).
- **Lifecycle:** N/A -- append-only record; entries are written once and never edited or deleted while the booking exists
- **Created by:** Booking & Payment Activity Record (FEAT-16), written automatically as other features act
- **Managed by:** N/A -- immutable by design
- **Referenced by:** No-Show Marking & Deposit Forfeiture (FEAT-11), Platform Support Read-Only Access (FEAT-19) [AUDIT-ADDED: 3 -- inverse check: FEAT-16 captures timeline entries that no inventory entity held]

### Entity: Access Link
- **Description:** A short-lived, single-purpose link that lets a client open their own bookings with one pro without a password — either a general "my bookings" link requested on demand or a booking-specific manage link carried in confirmations and reminders.
- **Lifecycle:** Issued -> (Used | Expired)
- **Created by:** Client Booking Identity (FEAT-06); booking-specific links are issued with messages sent by Automated Booking Messaging (FEAT-08)
- **Managed by:** Client Booking Identity (FEAT-06)
- **Referenced by:** Client-Initiated Cancel/Reschedule (FEAT-10) [AUDIT-ADDED: 3 -- inverse check: FEAT-06 issues and expires links that no inventory entity held]

### Entity: Balance Payment
- **Description:** A client's optional in-app payment of the remaining balance for one booking (v1), including any optional tip (Later).
- **Lifecycle:** Attempted -> (Succeeded | Failed) -> (optionally) Refunded
- **Created by:** In-App Balance Payment (FEAT-22)
- **Managed by:** In-App Balance Payment (FEAT-22), Pro Booking Management (FEAT-30) (refund if the appointment is cancelled after the balance was paid)
- **Referenced by:** Tipping at Checkout (FEAT-23), Pro Daily Schedule Dashboard (FEAT-12), Payout Account Connection & Payout Visibility (FEAT-28) [AUDIT-ADDED: 3 -- inverse check: FEAT-22 and FEAT-23 capture payment outcomes and tip amounts that no inventory entity held]

## Core Features

### Service & Pricing Management

**ID:** FEAT-01

**Description:** The Pro defines the services they offer — name, price, duration, and the deposit rule for that service (fixed amount or percentage) — and controls which services are currently bookable.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** Directly required by BRIEF.md's Vision: a client "picks a service" with "prices and how long each takes" stated in plain words before booking. Without this, the booking page has nothing to show. MVP: the product cannot function without at least one bookable service. [INFERRED: carried from Visionary draft]

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

**Validation & Limits:** Service name required (1–80 characters); price must be a positive amount in the Pro's account currency; duration required and must be a positive number of minutes (in 5-minute steps, up to 12 hours); deposit rule must be either a fixed amount no greater than the service price or a percentage between 1–100%, and the resulting deposit must be at least the smallest amount a card payment can be taken for. [MODIFIED: fixed-amount ceiling aligned with the 100% percentage ceiling so both deposit forms allow full prepayment consistently; minimum chargeable amount added because a deposit below it could never be collected -- synthesis consistency fix]

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

**Rationale:** BRIEF.md's Target Users & Roles states the Pro sets "working hours, buffer time between clients" as a core setup action. MVP: real-time availability has nothing to compute from without it. [INFERRED: carried from Visionary draft]

**Connected Entities:** Availability Rule (create, update)

**Key Capabilities:**
- Set weekly working hours (per day of week, with multiple windows per day allowed)
- Set default buffer time applied between consecutive bookings
- Override buffer time per service where a service genuinely needs more or less
- Set a minimum booking notice (how close to an appointment a client may still book) and a booking horizon (how far ahead clients may book) [AUDIT-ADDED: 1 -- journey walk of Client's First Booking found nothing stopping a client from booking a slot minutes away or a year out; both limits are needed for the "genuinely free time" promise to be workable for the Pro]

**Primary Flows & Alternates:**
- Happy path: Pro sets hours for each working day and a default buffer; the change is reflected in bookable slots going forward immediately.
- Alternate: Pro closes a normally-working day for a one-off reason using Manual Time Blocking (FEAT-17) rather than editing the recurring rule.
- Alternate: Pro changes hours mid-week; already-confirmed bookings outside the new hours are never silently cancelled — they remain honored and flagged for the Pro's attention.

**States:** Empty: a brand-new account has no hours set and cannot be booked until at least one working window exists; the setup wizard (FEAT-15) makes this the first required step. Loading: N/A — instant, small dataset. Error: a save failure preserves entered values with a retry option. Offline-degraded: N/A — setup screen, not an in-the-moment mobile flow.

**Validation & Limits:** Each working window requires a start time before its end time; buffer time must be zero or a positive number of minutes (up to 2 hours); overlapping windows on the same day are rejected with a clear message; minimum booking notice defaults to a few hours and may be set from zero to 7 days; booking horizon defaults to 8 weeks and may be set from 1 week to 12 months; all hours are interpreted in the Pro's account timezone. [AUDIT-ADDED: 1 -- boundary values for the new notice and horizon settings, a buffer ceiling, and timezone interpretation]

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

**Rationale:** BRIEF.md's Vision and Success Criteria are explicit: "a genuinely free time," and "nobody has ever had a double booking." This is the mechanism that makes that promise true; every other feature that touches time depends on it. [RESEARCH-INFORMED: added competitor weakness context -- glitches and crashes concentrated in the scheduling/calendar workflow are reported for two competitors serving the same dozens-of-bookings-a-week volume, so correctness of this engine is the product's clearest differentiator, from Capterra/GetApp/SoftwareAdvice-aggregated user reviews (MEDIUM confidence)]

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

**Validation & Limits:** A slot is only offered if the full service duration plus buffer fits entirely within an open working window with no conflicting booking, block, or external calendar event; slot holds during checkout expire after a short, fixed timeout (a few minutes) if payment is not completed; slots inside the Pro's minimum booking notice or beyond their booking horizon (FEAT-02) are never offered; slot times are always computed and shown in the Pro's timezone, labeled as such. [AUDIT-ADDED: 1 -- journey walk: notice/horizon limits and timezone labeling were unspecified for a client who may be browsing from another timezone]

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

**Rationale:** BRIEF.md's Ecosystem & Integrations states this is two-way and "both matter" for Google and Apple. A pro who lives partly off-platform (personal appointments, a second job) cannot trust the availability engine without it, directly serving the "never silently double-book" success criterion. [RESEARCH-INFORMED: added competitor weakness context -- mobile app lag in syncing calendar changes is a frequently mentioned complaint about one competitor, from aggregator-summarized user reviews (MEDIUM confidence)]

**Connected Entities:** Calendar Connection (create, update, delete)

**Key Capabilities:**
- Connect a Google or Apple calendar
- See connection health (connected / needs reconnection)
- Disconnect a calendar at any time

**Primary Flows & Alternates:**
- Happy path: Pro connects their calendar during onboarding; from that point, external busy time blocks Chairtime slots and new Chairtime bookings appear on the personal calendar within moments of confirmation.
- Alternate: the connection lapses (revoked or expired calendar permission); the Pro sees a clear "reconnect your calendar" prompt on their dashboard, and the availability engine visibly narrows its confidence rather than silently trusting stale data.
- Alternate: Pro disconnects intentionally; existing Chairtime bookings remain intact, but external busy time no longer factors into future availability until reconnected.
- Alternate: a Chairtime booking is cancelled or rescheduled (by either party); the matching entry on the Pro's personal calendar is removed or moved to the new time automatically, so the personal calendar never shows a ghost appointment. [AUDIT-ADDED: 1 -- journey walk of Riley Manages an Existing Booking promised "the pro's calendar reflects the change" but the feature only wrote new bookings out]

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

**Rationale:** This is the literal product described in BRIEF.md's Vision: "a client opens the pro's link... picks a service and a genuinely free time... pays a card deposit." It is the entire reason the product exists and the founder's stated one-minute benchmark. [RESEARCH-INFORMED: added the policy-disclosure gap -- clients disputing deposit and cancellation charges they did not understand at booking is a recurring complaint, and the disclosure moment itself is where trust is lost, from BBB complaint records and forum-derived complaint summaries (2 source types, MEDIUM confidence)]

**Connected Entities:** Booking (create), Client (create), Service (read), Cancellation Policy (read), Messaging Consent (create — captured here), Pro Account (read — public profile) [MODIFIED: Pro Account added for the public profile shown on the page]

**Key Capabilities:**
- View a Pro's services, prices, durations, and deposit rule in plain language
- Pick a service and a genuinely free time slot
- Enter name and phone, and opt in to text messages
- Explicitly acknowledge the deposit and cancellation policy, shown in plain words with the exact amounts and cut-off time for this booking, before paying [AUDIT-ADDED: 2 -- competitive cross-reference: plain-language policy disclosure is an "absent feature" no competitor solves; an explicit acknowledgment step turns the brief's "policy the client agreed to when booking" into a recorded fact]
- Provide an email address when declining texts, so confirmations and reminders can still arrive by the brief's email fallback; optionally add a short note for the Pro (e.g., "first full set") [AUDIT-ADDED: 3 -- inverse check: the email fallback in FEAT-08 and FEAT-14 had no captured email address to send to]
- Complete deposit payment and receive an immediate on-screen confirmation

**Primary Flows & Alternates:**
- Happy path: client taps the bio link -> sees services -> picks one -> picks a free time -> enters name and phone -> opts in to texts -> pays the deposit by card -> sees a confirmation on screen, in under a minute end to end.
- Alternate: returning client — a client who has booked with this Pro before is recognized by phone number and can skip re-entering their name (BRIEF.md's identity mechanism, FEAT-06).
- Alternate: payment fails or is declined — the client sees a clear, specific reason and can retry with the same or a different card without losing their selected slot (within the slot hold timeout).
- Alternate: the Pro's account is paused (subscription lapsed or the Pro paused bookings in FEAT-27); the page still shows the Pro's name and services but replaces time selection with a plain "not taking new bookings right now" message, and never takes a deposit.

**States:** Empty: N/A — the booking page always shows the Pro's current service list; if a Pro has zero active services, the page shows a plain "temporarily not accepting bookings" message rather than a broken page. Loading: a lightweight indicator while slots load; the page never appears interactive before real availability has loaded. Error: a failed step (slot no longer available, payment failure) keeps all previously entered information intact so the client never has to start over. Offline-degraded: booking requires a live connection to guarantee correctness (per the Real-Time Slot Availability Engine's own offline stance); a client who loses connection mid-flow sees a plain "check your connection and try again" message with nothing charged.

**Validation & Limits:** Name required (1–100 characters); phone number required and must be a valid, reachable format; the client must actively check the texting opt-in — it is never pre-checked; an email address is required when the client does not opt in to texts and optional otherwise; the policy acknowledgment must be actively checked before the payment step unlocks; the optional note is limited to 300 characters, with a plain hint not to include medical information (health intake is out of scope per BRIEF.md); a slot hold expires after a short fixed window if payment is not completed; the page reads comfortably and is fully operable at phone width inside an in-app social-media browser.

**Access:** Open to any Client, with no login required to reach the flow itself (BRIEF.md: "must not face a signup wall"); a Client only ever sees one Pro's public page and never any other client's booking. The Pro has Full access to the resulting bookings on their own dashboard. Platform Operator (Support) has View-only access. A visitor following a mistyped or closed-account link sees a plain "this booking page isn't available" message, never another pro's page. [MODIFIED: Pro access stated as Full to agree with the Access Matrix "Booking & Payment" column; unauthorized-visitor experience added -- synthesis Access Matrix audit]

**Communications:** Triggers the immediate booking confirmation handled by Automated Booking Messaging (FEAT-08).

**Data Notes:** Captured: chosen service and time, client name and phone, optional email, optional note to the Pro, texting opt-in, policy acknowledgment (with the exact policy version and wording shown), deposit payment outcome. Displayed: the Pro's public profile (FEAT-27), service list, prices, durations, deposit rule, cancellation policy, available slots. Derived: none directly; the resulting Booking record is the source others read from. [AUDIT-ADDED: 3 -- inverse check: email, note and policy acknowledgment now captured and mapped to the Client and Booking entities]

**Interactions:** Depends on Service & Pricing Management (FEAT-01), Real-Time Slot Availability Engine (FEAT-03), Client Booking Identity (FEAT-06), Deposit Payment at Booking (FEAT-07), Cancellation & No-Show Policy Engine (FEAT-09); feeds Automated Booking Messaging (FEAT-08) and Pro Daily Schedule Dashboard (FEAT-12).

**Signals:** booking_page_viewed, service_selected, slot_selected, policy_acknowledged, booking_completed, booking_abandoned (with last completed step).

### Client Booking Identity

**ID:** FEAT-06

**Description:** The lightweight, password-free way a client proves it's them when they come back to view, reschedule, or cancel a booking — a phone number plus a one-tap link sent to that phone, rather than any signup wall or password.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Open Questions names this exactly ("A phone number plus a magic link, or something else?") and its Target Users & Roles states clients "must not face a signup wall or need a password-style account." As the Visionary's product judgment on this open question: phone-plus-link is the lightest workable mechanism that still lets a client manage a specific booking without exposing any other client's or pro's data. MVP: without it, a client has no way to self-serve a reschedule or cancellation, which the brief requires (FEAT-10). [INFERRED: carried from Visionary draft]

**Connected Entities:** Client (read, update — matches by phone), Booking (read — scoped to that client and pro)

**Key Capabilities:**
- Request a one-tap access link sent by text to the phone number used at booking
- View only this pro's bookings tied to that phone number, past and upcoming
- Access expires and must be re-requested after a short period for security
- Open a specific booking directly from the manage link inside its confirmation or reminder, without requesting a new link [AUDIT-ADDED: 1 -- journey walk of Riley Manages an Existing Booking: the reminder's "I need to reschedule" tap must land straight in the booking, which needs a booking-specific link distinct from the on-demand "my bookings" link]
- Update their own texting consent and email address for this Pro [AUDIT-ADDED: 3 -- entity coverage: Messaging Consent had no client-facing place to re-grant consent after opting out]

**Primary Flows & Alternates:**
- Happy path: client taps "manage my booking" from a reminder text or the booking page, requests a link, taps it, and sees only their own bookings with this one pro.
- Alternate: client requests a link from a phone number with no bookings for this pro; they see a plain "no bookings found" message rather than an error, with no hint about whether the number exists elsewhere.
- Alternate: the access link expires before use; requesting a new one is a single tap, with no separate "reset" flow to learn.

**States:** Empty: a phone number with no bookings shows a plain, non-alarming message. Loading: brief indicator while the link is generated and sent. Error: failed link delivery offers an immediate retry. Offline-degraded: N/A — this is an online-only identity check by design (correctness over convenience).

**Validation & Limits:** On-demand access links are single-use and expire after 30 minutes; booking-specific manage links in confirmations and reminders open only that one booking and stop working once the appointment has passed; a client can never view another phone number's bookings even if they guess or mistype one; link requests are limited to a handful per phone number per hour to prevent message flooding. [MODIFIED: expiry narrowed from "minutes to a couple of hours" to a testable 30 minutes and a request limit added -- synthesis output-completeness fix]

**Access:** Open to any Client using their own phone number; a client can never see another client's bookings under any circumstance — this is a hard privacy boundary from BRIEF.md's Constraints. The Pro does not use this mechanism (the Pro has their own dashboard login, out of this feature's scope). Platform Operator (Support): None — support access does not use or bypass client identity.

**Communications:** Sends the one-tap access link by text when the client has active texting consent for this Pro, otherwise by email to the address on file, each time one is requested. [MODIFIED: channel choice tied to the client's texting consent rather than a general fallback, consistent with Messaging Consent Management]

**Data Notes:** Captured: issued Access Links and their used/expired state (Access Link entity); consent and email updates. Displayed: the client's own booking list. Derived: none. Source: matched against the phone number on existing Client records for this pro only. [AUDIT-ADDED: 3 -- inverse check: issued links now map to the Access Link entity]

**Interactions:** Depended on by Client-Initiated Cancel/Reschedule (FEAT-10); reads Client (created by FEAT-05) and Booking.

**Signals:** access_link_requested, access_link_used, access_link_expired_unused.

### Deposit Payment at Booking

**ID:** FEAT-07

**Description:** The client pays a card deposit — a fixed amount or a percentage of the service price, per the Pro's own rule — at the moment of booking, with the balance left due in person at the appointment.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision states the client "pays a card deposit," and the Business Context is explicit that the platform never stores or handles card data itself and takes no per-booking cut. This is the mechanism that solves the founder's core problem: deposits that used to be asked for by hand and often never arrived. [RESEARCH-INFORMED: added market validation -- deposit and card-on-file no-show protection is consistently credited with materially reducing no-shows across StyleSeat, Booksy and Fresha, from cross-referenced help-center data, feature documentation and Reddit-derived review summaries (3 sources, HIGH confidence)]

**Connected Entities:** Deposit Transaction (create), Booking (update — marks as paid/confirmed), Service (read — for the deposit rule)

**Key Capabilities:**
- Pay the exact deposit amount required by the selected service's rule, by card
- See a clear on-screen and confirmed record that the deposit succeeded
- Have a failed or declined payment explained clearly, with the slot held briefly to retry
- Deposit lands directly in the Pro's own payout account (FEAT-28), with the platform taking no cut; the only deduction is the payment processor's own card fee, shown to the Pro [AUDIT-ADDED: 1 -- value-flow walk: the draft named where the deposit enters but not where it goes or who holds it]

**Primary Flows & Alternates:**
- Happy path: client enters card details at the payment step; the deposit is authorized and captured; the booking flips from pending to confirmed instantly.
- Alternate: the card is declined; the client sees the decline reason from the processor in plain language and can retry with another card without losing the held slot (within its short hold window).
- Alternate: payment succeeds but the confirmation step fails to load; the booking is still correctly recorded as confirmed by the product and the client is shown the confirmation on next page load or via the confirmation text, never double-charged and never left unsure whether they are booked. [MODIFIED: wording made implementation-neutral per the functional-language rule; behavior unchanged]

**States:** Empty: N/A — payment is always tied to an in-progress booking, never a standalone screen. Loading: a clear "processing payment, do not close this page" state during authorization. Error: a specific, actionable message for each decline reason available from the processor. Offline-degraded: payment requires connectivity by nature; a connection drop mid-payment is treated as a failure with a safe retry, never an ambiguous charge.

**Validation & Limits:** Deposit amount is computed exactly from the service's rule (fixed amount, or percentage rounded to the nearest currency unit) in the Pro's account currency and cannot be altered by the client; one deposit charge per booking; a deposit cannot be taken for a Pro whose payout account (FEAT-28) is not active. [AUDIT-ADDED: 1 -- value-flow walk: currency and active payout account made preconditions]

**Access:** The Client pays for their own booking only. The Pro has Full access to the resulting deposit status on their own bookings but never sees the card number itself — the processor owns all card data, per BRIEF.md's Constraints. Platform Operator (Support) has View-only access to transaction status, never to card data.

**Communications:** Feeds the confirmation message in Automated Booking Messaging (FEAT-08); a payment failure shows in-flow only and sends no separate message.

**Data Notes:** Captured: deposit amount and payment outcome. Displayed: paid/unpaid status on bookings, on both the client's confirmation and the Pro's dashboard. Derived: none — amount is computed once from Service at the moment of booking and then fixed. Source: the payment-processing capability the product depends on (BRIEF.md, Business Context) for the actual card handling; Chairtime holds only the outcome and amount.

**Interactions:** Depends on Service & Pricing Management (FEAT-01), Public Booking Page & Booking Flow (FEAT-05), and Payout Account Connection & Payout Visibility (FEAT-28); feeds Cancellation & No-Show Policy Engine (FEAT-09), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Booking Management (FEAT-30), and Booking & Payment Activity Record (FEAT-16). [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** deposit_payment_attempted, deposit_payment_succeeded, deposit_payment_failed (with decline reason category).

### Payout Account Connection & Payout Visibility

**ID:** FEAT-28

**Description:** The Pro connects their own payout account with the payment-processing capability so every client deposit lands directly with them, and can see at a glance what came in, what was refunded, what the card processor charged, and when money reaches their bank — with nothing taken by Chairtime.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Business Context states the client's deposit "goes to the pro (the payment processor handles payout to the pro)" and that "the platform takes no cut of any of it"; its Success Criteria demand that nobody "has ever had... a lost deposit." The draft specified where a deposit enters (FEAT-07) but not where it lands, so the value flow was open. Core because it is load-bearing: no deposit can be taken for a Pro without an active payout account, so the product's headline loop cannot close without it. MVP for the same reason. Visibility is part of the feature, not an extra: fee unpredictability is the dominant trust complaint in this market (3 competitors, HIGH confidence) and payouts held without explanation are a frequently mentioned complaint about one competitor (MEDIUM confidence), so the Pro must be able to see every deposit, refund and processor fee plainly. [AUDIT-ADDED: 1 -- Core: value-flow walk found no destination for the deposit money; without a connected payout account the deposit, refund and no-show loop that BRIEF.md's Vision and Success Criteria rest on cannot operate. Supported by market research: integrated card processing with payout to the professional is present in all 5 profiled competitors (Common Features), and fee-transparency complaints are HIGH confidence]

**Connected Entities:** Payout Account (create, read, update), Deposit Transaction (read), Balance Payment (read — from v1)

**Key Capabilities:**
- Connect a payout account during onboarding through the payment processor's own secure identity and bank verification -- the Pro never types bank details into Chairtime itself
- See payout account status -- verification pending, active, or action required
- See a simple money list -- deposits received, refunds sent, the processor's card fees, and upcoming and past payouts to the bank
- Resolve a flagged verification or bank-detail problem -- a direct path back into the processor's own flow

**Primary Flows & Alternates:**
- Happy path: during setup the Pro connects a payout account and completes the processor's verification; once it is active the booking link can go live, each deposit appears in the money list the moment it is paid, and money reaches the Pro's bank on the processor's schedule.
- Alternate: verification is still pending at the end of setup; the Pro can finish every other step, but the booking link stays unpublished with a plain "finish verifying your payout account to start taking bookings" prompt, because a deposit could not be collected yet.
- Alternate: the processor later flags the account as needing action (for example, rejected bank details); existing and new bookings continue, the processor holds payouts until it is fixed, and the Pro sees a prominent banner and a notification explaining exactly what to do.
- Alternate: a refund is due but the Pro's processor balance cannot cover it yet; the refund is retried automatically and flagged to the Pro (FEAT-09), and it appears in the money list once complete.

**States:** Empty: before the first booking, the money list shows "your deposits will appear here after your first booking" rather than a zero-filled table. Loading: a brief in-place indicator while the money list loads. Error: if payout information cannot be retrieved, the last loaded figures are shown with the time they were loaded and a retry action. Offline-degraded: the most recently loaded money list stays viewable read-only.

**Validation & Limits:** One payout account per Pro Account; the payout account must be in the same country and currency as the Pro Account (US at MVP, with other countries following the geography phase-in); Chairtime's own fee on any deposit, balance or tip is always zero; the only deduction ever shown is the processor's own card fee.

**Access:** The Pro has Full access to their own payout account and money list only. Clients have no access and never see the Pro's payout details. Platform Operator (Support) has View-only access to status and the money list, never to bank or identity details, which the payment processor holds. Anyone not signed in as the Pro is sent to the Pro sign-in screen (FEAT-29).

**Communications:** A notification to the Pro when verification completes (the link can go live), when the account needs action, and when a refund cannot be completed yet.

**Data Notes:** Captured: a reference to the Pro's processor payout account and its status (bank and identity details stay with the processor). Displayed: account status, deposits, refunds, processor fees, payouts. Derived: net amount received per period. Source: the payment-processing capability's records plus Deposit Transaction (and, from v1, Balance Payment) records.

**Interactions:** Required by Deposit Payment at Booking (FEAT-07) and Pro Onboarding & Setup Wizard (FEAT-15); refunds from Cancellation & No-Show Policy Engine (FEAT-09) and Pro Booking Management (FEAT-30) draw on it; feeds Booking & Revenue Insights (FEAT-25); viewed by Platform Support Read-Only Access (FEAT-19).

**Signals:** payout_account_connect_started, payout_account_activated, payout_account_action_required, money_list_viewed.

### Automated Booking Messaging

**ID:** FEAT-08

**Description:** The client receives an immediate confirmation the moment a booking is paid, and an automatic reminder before the appointment with a one-tap "I'll be there / I need to reschedule" response — replacing the Pro's habit of texting reminders by hand.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision states this exactly: "a confirmation text lands immediately... a reminder arrives with a one-tap 'I'll be there / I need to reschedule.'" This directly replaces the founder's stated evening admin burden and is central to the "pro never chases" success criterion. [RESEARCH-INFORMED: automated client reminders are present in all 5 profiled competitors, so they are a baseline expectation rather than a differentiator, from the market-research Feature Comparison Matrix (5 profiles)]

**Connected Entities:** Message (create), Booking (read), Messaging Consent (read), Pro Account (read — studio location and notification preferences), Access Link (create — booking-specific manage links) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- Send an immediate confirmation message on successful booking
- Send an automatic reminder a set time before the appointment (two days, per BRIEF.md's example)
- Offer a one-tap "I'll be there" or "I need to reschedule" response from the reminder itself
- Include in every confirmation the service, date and time (with timezone), deposit paid, balance due in person, the studio location, the cancellation cut-off, a manage link, and an "add to my calendar" option [AUDIT-ADDED: 1 -- journey walk of Client's First Booking: the client had no way to learn where to go or when the cancellation window closes after leaving the page]
- Tell the client when their booking is cancelled, rescheduled or refunded by either party, including what happened to the deposit [AUDIT-ADDED: 1 -- counterpart symmetry: a Pro-initiated cancellation (FEAT-30) otherwise left the client uninformed]
- Notify the Pro of new bookings, client cancellations and reschedules, and anything needing attention (message delivery failure, calendar reconnection, refund failure, card-issuer dispute), in-app and — per the Pro's preferences in FEAT-27 — by text or email [AUDIT-ADDED: 4 -- cross-cutting Notifications concern: the Pro had no defined way to learn of changes except by opening the dashboard]

**Primary Flows & Alternates:**
- Happy path: booking completes -> confirmation text sent within moments; two days before the appointment -> reminder sent with the one-tap options; tapping "I'll be there" simply acknowledges, tapping "I need to reschedule" routes into Client-Initiated Cancel/Reschedule (FEAT-10).
- Alternate: the client never opted in to texting; the confirmation and reminder are sent by email instead, per BRIEF.md's fallback ("Email confirmations are acceptable as a fallback").
- Alternate: a text fails to deliver; the system retries once and, on continued failure, falls back to email and flags the delivery gap on the Pro's dashboard so the Pro is never blindsided by a client who "never got a reminder."

**States:** Empty: N/A — messages only ever exist tied to a booking. Loading: N/A — sending happens in the background with no user-facing wait. Error: a delivery failure is retried and then falls back to email; it is never silently dropped. Offline-degraded: N/A — sending is done by the product itself, not on either person's device, so neither party's connectivity affects it. [MODIFIED: wording made implementation-neutral per the functional-language rule; behavior unchanged]

**Validation & Limits:** No text is sent to a phone number without active Messaging Consent (FEAT-14); reminder timing defaults to two days before the appointment, matching BRIEF.md's stated example; a booking made after its reminder point gets no separate reminder — the confirmation serves instead; reminders are sent only during daytime hours (roughly 8am–9pm in the Pro's timezone), moving to the nearest allowed time otherwise; the one-tap replies are link taps, so a reply works the same by text or email. [AUDIT-ADDED: 4 -- Compliance concern: US texting rules expect sends at reasonable hours, and late bookings needed a defined reminder behavior]

**Access:** Only the Client tied to that Booking receives its messages; the Pro receives their own notifications and sees message history for their own bookings (via FEAT-16) but does not receive the client's replies as raw texts — only the resulting reschedule/confirmation status. Platform Operator (Support) has View-only access to delivery status for troubleshooting. A person who forwards a message on cannot act on another client's booking beyond that single booking's manage link, which stops working once the appointment has passed. [AUDIT-ADDED: 1 -- the unauthorized-recipient case for forwarded messages was unstated]

**Communications:** This feature *is* the communications: client booking confirmation, pre-appointment reminder with one-tap reply handling, client change and refund notices, and Pro notifications. [MODIFIED: expanded to list the audit-added change notices and Pro notifications]

**Data Notes:** Captured: message content summary and delivery status per send. Displayed: delivery status to the Pro (via FEAT-16). Derived: reminder timing is derived from the Booking's appointment time. Source: Booking and Messaging Consent records.

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), Messaging Consent Management (FEAT-14), and Pro Profile & Booking Page Settings (FEAT-27) (studio location, Pro notification preferences); triggered by Client-Initiated Cancel/Reschedule (FEAT-10) and Pro Booking Management (FEAT-30); feeds Client-Initiated Cancel/Reschedule (FEAT-10) (via the reschedule tap), Pro Daily Schedule Dashboard (FEAT-12) ("I'll be there" status), and Booking & Payment Activity Record (FEAT-16).

**Signals:** confirmation_sent, reminder_sent, reminder_reply_confirmed, reminder_reply_reschedule_requested, change_notice_sent, pro_notification_sent (with type), message_delivery_failed.

### Cancellation & No-Show Policy Engine

**ID:** FEAT-09

**Description:** The Pro defines their own cancellation window and what happens to the deposit inside vs. outside it; the system enforces that policy automatically and consistently on every cancellation and no-show, exactly as the client agreed to it at booking.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Business Context is exact: "a cancellation inside the pro's window forfeits the deposit... a cancellation outside the window refunds the deposit automatically," "under the pro's own cancellation policy that the client agreed to when booking." This is the mechanism behind the founder's headline promise: "the pro never chases a no-show again." [RESEARCH-INFORMED: added the policy-clarity finding -- clients dispute deposit and cancellation-fee charges when terms were unclear at booking, so the policy must be shown with this booking's exact cut-off time and amount rather than as a generic rule, from BBB complaint records and forum-derived complaint summaries (MEDIUM confidence)]

**Connected Entities:** Cancellation Policy (create, update), Deposit Transaction (update — refund or forfeit), Booking (read)

**Key Capabilities:**
- Set the cancellation/reschedule window (e.g., 24 hours before appointment)
- Automatically refund the deposit for a cancellation made outside the window
- Automatically flag a cancellation inside the window (or a no-show) for deposit forfeiture, applied via FEAT-11
- Apply the counterpart rules: a cancellation made by the Pro always refunds the client's deposit in full, whatever the timing; a reschedule outside the window carries the deposit over to the new time; a reschedule inside the window is treated as a late cancellation (deposit kept) plus a new booking with its own deposit, and the client is told so before confirming [AUDIT-ADDED: 1 -- counterpart symmetry: the draft defined only client-caused outcomes; a Pro-caused cancellation and a late reschedule each needed a stated deposit outcome]

**Primary Flows & Alternates:**
- Happy path: client cancels well outside the window -> deposit refunds automatically, no Pro action needed.
- Alternate: client cancels inside the window -> the client is shown, before confirming, exactly what will happen to their deposit under this pro's policy, so there is no surprise or dispute later.
- Alternate: Pro changes their policy going forward; every already-confirmed booking is still governed by the policy version shown to the client at the time they booked, never retroactively changed underneath them.

**States:** Empty: N/A — a default, sensible cancellation window is proposed during onboarding (FEAT-15) so no Pro Account exists without an active policy. Loading: N/A — policy evaluation is instantaneous at the moment of cancellation. Error: if automatic refund processing fails (for example, the Pro's payout balance cannot cover it yet), the refund is retried automatically, the outcome is flagged clearly on the Pro's dashboard, and the client is told the refund is in progress rather than silently failing. Offline-degraded: N/A — enforcement is carried out by the product itself regardless of either party's connectivity at the time. [MODIFIED: refund-failure path detailed from the value-flow walk and wording made implementation-neutral]

**Validation & Limits:** The cancellation window must be a whole number of hours from 1 to 168 (7 days) before the appointment; the policy is binary in v1 — full refund outside the window, deposit kept inside it or on a no-show; the policy in force is always the one shown to the client at the moment they booked (versioned, never edited retroactively for existing bookings); the client is told a refund returns to their card on their card issuer's usual timeline. [MODIFIED: window bounded at 1–168 hours and the binary rule and refund timing stated, per BRIEF.md's Business Context and the output-completeness rule]

**Access:** The Pro has Full access to set the policy. The Client sees the current policy in plain words during booking and at cancellation time (Own-only, read). Platform Operator (Support) has View-only access.

**Communications:** N/A — the policy's application is communicated in-flow (at booking and at cancellation), not as a separate standalone message; the outcome (refund confirmed / deposit kept) is included in the relevant confirmation.

**Data Notes:** Displayed: the plain-language policy on the booking page and at cancellation. Derived: the refund-vs-forfeit outcome for every cancellation is derived from comparing the cancellation timestamp to the booking's appointment time and the policy's window. Source: pro-set policy plus system clock.

**Interactions:** Feeds Public Booking Page & Booking Flow (FEAT-05) (policy display), Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11), and Pro Booking Management (FEAT-30) (pro-caused cancellation rule); refunds draw on the Pro's payout account (FEAT-28). [AUDIT-ADDED: 1 -- links to the audit-added FEAT-28 and FEAT-30]

**Signals:** cancellation_policy_updated, cancellation_within_window_flagged, deposit_refund_triggered, deposit_refund_failed, late_reschedule_treated_as_cancellation.

### Client-Initiated Cancel/Reschedule

**ID:** FEAT-10

**Description:** A client can cancel or reschedule their own booking, within the Pro's stated policy, without a phone call or a DM — from the reminder's one-tap option or the client's own booking-management link.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Client "reschedules or cancels within the policy window" as a defined capability, and the Vision's reminder flow ("I need to reschedule") depends on it existing. [RESEARCH-INFORMED: added market validation -- professionals report that self-service client booking meaningfully reduces phone and DM interruptions during the workday, from Capterra/SoftwareAdvice-aggregated reviews (MEDIUM confidence)]

**Connected Entities:** Booking (update — cancel or reschedule), Deposit Transaction (read — for policy outcome preview), Cancellation Policy (read)

**Key Capabilities:**
- Cancel an upcoming booking and see the deposit outcome before confirming
- Reschedule to a new genuinely free time for the same service, without a new deposit charge if within policy
- See the applicable cancellation window countdown before acting

**Primary Flows & Alternates:**
- Happy path: client accesses their booking (via FEAT-06), taps reschedule, picks a new free slot; the booking updates and both parties are notified.
- Alternate: client cancels inside the policy window; they see the deposit-forfeiture outcome plainly before confirming, so the action is never a surprise.
- Alternate: the desired new time is not available; the client sees the same real-time slot list as a fresh booking, never a stale or misleading option.
- Alternate: the client tries to reschedule inside the policy window; before confirming, they see plainly that the original deposit is kept under the Pro's policy and a new deposit is needed for the new time, and can back out with nothing changed. [AUDIT-ADDED: 1 -- counterpart/reversal walk: a late reschedule's deposit outcome was undefined]

**States:** Empty: N/A — this flow only exists against an existing booking. Loading: brief indicator while re-checking live availability for a reschedule. Error: if the update fails to save, the original booking remains untouched and intact rather than left in an ambiguous state. Offline-degraded: requires connectivity, consistent with the availability engine's correctness-first stance.

**Validation & Limits:** A reschedule must land on a slot that passes the same validation as a new booking (FEAT-03); a booking already marked completed or no-show cannot be cancelled or rescheduled.

**Access:** Own-only for the Client (their own booking only, verified via FEAT-06); anyone without a valid link for that booking sees only a "request a new link" prompt. The Pro sees the resulting change on their dashboard and makes their own changes through Pro Booking Management (FEAT-30). Platform Operator (Support) has View-only access. [MODIFIED: the Pro's own cancel/reschedule actions moved to the new Pro Booking Management feature (FEAT-30) so client-side and Pro-side deposit rules are each owned by one feature]

**Communications:** Triggers a cancellation or reschedule confirmation to the client and a change notice to the Pro's dashboard (via FEAT-08's messaging mechanism).

**Data Notes:** Captured: the new time (if rescheduling) or cancellation timestamp. Displayed: updated booking status to both parties. Derived: deposit outcome, via Cancellation & No-Show Policy Engine (FEAT-09).

**Interactions:** Depends on Client Booking Identity (FEAT-06), Real-Time Slot Availability Engine (FEAT-03), Cancellation & No-Show Policy Engine (FEAT-09); feeds Automated Booking Messaging (FEAT-08), Waitlist for Cancelled Slots (FEAT-20), and Booking & Payment Activity Record (FEAT-16).

**Signals:** booking_cancelled_by_client, booking_rescheduled_by_client, late_reschedule_warning_shown.

### No-Show Marking & Deposit Forfeiture

**ID:** FEAT-11

**Description:** The Pro marks a booking as a no-show when a client fails to appear, and the deposit is forfeited to the Pro automatically under the agreed policy — with no manual chasing, invoicing, or renegotiation required.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the founder's headline promise verbatim: "the pro never chases a no-show again," and BRIEF.md's Success Criteria states "Pros say 'I haven't had an unpaid no-show since I switched.'" The mechanism must be a single, low-effort action for the Pro. [RESEARCH-INFORMED: deposit-based no-show protection is consistently credited with materially reducing no-shows across three competitors (3 sources, HIGH confidence); because the deposit was already captured at booking, marking a no-show moves no money — it only makes the deposit non-refundable, so there is nothing left to chase]

**Connected Entities:** Booking (update — mark no-show), Deposit Transaction (update — forfeit)

**Key Capabilities:**
- Mark a past-due booking as a no-show in one tap from the daily schedule
- See the deposit automatically reflected as kept, with no separate invoicing step
- Reverse a mistaken no-show mark (e.g., the client did show up) within a short grace period
- Choose goodwill instead — refund the deposit in full for a genuine emergency through Pro Booking Management (FEAT-30) rather than marking a no-show [AUDIT-ADDED: 1 -- reversal path: BRIEF.md lets the Pro "refund within policy", which the no-show flow did not connect to]

**Primary Flows & Alternates:**
- Happy path: appointment time passes with the client absent; the Pro taps "no-show" from the dashboard; the deposit is marked forfeited automatically, and the record is retained for any future dispute.
- Alternate: Pro mistakenly marks a no-show; they can undo it within a short grace period, restoring the booking to completed and the deposit to its prior state.
- Alternate: a client disputes the no-show later; the Pro (or Platform Operator Support, if asked to help) can pull up the exact booking, its agreed policy version, and its timeline from Booking & Payment Activity Record (FEAT-16) as evidence.

**States:** Empty: N/A — this action only appears against a specific past-due booking. Loading: N/A — instantaneous local action. Error: a failed forfeiture write is retried and flagged, never silently dropped, since money is at stake. Offline-degraded: marking requires connectivity so the forfeiture is recorded reliably and immediately — this is exactly the "never lose a deposit" correctness bar from BRIEF.md.

**Validation & Limits:** A booking can only be marked no-show after its appointment start time has passed and before it auto-completes (7 days after the appointment, per FEAT-12); the undo grace period is fixed at 24 hours after marking, to prevent indefinite ambiguity in the client's own records. [MODIFIED: "on the order of a day" fixed to 24 hours and the marking window bounded by auto-completion -- synthesis output-completeness fix]

**Access:** The Pro has Full access to mark and unmark no-shows on their own bookings only. The Client sees the outcome (deposit kept) reflected in their own booking history but cannot mark or dispute it in-app beyond contacting the Pro directly. Platform Operator (Support) has View-only access, useful for dispute troubleshooting.

**Communications:** N/A — the deposit outcome is visible in the client's own booking history rather than triggering a separate confrontational notification; the Pro's action is deliberately low-friction and silent toward the client. [CHALLENGED: clients disputing deposit charges they were surprised by after the fact is a recurring complaint pattern, suggesting a neutral, factual outcome notice that restates the agreed policy may reduce disputes (2 source types, MEDIUM confidence) -- original retained; worth revisiting against the Policy Clarity at Booking metric]

**Data Notes:** Captured: the no-show marking action and timestamp. Displayed: updated booking status and deposit outcome to the Pro; deposit status to the client. Derived: the forfeiture amount, from Cancellation & No-Show Policy Engine (FEAT-09).

**Interactions:** Depends on Cancellation & No-Show Policy Engine (FEAT-09) and Deposit Payment at Booking (FEAT-07); invoked from Pro Daily Schedule Dashboard (FEAT-12); alternative path through Pro Booking Management (FEAT-30) (goodwill refund); feeds Booking & Payment Activity Record (FEAT-16) and Booking & Revenue Insights (FEAT-25). [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** booking_marked_no_show, no_show_mark_undone, deposit_forfeited.

### Pro Booking Management

**ID:** FEAT-30

**Description:** The Pro can cancel, reschedule, or refund any of their own bookings, and book a client in themselves (for example, rebooking a regular at the chair), with the deposit handled the same fair way every time and the client told what happened.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Pro "can mark no-shows, reschedule, cancel and refund within policy." The draft gave the Pro a feature for marking no-shows (FEAT-11) but only referenced the other three actions in passing inside the client-side feature, and never defined what happens to a client's deposit when the Pro is the one who cancels. Core because it is load-bearing for the brief's correctness bar: a Pro who falls ill with a full day booked must be able to unwind every booking without losing a client's money or trust. The in-person rebooking path keeps BRIEF.md's Success Criterion — "the link is the only way to book them" — true even at the chair, because the client still pays their deposit through a link. MVP: pros face cancellations and rebookings from their first week. [AUDIT-ADDED: 1 -- Core: counterpart-symmetry walk found no feature owning Pro-caused cancellations, Pro reschedules, or goodwill refunds, all of which BRIEF.md lists as Pro capabilities; without it the Pro has no correct way to cancel on a client, which directly threatens the "never lose a deposit" success criterion]

**Connected Entities:** Booking (create, update — cancel, reschedule), Deposit Transaction (update — refund), Client (create, read), Balance Payment (update — refund, from v1)

**Key Capabilities:**
- Cancel a client's booking -- the client's deposit is refunded in full automatically, whatever the timing, and the client is told
- Reschedule a booking to another genuinely free time -- the deposit carries over and the client receives the new time with a fresh manage link
- Refund a deposit in full as goodwill -- for an inside-window cancellation or instead of marking a no-show
- Book a client in on their behalf -- choose the service and time and an existing or new client; the client receives a deposit request link (or scans it from the Pro's screen at the chair) and the slot is held until they pay
- Cancel several bookings at once -- when blocking off a day that already has bookings (FEAT-17)

**Primary Flows & Alternates:**
- Happy path: at the end of an appointment the Pro books the client's next visit three weeks out; the client receives a payment link, pays the deposit in a tap, and the booking is confirmed exactly as if they had booked from the Instagram link.
- Alternate: the client does not pay a Pro-created deposit request in time; the held slot is released, the pending booking is marked expired, and the Pro is notified.
- Alternate: the Pro is ill and cancels the rest of the day; every affected client is refunded in full and notified; any refund that cannot complete yet is retried and flagged, never dropped.
- Alternate: the Pro reschedules a client to a time the client cannot make; the client can reschedule again from their manage link or cancel for a full refund, because the Pro moved the booking — the client is never penalized by the policy window for a change the Pro made.

**States:** Empty: N/A — this feature acts on an existing booking or on a new one the Pro is creating; it has no list of its own. Loading: a brief indicator while live availability is rechecked or a refund is processed. Error: a failed cancel, reschedule or refund leaves the booking exactly as it was with a retry action; a multi-booking cancellation reports the outcome for each booking. Offline-degraded: actions require connectivity for correctness; the most recently loaded schedule stays viewable read-only.

**Validation & Limits:** Pro-created and Pro-rescheduled bookings pass the same slot validation as any booking (FEAT-03), except that the Pro may book inside their own minimum notice or beyond their booking horizon; a Pro-created deposit request holds its slot for up to 24 hours or until 2 hours before the appointment, whichever comes first; refunds are always full in v1 (no partial amounts) and a deposit can be refunded only once; a goodwill refund is available until the booking is completed; a completed or auto-completed booking cannot be cancelled.

**Access:** The Pro has Full access to their own bookings only. Clients cannot use this feature — they change their own bookings through Client-Initiated Cancel/Reschedule (FEAT-10) and only receive the results of the Pro's actions. Platform Operator (Support) has View-only access to the outcomes. Anyone not signed in as the Pro is sent to the Pro sign-in screen (FEAT-29).

**Communications:** Client notices (via FEAT-08) for a Pro cancellation with refund, a Pro reschedule with the new time, a goodwill refund, and a deposit request; the deposit request goes by text only when the client has active texting consent, otherwise by email, or it can be shown on the Pro's screen for the client to scan; a Pro notification when a deposit request expires unpaid.

**Data Notes:** Captured: the action taken, when, an optional private reason, the new time for reschedules, refund amounts, and client details the Pro enters for a new booking. Displayed: the updated booking and deposit status. Derived: the deposit outcome, from the counterpart rules in Cancellation & No-Show Policy Engine (FEAT-09).

**Interactions:** Depends on Real-Time Slot Availability Engine (FEAT-03), Deposit Payment at Booking (FEAT-07), Cancellation & No-Show Policy Engine (FEAT-09), and Payout Account Connection & Payout Visibility (FEAT-28); invoked from Pro Daily Schedule Dashboard (FEAT-12), Manual Time Blocking (FEAT-17), and No-Show Marking & Deposit Forfeiture (FEAT-11) (goodwill instead of no-show); triggers Automated Booking Messaging (FEAT-08) and Two-Way Calendar Sync (FEAT-04) updates; feeds Booking & Payment Activity Record (FEAT-16).

**Signals:** booking_cancelled_by_pro, booking_rescheduled_by_pro, goodwill_refund_issued, pro_booking_created, deposit_request_paid, deposit_request_expired.

### Pro Daily Schedule Dashboard

**ID:** FEAT-12

**Description:** The Pro's primary, phone-first view: today's (and upcoming) bookings, each with a paid badge, a client note, and how much balance is still due in person — the screen the Pro glances at between clients.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Vision describes this exactly: "As the pro, you glance at your phone between clients: today's list, each booking with a paid badge, a client note, and how much is still due in person." This is the Pro's single most frequent touchpoint with the product. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read, update — quick actions and completion), Client (read), Deposit Transaction (read), Message (read — delivery flags) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- View today's bookings at a glance, in time order, with paid/unpaid and balance-due status
- View upcoming bookings beyond today
- Take quick actions directly from the list: mark no-show, view client note, jump to reschedule/cancel
- Mark a finished appointment as completed (recording the balance as settled in person), or let it complete automatically [AUDIT-ADDED: 3 -- the Booking entity's Completed state had no owning transition]
- See which clients tapped "I'll be there", and an attention list of anything needing action (sync issue, message delivery failure, refund failure, card-issuer dispute, bookings left outside changed hours) [AUDIT-ADDED: 1 -- journey walk of Talia's Between-Clients Day: flags raised by FEAT-03, FEAT-04, FEAT-08, FEAT-09 and FEAT-16 all pointed at "the dashboard" with no defined place to land]
- Browse past bookings by date [AUDIT-ADDED: 1 -- journey walk of Resolving a No-Show Dispute: finding a past booking needed a defined path]

**Primary Flows & Alternates:**
- Happy path: Pro opens the dashboard between clients and sees the day's remaining bookings, each with status, in seconds.
- Alternate: an empty day (no bookings) shows a plain, encouraging state rather than looking broken, with a shortcut to share the booking link.
- Alternate: a booking's calendar-sync status is uncertain (FEAT-04 flagged an issue); the dashboard visibly marks that booking's reliability rather than presenting it with false confidence.

**States:** Empty: a day with zero bookings shows a friendly "nothing booked yet today" state, never a bare blank screen. Loading: bookings render with a lightweight in-place indicator on slow connections. Error: a failed load shows the last successfully loaded data with a retry action, never an unexplained blank dashboard. Offline-degraded: the most recently loaded schedule remains viewable read-only; actions (like marking no-show) require reconnecting.

**Validation & Limits:** Quick actions are validated by their own features (FEAT-11, FEAT-30); a booking can be marked completed only after its appointment start time; a booking not marked either way auto-completes 7 days after the appointment; all times display in the Pro's timezone. [AUDIT-ADDED: 3 -- completion rules for the Booking entity's Completed state]

**Access:** The Pro has Full access to their own schedule only. Clients have no access to this view (they see only their own bookings, through FEAT-06); anyone who is not signed in as the Pro is sent to the Pro sign-in screen (FEAT-29). Platform Operator (Support) has View-only access for troubleshooting a specific reported issue.

**Communications:** N/A — this is a viewing surface; it does not itself send messages.

**Data Notes:** Captured: completion marks (and whether the balance was settled in person). Displayed: booking time, service, client name, the Pro's private note and the client's booking note, paid status, balance due, "I'll be there" status, attention flags. Derived: balance due (service price minus deposit paid, minus any in-app balance payment from v1). Source: reads Booking, Client, Deposit Transaction, and Message records created elsewhere. [AUDIT-ADDED: 3 -- completion capture and message flags mapped to entities]

**Interactions:** Depends on Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), Automated Booking Messaging (FEAT-08), Client Record Management (FEAT-13); surfaces quick actions into No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Booking Management (FEAT-30), and Manual Time Blocking (FEAT-17). [MODIFIED: pro-initiated reschedule/cancel quick actions now route to Pro Booking Management (FEAT-30) instead of the client-side feature]

**Signals:** dashboard_viewed, quick_action_taken (with action type), booking_marked_completed, booking_auto_completed, attention_item_resolved.

## Important Features

### Client Record Management

**ID:** FEAT-13

**Description:** The Pro maintains a simple record for each client — contact details, private notes, and booking history with that Pro — and can permanently delete a client's record on request.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Target Users & Roles states the Pro "sees every... client" and "can delete a client's record on request," and the Constraints section makes deletion-on-request an explicit regulatory-adjacent obligation. Ranked Important rather than Core because a client record is created automatically by the act of booking (FEAT-05) — this feature is about managing that record afterward, not about the core booking loop itself. MVP phase: the delete-on-request obligation is a launch-blocking privacy commitment, not something safe to defer. [RESEARCH-INFORMED: added competitor weakness context -- difficulty removing stored client payment-card data, requiring manual request forms, is a frequently mentioned complaint about one competitor; because Chairtime never holds card data, a client deletion here is complete in one action, from Capterra reviews and BBB complaint records (MEDIUM confidence)]

**Connected Entities:** Client (read, update, delete)

**Key Capabilities:**
- View a client's contact details and full booking history with this Pro
- Add or edit a private note about a client (preferences, allergies noted informally, etc.)
- Permanently delete a client's record on their request
- Correct a client's name, email, or phone number (for example, a typo at booking) [AUDIT-ADDED: 3 -- entity coverage: the Client entity had no edit path for its contact details]

**Primary Flows & Alternates:**
- Happy path: Pro opens a client from the dashboard or booking list, reviews history, and adds a quick note after an appointment.
- Alternate: a client requests deletion; the Pro deletes the record, which removes contact details and notes while past financial records needed for dispute/audit purposes (FEAT-16) are retained in de-identified form per data-retention obligations.
- Alternate: a deleted client books again later; a new client record is created — the system does not silently resurrect the old one.

**States:** Empty: a client with no notes yet shows a plain empty note field, not an error. Loading: N/A — instant for the small per-pro client volumes described in BRIEF.md (100–500 clients). Error: a failed save preserves entered text with a retry option. Offline-degraded: the most recently loaded client list remains viewable read-only.

**Validation & Limits:** Notes are free text up to 1,000 characters; a changed phone number takes effect for access links (FEAT-06) and texting consent must be given again for the new number; a client with an upcoming booking cannot be deleted until that booking is cancelled (the Pro is offered a one-step cancel with full refund through FEAT-30); deletion is a deliberate, confirmed action (not reversible, consistent with "delete on request" meaning delete). [MODIFIED: note length fixed at 1,000 characters and the upcoming-booking and phone-change rules added -- entity coverage audit]

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

**Rationale:** BRIEF.md's Constraints state plainly: "clients must explicitly agree to receive texts when they book, and reminders must respect that consent," citing US texting rules as the reason. Ranked Important rather than Core because it is a compliance-and-preference layer underneath the Core messaging feature (FEAT-08) rather than a capability a client seeks out on its own; it must still ship in MVP because it is a regulatory precondition for FEAT-08 to operate lawfully. [INFERRED: carried from Visionary draft]

**Connected Entities:** Messaging Consent (create, update)

**Key Capabilities:**
- Capture explicit opt-in at the moment of booking (never pre-checked)
- Let a client withdraw consent at any time via a link included in messages
- Fall back to email automatically for any client without active texting consent
- Let a client opt back in to texts later from their own booking view (FEAT-06) or on their next booking [AUDIT-ADDED: 3 -- entity coverage: the Messaging Consent lifecycle had no re-grant path]

**Primary Flows & Alternates:**
- Happy path: client checks the opt-in box at booking; all future confirmations and reminders for that pro go by text.
- Alternate: client replies "STOP" or uses an opt-out link; texting consent is revoked immediately and future messages fall back to email.
- Alternate: a client never opted in at all; every message for their bookings goes by email from the start, with no degraded experience implied.

**States:** Empty: N/A — consent is always tied to a specific client-pro relationship, captured at first booking. Loading: N/A — instantaneous local state. Error: a failed consent-state update is treated conservatively — if in doubt, the system defaults to the safer (no-text) state rather than risk texting without valid consent. Offline-degraded: N/A — this is a background compliance state, not an interactive screen.

**Validation & Limits:** Consent must be an explicit, unchecked-by-default action; a revoke request is honored on the very next message sent, with no grace period.

**Access:** A Client manages only their own consent (Own-only). The Pro sees whether a given client can currently be texted (View, for planning purposes) but cannot override a client's revoked consent. Platform Operator (Support) has View-only access.

**Communications:** N/A — this feature governs communications rather than sending its own, aside from an opt-out confirmation acknowledgment.

**Data Notes:** Captured: opt-in/opt-out state, timestamp, channel, and the exact consent wording shown, kept as evidence of consent. Displayed: current consent status to the Pro (for planning) and to the client (in their own preferences). Derived: none. Source: direct client action at booking or via an opt-out link. [AUDIT-ADDED: 4 -- Compliance concern: the consent wording shown is kept as evidence]

**Interactions:** Depended on by Automated Booking Messaging (FEAT-08); fed by Public Booking Page & Booking Flow (FEAT-05).

**Signals:** consent_granted, consent_revoked, message_routed_to_fallback_email.

### Pro Onboarding & Setup Wizard

**ID:** FEAT-15

**Description:** The guided, one-time setup flow that takes a brand-new Pro from signup to a live, shareable booking link — services, hours, deposit rule, cancellation policy, and calendar connection, in a sensible order with sensible defaults.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** Every product has a first run, and this one has an unusually high stakes first run: BRIEF.md's Constraints name a three-month runway to the first paying pro, so setup friction directly threatens the founder's timeline. Ranked Important rather than Core because, once complete, the wizard itself is never used again — the Core features it configures are what deliver ongoing value. MVP phase: a Pro cannot reach any Core feature without it. [RESEARCH-INFORMED: simplicity and fast setup are frequently praised in comparisons aimed at solo operators, contrasted against salon-scale tools, from independent comparison guides (MEDIUM confidence)]

**Connected Entities:** Pro Account (create), Service (create), Availability Rule (create), Cancellation Policy (create), Subscription (create), Payout Account (create — via FEAT-28), Calendar Connection (create — offered, not required, to complete) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- Guided, ordered setup: account and sign-in (FEAT-29) -> profile basics and studio location (FEAT-27) -> at least one service -> working hours -> deposit rule -> cancellation policy -> payout account (FEAT-28) -> (optional) calendar connection -> subscription payment [MODIFIED: sign-in, profile and payout-account steps added because the completeness audit found a Pro could otherwise finish setup with no way to sign back in, no studio location for clients, and nowhere for deposits to go]
- A preview of the booking page exactly as a client will see it, plus short plain-language tips at each step [AUDIT-ADDED: 4 -- Help and Guidance concern: the draft had no contextual guidance for a non-technical Pro]
- Sensible defaults offered at each step (e.g., a common cancellation window) that the Pro can accept or change
- A shareable booking link generated the moment setup is minimally complete

**Primary Flows & Alternates:**
- Happy path: Pro signs up, moves through each step in order, accepting or adjusting defaults, and receives their shareable link at the end.
- Alternate: Pro abandons setup partway through; on return, the wizard resumes exactly where they left off with earlier answers preserved, never forcing a restart.
- Alternate: Pro skips connecting a calendar during setup; they can complete the rest of onboarding and connect it later from settings without being blocked.

**States:** Empty: N/A — the wizard itself is the empty-state handler for a new account. Loading: N/A — each step is a simple form with instant local response. Error: a failed step preserves entered values with a retry option, consistent with every setup screen in this product. Offline-degraded: N/A — setup is a deliberate, connected session, not an in-the-moment mobile flow.

**Validation & Limits:** The booking link is not generated until the minimum required steps (account and sign-in, display name and studio location, one service, working hours, deposit rule, cancellation policy, an active payout account, active subscription) are complete; calendar connection is the one optional step. [MODIFIED: profile and payout-account prerequisites added to match the revised setup order]

**Access:** The Pro has Full access to their own onboarding. No other role touches this feature. Platform Operator (Support) has View-only access to see how far a specific pro has progressed, useful for support.

**Communications:** A welcome confirmation once the booking link goes live.

**Data Notes:** Captured: every setup field listed under Connected Entities. Displayed: setup progress and the resulting live link. Derived: none. Source: direct pro input at each step.

**Interactions:** Depends on Pro Sign-In & Account Lifecycle (FEAT-29) and Payout Account Connection & Payout Visibility (FEAT-28); feeds Service & Pricing Management (FEAT-01), Availability & Working Hours Setup (FEAT-02), Cancellation & No-Show Policy Engine (FEAT-09), Two-Way Calendar Sync (FEAT-04), Pro Subscription Billing & Account Management (FEAT-18), and Pro Profile & Booking Page Settings (FEAT-27). [MODIFIED: dependencies on the audit-added FEAT-27, FEAT-28 and FEAT-29 added]

**Signals:** onboarding_started, onboarding_step_completed (with step name), onboarding_completed, onboarding_link_shared.

### Pro Profile & Booking Page Settings

**ID:** FEAT-27

**Description:** The Pro controls how they appear and how their booking page behaves — display name, photo, short intro, studio location, booking link name, timezone and currency, pausing new bookings for a holiday, and which notifications they get — and can ask for help from the same place.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** BRIEF.md's Experience has the client see "the pro's name" on the booking page, its Scale section requires that "timezone and currency must not be hard-coded," and a client of a rented chair or home studio cannot find the appointment without a location — yet no draft feature let the Pro edit any of these after onboarding. Important rather than Core because the booking loop still works on the values set once during setup; MVP because a Pro who moves studio or goes on holiday must be able to change them from day one. The studio's full address appears only in a booked client's confirmation, protecting home-studio pros from publishing their home address to anyone who taps the link. [AUDIT-ADDED: 3 -- the Pro Account entity had no feature for editing its profile, link, timezone, currency, pause state, or notification preferences] [RESEARCH-INFORMED: the visual quality and branding of the booking page is consistently praised by users of the closest competitor, from Capterra/GetApp reviews (HIGH confidence); a photo and short intro serve that without adding design tooling]

**Connected Entities:** Pro Account (read, update)

**Key Capabilities:**
- Edit the public profile -- display name, photo, short intro, and the general area shown publicly
- Set the full studio address -- shown only in booked clients' confirmations and reminders
- Choose or change the booking link name -- the previous link keeps forwarding so an Instagram bio link never breaks
- Set timezone and currency for the account
- Pause new bookings with an optional message (for example, "on holiday until 3 June") and resume them -- existing bookings are unaffected
- Choose which Pro notifications to receive and how (in-app, text, or email)
- Preview the booking page as a client sees it
- Send a help request to support from settings

**Primary Flows & Alternates:**
- Happy path: the Pro updates their photo and intro between clients; the public booking page shows the change immediately.
- Alternate: the Pro renames their booking link; the old link forwards to the new one, so clients using an old bio link or an old story still land on the right page.
- Alternate: the Pro pauses bookings for a week; the booking page shows their message instead of available times, existing bookings keep their reminders, and bookings resume automatically on the chosen date or when the Pro turns the pause off.
- Alternate: the Pro changes timezone (for example, after moving); existing bookings keep their real moment in time and are shown converted, with a clear warning before saving.

**States:** Empty: a new profile shows the values captured during onboarding, with a gentle prompt to add a photo. Loading: N/A — a small settings screen that renders at once. Error: a failed save keeps the entered values with a retry action; a link name already taken is explained with suggestions. Offline-degraded: current settings stay viewable read-only; changes require connectivity.

**Validation & Limits:** Display name 1–60 characters; intro up to 300 characters; photo must be a standard image under a reasonable size limit; booking link name 3–40 letters, numbers or hyphens and unique across all pros; currency cannot be changed after the first deposit is taken; the old link name forwards for at least 12 months and cannot be claimed by another pro during that time; a pause end date cannot be in the past.

**Access:** The Pro has Full access to their own settings. Clients see only the public profile fields on the booking page (and the full studio address only in their own booking confirmation) and never see settings. Platform Operator (Support) has View-only access for troubleshooting. Anyone not signed in as the Pro is sent to the Pro sign-in screen (FEAT-29).

**Communications:** An acknowledgment when a help request is received; changes to settings send no client messages.

**Data Notes:** Captured: display name, photo, intro, general area, full studio address, link name (and previous names), timezone, currency, pause state and message, notification preferences, help requests. Displayed: the public profile on the booking page; the full address in confirmations. Derived: none. Source: Pro input.

**Interactions:** Feeds Public Booking Page & Booking Flow (FEAT-05) (profile, pause state) and Automated Booking Messaging (FEAT-08) (studio address, Pro notification preferences); initial values come from Pro Onboarding & Setup Wizard (FEAT-15).

**Signals:** profile_updated, booking_link_renamed, bookings_paused, bookings_resumed, booking_page_previewed, help_request_sent.

### Pro Sign-In & Account Lifecycle

**ID:** FEAT-29

**Description:** The Pro signs in securely from their phone and stays signed in between clients, can get back in if they lose access to one of their contact methods, can take a copy of their clients and booking history with them at any time, and can close their account with their data deleted afterward.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The draft referred to "the Pro's own dashboard login" (FEAT-06) but no feature owned signing in, recovering access, or closing the account. The Pro's account holds every client's contact details, so protecting it is part of BRIEF.md's Privacy requirement ("a client's data is visible only to their pro"), and BRIEF.md's Business Context promises "cancel anytime," which is only honest if a Pro can leave with their records. Important rather than Core because it guards access to the Core features rather than delivering the booking value itself; MVP because a Pro cannot reach their dashboard without it. [AUDIT-ADDED: 4 -- Account Management, Data Management, and Security and Privacy Posture concerns were not owned by any feature; sign-in, recovery, data export and account closure are grouped here]

**Connected Entities:** Pro Account (create — sign-in identity; update; delete), Client (read — for export), Booking (read — for export), Deposit Transaction (read — for export)

**Key Capabilities:**
- Create a sign-in during onboarding using an email address and a mobile number, with a one-time code rather than a password to remember
- Stay signed in on their own phone between clients, and sign out of every device at once
- Recover access through whichever of the two contact methods they still have
- Change the sign-in email or phone, confirmed through both the old and the new contact
- Download a copy of their client list and booking history as a spreadsheet-friendly file
- Close the account -- subscription cancelled, booking page taken down, data deleted after a cooling-off period

**Primary Flows & Alternates:**
- Happy path: the Pro signs in once on their phone and stays signed in for weeks, opening the dashboard with a single tap between clients.
- Alternate: the Pro gets a new phone number; they sign in with their email, confirm the new number, and carry on without contacting anyone.
- Alternate: the Pro closes their account while they still have upcoming bookings; they are shown those bookings and offered a one-step cancellation with full refunds (FEAT-30) before closure can proceed.
- Alternate: the Pro changes their mind during the cooling-off period; they sign in and reopen the account with everything intact (the booking page stays down until they resume).

**States:** Empty: N/A — sign-in is the entry screen itself, never a list. Loading: a brief indicator while a code is sent or checked. Error: a wrong or expired code is explained plainly with a "send a new code" action; repeated failures slow further attempts. Offline-degraded: signing in requires connectivity; a Pro already signed in keeps read-only access to their last loaded schedule.

**Validation & Limits:** One-time codes expire after 10 minutes; after 5 failed attempts, further attempts are paused for 15 minutes; a sign-in stays active on a device for up to 30 days of inactivity; the cooling-off period before permanent deletion is 30 days; after deletion, only the financial records the law requires to be retained are kept, in de-identified form; the data export covers the Pro's own clients and bookings only.

**Access:** The Pro has Full access to their own sign-in and account. Clients never have Pro-style sign-ins — they use the password-free Client Booking Identity (FEAT-06) — and cannot reach this feature. Platform Operator (Support) has View-only access to account status (active, paused, closing) and can never see sign-in codes or sign in as the Pro. A person who fails sign-in sees only a generic "that code didn't work" message that reveals nothing about whether an account exists.

**Communications:** One-time sign-in codes; an alert when the account is signed in on a new device; confirmations of contact-detail changes to both old and new contacts; an account-closure confirmation and a final notice when data is permanently deleted.

**Data Notes:** Captured: sign-in email and mobile number, signed-in devices, closure request date. Displayed: signed-in devices, account status. Derived: the export file, generated on request from Client, Booking and Deposit Transaction records. Source: Pro input.

**Interactions:** Provides the sign-in used by every Pro-facing feature, including Pro Daily Schedule Dashboard (FEAT-12); created during Pro Onboarding & Setup Wizard (FEAT-15); closure cancels Pro Subscription Billing & Account Management (FEAT-18) and hands upcoming bookings to Pro Booking Management (FEAT-30).

**Signals:** pro_signed_in, sign_in_code_failed, signed_out_everywhere, contact_details_changed, data_export_downloaded, account_closure_requested, account_reopened, account_deleted.

### Booking & Payment Activity Record

**ID:** FEAT-16

**Description:** An always-on, append-only record of every booking's key events — created, paid, confirmed, messaged, cancelled/rescheduled, marked no-show, refunded/forfeited — so the Pro (or, when asked to help, Platform Operator Support) has a trustworthy timeline to point to if a client ever disputes a charge.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Problem Statement names this precisely as a current failure: "no record when a client disputes a no-show charge." Ranked Important rather than Core because it is a record-keeping layer that supports the Core booking/deposit/no-show features rather than something a user directly seeks out day to day; it must still ship at MVP because the dispute scenario it prevents is a launch-day risk, not a later refinement. [INFERRED: carried from Visionary draft]

**Connected Entities:** Activity Event (create — written automatically), Booking (read), Deposit Transaction (read), Message (read) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- View a chronological timeline of everything that happened to a specific booking
- See exactly which cancellation policy version applied and when it was shown to the client
- Reference this record when responding to a client's dispute
- See a booking flagged when the client raises a dispute with their card issuer, and download a plain, shareable summary of the booking's timeline (policy shown and acknowledged, booking time, messages sent, no-show mark) to use as evidence with the payment processor [AUDIT-ADDED: 1 -- counterpart symmetry: a client can contest a kept deposit through their card issuer, not only by messaging the Pro; the Pro needed the record in a form they can submit]

**Primary Flows & Alternates:**
- Happy path: a client disputes a no-show charge; the Pro opens the booking's timeline and sees the exact policy shown at booking, the appointment time, and the no-show mark's timestamp.
- Alternate: Platform Operator (Support) is asked to help with a dispute; they can view the same timeline read-only without being able to alter it.
- Alternate: a booking has an unusual gap (e.g., a message failed to send); the timeline shows that gap plainly rather than presenting a falsely clean record.

**States:** Empty: N/A — a timeline only exists for bookings that have happened; a brand-new booking simply starts its timeline at "created." Loading: N/A — small per-booking dataset, loads instantly. Error: N/A — this is a read-only, append-only log; there is no user-facing write path to fail. Offline-degraded: the most recently loaded timeline remains viewable read-only.

**Validation & Limits:** Entries are append-only and immutable once written — this is the property that makes the record trustworthy as dispute evidence; retained for as long as the associated Booking record exists.

**Access:** The Pro has View access to their own bookings' timelines (nobody, including the Pro, can edit an entry). Platform Operator (Support) has View-only access. [MODIFIED: "Full (view)" restated as View to agree with the Access Matrix "Activity Record & Insights" column] Clients do not see this internal timeline directly — they see the outcomes (their own confirmation, deposit status) through their own booking view, not this operational record.

**Communications:** A Pro notification (via FEAT-08) when a card-issuer dispute is raised on a booking; otherwise a passive record.

**Data Notes:** Displayed: a chronological event list per booking. Derived: entirely — every entry is written automatically as other features act on the booking; nothing is directly entered here.

**Interactions:** Reads from Public Booking Page & Booking Flow (FEAT-05), Deposit Payment at Booking (FEAT-07), Automated Booking Messaging (FEAT-08), Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11).

**Signals:** activity_record_viewed, card_dispute_flagged, dispute_summary_downloaded.

### Manual Time Blocking

**ID:** FEAT-17

**Description:** The Pro can block off a span of time — a doctor's appointment, a vacation day, a personal commitment — removing it from bookable availability without needing to edit their recurring working hours.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** A direct, near-universal need once recurring hours exist: a Pro's actual availability always has one-off exceptions. Without it, the Pro would be forced to edit recurring hours for a single day, which is error-prone and easy to forget to revert. Important rather than Core: the product still functions on recurring hours alone at a pinch, but reliability (a hallmark of this brief) suffers without it. MVP phase: this is a day-one operational need, not a later refinement. [INFERRED: carried from Visionary draft]

**Connected Entities:** Time Block (create, update, delete)

**Key Capabilities:**
- Block a span of time on a specific date (or a recurring pattern, e.g., "every Sunday")
- Remove a block to restore availability
- See blocked time reflected immediately in the slot engine

**Primary Flows & Alternates:**
- Happy path: Pro blocks tomorrow afternoon; that window disappears from bookable slots immediately.
- Alternate: a block is added over an already-booked slot; the existing booking is never silently affected — the Pro is warned and must explicitly decide (cancel or reschedule the affected bookings through Pro Booking Management, FEAT-30, with full refunds for cancellations, or leave the booking as an exception to the block). [AUDIT-ADDED: 1 -- reversal path: "contact the client" left the Pro to handle deposits by hand; the choice now routes into FEAT-30]
- Alternate: Pro removes a block early; the previously blocked time becomes bookable again right away.

**States:** Empty: a Pro with no blocks sees a plain "no time blocked" state. Loading: N/A — instant, small dataset. Error: a failed save is retried with entered values preserved. Offline-degraded: N/A — a setup-style action requiring connectivity for correctness.

**Validation & Limits:** A block's end time must be after its start time; a block cannot silently delete an existing conflicting booking.

**Access:** The Pro has Full access to their own blocks. Clients never see blocks directly — only their absence from available slots. Platform Operator (Support) has View-only access.

**Communications:** N/A — blocking time is a private scheduling action with no message trigger.

**Data Notes:** Captured: block start/end and an optional label (private to the Pro). Displayed: on the Pro's own schedule view. Derived: none.

**Interactions:** Feeds Real-Time Slot Availability Engine (FEAT-03); hands conflicting bookings to Pro Booking Management (FEAT-30). [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** time_block_added, time_block_removed, time_block_conflict_flagged.

### Pro Subscription Billing & Account Management

**ID:** FEAT-18

**Description:** The Pro's own flat monthly subscription to Chairtime — one price tier, card-based, cancel anytime — including seeing their current plan status and updating their payment method.

**Priority:** Important

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** BRIEF.md's Business Context is explicit: "revenue comes from a flat monthly subscription paid by each pro... cancel anytime, with one price tier in v1... no per-booking cut." Ranked Important rather than Core because it is the business's monetization mechanism rather than part of the client-facing booking loop the founder's headline promise describes; it must still ship at MVP because the product has no revenue model without it. [RESEARCH-INFORMED: added the market's dominant trust complaint -- unpredictable, layered fees on top of the advertised subscription are reported across three competitors, and no profiled competitor offers a flat, all-inclusive price with no per-booking or per-new-client charge, so the one price is shown up front with every feature included and no add-ons, from BBB, Capterra and Reddit-derived sources (3 products, HIGH confidence)]

**Connected Entities:** Subscription (create, update, cancel)

**Key Capabilities:**
- Subscribe during onboarding with a card
- View current plan status and next billing date
- Update the payment method on file
- Cancel the subscription at any time, effective at the end of the current billing period
- See the single all-inclusive price and a plain statement that Chairtime takes nothing from deposits, balances, or tips [RESEARCH-INFORMED: fee transparency, per the HIGH-confidence finding above]

**Primary Flows & Alternates:**
- Happy path: Pro enters payment details once during onboarding; the subscription renews automatically each month with no further action.
- Alternate: a renewal payment fails; the Pro is notified and given a grace period to update their payment method before the account is paused (booking page taken offline to new bookings, but existing bookings and data preserved, and existing bookings keep their reminders, refunds and client self-service exactly as before). [AUDIT-ADDED: 1 -- counterpart symmetry: clients with existing bookings must not be affected by the Pro's billing lapse]
- Alternate: Pro cancels; the subscription remains active through the already-paid period and then lapses, with the account paused (not deleted) afterward.

**States:** Empty: N/A — an account cannot exist past onboarding without an active subscription. Loading: N/A — plan status is a small, instant read. Error: a failed payment update is retried with a clear reason and retry action. Offline-degraded: the last known plan status remains viewable read-only.

**Validation & Limits:** One price tier only in v1 — no plan selection is offered; the grace period after a failed renewal is 7 days; cancellation takes effect at the end of the already-paid period, never an immediate mid-period cutoff that would feel like losing paid time; any future price change is announced to the Pro at least 30 days before it applies; closing the account and deleting its data is handled by Pro Sign-In & Account Lifecycle (FEAT-29). [MODIFIED: grace period fixed at 7 days and 30-day price-change notice added, per the HIGH-confidence fee-trust finding above]

**Access:** The Pro has Full access to their own subscription. Platform Operator (Support) has View-only access to plan status, useful for billing support questions. Clients have no visibility into this at all.

**Communications:** Payment-failure notice with a grace-period deadline; renewal receipt; cancellation confirmation.

**Data Notes:** Captured: subscription status, billing cycle, payment method reference (the payment-processing capability owns the actual card data, per BRIEF.md's Constraints). Displayed: current plan status and next billing date. Derived: none.

**Interactions:** Created during Pro Onboarding & Setup Wizard (FEAT-15); a lapse pauses Public Booking Page & Booking Flow (FEAT-05) for new bookings; cancelled as part of account closure in Pro Sign-In & Account Lifecycle (FEAT-29); referenced by Platform Support Read-Only Access (FEAT-19). [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** subscription_started, subscription_payment_failed, subscription_payment_recovered, subscription_cancelled.

### Platform Support Read-Only Access

**ID:** FEAT-19

**Description:** The founder, in a support capacity, can open a read-only view into a specific Pro's account — services, schedule, bookings, and billing status — to help troubleshoot a reported problem, with no ability to edit anything and no client-facing access of any kind.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** BRIEF.md's Target Users & Roles names this directly: "the founder needs only a read-only support view of a pro's account to help them... It is minimal admin access." Ranked Important rather than Core because it serves the business's operational need rather than either product role's own value; still needed at MVP because support requests will arrive from day one with real, paying pros. [RESEARCH-INFORMED: slow, email-only customer support and unanswered payout questions are frequently mentioned complaints about three competitors, so a support view that lets the founder diagnose a problem without a screen-share is a trust asset, from BBB complaint records and aggregator reviews (MEDIUM confidence)]

**Connected Entities:** Pro Account (read), Service (read), Booking (read), Client (read — excluding private notes), Subscription (read), Payout Account (read — status only), Activity Event (create — each support view is logged) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- Look up a specific Pro's account by request
- View their services, schedule, bookings, and billing status read-only
- View booking timelines (FEAT-16) to help resolve a dispute
- Every support view is recorded in the Pro's account activity, which the Pro can see [AUDIT-ADDED: 4 -- Audit Logging concern: access to a Pro's client data by anyone other than the Pro needed a "who looked, when" trail consistent with BRIEF.md's privacy posture]

**Primary Flows & Alternates:**
- Happy path: a Pro reports a confusing issue; the founder opens the read-only view, diagnoses it (e.g., a lapsed calendar connection), and guides the Pro to fix it themselves.
- Alternate: the founder attempts an action outside read-only scope (there is none available in this view by design) — the interface simply offers no edit controls at all, removing the possibility rather than blocking it after the fact.
- Alternate: the founder is asked about a client dispute; they view the relevant booking's activity record (FEAT-16) but not the Pro's private client notes, respecting the client-record privacy boundary even in support.

**States:** Empty: N/A — this view only exists once a specific Pro Account is looked up. Loading: N/A — small per-account dataset. Error: N/A — a read-only view with no write path to fail. Offline-degraded: N/A — an operational tool used in a connected context.

**Validation & Limits:** No write actions exist in this view at all — the constraint is structural, not a permission check that could be bypassed.

**Access:** Platform Operator (Support) has View access product-wide, scoped to one account at a time, and never to the Pro's private client notes, bank or identity details, or sign-in credentials. It is used only when a Pro has asked for help, and the Pro has View access to the log of when support viewed their account. Clients have no access and are unaffected. [MODIFIED: support views are now visible to the Pro rather than invisible, per the audit-logging addition above]

**Communications:** N/A — this is an internal tool with no client- or pro-facing messages of its own.

**Data Notes:** Displayed: read-only mirror of the Pro's own data, minus the Pro's private client notes. Derived: none. Source: reads existing records only; creates nothing.

**Interactions:** Reads Pro Onboarding & Setup Wizard (FEAT-15) output, Pro Daily Schedule Dashboard (FEAT-12) data, Booking & Payment Activity Record (FEAT-16), Pro Subscription Billing & Account Management (FEAT-18).

**Signals:** support_view_opened (with reason/ticket reference), support_view_log_viewed_by_pro.

## Nice-to-Have Features

### Waitlist for Cancelled Slots

**ID:** FEAT-20

**Description:** A client can ask to be notified if a specific service and day opens up from someone else's cancellation, instead of repeatedly checking the booking page.

**Priority:** Nice-to-Have

**Phase:** v1

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions names this exactly: "should a pro be able to offer a waitlist for slots that open up from cancellations?" As Visionary judgment: valuable but not required for the core one-minute-booking promise, and it depends on Client-Initiated Cancel/Reschedule (FEAT-10) already existing to generate openings. Phased to v1, once the core cancellation flow is proven. [RESEARCH-INFORMED: waitlists are offered by the closest solo-focused competitor as part of its business tools, from independent review aggregators (1 profile, MEDIUM confidence), confirming the pattern without making it a baseline expectation]

**Connected Entities:** Waitlist Entry (create, update), Service (read)

**Key Capabilities:**
- Join a waitlist for a specific service/day when no slot is currently free
- Get notified the moment a matching slot opens from a cancellation
- Book directly from the notification before anyone else can grab the slot
- Leave a waitlist at any time from their own booking view [AUDIT-ADDED: 3 -- entity coverage: Waitlist Entry had no client-side removal]

**Primary Flows & Alternates:**
- Happy path: client finds no free slot, joins the waitlist for that day; a cancellation opens a matching slot; the client is notified and books within a short priority window.
- Alternate: two clients are on the waitlist for the same opening; the first to act on the notification gets it, and the other remains on the waitlist for the next opportunity.
- Alternate: the waitlist entry expires unclaimed after a set period with no matching opening; the client is informed rather than left wondering indefinitely.

**States:** Empty: no waitlist entries shows a plain "you're not on any waitlists" state. Loading: N/A — small dataset. Error: a failed join is retried. Offline-degraded: N/A — requires connectivity to reliably capture a fast-moving opening.

**Validation & Limits:** A waitlist entry is tied to one service and one day (or a range of up to 7 days); a client may hold at most 3 active waitlist entries per Pro; the claim window after a notification is 30 minutes; a waitlist notification goes by text only with active texting consent, otherwise by email. [MODIFIED: "small, bounded" limits made testable -- synthesis output-completeness fix]

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

**Rationale:** BRIEF.md's Open Questions asks directly whether this is "v1 or later." As Visionary judgment: valuable for retention-style services (lash fills, haircuts) but not required for the founder's three-month first-paying-pro timeline, and it adds real complexity to the availability engine. Phased to v1, once the single-booking core loop is proven reliable. [INFERRED: carried from Visionary draft]

**Connected Entities:** Recurring Series (create, update, cancel), Booking (create — generated per occurrence)

**Key Capabilities:**
- Set up a recurring pattern from an existing booking ("repeat this every N weeks")
- See and manage the upcoming generated occurrences as a group
- Cancel the whole series, or just one upcoming occurrence, independently

**Primary Flows & Alternates:**
- Happy path: after booking, the client opts into a recurring pattern; future occurrences are generated and each still requires its own deposit per the Pro's rule, paid through a deposit link sent about a week before that occurrence (card details are never kept between occurrences). [AUDIT-ADDED: 1 -- value-flow walk: with no stored card, each occurrence's deposit needed a defined collection path]
- Alternate: a future occurrence's usual time is no longer available (the Pro changed hours); the client is notified in advance and asked to pick a new time for that occurrence only, without breaking the rest of the series.
- Alternate: client cancels just one occurrence; the series continues generating future ones normally.

**States:** Empty: a client with no recurring series sees no extra UI at all — this is fully optional. Loading: N/A — series management is a small, occasional action. Error: a failed occurrence generation is retried and, if it keeps failing, surfaces to the Pro as a flagged gap rather than a silently missed appointment. Offline-degraded: N/A — requires connectivity for correctness, consistent with the rest of scheduling.

**Validation & Limits:** A series repeats every 1 to 12 weeks and generates occurrences no further ahead than the Pro's booking horizon; each generated occurrence is still subject to the same slot validation as any booking (FEAT-03); an occurrence whose deposit is unpaid by its cancellation cut-off is released and both the client and the Pro are told. [MODIFIED: "reasonable minimum/maximum" made testable and the unpaid-occurrence rule added from the value-flow walk]

**Access:** Own-only for the Client (their own series). The Pro has Full access to see and manage series tied to their own schedule, including setting one up for a client at the chair through Pro Booking Management (FEAT-30). Platform Operator (Support) has View-only access. [MODIFIED: "Full/View" resolved to Full to agree with the Access Matrix "Recurring Appointments" column]

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

**Rationale:** BRIEF.md's Open Questions asks directly whether the balance "should be payable in the app, or stay in person." As Visionary judgment: the brief's default flow ("the balance is due at the appointment") works fine without this, so it is not required for MVP, but it is a natural, low-risk enhancement once deposit payment (FEAT-07) is proven. Phased to v1. This answers BRIEF.md's balance open question for MVP: the balance stays in person at MVP and becomes optionally payable in-app at v1.

**Connected Entities:** Balance Payment (create), Deposit Transaction (read), Booking (read, update — balance payment status) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- Pay the remaining balance in-app at any point before or at the appointment
- See a running record of deposit paid vs. balance remaining
- Balance goes straight to the Pro's payout account (FEAT-28) with no platform cut, and is refunded in full if the appointment is later cancelled by either party — a balance is never subject to forfeiture [AUDIT-ADDED: 1 -- value-flow walk: the balance's destination and its exit path on cancellation were unspecified]

**Primary Flows & Alternates:**
- Happy path: client opens their booking and pays the balance in-app; the Pro's dashboard reflects "fully paid" instead of "balance due."
- Alternate: client chooses to pay in person instead — this remains the default, unaffected experience; in-app balance payment is purely additive, never required.
- Alternate: an in-app balance payment fails; the booking remains marked "balance due" exactly as if the client had never attempted it, with no partial or ambiguous state.

**States:** Empty: N/A — only appears against an existing confirmed booking with a balance due. Loading: standard payment-processing indicator. Error: a specific decline message, matching the deposit payment feature's pattern. Offline-degraded: requires connectivity, consistent with all payment actions.

**Validation & Limits:** The balance amount is fixed by the service price minus the deposit already paid and cannot be altered by the client.

**Access:** Own-only for the Client (pays their own balance). The Pro has Full access to the resulting paid/unpaid status on their own bookings and refunds a paid balance through Pro Booking Management (FEAT-30). Platform Operator (Support) has View-only access, never to card data. [MODIFIED: Pro access aligned with the Access Matrix "Booking & Payment" column]

**Communications:** A payment confirmation on successful balance payment.

**Data Notes:** Captured: balance payment outcome. Displayed: updated balance-due status to both parties. Derived: balance amount, from Service price minus Deposit Transaction amount.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-07) and Payout Account Connection & Payout Visibility (FEAT-28); updates Pro Daily Schedule Dashboard (FEAT-12); refunded through Pro Booking Management (FEAT-30). [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** balance_payment_attempted, balance_payment_succeeded, balance_payment_failed.

### Tipping at Checkout

**ID:** FEAT-23

**Description:** A client can optionally add a tip when paying in-app (at deposit or, once available, at balance payment), which passes through to the Pro.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** BRIEF.md's Open Questions asks directly "where does tipping fit, if anywhere?" As Visionary judgment: tipping has no bearing on the core no-show/deposit problem the product exists to solve, and depends on in-app balance payment (FEAT-22) to be meaningful (tipping on a deposit alone is an unusual pattern). Phased to Later. [INFERRED: carried from Visionary draft]

**Connected Entities:** Balance Payment (update — tip amount), Booking (read) [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Key Capabilities:**
- Add an optional tip amount at in-app payment time
- See tips reflected in the Pro's own payment records

**Primary Flows & Alternates:**
- Happy path: client is offered an optional tip at balance payment; they choose an amount (or none) and complete payment.
- Alternate: client skips tipping entirely — this must never feel like a required step or block the underlying payment.
- Alternate: client pays their balance in person instead, bypassing in-app tipping entirely; this remains a fully normal path.

**States:** Empty: N/A — only appears within an existing payment flow. Loading: N/A — part of the standard payment flow's own states. Error: N/A — tipping failure is treated as part of the underlying payment's own error handling, never a separate failure mode. Offline-degraded: N/A — inherits the payment flow's connectivity requirement.

**Validation & Limits:** Tip amount, when given, must be a non-negative value; never pre-selected to a default that could feel presumptive; the whole tip goes to the Pro's payout account with no platform cut, and is refunded with the balance if the appointment is cancelled. [AUDIT-ADDED: 1 -- value-flow walk: the tip's destination and refund path were unspecified]

**Access:** Own-only for the Client (chooses their own tip). The Pro sees tips received on their own bookings as part of their Full "Booking & Payment" access, but a tip amount is only ever set by the client. Platform Operator (Support) has View-only access, never to card data.

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

**Rationale:** BRIEF.md's Scale & Non-Functional Expectations states a Pro may have "100–500 clients," a volume where an unfiltered list becomes genuinely unwieldy. Not required at launch (a new Pro starts with very few clients), so phased to v1 rather than MVP. [INFERRED: carried from Visionary draft]

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

**Rationale:** BRIEF.md's Success Criteria centers on the Pro noticing outcomes ("I haven't had an unpaid no-show since I switched"); a simple summary makes that outcome visible rather than only felt anecdotally, which supports the brief's referral-driven go-to-market ("most new pros arrive because another pro told them about it"). Not required for the core loop, so phased to v1. [INFERRED: carried from Visionary draft]

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

**Access:** The Pro has View access to their own figures only — never compared against or visible to any other pro. Platform Operator (Support) has View-only access. Clients have no access.

**Communications:** N/A — a self-initiated view with no notification trigger.

**Data Notes:** Displayed: aggregated figures. Derived: entirely — every figure here is computed from existing Booking, Deposit Transaction, and Service records; nothing is captured directly.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-07) and No-Show Marking & Deposit Forfeiture (FEAT-11) for the "saved" figure, and on Payout Account Connection & Payout Visibility (FEAT-28) for amounts received. [MODIFIED: cross-references added for the audit-added features this one now connects to]

**Signals:** insights_viewed, insights_period_changed.

### WhatsApp Reminders

**ID:** FEAT-26

**Description:** Confirmations and reminders can optionally be sent over WhatsApp instead of, or alongside, SMS, for clients and pros who prefer it.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** BRIEF.md's Ecosystem & Integrations states this explicitly: "WhatsApp is a nice-to-have later, not v1." Phased to Later exactly as the brief specifies, and it slots into the existing Automated Booking Messaging (FEAT-08) mechanism as an additional channel rather than a new capability. [INFERRED: carried from Visionary draft]

**Connected Entities:** Message (create — additional channel), Messaging Consent (read)

**Key Capabilities:**
- Opt for WhatsApp as the delivery channel for confirmations and reminders
- Fall back to SMS or email automatically if WhatsApp delivery is unavailable

**Primary Flows & Alternates:**
- Happy path: client indicates a WhatsApp preference; confirmations and reminders deliver there instead of SMS.
- Alternate: WhatsApp delivery fails or is unavailable for that number; the system falls back to SMS or email per the client's existing consent state, exactly as FEAT-08 already does for SMS failures.

**States:** Empty: N/A — inherits Automated Booking Messaging's own states as an additional channel. Loading: N/A. Error: falls back per the alternate flow above, never silently dropped. Offline-degraded: N/A — sending is done by the product itself, not on either person's device. [MODIFIED: wording made implementation-neutral per the functional-language rule]

**Validation & Limits:** Requires the same explicit consent discipline as texting (FEAT-14) before use — consent is channel-aware, not a blanket "texting is fine" assumption.

**Access:** Clients opt in for their own messages (Own-only). The Pro sees delivery channel/status like any other message (View, via FEAT-16). Platform Operator (Support) has View-only access.

**Communications:** This feature is itself an additional communications channel for the messages FEAT-08 already sends.

**Data Notes:** Captured: channel preference. Displayed: delivery channel/status. Derived: none.

**Interactions:** Extends Automated Booking Messaging (FEAT-08); depends on Messaging Consent Management (FEAT-14).

**Signals:** whatsapp_channel_selected, whatsapp_delivery_failed_fallback_used.

## Feature Interaction Summary

[MODIFIED: rows updated and FEAT-27 to FEAT-30 added to reflect the audit-added features and their dependencies]

| Feature | Depends On |
|---------|------------|
| FEAT-01 Service & Pricing Management | None |
| FEAT-02 Availability & Working Hours Setup | None |
| FEAT-03 Real-Time Slot Availability Engine | FEAT-02, FEAT-04, FEAT-17, FEAT-21 |
| FEAT-04 Two-Way Calendar Sync | None |
| FEAT-05 Public Booking Page & Booking Flow | FEAT-01, FEAT-03, FEAT-06, FEAT-07, FEAT-09, FEAT-27 |
| FEAT-06 Client Booking Identity | FEAT-05 (reads Client created there) |
| FEAT-07 Deposit Payment at Booking | FEAT-01, FEAT-05, FEAT-28 |
| FEAT-08 Automated Booking Messaging | FEAT-05, FEAT-07, FEAT-14, FEAT-27 |
| FEAT-09 Cancellation & No-Show Policy Engine | None |
| FEAT-10 Client-Initiated Cancel/Reschedule | FEAT-03, FEAT-06, FEAT-09 |
| FEAT-11 No-Show Marking & Deposit Forfeiture | FEAT-07, FEAT-09 |
| FEAT-12 Pro Daily Schedule Dashboard | FEAT-05, FEAT-07, FEAT-08, FEAT-13, FEAT-29 |
| FEAT-13 Client Record Management | FEAT-05 |
| FEAT-14 Messaging Consent Management | FEAT-05 |
| FEAT-15 Pro Onboarding & Setup Wizard | FEAT-28, FEAT-29 |
| FEAT-16 Booking & Payment Activity Record | FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11 |
| FEAT-17 Manual Time Blocking | FEAT-30 (to resolve conflicting bookings) |
| FEAT-18 Pro Subscription Billing & Account Management | FEAT-15 |
| FEAT-19 Platform Support Read-Only Access | FEAT-15, FEAT-12, FEAT-16, FEAT-18 |
| FEAT-20 Waitlist for Cancelled Slots | FEAT-03, FEAT-10 |
| FEAT-21 Recurring/Standing Appointments | FEAT-03, FEAT-05 |
| FEAT-22 In-App Balance Payment | FEAT-07, FEAT-28 |
| FEAT-23 Tipping at Checkout | FEAT-22 |
| FEAT-24 Client List Search & Filter | FEAT-13 |
| FEAT-25 Booking & Revenue Insights | FEAT-07, FEAT-11, FEAT-28 |
| FEAT-26 WhatsApp Reminders | FEAT-08, FEAT-14 |
| FEAT-27 Pro Profile & Booking Page Settings | None |
| FEAT-28 Payout Account Connection & Payout Visibility | None |
| FEAT-29 Pro Sign-In & Account Lifecycle | None |
| FEAT-30 Pro Booking Management | FEAT-03, FEAT-07, FEAT-09, FEAT-28 |


## How the Features Depend on Each Other

The dependency map: feature table, navigation connections, cross-feature business rules (XBR) and external touchpoints.

## Features

| Number | Slug | Name | Priority | Phase | Type | Depends On | Depended On By |
|--------|------|------|----------|-------|------|------------|----------------|
| FEAT-01 | service-pricing-management | Service & Pricing Management | Core | MVP | User-Facing | -- | FEAT-05, FEAT-07 |
| FEAT-02 | availability-working-hours-setup | Availability & Working Hours Setup | Core | MVP | User-Facing | -- | FEAT-03 |
| FEAT-03 | real-time-slot-availability-engine | Real-Time Slot Availability Engine | Core | MVP | Platform | FEAT-02, FEAT-04, FEAT-17, FEAT-21 | FEAT-05, FEAT-10, FEAT-20, FEAT-21, FEAT-30 |
| FEAT-04 | two-way-calendar-sync | Two-Way Calendar Sync | Core | MVP | Platform | -- | FEAT-03 |
| FEAT-05 | public-booking-page-booking-flow | Public Booking Page & Booking Flow | Core | MVP | User-Facing | FEAT-01, FEAT-03, FEAT-06, FEAT-07, FEAT-09, FEAT-27 | FEAT-06, FEAT-07, FEAT-08, FEAT-12, FEAT-13, FEAT-14, FEAT-16, FEAT-21 |
| FEAT-06 | client-booking-identity | Client Booking Identity | Core | MVP | Platform | FEAT-05 | FEAT-05, FEAT-10 |
| FEAT-07 | deposit-payment-at-booking | Deposit Payment at Booking | Core | MVP | User-Facing | FEAT-01, FEAT-05, FEAT-28 | FEAT-05, FEAT-08, FEAT-11, FEAT-12, FEAT-16, FEAT-22, FEAT-25, FEAT-30 |
| FEAT-08 | automated-booking-messaging | Automated Booking Messaging | Core | MVP | User-Facing | FEAT-05, FEAT-07, FEAT-14, FEAT-27 | FEAT-12, FEAT-16, FEAT-26 |
| FEAT-09 | cancellation-no-show-policy-engine | Cancellation & No-Show Policy Engine | Core | MVP | Platform | -- | FEAT-05, FEAT-10, FEAT-11, FEAT-30 |
| FEAT-10 | client-initiated-cancel-reschedule | Client-Initiated Cancel/Reschedule | Core | MVP | User-Facing | FEAT-03, FEAT-06, FEAT-09 | FEAT-16, FEAT-20 |
| FEAT-11 | no-show-marking-deposit-forfeiture | No-Show Marking & Deposit Forfeiture | Core | MVP | User-Facing | FEAT-07, FEAT-09 | FEAT-16, FEAT-25 |
| FEAT-12 | pro-daily-schedule-dashboard | Pro Daily Schedule Dashboard | Core | MVP | User-Facing | FEAT-05, FEAT-07, FEAT-08, FEAT-13, FEAT-29 | FEAT-19 |
| FEAT-13 | client-record-management | Client Record Management | Important | MVP | User-Facing | FEAT-05 | FEAT-12, FEAT-24 |
| FEAT-14 | messaging-consent-management | Messaging Consent Management | Important | MVP | Platform | FEAT-05 | FEAT-08, FEAT-26 |
| FEAT-15 | pro-onboarding-setup-wizard | Pro Onboarding & Setup Wizard | Important | MVP | Lifecycle | FEAT-28, FEAT-29 | FEAT-18, FEAT-19 |
| FEAT-16 | booking-payment-activity-record | Booking & Payment Activity Record | Important | MVP | Platform | FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11 | FEAT-19 |
| FEAT-17 | manual-time-blocking | Manual Time Blocking | Important | MVP | User-Facing | FEAT-30 | FEAT-03 |
| FEAT-18 | pro-subscription-billing-account-management | Pro Subscription Billing & Account Management | Important | MVP | Lifecycle | FEAT-15 | FEAT-19 |
| FEAT-19 | platform-support-read-only-access | Platform Support Read-Only Access | Important | MVP | Platform | FEAT-12, FEAT-15, FEAT-16, FEAT-18 | -- |
| FEAT-20 | waitlist-for-cancelled-slots | Waitlist for Cancelled Slots | Nice-to-Have | v1 | User-Facing | FEAT-03, FEAT-10 | -- |
| FEAT-21 | recurring-standing-appointments | Recurring/Standing Appointments | Nice-to-Have | v1 | User-Facing | FEAT-03, FEAT-05 | FEAT-03 |
| FEAT-22 | in-app-balance-payment | In-App Balance Payment | Nice-to-Have | v1 | User-Facing | FEAT-07, FEAT-28 | FEAT-23 |
| FEAT-23 | tipping-at-checkout | Tipping at Checkout | Nice-to-Have | Later | User-Facing | FEAT-22 | -- |
| FEAT-24 | client-list-search-filter | Client List Search & Filter | Nice-to-Have | v1 | User-Facing | FEAT-13 | -- |
| FEAT-25 | booking-revenue-insights | Booking & Revenue Insights | Nice-to-Have | v1 | User-Facing | FEAT-07, FEAT-11, FEAT-28 | -- |
| FEAT-26 | whatsapp-reminders | WhatsApp Reminders | Nice-to-Have | Later | User-Facing | FEAT-08, FEAT-14 | -- |
| FEAT-27 | pro-profile-booking-page-settings | Pro Profile & Booking Page Settings | Important | MVP | User-Facing | -- | FEAT-05, FEAT-08 |
| FEAT-28 | payout-account-connection-payout-visibility | Payout Account Connection & Payout Visibility | Core | MVP | User-Facing | -- | FEAT-07, FEAT-15, FEAT-22, FEAT-25, FEAT-30 |
| FEAT-29 | pro-sign-in-account-lifecycle | Pro Sign-In & Account Lifecycle | Important | MVP | Lifecycle | -- | FEAT-12, FEAT-15 |
| FEAT-30 | pro-booking-management | Pro Booking Management | Core | MVP | User-Facing | FEAT-03, FEAT-07, FEAT-09, FEAT-28 | FEAT-17 |

Mutual pairs carried from Stage 2 (FEAT-05/FEAT-06, FEAT-05/FEAT-07, FEAT-03/FEAT-21, FEAT-17/FEAT-30) are data-flow loops, not build-order cycles: in each pair one feature supplies data or a hand-off the other consumes (for example, FEAT-17 hands conflicting bookings to FEAT-30, while FEAT-30's bulk cancel is invoked from a FEAT-17 block).


## Navigation Connections

| From Feature | From Context | To Feature | To Context | Trigger |
|-------------|-------------|------------|-----------|---------|
| FEAT-15 | setup step: account | FEAT-29 | sign-in creation (email, mobile, one-time code) | Pro starts setup |
| FEAT-15 | setup step: profile | FEAT-27 | display name, photo, studio location | Pro completes sign-in step |
| FEAT-15 | setup step: services | FEAT-01 | add first service and deposit rule | Pro continues setup |
| FEAT-15 | setup step: hours | FEAT-02 | working hours and buffer | Pro continues setup |
| FEAT-15 | setup step: policy | FEAT-09 | cancellation window (default offered) | Pro continues setup |
| FEAT-15 | setup step: getting paid | FEAT-28 | payout account connection (processor verification) | Pro continues setup |
| FEAT-15 | setup step: calendar | FEAT-04 | connect personal calendar (skippable) | Pro continues setup |
| FEAT-15 | setup step: subscription | FEAT-18 | subscribe with card | Pro continues setup |
| FEAT-15 | link is live | FEAT-05 | booking page preview as a client sees it | Pro taps preview |
| FEAT-05 | service list | FEAT-03 | live slot list for the chosen service | Client picks a service |
| FEAT-05 | details and policy acknowledgment | FEAT-07 | deposit payment step | Client ticks policy agreement |
| FEAT-07 | payment result | FEAT-05 | on-screen confirmation | Deposit succeeds |
| FEAT-07 | decline / hold expired | FEAT-03 | live slot list | Hold expires before retry |
| FEAT-05 | fully booked service | FEAT-20 | join waitlist (v1) | Client taps join waitlist |
| FEAT-05 | returning client | FEAT-06 | phone recognition | Client enters a known phone number |
| FEAT-08 | confirmation / reminder message | FEAT-06 | booking opened via manage link | Client taps manage link |
| FEAT-08 | reminder message | FEAT-10 | reschedule flow for that booking | Client taps "I need to reschedule" |
| FEAT-08 | any client message | FEAT-14 | texting opt-out | Client taps opt-out link or replies STOP |
| FEAT-06 | my bookings | FEAT-10 | cancel or reschedule a booking | Client chooses cancel or reschedule |
| FEAT-06 | my bookings | FEAT-14 | texting consent and email preferences | Client opens preferences |
| FEAT-06 | my bookings | FEAT-20 | leave a waitlist (v1) | Client taps leave |
| FEAT-06 | my bookings | FEAT-22 | pay balance in-app (v1) | Client taps pay balance |
| FEAT-10 | reschedule | FEAT-03 | live slot list | Client picks a new time |
| FEAT-20 | waitlist notification | FEAT-05 | booking flow for the opened slot | Client taps claim link |
| FEAT-29 | sign-in | FEAT-12 | today's schedule | Pro signs in / opens the app |
| FEAT-12 | booking row | FEAT-13 | client record and private note | Pro taps a client |
| FEAT-12 | booking row | FEAT-11 | mark no-show / undo | Pro taps no-show |
| FEAT-12 | booking row | FEAT-30 | cancel, reschedule, refund, book next visit | Pro taps a booking action |
| FEAT-12 | schedule view | FEAT-17 | add a time block | Pro taps block time |
| FEAT-12 | past bookings | FEAT-16 | booking activity timeline | Pro opens a past booking's history |
| FEAT-12 | attention list | FEAT-04 | reconnect calendar | Pro taps reconnect banner |
| FEAT-12 | attention list | FEAT-28 | refund in progress / payout action required | Pro taps money attention item |
| FEAT-12 | attention list | FEAT-16 | card-issuer dispute flag and summary download | Pro taps dispute flag |
| FEAT-12 | navigation | FEAT-28 | money list | Pro opens money list |
| FEAT-12 | navigation | FEAT-27 | profile and booking page settings | Pro opens settings |
| FEAT-12 | navigation | FEAT-25 | insights (v1) | Pro opens insights |
| FEAT-11 | no-show prompt | FEAT-30 | goodwill refund instead | Pro chooses refund as goodwill |
| FEAT-16 | booking timeline | FEAT-30 | goodwill refund | Pro decides to refund |
| FEAT-17 | block over existing bookings | FEAT-30 | bulk cancel or reschedule | Pro chooses to cancel affected bookings |
| FEAT-13 | delete client with upcoming booking | FEAT-30 | cancel with full refund | Pro confirms deletion |
| FEAT-13 | client list | FEAT-24 | search and filter (v1) | Pro types in search |
| FEAT-29 | close account with upcoming bookings | FEAT-30 | bulk cancel with full refunds | Pro requests closure |
| FEAT-29 | account settings | FEAT-18 | subscription status and payment method | Pro opens billing |
| FEAT-27 | settings | FEAT-05 | booking page preview | Pro taps preview |
| FEAT-27 | help request | FEAT-19 | read-only support view of the account | Support opens the Pro's account after a help request |
| FEAT-19 | support view | FEAT-16 | booking timeline (read-only) | Support opens a disputed booking |


## Cross-Feature Business Rules

| Rule ID | Description | Affected Features | Authority |
|---------|-------------|-------------------|-----------|
| XBR-01 | A time is offered, held or booked only if it passes the live slot check (full duration plus buffer inside an open window, no conflicting booking, block, recurring reservation or personal-calendar busy time); the first client to complete payment wins a contested slot and the other sees a plain "just taken" message, never a payment error. Applies to every booking path. | FEAT-03, FEAT-05, FEAT-07, FEAT-10, FEAT-20, FEAT-21, FEAT-30 | FEAT-03 (owns slot truth and holds) |
| XBR-02 | Slot holds are time-limited and release automatically: checkout hold of a few minutes; Pro-created deposit request holds up to 24 hours or until 2 hours before the appointment; waitlist claim window 30 minutes; an unpaid recurring occurrence is released at its cancellation cut-off. | FEAT-03, FEAT-05, FEAT-07, FEAT-20, FEAT-21, FEAT-30 | FEAT-03 (owns slot holds) |
| XBR-03 | Minimum booking notice and booking horizon limit every client-facing booking path; the Pro alone may book inside notice or beyond horizon when booking a client in or rescheduling. | FEAT-02, FEAT-03, FEAT-05, FEAT-10, FEAT-20, FEAT-21, FEAT-30 | FEAT-02 (owns notice and horizon settings) |
| XBR-04 | Service edits and archiving apply to future bookings only; confirmed bookings keep the price, duration and deposit agreed at booking. | FEAT-01, FEAT-05, FEAT-07, FEAT-12 | FEAT-01 (owns Service) |
| XBR-05 | The deposit is computed once, exactly, from the service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking; card data is never held by the product — each deposit is paid fresh. | FEAT-01, FEAT-05, FEAT-07, FEAT-21, FEAT-30 | FEAT-07 (owns deposit capture) |
| XBR-06 | No deposit can be taken, and the booking link cannot go live, unless the Pro's payout account is active. | FEAT-28, FEAT-07, FEAT-05, FEAT-15 | FEAT-28 (owns payout account status) |
| XBR-07 | Money never rests with the platform: Chairtime's fee on deposits, balances and tips is always zero; money goes to the Pro's payout account and the only deduction shown is the processor's card fee. | FEAT-07, FEAT-18, FEAT-22, FEAT-23, FEAT-28 | FEAT-28 (owns money visibility) |
| XBR-08 | Every booking is governed by the cancellation policy version shown and acknowledged at booking; policy edits never change existing bookings. | FEAT-09, FEAT-05, FEAT-10, FEAT-11, FEAT-16, FEAT-30 | FEAT-09 (owns policy versions) |
| XBR-09 | Deposit outcomes are binary and symmetric: client cancellation outside the window = full refund; inside the window or no-show = kept; any Pro cancellation = full refund; client reschedule outside the window carries the deposit over; inside the window = late cancellation plus a new deposit, shown before confirming; a Pro-made reschedule never exposes the client to the window. | FEAT-09, FEAT-10, FEAT-11, FEAT-30 | FEAT-09 (owns deposit outcome rules) |
| XBR-10 | Refunds are always full and happen at most once per deposit; a refund that cannot complete is retried automatically, flagged to the Pro and shown to the client as in progress, never dropped. | FEAT-09, FEAT-28, FEAT-30, FEAT-12, FEAT-08 | FEAT-09 (owns refund triggering) |
| XBR-11 | Setup changes never silently cancel a confirmed booking: changed hours, new time blocks, archived services, a Pro pause and a subscription lapse all leave existing bookings honored; conflicts are flagged on the dashboard and resolved only by an explicit Pro choice through Pro Booking Management. | FEAT-01, FEAT-02, FEAT-17, FEAT-18, FEAT-27, FEAT-12, FEAT-30 | FEAT-30 (owns Pro-side changes to bookings) |
| XBR-12 | Booking outcome windows: a booking can be marked no-show or completed only after its start time; it auto-completes 7 days after the appointment; a no-show mark can be undone for 24 hours; a completed or no-show booking can no longer be cancelled or rescheduled; a goodwill refund is available until completion. | FEAT-11, FEAT-12, FEAT-10, FEAT-30 | FEAT-12 (owns completion and auto-completion) |
| XBR-13 | The Pro's personal calendar mirrors Chairtime: every booking created, rescheduled or cancelled by either party is written, moved or removed there; personal busy time blocks availability; if sync lapses, availability falls back to Chairtime data with reduced confidence shown to the Pro only, never to clients. | FEAT-04, FEAT-03, FEAT-05, FEAT-10, FEAT-12, FEAT-30 | FEAT-04 (owns the calendar connection) |
| XBR-14 | A paused account (subscription lapse after the 7-day grace, or a Pro-chosen pause) takes no new bookings or deposits, while existing bookings keep their reminders, refunds and client self-service unchanged. | FEAT-27, FEAT-18, FEAT-05, FEAT-08, FEAT-10 | FEAT-27 (owns the Pro Account pause state) |
| XBR-15 | No text is sent without active texting consent for that client and Pro; otherwise email is used; a revoke is honored on the very next message; a changed phone number requires fresh consent. | FEAT-14, FEAT-05, FEAT-06, FEAT-08, FEAT-13, FEAT-20, FEAT-21, FEAT-26, FEAT-30 | FEAT-14 (owns consent state) |
| XBR-16 | Automatic reminders go out only between roughly 8am and 9pm in the Pro's timezone; confirmations arrive within about a minute of payment; a booking made after its reminder point gets no separate reminder. | FEAT-08, FEAT-05, FEAT-07, FEAT-27 | FEAT-08 (owns message timing) |
| XBR-17 | A failed text is retried once, then sent by email, and the delivery gap is flagged on the Pro's dashboard and recorded in the booking timeline — never silently dropped. | FEAT-08, FEAT-12, FEAT-16 | FEAT-08 (owns delivery) |
| XBR-18 | Client identity is a phone number with one Pro only; access links open only that client's bookings with that Pro; on-demand links are single-use for 30 minutes; booking-specific links stop working once the appointment passes; a Pro reschedule issues a fresh manage link. | FEAT-06, FEAT-05, FEAT-08, FEAT-10, FEAT-13, FEAT-30 | FEAT-06 (owns client access) |
| XBR-19 | Client deletion: refused while an upcoming booking exists (the Pro is offered cancel-with-full-refund); removes contact details, notes and consent; financial and timeline records are retained only in de-identified form; a later booking creates a new record. | FEAT-13, FEAT-30, FEAT-16, FEAT-14, FEAT-05 | FEAT-13 (owns Client deletion) |
| XBR-20 | Account closure: upcoming bookings must first be cancelled with full refunds; the subscription is cancelled and the booking page taken down; data is deleted after a 30-day cooling-off period, keeping only legally required de-identified financial records. | FEAT-29, FEAT-30, FEAT-18, FEAT-05, FEAT-27 | FEAT-29 (owns account lifecycle) |
| XBR-21 | Every booking, payment, messaging and support-view event is written to an append-only, immutable activity record that no role can edit. | FEAT-16, FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11, FEAT-19, FEAT-30 | FEAT-16 (owns the activity record) |
| XBR-22 | A card-issuer dispute flags the booking on the dashboard, notifies the Pro, marks the deposit Disputed, and makes the plain timeline summary available to submit as evidence; Chairtime never rules on the dispute. | FEAT-16, FEAT-12, FEAT-08, FEAT-28 | FEAT-16 (owns dispute evidence) |
| XBR-23 | Balance due = service price − deposit − any in-app balance payment; a paid balance (and any tip) is never forfeited and is refunded in full if either party cancels. | FEAT-22, FEAT-23, FEAT-12, FEAT-30, FEAT-07 | FEAT-22 (owns balance payment) |
| XBR-24 | Support access is read-only, one account at a time, used only after a Pro's help request, never shows private client notes, bank or identity details or sign-in codes, and every view is logged in the Pro's visible account activity. | FEAT-19, FEAT-16, FEAT-13, FEAT-27, FEAT-28, FEAT-29 | FEAT-19 (owns support access) |
| XBR-25 | Timezone and currency are per-account settings: every slot and appointment time is computed and shown in the Pro's timezone (labeled); currency is fixed once the first deposit is taken; the payout account must match the account's country and currency. | FEAT-27, FEAT-01, FEAT-02, FEAT-03, FEAT-07, FEAT-08, FEAT-12, FEAT-28 | FEAT-27 (owns timezone and currency) |
| XBR-26 | The booking link goes live only when sign-in, display name and studio location, one service, working hours, deposit rule, cancellation policy, an active payout account and an active subscription are all in place; calendar connection is the only optional step. | FEAT-15, FEAT-29, FEAT-27, FEAT-01, FEAT-02, FEAT-09, FEAT-28, FEAT-18, FEAT-04 | FEAT-15 (owns go-live) |
| XBR-27 | A renamed booking link keeps forwarding from the old name for at least 12 months; a closed, paused-to-closure or mistyped link shows a plain "this booking page isn't available" message, never another Pro's page. | FEAT-27, FEAT-05, FEAT-29 | FEAT-27 (owns the booking link name) |
| XBR-28 | A slot freed by a cancellation becomes publicly bookable immediately; from v1, matching waitlisted clients are notified first and have a 30-minute priority window before it returns to general availability. | FEAT-10, FEAT-30, FEAT-20, FEAT-03, FEAT-08 | FEAT-20 (owns waitlist priority) |
| XBR-29 | Every Pro-facing screen requires a signed-in Pro; anyone else is sent to the Pro sign-in screen, and a failed sign-in never reveals whether an account exists. | FEAT-29, FEAT-12, FEAT-13, FEAT-27, FEAT-28, FEAT-30, FEAT-01, FEAT-02, FEAT-17 | FEAT-29 (owns Pro sign-in) |


## External Touchpoints

Traced from `## Dependencies` in assumptions-constraints.md. ASMP-34 (a per-pro record store) is the product's own persistence, not an external capability, so it has no touchpoint row; Stage 4 addresses it directly. The Integration Specs column was completed batch by batch as each feature's Brief was validated, and the final analysis batch confirmed full coverage in both directions: every row below has at least one covering Integration spec, and every Integration spec in every Brief is cited here. The WhatsApp messaging row was added from a validated Brief's Integration spec (FEAT-26.SPEC-002); its citation trail is ASMP-32 and BRIEF.md's Ecosystem & Integrations.

| Capability Category | Features Involved | Integration Specs |
|---------------------|-------------------|-------------------|
| Payment processing — client card charges and refunds (deposits; from v1 balances; Later tips) (ASMP-31) | FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23 | FEAT-07.SPEC-005 (deposit card authorization and capture, processor fee reporting, zero platform fee), FEAT-09.SPEC-005 (automatic full deposit refund on client cancellation outside the window and every Pro cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry), FEAT-30.SPEC-011 (goodwill refunds and the per-booking refund set of a Pro bulk cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry; a single Pro cancellation's refund is executed by FEAT-09.SPEC-005), FEAT-22.SPEC-005 (in-app balance card authorization and capture with zero platform fee, and the outbound full refund of a paid balance when either party cancels, per XBR-23); FEAT-23's optional tip is charged and refunded as part of that same balance payment through FEAT-22.SPEC-005 and needs no Integration spec of its own |
| Payment processing — connected payout accounts with identity and bank verification (ASMP-31) | FEAT-28, FEAT-15, FEAT-07, FEAT-22, FEAT-23 | FEAT-07.SPEC-005 (routes each captured deposit to the Pro's connected payout account), FEAT-28.SPEC-006 (hand-off into the processor's own identity and bank verification, action-required resolution, and inbound account-status, payout and processor-fee reporting; FEAT-15 reaches it through its getting-paid step), FEAT-22.SPEC-005 (routes each captured balance, including any FEAT-23 tip, to the Pro's connected payout account with zero platform fee) |
| Payment processing — card-issuer dispute notifications (ASMP-31) | FEAT-16, FEAT-12 | FEAT-16.SPEC-003 (inbound card-issuer dispute notice: flags the booking, sets the Deposit Transaction's Disputed overlay without erasing its outcome, and records the dispute event; FEAT-12 consumes the flag through FEAT-12.SPEC-005 and needs no Integration spec of its own) |
| Payment processing — Pro subscription billing (ASMP-31) | FEAT-18, FEAT-15 | FEAT-18.SPEC-006 (subscribe, payment-method update and cancel requests, including cancellation invoked by FEAT-29 account closure, plus inbound renewal outcomes; FEAT-15 reaches it through its subscription setup step) |
| Transactional text messaging (ASMP-32) | FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-20, FEAT-21, FEAT-26 | FEAT-08.SPEC-012 (text send and delivery-status reporting for every product text, including FEAT-06 access links, the FEAT-14 opt-out confirmation and inbound STOP replies consumed by FEAT-14.SPEC-004, and the FEAT-30 Pro-action notices and deposit requests); FEAT-15's go-live welcome confirmation (FEAT-15.SPEC-008) and FEAT-18's billing notices (FEAT-18.SPEC-007) are also delivered through FEAT-08.SPEC-012; FEAT-20's waitlist opening and expiry notices (FEAT-20.SPEC-008, FEAT-20.SPEC-009) and FEAT-29's sign-in code, new-device alert, contact-change and closure/deletion notices (FEAT-29.SPEC-014 to FEAT-29.SPEC-017) are also delivered through FEAT-08.SPEC-012; FEAT-21's occurrence-generated, occurrence time-change and occurrence deposit-link/release notices (FEAT-21.SPEC-007 to FEAT-21.SPEC-009) are also delivered through FEAT-08.SPEC-012; FEAT-26's WhatsApp fallback (FEAT-26.SPEC-003) re-sends a failed or unavailable WhatsApp message as a text through FEAT-08.SPEC-012 when the client has texting consent |
| Transactional email (fallback channel) (ASMP-32) | FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-18, FEAT-21, FEAT-26 | FEAT-08.SPEC-013 (fallback and consent-declined email send and delivery-status reporting, including FEAT-06 access links, FEAT-14 fallback routing and the FEAT-30 Pro-action notices and deposit requests for clients without texting consent); FEAT-18's billing notices (FEAT-18.SPEC-007) and FEAT-15's go-live welcome confirmation (FEAT-15.SPEC-008) are also delivered through FEAT-08.SPEC-013; FEAT-29's notices (FEAT-29.SPEC-014 to FEAT-29.SPEC-017) are delivered through FEAT-08.SPEC-013 as the email fallback, as are FEAT-20's waitlist notices (FEAT-20.SPEC-008, FEAT-20.SPEC-009) and FEAT-21's occurrence notices (FEAT-21.SPEC-007 to FEAT-21.SPEC-009) for clients without texting consent; FEAT-26's WhatsApp fallback (FEAT-26.SPEC-003) re-sends through FEAT-08.SPEC-013 when the client has no texting consent |
| Transactional WhatsApp messaging — optional client channel for confirmations, reminders and change notices, from Later (ASMP-32; BRIEF.md Ecosystem & Integrations: "WhatsApp is a nice-to-have later, not v1") | FEAT-26, FEAT-08, FEAT-14 | FEAT-26.SPEC-002 (WhatsApp send and inbound delivery-status reporting for FEAT-08's confirmation, reminder and change-notice content when the client has chosen WhatsApp and FEAT-26.SPEC-004 finds the channel eligible under channel-aware Messaging Consent owned by FEAT-14; failed or unavailable sends fall back through FEAT-26.SPEC-003 to FEAT-08.SPEC-012/FEAT-08.SPEC-013) |
| Calendar sync — reading busy time from and writing bookings to a Pro's personal calendar (ASMP-33) | FEAT-04, FEAT-03, FEAT-15 | FEAT-04.SPEC-003 (connection handshake, busy-time pull, booking write/move/remove), FEAT-03.SPEC-006 (busy-time consumption and degraded mode) |
| File storage — Pro profile photos (ASMP-35) | FEAT-27, FEAT-05 | FEAT-27.SPEC-012 (stores, replaces and serves the Pro's profile photo within the size/format limits; FEAT-05 reads the served photo on the booking page and needs no Integration spec of its own; the page still works without a photo) |


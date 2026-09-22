---
document_type: product-features
produced_by: product-synthesizer
variant: final
status: final
created: 2026-09-21
synthesis_check: passed (2 fixes applied)
---

# Product Features

## Summary

This product includes 29 features: 17 Core, 4 Important, 8 Nice-to-Have. By phase: 19 MVP, 2 v1, 8 Later. By type: 20 User-Facing, 6 Platform, 3 Lifecycle. The product manages 13 domain entities. Core features close the end-to-end loop the brief describes — a client books and pays a deposit against a genuinely free slot, and the Pro runs her day off a clean, trustworthy list — plus the account, calendar-sync, and business-configuration scaffolding that loop depends on. Important features remove the two gaps that would otherwise resurface the brief's original pain points (no dispute record, no way to leave cleanly). Nice-to-Have features answer the brief's own open questions (waitlist, tipping, in-app balance payment, recurring bookings) plus two domain-standard conveniences (import, export) — all deferred to v1 or Later rather than dropped, so nothing the brief raised is silently lost. Market research (market-research.md) confirmed every common competitor feature is present here or deliberately excluded, and confirmed the brief's no-per-booking-fee positioning is a validated market differentiator rather than an untested assumption. The completeness audit added three capability enrichments within existing features — Pro-initiated booking changes in FEAT-07, consent-withdrawal handling in FEAT-16, and Operator action logging in FEAT-21 — plus two Domain Entity Inventory corrections and one explicit value-flow scope clarification (see scope-boundaries.md), all without changing the feature count or the tier/phase breakdown.

## Domain Entity Inventory

### Entity: Pro Profile
- **Description:** The business identity of a single Pro — display name, bio-link handle, timezone, currency, and contact details (email and/or phone, used for account communications and recovery) [AUDIT-ADDED: 3 -- Entity Coverage Verification's inverse check found that Pro Account & Authentication (FEAT-14) captures Mara's contact details, but the Domain Entity Inventory did not list them as a stored attribute of any entity].
- **Lifecycle:** Created -> Active -> (rare) Closed
- **Created by:** Pro Onboarding & Setup (FEAT-13)
- **Managed by:** Pro Account & Authentication (FEAT-14), Account Closure & Client Data Deletion (FEAT-20)
- **Referenced by:** Public Booking Page (FEAT-01), Pro Daily Dashboard (FEAT-07), Operator Support Console (FEAT-21)

### Entity: Service
- **Description:** A bookable offering the Pro sells — name, price, duration, and which deposit rule applies to it.
- **Lifecycle:** Created -> Active -> Archived (kept for historical bookings, hidden from new booking)
- **Created by:** Business Settings & Policy Configuration (FEAT-09)
- **Managed by:** Business Settings & Policy Configuration (FEAT-09)
- **Referenced by:** Public Booking Page (FEAT-01), Live Availability & Slot Booking (FEAT-02), Recurring Appointment Booking (FEAT-26)

### Entity: Cancellation & Deposit Policy
- **Description:** The Pro's rule set for a booking: deposit amount or percentage, and the cancellation window before forfeiture applies.
- **Lifecycle:** Created -> Active -> Updated (new bookings use the current version; existing bookings keep the version agreed at booking time)
- **Created by:** Business Settings & Policy Configuration (FEAT-09)
- **Managed by:** Business Settings & Policy Configuration (FEAT-09)
- **Referenced by:** Deposit Payment at Booking (FEAT-04), Client Self-Service Reschedule & Cancellation (FEAT-06), No-Show & Cancellation Deposit Handling (FEAT-08), Booking Record & Dispute Trail (FEAT-18)

### Entity: Working Hours & Buffer Rule
- **Description:** The Pro's weekly working hours and the buffer time required between consecutive bookings.
- **Lifecycle:** Created -> Active -> Updated
- **Created by:** Business Settings & Policy Configuration (FEAT-09)
- **Managed by:** Business Settings & Policy Configuration (FEAT-09)
- **Referenced by:** Live Availability & Slot Booking (FEAT-02)

### Entity: Blocked Time
- **Description:** A manually-created span of time the Pro marks unavailable (personal time, travel, a held slot).
- **Lifecycle:** Created -> Active -> Removed
- **Created by:** Pro Manual Schedule Blocking (FEAT-10)
- **Managed by:** Pro Manual Schedule Blocking (FEAT-10)
- **Referenced by:** Live Availability & Slot Booking (FEAT-02)

### Entity: Calendar Connection
- **Description:** The link between a Pro's account and their personal external calendar, used to pull busy times in and push confirmed bookings out.
- **Lifecycle:** Connected -> Active (syncing) -> Disconnected
- **Created by:** Two-Way Calendar Sync (FEAT-12)
- **Managed by:** Two-Way Calendar Sync (FEAT-12)
- **Referenced by:** Live Availability & Slot Booking (FEAT-02), Pro Onboarding & Setup (FEAT-13)

### Entity: Booking
- **Description:** A single scheduled appointment — service, time, Pro, Client, and current status (pending payment, confirmed, completed, rescheduled, cancelled-in-window, cancelled-out-of-window, no-show).
- **Lifecycle:** Pending payment -> Confirmed -> (Completed | Rescheduled | Cancelled-in-window | Cancelled-out-of-window | No-show)
- **Created by:** Deposit Payment at Booking (FEAT-04) confirms a booking begun in Live Availability & Slot Booking (FEAT-02); Recurring Appointment Booking (FEAT-26) creates a linked series
- **Managed by:** Client Self-Service Reschedule & Cancellation (FEAT-06), Pro Daily Dashboard (FEAT-07) for Pro-initiated reschedule/cancellation [AUDIT-ADDED: 1 -- Persona Journey Walkthrough's counterpart-symmetry check found no feature specified the Pro-side mirror of client-initiated cancellation], No-Show & Cancellation Deposit Handling (FEAT-08), Pro Manual Schedule Blocking (FEAT-10) indirectly through availability
- **Referenced by:** Pro Daily Dashboard (FEAT-07), Booking Confirmation & Reminders (FEAT-05), Booking Record & Dispute Trail (FEAT-18), Client Self-Service Booking History (FEAT-19), Simple Business Insights (FEAT-27), Data Export (FEAT-28)

### Entity: Deposit/Payment Record
- **Description:** The money side of a booking — deposit amount, payment status (pending, held, forfeited, refunded), and, for the Pro, the subscription charge history.
- **Lifecycle:** Pending -> Paid -> (Refunded | Forfeited)
- **Created by:** Deposit Payment at Booking (FEAT-04)
- **Managed by:** No-Show & Cancellation Deposit Handling (FEAT-08), Pro Subscription & Billing (FEAT-17) for the separate subscription charge
- **Referenced by:** Pro Daily Dashboard (FEAT-07), Booking Record & Dispute Trail (FEAT-18), Simple Business Insights (FEAT-27), In-App Balance Payment (FEAT-25)

### Entity: Client Record
- **Description:** A Pro's private record of one client — name, phone number, and freeform notes — built from bookings made with that Pro.
- **Lifecycle:** Created -> Active -> Deleted (on request)
- **Created by:** Client Identity & Booking Details Capture (FEAT-03) on a client's first booking; Client List Import (FEAT-22) in bulk
- **Managed by:** Client Record Management (FEAT-11)
- **Referenced by:** Pro Daily Dashboard (FEAT-07), Account Closure & Client Data Deletion (FEAT-20), Data Export (FEAT-28)

### Entity: Messaging Consent Record
- **Description:** A record of a client's explicit agreement to receive text (or, later, WhatsApp) messages, captured at the moment of booking, and of any later withdrawal of that consent.
- **Lifecycle:** Granted -> Active -> Withdrawn
- **Created by:** Client Identity & Booking Details Capture (FEAT-03)
- **Managed by:** SMS Messaging & Consent Capability (FEAT-16)
- **Referenced by:** Booking Confirmation & Reminders (FEAT-05), WhatsApp Messaging Channel (FEAT-29)

### Entity: Subscription
- **Description:** The Pro's own paid plan with the platform — status (active, past due, cancelled) and billing history.
- **Lifecycle:** Trial/Active -> Past Due -> (Reinstated | Cancelled)
- **Created by:** Pro Subscription & Billing (FEAT-17)
- **Managed by:** Pro Subscription & Billing (FEAT-17)
- **Referenced by:** Payment Processing Capability (FEAT-15), Pro Onboarding & Setup (FEAT-13)

### Entity: Waitlist Entry
- **Description:** A client's request to be notified if an earlier slot opens up for a given service and date range.
- **Lifecycle:** Requested -> (Offered -> Booked | Expired) | Withdrawn
- **Created by:** Cancellation Waitlist (FEAT-23)
- **Managed by:** Cancellation Waitlist (FEAT-23)
- **Referenced by:** Live Availability & Slot Booking (FEAT-02)

### Entity: Tip
- **Description:** An optional extra amount a client adds at checkout, on top of the deposit or balance.
- **Lifecycle:** N/A -- created once at checkout, immutable thereafter
- **Created by:** In-App Tipping at Checkout (FEAT-24)
- **Managed by:** N/A -- once captured, a tip is not edited, only refunded as part of a broader payment reversal
- **Referenced by:** Pro Daily Dashboard (FEAT-07), Simple Business Insights (FEAT-27)

## Core Features

### Public Booking Page

**ID:** FEAT-01

**Description:** The page a client lands on after tapping the Pro's Instagram bio link: the Pro's name, their services with prices and durations, and the deposit rule stated in plain words — everything needed to decide before picking a time.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the brief's literal entry point — "you tap the link in the pro's Instagram bio... and see the pro's name, their services with prices and durations, and the deposit rule in plain words" (BRIEF.md, The Experience). MVP: without this page nothing else in the product is reachable. [INFERRED: carried from Visionary draft]

**Connected Entities:** Pro Profile (read), Service (read), Cancellation & Deposit Policy (read)

**Key Capabilities:**
- View services and prices -- Client sees every bookable service with its price and duration
- View deposit rule -- Client sees the deposit amount or percentage and cancellation policy before proceeding
- Enter the booking flow -- Client moves from browsing to picking a time

**Primary Flows & Alternates:**
- Happy path: client taps the bio link -> page loads inside Instagram's in-app browser -> client reads services, prices, and the deposit rule -> selects a service to continue into slot selection
- No services configured yet: page shows a plain "this Pro hasn't set up booking yet" message rather than an empty or broken layout
- Returning client: the page is identical on every visit — no login, no personalization that would slow the path to booking

**States:** Empty: if the Pro has not configured any services, the page shows a clear "not yet available for booking" message instead of a blank list. Loading: services and the deposit rule render within roughly a second; a lightweight placeholder is shown while they load. Error: if the page fails to load Pro data, a plain retry message appears — never a raw error. Offline-degraded: the page requires connectivity to load initial data; a clear "check your connection" message appears rather than a silent failure.

**Validation & Limits:** No user input on this page — it is a read-only landing view; the only constraint is that at least one active service must exist for the page to allow booking to proceed.

**Access:** Per the Access Matrix in user-persona.md, this is public — any Client (Own-only scope) can view it without identity; Mara (Full) manages what appears here through Business Settings & Policy Configuration. There is no "unauthorized" state — the page is designed to be openly shareable.

**Communications:** N/A — this page sends no messages; it is a passive landing view.

**Data Notes:** Displayed: service names, prices, durations, and the deposit rule. Source: read directly from Business Settings & Policy Configuration (FEAT-09); no data is captured here.

**Interactions:** Depends on Business Settings & Policy Configuration (FEAT-09) for services and policy; feeds into Live Availability & Slot Booking (FEAT-02) once a service is selected.

**Signals:** booking_page_viewed, service_selected.

---

### Live Availability & Slot Booking

**ID:** FEAT-02

**Description:** After picking a service, the client sees only genuinely free times — a live view that accounts for existing bookings, the Pro's working hours and buffer time, manually blocked time, and any busy time from the Pro's connected personal calendar. Double-booking is never possible from this view.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief names double-booking integrity as a non-negotiable quality: "the moment it silently double-books... the pro is gone" (BRIEF.md, Scale & Non-Functional Expectations). This feature is the mechanism that makes that promise true. MVP: the core booking loop cannot function without it. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read, for existing holds), Working Hours & Buffer Rule (read), Blocked Time (read), Calendar Connection (read), Waitlist Entry (update, when a slot releases)

**Key Capabilities:**
- View free slots for a chosen service -- Client sees only times that are actually available, given the service's duration
- Hold a slot during checkout -- The chosen slot is reserved for the duration of the booking flow so two clients cannot claim it at once

**Primary Flows & Alternates:**
- Happy path: client picks a service -> sees a calendar of days and free times -> picks Thursday 2:30pm -> the slot is held while the client completes booking details and payment
- Slot taken during checkout: if another client completes payment for the same slot first, the held client sees an immediate "just booked, please pick another time" message rather than a failed payment
- No slots in the visible window: client can page forward to later dates rather than hitting a dead end

**States:** Empty: a day or week with no free slots shows "fully booked — try another date" rather than a blank grid. Loading: slot computation shows a brief loading indicator, expected within about a second. Error: if availability cannot be computed (e.g., calendar sync is temporarily unreachable), the page falls back to internally-known bookings and buffers only, and shows a discreet notice that externally-blocked time may not be reflected, rather than presenting slots that turn out to be double-booked. Offline-degraded: slot selection requires connectivity; the client sees a clear "reconnect to see live availability" message.

**Validation & Limits:** A slot is only offered if the full service duration plus buffer fits before the next booking or blocked period; a held slot expires after a short checkout window (long enough to complete payment, short enough that it does not lock out other clients if abandoned).

**Access:** Any Client (Own-only per the Access Matrix) can view and hold slots for the Pro whose page they are on; Mara (Full) sees the same computed availability reflected in her own schedule view.

**Communications:** N/A — this feature has no messages of its own; confirmation is handled by Booking Confirmation & Reminders (FEAT-05) once a slot converts to a paid booking.

**Data Notes:** Displayed: computed free slots. Derived: entirely computed from existing Bookings, Working Hours & Buffer Rule, Blocked Time, and Calendar Connection busy time — no new data is captured here beyond the temporary hold.

**Interactions:** Depends on Business Settings & Policy Configuration (FEAT-09), Pro Manual Schedule Blocking (FEAT-10), and Two-Way Calendar Sync (FEAT-12); feeds Client Identity & Booking Details Capture (FEAT-03); also consulted by Cancellation Waitlist (FEAT-23) and Recurring Appointment Booking (FEAT-26).

**Signals:** slots_viewed, slot_held, slot_hold_expired, slot_conflict_detected.

---

### Client Identity & Booking Details Capture

**ID:** FEAT-03

**Description:** Once a slot is held, the client provides just enough to be identified and reachable — name and phone number — and explicitly agrees to receive texts and to the cancellation policy. No password or account creation is involved; the phone number itself, verified with a one-time code sent to it, is the client's identity for future visits to this Pro.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief is explicit that the client "must not need a password-style account just to book" and names "phone number plus a magic link" as its own leading candidate for "the lightest workable identity" (BRIEF.md, Target Users & Roles; Open Questions). This feature resolves that open question in the brief's own preferred direction. MVP: identity and consent capture sit directly in the booking path. [INFERRED: carried from Visionary draft]

**Connected Entities:** Client Record (create), Messaging Consent Record (create), Booking (update, attaching client identity to the held slot)

**Key Capabilities:**
- Enter name and phone -- Client provides the minimum identifying details
- Verify by phone -- A one-time code confirms the phone number without a password
- Agree to texts and policy -- Client gives explicit, recorded consent to messaging and to the cancellation policy before paying

**Primary Flows & Alternates:**
- Happy path: client enters name and phone -> receives and enters a one-time code -> reviews and checks agreement to texts and the cancellation policy -> proceeds to payment
- Returning client: a phone number matching an existing Client Record for this Pro pre-fills the name and skips straight to verification, recognizing the client without a password
- Code not received: client can request the code again after a short cooldown, with a plain-language note to check the number entered

**States:** Empty: N/A — this is a form-entry step with no list or collection to be empty. Loading: code delivery shows a brief "sending code" state; verification shows an immediate check. Error: an incorrect or expired code gives a clear retry message without discarding the name/phone already entered. Offline-degraded: this step requires connectivity to send and verify the code; a clear message asks the client to reconnect.

**Validation & Limits:** Name required (1–100 characters); phone number required and must be a valid, deliverable format; the one-time code expires after a short window and allows a limited number of attempts before requiring a fresh code; consent to texts and to the cancellation policy are both required checkboxes — payment cannot proceed without them.

**Access:** Any Client (Own-only) can complete this step for themselves only; Mara (Full, per Client Records) later sees the resulting Client Record but does not participate in this step.

**Communications:** Sends the one-time verification code by text at this step (distinct from the later booking confirmation).

**Data Notes:** Captured: name, phone number, consent to messaging, consent to cancellation policy, and the specific policy version agreed to. Displayed: pre-filled name for a recognized returning phone number. Source: entirely user-entered at this step.

**Interactions:** Depends on Live Availability & Slot Booking (FEAT-02) for the held slot; depends on SMS Messaging & Consent Capability (FEAT-16) to deliver the verification code; feeds Deposit Payment at Booking (FEAT-04) and Client Record Management (FEAT-11).

**Signals:** identity_form_started, phone_verified, phone_verification_failed, consent_recorded.

---

### Deposit Payment at Booking

**ID:** FEAT-04

**Description:** The client pays the required deposit by card to convert a held slot into a confirmed, protected booking. The product itself never sees or stores the card number — the payment-processing capability owns that.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief's central mechanism — "pay a card deposit under the pro's own cancellation policy" — is what makes a booking real instead of a DM promise (BRIEF.md, Vision). MVP: this is the moment the product's entire value proposition (deposit-first booking) is delivered. [INFERRED: carried from Visionary draft]

**Connected Entities:** Deposit/Payment Record (create), Booking (update, from held to confirmed), Cancellation & Deposit Policy (read, to determine the deposit amount)

**Key Capabilities:**
- Pay the deposit -- Client enters card details (handled entirely by the payment-processing capability) and confirms payment
- See the deposit amount clearly -- Client sees exactly what will be charged before confirming, matching the amount shown on the Public Booking Page

**Primary Flows & Alternates:**
- Happy path: client reviews the deposit amount -> completes card payment -> booking converts from held to confirmed within the same screen, in under a minute total from link tap to done
- Payment declined: client sees a plain decline message and can retry with different card details without losing the held slot, as long as the hold has not expired. [CHALLENGED: GlossGenius reviews report that card validation for deposits creates booking friction for some clients, particularly with debit cards, on an otherwise comparable flat-subscription, no-marketplace-fee product (source: App Store reviews, confidence: MEDIUM) -- original retained per SYN-04 protection, since deposit-by-card is the brief's own central, non-negotiable mechanism (BRIEF.md, Vision); the finding is flagged for Stage 3/4 card-entry UX attention rather than a scope change here.]
- Hold expires during payment: if the slot's temporary hold lapses before payment completes, the client is told the slot is no longer guaranteed and returned to slot selection rather than being charged for a slot that may now be gone

**States:** Empty: N/A — this is a single-purpose payment action, not a list. Loading: a brief "processing payment" state is shown; the client is never left uncertain whether payment went through. Error: a failed or declined payment shows a clear reason where the processor provides one, and a retry path that does not require re-entering booking details. Offline-degraded: payment requires connectivity; a lost connection mid-payment resolves to a definite confirmed-or-not state on reconnect rather than an ambiguous one — the client is never double-charged.

**Validation & Limits:** The deposit amount charged must exactly match the amount or percentage configured for the selected service; a booking is not marked confirmed until payment is verified as successfully captured.

**Access:** Any Client (Own-only) pays only for their own booking; Mara (Full) sees the resulting paid status on her dashboard but does not participate in or see card details during this step.

**Communications:** N/A directly — payment success triggers Booking Confirmation & Reminders (FEAT-05), which owns the client-facing message.

**Data Notes:** Captured: deposit amount charged and payment status. Source: the amount is derived from the service's configured deposit rule; card handling itself is delegated entirely to the payment-processing capability and never touches the product's own records.

**Interactions:** Depends on Client Identity & Booking Details Capture (FEAT-03) and Payment Processing Capability (FEAT-15); confirms the Booking read by Live Availability & Slot Booking (FEAT-02); feeds Pro Daily Dashboard (FEAT-07) and Booking Record & Dispute Trail (FEAT-18).

**Signals:** deposit_payment_started, deposit_payment_succeeded, deposit_payment_failed, booking_confirmed.

---

### Booking Confirmation & Reminders

**ID:** FEAT-05

**Description:** The moment a booking is confirmed, the client gets an immediate text confirmation. Two days before the appointment, a reminder text arrives with a one-tap choice: "I'll be there" or "I need to reschedule."

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief describes this exactly: "a confirmation text lands immediately... two days before, a reminder arrives with a one-tap 'I'll be there / I need to reschedule'" (BRIEF.md, The Experience). This is the feature that replaces Mara's manual, late-night reminder texting. MVP: it directly answers the brief's stated problem of hand-sent reminders. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read), Messaging Consent Record (read)

**Key Capabilities:**
- Send instant confirmation -- Client receives a text the moment a deposit payment succeeds
- Send a timed reminder -- Client receives a reminder text two days ahead of the appointment
- One-tap response -- Client can confirm attendance or start a reschedule directly from the reminder

**Primary Flows & Alternates:**
- Happy path: booking confirms -> confirmation text sends immediately -> two days before the appointment, a reminder text sends with "I'll be there" / "I need to reschedule" -> client taps one, and the outcome is reflected instantly on Mara's dashboard
- Consent withdrawn: if a client has withdrawn messaging consent since booking, no reminder is sent, and the booking still stands — attendance simply is not nudged
- Very-short-notice booking: if a booking is made less than two days out, only the confirmation is sent; no reminder window exists to schedule

**States:** Empty: N/A — this is an outbound messaging feature with no list view of its own. Loading: N/A — message sending is near-instantaneous and has no user-facing loading state. Error: if a confirmation or reminder fails to deliver, the booking itself is unaffected and Mara's dashboard shows the delivery gap so she is not blindsided. Offline-degraded: N/A — sending happens server-side regardless of the client's own connectivity at send time.

**Validation & Limits:** A reminder is only scheduled if messaging consent is active at send time; reminder timing is fixed at two days before the appointment (not user-configurable in this feature).

**Access:** Each Client (Own-only) receives messages only about their own bookings; Mara (Full, via the dashboard) sees whether messages were sent and how the client responded, but does not compose them herself.

**Communications:** This feature is entirely communications: an instant confirmation text and a timed reminder text, both to the client.

**Data Notes:** Displayed to the Pro: confirmation/reminder send status and the client's reminder response. Derived: reminder send time is computed from the booking's appointment time. Source: Booking data and Messaging Consent Record.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-04) for the trigger and SMS Messaging & Consent Capability (FEAT-16) for delivery; a client's "I need to reschedule" tap hands off to Client Self-Service Reschedule & Cancellation (FEAT-06); may also use WhatsApp Messaging Channel (FEAT-29) once available.

**Signals:** confirmation_sent, reminder_sent, reminder_response_received (attending / rescheduling), message_delivery_failed.

---

### Client Self-Service Reschedule & Cancellation

**ID:** FEAT-06

**Description:** From the reminder, or by returning to their booking, a client can move their appointment to a new available time or cancel it — automatically respecting the Pro's cancellation window without either party negotiating by hand.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states the client "reschedules or cancels within the policy window" as a core capability (BRIEF.md, Target Users & Roles), and the whole product's promise is that "nobody negotiated anything" (BRIEF.md, The Experience). MVP: without self-service reschedule, every plan change reverts to a DM, defeating the product's purpose. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (update), Cancellation & Deposit Policy (read)

**Key Capabilities:**
- Reschedule to a new slot -- Client picks a new genuinely-free time for the same service
- Cancel a booking -- Client cancels outright, with the outcome (refund or forfeiture) determined by the cancellation window
- See the policy outcome before confirming -- Client is shown, before finalizing, whether the change is inside or outside the free window

**Primary Flows & Alternates:**
- Happy path: client taps "I need to reschedule" from the reminder (or opens their booking directly) -> is shown current live availability -> picks a new time -> the original slot releases and the new one holds, with no new payment required if within policy
- Cancel inside the window: client cancels with time to spare -> deposit is refunded automatically, per policy
- Cancel or reschedule outside the window: client is shown plainly, before confirming, that the deposit will be forfeited or that rescheduling this close counts as the one change allowed near the appointment — the outcome is never a surprise after the fact

**States:** Empty: N/A — a single booking is being acted on, not a list. Loading: available new slots load with the same brief indicator as initial booking. Error: if a reschedule fails partway (e.g., the newly chosen slot is claimed by someone else first), the original booking remains untouched and the client is asked to pick again. Offline-degraded: this action requires connectivity; a lost connection leaves the original booking exactly as it was.

**Validation & Limits:** Whether an action is treated as "inside" or "outside" the window is evaluated against the exact cancellation-window rule configured for that booking's service at the time it was booked; a booking may be rescheduled to another genuinely free slot only — never into an already-taken one.

**Access:** Any Client (Own-only) can reschedule or cancel only their own booking; Mara (Full) can also reschedule or cancel any booking on her side — see Pro Daily Dashboard (FEAT-07), which specifies that Pro-initiated capability directly.

**Communications:** A confirmation text is sent for the new time on reschedule, or a cancellation acknowledgment (with refund or forfeiture stated plainly) on cancel.

**Data Notes:** Captured: the new time (on reschedule) or the cancellation action and timestamp. Derived: refund-vs-forfeit outcome, computed against the Cancellation & Deposit Policy in force at booking time. Source: client action plus policy lookup.

**Interactions:** Depends on Live Availability & Slot Booking (FEAT-02) for new slots and Business Settings & Policy Configuration (FEAT-09) for the window rule; feeds No-Show & Cancellation Deposit Handling (FEAT-08) for the forfeiture path and Cancellation Waitlist (FEAT-23) by releasing a slot.

**Signals:** reschedule_started, reschedule_completed, cancellation_completed_refunded, cancellation_completed_forfeited.

---

### Pro Daily Dashboard

**ID:** FEAT-07

**Description:** Mara's main working view: today's list of bookings, each showing a paid badge, any client note, and how much is still due at the chair — designed to be glanced at between clients, not studied.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief describes this precisely: "the pro glances at their phone between clients and sees today's list, each booking with a paid badge, a client note, and how much is still due at the chair" (BRIEF.md, The Experience). MVP: this is Mara's primary daily reason to open the product at all. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read, update -- for Pro-initiated reschedule/cancellation [AUDIT-ADDED: 1]), Deposit/Payment Record (read), Client Record (read), Tip (read)

**Key Capabilities:**
- View today's bookings -- Mara sees every booking for the current day in order
- See paid status at a glance -- Each booking shows a clear paid/unpaid badge
- See client notes -- Any note Mara has kept on that client appears with the booking
- See balance due -- Mara sees the remaining amount owed at the chair for each booking
- Navigate to other days -- Mara can look ahead or back from today
- Reschedule or cancel a booking herself -- Mara can move or cancel a client's appointment when she needs to (illness, emergency, a personal conflict), with the client notified immediately and their deposit refunded in full regardless of the cancellation window, since the disruption originates with her, not the client [AUDIT-ADDED: 1 -- Persona Journey Walkthrough's counterpart-symmetry check (completeness-audit.md Section 1) found this capability was referenced in FEAT-06's Access field ("Mara can also reschedule or cancel any booking on her side") but never actually specified anywhere; a client-initiated path existed with no Pro-side mirror]

**Primary Flows & Alternates:**
- Happy path: Mara opens the product between clients -> sees today's remaining bookings in time order -> glances at paid badges and notes -> knows exactly what to expect from the next client
- No bookings today: dashboard shows a plain "nothing booked today" state, not a blank or broken screen
- A booking changed since last look: a client-initiated reschedule or cancellation is reflected immediately, so Mara is never working from a stale list
- Pro-initiated change: Mara needs to cancel or move a booking herself -> she selects it from today's list and reschedules or cancels it -> the client is notified immediately and, because the change originates with Mara rather than the client, any deposit already paid is refunded in full regardless of the cancellation window [AUDIT-ADDED: 1]

**States:** Empty: a day with no bookings shows an explicit "no bookings today" message. Loading: today's list renders within about a second; a lightweight placeholder covers the brief gap. Error: if the list cannot load, Mara sees the last successfully loaded version of today with a retry option, never a blank dashboard. Offline-degraded: the most recently loaded day remains viewable read-only; actions that change data (marking no-show, Pro-initiated reschedule/cancellation, etc.) are disabled until connectivity returns, with a clear notice why.

**Validation & Limits:** No direct input on this view beyond navigation between days and the Pro-initiated reschedule/cancellation action; the day range Mara can browse is unlimited in the past and future, bounded only by what data exists; a Pro-initiated reschedule may only target another genuinely free slot, exactly as the client-initiated path requires.

**Access:** Only Mara (Full, per the Access Matrix) sees this view; Taylor and other clients have no access to it at all — they see only their own bookings via Client Self-Service Booking History (FEAT-19). The Operator (View, read-only) may see an equivalent read-only rendering strictly for troubleshooting a specific Pro's reported issue. The Pro-initiated reschedule/cancellation action is likewise Mara-only (Full) — a client cannot initiate a change on Mara's behalf, and the Operator's view of it remains read-only [AUDIT-ADDED: 1].

**Communications:** N/A for viewing the dashboard itself; the Pro-initiated reschedule or cancellation capability triggers a client-facing notice explaining the change and confirming the full refund, mirroring the notice Client Self-Service Reschedule & Cancellation (FEAT-06) already sends for a client-initiated change [AUDIT-ADDED: 1].

**Data Notes:** Displayed: booking time, service, client name, paid badge, client note, and balance due. Source: aggregated read from Booking, Deposit/Payment Record, Client Record, and Tip; nothing is captured here except the day-navigation choice and, for the Pro-initiated action, the new time or cancellation record.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-04), Client Record Management (FEAT-11), No-Show & Cancellation Deposit Handling (FEAT-08), and In-App Tipping at Checkout (FEAT-24); feeds Operator Support Console (FEAT-21) and Data Export (FEAT-28). The Pro-initiated reschedule/cancellation capability depends on Live Availability & Slot Booking (FEAT-02) for a new slot exactly as Client Self-Service Reschedule & Cancellation (FEAT-06) does, and writes to Booking Record & Dispute Trail (FEAT-18) exactly as a client-initiated change would [AUDIT-ADDED: 1].

**Signals:** dashboard_viewed, day_navigated, empty_day_viewed, pro_initiated_reschedule, pro_initiated_cancellation_refunded [AUDIT-ADDED: 1].

---

### No-Show & Cancellation Deposit Handling

**ID:** FEAT-08

**Description:** When a client doesn't turn up, Mara marks the no-show in one tap and the deposit stays with her automatically. A cancellation made outside the policy window forfeits the deposit on its own, with no action required from Mara at all.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** This is the brief's central financial protection: "a client who doesn't turn up gets one tap to mark the no-show and the deposit stays put, and a cancellation outside the window forfeits on its own" (BRIEF.md, The Experience) — directly solving the founder's stated problem of no-shows costing real money. MVP: without this, the deposit mechanism has no teeth. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (update, to no-show or cancelled-out-of-window), Deposit/Payment Record (update, to forfeited or refunded), Cancellation & Deposit Policy (read, to determine automatic forfeiture eligibility)

**Key Capabilities:**
- Mark a no-show -- Mara marks a booking as a no-show in one tap; the deposit is kept automatically
- Automatic out-of-window forfeiture -- A cancellation past the policy's window forfeits the deposit without Mara doing anything
- Issue a refund within policy -- Mara can still choose to refund a deposit at her own discretion, even when policy would otherwise allow her to keep it

**Primary Flows & Alternates:**
- Happy path (no-show): appointment time passes with the client absent -> Mara taps "no-show" on that booking -> status updates and the deposit is retained, reflected instantly on the dashboard
- Automatic forfeiture: a client cancels outside the window (see FEAT-06) -> the deposit is forfeited without any tap from Mara -> the outcome appears on her dashboard as already resolved
- Discretionary refund: Mara chooses to refund a kept deposit anyway (e.g., a client with a documented emergency) -> she issues the refund manually, overriding the default policy outcome in the client's favor

**States:** Empty: N/A — this acts on a specific existing booking, not a list. Loading: a brief confirmation state shows while the no-show mark or refund is processed. Error: if marking a no-show or issuing a refund fails to process, the booking's prior state is preserved and Mara is shown a clear retry option — the deposit status is never left ambiguous. Offline-degraded: marking a no-show or issuing a refund requires connectivity; the action is disabled with a clear notice until reconnected.

**Validation & Limits:** A booking can only be marked no-show after its scheduled time has passed; a no-show mark can be undone shortly after (to correct a mis-tap) but not after the deposit has already been included in a settlement; a discretionary refund cannot exceed the original deposit amount.

**Access:** Only Mara (Full) can mark no-shows or issue discretionary refunds; Taylor (a Client) has no access to this action — a client cannot mark their own no-show or self-approve a refund outside policy. The Operator (View) can see the resulting status for support purposes only.

**Communications:** The client receives a plain notice when a deposit is forfeited (no-show or out-of-window cancellation) or refunded, stating the outcome and, where forfeited, the policy it was based on. [RESEARCH-INFORMED: guaranteeing an explicit client-facing notice on every forfeiture or refund outcome is a deliberate response to a documented, market-wide gap — competitors report cases of a client being charged for a no-show with no in-app notification, discovered only via a bank statement (source: App Store reviews and BBB complaint filings for Booksy, confidence: MEDIUM-HIGH), a pattern this feature is designed not to repeat.]

**Data Notes:** Captured: the no-show mark, its timestamp, and any discretionary refund with Mara's stated reason (optional freeform note). Derived: automatic forfeiture is computed from the Cancellation & Deposit Policy in force at booking time. Source: Mara's action, or the policy engine for automatic forfeiture.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-04) and Business Settings & Policy Configuration (FEAT-09); writes to Booking Record & Dispute Trail (FEAT-18); reflected on Pro Daily Dashboard (FEAT-07) and Simple Business Insights (FEAT-27).

**Signals:** no_show_marked, no_show_mark_undone, deposit_forfeited_automatically, deposit_refunded_discretionary.

---

### Business Settings & Policy Configuration

**ID:** FEAT-09

**Description:** The place Mara sets up how her business runs: her services with prices and durations, her deposit rule, her working hours and buffer time between clients, her cancellation window, and her timezone and currency.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** Every other feature depends on rules Mara defines here — the brief lists exactly these controls as what the Pro "sets up" (BRIEF.md, Target Users & Roles). MVP: the product has nothing to show a client until this exists. [INFERRED: carried from Visionary draft]

**Connected Entities:** Service (create, update, archive), Cancellation & Deposit Policy (create, update), Working Hours & Buffer Rule (create, update)

**Key Capabilities:**
- Manage services -- Mara adds, edits, and archives services with names, prices, and durations
- Set the deposit rule -- Mara sets a flat amount or percentage deposit, per service or business-wide
- Set working hours and buffer -- Mara defines her weekly working hours and the buffer time required between bookings
- Set the cancellation window -- Mara defines how far ahead a client must cancel or reschedule to avoid forfeiting the deposit
- Set timezone and currency -- Mara's business operates in her own timezone and currency, not a hard-coded one

**Primary Flows & Alternates:**
- Happy path: Mara adds a service with a name, price, and duration -> sets a deposit rule -> sets her working hours, buffer, and cancellation window -> settings take effect immediately for new bookings
- Editing an in-use service: changing a service's price or deposit rule applies to new bookings only; already-confirmed bookings keep the terms the client agreed to
- Business-wide vs per-service deposit: Mara can set one deposit rule for everything, or override it for a specific service (e.g., a higher-value service carries a higher deposit)

**States:** Empty: a Pro with no services yet sees a clear prompt to add the first one, not a blank settings page. Loading: settings load and save with brief, visible confirmation. Error: a failed save preserves Mara's entered changes and offers a retry, never silently discarding edits. Offline-degraded: settings can be viewed read-only offline; changes require connectivity to save, with a clear notice.

**Validation & Limits:** Service name required (1–100 characters); price must be a positive value; duration must be a positive number of minutes; deposit must be a positive flat amount or a percentage between 1–100; working hours must not overlap themselves; buffer time is a non-negative number of minutes; cancellation window must be a positive number of hours.

**Access:** Only Mara (Full) can view or change these settings; Taylor and other clients have no access at all — they only ever see the resulting Public Booking Page. The Operator (View) can see current settings read-only for troubleshooting.

**Communications:** N/A — configuration changes do not trigger client-facing messages by themselves.

**Data Notes:** Captured: services, prices, durations, deposit rule, working hours, buffer, cancellation window, timezone, currency. Displayed: current configuration and its effective date. Source: entirely Mara's own input.

**Interactions:** Feeds Public Booking Page (FEAT-01), Live Availability & Slot Booking (FEAT-02), Deposit Payment at Booking (FEAT-04), Client Self-Service Reschedule & Cancellation (FEAT-06), and No-Show & Cancellation Deposit Handling (FEAT-08); consumed during Pro Onboarding & Setup (FEAT-13).

**Signals:** service_created, service_updated, service_archived, policy_updated, hours_updated.

<!-- [INFERRED: carried from Visionary draft] -->

---

### Pro Manual Schedule Blocking

**ID:** FEAT-10

**Description:** Mara can block off a span of time — personal time, travel, or simply keeping a slot free — so it never appears as bookable, without having to invent a fake appointment to hide it.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief lists "block off time" among the Pro's core capabilities (BRIEF.md, Target Users & Roles). MVP: without it, Mara's only way to protect personal time is through her external calendar, and the brief's calendar sync is confirmed but not guaranteed to be connected or perfectly timed — a direct blocking tool is the reliable fallback the non-negotiable double-booking promise requires. [INFERRED: carried from Visionary draft]

**Connected Entities:** Blocked Time (create, update, delete)

**Key Capabilities:**
- Block a span of time -- Mara marks a date/time range as unavailable
- Remove a block -- Mara un-blocks time she no longer needs held
- Recurring block -- Mara can mark a block as repeating (e.g., every Sunday)

**Primary Flows & Alternates:**
- Happy path: Mara selects a date and time range -> confirms the block -> that time immediately stops appearing as bookable in Live Availability & Slot Booking
- Recurring block: Mara sets a weekly recurring block (e.g., Sundays) once, rather than repeating the action every week
- Removing a block with an existing booking: if Mara later tries to block time that already holds a confirmed booking, she is warned and must resolve the conflicting booking first rather than silently orphaning it

**States:** Empty: a Pro with no blocks sees a plain "no blocked time" state. Loading: blocks load and apply with a brief, visible confirmation. Error: a failed block save leaves prior availability unchanged and offers a retry. Offline-degraded: existing blocks remain visible read-only; adding or removing a block requires connectivity.

**Validation & Limits:** A block must have a start before its end; a block cannot be silently created over a slot with an existing confirmed booking — Mara must address the conflict explicitly.

**Access:** Only Mara (Full) can create or remove blocked time; clients have no visibility into blocks beyond simply not seeing that time as available.

**Communications:** N/A — blocking time is a private scheduling action with no client-facing message.

**Data Notes:** Captured: block start, end, and optional recurrence. Source: entirely Mara's own input.

**Interactions:** Feeds Live Availability & Slot Booking (FEAT-02); interacts with Booking Conflict Recovery flows.

**Signals:** block_created, block_removed, block_conflict_warned.

---

### Client Record Management

**ID:** FEAT-11

**Description:** Mara's private list of every client who has booked with her — name, phone, and any freeform note she keeps (allergies, preferences, history) — visible only to her, with the ability to delete a client's record on request.

**Priority:** Core

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states Mara "sees every booking, every client" and that "the Pro is the only person who ever sees the client list" (BRIEF.md, Target Users & Roles), and separately requires that "a pro must be able to delete a client's record on request" (BRIEF.md, Constraints). MVP: this is core to running the day-to-day relationship with returning clients, and the deletion right is a stated regulatory-adjacent constraint, not an optional add-on. [INFERRED: carried from Visionary draft]

**Connected Entities:** Client Record (read, update, delete)

**Key Capabilities:**
- View the client list -- Mara sees every client who has booked with her
- Search clients -- Mara finds a specific client by name or phone
- Add or edit a note -- Mara keeps a freeform note per client
- Delete a client's record -- Mara permanently removes a client's record on request

**Primary Flows & Alternates:**
- Happy path: Mara opens her client list -> searches or scrolls to find someone -> views or edits their note
- First booking creates the record: a client's first successful booking with Mara automatically creates their Client Record — Mara never has to add clients manually for this to work
- Deletion request: a client asks Mara to delete their data -> Mara finds them in the list -> deletes the record -> the client's personal details are removed while past booking history is anonymized rather than silently vanishing from Mara's own financial records

**States:** Empty: a brand-new Pro with no clients yet sees a plain "no clients yet — your first booking will appear here" message. Loading: the list and search render within about a second. Error: a failed load shows the last successfully loaded list with a retry option. Offline-degraded: the client list remains viewable read-only; edits and deletion require connectivity.

**Validation & Limits:** A note is freeform text up to a generous length (e.g., 1,000 characters); deletion is a deliberate, confirmed action (not a single accidental tap) given it is irreversible.

**Access:** Only Mara (Full, own clients only) can view or manage her client list; no other Pro can ever see it, and clients themselves have no access to this view of their own record — they see only their own bookings via Client Self-Service Booking History (FEAT-19).

**Communications:** N/A — viewing or noting a client record does not itself message the client; deletion may prompt an optional acknowledgment to the client that their data was removed.

**Data Notes:** Captured: freeform notes (Mara's input). Displayed: name, phone, note, and booking history summary. Source: name and phone originate from Client Identity & Booking Details Capture (FEAT-03) or Client List Import (FEAT-22); notes are Mara's own input.

**Interactions:** Depends on Client Identity & Booking Details Capture (FEAT-03) and Client List Import (FEAT-22); feeds Pro Daily Dashboard (FEAT-07), Account Closure & Client Data Deletion (FEAT-20), and Data Export (FEAT-28).

**Signals:** client_list_viewed, client_searched, client_note_updated, client_record_deleted.

---

### Two-Way Calendar Sync

**ID:** FEAT-12

**Description:** Mara connects her personal calendar so that busy time she has already committed elsewhere blocks availability here, and confirmed bookings made here appear on her personal calendar automatically — one true schedule instead of two she has to reconcile by hand.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief names this as a confirmed, two-way integration: "the pro's personal... calendar... busy times there block availability here, and bookings made here appear there" (BRIEF.md, Ecosystem & Integrations). MVP: double-booking integrity is a stated non-negotiable, and Mara's personal life commitments live on her existing calendar today — sync is required from day one to keep that promise true. [INFERRED: carried from Visionary draft]

**Connected Entities:** Calendar Connection (create, update, delete)

**Key Capabilities:**
- Connect a personal calendar -- Mara links her existing calendar to the product
- Pull busy time in -- Existing personal events block matching availability automatically
- Push bookings out -- Confirmed bookings appear on Mara's personal calendar automatically
- Disconnect -- Mara can unlink her calendar at any time

**Primary Flows & Alternates:**
- Happy path: Mara connects her calendar during onboarding (or later from settings) -> busy events immediately start blocking matching availability -> every new confirmed booking appears on her personal calendar within a short delay
- Sync temporarily unavailable: if the external calendar is briefly unreachable, availability falls back to internally-known bookings and blocks only, with a discreet notice that external busy time may be briefly stale, rather than silently trusting outdated data
- Disconnection: Mara disconnects her calendar -> external busy time stops being pulled in going forward, and previously pushed bookings remain on her personal calendar (the product does not reach back to remove them)

**States:** Empty: a Pro who has not connected a calendar sees a plain, optional prompt to do so — connection is not mandatory to use the product. Loading: initial sync after connecting shows a visible "syncing your calendar" state. Error: a sync failure is shown discreetly to Mara with guidance to reconnect if it persists; it never silently and permanently breaks availability accuracy without telling her. Offline-degraded: the last successfully synced busy-time snapshot continues to inform availability until sync resumes.

**Validation & Limits:** Only one personal calendar connection per Pro in v1; sync delay for busy time and pushed bookings is expected to be short (on the order of a few minutes), not instantaneous.

**Access:** Only Mara (Full) can connect, view sync status, or disconnect her own calendar; this has no client-facing surface at all.

**Communications:** N/A — sync is a background capability with no messages of its own beyond an in-product status notice to Mara if it fails.

**Data Notes:** Captured: connection credentials/authorization (handled by the calendar capability itself, not stored as product data beyond what is needed to maintain the connection). Displayed: sync status. Derived: busy-time blocks used by Live Availability & Slot Booking. Source: the Pro's connected personal calendar.

**Interactions:** Feeds Live Availability & Slot Booking (FEAT-02); consumed during Pro Onboarding & Setup (FEAT-13); interacts with Pro Manual Schedule Blocking (FEAT-10) as a second, external source of blocked time.

**Signals:** calendar_connected, calendar_disconnected, calendar_sync_succeeded, calendar_sync_failed.

---

### Pro Onboarding & Setup

**ID:** FEAT-13

**Description:** The guided first-run experience that takes Mara from signing up to having a working, shareable booking page: creating her account, adding her first services and policy, optionally connecting her calendar, and getting her bio-link URL.

**Priority:** Core

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** Every product has a first run, and this one gates the founder's own three-month goal of "first paying pro" (BRIEF.md, Constraints) — a Pro who cannot quickly reach a working page never becomes that first paying customer. MVP: it is the on-ramp to every other Core feature. [INFERRED: carried from Visionary draft]

**Connected Entities:** Pro Profile (create), Subscription (create), Service (create, via handoff to FEAT-09), Calendar Connection (create, optional, via handoff to FEAT-12)

**Key Capabilities:**
- Create a Pro account -- Mara signs up and creates her business profile
- Add first services and policy -- Mara is guided through adding at least one service and setting her deposit/cancellation rule
- Connect calendar (optional) -- Mara can connect her personal calendar during setup or skip and do it later
- Get the bio-link URL -- Mara receives the link to put in her Instagram bio

**Primary Flows & Alternates:**
- Happy path: Mara signs up -> adds her business name, timezone, and currency -> adds at least one service with a price, duration, and deposit rule -> sets basic hours -> optionally connects her calendar -> receives her bio-link URL, ready to share
- Skip calendar connection: Mara completes setup without connecting a calendar; her availability is still fully protected by internal bookings and manual blocking, and she can connect a calendar later from settings with no penalty
- Abandon mid-setup: Mara closes the product partway through; on return, setup resumes exactly where she left off with earlier answers preserved

**States:** Empty: N/A — onboarding is itself the empty-state resolution for a brand-new Pro. Loading: each step saves with brief, visible confirmation before advancing. Error: a failed save at any step preserves entered data and offers a retry, never forcing Mara to restart. Offline-degraded: onboarding requires connectivity to create the account and save progress; a clear message asks her to reconnect.

**Validation & Limits:** At least one service with a valid price, duration, and deposit rule is required before the bio-link URL is issued; all other steps (hours refinement, calendar connection) can be completed or revisited later from settings.

**Access:** Only a new or existing Mara (Full, for her own onboarding) goes through this; there is no client- or operator-facing equivalent.

**Communications:** A welcome message confirms account creation and provides the bio-link URL once minimum setup is complete.

**Data Notes:** Captured: business name, timezone, currency, initial services, initial policy, optional calendar connection. Source: entirely Mara's own input during this guided flow.

**Interactions:** Hands off to Business Settings & Policy Configuration (FEAT-09) and Two-Way Calendar Sync (FEAT-12); depends on Pro Account & Authentication (FEAT-14) and Pro Subscription & Billing (FEAT-17).

**Signals:** onboarding_started, onboarding_step_completed, onboarding_resumed, onboarding_completed, bio_link_issued.

---

### Pro Account & Authentication

**ID:** FEAT-14

**Description:** Mara's own account — how she signs in securely, recovers access if she's locked out, and manages her basic profile details.

**Priority:** Core

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** A paid, single-operator business tool requires a durable, securable account for its one operator — this is domain-standard scaffolding every feature in the product depends on. [INFERRED from: domain knowledge — a subscription business tool with a Pro who "sets up services, prices... working hours" (BRIEF.md, Target Users & Roles) cannot exist without an account that reliably belongs to that one Pro across sessions.] MVP: every other Pro-facing feature assumes an authenticated Mara. [INFERRED: carried from Visionary draft]

**Connected Entities:** Pro Profile (create, update)

**Key Capabilities:**
- Sign in securely -- Mara accesses her account from her phone
- Recover access -- Mara regains access if she loses her sign-in method
- Manage basic profile -- Mara updates her name, business name, and contact details

**Primary Flows & Alternates:**
- Happy path: Mara signs in on her phone -> lands on her dashboard
- Lost access: Mara requests recovery -> verifies she owns the account -> regains access without losing any of her data
- Profile update: Mara edits her business name or contact details -> change applies immediately to her Public Booking Page

**States:** Empty: N/A — an account either exists or the Pro is in onboarding. Loading: sign-in shows a brief confirmation state. Error: a failed sign-in shows a clear, non-technical reason (wrong details, needs recovery) without revealing whether a given identifier is registered. Offline-degraded: previously signed-in sessions may continue to view already-loaded data; signing in fresh requires connectivity.

**Validation & Limits:** Standard account-security expectations apply — a functional, non-technical way to prove ownership at sign-in and at recovery; one account per Pro.

**Access:** Only Mara (Full) accesses her own account; the Operator (View) may look up account status read-only for support but cannot sign in as Mara.

**Communications:** Account-related notices (e.g., a recovery request was made) are sent to Mara's own registered contact method.

**Data Notes:** Captured: Mara's name, business name, and contact details (now reflected as a Pro Profile attribute — see Domain Entity Inventory). Source: Mara's own input, established during Pro Onboarding & Setup (FEAT-13).

**Interactions:** Depended on by every Pro-facing feature; feeds Pro Subscription & Billing (FEAT-17) and Two-Way Calendar Sync (FEAT-12).

**Signals:** signed_in, sign_in_failed, recovery_requested, profile_updated.

---

### Payment Processing Capability

**ID:** FEAT-15

**Description:** The underlying capability that collects client deposits and the Pro's own subscription payments by card, without the product's own code ever seeing or storing a card number.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief is explicit and non-negotiable here: "an established card payment processor — collects deposits and subscription payments; owns all card data. The product never sees or stores a card number" (BRIEF.md, Ecosystem & Integrations). MVP: both deposit collection and subscription billing depend on it from day one. [INFERRED: carried from Visionary draft]

**Connected Entities:** Deposit/Payment Record (create, via FEAT-04), Subscription (create, via FEAT-17)

**Key Capabilities:**
- Collect a client deposit -- Processes a card payment into the Pro's own payout account
- Collect a Pro's subscription payment -- Processes the Pro's own monthly card charge
- Process a refund -- Returns a previously collected deposit to the client's card

**Primary Flows & Alternates:**
- Happy path: a deposit or subscription charge is requested -> payment-processing capability handles card entry and authorization -> result (success/decline) is returned to the requesting feature
- Decline: the capability returns a clear decline reason where available, without exposing any card details back to the product
- Refund: a previously captured deposit is returned to the original card via the same capability

**States:** Empty: N/A — this is a capability invoked by other features, not a standalone view. Loading: N/A — handled within the invoking feature's own loading state. Error: N/A — handled within the invoking feature's own error state (see FEAT-04, FEAT-17). Offline-degraded: N/A — requires connectivity by nature; handled within the invoking feature.

**Validation & Limits:** The product's own code never receives or stores raw card numbers, per the brief's stated regulatory constraint (BRIEF.md, Constraints); all money the client pays as a deposit flows directly into the Pro's own payout account, with no per-booking platform cut (BRIEF.md, Business Context).

**Access:** N/A — this is a Platform-type capability with no direct user-facing access surface of its own; it is invoked by Deposit Payment at Booking (FEAT-04), No-Show & Cancellation Deposit Handling (FEAT-08), and Pro Subscription & Billing (FEAT-17), which carry their own Access rules.

**Communications:** N/A — this capability itself sends no messages; the features that invoke it own their own communications.

**Data Notes:** Captured: none directly by the product beyond payment status and amount; card data is owned entirely by the processing capability. Derived: N/A. Source: N/A.

**Interactions:** Invoked by Deposit Payment at Booking (FEAT-04), No-Show & Cancellation Deposit Handling (FEAT-08, for refunds), Pro Subscription & Billing (FEAT-17), In-App Tipping at Checkout (FEAT-24), and In-App Balance Payment (FEAT-25).

**Signals:** payment_capability_invoked, payment_capability_succeeded, payment_capability_failed.

---

### SMS Messaging & Consent Capability

**ID:** FEAT-16

**Description:** The underlying capability that delivers text messages to clients — verification codes, confirmations, and reminders — while enforcing that a client's explicit consent is captured and respected before any message beyond the initial verification is sent, and that a client can withdraw that consent at any time.

**Priority:** Core

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief requires explicit consent captured at booking with reminders that "respect that consent (US texting rules)" (BRIEF.md, Constraints; Ecosystem & Integrations). MVP: confirmations and reminders — a Core capability — cannot legally or functionally exist without this. [INFERRED: carried from Visionary draft]

**Connected Entities:** Messaging Consent Record (create, read, update)

**Key Capabilities:**
- Deliver a text message -- Sends a verification code, confirmation, or reminder to a client's phone
- Enforce consent -- Blocks any consent-gated message (confirmation, reminder) if consent is not active
- Record consent state -- Tracks whether a client has granted or withdrawn consent
- Respect an opt-out reply -- A client can withdraw consent at any time by replying STOP (or an equivalent recognized keyword) to any text message received from the product, per standard US texting practice; consent-gated messages for that client stop immediately on receipt [AUDIT-ADDED: 3 -- Entity Coverage Verification's inverse check found the Messaging Consent Record entity had no specified withdrawal mechanism — Booking Confirmation & Reminders (FEAT-05)'s "Consent withdrawn" alternate flow already assumed clients could withdraw consent, but no feature specified how]

**Primary Flows & Alternates:**
- Happy path: a feature requests a message be sent -> capability checks consent state where required -> delivers the message -> reports delivery status back
- Consent withdrawn: a client replies STOP (or an equivalent recognized keyword) to any text -> consent is recorded as withdrawn immediately -> future consent-gated messages for that client stop, and already-scheduled reminders for existing bookings are cancelled rather than sent anyway [AUDIT-ADDED: 3]
- Delivery failure: the capability reports a failure back to the requesting feature rather than silently dropping the message

**States:** Empty: N/A — this is a capability, not a standalone view. Loading: N/A — handled within the invoking feature. Error: delivery failures are reported to the invoking feature to display appropriately (see FEAT-05). Offline-degraded: N/A — requires connectivity by nature.

**Validation & Limits:** No consent-gated message (confirmation, reminder) is ever sent without an active, recorded consent for that client; the one-time verification code at booking is the one message type sent prior to consent being granted, since it is part of establishing identity, not marketing or reminder content; a STOP reply (or equivalent) is recognized and processed immediately regardless of which specific message it is sent in reply to [AUDIT-ADDED: 3].

**Access:** N/A — this is a Platform-type capability with no direct user-facing access surface; it is invoked by Client Identity & Booking Details Capture (FEAT-03) and Booking Confirmation & Reminders (FEAT-05), which carry their own Access rules. The opt-out mechanism is exercised directly by the Client (Own-only) via text reply, outside any screen.

**Communications:** N/A — this capability performs communications on behalf of other features rather than owning any of its own.

**Data Notes:** Captured: consent grant/withdrawal events and timestamps. Source: client action during Client Identity & Booking Details Capture (FEAT-03), a later STOP reply, or another consent-withdrawal action.

**Interactions:** Invoked by Client Identity & Booking Details Capture (FEAT-03) and Booking Confirmation & Reminders (FEAT-05); extended by WhatsApp Messaging Channel (FEAT-29).

**Signals:** message_send_requested, message_delivered, message_delivery_failed, consent_withdrawn, opt_out_reply_received [AUDIT-ADDED: 3].

---

### Pro Subscription & Billing

**ID:** FEAT-17

**Description:** Mara's own flat monthly subscription to use the product, paid by card inside the product itself — with no per-booking cut ever taken from her deposits.

**Priority:** Core

**Phase:** MVP

**Type:** Lifecycle

**Rationale:** The brief's business model is explicit: "a flat monthly subscription per pro... absolutely no per-booking cut" and "the pro pays the subscription by card inside the product" (BRIEF.md, Business Context). MVP: this is how the product is commercially viable from the very first paying Pro. [RESEARCH-INFORMED: market research found the per-new-client marketplace fee charged by StyleSeat, Booksy (opt-in), Fresha, and theCut is the space's most consistently documented professional-side resentment — particularly clients misclassified as "new" on repeat or referral visits — while GlossGenius, the one profiled fee-free competitor, draws no comparable complaint pattern (source: Trustpilot and App Store reviews across 3+ products, confidence: HIGH). This corroborates the brief's flat-fee, no-commission choice as a validated market differentiator, not merely an untested assumption.]

**Connected Entities:** Subscription (create, update)

**Key Capabilities:**
- Subscribe -- Mara starts her paid subscription by card
- View billing status -- Mara sees her current plan status and next charge date
- Update payment method -- Mara updates the card used for her own subscription
- Handle a failed charge -- Mara is clearly told if her subscription payment fails and how to fix it
- Cancel subscription -- Mara can cancel her own subscription at any time; her Public Booking Page stops accepting new bookings at the end of the current billing period, while existing confirmed bookings remain honored [AUDIT-ADDED: 3 -- Entity Coverage Verification found the Subscription entity's cancellation path described only in this feature's Primary Flows & Alternates, not listed as a Key Capability Mara can initiate directly]

**Primary Flows & Alternates:**
- Happy path: Mara enters card details during or shortly after onboarding -> subscription activates -> she is charged automatically each month going forward
- Payment fails: a monthly charge fails -> Mara is notified clearly and given a grace period and a simple way to update her card before booking capability is affected
- Cancellation: Mara cancels her subscription -> her Public Booking Page stops accepting new bookings at the end of the current billing period, while existing confirmed bookings remain honored

**States:** Empty: N/A — a Pro is either mid-onboarding (no subscription yet) or subscribed. Loading: billing actions show a brief confirmation state. Error: a failed charge or update shows a clear, specific reason and next step, never a silent failure. Offline-degraded: billing status can be viewed read-only offline; changes require connectivity.

**Validation & Limits:** Subscription is billed at a single flat monthly rate, per pro, with no usage- or booking-based variable component; a grace period applies before a failed payment suspends new-booking capability, so a single card decline does not instantly take Mara offline.

**Access:** Only Mara (Full) manages her own subscription; there is no client-facing surface, and the Operator (View) can see subscription status read-only for support purposes only.

**Communications:** Billing confirmations, upcoming-charge notices, and payment-failure alerts are sent to Mara.

**Data Notes:** Captured: subscription status and billing history. Source: Payment Processing Capability (FEAT-15) reports charge outcomes here.

**Interactions:** Depends on Pro Account & Authentication (FEAT-14) and Payment Processing Capability (FEAT-15); consumed during Pro Onboarding & Setup (FEAT-13).

**Signals:** subscription_started, subscription_charge_succeeded, subscription_charge_failed, subscription_cancelled.

---

## Important Features

### Booking Record & Dispute Trail

**ID:** FEAT-18

**Description:** A durable, timestamped record of each booking's history — what policy was agreed to, when payment happened, and what status changes occurred — so that if a client disputes a no-show charge, Mara has something concrete to point to.

**Priority:** Important

**Phase:** MVP

**Type:** Platform

**Rationale:** The brief names this exact gap as a current pain point: "no record when a client disputes a no-show charge" (BRIEF.md, Problem Statement). Ranked Important rather than Core because the booking loop itself still functions without a dedicated dispute view — but phased to MVP rather than later, because the underlying record it depends on (what was agreed, when, and what changed) must be captured from the very first booking, or it is unrecoverable retroactively once a dispute arises. [RESEARCH-INFORMED: this is a market-wide, unresolved gap, not merely the brief's own pain point — every profiled competitor with reviewable enforcement evidence (GlossGenius, Booksy, Fresha) has at least one documented case of a no-show or deposit charge failing silently or arriving with no client notification (source: App Store reviews across 3 products, confidence: HIGH). This is the direct market evidence for keeping this feature's timestamped record and policy-version capture in MVP rather than deferring it.]

**Connected Entities:** Booking (read), Deposit/Payment Record (read), Cancellation & Deposit Policy (read, as it stood at booking time)

**Key Capabilities:**
- View a booking's full history -- Mara sees every status change for a booking (booked, confirmed, reminded, no-show marked, etc.) with timestamps
- See the policy as agreed -- Mara sees exactly what deposit and cancellation terms the client agreed to at booking time, even if her current settings have since changed

**Primary Flows & Alternates:**
- Happy path: a client disputes a forfeited deposit -> Mara opens that booking's history -> sees the exact policy version agreed to, the payment timestamp, and the no-show or cancellation timestamp -> has a concrete answer
- Policy changed since booking: the history always shows the policy version in force when that specific booking was made, never the currently-active version, so a later policy change cannot retroactively look like it applied to an old booking

**States:** Empty: N/A — every booking has at least a creation event; there is no truly empty history. Loading: history renders with the same brief indicator as the dashboard. Error: if history cannot load, the underlying booking and payment status (from FEAT-07) are still visible even if the detailed timeline is temporarily unavailable. Offline-degraded: previously viewed history remains available read-only.

**Validation & Limits:** History entries are immutable once recorded — no status-change event can be edited or deleted after the fact, only added to, so the trail itself cannot become part of a dispute.

**Access:** Only Mara (Full) can view a booking's dispute trail; the Operator (View, read-only) can see the same trail strictly to help troubleshoot a reported issue. Clients do not see this internal history — they see their own booking's current status via Client Self-Service Booking History (FEAT-19).

**Communications:** N/A — this is an internal record-keeping feature with no messages of its own.

**Data Notes:** Captured: every status-change event on a booking, with timestamp and the policy version in force at booking time. Derived: none — this is a faithful log, not a computed summary. Source: every other Core feature that changes a booking's status writes an entry here, including Pro Daily Dashboard (FEAT-07)'s Pro-initiated reschedule/cancellation capability.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-04), No-Show & Cancellation Deposit Handling (FEAT-08), and Pro Daily Dashboard (FEAT-07) as writers; read by Operator Support Console (FEAT-21).

**Signals:** dispute_trail_viewed.

---

### Client Self-Service Booking History

**ID:** FEAT-19

**Description:** A client's own simple view of their upcoming and past bookings with this specific Pro — nothing about any other Pro or any other client.

**Priority:** Important

**Phase:** MVP

**Type:** User-Facing

**Rationale:** The brief states the client "sees only their own upcoming and past bookings with that pro" (BRIEF.md, Target Users & Roles). Ranked Important rather than Core because the booking-and-reminder loop functions without a dedicated history view (the confirmation and reminder texts already carry the essential details) — but phased to MVP because it is a small, low-risk addition that meaningfully reduces "did I already book this?" confusion from day one. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read)

**Key Capabilities:**
- View upcoming bookings -- Client sees any future bookings with this Pro
- View past bookings -- Client sees a simple history of past appointments with this Pro

**Primary Flows & Alternates:**
- Happy path: client re-verifies their phone number (same lightweight identity as booking) -> sees their upcoming and past bookings with this Pro
- No bookings yet: a client who has never booked with this Pro sees a plain empty state rather than an error

**States:** Empty: a client with no bookings sees "no bookings yet" rather than a blank screen. Loading: the list loads with the same brief indicator used elsewhere. Error: a failed load shows a retry option. Offline-degraded: the most recently loaded view remains available read-only.

**Validation & Limits:** A client can only view bookings tied to their own verified phone number, and only with this one Pro — never a cross-Pro view.

**Access:** Any Client (Own-only) sees only their own bookings with this Pro; Mara has no reason to use this view herself (she has the fuller Pro Daily Dashboard, FEAT-07).

**Communications:** N/A — this is a viewing surface with no messages of its own.

**Data Notes:** Displayed: booking time, service, and status for this client's bookings with this Pro. Source: read from Booking.

**Interactions:** Depends on Client Identity & Booking Details Capture (FEAT-03) for identity verification; reads Booking data written by Deposit Payment at Booking (FEAT-04) and updated by Client Self-Service Reschedule & Cancellation (FEAT-06) and Pro Daily Dashboard (FEAT-07).

**Signals:** booking_history_viewed.

---

### Account Closure & Client Data Deletion

**ID:** FEAT-20

**Description:** Mara can close her own account if she stops using the product, and can permanently delete an individual client's record on that client's request — both cleanly, without leaving orphaned or ambiguous data behind.

**Priority:** Important

**Phase:** v1

**Type:** User-Facing

**Rationale:** The brief requires client-record deletion on request as a stated privacy constraint (BRIEF.md, Constraints); account closure is the natural, domain-standard counterpart for the Pro's own account. [INFERRED from: domain knowledge — any subscription product needs a clean way for its one paying customer to leave.] Ranked Important because it is not part of the daily value loop. Phased to v1 rather than MVP: the individual client-deletion capability itself is required from MVP (see FEAT-11's Key Capabilities, which already includes it) — this feature specifically covers the Pro's own full account closure, which can reasonably wait until real Pros exist who might want to leave. [INFERRED: carried from Visionary draft]

**Connected Entities:** Pro Profile (delete), Client Record (delete, in bulk on account closure), Subscription (update, to cancelled)

**Key Capabilities:**
- Close the Pro account -- Mara permanently closes her account and stops billing
- Understand what happens to data -- Mara is told plainly what is deleted versus retained (e.g., for legal/financial record-keeping) before confirming

**Primary Flows & Alternates:**
- Happy path: Mara requests account closure -> is shown plainly what will be deleted and what (if anything) is retained for financial record-keeping -> confirms -> subscription is cancelled and her Public Booking Page stops accepting bookings immediately
- Change of mind: Mara can cancel a closure request within a short grace window before it takes final effect

**States:** Empty: N/A — this is a deliberate account action, not a list. Loading: closure shows a clear "processing" state. Error: a failed closure leaves the account fully active and notifies Mara to retry. Offline-degraded: closure requires connectivity.

**Validation & Limits:** Account closure requires explicit confirmation of a clear warning (not a single accidental tap), given it is largely irreversible after the grace window.

**Access:** Only Mara (Full) can close her own account; only Mara (Full, own clients only) can delete an individual client's record, as already established in Client Record Management (FEAT-11).

**Communications:** A confirmation is sent to Mara when closure completes; any clients with future bookings at the time of closure are notified their upcoming appointments are cancelled and any deposits are refunded.

**Data Notes:** Captured: closure request and timestamp. Derived: what is deleted versus retained is determined by financial record-keeping needs (e.g., payment records may be retained in minimal form) versus personal data (deleted). Source: Mara's request.

**Interactions:** Depends on Pro Account & Authentication (FEAT-14), Pro Subscription & Billing (FEAT-17), and Client Record Management (FEAT-11).

**Signals:** account_closure_requested, account_closure_completed, account_closure_cancelled, client_record_deleted.

---

### Operator Support Console

**ID:** FEAT-21

**Description:** A narrow, read-only view the founder uses to see exactly what a specific Pro sees — their setup and their bookings — in order to help when that Pro reports a problem, without ever acting on the Pro's behalf.

**Priority:** Important

**Phase:** v1

**Type:** Platform

**Rationale:** The brief confirms this actor and its read-only scope explicitly (BRIEF.md, Target Users & Roles). Ranked Important rather than Core because it supports the founder's ability to help Pros, rather than being part of any Pro's or Client's own value loop. Phased to v1 rather than MVP: with only a handful of Pros in the earliest weeks (drawn from the founder's own network per BRIEF.md, Business Context), direct, ad hoc troubleshooting is workable briefly, but a proper read-only console becomes necessary as soon as the Pro base grows past a size the founder can track personally. [INFERRED: carried from Visionary draft]

**Connected Entities:** Pro Profile (read), Booking (read), Deposit/Payment Record (read)

**Key Capabilities:**
- Look up a Pro's setup -- Operator sees a specific Pro's services, policy, and hours, read-only
- Look up a Pro's bookings -- Operator sees that Pro's bookings and their statuses, read-only
- View the dispute trail -- Operator sees the same booking history trail Mara would see, for the reported booking only
- Operator actions are logged -- Every read-only lookup the Operator performs (which Pro, which booking, when) is itself recorded, so the practice remains fully accountable to its own read-only, single-Pro-scoped promise [AUDIT-ADDED: 4 -- Cross-Cutting Concerns Verification (completeness-audit.md Section 4, Audit Logging) found this concern was not mentioned anywhere in the draft; the Operator's own access needed a record for accountability, directly serving BRIEF.md's confirmation that this actor must "never see more of client data than the pro's own screens show" (BRIEF.md, Target Users & Roles)]

**Primary Flows & Alternates:**
- Happy path: a Pro reports an issue -> Operator opens that Pro's read-only view -> reviews setup, bookings, or a specific booking's history -> diagnoses the issue without changing anything
- Attempted action: the console has no controls that modify data — there is nothing to attempt beyond viewing, by design

**States:** Empty: N/A — the Operator only opens a Pro's view when there is something to look up. Loading: same brief indicator as other read views. Error: a failed load shows a retry option. Offline-degraded: N/A — this is an internal tool used at a desk, not in the field.

**Validation & Limits:** Strictly read-only — no create, update, or delete action exists anywhere in this console; access is scoped to one Pro's data at a time, never a cross-Pro view.

**Access:** Only the Operator (Full, of this console specifically) can use it; Mara and Taylor have no access to it at all — it is not part of either persona's product experience.

**Communications:** N/A — this is an internal viewing tool with no messages of its own.

**Data Notes:** Displayed: the same data Mara's own screens would show her, read-only. Source: read directly from existing Pro, Booking, and Deposit/Payment Record data — no new data is captured. Operator lookups themselves are logged with a timestamp, the Operator's identity, and the Pro/booking scope accessed, for accountability [AUDIT-ADDED: 4].

**Interactions:** Reads data written by Business Settings & Policy Configuration (FEAT-09), Pro Daily Dashboard (FEAT-07), and Booking Record & Dispute Trail (FEAT-18).

**Signals:** operator_lookup_performed, operator_lookup_logged [AUDIT-ADDED: 4].

---

## Nice-to-Have Features

### Client List Import

**ID:** FEAT-22

**Description:** Mara can bring in her existing client list (names and phone numbers she already has from Instagram DMs or elsewhere) in bulk, so her client history is not starting from zero on day one.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** [INFERRED from: domain knowledge — the brief describes Mara currently running her business "out of Instagram DMs" with an existing base of "roughly 100–500 clients" (BRIEF.md, Vision; Scale & Non-Functional Expectations); a domain-standard bulk-import convenience meaningfully lowers the switching cost from her current DM-based system.] Nice-to-Have because Client Record Management (FEAT-11) already builds the list organically from first bookings — import is a convenience, not a requirement. Phased Later: it addresses a one-time migration moment, not the recurring value loop. [INFERRED: carried from Visionary draft]

**Connected Entities:** Client Record (create, in bulk)

**Key Capabilities:**
- Bulk-add clients -- Mara adds multiple existing clients' names and phone numbers at once

**Primary Flows & Alternates:**
- Happy path: Mara provides a list of existing clients' names and phone numbers -> records are created in bulk -> they appear in Client Record Management going forward
- Duplicate detection: an imported entry matching an existing client (by phone number) updates rather than duplicates that record

**States:** Empty: N/A — this is a one-time bulk action, not a persistent list view. Loading: import shows progress for larger lists. Error: entries that fail to import (e.g., invalid phone format) are reported individually so valid entries still succeed. Offline-degraded: import requires connectivity.

**Validation & Limits:** Each imported entry requires at minimum a name and a valid phone number; a reasonable batch-size limit per import applies to keep processing predictable.

**Access:** Only Mara (Full, own clients only) can import; there is no client- or operator-facing surface.

**Communications:** N/A — importing existing clients does not itself message them.

**Data Notes:** Captured: name and phone number per imported entry. Source: entirely Mara's own provided list.

**Interactions:** Feeds Client Record Management (FEAT-11).

**Signals:** client_import_started, client_import_completed, client_import_row_failed.

---

### Cancellation Waitlist

**ID:** FEAT-23

**Description:** A client who wants an earlier time than what's currently free can ask to be notified if a slot opens up from a cancellation, instead of repeatedly checking back.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** The brief raises this directly as an open question: "should a pro be able to offer a waitlist for slots that open up from cancellations?" (BRIEF.md, Open Questions). Nice-to-Have because the core booking loop closes fully without it — a client can always book the next genuinely free slot. Phased Later: it adds meaningful scheduling complexity that is better tackled once the core loop is proven with real bookings. [INFERRED: carried from Visionary draft]

**Connected Entities:** Waitlist Entry (create, update)

**Key Capabilities:**
- Join a waitlist -- Client requests notification if an earlier slot opens for a chosen service and date range
- Get notified and book -- Client is notified when a matching slot opens and can claim it quickly

**Primary Flows & Alternates:**
- Happy path: client sees no slot they want -> joins the waitlist for a service and date range -> a matching cancellation occurs -> client is notified -> claims the newly-open slot through the normal booking flow
- Slot claimed by someone else first: if the notified client does not claim the slot in time, it becomes normally bookable and the waitlist entry simply expires without penalty

**States:** Empty: a client with no waitlist entries sees nothing to manage — the join action is available wherever no matching slot exists. Loading: joining shows a brief confirmation. Error: a failed join preserves the client's chosen criteria for retry. Offline-degraded: requires connectivity to join or be notified.

**Validation & Limits:** A waitlist entry has an expiration (it does not wait forever); a client can hold a reasonable number of active waitlist entries at once.

**Access:** Any Client (Own-only) manages only their own waitlist entries; Mara has no separate waitlist-management view — releases happen automatically through cancellations.

**Communications:** A notification (matching the client's consented channel) is sent when a matching slot opens.

**Data Notes:** Captured: service, date range, and client identity for the waitlist request. Source: client's own input.

**Interactions:** Depends on Live Availability & Slot Booking (FEAT-02) for slot matching and Client Self-Service Reschedule & Cancellation (FEAT-06) as the source of released slots.

**Signals:** waitlist_joined, waitlist_notified, waitlist_slot_claimed, waitlist_entry_expired.

---

### In-App Tipping at Checkout

**ID:** FEAT-24

**Description:** A client can optionally add a tip on top of their deposit at checkout, rather than needing cash at the chair.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** The brief raises this directly as an open question: "where does tipping fit, if anywhere?" (BRIEF.md, Open Questions). Nice-to-Have because tipping is not part of the deposit/no-show protection loop the brief centers the product around. Phased Later: it depends on In-App Balance Payment groundwork being reasonably mature and is a genuine "nice to have" rather than something the earliest Pros are asking for. [INFERRED: carried from Visionary draft]

**Connected Entities:** Tip (create)

**Key Capabilities:**
- Add an optional tip -- Client can add a tip amount at checkout, entirely optional

**Primary Flows & Alternates:**
- Happy path: at deposit checkout, client is shown an optional tip prompt -> adds an amount or skips it -> tip (if any) is included in the same payment
- Skip: client proceeds without adding a tip, with no friction or repeated prompting

**States:** Empty: N/A — this is an optional add-on at a single checkout moment. Loading: handled within the same checkout loading state as FEAT-04. Error: handled within the same checkout error state as FEAT-04. Offline-degraded: N/A — requires connectivity as part of checkout.

**Validation & Limits:** Tip amount must be zero or a positive value; it cannot exceed a sensible upper bound relative to the service price, to guard against accidental entry.

**Access:** Any Client (Own-only) can add a tip to their own booking's checkout; Mara cannot add or edit a tip on a client's behalf.

**Communications:** N/A — a tip is reflected in the existing payment confirmation rather than triggering a separate message.

**Data Notes:** Captured: tip amount, if any. Source: client's own input at checkout.

**Interactions:** Depends on Deposit Payment at Booking (FEAT-04) and Payment Processing Capability (FEAT-15); reflected in Pro Daily Dashboard (FEAT-07) and Simple Business Insights (FEAT-27).

**Signals:** tip_prompted, tip_added, tip_skipped.

---

### In-App Balance Payment

**ID:** FEAT-25

**Description:** Instead of settling the remaining balance in person, a client can optionally pay it in-app before or at the appointment.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** The brief raises this directly as an open question: "should the balance after the deposit be payable in the app, or does it stay in person?" (BRIEF.md, Open Questions; Business Context). Nice-to-Have because the brief's own default assumption is in-person settlement, which the product already supports today by simply showing the balance due on the Pro Daily Dashboard (FEAT-07) — see scope-boundaries.md for the explicit value-flow statement of this default. Phased Later: it is a genuine option to build once real usage shows Pros or clients want it, not a day-one requirement. [INFERRED: carried from Visionary draft]

**Connected Entities:** Deposit/Payment Record (update, with balance payment)

**Key Capabilities:**
- Pay the remaining balance in-app -- Client pays the balance due before or at the appointment instead of in person

**Primary Flows & Alternates:**
- Happy path: client opens their upcoming booking -> sees the balance due -> pays it in-app -> Pro Daily Dashboard reflects the booking as fully paid
- Partial timing: a client can pay the balance any time between booking and the appointment; paying it does not change the appointment time or service

**States:** Empty: N/A — this acts on a specific existing booking's balance. Loading: same brief confirmation as other payment actions. Error: a failed balance payment leaves the booking's prior paid status unchanged. Offline-degraded: requires connectivity.

**Validation & Limits:** The amount payable is exactly the remaining balance (service price minus deposit already paid); it cannot be paid twice.

**Access:** Any Client (Own-only) can pay only their own booking's balance; Mara sees the resulting fully-paid status but does not collect it directly for balances paid this way.

**Communications:** A payment confirmation is sent to the client when the balance is paid.

**Data Notes:** Captured: balance payment amount and timestamp. Derived: remaining balance, computed as service price minus deposit paid. Source: Payment Processing Capability (FEAT-15) reports the outcome.

**Interactions:** Depends on Payment Processing Capability (FEAT-15) and Deposit Payment at Booking (FEAT-04); reflected in Pro Daily Dashboard (FEAT-07).

**Signals:** balance_payment_started, balance_payment_succeeded, balance_payment_failed.

---

### Recurring Appointment Booking

**ID:** FEAT-26

**Description:** A client who sees the same Pro on a regular schedule (e.g., "every 3 weeks") can set up a standing series of appointments instead of booking each one individually.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** The brief raises this directly as an open question: "recurring / standing appointments... v1 or later?" (BRIEF.md, Open Questions). Nice-to-Have because a single booking at a time already fully closes the core value loop the brief centers on (deposit-protected, no-DM booking). Phased Later, resolving the brief's own open question in favor of not blocking MVP: recurrence adds real scheduling complexity (handling a series when one occurrence is rescheduled, cancelled, or a no-show) that is safer to design once the single-booking loop is proven in production. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (create, in a linked series)

**Key Capabilities:**
- Set up a recurring series -- Client books a repeating cadence (e.g., every 3 weeks) for a service
- Manage one occurrence independently -- Client can reschedule or cancel a single occurrence without affecting the rest of the series

**Primary Flows & Alternates:**
- Happy path: client selects a recurring cadence during booking -> a series of linked bookings is created against genuinely free slots at that cadence -> each occurrence is confirmed and deposited individually
- Occurrence unavailable: if a future occurrence's usual slot is not free (e.g., a conflict has appeared), the client is asked to pick an alternate time for that occurrence only, without breaking the rest of the series

**States:** Empty: N/A — this extends the existing booking flow rather than introducing a new list. Loading: same as Live Availability & Slot Booking for each occurrence. Error: a failure to schedule one occurrence does not roll back already-confirmed occurrences in the series. Offline-degraded: requires connectivity, same as standard booking.

**Validation & Limits:** Cadence must be a supported recurrence pattern (e.g., every N weeks); a series has a maximum number of pre-scheduled future occurrences to keep availability commitments realistic.

**Access:** Any Client (Own-only) manages only their own recurring series; Mara sees each occurrence on her dashboard exactly as any other booking.

**Communications:** Confirmation and reminder messages are sent per occurrence, same as a standalone booking.

**Data Notes:** Captured: recurrence cadence and the linked series identifier. Source: client's own input at booking time.

**Interactions:** Depends on Live Availability & Slot Booking (FEAT-02) and Deposit Payment at Booking (FEAT-04) for each occurrence.

**Signals:** recurring_series_created, recurring_occurrence_rescheduled, recurring_occurrence_cancelled, recurring_series_ended.

---

### Simple Business Insights

**ID:** FEAT-27

**Description:** A simple, plain-language snapshot of how the business is doing — bookings this week, no-show rate, and deposits (and tips, if enabled) collected — so Mara can see her own success without doing any math.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** [INFERRED from: domain knowledge — the brief's own Success Criteria include Mara being able to say "I haven't had an unpaid no-show since I switched" (BRIEF.md, Success Criteria); a simple insights view gives her the evidence for that feeling rather than requiring her to remember it.] Nice-to-Have because the Pro Daily Dashboard already delivers the operational, day-to-day value without any summary view. Phased Later: it is only meaningful once a Pro has accumulated enough bookings to summarize. [INFERRED: carried from Visionary draft]

**Connected Entities:** Booking (read), Deposit/Payment Record (read), Tip (read)

**Key Capabilities:**
- View a weekly snapshot -- Mara sees bookings, no-shows, and money collected for the current week
- See the no-show rate -- Mara sees what share of bookings resulted in a no-show

**Primary Flows & Alternates:**
- Happy path: Mara opens the insights view -> sees a plain-language summary of the current week -> can page back to prior weeks
- Insufficient history: a new Pro with very few bookings sees an encouraging "still gathering data" message rather than a misleadingly precise-looking statistic from a tiny sample

**States:** Empty: a Pro with no bookings yet sees "not enough data yet." Loading: summary renders with a brief indicator. Error: a failed summary load shows the last successfully computed week with a retry option. Offline-degraded: the last viewed week remains available read-only.

**Validation & Limits:** The summary window is a rolling week by default; no user input is captured in this view beyond week navigation.

**Access:** Only Mara (Full) sees her own business insights; there is no client- or operator-facing equivalent.

**Communications:** N/A — this is a self-initiated viewing feature with no messages of its own.

**Data Notes:** Displayed: weekly booking count, no-show rate, deposits and tips collected. Derived: entirely computed from Booking, Deposit/Payment Record, and Tip data — no new data captured here.

**Interactions:** Depends on No-Show & Cancellation Deposit Handling (FEAT-08), Deposit Payment at Booking (FEAT-04), and In-App Tipping at Checkout (FEAT-24).

**Signals:** insights_viewed, insights_week_navigated.

---

### Data Export

**ID:** FEAT-28

**Description:** Mara can export her client list and booking history for her own records outside the product.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** User-Facing

**Rationale:** [INFERRED from: domain knowledge — a solo business owner reasonably wants her own client and booking data portable for her own records, independent of any single tool she uses.] Nice-to-Have because nothing in the brief's stated success criteria depends on export. Phased Later: it is a data-portability convenience, not part of the core loop. [INFERRED: carried from Visionary draft]

**Connected Entities:** Client Record (read), Booking (read)

**Key Capabilities:**
- Export client list -- Mara downloads her client records for her own use
- Export booking history -- Mara downloads her booking history for her own use

**Primary Flows & Alternates:**
- Happy path: Mara requests an export -> a downloadable file of her client list and/or booking history is produced
- Large history: for a Pro with several years of history, the export is prepared and made available shortly after the request rather than blocking the screen until complete

**States:** Empty: a Pro with no clients or bookings yet sees a plain notice that there is nothing to export. Loading: export preparation shows a visible "preparing your export" state for larger datasets. Error: a failed export offers a retry without losing the request. Offline-degraded: requires connectivity to request or retrieve an export.

**Validation & Limits:** An export contains only that Pro's own data — never another Pro's; Mara can request a fresh export at any time without limit beyond reasonable rate protection.

**Access:** Only Mara (Full, own clients only) can export her own data; no other role has access to this feature.

**Communications:** N/A directly, though a notice may indicate the export is ready if preparation takes more than a moment.

**Data Notes:** Displayed/exported: client names, phone numbers, notes, and booking history. Source: read from Client Record and Booking; nothing new is captured here.

**Interactions:** Depends on Client Record Management (FEAT-11) and Pro Daily Dashboard (FEAT-07) data.

**Signals:** export_requested, export_ready, export_downloaded.

---

### WhatsApp Messaging Channel

**ID:** FEAT-29

**Description:** An additional channel — alongside text messages — for sending confirmations and reminders, for clients who prefer it.

**Priority:** Nice-to-Have

**Phase:** Later

**Type:** Platform

**Rationale:** The brief names this directly: "WhatsApp is a later nice-to-have, not v1" (BRIEF.md, Ecosystem & Integrations). This is a discovered, brief-named capability and is documented in full here rather than only as a scope note, per the requirement that nothing the brief raises is silently folded into an exclusion. [INFERRED: carried from Visionary draft]

**Connected Entities:** Messaging Consent Record (update, to record channel preference)

**Key Capabilities:**
- Choose a preferred channel -- Client can opt to receive confirmations and reminders via this channel instead of text
- Deliver via the alternate channel -- Confirmations and reminders are sent through this channel when chosen

**Primary Flows & Alternates:**
- Happy path: client indicates a channel preference at booking -> confirmations and reminders are delivered via that channel instead of text going forward
- Channel unavailable for a given client: if delivery via this channel is not possible for a client, the product falls back to text messaging rather than silently failing to notify them

**States:** Empty: N/A — this extends existing messaging rather than introducing a new list. Loading: N/A — handled within the invoking feature (FEAT-05). Error: a delivery failure on this channel falls back to text messaging. Offline-degraded: N/A — requires connectivity, same as any messaging.

**Validation & Limits:** A client's channel preference applies only to their own future messages; consent rules from SMS Messaging & Consent Capability (FEAT-16) apply equally to this channel, including the STOP-equivalent opt-out mechanism.

**Access:** Any Client (Own-only) can set their own channel preference; Mara has no separate control over which channel is used for a given client.

**Communications:** This feature is itself a communications channel: confirmations and reminders delivered via this channel instead of text, when chosen.

**Data Notes:** Captured: channel preference. Source: client's own input.

**Interactions:** Extends SMS Messaging & Consent Capability (FEAT-16); used by Booking Confirmation & Reminders (FEAT-05).

**Signals:** channel_preference_set, message_delivered_alternate_channel, message_fallback_to_sms.

---

## Feature Interaction Summary

| Feature | Depends On |
|---------|------------|
| FEAT-01 Public Booking Page | FEAT-09 (services/policy display), FEAT-02 (availability) |
| FEAT-02 Live Availability & Slot Booking | FEAT-09 (hours/buffer), FEAT-10 (blocked time), FEAT-12 (calendar busy time), FEAT-23 (waitlist release) |
| FEAT-03 Client Identity & Booking Details Capture | FEAT-02 (held slot), FEAT-16 (verification code delivery) |
| FEAT-04 Deposit Payment at Booking | FEAT-03 (booking details), FEAT-15 (payment capability), FEAT-09 (deposit amount) |
| FEAT-05 Booking Confirmation & Reminders | FEAT-04 (confirmed booking), FEAT-16 (delivery), FEAT-29 (alternate channel) |
| FEAT-06 Client Self-Service Reschedule & Cancellation | FEAT-05 (reminder link), FEAT-02 (new slot), FEAT-09 (policy window) |
| FEAT-07 Pro Daily Dashboard | FEAT-04 (paid status), FEAT-11 (client notes), FEAT-08 (no-show/cancel status), FEAT-24 (tip amount), FEAT-02 (new slot for Pro-initiated reschedule) [AUDIT-ADDED: 1] |
| FEAT-08 No-Show & Cancellation Deposit Handling | FEAT-04 (deposit record), FEAT-09 (policy), FEAT-18 (writes dispute trail) |
| FEAT-09 Business Settings & Policy Configuration | None |
| FEAT-10 Pro Manual Schedule Blocking | FEAT-02 (affects availability) |
| FEAT-11 Client Record Management | FEAT-03 (client creation), FEAT-22 (import) |
| FEAT-12 Two-Way Calendar Sync | FEAT-14 (pro account) |
| FEAT-13 Pro Onboarding & Setup | FEAT-14, FEAT-09, FEAT-12 (optional), FEAT-17 |
| FEAT-14 Pro Account & Authentication | None |
| FEAT-15 Payment Processing Capability | None |
| FEAT-16 SMS Messaging & Consent Capability | None |
| FEAT-17 Pro Subscription & Billing | FEAT-14, FEAT-15 |
| FEAT-18 Booking Record & Dispute Trail | FEAT-04, FEAT-08, FEAT-07 (Pro-initiated changes) [AUDIT-ADDED: 1] |
| FEAT-19 Client Self-Service Booking History | FEAT-03 (identity), FEAT-04 (booking data) |
| FEAT-20 Account Closure & Client Data Deletion | FEAT-11, FEAT-14, FEAT-17 |
| FEAT-21 Operator Support Console | FEAT-07, FEAT-18 |
| FEAT-22 Client List Import | FEAT-11 |
| FEAT-23 Cancellation Waitlist | FEAT-02, FEAT-06 |
| FEAT-24 In-App Tipping at Checkout | FEAT-04, FEAT-15 |
| FEAT-25 In-App Balance Payment | FEAT-15, FEAT-07 |
| FEAT-26 Recurring Appointment Booking | FEAT-02, FEAT-04 |
| FEAT-27 Simple Business Insights | FEAT-04, FEAT-08, FEAT-24 |
| FEAT-28 Data Export | FEAT-11, FEAT-07 |
| FEAT-29 WhatsApp Messaging Channel | FEAT-16, FEAT-05 |
</content>

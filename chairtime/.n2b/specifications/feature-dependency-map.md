---
document_type: feature-dependency-map
produced_by: requirements-architect
status: final
created: 2026-09-26
feature_count: 30
shared_entity_count: 18
cross_feature_rule_count: 29
navigation_connection_count: 46
external_touchpoint_count: 9
---

# Feature Dependency Map

Chairtime is a single-operator booking-and-deposit product for one solo beauty or wellness professional (BRIEF.md, Vision). Two product roles exist — the Pro and the Client — plus the narrow, read-only Platform Operator (Support) function (user-persona.md, Access Matrix). Every role named below traces to that matrix. Feature numbers, names, priorities, phases and types are carried unchanged from product-features.md; "Depends On" is carried from its Feature Interaction Summary and "Depended On By" is its inverse.

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

## Shared Data Entities

All 18 entities in the Domain Entity Inventory are referenced by two or more features, so every one appears below.

### Pro Account

- **Lifecycle:** Created by FEAT-15 (sign-in identity established by FEAT-29). Read by FEAT-01, FEAT-02, FEAT-05, FEAT-08, FEAT-19. Updated by FEAT-27 (profile, link name, timezone, currency, pause, notification preferences), FEAT-29 (sign-in contacts, devices, closure) and FEAT-18 (paused on subscription lapse). Deleted by FEAT-29 (after a 30-day cooling-off period).
- **Fields (functional):**
  - sign_in_email / sign_in_mobile -- both required; each change confirmed through old and new contact (FEAT-29)
  - display_name -- required, 1–60 characters, public
  - photo -- optional standard image under a size limit, public
  - intro -- optional, up to 300 characters, public
  - general_area -- public area shown on the booking page
  - studio_address -- required before go-live; shown only in booked clients' confirmations and reminders
  - booking_link_name -- 3–40 letters, numbers or hyphens, unique across all pros; previous names forward for at least 12 months
  - timezone -- required, per account, never hard-coded
  - currency -- required, per account; locked once the first deposit is taken
  - status -- Active | Paused (subscription lapse or Pro-chosen pause, with optional message and end date) | Closing (cooling-off) | Closed
  - notification_preferences -- which Pro notifications arrive in-app, by text or by email
  - signed_in_devices -- list of active sign-ins (up to 30 days of inactivity each)
- **Relationships:** The root of every other record: one Pro Account owns its Services, Availability Rules, Time Blocks, Calendar Connection(s), Clients, Bookings, Cancellation Policy versions, Subscription and Payout Account. No record is ever shared between two Pro Accounts.
- **Contention:** Only the Pro (Access Matrix: Profile & Account Settings = Full) edits the account, possibly from two signed-in devices at once, while FEAT-18 can set a Paused state automatically on a failed renewal. Resolution: last-write-wins for ordinary profile and preference fields; reject-with-refresh when a booking link name was claimed by another pro in the meantime or when currency has become locked by a first deposit; a system-imposed subscription pause cannot be cleared by the Pro's "resume bookings" toggle until billing is restored. Platform Operator (Support) is View-only and never writes.
- **Data Sensitivity:** Personal data of the Pro — sign-in email and mobile number, and a full studio address that may be a home address (hence confirmation-only disclosure). Protected by one-time-code sign-in with new-device alerts (assumptions-constraints.md ASMP-30); support sees account status only, never sign-in codes (ASMP-20). Public profile fields are intentionally public.
- **Source:** Domain Entity Inventory, product-features.md

### Service

- **Lifecycle:** Created by FEAT-01 and FEAT-15 (first service during setup). Read by FEAT-02, FEAT-03, FEAT-05, FEAT-07, FEAT-19, FEAT-20, FEAT-25. Updated by FEAT-01 (name, price, duration, deposit rule, display order, status) and FEAT-02 (buffer_override only, via its per-service buffer override screen). Archived by FEAT-01 (never hard-deleted while bookings reference it).
- **Fields (functional):**
  - name -- required, 1–80 characters
  - price -- required, positive amount in the Pro's account currency
  - duration -- required, 5-minute steps, up to 12 hours
  - deposit_rule -- required; fixed amount no greater than the price, or 1–100% of price; resulting deposit at least the smallest chargeable card amount
  - buffer_override -- optional per-service buffer, 0 to 120 minutes; written only by FEAT-02, read by FEAT-03
  - display_order -- position on the booking page
  - status -- Active | Archived
- **Relationships:** Belongs to one Pro Account; referenced by many Bookings (each Booking keeps the price and deposit agreed at booking time) and by Waitlist Entries.
- **Contention:** Only the Pro edits services (Access Matrix: Service & Availability Setup = Full), through two features writing disjoint fields — FEAT-01 (all fields except buffer_override) and FEAT-02 (buffer_override only) — so concurrent saves from the two screens resolve field by field with no overlap, while Clients concurrently read them on the booking page (FEAT-05) and FEAT-07 computes deposits from them. Resolution: last-write-wins between the Pro's own sessions; confirmed bookings are never changed by an edit (price and deposit are fixed at booking); a client mid-checkout pays the amounts shown when they acknowledged the policy, and if the service is archived before payment the client is refused with refresh back to the service list.
- **Data Sensitivity:** None — service names, prices and durations are public commercial information shown on the booking page.
- **Source:** Domain Entity Inventory, product-features.md

### Availability Rule

- **Lifecycle:** Created by FEAT-02 and FEAT-15. Read by FEAT-03 (and via it FEAT-05, FEAT-10, FEAT-20, FEAT-21, FEAT-30). Updated by FEAT-02 (versioned by effective date). Deleted: N/A — superseded by a newer version, never removed while past bookings reference it.
- **Fields (functional):**
  - weekly_windows -- per day of week, multiple non-overlapping windows allowed; start before end; interpreted in the Pro's timezone
  - default_buffer -- 0 to 120 minutes
  - per_service_buffer -- not held on this entity: the per-service override is stored as Service.buffer_override (written by FEAT-02) and combined with this rule's default_buffer by FEAT-03
  - minimum_booking_notice -- 0 to 7 days, default a few hours
  - booking_horizon -- 1 week to 12 months, default 8 weeks
  - effective_from -- date from which this version applies
- **Relationships:** Belongs to one Pro Account; combined with Time Blocks, Bookings, Calendar Connection busy time and Recurring Series by FEAT-03 to compute open slots.
- **Contention:** Only the Pro edits (Service & Availability Setup = Full); clients concurrently read computed slots. Resolution: last-write-wins between the Pro's sessions; a slot held under the previous version is re-validated at confirmation and refused with a refreshed slot list if it no longer fits; already-confirmed bookings outside new hours are never cancelled — they stay and are flagged to the Pro (XBR-11).
- **Data Sensitivity:** Low — the Pro's working pattern is private to the Pro; clients see only the resulting open times, never the rule itself.
- **Source:** Domain Entity Inventory, product-features.md

### Time Block

- **Lifecycle:** Created by FEAT-17. Read by FEAT-03 and FEAT-12 (shown on the schedule). Updated by FEAT-17. Deleted by FEAT-17 (or expires once its time has passed).
- **Fields (functional):**
  - start / end -- required; end after start; Pro's timezone
  - recurrence -- optional pattern (e.g., every Sunday)
  - label -- optional, private to the Pro
- **Relationships:** Belongs to one Pro Account; removes availability in FEAT-03; may overlap confirmed Bookings, which are then handed to FEAT-30 by the Pro's explicit choice.
- **Contention:** The Pro creates a block (Service & Availability Setup = Full) while a Client may simultaneously be paying for a slot inside it (FEAT-05/FEAT-07). Resolution: first committed wins — a slot already held or confirmed becomes a conflict the Pro must explicitly decide (cancel, reschedule or keep as an exception, XBR-11); a block committed first removes the slot and the client's confirmation is refused with a refreshed slot list. Block edits between Pro sessions are last-write-wins.
- **Data Sensitivity:** Private to the Pro — labels can describe personal commitments (e.g., a doctor's appointment) and are never shown to clients; support sees them View-only.
- **Source:** Domain Entity Inventory, product-features.md

### Calendar Connection

- **Lifecycle:** Created by FEAT-04 (offered in FEAT-15). Read by FEAT-03 and FEAT-12 (health banner). Updated by FEAT-04 (sync health changes, reconnect). Deleted by FEAT-04 (disconnect).
- **Fields (functional):**
  - calendar_kind -- one of the two supported personal-calendar kinds; at most one connection per kind
  - status -- Connected | Syncing | Needs Reconnection | Disconnected
  - last_successful_sync -- time of last good sync in each direction
  - busy_periods -- minimum busy/free data needed to block slots (never full event details)
- **Relationships:** Belongs to one Pro Account; feeds busy time into FEAT-03 and receives Chairtime bookings created or changed by FEAT-05, FEAT-10, FEAT-11 and FEAT-30.
- **Contention:** The Pro connects or disconnects (Service & Availability Setup = Full) while automatic sync updates health status. Resolution: a Pro disconnect wins over any in-flight sync; health-status updates are last-write-wins; bookings already written to the personal calendar are reconciled on reconnect.
- **Data Sensitivity:** Sensitive — the connection grants access to the Pro's personal calendar; only busy/free periods are kept, never event titles or details (product-features.md FEAT-04 Data Notes). Support sees connection health only.
- **Source:** Domain Entity Inventory, product-features.md

### Client

- **Lifecycle:** Created by FEAT-05 (first booking) and FEAT-30 (Pro books a client in). Read by FEAT-06, FEAT-12, FEAT-19 (excluding private notes), FEAT-24, FEAT-29 (export). Updated by FEAT-13 (Pro edits contact details and notes) and FEAT-06 (client updates own email and consent). Deleted by FEAT-13 (hard delete of contact details and notes on request; de-identified financial history retained).
- **Fields (functional):**
  - name -- required, 1–100 characters
  - phone -- required, valid reachable format; identity key within one Pro
  - email -- required when texts are declined, otherwise optional
  - private_note -- Pro-only, up to 1,000 characters
  - booking_notes -- the client's optional note per booking, up to 300 characters, no medical information
  - booking_history -- derived list of this client's Bookings with this Pro only
- **Relationships:** Belongs to exactly one Pro Account (a person booking two pros has two unconnected records, scope-boundaries SC-04); has many Bookings, one Messaging Consent per channel, Access Links and Waitlist Entries.
- **Contention:** The Pro (Client Records = Full) can edit contact details or notes while the Client (Own-only through FEAT-06) updates their own email, and two first bookings with the same phone can arrive close together. Resolution: phone-number match within a Pro resolves to a single Client record (merge, never a duplicate); field edits are last-write-wins; a Pro phone-number change invalidates access links and requires fresh texting consent; deletion is refused while an upcoming booking exists, and once deleted any concurrent edit is refused with refresh — a later booking creates a new record, never resurrects the old one.
- **Data Sensitivity:** Personal data — name, phone, email and the Pro's private notes; visible only to the Pro and (for their own details and bookings) the Client (ASMP-23); support never sees private notes; deletable on request; no health data is captured (SC-08).
- **Source:** Domain Entity Inventory, product-features.md

### Booking

- **Lifecycle:** Created by FEAT-05, FEAT-30 and FEAT-21. Read by FEAT-03, FEAT-04, FEAT-06, FEAT-08, FEAT-09, FEAT-16, FEAT-19, FEAT-20, FEAT-25, FEAT-29. Updated by FEAT-07 (pending → confirmed), FEAT-10, FEAT-11, FEAT-12 (completed, auto-completed), FEAT-22 (balance status), FEAT-30. Deleted: N/A — bookings are kept for the life of the account (SC-22); cancelled and expired bookings remain as history.
- **Fields (functional):**
  - service, start_time (Pro's timezone), duration -- fixed at booking
  - client -- required reference
  - price_agreed / deposit_amount -- fixed at booking
  - policy_version -- the cancellation policy shown and acknowledged, with acknowledgment time
  - state -- Pending Payment | Confirmed | Awaiting Outcome | Completed | No-Show | Cancelled by Client | Cancelled by Pro | Rescheduled | Expired (unpaid)
  - attendance_reply -- "I'll be there" / reschedule requested, from reminders
  - balance_due -- derived: price − deposit − any in-app balance payment
  - source -- client link, Pro booked-in, recurring occurrence
  - cancellation / reschedule timestamps and optional private Pro reason
- **Relationships:** Belongs to one Pro Account and one Client; references one Service and one Cancellation Policy version; has one Deposit Transaction, zero or one Balance Payment, many Messages and many Activity Events; may belong to a Recurring Series.
- **Contention:** High. The Client (Cancellation & No-Show Handling = Own-only) may cancel or reschedule via FEAT-10 while the Pro (Booking & Payment = Full) cancels, reschedules, marks no-show or completes via FEAT-30/FEAT-11/FEAT-12, and automations expire holds and auto-complete. Resolution: reject-with-refresh — the first committed state transition wins, and the other actor is shown the current state and must re-decide; transitions are never merged, and every transition is validated against the current state (e.g., a completed or no-show booking cannot be cancelled).
- **Data Sensitivity:** Personal data linked to an identifiable client (appointment time, service, client note); visible only to the Pro and that Client (ASMP-23); support views read-only.
- **Source:** Domain Entity Inventory, product-features.md

### Deposit Transaction

- **Lifecycle:** Created by FEAT-07. Read by FEAT-10 (outcome preview), FEAT-12, FEAT-16, FEAT-22, FEAT-25, FEAT-28, FEAT-29 (export). Updated by FEAT-09 (automatic refund or forfeit flag), FEAT-11 (forfeit, undo), FEAT-30 (Pro refund); Disputed state set on card-issuer dispute notice. Deleted: N/A — financial records are retained, de-identified after client deletion or account closure (SC-22).
- **Fields (functional):**
  - amount / currency -- computed once from the Service's rule
  - status -- Authorized | Captured | Applied | Refunded | Refund in Progress | Forfeited | Disputed
  - processor_fee -- the payment processor's own card fee (Chairtime fee always zero)
  - outcome_reason / timestamps -- which rule or action produced the outcome
- **Relationships:** Belongs to one Booking; paid out to the Pro's Payout Account; listed in the money list (FEAT-28).
- **Contention:** The Pro (Cancellation & No-Show Handling = Full) may mark a no-show or issue a goodwill refund while FEAT-09 applies an automatic refund from a Client cancellation, and dispute notices can arrive at any time. Resolution: reject-with-refresh — one terminal outcome per deposit; the first committed transition wins and a conflicting second action is refused with the current status shown; a deposit can be refunded only once; a Disputed overlay never erases the underlying outcome.
- **Data Sensitivity:** Financial record — amounts and outcomes only; card data is never held (ASMP-15, SC-11); retained de-identified after deletion for legal and dispute purposes.
- **Source:** Domain Entity Inventory, product-features.md

### Cancellation Policy

- **Lifecycle:** Created by FEAT-15. Read by FEAT-05, FEAT-10, FEAT-11, FEAT-16, FEAT-30. Updated by FEAT-09 (each edit creates a new version). Deleted: N/A — versions are kept while any booking references them.
- **Fields (functional):**
  - window_hours -- whole hours, 1–168
  - inside_window_outcome -- deposit kept (binary in v1)
  - outside_window_outcome -- full refund
  - plain_language_wording -- the text shown to clients
  - version / effective_from
- **Relationships:** Belongs to one Pro Account; each Booking points at the version the client acknowledged.
- **Contention:** Only the Pro edits (Cancellation & No-Show Handling = Full); clients read it during booking. Resolution: every edit creates a new version (no overwrite); if the version changes between a client's acknowledgment and payment, the client is refused with refresh and asked to acknowledge the current wording; existing bookings keep their version.
- **Data Sensitivity:** None — the policy is public terms shown to every client on the booking page.
- **Source:** Domain Entity Inventory, product-features.md

### Messaging Consent

- **Lifecycle:** Created by FEAT-05 (captured at booking). Read by FEAT-06, FEAT-08, FEAT-12 (Pro sees textability), FEAT-20, FEAT-26, FEAT-30. Updated by FEAT-14 (revoke via STOP or opt-out link) and FEAT-06 (client re-grants). Deleted by FEAT-13 only as part of client deletion (evidence retained where law requires).
- **Fields (functional):**
  - channel -- text (WhatsApp from Later)
  - state -- Granted | Revoked | Re-granted
  - timestamp and exact consent wording shown -- kept as evidence
  - phone_number -- the number the consent applies to
- **Relationships:** Belongs to one Client with one Pro; consulted before every message.
- **Contention:** Only the Client changes consent (Messaging & Consent = Own-only for their own consent); the Pro can see but never override. A STOP reply and an in-app re-grant could arrive close together. Resolution: the most recent explicit client action by timestamp wins; if the state is uncertain the no-text state applies (FEAT-14 Error state).
- **Data Sensitivity:** Compliance evidence tied to a phone number (US texting-consent rules, ASMP-24); personal data visible to the Client and their Pro only.
- **Source:** Domain Entity Inventory, product-features.md

### Message

- **Lifecycle:** Created by FEAT-08 (and FEAT-26 from Later; access-link and deposit-request messages on behalf of FEAT-06 and FEAT-30). Read by FEAT-12 (delivery-failure flags), FEAT-16, FEAT-19. Updated: delivery status only (Queued → Sent → Delivered | Failed). Deleted: N/A — immutable once sent.
- **Fields (functional):**
  - type -- confirmation, reminder, change notice, access link, deposit request, Pro notification
  - channel -- text, email (WhatsApp from Later)
  - content_summary, recipient, send time
  - delivery_status
- **Relationships:** Belongs to one Booking (or to the Pro Account for Pro notifications).
- **Contention:** None — messages are written once by the product; no role edits them.
- **Data Sensitivity:** Personal data (recipient contact and appointment details); visible only to the Pro for their own bookings and to support for delivery status.
- **Source:** Domain Entity Inventory, product-features.md

### Subscription

- **Lifecycle:** Created by FEAT-15. Read by FEAT-19. Updated by FEAT-18 (renewal, payment failure, payment-method update, cancellation). Cancelled by FEAT-18 or by FEAT-29 on account closure.
- **Fields (functional):**
  - status -- Active | Payment Failed (7-day grace) | Cancelled (active to period end)
  - billing_cycle / next_billing_date
  - payment_method_reference -- the processor holds the card
  - price -- single all-inclusive tier
- **Relationships:** One per Pro Account; its lapse pauses new bookings (XBR-14).
- **Contention:** The Pro (Subscription & Billing = Full) may update the payment method while an automatic renewal retry is in progress. Resolution: the payment processor's recorded outcome is authoritative; a stale Pro screen is refused with refresh; cancellation is last-write-wins against reactivation before period end.
- **Data Sensitivity:** Financial — billing status and a payment-method reference only; card data held by the processor (ASMP-15).
- **Source:** Domain Entity Inventory, product-features.md

### Waitlist Entry

- **Lifecycle:** Created by FEAT-20. Read by FEAT-10 (cancellation triggers notification), FEAT-12/FEAT-20 (Pro sees demand), FEAT-19. Updated by FEAT-20 (notified, converted, expired). Deleted by FEAT-20 (client leaves).
- **Fields (functional):**
  - service, date or range (up to 7 days)
  - state -- Requested | Notified | Converted | Expired
  - claim_deadline -- 30 minutes after notification
- **Relationships:** Belongs to one Client with one Pro; converts into a Booking.
- **Contention:** Several Clients (Waitlist = Own-only) can be notified of the same opening; a Client may leave while being notified. Resolution: the first client to complete a booking wins the slot (XBR-01); the others remain waiting; a leave request wins over a pending notification.
- **Data Sensitivity:** Personal — reveals a client's desired appointment times; Own-only for the Client, aggregate counts only for the Pro.
- **Source:** Domain Entity Inventory, product-features.md

### Recurring Series

- **Lifecycle:** Created by FEAT-21 (client, or Pro at the chair via FEAT-30). Read by FEAT-03 (reserves future slots). Updated by FEAT-21 (pause, per-occurrence changes). Ended by FEAT-21.
- **Fields (functional):**
  - interval -- every 1–12 weeks
  - originating service and time
  - state -- Active | Paused | Ended
  - generated occurrences -- within the booking horizon
- **Relationships:** Belongs to one Client with one Pro; generates Bookings, each with its own deposit.
- **Contention:** Both the Client (Recurring Appointments = Own-only) and the Pro (Full) can change a series or one occurrence. Resolution: reject-with-refresh — the first committed change wins and the other party sees the updated series before acting.
- **Data Sensitivity:** Personal — a client's standing appointment pattern; visible to that Client and their Pro only.
- **Source:** Domain Entity Inventory, product-features.md

### Payout Account

- **Lifecycle:** Created by FEAT-28 (reached from FEAT-15). Read by FEAT-07 (must be active), FEAT-19 (status only), FEAT-22, FEAT-30. Updated by FEAT-28 (status changes reported by the payment processor, action-required resolution). Disconnected by FEAT-28.
- **Fields (functional):**
  - processor_account_reference -- bank and identity details stay with the processor
  - status -- Not Connected | Verification Pending | Active | Action Required | Disconnected
  - country / currency -- must match the Pro Account
  - payout_schedule and recent payouts -- as reported by the processor
- **Relationships:** One per Pro Account; receives every Deposit Transaction (and, from v1, Balance Payments and tips).
- **Contention:** The Pro (Payouts = Full) acts through the processor's own flow while the processor reports status changes. Resolution: the processor-reported status is authoritative (last-write-wins by the processor's event time); the Pro never edits bank details inside Chairtime.
- **Data Sensitivity:** Financial — bank and identity details are never held by the product; support sees status and the money list only, never bank or identity details (Access Matrix, Payouts).
- **Source:** Domain Entity Inventory, product-features.md

### Activity Event

- **Lifecycle:** Created by FEAT-16 (written automatically as FEAT-05, FEAT-07, FEAT-08, FEAT-10, FEAT-11, FEAT-30 act) and by FEAT-19 (each support view logged). Read by FEAT-11, FEAT-16, FEAT-19. Updated: never. Deleted: never while the booking exists; de-identified after client deletion.
- **Fields (functional):**
  - event_type, time, actor (Client, Pro, the product automatically, or a support view)
  - details -- e.g., policy version and wording shown, message sent, deposit outcome
- **Relationships:** Belongs to one Booking or to the Pro Account (support-view log).
- **Contention:** None — append-only; entries are written once and no role, including the Pro, can edit them.
- **Data Sensitivity:** Contains personal and financial event details; View for the Pro and support only; forms dispute evidence, so immutability is a trust requirement.
- **Source:** Domain Entity Inventory, product-features.md

### Access Link

- **Lifecycle:** Created by FEAT-06 (on-demand "my bookings" link) and FEAT-08 (booking-specific manage links in confirmations and reminders; fresh link after a FEAT-30 reschedule). Read by FEAT-10. Updated by FEAT-06 (used, expired). Deleted: expires automatically.
- **Fields (functional):**
  - scope -- all of this client's bookings with one Pro, or one booking
  - expiry -- 30 minutes and single-use for on-demand links; until the appointment passes for booking-specific links
  - state -- Issued | Used | Expired
- **Relationships:** Belongs to one Client with one Pro (and optionally one Booking).
- **Contention:** A single-use link could be opened on two devices at once. Resolution: the first use wins; the second sees the plain "request a new link" prompt.
- **Data Sensitivity:** Security-sensitive — a bearer link that opens a client's bookings; short-lived and scoped to one client with one Pro (ASMP-30).
- **Source:** Domain Entity Inventory, product-features.md

### Balance Payment

- **Lifecycle:** Created by FEAT-22 (v1). Read by FEAT-12, FEAT-28. Updated by FEAT-23 (tip amount, Later) and FEAT-30 (refund on cancellation). Deleted: N/A — financial record retained.
- **Fields (functional):**
  - amount -- price minus deposit, not client-alterable
  - tip -- optional, non-negative, never pre-selected (Later)
  - state -- Attempted | Succeeded | Failed | Refunded
- **Relationships:** Belongs to one Booking; paid out to the Pro's Payout Account.
- **Contention:** The Client (Booking & Payment = Own-only) may be paying while the Pro (Full) cancels the booking. Resolution: reject-with-refresh — a cancellation committed first blocks the payment; a payment committed first is refunded in full by the cancellation (XBR-23).
- **Data Sensitivity:** Financial — amounts and outcomes only; no card data (ASMP-15).
- **Source:** Domain Entity Inventory, product-features.md

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

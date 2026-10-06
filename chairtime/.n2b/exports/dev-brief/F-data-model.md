# Part F — Data Model

This part is the product's data vocabulary — the entities behind every feature. The physical schema is Part G3.

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


## Entities Shared Across Features

The entities that more than one feature reads or writes, with contention and data-sensitivity notes.

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


